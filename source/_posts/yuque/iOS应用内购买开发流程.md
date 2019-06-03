
title: iOS应用内购买开发流程
date: 2018-12-25 10:30:27 +0800
categories: iOS
---


<a name="o659uw"></a>
## 前期准备

1. 完善协议、税务和银行业务等信息 [具体可参考此文章](https://juejin.im/post/5a7b0e97f265da4e8e784c9e)

2. 为app添加内购产品 一般而言产品ID使用bundleId+产品信息

3. 添加沙箱技术测试员


<a name="3za6sp"></a>
## [](#3za6sp)客户端代码流程

1. 封装内购单例 AppDelegate中应用启动时调用

2. 在单例的init方法中注册对购买结果的监听

3. 用户点击充值时，调用SKProductsRequest发起一个请求，请求商品信息

4. 商品信息查询成功，请求支付

5. 处理payment的监听回调

6. 交易完成 向服务器端发送凭证


<a name="yp2bsd"></a>
## [](#yp2bsd)服务端代码流程

1. 接受iOS端发送过来的购买凭证

2. 判断凭证是否已经存在或验证过，存储凭证

3. 将凭证发送到苹果服务器验证

4. 根据验证结果修改用户的会员权限


<a name="7cg3rw"></a>
## [](#7cg3rw)自动续期订阅的注意点

1. 启动针对自动续期订阅的服务器通知

2. 客户端启动时可能会收到苹果给的续期成功的回调


<a name="gax1xn"></a>
## [](#gax1xn)恢复购买


<a name="fcq5mb"></a>
## [](#fcq5mb)漏单、退单的处理

<a name="8gvxvb"></a>
## [](#8gvxvb)参考文章
[链接](https://juejin.im/post/5c204bdf5188256d98331363)


