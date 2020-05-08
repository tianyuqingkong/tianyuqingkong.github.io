---
title: SSL/TLS
date: 2020/3/6
tags:
- nodejs docker tcpdump
---

## 原始加密
假设A向B通信没有任何的加密，中间一个C拦截掉A的消息后，那么A与B之间的通信毫无隐私可言。那么就出现了第一种加密方式。  

`对称加密`：加密和解密使用同一个密钥。
接下来的A向B通信方式是：
* 1 协商一个两者都有的密钥  
* 2 A用密钥将消息加密后传输  
* 3 B得到加密后的消息，用同一个密钥解密得到原始消息  

优点：   
就算C拦截掉了A发给B加密的信息，没有密钥那么C也没办法获取到原始信息

缺点:  
* 1 协商密钥只能线下获取，否则密钥被C拿到到了一样C就可以得到A和B之间通信的消息
* 2 通过暴力破解的方式可以破解56bit的DES加密信息，为了应对这样的情况只好提出新的协议如 triple-DES(最长168bit)、AES(最长256bit)，在目前的计算机计算来看无法短期内被暴力破解

## 升级加密
由于A可能和很多人通信，不可能每次通信前必须线下交换密钥。接下来出现了可以将自己的 pulic key（公钥）在网上传播的加密方式。 

`非对称加密`： 每个用户有两个密钥一个public key(公钥)另一个private key（私钥），加密用其中的一个key解密必须用另一个key。（比如使用pulick key加密的信息就只能用private key来解密，抑或加密用private key解密就只能用public key）双方各执持有对方的public key，各执保留自己的private key。  

这样C拿到一方的public key没有private key也是没办法破解消息。

接下来的A向B通信方式是：
* 1 A和B在网上交换各自的public key，各自保留自己的private key
* 2 A将要发送的数据进行hash计算出一段hash值，用自己private key对这段hash值进行加密。A将要发送的数据使用B的public key进行加密。最后将加密后的hash以及加密后的数据加上一些其他东西发送给B
* 3 B收到加密后的hash以及加密的数据。B用A的public key将加密后hash解密得到原始hash。B用自己的private key解密加密后的数据得到原始数据，在将原始数据进行hash计算得到hash值，与前面解密得到的hash对比。如果一致那么数据可以使用。

优点：  
* 1 就算知道public key也没办法解密消息

缺点：
* 1 非对称加密由于数学原因相对于对称加密要消耗更多（指数级别）的CPU资源
* 2 非对称加密情况下，如果C分别获取了A和B的public key并且用自己的public key分别发给A和B,就可能出现以下情况。 
  * 2.1 A将用自己的private key加密数据的hash值，用C的public key加密数据
  * 2.2 C拦截到加密后的hash值，以及加密后的数据。C用自己的private key解密加密后的数据（这样就得到原始数据），并且C将篡改后的原始数据计算hash值，并且用C自己的private key计算出加密后hash值，再用先前拦截到B的public key加密篡改后的数得到加密后的数据。将篡改后的加密hash值、加密数据发送给B
  * 2.3 B收到以后两个数据后，先用自己的private key解密数据之后计算hash值，再用C传来的假public key解密hash值，发现匹配。

  ## 现代加密
  上面非对称加密的第二个缺陷的关键问题：public key无法保证是对方的。  

  CA（Certificate Authority）证书认证就算解决这个问题的，CA的工作方式如下  
  1： B先将自己的public key交给


