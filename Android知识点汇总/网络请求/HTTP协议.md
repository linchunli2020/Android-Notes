【 HTTPS 】

***简介***

HTTP 的端口号是 80，HTTPS 是 443，HTTPS 需要到 CA 申请证书，一般免费证书很少，需要交费
SSL 的全称是 Secure Sockets Layer，即安全套接层协议，是为网络通信提供安全及数据完整性的一种安全协议。SSL协议在1994年被Netscape发明，后来各个浏览器均支持 SSL，其最新的版本是 3.0
TLS 的全称是 Transport Layer Security，即安全传输层协议，最新版本的 TLS是 IETF 制定的一种新的协议，它建立在 SSL 3.0 协议规范之上，是SSL 3.0的后续版本。在 TLS 与SSL 3.0 之间存在着显著的差别，主要是它们所支持的加密算法不同，所以 TLS 与 SSL3.0 不能互操作。虽然 TLS 与 SSL 3.0 在加密算法上不同，但在理解 HTTPS 的过程中，可以把 SSL 和 TLS 看做是同一个协议。
SSL（Secure Sockets Layer 安全套接层)，及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密。

****加密原理****

HTTPS 为了兼顾安全与效率，同时使用了对称加密和非对称加密。数据是被对称加密传输的，对称加密过程需要客户端的一个密钥，
为了确保能把该密钥安全传输到服务器端，采用非对称加密对该密钥进行加密传输，总的来说，对数据进行对称加密，对称加密所要使用的密钥通过非对称加密传输。


【 HTTP和HTTPS的区别 】

 1、https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。
 
 2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。 
 
 3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
 
 
【 TCP协议：传输层协议 】

***TCP的三次握手-建立连接***

<img width="566" alt="image" src="https://user-images.githubusercontent.com/67937122/161369298-233343be-1da1-4cf5-9b3d-99a74e9763a0.png">

“三次握手”的目的：

防止已经失效的连接请求报文段突然又传送到了服务端，因而产生错误。

***TCP的四次挥手-断开连接-连接终止协议***

<img width="652" alt="image" src="https://user-images.githubusercontent.com/67937122/161370231-01ea08bd-19f9-40f7-beb8-2476b4e4aae5.png">

为什么需要四次挥手？

在Tcp连接握手时，为何ack是和syn一起发送，这里ack并没有和fin一起发送呢？
原因是因为tcp是全双工模式，接收到fin时意味将没有数据再发来，但是还是可以继续发送数据。


【 TCP与UDP的区别 】

（1）TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接

（2）TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保证可靠交付

（3）UDP具有较好的实时性，工作效率比TCP高，适用于对高速传输和实时性有较高的通信或广播通信。

（4）每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信

（5）TCP对系统资源要求较多，UDP对系统资源要求较少。

（6）TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道


<img width="677" alt="image" src="https://user-images.githubusercontent.com/67937122/161370652-24db0cf5-0796-4926-9b85-31275a4f3d5c.png">




参考：

https://github.com/hunanmaniu/AndroidNotes/blob/main/Docs/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80.md#tcp-%E4%B8%8E-udp-%E7%9A%84%E5%8C%BA%E5%88%AB

https://blog.csdn.net/qq_33229669/article/details/101905021
