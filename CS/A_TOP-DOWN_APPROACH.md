# 第三章 运输层

#### 运输层的作用？

将网络层的在**两个端系统**之间的交付服务扩展到运行在两个**不同端系统上的应用层进程**之间的交付服务（逻辑通信）

## 3.1 概述和运输层服务

注意：

网络路由器仅作用于该数据报的网络层字段；即它们不检查封装在该数据报的运输层报文段的字段。

在接收端，网络层从数据报中提取运输层报文段，并将该报文段向上交给运输层。运输层则处理接收到的报文段，使该报文段中的数据为接收应用进程使用。

### 3.1.1 运输层和网络层的关系

#### 举例说明运输层和网络层的关系？（东西海岸）

直接看书吧，例子非常的生动。

### 3.1.2 因特网运输层概述

#### udp和tcp的区别？

##### udp：

- 为应用程序提供了一种**不可靠、无连接**的服务；
- **仅有数据交付和差错检查**（运输层最低限度的服务）；
- 一个udp套接字由一个**二元组标识**（目的ip地址+目的端口号），如果两个udp报文段有**不同的源ip地址和/或源端口号**，但是具有**相同的目的ip地址和目的端口号**，那么这两个报文段将**被定向到相同的套接字。**

##### tcp：

- 为应用程序提供了一种**可靠、面向连接**的服务；
- 除了数据交付和差错检查，还提供了**可靠数据传输、拥塞控制**。
- 一个tcp套接字由**四元组标识**（源ip地址+源端口号+目的ip地址+目的端口号），如果两个tcp报文段有不同的源ip地址和/或源端口号，但是具有相同的目的ip地址和目的端口号，那么这两个报文段将**被定向到不同的套接字**

##### ps：

- udp的无连接：使用udp时，在发送报文段之前，发送方和接收方的运输层实体之间没有握手

#### 传输层将数据交付给了谁？

- 一个进程(作为网络应用的一部分)有一个或多个套接字(socket),它相当于从网络向进程传递数据和从进程向网络传递数据的门户。
- 运输层实际上并没有直接将数据交付给进程，而是将数据交给了一个中间的**套接字**

#### 多路复用和多路分解？

##### 多路分解：

- 在接收端，运输层检查报文段中的对应字段（标识出套接字），从而将**报文段定向到对应的套接字。**
- 字段是源端口号字段+目的端口号字段

##### 多路复用：

**源主机从不同套接字中收集数据块**，并为每个数据块封装上首部信息，从而生成报文段。

## 3.3 无连接运输udp

#### 为什么有些应用选择在udp上构建？

- **关于发送什么数据以及何时发送的应用层控制更为精细**。**udp只需要将数据打包就可以马上传递给网络层**；tcp有拥塞控制机制且tcp在收到目的主机确认前会重新发送数据报文段。实时应用通常要求最小的发送速率，且可以容忍一些数据丢失，所以此时udp更加合适。
- **无须建立连接**。tcp进行数据传输前需要三次握手；**而udp不需要准备就可以开始传输，所以udp没有建立连接的时延**。这可能是dns运行在udp之上的原因。
- **无连接状态**。tcp需要在端系统中维护连接状态（接受和发送缓存、拥塞控制参数、序号和确认号的参数）；**udp无须维护连接状态。**所以当服务器运行在udp之上时，一般可以支持更多的活跃客户。
- **分组首部开销小**。每个TCP报文段都有20字节的首部开销，而**UDP仅有8字节的开销。**

#### udp的坏处？

udp没有拥塞控制，这可能会导致路由器中有大量的分组溢出，以至于丢包率很高，甚至会导致tcp发送方减小它们的速率。

### 3.3.1 UDP报文段结构

#### udp报文段的结构？

![image-20240509110837658](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240509110837658.png)

- udp首部有四个字段，每个字段两个字节
- 端口号：用于分解功能
- 长度：表示udp报文段中的字节数（首部+数据）
- 校验和：检查该报文段中是否出现了差错

### 3.3.2 UDP校验和

#### 为什么udp需要校验和？

因为不是所有链路层协议都提供了差错检查，而且就算报文段通过链路正确传输，当报文存储在路由器内存的时候，也可能出现差错，如果端到端数据传输服务要提供差错检测，UDP就必须在端到端基础上的运输层提供差错检测。这是端到端原则。

端到端原则：某些功能必须基于端到端实现---------与在较高级别提供这些功能的代价相比，在较低级别上设置的功能可能是冗余的或几乎没有价值的

#### udp提供了差错检查为啥不是可靠传输？

- 虽然UDP提供差错检测，但它对差错恢复无能为力。
- udp的某种实现只是丢弃受损的报文段；其他实现是将受损的报文段交给应用程序并给出警告。

## 3.4 可靠数据传输原理

### 3.4.1 构建可靠数据传输协议

