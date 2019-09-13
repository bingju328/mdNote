## 一、Pod概念

- 自主式Pod：不是被控制器管理的Pod

  自主式Pod死亡后不会重启，RC也不会自动创建Pod来维护副本数

- 控制器管理的Pod：被控制器管理的Pod

  死亡后会重启，RC会自动创建Pod来维护副本数

### 控制器

**1、ReplicationController (RC)**

ReplicationController  用来确保容器应用的副本数始终保持在用户定义的副本数，即如果
有容器异常退出，会自动创建新的 Pod  来替代；而如果异常多出来的容器也会自动回收。
在新版本的 Kubernetes  中建议使用 ReplicaSet  来取代 ReplicationController

**2、ReplicaSet**

ReplicaSet  跟 ReplicationController  没有本质的不同，只是名字不一样，并且
ReplicaSet  支持集合式的 selector

虽然 ReplicaSet  可以独立使用，但一般还是建议使用 Deployment  来自动管理
ReplicaSet  ，这样就无需担心跟其他机制的不兼容问题（比如 ReplicaSet  不支持
rolling-update  但 Deployment  支持）

> 集合式的Selector: 创建Pod的时候会给Pod打标签(Label) , 比如：Pod1 label：v1,apache  Pod2 label: v2,apache  , 处理Pod时可以直接跟椐某个标签 集合式的处理。RC不支持这种集合方式。

**3、Deployment** 

Deployment并不负责Pod的创建。如图：

![](images/deploymentRS.png)

比如：有Pod 版本 v1,v1,v1 三个实例。滚动更新v2时：deployment先创建一个RS-1,由RS-1来创建v2Pod , 创建一个v2, 停止一个v1,直到更新完。回滚同理。

Deployment  为 Pod  和 ReplicaSet  提供了一个 声明式定义 (declarative)  方法，用来替
代以前的 ReplicationController  来方便的管理应用。典型的应用场景包括：

- 定义 Deployment  来创建 Pod  和 ReplicaSet
- 滚动升级和回滚应用
- 扩容和缩容
- 暂停和继续 Deployment

**4、HPA （ HorizontalPodAutoScale ）**

![](images/HPA.png)

HPA 基于RS 创建出来的，用来平滑的扩展Pod .  比如：HAP中定义：当CPU利用率达到80 以上时，扩容Pod , 最大10 ， 最小2 。当CPU利用率低于80就会自动缩容  最小到2个。 （目前这个功能是bate版本）

Horizontal Pod Autoscaling  仅适用于 Deployment  和 ReplicaSet  ，在 V1  版本中仅支持根据 Pod
的 CPU  利用率扩所容，在 v1alpha  版本中，支持根据内存和用户自定义的 metric 扩缩容.

**5、StatefullSet**

StatefulSet 是为了解决有状态服务的问题（对应 Deployments  和 ReplicaSets 是为无状态服务而设
计），其应用场景包括：
* 稳定的持久化存储，即 Pod  重新调度后还是能访问到相同的持久化数据，基于 PVC  来实现

* 稳定的网络标志，即 Pod  重新调度后其 PodName 和 HostName 不变，基于 Headless Service
  （即没有 Cluster IP  的 Service  ）来实现

* 有序部署，有序扩展，即 Pod  是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进
  行（即从 0  到 N-1 ，在下一个 Pod  运行之前所有之前的 Pod  必须都是 Running  和 Ready  状态），
  基于 init  containers  来实现

* 有序收缩，有序删除（即从 N-1  到 0 0 ）

  > 稳定性是指：Pod升级，重新创建，还用的原来的持久化数据，Name等。
  >
  > 有序性是指：Pod1 依赖Pod2，Pod2依赖Pod3.
  >
  > ​			启动的时候：Pod3->Pod2->Pod1
  >
  > ​			停止的时候：Pod1->Pod2->Pod3

**6、DaemonSet**

DaemonSet  确保全部（或者一些） Node  上运行一个 Pod  的副本。当有 Node  加入集群时，也会为他们
新增一个 Pod  。当有 Node  从集群移除时，这些 Pod  也会被回收。删除 DaemonSet  将会删除它创建
的所有 Pod
使用 DaemonSet  的一些典型用法：

