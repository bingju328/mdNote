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
yum install -y conntrack ntpdate ntp ipvsadm ipset jq iptables curl sysstat libseccomp wget vim
net-tools git
```

## 设置防火墙为 Iptables 并设置空规则

```shell
systemctl stop firewalld && systemctl disable firewalld
yum -y install iptables-services && systemctl start iptables && systemctl enable iptables
&& iptables -F && service iptables save
#iptables -F 清空规则
#service iptables save 保存默认规则
```

## 关闭 SELINUX

```shell
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

## 升级系统内核为 4.44

```shell
CentOS 7.x 系统自带的 3.10.x 内核存在一些 Bugs，导致运行的 Docker、Kubernetes 不稳定，例如： rpm -Uvh
http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

```shell
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
# 安装完成后检查 /boot/grub2/grub.cfg 中对应内核 menuentry 中是否包含 initrd16 配置，如果没有，再安装
一次！
yum --enablerepo=elrepo-kernel install -y kernel-lt
# 设置开机从新内核启动
grub2-set-default 'CentOS Linux (4.4.189-1.el7.elrepo.x86_64) 7 (Core)'
```



## kube-proxy开启ipvs的前置条件

```shell
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
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules &&
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

## 安装 Docker 软件

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum update -y && yum install -y docker-ce
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
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum -y install kubeadm-1.15.1 kubectl-1.15.1 kubelet-1.15.1
systemctl enable kubelet.service
```

## 初始化主节点 （master）

**注意**：初始化的时候kubeadm会从谷歌拉取需要的镜像，我们可以把本地已有的镜像导入到docker中。

```shell
#导入镜像的过程 各个节点都需要导入
tar -zxvf kubeadm-basic.images.tar.gz
#有coredns.tar etcd.tar pause.tar apiserver.tar等
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

## 初始化后需要做的

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
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-
flannel.yml
#或者先下载下来 通过kubectl create -f kube-flannel.yml来安装
```







