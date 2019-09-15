## 说明

准备三台：

k8s-master01    192.168.0.245

k8s-node01       192.168.0.246

k8s-node02       192.168.0.247

**以下操作没有特别说明，都在三台服务器上执行**

## 设置系统主机名以及 Host 文件的相互解析

```shell
hostnamectl set-hostname k8s-master01
```

## 安装依赖包

```shell
sudo apt-get install conntrack ntpdate ntp ipvsadm ipset jq iptables curl sysstat wget vim net-tools git selinux-utils
```

## 设置防火墙为 Iptables 并设置空规则

在每台机器上关闭防火墙，清理防火墙规则，设置默认转发策略：

```shell
systemctl stop firewalld
systemctl disable firewalld
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat
iptables -P FORWARD ACCEPT
#FORWARD 默认是DROP 这个是不能被k8s转发调用的需要改成ACCEPT
#iptables -nL 查看转发策略
#iptables -F 清空规则
#service iptables save 保存默认规则
```

## 关闭 SELINUX

如果开启了 swap 分区，kubelet 会启动失败(可以通过将参数 --fail-swap-on 设置为 false 来忽略 swap on)，故需要在每台机器上关闭 swap 分区。同时注释 /etc/fstab 中相应的条目，防止开机自动挂载 swap 分区：

```shell
#swapoff -a
#swapon -a 这个是分区的开启
#sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
#这个命令是把/etc/fstab文件中swap这一行注释掉

swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0 && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
#关闭交换分区  如果不关 pod可能会放到虚拟内存中，大大影响效率。
```

## 调整内核参数，对于 K8S

```shell
cat > kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963 
fs.nr_open=52706963 
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF
cp kubernetes.conf /etc/sysctl.d/kubernetes.conf
sysctl -p /etc/sysctl.d/kubernetes.conf  #手动刷新一下可立即生效

#执行这些的时候会报错：sysctl: cannot stat /proc/sys/net/netfilter/nf_conntrack_max: 没有那个文件或目录
#等下面升级了内核版本就好了。4.0以后的版本就可以了

# kubernetes.conf 这个文件在/etc/sysctl.d/下 在开机的时候能被调用到
#上面必备的参数是  以下三个
# net.ipv6.conf.all.disable_ipv6=1 		#关闭IPv6的协议
# net.bridge.bridge-nf-call-iptables=1	#开启网桥模式
# net.bridge.bridge-nf-call-ip6tables=1 #开启网桥模式
```

## 调整系统时区

```shell
#查看时区
timedatectl|grep "Timezone"
#安装系统的时候选过  这里就不用调了。
# 设置系统时区为 中国/上海
timedatectl set-timezone Asia/Shanghai
# 将当前的 UTC 时间写入硬件时钟
timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
systemctl restart rsyslog
systemctl restart crond
```

## 关闭系统不需要服务

```shell
systemctl stop postfix && systemctl disable postfix
#邮件服务是不需要的，这里给关闭了。
```

## 设置 rsyslogd 和 systemd journald

```shell
#设置日志的保存方式 centos 7 默认用的rsyslogd 这里我们用systemd journald 

mkdir /var/log/journal # 持久化保存日志的目录
mkdir /etc/systemd/journald.conf.d #配制文件存放目录
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent
# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000
# 最大占用空间 10G
SystemMaxUse=10G
# 单日志文件最大 200M
SystemMaxFileSize=200M
# 日志保存时间 2 周
MaxRetentionSec=2week
# 不将日志转发到 syslog
ForwardToSyslog=no
EOF
systemctl restart systemd-journald
```

## kube-proxy开启ipvs的前置条件

```shell
mkdir -p /etc/sysconfig/modules
modprobe br_netfilter #加载netfilter模式
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
#上面这个配置文件加载一些相关依赖 这里的依赖是一些模块
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

## 安装 Docker 软件

```shell
apt install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
apt update -y && apt install -y docker-ce
## 创建 /etc/docker 目录
mkdir /etc/docker
# 配置 daemon.
cat > /etc/docker/daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
}
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
# 重启docker服务
systemctl daemon-reload && systemctl restart docker && systemctl enable docker

#这里设置用systemd 的 cgroup组
#把存储日志的方式改为 json file的方式
#这里配制好后 后期查日志方便  var/log/content 目录下找
```

## 安装 Kubeadm （主从配置）

```shell

curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y apt-transport-https
apt-get install -y kubeadm kubectl kubelet
systemctl enable kubelet.service
```

## 初始化主节点 （master）

**注意**：初始化的时候kubeadm会从谷歌拉取需要的镜像，我们可以把本地已有的镜像导入到docker中。

```shell
#导入镜像的过程 各个节点都需要导入
tar -zxvf kubeadm-basic.images.tar.gz
#有coredns.tar etcd.tar pause.tar apiserver.tar等
chmod +x test2.sh
#写个脚本导入到docker-----------
#!/bin/bash
#把kubeadm-basic.images中的镜像压缩文件名列表写入到 image-list.txt中
ls cd /root/kubeadm-basic.images > /tmp/image-list.txt
cd /root/kubeadm-basic.images
for i in $( cat /tmp/image-list.txt )
do
	docker load -i $i
