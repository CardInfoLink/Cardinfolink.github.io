---
layout: post
title:  "PAX A920 x EvoCashier"
date:   2018-10-03
categories: note
tags: PAX EvoCashier
excerpt: PAX A920 上安装 EvoCashier，在他们原来的 APP 上加一个按钮，唤起 EvoCashier...
author: ChenFeng
---

## Preview

<video controls
       muted
       src="../../../../image/device-2018-10-03-123400.mp4"
       width="240"
       height="480">
    Sorry, your browser doesn't support embedded videos.
</video>

## Code

Some code should add to your projects.

### APP A(PAX A920 Cashier):

* Add a button named FPS in your appliaction

![](../../../../image/fps.png)

* Implement the OnClickListener :

```java
    Uri data = Uri.parse("cil://www.everonet.com/payment");
    intent = new Intent(Intent.ACTION_VIEW, data);
    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    try {
        startActivityForResult(intent, 666);
    } catch (Exception e) {
        e.printStackTrace();
        Toast.makeText(getActivity(), "the EvoCashier is not installed", Toast.LENGTH_SHORT).show();
    }
```

### APP B(EvoCashier):

`APP B` needs `APP A`'s `package name` and `application id` for back to `APP A`, such as:

* packageName: "com.cardinfolink.smart.pos"
* applicationId: "com.cardinfolink.smart.pos"
