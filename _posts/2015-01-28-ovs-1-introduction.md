---
layout: default
title: Open vSwitch系列1——介绍
comments: true
---

##写在前面的话
最近在学习Open vSwitch源码，因为这个项目在Linux kernel space和user space都有代码，所以读起来非常有挑战性。当然，痛苦总是伴随着收获，整个过程里我学到非常多的知识，也积累了一些经验，所以我想把自己的收获记录下来，与大家分享。

我打算写一个系列，详细地介绍Open vSwitch各个功能的使用和实现细节。今天这是本系列的开篇，先介绍一下什么是Open vSwitch，以及为什么需要Open vSwitch。

与前几年相比，云计算不再是一个只停留在炒作阶段的热门词汇，一大批云计算相关的产品涌入我们的生活，例如阿里云、七牛云等云计算平台。阿里云平台提供云服务器给用户使用，不同的用户会买多个云服务器，组织成一个封闭的网络环境。很显然，不同的用户之间的网络是彼此隔离的，Open vSwitch在隔离的过程中起到了很重要的作用。

##核心功能

Open vSwitch有两种工作模式，一种是“Normal”模式，另一种是“Flow”模式。在“Normal”模式中，它其实就是一个普通数据链路层的交换机，对于每一个收到的Ethernet包，都会记录下来包的MAC地址，并与相应端口绑定。如果之前已经记录了这个Ethernet包的目标MAC地址，Open vSwitch会将包发往某个特定的端口；如果目标MAC地址从未见过，Open vSwitch会将包从每个端口（存疑：是否包括接收这个包的端口？）广播出去。

在“Flow”模式中，Open vSwitch变身成为一个基于Flow的交换机。什么是Flow呢？简单点说，一条Flow是规则(Match)和动作(Action)的组合。如果收到一个包，它满足某个Flow的规则(Match)，Open vSwitch会在这个包上执行此Flow的动作(Action)。
     
那规则和动作分别指的是什么呢？规则是指L2-L4包头中的一些field，我们将Flow1的Match定义成下面这种样子：
```C
    Source IP: 192.168.2.1
    Source port: 4321
```
如果一个包，它来自IP为192.168.2.1的机器上的1234端口，那么就可以说这个包满足这个Flow的规则。

之后就可以在这个包上执行动作，有哪些动作呢？总的来说，有3类动作，第一类是转发包，也就是将包从某个端口发出去，第二类是扔掉包，第三类是对包的内容做修改。只有满足特定条件的包才能被转发（也可能被修改）出去。

如何设置Open vSwitch中的Flow呢？有2种方式，第一种是直接使用它提供的工具ovs-ofctl，执行add-flow命令插入一条Flow进去。第二种方式是使用OpenFlow接口插入Flow，OpenFlow是一套控制交换机的接口(不仅仅局限于Open vSwitch)，通过这套接口，外部可以动态地在交换机中增删改查Flow。基于OpenFlow，我们就可以使用一个软件控制器管理Open vSwitch的行为。这种由外部控制Open vSwitch的想法正是Software Defined Network的核心概念。如果整个网络的拓扑结构由一个集中控制器决定，能够做的事情就太多了，比如我们可以监控网络流量，然后动态地调整流量，以实现流量负载均衡。

##其他功能
以上就是Open vSwitch的最核心的功能，它还有一些其他的功能。官网上详细地列出了Open vSwitch的所有特点，如下所示。

![features](/images/ovs-1-features.png)

除了之前提到的核心功能，剩余的特点可以分成3类：
* 安全，比如Tunnel、VLAN、traffic filter
  * Tunnel这些功能非常重要，它能够帮助我们建立相互隔离的网络空间，这点我会在后面的文章中详细介绍。
* 性能功能，比如QoS控制
* 监控：比如NetFlow、sFlow等


     
     
