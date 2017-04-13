---
layout: post
title:  "智能 POS 后台接口改造（一）- 服务平台已有接口整理"
date:   2017-04-13
categories: note
tags: api
excerpt: 计划将智能 POS 所涉及到的接口使用 Golang 重新开发。
author: ChenFeng
---

### 一、背景

智能 POS 相关接口一直是服务平台那边开发与维护，考虑到服务平台有时候因为太忙导致的对接不及时问题，决定将这部分接口任务由移动组同事承当。
目前移动组这边的 BigCat 使用的是 Golang 相关技术开发，为了保持技术栈一致，才决定放弃原来的 SpringMVC 从而转向 GO。

### 二、接口
##### 1、接口概览

终端管理类 (path: "/merServPlat/intellectTerm")

| path | method | desc | 
| :-- | :--: | :-- |
| /active | POST | 激活 |
| /activeCode | POST | 激活码激活 |
| /loadTermInfo | POST | 终端参数下载 |
| /findVersion | POST | 更新（查询版本号） |
| /getMerName | POST | 根据激活码获取商户名称 |

交易查询类 (path: "/merServPlat/trans")

| path | method | desc | 
| :-- | :--: | :-- |
| /findTrans | POST | 交易查询 |
| /findTransSumInfo | POST | 交易汇总信息查询 |
| /getTransSettle | POST | 交易结算（根据批次号查询交易明细）|

##### 2、接口详细内容

* /active

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| deviceToken | String | | | 友盟推送的 Device Token |
| timestamp | String | | | 时间戳 |

* /activeCode

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| activeCode | String | | | 激活码 |
| model | String | | | 机器型号（目前 N900 、A8、其他) |
| deviceToken | String | | | 友盟推送的 Device Token |
| timestamp | String | | | 时间戳 |

* /loadTermInfo

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| timestamp | String | | | 时间戳 |

* /findVersion

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| deviceToken | String | | | 友盟推送的 Device Token |
| timestamp | String | | | 时间戳 |

* /getMerName

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| activeCode | String | | | 激活码 |

* /findTrans

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| timestamp | String | | | 时间戳 |
| refNum | String | | Y | 参考号(渠道订单号?) |
| traceNum | String | | Y | 流水号 |
| batchNum | String | | Y | 批次号 |
| txnType | String | | Y | 账单类型(0-银行卡;1-扫码;不传-所有类型) |
| outOrderNum | String | | Y | 外部订单号 |
| page | int | | Y | 第几页(默认1) |
| size | int | | Y | 每页大小(默认50;最大50) |


* /findTransSumInfo

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| timestamp | String | | | 时间戳 |
| txnType | String | | Y | 账单类型(0-银行卡;1-扫码;不传-所有类型) |
| appVersion | String | | Y | 终端风狐应用版本号(用于收集商户的终端当前版本号) |

* /getTransSettle

| field | type | length | optional | desc |
|:-- | :--: | :--: | :--: | :-- |
| merCode | String | | | 商户号 |
| termCode | String | | | 终端号 |
| snCode | String | | | POS 机器的序列号 |
| timestamp | String | | | 时间戳 |
| termBatchId | String | | | 结算批次号 |

> 附：所有接口校验签名: addHeader("signature", SHA256(request))


### 三、数据库

##### 1、相关表概览

| table_name | comment | db |
|:-- |:-- |:-- |
| tbl_apk_version | 应用版本管理表 | mysql |
| intellect_term_config | 终端交易配置表 | mysql |
| term_info | 终端信息表 | mysql |
| term_stage | ? | mysql |
| mer_info | 商户表 | mysql |
| tbl_direct_pos | 交易明细表 | oracle |

##### 2、表结构

* tbl_apk_version 应用版本管理表

