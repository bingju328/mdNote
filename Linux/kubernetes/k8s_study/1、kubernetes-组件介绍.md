## 一、资源管理器介绍

### 1、Apache MESOS

这是个比较古老的开源的**分布式资源管理器**，并不是为容器化而诞生的，只是作为一个基础的资源管理平台来用的。被称为分布式资源框架的内核。

### 2、Docker  SWARM

是一个资源管理器，是docker的集群化管理方案，新版的docker里面已经集成了swarm，是docker内部的一个组件了。相对于kubernetes来说功能比较少，比如要实现**滚动更新**在swarm中非常复杂 (需要手动定义流程)。

### 3、kubernetes

它是基于Google公司的Borg（在google已经服务十多年，稳定）资源管理器，并用go语言开发出来的，并且开源给容器基金会。

特点：

- 轻量级：go语言解释性语言，在语言级别支持进程管理（C需要人为控制），对系统资源的消耗比较少。
- 开源
- 弹性伸缩：Worker node 平缓增加，平缓减少。
- 负载均衡：自己在内部实现了模块上的负载均衡，不需要额外的搭建。采用的是**IPVS** (IPVS基本上是一种高效的Layer-4交换机，它提供[负载平衡](https://baike.baidu.com/item/负载平衡)的功能，LVS的IP负载平衡技术就是通过IPVS模块来实现的- 单机10万qps) 

## 二、知识图谱

[知识图谱点击这里Xmind](files/Kubernetes 结构 .xmind)

## 三、组件说明

![Borg架构图](images/borg架构图.png)

![kubernetes架构图](images/k8s架构图.png)

