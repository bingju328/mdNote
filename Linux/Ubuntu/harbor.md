lHarbor 环境依赖：docker docker-compose

1、开SSH

systemctl enable sshd

2、关闭SELINUX

vi /etc/sysconfig/selinux

修改下边红字部分

\# This file controls the state of SELinux on the system.

\# SELINUX= can take one of these three values:

\# enforcing - SELinux security policy is enforced.

\# permissive - SELinux prints warnings instead of enforcing.

\# disabled - No SELinux policy is loaded.

SELINUX=disabled

\# SELINUXTYPE= can take one of three two values:

\# targeted - Targeted processes are protected,

\# minimum - Modification of targeted policy. Only selected processes are protected.

\# mls - Multi Level Security protection.

SELINUXTYPE=targeted

#### 安装docker

#### 安装docker-compose

#### 安装harbor

#### 下载Harbor

官网地址：https://github.com/goharbor/harbor/releases

```shell
wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-online-installer-v1.8.2.tgz

tar -xvf harbor-online-installer-v1.8.2.tgz
```

#### 配置harbor

1. 修改harbor.cfg

   ```shell
   cd harbor
   vi harbor.cfg
   ```

   将 hostname的值修改成本机IP，比如198.127.0.1

   部分配置含义：

   ```shell
   #配置访问的地址
   hostname = 198.127.0.1
   #使用http方式访问管理界面
   ui_url_protocol = http
   #配置admin的密码，默认是Harbor12345
   harbor_admin_password = 12345
   #更改harbor存储路径，默认是/data
   secretkey_path = /mnt/vdc/harbor_data
   ```
   
2. 开始安装

   ```
   ./install.sh
   ```

3. 配置端口

   - 第一种方式 ： 修改docker-compose.yml

     ```yaml
       proxy:
         image: goharbor/nginx-photon:v1.8.2
         container_name: nginx
         restart: always
         cap_drop:
           - ALL
         cap_add:
           - CHOWN
           - SETGID
           - SETUID
           - NET_BIND_SERVICE
         volumes:
           - ./common/config/nginx:/etc/nginx:z
         networks:
           - harbor
         dns_search: .
         #这个地方修改
         ports:
           - 18071:80
         depends_on:
           - postgresql
           - registry
           - core
           - portal
           - log
     ```

     

   - 第二种方式 ： 修改config.yml

4. 重新生成配置文件

   ```shell
   ./prepare
   ```

#### 启动harbor

```shell
docker-compose up -d
#停止Harbor：docker-compose down -v
```

#### **修改insecure-registry**

Linux，比如Ubuntu
有好几种方式，主要分为 修改daemon.json增加仓库地址（推荐） 和  在docker启动文件配置仓库地址

下面方法二选一（推荐第一种，方便）

- 修改daemon.json

  文件目录：/etc/docker/daemon.json （没有则新建该文件）

  ```yaml
  {
  "insecure-registries": ["198.127.0.1:18071"]
  }
  ```

  然后重启docker：

  ```shell
  service docker restart
  ```

- 修改启动文件 docker.service,增加Harbor的Ip和Port

  ```shell
  vim /lib/systemd/system/docker.service 
  #在ExecStart的最后增加：--insecure-registry=198.127.0.1:18071
  
  #重新启动docker：
  #修改docker.service一定要执行systemctl daemon-reload刷新配置
  systemctl daemon-reload
  systemctl restart docker
  
  ```

####  docker push镜像：

1. 登录Harbor

   ```shell
   docker login 198.127.0.1:18071
   #输入
   #用户名admin
   #密码Harbor12345
   docker logout
   ```

2. docker tag 和 docker push：

   ```shell
   docker tag blog:v2.0 192.127.0.1:18071/tom/blog:v2.0
   #push之前需要在harbor先创建一个tom项目
   docker push 192.127.0.1:18071/tom/blog:v2.0
   #查看仓库
   curl 192.168.0.245:18071/v2/_catalog
   #curl -XGEThttp://192.168.1.8:5000/v2/_catalog
   #查看harbor仓库
   curl -u "admin" -X GET -H "Content-Type:application/json"  "http://192.168.0.245:18071/api/search?"
   ```

#### push常见错误：

（1）Error response from daemon: Get https://192.127.0.1/v1/users/: dial tcp 192.127.0.1:443: getsockopt: connection refused

（2）Error response from daemon: Get https://192.127.0.1:81/v1/users/: http: server gave HTTP response to HTTPS client

原因：

1 端口错了，比如不是默认端口80，而是18071

2 没有在docker启动文件中添加--insecure-registry 信任关系，解决办法在上面  3 修改insecure-registry





proxy是registry v2的pull though cache功能。开启了cache的话就无法做push了。registry的官网有这个说明。
因为我这里是harbor的本地仓库 并且是docker部署的
之后找到harbor registry的配置文件
common/config/registry/config.yml
