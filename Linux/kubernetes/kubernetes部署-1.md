安装集群前需要先安装Docker，CRI-O，Containerd

### Cgroup驱动程序

当systemd被选为Linux发行版的init系统时，init进程会生成并使用根控制组（`cgroup`）并充当cgroup管理器。Systemd与cgroup紧密集成，并将为每个进程分配cgroup。可以配置容器运行时和要使用的kubelet `cgroupfs`。`cgroupfs`与systemd一起使用意味着将有两个不同的cgroup管理器。

控制组用于约束分配给进程的资源。单个cgroup管理器将简化正在分配的资源的视图，并且默认情况下将具有更可靠的可用和使用资源视图。当我们有两个经理时，我们最终会得到两个这些资源的观点。我们已经看到了现场的情况，其中配置`cgroupfs`用于kubelet和Docker `systemd` 的节点以及在节点上运行的其余进程在资源压力下变得不稳定。

更改设置，使容器运行时和kubelet `systemd`用作cgroup驱动程序，从而使系统稳定。请注意`native.cgroupdriver=systemd`下面Docker设置中的选项。

> **警告：**非常不推荐更改已加入群集的节点的cgroup驱动程序。如果kubelet使用一个cgroup驱动程序的语义创建了Pods，则在尝试为此类现有Pod重新创建PodSandbox时，将容器运行时更改为另一个cgroup驱动程序可能会导致错误。重新启动kubelet可能无法解决此类错误。建议将节点从其工作负载中排出，将其从群集中删除并重新加入。

### 安装Docker

```bash

# Install Docker CE
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add Docker apt repository.
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

## Install Docker CE.
apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "insecure-registries":["192.168.0.245:18071"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
   "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```

## CRI-O

使用以下命令在系统上安装CRI-O：

```shell
modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

```shell

# Install prerequisites
apt-get update
apt-get install software-properties-common

add-apt-repository ppa:projectatomic/ppa
apt-get update

# Install CRI-O
apt-get install cri-o-1.13
```

### 启动CRI-O

```shell
systemctl start crio
```

## Containerd

```shell
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

### 安装containerd

```shell
# Install containerd
## Set up the repository
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install -y apt-transport-https ca-certificates curl software-properties-common

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add Docker apt repository.
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

## Install containerd
apt-get update && apt-get install -y containerd.io

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Restart containerd
systemctl restart containerd
```

