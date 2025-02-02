---
layout:    post   				    # 使用的布局（不需要改）
title:     「Computer Network-3」Data Link Layer & LANs # 标题 
subtitle:  The Bridge between Physical realization and Logic design #副标
date:      2021-01-11 				# 时间
author:    Culaccino					# 作者
header-img: img/upd_img10.png       #这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 专业课
    - Computer Networking


---

数据链路层在物理层提供服务的基础上向网络层提供服务，其最基本的服务是将源自网络层来的数据可靠地传输到相邻节点的目标网络层。其主要作用是**加强物理层传输原始比特流的功能**，即将物理层提供的可能出错的物理连接改造成为"逻辑上无差错的数据链路"，使之对网络层表现为一条无差错的链路。

数据链路层上的PDU被称为**帧**，该篇主要讲述数据链路层如何通过帧实现该层的功能。

## 1. 功能

#### 为网络层提供服务

①无确认无连接 ②有确认无连接 ③有确认有连接

#### 链路管理

连接的建立、维持、释放

#### 流量控制

控制因为较高的发送速度和较低的接受能力不匹配而造成的传输错误。区别于传输层端到端（主机间）且会给发送端一个窗口公告，数据链路层的控制是相邻节点间的，且当接收不下时就不回复确认。

- 停止-等待协议：每发送完一帧就要等待接收方的应答信号才能发送下一帧（发送1-确认1-发送2-确认2）。

  <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjuwf7rwsj316o0gutom.jpg" style="zoom:80%;" />

- 滑动窗口协议

  **滑动窗口①通过限定窗口大小控制了流量，②在没收到确定信息时(设定确认时限)让发送方自动重传，保证了可靠传输**

  - 后退N帧协议（GBN）：发送方一次可以发送N帧，按序接收；重传从最后一个确认开始。

  - 选择重传协议（SR）：发送方一次可以发送N帧，可不按序接收；重传没有确认的帧。

    | 可靠数据传输协议 | 发送窗口值Wt                | 接收窗口值Wr               |
    | ---------------- | --------------------------- | -------------------------- |
    | 停-等协议        | Wt=1                        | Wr=1                       |
    | 回退N协议        | 1<=Wt<=2^K-1                | 1                          |
    | 选择重传协议     | 1<=Wt<=min(2^(K-1)，2^K-Wr) | 1<=Wr<=min(2^(K-1),2^K-Wt) |

#### 差错控制

差错控制是为了让错误可以尽早被发现，不会让一个错误的数据包发送了很长时间到达最终目的地之后才被发现，从而导致网络资源的浪费。

差错主要由**比特错**和**帧错**两种，在链路层中主要控制的是比特错，可以采用检错编码和纠错编码两种，其中检错码仅能知道帧是否出错，而纠错编码能够知道错误的具体位置；帧错的控制在传输层会详细介绍。

- **检错编码**

  - **奇偶校验码**

    最终信息n位：(n-1)位有效数据+1位校验元。方式有奇校验、偶校验两种，其中奇校验使得加上校验元后一共有奇数个1，偶校验使得共有偶数个1；根据发送-接收双方的1的个数是否都是奇数/偶数来判断是否发生了错误。

    但奇偶校验只能检查出有奇数个位错的情况，比如当一个1变成0、一个0变成1，或两个1变成0这种偶数个位错的情况时，1的总个数的奇偶性不会发生改变，此时无法检查出。

    <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjsftlxdbj30ya084dm8.jpg" style="zoom:50%;" />

  - **循环冗余码CRC**

    会给出生成多项式，要传的数据/生成多项式=商……余数（FCS帧检验序列/冗余码），在接收方，节点将得到的（得到的数据+余数）/生成多项式，若余数为0，则表示正确，反之则判定为出错，丢弃。

    <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjt5c8gv2j315o0d24e8.jpg" style="zoom:80%;" />

