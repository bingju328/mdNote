
docker 卸载
apt-get remove docker-ce docker-ce-cli containerd.io
rm -rf /val/lib/docker
docker安装

1,在第一次在新主机上安装Docker CE之前，需要设置Docker存储库。然后，您可以从存储库安装和更新Docker。
1，
```
apt-get update
```
2,安装包以允许apt通过HTTPS使用存储库:
```
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```
3,添加Docker的官方GPG密钥:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
通过搜索指纹的最后8个字符，验证您现在拥有指纹9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88的密钥。
```
apt-key fingerprint 0EBFCD88
```
4,使用以下命令设置稳定的存储库。要添加夜间或测试存储库，请在下面的命令中在单词stable后面添加词语nightly或test(或两者都)。了解夜间和测试频道。
```
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
2,INSTALL DOCKER CE
1，更新一下apt包索引：
```
apt-get update
```
2,安装最新版本的Docker CE和containerd，或进入下一步安装特定版本:
```
apt-get install docker-ce docker-ce-cli containerd.io
```
3,要安装特定版本的Docker CE，请在repo中列出可用版本，然后选择并安装:
a.列出你的仓库中可用的版本:
```
apt-cache madison docker-ce
```
b.使用第二列中的版本字符串安装特定的版本，例如，5:18.09.1~3-0~ubuntu-xenial。
```
apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```
4,通过运行hello-world映像，验证Docker CE是否正确安装
```
docker run hello-world
```

docker.service
lib/systemd/system/docker.service
usr/lib/systemd/system/docker.servie
•创建软件链接：ln -s /usr/lib/systemd/system/docker.service /etc/systemd/system/multi-user.target.wants/docker.service

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket
#Requires=docker.socket，也就是说，docker.service
#默认依赖于docker.socket，因为需要使用docker.socket来获取容器的信息。
#/etc/systemd/system/docker.service.requires/ 这个里面有requires的软连接

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
