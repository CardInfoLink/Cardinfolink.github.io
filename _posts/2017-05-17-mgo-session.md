---
layout: post
title:  "mgo 的 session 与连接池"
date:   2017-05-17
categories: note
tags: issues
excerpt: mgo是由Golang编写的开源mongodb驱动。由于mongodb官方并没有开发Golang驱动，因此...
author: JodeZer
---

### 简介
&#160; &#160; &#160; &#160;mgo是由Golang编写的开源mongodb驱动。由于mongodb官方并没有开发Golang驱动，因此这款驱动被广泛使用。mongodb官网也推荐了这款开源驱动，并且作者在github也表示受到了mongodb官方的赞助。但由于作者的个人安排原因，该驱动的更新、bug修复、issue维护略微受到诟病。

&#160; &#160; &#160; &#160;mgo在功能方面还是比较完善的，api使用也方便。由于mongodb丰富的玩法，mgo代码庞大，其中大部分是与mongodb的协议代码。核心的处理连接和请求的结构，逻辑上还是比较清晰的。

### 简单的使用
```go
func dial() {
	session ,_ := mgo.Dial("mongodb://127.0.0.1")
}
```
mgo面向调用者的核心数据结构是`mgo.Session`，dial函数演示了如何获取一个session
```go
func foo1() {
	session.DB("test").C("coll").Insert(bson.M{"name":"zhangsan"})
}
```
foo1函数通过生成的session，向test数据库的coll集合写入了一条数据。

但mgo的正确使用方法并非如此，而是应该在每次使用时从源session拷贝
```go
func foo2() {
	s := session.Copy()
	defer s.Close()
	s.DB("test").C("coll").Insert(bson.M{"name":"zhangsan"})
}
```
foo2函数从源session拷贝出了一个临时的session，使用临时session写入一条数据，在函数退出时关闭这个临时的session。

### session的拷贝与并发
&#160; &#160; &#160; &#160;为什么要在每次使用时都Copy，而不是直接使用Dial生成的session实例呢？个人认为，这与`mgo.Session`的Socket缓存机制有关。来看Session的核心数据结构。
```go
type Session struct {
	m                sync.RWMutex
	...
	slaveSocket      *mongoSocket
	masterSocket     *mongoSocket
	...
	consistency      Mode
	...
	poolLimit        int
	...
}
```
这里列出了`mgo.Session`的五个私有成员变量，与Copy机制有关的是，m,slaveSocket,masterSocket。

`m`是`mgo.Session`的并发锁，因此所有的Session实例都是线程安全的。

`slaveSocket`,`masterSocket`代表了该Session到mongodb主节点和从节点的一个物理连接的缓存。而Session的策略总是优先使用缓存的连接。是否缓存连接，由`consistency`也就是该Session的模式决定。假设在并发程序中，使用同一个Session实例，不使用Copy，而该Session实例的模式又恰好会缓存连接，那么，所有的通过该Session实例的操作，都会通过同一条连接到达mongodb。虽然mongodb本身的网络模型是非阻塞通信，请求可以通过一条链路，非阻塞地处理；但经过比较简陋的性能测试，在mongodb3.0中，10条连接并发写比单条连接的效率高一倍（在mongodb3.4中基本没有差别）。所以，使用Session Copy的一个重要原因是，可以将请求并发地分散到多个连接中。

以上只是效率问题，但第二个问题是致命的。`mgo.Session`缓存的一主一从连接，实例本身不负责维护。也就是说，当`slaveSocket`,`masterSocket`任意其一，连接断开，Session自己不会重置缓存，该Session的使用者如果不主动重置缓存，调用者得到的将永远是`EOF`。这种情况在主从切换时就会发生，在网络抖动时也会发生。在业务代码中主动维护数据库Session的可用性，显然是不招人喜欢的。

```go
func (s *Session) Copy() *Session {
	s.m.Lock()
	scopy := copySession(s, true)
	s.m.Unlock()
	scopy.Refresh()
	return scopy
}
```
以上是Copy函数的实现，解决了使用全局Session的两个问题。其中，`copySession`将源Session浅拷贝到临时Session中，这样源Session的配置就拷贝到了临时Session中。关键的`Refresh`，将源Session浅拷贝到临时Session的连接缓存指针，也就是`slaveSocket`,`masterSocket`置为空，这样临时Session就不存在缓存连接，而转为去尝试获取一个空闲的连接。

### Session的连接从哪里来？连接池
明确了使用Session Copy机制的必要性，那么问题来了，Copy出来的临时Session是怎么获取一个到mongodb的物理连接的。答案就是连接池。mgo自身维护了一套到mongodb集群的连接池。这套连接池机制以mongodb数据库服务器为最小单位，每个mongodb都会在mgo内部，对应一个`mongoServer`结构体的实例，一个实例代表着mgo持有的到该数据库的连接。来看该连接池的定义。
```go
type mongoServer struct {
	sync.RWMutex
	...
	unusedSockets []*mongoSocket
	liveSockets   []*mongoSocket
	...
	info          *mongoServerInfo
}
```
其中，`info`代表了该实例对应的数据库服务器在集群中的信息——是否master，ReplicaSetName等。而两个Slice，就是传说中的连接池。`unusedSockets`存储当前空闲的连接，`liveSockets`存储当前活跃中的连接，Session缓存的连接就同时存放在`liveSockets`切片中，而临时Session获取到的连接就位于`unusedSockets`切片中。