* 运行集群存储 daemon ，例如在每个 Node  上运行 glusterd 、 ceph 。
* 在每个 Node  上运行日志收集 daemon ，例如 fluentd 、 logstash 。
* 在每个 Node  上运行监控 daemon ，例如 Prometheus Node Exporter

**7、Job ，Cronjob**

Job  负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个 Pod  成功结束
Cron Job    管理基于时间的    Job ，即：
* 在给定时间点只运行一次

* 周期性地在给定时间点运行

  > Job处理任务：比如 把Mysql备份的脚本封装成一个Pod,Job可以按一定规则来执行。
  >
  > Cron Job ： 可以定时的执行这个Pod。

**8、服务发现**

![](images/服务发现.png)

Client 访问一组相关的Pod（通过同一个RS创建的或有同一组标签的） , 这些Pod通过Service统一代理。即Service是同过标签来收集相关的Pod的，Service会有自己的IP  Port，Client 访问Service的IP Port , 然后负载到Pod上。

## 二、网络通讯方式

Kubernetes  的网络模型假定了所有 Pod  都在一个可以直接连通的扁平的网络空间中，这在GCE （ Google Compute Engine ）里面是现成的网络模型， Kubernetes  假定这个网络已经存在。而在私有云里搭建 Kubernetes  集群，就不能假定这个网络已经存在了。我们需要自己实现这个网络假设，将不同节点上的 Docker  容器之间的互相访问先打通，然后运行 Kubernetes。

- 同一个 Pod  内的多个容器之间： Io (即本地访问)
- 各 Pod  之间的通讯： Overlay Network (全覆盖网络)
- Pod  与 Service  之间的通讯：各节点的 Iptables 、（新版本中LVS）

### Flannel 

Flannel  是 CoreOS  团队针对 Kubernetes  设计的一个网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的 Docker  容器都具有全集群唯一的虚拟 IP 地址。而且它还能在这些 IP  地址之间建立一个覆盖网络（ Overlay Network ），通过这个覆盖网络，将数据包原封不动地传递到目标容器内。

![](images/Flannel网络.png)

 每个Node上启动一个Flanneld服务的守护进程，这个进程监听一个端口，这个端口就是和其它Node通信用的端口。

这个进程开启一个网桥Flannel0  用来和Docker0网桥交互。

Docker0会分配IP到Docker0下面的Pod上。

- 如果是同一台Node下的两个Pod，网络走的是Docker0网桥转发。

- 不同Node的两个Pod：比如 Pod1(10.1.15.2/24) -> Pod2(10.1.20.2/24) 

  Pod1先到Docker0,Docker0发现是不同网段的，触发Flannel0, Flanneld进程去etcd寻找具体的地址，Flanneld对报文封装，转发到Pod2中Node的Flanneld中，到Flannel0 -> Docker0 -> Pod2

**ETCD  之  Flannel  提供说明：**

>  存储管理 Flannel  可分配的 IP  地址段资源
>         监控 ETCD  中每个  Pod  的实际地址，并在内存中建立维护  Pod 

同一个 Pod  内部通讯：同一个 Pod  共享同一个网络命名空间，共享同一个 Linux  协议栈Pod1  至 Pod2

Pod1  与 Pod2  不在同一台主机， Pod 的地址是与 docker0 在同一个网段的，但 docker0 网段与宿主机网卡是两个完全不同的 IP 网段，并且不同 Node 之间的通信只能通过宿主机的物理网卡进行。将 Pod 的 IP 和所在 Node 的 IP 关联起来，通过这个关联让 Pod 可以互相访问

Pod1  与 Pod2  在同一台机器，由 Docker0  网桥直接转发请求至 Pod2 ，不需要经过 Flannel 演示
Pod  至  Service 的网络：目前基于性能考虑，全部为 iptables (新版LVS)  维护和转发

Pod  到外网： Pod  向外网发送请求，查找路由表 ,  转发数据包到宿主机的网卡，宿主网卡完成路由选择后， iptables 执行 Masquerade ，把源 IP  更改为宿主网卡的 IP ，然后向外网服务器发送请求

外网访问 Pod ：借助于Service的NodePort端口进行映射。



![](images/k8s网络.png)

> kubernetes中有三层网络，Service网络、节点网络、Pod网络 。Service和节点网络是虚拟网络，节点网络是真实的。

