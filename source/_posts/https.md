---
title: 一文理解HTTPS
category: protocal
tag: http 
---
## HTTPS理论基础

### http 通信有什么问题呢

#### 可能被窃听

- HTTP 本身不具备加密的功能,HTTP 报文使用明文方式发送
- 由于互联网是由联通世界各个地方的网络设施组成,所有发送和接收经过某些设备的数据都可能被截获或窥视。(例如大家都熟悉的抓包工具:Wireshark),即使经过加密处理,也会被窥视是通信内容,只是可能很难或者无法破解出报文的信息而已

#### 认证问题

- 无法确认你发送到的服务器就是真正的目标服务器(可能服务器是伪装的)
- 无法确定返回的客户端是否是按照真实意图接收的客户端(可能是伪装的客户端)
- 无法确定正在通信的对方是否具备访问权限,Web 服务器上某些重要的信息，只想发给特定用户
- 即使是无意义的请求也会照单全收。无法阻止海量请求下的 DoS 攻击（Denial of Service，拒绝服务攻击）。

#### 可能被篡改

请求或响应在传输途中，遭攻击者拦截并篡改内容的攻击被称为中间人攻击（Man-in-the-Middle attack，MITM）。

### HTTPS如何解决上述三个问题

HTTPS 只是在通信接口部分用 TLS（Transport Layer Security）协议代替而已。
![](/uploads/https1.png)

SSL 和 TLS 的区别:
> 传输层安全性协议（英语：Transport Layer Security，缩写作 TLS），及其前身安全套接层（Secure Sockets Layer，缩写作 SSL）是一种安全协议，目的是为互联网通信，提供安全及数据完整性保障。网景公司（Netscape）在1994年推出首版网页浏览器，网景导航者时，推出HTTPS协议，以SSL进行加密，这是SSL的起源。IETF将SSL进行标准化，1999年公布第一版TLS标准文件。随后又公布RFC 5246 （2008年8月）与 RFC 6176 （2011年3月）。

**TLS是SSL的标准**. **HTTPS** 就是 **HTTP + SSL**
下面我们就要详细了解一下SSL

#### SSL 协议的实现需要用到的几类算法

##### 对称加密

常用的加密算法:`DES`、`AES` ,'RC2','RC4'
优点：算法公开、计算量小、加密速度快、加密效率高。
缺点：交易双方都使用同样钥匙，安全性得不到保证。
![](/uploads/https2.png)

##### 非对称加密技术

常用的非对称加密算法:`RSA`,`DSA`,`Diffie-Hellman`
优点:安全性高
缺点:速度慢

![](/uploads/https3.png)

##### 完整性验证算法

HASH算法：MD5，SHA1，SHA256
用作校验消息的完整性

##### SSL协议

> 由于非对称加密的速度比较慢，所以它一般用于密钥交换，双方通过公钥算法协商出一份密钥，然后通过对称加密来通信，当然，为了保证数据的完整性，在加密前要先经过HMAC的处理(HMAC是密钥相关的哈希运算消息认证码，HMAC运算利用哈希算法，以一个密钥和一个消息为输入，生成一个消息摘要作为输出)

一般为: 非对称加密+对称加密+SSL证书

##### 密钥协商的过程如下

[点击查看大图](/uploads/https_ssl.svg)
![](/uploads/https_ssl.svg)
[来源](https://upload.wikimedia.org/wikipedia/commons/a/ae/SSL_handshake_with_two_way_authentication_with_certificates.svg)

简单描述下流程:


## 参考资料

- [详解 HTTPS、TLS、SSL、HTTP区别和关系](https://www.wosign.com/info/https_tls_ssl_http.htm)
- [https 那些事儿](http://galaxylab.org/https%E9%82%A3%E4%BA%9B%E4%BA%8B/)
- [WIKI传输层安全性协议](https://zh.wikipedia.org/zh-cn/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)
- [图解TCP/IP]()
- [绿盟月刊SSL/TLS/WTLS原理](http://www.nsfocus.net/index.php?act=magazine&do=view&mid=841)