| column | comment |
|:--|:--|
| id | 主键 |
| apk_name | 应用名称,默认smartpos(渠道名称） |
| version_num |	应用版本名 |
| version_code | 应用版本号(版本时间) |
| apk_package_name | Apk包名(上传的文件名称） |
| apk_url | 下载url |
| change_log | 更新日志 |
| apk_time | 更新时间 |
| check_sum | apk包特征值 |

* intellect_term_config 终端交易配置表

| column | comment |
|:--|:--|
| intellect_term_id | 主键 |
| application | 接口选项(交易渠道)。接口实现插件化，程序可根据选择的插件组装不同的报文；同时可以定义不同的打印格式；接口可约定         例如：讯联数据8583接口-“cil-8583”；讯联数据Json接口-“cil-json”； |
| decryption | 加密算法1DES、2DES、3DES、SM4 |
| device | 品牌（同一品牌，sdk相同） |
| print | 打印参数 |
| link_name | 通讯名称 |
| trans_apn	| APN号 |
| trans_apn_user | apn用户名 |
| trans_apn_passwd | apn密码 |
| trans_ip1 | 交易IP（主） |
| trans_port1 | 交易端口（主） |
| trans_ip2 | 交易IP（备） |
| trans_port2 | 交易端口（备） |
| trans_tpdu | TPDU |
| trans_outtime | 交易超时时间，单位：秒 |
| describe | 方案描述 |

* term_info 终端信息表

| column | comment |
|:--|:--|
| term_id |	终端Id，全局唯一 |
| mer_id |	商户id |
| term_code |	终端编号 |
| term_serial |	终端序列号 |
| term_address |	终端地址 |
| term_status |	终端状态：0-签到,1-签退,2-删除,3-锁定,5-初始 |
| term_model_id |	终端型号id |
| dial_phone |	拨号电话 |
| support_consumer |	支持消费 |
| support_undo |	消费撤销 |
| support_return |	退货 |
| support_inquiry |	余额查询 |
| support_pre_auth |	预授权 |
| support_undo_auth |	 |
| support_completed |	 |
| is_delete |	该记录是否已删除 |
| term_type |	1固定,2 移动 ,3 智能 |
| create_time |	记录创建时间 |
| modify_time |	最好修改时间 |
| SUPPORT_IC_CARD |	支持IC卡：0-不支持，1-支持 |
| mer_code |	商户编号 |
| tips_support_flag |	支持小费：0-不支持，1-支持 |
| support_func_flag |	支持功能 |
| stage_id |	 |
| term_model_desc |	终端品牌和型号 |
| tmk_down_method |	TMK下载方式 |
| tmk_down_flag |	TMK下载标志  |
| msg_encrypt_flag |	报文加密 |
| device_token |	智能终端token (term_type =3 时有用) |
| intellect_term_id |	智能终端参数配置id (term_type =3 时有用) |
| open_time	| |
| value_card |	是否开通百馏卡 |
| app_version |	 |
| last_app_version |	 |
| cd_key |	激活码 |

* term_stage ？

| column | comment |
|:--|:--|
| TERM_ID |	终端Id，全局唯一 |
| MER_ID |	商户id |
| TERM_CODE |	终端编号 |
| TERM_SERIAL |	终端序列号 |
| TERM_ADDRESS |	终端地址 |
| TERM_STATUS |	终端状态：1-初始，2－锁定，3-撤机 |
| TERM_MODEL_ID |	 |
| DIAL_PHONE |	拨号电话 |
| SUPPORT_CONSUMER |	 |
| SUPPORT_UNDO |	 |
| SUPPORT_RETURN |	 |
| SUPPORT_INQUIRY |	 |
| SUPPORT_PRE_AUTH |	 |
| SUPPORT_UNDO_AUTH |	 |
| SUPPORT_COMPLETED |	 |
| IS_DELETE	|是否已删除：1-未删除，2-已删除 |
| TERM_TYPE |	1 固定, 2 移动 ,3 智能 |
| create_time |	记录创建时间 |
| modify_time |	最好修改时间 |
| SUPPORT_IC_CARD |	支持IC卡：0-不支持，1-支持 |
| MER_CODE |	商户编号 |
| TIPS_SUPPORT_FLAG |	支持小费：0-不支持，1-支持 |
| SUPPORT_FUNC_FLAG |	支持功能 |
| STAGE_ID |	与mer_stage表的stage_id关联 |
| TERM_MODEL_DESC |	终端品牌和型号 |
| TMK_DOWN_METHOD |	TMK下载方式 |
| TMK_DOWN_FLAG	 |TMK下载标志 |
| MSG_ENCRYPT_FLAG |	报文加密 |
| DEVICE_TOKEN |	智能终端token (term_type =3 时有用) |
| INTELLECT_TERM_ID |	智能终端参数配置id (term_type =3 时有用) |
| OPEN_TIME	 | |
| value_card |	是否开通百馏卡 |
| app_version |	 |
| last_app_version |	 |
| cd_key |	激活码 |

* mer_info 商户表

* term_model 终端型号表

| column | comment |
|:--|:--|
| term_model |	终端型号 |
| brand_name |	终端品牌 |
| term_model_id |	 |
| is_intellect |	是否是智能终端 |

* tbl_direct_pos 交易明细表

| column | comment |
|:--|:--|
|||


