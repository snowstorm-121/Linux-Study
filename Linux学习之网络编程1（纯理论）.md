## 写在前面

刚刚更新完Linux系统编程，特别推荐大家去看的[Linux系统编程](https://www.bilibili.com/video/BV1KE411q7ee/?spm_id_from=333.337.search-card.all.click&vd_source=3f806d18e0576cd0e3a9f200d67a10f6)，总共44个小时，老师讲的非常好，我是十天肝完的，每天大概看20集，每天还要以写blog的形式来写笔记来总结一下，虽然这十天有点累，但是这套视频值得这些时间的付出。我现在在学习[Linux网络编程](https://www.bilibili.com/video/BV1iJ411S7UA/?p=108&spm_id_from=pageDriver&vd_source=3f806d18e0576cd0e3a9f200d67a10f6)，学习看的视频还是Linux系统编程同一个老师讲的，从今天开始更新Linux网络编程的笔记，先立一个flag：争取十天更新完（虽然我觉得这不太可能，但起码要在今年过年前更新完）

---

## 协议

### OSI七层模型

- 物理层：OSI参考模型中的最底层，面向实际承担数据传输的物理媒介，即通信通道。简单来说就是确保原始数据课在各种物理媒介上传输。
- 数据链路层：定义了单个链路上如何传输数据。
- 网络层：定义了端到端数据包传输，能够标识所有结点的逻辑地址，路由实现的方式和学习方式。
- 运输层：又叫传输层。任务是向两台主机之间的进程通信提供通用的数据传输服务，应用进程利用该服务传送应用层报文。
- 会话层：定义了如何开始，控制和结束一个会话，包括对多个双向消息的控制和管理，以便在只完成连续信息的一部分时可以通知应用程序，使表示层看到的数据是连续的。
- 表示层：定义数据格式及加密。
- 应用层：OSI体系结构的最高层。任务是通过应用进程的交互来完成特定的网络应用。
  
  > 这个OSI七层模型**只用了解即可**，我们后面写代码不用这个七层模型，用的是TCP/IP四层协议体系结构。
  > 可以简记为“**物数网传会应表**”

### TCP/IP四层协议体系结构

- 应用层：面向不同网络应用引入不同的应用层协议。FTP，HTTP，NFS，SSH
- 传输层：使源端主机和目的端主机上的对等实体可以进行对话。TCP，UDP
- 网络层：整个协议体系结构的核心。功能是把分组发往目的网路或主机。IP，ICMP，IGMP
- 链路层（网络接口层）：实际上，TCP/IP协议体系并没有真正描述网络接口层的实现，只是要求能够提供给网络层一个访问接口，以便在网络接口上传递IP分组。以太网帧协议，APR
  
  > 可以简记为“**网网传应**”

![1](https://img-blog.csdnimg.cn/img_convert/4a69d65c034dd35987fc151023b8dd24.png#pic_center)

---

## 网络基础概念

### MAC地址

介质访问控制（Media Access Control,MAC)地址也称硬件地址，长度是48位（6字节），由十六进制的数字组成，分为前24位和后24位。前24位称为组织唯一标志符（OUI），是由IEEE的注册管理机构分配给厂家的代码，用来区分不同的厂家；后24位由厂家自己分配的代码，成为拓展标识符。
在Linux系统下可以用`ifconfig`命令来查看MAC地址

### IP地址

IP地址是IP提供的一种统一的地址格式，它为互联网上的每一个网络和每一个主机分配一个逻辑地址，以此来屏蔽物理地址的差异。
IP地址分为IPv4和IPv6两类，由于现在主要使用IPv4地址，故下面只介绍IPv4地址
IPv4地址是一个32位的二进制数，通常分隔为四个八位二进制数（即四字节）。IPv4地址通常用点分十进制表示成`a.b.c.d`，其中a，b，c，d都是0-255内的十进制整数

### 子网掩码

子网掩码又称为网络掩码，地址掩码。计算机处理需要IP地址外，还需要知道多少位表示主机号及多少位表示子网号。这要求子网掩码不能单独存在，要结合IP地址一起使用，将某个IP地址划分为网络地址和主机地址两部分，子网掩码只针对IPv4地址。
在Linux系统下可以用`ifconfig`命令来查看子网掩码

### 端口

端口包括物理端口和逻辑端口。物理端口是用于连接物理设备的接口，逻辑端口是逻辑上用于区分服务端口。我们这里只讲逻辑端口。
Linux操作系统会给那些有需求的进程分配IP地址和端口，每一个端口由一个正整数标识。在互联网上，各个主机根据TCP/IP发送和接收数据包，数据包根据其目的主机IP地址选择路由器被传送到目的主机，当目的主机接收到数据包后，根据数据包中的目的端口号把数据发送到目的端口进行处理。
端口主要分为以下几类：

- 周知端口：众所周知的端口号，范围是从0到2023。例如，其中80端口号分配给www服务，21端口分配给FTP服务等等。
- 注册端口：分配给用户进程或应用程序，范围是从1024到49151.
- 动态端口：范围是从49152到65535.之所以被称为动态端口，是因为它不固定分配某种服务端口号，而是动态分配服务端口号。

---

## TCP

### 三次握手

![1](https://img-blog.csdnimg.cn/direct/191e3149d0f4436f9bc47cc1155fbe5e.png#pic_center)
![2](https://img-blog.csdnimg.cn/direct/041adcc06a464462a09eaa1e3a098779.jpeg#pic_center)

建立TCP连接的过程又称为三次握手。建立一个TCP连接时，需要发送端和接收端共发送三个数据报文以确立连接建立。

- 第一次握手，发送端将标志位`SYN`置为1，发送端进入`SYN_SENT`状态，随机产生一个值`seq=J`将该数据发送给接收端，等待接收端确认。
- 第二次握手，接收端进入`SYN_RCVD`状态，收到数据后由标志位`SYN=1`确认发送端请求建立连接，接收端将`SYN` 和`ACK`都置为1，`ack=J+1`，并随机产生一个值`seq=K`,并将该数据发送给发送端以确认收到请求连接。
- 第三次握手，发送端收到请求后，检查ack是否为J+1，ACK是否为1，如果正确，将`ACK`置为1，ack=K+1，并将数据发送给接收端，接收端检查ack是否是K+1，ACK是否为1，如果正确则连接建立成功，接收端和发送端进入`ESTABLISHED`状态，完成三次握手，随后开始传输数据。

### 四次挥手

- 第一次挥手，发送端发送一个`FIN`包，用来关闭到接收端的数据传送，发送端进入`FIN_WAIT_1`状态。
- 第二次挥手，接收端收到`FIN`包，发送`ACK`给发送端，接收端进入到`CLOSE_WAIT`状态。
- 第三次挥手，接收端发送一个`FIN`包，用来关闭到发送端的数据传送，接收端进入`LAST_ACK`状态。
- 第四次挥手，发送端收到`FIN`包后，进入`TIME_WAIT`状态，并发送一个`ACK`确认包给接收端，经过2MSL后，进入`CLOSE`状态，完成四次挥手。连接完全断开。

![3](https://img-blog.csdnimg.cn/direct/8ea194a2f06942ef9b32c57dc5809100.png#pic_center)
![4](https://img-blog.csdnimg.cn/direct/31150b9228204f689e5e1c57897a9156.png#pic_center)

> 这一篇全是理论，看起来应该有些枯燥，如果实在看不下去，可以先跳过，先看后套接字部分，等看了套接字之后再来看这可能会就会好一些。

---

## 写在最后

个人亲身经验：我们学习的一系列Linux命令，一定要自己**亲手去敲**。不要只是看别人敲代码，不要只是停留在眼睛看，脑袋以为自己懂了，等你实际上手去敲会发现许许多多的这样那样的问题。毕竟“**实践出真知**”。

---

> 如果你觉得我写的题解还不错的，请各位**王子公主**移步到我的其他题解看看
> 
> 1. **数据结构与算法部分（还在更新中）：**
>    - [***C++ STL总结 - 基于算法竞赛（强力推荐***）](https://blog.csdn.net/yourgrandfather_/article/details/135051716?spm=1001.2014.3001.5501)
>    - [动态规划——完全背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135111459)
>    - [动态规划——01背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135103012?spm=1001.2014.3001.5501)
>    - [动态规划——多重背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135125267)
>    - [动态规划——分组背包问题](https://blog.csdn.net/yourgrandfather_/article/details/135134277)
>    - [动态规划——最长上升子序列（LIS)](https://blog.csdn.net/yourgrandfather_/article/details/135150351)
>    - [二叉树的中序遍历（三种方法）](https://blog.csdn.net/yourgrandfather_/article/details/135167817)
>    - [最长回文子串](https://blog.csdn.net/yourgrandfather_/article/details/135183977)
>    - [最短路算法——Dijkstra（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134869064?spm=1001.2014.3001.5501)
>    - [最短路算法———Bellman_Ford算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/134935786?spm=1001.2014.3001.5501)
>    - [最短路算法———SPFA算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135004393?spm=1001.2014.3001.5501)
>    - [最小生成树算法———prim算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135026901?spm=1001.2014.3001.5501)
>    - [最小生成树算法———Kruskal算法（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135039904?spm=1001.2014.3001.5501)
>    - [染色法判断二分图（C++实现）](https://blog.csdn.net/yourgrandfather_/article/details/135094296?spm=1001.2014.3001.5501)
> 2. **Linux部分（还在更新中）：**
>    - [Linux学习之初识Linux](https://blog.csdn.net/yourgrandfather_/article/details/134953315?spm=1001.2014.3001.5501)
>    - [Linux学习之命令行基础操作](https://blog.csdn.net/yourgrandfather_/article/details/134956923?spm=1001.2014.3001.5501)
>    - [Linux学习之基础命令（适合小白）](https://blog.csdn.net/yourgrandfather_/article/details/135189166)
>    - [Linux学习之权限管理和用户管理](https://blog.csdn.net/yourgrandfather_/article/details/135222868)
>    - [Linux学习之制作静态库和动态库](https://blog.csdn.net/yourgrandfather_/article/details/135275850)
>    - [Linux学习之makefile](https://blog.csdn.net/yourgrandfather_/article/details/135288190)
>    - [Linux学习之系统编程1（关于读写系统函数）](https://blog.csdn.net/yourgrandfather_/article/details/135324720)
>    - [Linux学习之系统编程2（关于进程及其相关的函数）](https://blog.csdn.net/yourgrandfather_/article/details/135338115)
>    - [Linux学习之系统编程3（进程及wait函数）](https://blog.csdn.net/yourgrandfather_/article/details/135354402)
>    - [Linux学习之系统编程4(进程间通信）](https://blog.csdn.net/yourgrandfather_/article/details/135383973)
>    - [Linux学习之系统编程5(信号）](https://blog.csdn.net/yourgrandfather_/article/details/135408188)
>    - [Linux学习之系统编程6(线程）](https://blog.csdn.net/yourgrandfather_/article/details/135438595)
>    - [Linux学习之系统编程7(线程同步/互斥锁/信号量/条件变量）](https://blog.csdn.net/yourgrandfather_/article/details/135460669)

## ✨🎉总结

“种一颗树最好的是十年前,其次就是现在”
所以,
“让我们一起努力吧,去奔赴更高更远的山海”
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7bb6bf71a1814bd88fb9615df5cc258d.png#pic_center =400x250)
如果有错误❌,欢迎指正哟😋

🎉如果觉得收获满满,可以动动小手,点点赞👍,支持一下哟🎉
