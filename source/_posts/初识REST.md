---
title: 初识RESTful
date: 2017/10/29
tags:
- 构架
---
RESTful API 是目前比较成熟和流行的互联网应用程序API设计理论，REST是表述性状态转移(REpresentational State Transfer)的简称。它是一种软件架构风格，设计风格而不是标准，只是提供了一组设计原则和约束条件。
<!--more-->

REST 关键原则
* 为所有“事物”(资源)定义ID
* 将所有事物链接在一起
* 使用标准方法
* 资源多重表述
* 无状态通信


### 为所有“事物”(资源)定义ID
对事物使用一致的命名规则，将每个可标记的资源使用一个唯一的标志来标识(URI)
```
http://example.com/customers/1234
http://example.com/orders/2007/10/776654
http://example.com/products/4554
http://example.com/processes/salary-increase-234 

http://example.com/orders/2007/11
http://example.com/products?color=green 
```

### 将所有事物链接在一起
任何可能的情况下，使用链接指引可以被标识的事物（资源）。也正是超链接造就了现在的Web。  
换句话说：是链接的思想。链接是我们在HTML中常见的概念，但是它的用处绝不局限于此（用于人们网络浏览）。
```
<order self="http://example.com/customers/1234"> 
   <amount>23</amount> 
   <product ref="http://example.com/products/4554"> 
   <customer ref="http://example.com/customers/1234"> 
</customer> </product></order>
```

### 使用标准方法
为使客户端程序能与你的资源相互协作，资源应该正确地实现默认的应用协议（HTTP），也就是使用标准的GET、PUT、POST和DELETE方法。

### 资源多重表述
针对不同的需求提供资源多重表述。

### 无状态通信
一个客户端从某台服务器上收到一份包含链接的文档，当它要做一些处理时，这台服务器宕掉了，可能是硬盘坏掉而被拿去修理，可能是软件需要升级重启——如果这个客户端访问了从这台服务器接收的链接，它不会察觉到后台的服务器已经改变了。

具体实施可以参考 [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)