每个`mongoServer`都会隶属于一个`mongoCluster`结构，相当于mgo在内部，模拟出了mongo数据库集群的模型。
```go
type mongoCluster struct {
	sync.RWMutex
	...
	servers      mongoServers
	masters      mongoServers
	...
	setName      string
	...
}
```
如定义所示，`mongoCluster`持有一系列`mongoServer`的实例，以主从结构分散到两个数组中。

每个Session都会存储自己对应的，要操作的`mongoCluster`的引用。

### 长途跋涉的Session
&#160; &#160; &#160; &#160;以下描述一个Copy出来的临时Session是如何获取到一个mongodb物理连接的。

当临时Session被Copy出来，并且通过调用一系列api，将一次数据库操作设置到了Session内部后，此时万事俱备，只差连接。新生的Session首先会检查自己的缓存里是否有连接可用，初来乍到的他当然不知道自己是一个一无所有的光杆司令。由于mgo的实现，可怜的他还要去检查两次，一次使用读锁，一次使用写锁。作者的意图应该是期望在对同一个session并发操作时，能在第二次排他锁检查之前，恰巧缓存到一条连接，那么就可以减少一次对连接池的操作。但这次，这种好事没有发生在这个Session身上，“摸”了两次“口袋”反复确认以后，他终于还是发现自己身无分文。没有连接的他向组织求救，也就是这个session所要操作的mongodb集群，也就是所提到的`mongoCluster`结构。


“组织”问了这个Session一系列问题，其中最主要的是两个问题，一是”你要主库连接还是从库连接”，二是“你期望的连接池最大大小是多少”。第一个问题，Session很好回答，他首先看了看自己的模式，是必须到主库还是必须到从库，还是两者皆可看情况而定。再看了看自己手里的操作是读还是写，写操作当然不可能到从库去完成。第二个问题就有点强人所难，但是他不用自己思考，因为这是从源Session那里拿过来的配置，也算是一点祖产吧。

这个Cluster此时表现得像一个掌柜，他先根据主从，从自己手下的`mongoServer`里挑出了一个，然后问他，你现在手里有没有空闲的连接。如果有，那幸运的Session就可以顺利地获取到这个空闲的连接，高高兴兴的揣到兜里回家干活。但如果不巧，正好`unusedSockets`为空，那么掌柜会问另一个问题，你有没有超过这个家伙的期望的最大连接数。如果没有超过，那还好，作为伙计的`mongoServer`就干活了，他会跑到他负责的数据库服务器那里去申请一条全新的连接，亲手交到Session的手里。但如果这个伙计算了算，还去申请新连接的话，恐怕就超限了，那就Session同学对不起了您，您等吧。每100ms,伙计自旋一次，等着`unusedSockets`里出现可用的连接。

当然有人会问，那这么自旋下去，如果连接一直被其他Session占用，会不会就死循环了呢，答案是不会。这个伙计作为一个数据库服务器的管理员吧可以说，他自己也要常常去确认他负责的这个服务器是不是还活着。因此，伙计同学每15s会给服务器发一个`ping`命令。作为管理员，伙计可就不管什么连接池大小超不超的问题了，那是他们那些普通session要考虑的琐事。伙计同学要ping的时候，也去`unusedSockets`里看，如果有最好，就拿一个来用；没有的话，直接去问服务器要新的。ping完之后，新的连接就会被放入`unusedSockets`中。这样的话，自旋中的获取连接请求，就可以拿到连接了。

经过摸口袋，找组织，问伙计，伙计再干点小活，临时Session终于拿到了梦寐以求的数据库物理连接，把他放到了自己的口袋里（当然有些模式的Session不会这么干）。心满意足地将自己手里的操作通过这条连接写了出去，等到数据库给了他想要的应答，他的生命也就结束了。通过Close方法，我们剥夺了他口袋里的得之不易的连接，放回到了对应`mongoServer`的`unusedSockets`中。不久之后，GC又杀死了这个Session。

### 为什么我司的代码没有使用Copy也没有出问题？
看过我司Go项目代码的同学可能知道，我司的服务端代码中并没有使用Copy，而是类似如下的使用
```go
func Dial(){
	localSession ,_ := mgo.Dial(mongoUrl)
	localSession.SetMode(mgo.Eventual)
	globalDatabase = localSession.DB("db")
}

func Insert() {
	globalDatabase.C("coll").Insert(...)
}
```
使用了一个全局的`mgo.Database`实例，所有的对该db的操作，都通过这个实例完成。
原因就在于，我们使用的是模式是`mgo.Eventual`，该模式最大的特点就是不会缓存连接，拒绝持有mongodb的一针一线。通过该`mgo.Database`实例的操作，每次都会发现自己的口袋里一无所有，都会经过一次上一节所述的长途跋涉获取连接，因此也规避了不使用Copy带来的两个副作用。一并发效率问题，Eventual的Session每次操作都从连接池取连接，相当于分散在连接池中完成了操作，二连接可用性问题，连接池机制确保了，从`mongoServer`取得的连接，都是活的连接。

