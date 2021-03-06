# Kubernetes + Opensds 学习笔记


## 什么是kubernetes

  首先是kubernetes ，Kubernetes是一个全新的基于容器技术的分布式架构方案，又称k8s，是google整理了15年来borg的运行经验构建的新的开
源产品。主要用于管理云上的多个主机上的容器化应用，简单来说就是管理你买的容器（这个容器我理解为装载运行可执行应用程序的环境）。并完成基于容器的应用程序的调度，均匀地分配负载于集群中的许多机器中，针对负载高峰的应用程序扩展，容器的创建管理删除等等，这些任务。


## Opensds是啥

[Opensds](https://github.com/opensds) 是一个开源项目。
k8是管理容器的，容器的存储资源需要有外部提供，由各个不同厂商来提供。早期各商家的驱动代码需要嵌入k8中，这带来了一系列问题：
* 不同驱动接口不同，维护难度大
* 驱动代码需要和k8版本一起发布
* 驱动代码将由k8社区维护
* 代码需嵌入k8源码中，带来安全隐患

于是k8社区制定了一系列标准CSI(container storage interface)，要求存储设备满足这个接口标准。可是硬件上的东西不是说改就改啊，于是人们就只能
退而求次，新建opensds项目，向下统一不同设备的驱动差别，向上提供唯一一套CSI标准接口给k8，就相当于一个虚拟文件系统一样。

## 为什么我要用k8s

   使用k8s的理由很多，最根本的一个理由就是:IT行业从来都是一个由新技术驱动的行业。
   使用k8s带来的好处：
* 我们可以“轻装上阵”地开发复杂的系统了，不需要维护容器，不需要手动备份，不需要考虑调度等
* 系统可以随时搬迁到公有云上
* k8s提供了超强的扩容能力

### a.为什么我要用容器（container）

对于我的一个想要运行的应用程序，可能它不需要完整的操作系统平台（linux、windows等），也不需要很多的资源（一块4核cpu可以运行上千个这样的程序）。
所以划分单个虚拟机来给我这个程序实在是太浪费了。容器在这样的条件下出现了，它只配置了运行需要的应用程序最基础的也是必须的环境和计算资源，一台家用pc的配置就可以搭建几十个容器。并且，我容器之间是完全隔离的，彼此都不知道对方的存在，有着自己的文件系统。而且容器中的程序出了问题，也能很好地检测恢复。 

### b.容器（container）的运行环境在哪？

容器有效的时候，是位于pod内的，一个pod可以有多个容器，容器上加一层Docker环境以实现各种功能，然后再把docker放在pod内。

### c.Kubernetes都干了啥

k8s主要提供
* 自动化管理云平台中的主机资源
* 多个资源中心之间的任务分配，资源利用
* 分布式系统支撑，提供更高的可靠性，自恢复性，可扩展性和在线扩容能力
* 用户直接使用k8s来管理容器资源，k8s提供稳定的运行环境给用户。
### d.Kubernetes的主要资源

#### Pod
   Pod是k8s的最基本操作单元，它可容纳许多容器，相当于豌豆荚的概念。
   
   为什么k8要在容器上再加一层Pod？ 一个很重要的原因是，Docker容器之间的通信收到Docker网络机制的限制。在Docker的世界里，一个容器需要link方式才能访问另一个容器提供的服务，大量容器之间的link将会是一个非常繁琐的工作。通过Pod的概念将多个容器组合在一个虚拟的主机内，容器之间仅需要通过localhost就能相互通信了。
   
   一个Pod钟的容器共享共享同一组资源，如：
   * PID命名空间，Pod中不同应用程序可以看到其他程序的PID
   * 网络命名空间，容器能够访问同一个IP和端口范围
   * 存储卷，各个容器共用一个Pod的存储卷
   
####  Master / Node 节点

Pod是挂载在节点上的，一个节点可以挂载多个pod，节点分为master节点和node节点，master是集群控制节点(具体参照集群的概念)，负责整个集群的控制管理。除了master外就是node节点，node是k8s的工作节点，可以是物理机也可以是虚拟机。当一个node挂掉后，k8s会处理其上的工作，分配给另外的node之类的。
Pod是一群容器的运行环境，可以看做是这群容器的逻辑主机。  POD可能包含一个或多个紧密相连的应用，这些应用可能是在同一物理主机或虚拟机上。
同一个Pod中的应用可以共享磁盘，通过localhost通信，Pod是用后即毁的，直到被终止或者删除。当一个Node死掉后，上面的Pod也会被删除。

####  Volume

一个容器总得有地方放可执行文件，volume 是pod中能够被多个容器共同访问的目录，当容器出了问题时，volume中的数据也不会丢失。但当pod出了问题时，volume中的数据便也跟着没了。

####  Service

在k8s中，虽然每个Pod都有一个IP，但这个IP会随着Pod的销毁而消失，Service可以看作一组提供相同服务的Pod的对外访问接口，Service作用于哪些Pod是通过Label Selector决定的。

####  Labels
Label以key/value键值对的形式附加到各种对象上（Pod，service，Node等），Label定义了这些对象的可识别属性，用来对它们进行管理和选择。

## 5.	Kubernetes + CSI + opensds 宏观联系