done
#写个脚本导入到docker------------
```

```shell
kubeadm config images list

k8s.gcr.io/kube-apiserver:v1.15.3
k8s.gcr.io/kube-controller-manager:v1.15.3
k8s.gcr.io/kube-scheduler:v1.15.3
k8s.gcr.io/kube-proxy:v1.15.3
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
#想办法拉取这些镜像到docker
#我先拉取到docker hub
k8s.gcr.io/kube-apiserver:v1.15.3
k8s.gcr.io/kube-controller-manager:v1.15.3
k8s.gcr.io/kube-scheduler:v1.15.3
k8s.gcr.io/kube-proxy:v1.15.3
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
	
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName

docker pull bingju328/coredns1.3.1
docker pull bingju328/etcd3.3.10
docker pull bingju328/kube-proxyv1.15.3
docker pull bingju328/kube-schedulerv1.15.3
docker pull bingju328/kube-controller-managerv1.15.3
docker pull bingju328/kube-apiserverv1.15.3
docker pull bingju328/k8s-pause


docker tag bingju328/coredns1.3.1:latest k8s.gcr.io/coredns:1.3.1
docker tag bingju328/kube-proxyv1.15.3:latest k8s.gcr.io/kube-proxy:v1.15.3
docker tag bingju328/kube-schedulerv1.15.3:latest k8s.gcr.io/kube-scheduler:v1.15.3
docker tag bingju328/kube-controller-managerv1.15.3:latest k8s.gcr.io/kube-controller-manager:v1.15.3
docker tag bingju328/kube-apiserverv1.15.3:latest k8s.gcr.io/kube-apiserver:v1.15.3
docker tag bingju328/k8s-pause:latest k8s.gcr.io/pause:3.1
docker tag bingju328/etcd3.3.10:latest k8s.gcr.io/etcd:3.3.10

kubeadm config images pull
#显示我们init-defaults的初始化文件 并 导入到 kubeadm-config.yaml中
kubeadm config print init-defaults > kubeadm-config.yaml
#修改kubeadm-config.yaml  advertiseAddress   kubernetesVersion
# podSubnet 添加一下pod的网段  因为默认安装的flannel插件去实现覆盖网络，而flannel默认的网段就是 
#10.244.0.0/16  如果不一致后期还得修改
#添加------- 这段的含义是把默认的调度方式改为 ipvs方式
#apiVersion: kubeproxy.config.k8s.io/v1alpha1
#kind: KubeProxyConfiguration
#featureGates:
#  SupportIPVSProxyMode: true
#mode: ipvs
#-----------

localAPIEndpoint:
advertiseAddress: 192.168.0.245
kubernetesVersion: v1.15.1
networking:
podSubnet: "10.244.0.0/16"
serviceSubnet: 10.96.0.0/12
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
# 指定yaml安装 
#--experimental-upload-certs 自动颁发证书 1.13后才有这个命令
#把所有信息写入到 kubeadm-init.log中
kubeadm init --config=kubeadm-config.yaml --experimental-upload-certs | tee kubeadm-init.log
```

## 初始化后需要做的(master)

```shell
#.kube 这里会保存我们的连接配置 kubectl 和 kubeapi 交互，交互时采用的https协议，交互时候的一些缓存
#还有https的认证文件 都在.kube目录中
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 查看状态

```shell
kubectl get node
NotReady
#因为还没有构建flannel 所以现在显示NotReady状态
```

## 加入主节点以及其余工作节点（Node节点）

```shell
执行安装日志中的加入命令即可
#这个在初始化打印出来的日志里面  kubeadm-init.log
#kubeadm join 192.168.0.246 --token ........
#这个命令是在Node节点执行的
```

## 部署网络（在master节点执行就行）

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#或者先下载下来 通过kubectl create -f kube-flannel.yml来安装
```

## 移除节点

```shell
kubectl delete node hp
```

## 重新加入

使节点加入集群的命令格式是kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

如果我们忘记了Master节点的token，可以使用下面的命令来查看：

```shell
kubeadm token list
```

默认情况下，token的有效期是24小时，如果token已经过期的话，可以使用以下命令重新生成：

```shell
kubeadm token create
```

如果你找不到–discovery-token-ca-cert-hash的值，可以使用以下命令生成：

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

除了上面通过两次命令找token和hash，也可以直接一次性执行如下命令来获取：

```shell
kubeadm token create --print-join-command 
```