### Copy机制或Eventual模式的并发模型的问题
#### 并发锁效率
Copy机制或Eventual模式的共同点是，每次的数据库操作都要经过一次代码路径略深的获取连接的过程。而这个路径中，会操作多个线程安全的结构体，包括`mongoServer`、`mongoCluster`等，线程安全的代价就是并发锁冲突带来的性能下降。举个例子，假设有十个写操作并发，无论使用Copy还是Eventual，最终都会走到cluster的masterServer，请求一个主库连接；而完成这个请求，需要写两个slice，将连接从`unusedSockets`删除，并加入`liveSockets`，对切片的更新势必要加排他锁，因此这十个请求很有可能会产生锁冲突。
#### 连接池上限与冲击数据库
mgo对Session有一个poolLimit配置，也就是上文中所说的cluster问session的第二个问题——代表了对连接池连接数的上限限制。默认配置的连接数上限是4096，显然对生产环境来说太过大了。但这个配置我以为非常的鸡肋，属于设置也不好，不设置也不好；个人认为这是被mgo的并发模型所拖累了。

假设高并发场景，若设置的连接池上限为4096，并发为10000，那么理论上，一瞬间，mgo可能会产生4096个到mongodb的物理连接，而剩下的六千的请求会自旋等待。4096个连接对mongodb来说，首先意味着4096 * 10M的内存消耗，如此高的连接会导致各种各样的问题。那么加入设置连接池上限为100，并发为10000，9900个等待的请求每次100个排队完成，对应用的效率又是不小的消耗。况且实际测试中，poolLimit的设置也无法严格地限制住连接数。
#### 连接池只伸不缩
mgo另一个问题是连接池连接不释放，一旦由于并发原因，连接池的数量被撑大，之后再也不会变小，除非客户机或服务器重启。

### M:1? 1:1 ? M:N!
排除连接可用性问题，全局缓存连接的Session的问题是M个数据库操作通过1个连接完成。通过Copy、Eventual完成数据库操作的问题是，取到一个连接后，只做一件事情就归还了连接。这两种并发模型都存在问题。因此，最好的模型是M:N，有M个数据库操作需要完成，一次性取N个连接，分散到N个连接中完成，此后无论有多少批请求，都可以在N个连接中分散完成。第一可以规避连接池锁冲突，第二不会大规模产生真实连接，充分利用已建立的连接。

### SessionPool
M:N的模型无法通过mgo原生支持完成，api也无法支持用户获取到物理连接。

可以利用Session会缓存连接的特性，通过一些小技巧实现一个SessionPool。例如，有M个写操作，则可以一次性生成N个StrongSession，每个StrongSession自己会缓存一条`masterSocket`；于是，之后的写操作，可以以某种方式负载均衡到这N个由Strong模式的Session缓存的连接中。

[mgop](http://github.com/JodeZer/mgop)简单实现了上述的StrongSessionPool，以轮询的方式负载。对从库连接的缓存以及动态负载还有待实现。

实现SessionPool要特别注意的问题是刷新问题，缓存Session中的连接随时可能会失效，mgop的方式是遍历发送`isMaster`命令，第一确认连接存活，第二确认连接确实是到主库的。若发现问题，则马上重置缓存。
### 空闲连接释放
mgo的连接池释放问题，在我的mgo fork中做了一个简单的实现解决这个问题。[github.com/JodeZer/mgo](http://github.com/JodeZer/mgo)

在mongodb url标准中，有两个option：minPoolSize，maxIdleTimeMS。mgo没有支持这两个选项，通过实现这两个选项可以达到释放连接的目的。官网描述：
``````
minPoolSize
The minimum number of connections in the connection pool. The default value is 0.

maxIdleTimeMS
The maximum number of milliseconds that a connection can remain idle in the pool before being removed and closed.
```
实现方式是，将`mongoServer`中的`unusedSockets`类型改造为`timedMongoSocket`(fork中自定义的类型)
```go
type timedMongoSocket struct {
	soc          *mongoSocket
	lastTimeUsed *time.Time
}
```
每次有连接被重置到空闲池时，打一个时间戳。在轮询goroutine中每隔一段时间review空闲连接的空闲时长，当时长大于`maxIdleTimeMS`时，就释放连接，将空闲池的大小控制在`minPoolSize`。

实现中，没有特地写设置函数，可以通过在mongo url中写入选项设置，如：`mongodb://127.0.0.1?minPoolSize=0&maxIdleTimeMS=3000`，若`maxIdleTimeMS`不设置或为0，则默认为不进行释放
