## 一、K8S中的资源

K8s  中所有的内容都抽象为资源， 资源实例化之后，叫做对象

### 名称空间级别

1、工作负载型资源 ( workload ) ： Pod 、 ReplicaSet 、 Deployment 、 StatefulSet 、 DaemonSet 、 Job 、
CronJob  (  ReplicationController 在 v1.11  版本被废弃 ) 

2、服务发现及负载均衡型资源 (  ServiceDiscovery LoadBalance  ): Service 、 Ingress 、

3、配置与存储型资源： Volume(  存储卷 ) )、 CSI(  容器存储接口, , 可以扩展各种各样的第三方存储卷 ) 

4、特殊类型的存储卷： ConfigMap (  当配置中心来使用的资源类型 )  、 Secret( 保存敏感数据) ) 、
DownwardAPI(把外部环境中的信息输出给容器) 

5、集群级资源： Namespace 、 Node 、 Role 、 ClusterRole 、 RoleBinding 、ClusterRoleBinding

6、元数据型资源： HPA 、 PodTemplate 、LimitRange

## 二、资源清单

## 三、常用字段解释说明

## 四、容器生命周期

## 