- **纠错编码-海明码**

  ​	**[Reference](https://blog.csdn.net/lycb_gz/article/details/8214961?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-2&spm=1001.2101.3001.4242)**

#### 透明传输

链路层”看不到“传输数据的具体内容，任何比特组合的传输数据都能在链路上传送——所以当数据中的比特组合恰巧与某一控制信息完全相同时，为了不被接收方认为是控制信息，必须采取适当的措施，以保证数据链路层的传输是透明的（见组帧部分）。

## 2. 组帧

**帧**的组成：(帧首部，帧的数据部分（IP数据报），帧尾部)。其中首部包含帧开始符，尾部包含帧结束符，这两个**帧界定符**可用于**帧定界**；**帧同步**指接收方能从接收到的二进制比特流中区分出帧的起始和终止。

#### 字符计数法

在帧首部的第一个字段（字节）标明该帧的字符（字节）数。

缺点：帧定界全靠帧的第一个字段，可靠性不足。当第一个字段中的内容出现错误时容易形成多米诺骨牌效应，很难再进行同步调整，目前使用较少

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjqx3635nj3108088tcw.jpg" style="zoom:67%;" />

#### 首位界定法（填充法）

字符填充和比特填充对通信双方计算机网络层是完全透明的，即添加和删除操作都在链路层中完成，网络层感觉不到、也不需要在意具体的技术实现方法，任意数据内容都能被发送。若接收方失去同步，只需要重新寻找位流中的标志序列即可重新获得同步。

1. **字符填充法**

   用ASCII字符（8位）作为真的边界。在帧的首位分别添加ASCII字符序列DLE STX和DLE ETX，目的节点可以通过寻找对应的STX和ETX进行帧定界。

   但当原始数据中存在和STX或ETX的比特组合相同的字节时，可能会导致读取的过早结束/错误情况。字符填充法的解决方式是，当发现数据中有DLE字符（STX、ETX、转义字符ESC）时，在这个DLE字符前在插入一个转义字符ESC，当接收方读到ESC时表示继续读即可，直到读到没有跟着ESC的ETX，表示该帧的读取结束。

   <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjr9f27zaj31700l67hc.jpg" style="zoom:80%;" />

2. **比特填充法**

   用”01111110“作为帧首部和帧尾部，对中间的数据内容，若发现存在”11111“则在其后面添加一个“0”，接收时删除即可。

   <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjrickag1j31660lqqmh.jpg" style="zoom:80%;" />

#### 违规编码法

在数据信息转化为数字信号后，可以将所采用的的编码方法中的“违规编码”来作为帧的起始和终止。以曼彻斯特编码为例，其“1”信号在一个周期内表现为高-低，而“0”信号表现为低-高，故我们可以用“高-高”或“低-低”来表示帧的起始和终止。



## 3. 介质访问-多路访问协议

### 信道划分（静态）

- 频分复用：将多路信号调制到不同频率载波上叠加形成一个符合信号。
- 时分复用：将物理信道按时间分为若干时间片，轮流给不同信号使用。
- 波分复用：在一根光纤中传输多种不同波长（频率）的光信号。
- 码分复用：靠不同的编码来区分各路原始信号，例如CDMA技术

### 随机访问（动态）

随机访问协议适用于**广播信道**，节点以随机的方式发送数据，当多个结点同时发送数据时，每个节点总是想立即占用整个信道，**存在冲突和竞争发送**。

- ALOHA

  - 纯ALOHA：不检测直接发送，无确认则等待重发
  - 时隙ALOHA：将时间划分为若干等长时隙，按时发送

- CSMA

  - 1-坚持：闲则发送，忙则继续监听
  - 非坚持：闲则发送，忙则等待一个随机时间再听
  - p-坚持：闲则以概率p发送，1-p等待下一个时隙；忙则等待一个随机事件再听

- CSMA/CD

  - 流程：先听后发，边听边发(信号在最远两个端点间往返传输的全程一直监听)，冲突停发，随机重发
  - 碰撞解决：采用二进制指数退避算法来解决碰撞问题

  二进制指数退避算法以冲突窗口大小为基准，退避时间和该节点的冲突次数呈指数关系，当打到限定的冲突次数，该节点就停止发送数据。该算法考虑了网络负载对冲突的影响。

  <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjyd9tetdj314o07a7e9.jpg" style="zoom:80%;" />

- CSMA/CA

  - 避免碰撞：预约信道、ACK帧、RTS/CTS帧
  - 碰撞解决：采用二进制指数退避算法来解决碰撞问题

### 轮询访问（动态）

令牌传递协议：只有得到令牌的机器才能发送数据，其他必须等待



## 4. 局域网

#### 以太网

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjzkhncyuj315u0jcwsk.jpg" style="zoom:80%;" />

**10BASE-T**以太网：

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjzp41gjtj316i0b6wq1.jpg" style="zoom:80%;" />

## 广域网

#### PPP协议&HDLC协议

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmjztf4be1j30y80pq7k2.jpg" style="zoom:70%;" />



## 5. 设备

#### 网桥

网桥可以互连不同物理层、不同mac子层和不同速率的以太网。其会根据MAC帧的目的地址对帧进行转发和过滤。当网桥收到一个帧时，并不向所有接口转发此帧，而是先检查此帧的目的MAC地址，然后再确定将该帧转发到哪一个接口，或是将其丢弃（过滤）掉。

**两种网桥**

1. 透明网桥：按照自学习算法（根据历史记录）填写转发表，然后按照转发表转发帧
2. 源路由网桥：通过广播的方式向目标地址发送广播，目标地址收到后将每条路径都发一个响应帧给网桥，经过对比后即可得到最佳路径。发送方再将最佳路径放到帧首部即可。

**网桥的优点**

- 过滤通信量，增大了吞吐量
- 扩大了物理范围
- 提高了可靠性

#### 交换机

多端口的网桥进化为交换机。交换机有两种交换方式：

1. 直通式：帧再接收后只检查目的地址，几乎马上转发。延迟小，可靠性低，无法支持具有不同速率的端口的交换。
2. 存储转发式：将帧放入Cache中并检查正确性，错误则丢弃。延迟大，可靠性高，可以支持具有不同速率的端口的交换。



冲突域：连接在同一导线上的所有工作站的集合（同一物理网段上所有节点的集合），或以太网上竞争同一带宽的结点集合。

广播域：接收同样广播消息的节点的集合。

|                              | 能否隔离冲突域 | 能否隔离广播域 |
| ---------------------------- | -------------- | -------------- |
| 物理层设备（中继器、集线器） | x              | x              |
| 链路层设备（网桥、交换机）   | √              | x              |
| 网络层设备（路由器）         | √              | √              |

