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

BorgMaster： Borg系统的大脑。

Borglet：Borg系统工作结点。

![kubernetes架构图](images/k8s架构图.png)

Master: scheduler、replication controller、api server

Node: kubelet、kube proxy

### 1、Scheduler 调度器

调度任务用，任务过来把任务通过api server分发到不同的node中，也是通过api server 把信息存到etcd中。

### 2、Replication controller控制器

维持副本期望数目，用来维护副本的数目，创建、删除Pod。

### 3、Api Server 

所有服务统一访问入口，Scheduler , RC , kubectl 都是通过和Api Server交互来完成相应的功能的。



### 4、ETCD

键值对数据库、储存k8s集群所有重要信息(持久化)

一个可信赖的分布式键值存储服务(key-v扩容方便)，它能够为整个分布式集群存储一些关键数据，协助分布式集群的正常运转。

推荐在Kubernetes集群中使用Etcd v3 , v2版本已在Kubernetes v1.11中弃用。

![ETCD架构图](images/ETCD架构图.png)

ETCD采用HTTP Server的形式，即采用Http协议进行**CAS**结构 (Central Authentication Service的缩写，中央认证服务，一种独立开放指令协议) 构建的服务。**kubernetes也是采用这种构建。**

Raft : etcd所有读写信都会存在这里。

WAL：预写式日志（Write-Ahead Logging）。作用防止Raft信息丢失，在raft变更的时候会在wal中存一下，并且会定时的对日志完整的备份一下到Snapshot。

Store：V3会实时的把日志和Raft中的信息存到本地磁盘中去。

### 5、Kubelet

直接跟容器引擎交互实现容器的生命周期管理。

会和CRI(Container Runtime Interface 容器，运行环境，接口) 进行交互，操作docker创建对应的容器，会维持Pod的生命周期。

### 6、Kube proxy

负责写入规则至IPTABLES、IPVS实现服务映射访问

通过Kube proxy来完成Pod之间的负载，kube proxy操作firewal来对Pod进行映射。新版本还支持IPVS

### 7、Container

容器

### 8、其它插件

- CoreDNS : 可以为集群中的SVC创建一个域名IP的对应关系解析
- Dashboard: 给k8s集群提供一个 B/S结构访问体系
- Ingress Controller: 官方只能实现四层代理，Ingress可以实现七层代理。
- Federation : 提供一个可以跨集群中心多k8s统一管理功能。
- Prometheus (普罗米修斯):  提供一个k8s集群的监控能力。
- ELK：提供k8s集群日志统一分析接入平台。

> 服务分类：
>
> ​		有状态服务：DBMS （删除后系统不能用）
>
> ​	    无状态服务：LVS  APACHE （删除不影响系统功能）
>
> 高可用集群副本数最好是 >=3的奇数个  Leader好选举。



