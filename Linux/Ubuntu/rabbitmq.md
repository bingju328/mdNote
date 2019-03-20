准备工作
一台服务器：Ubuntu Server 16.04.1 LTS 64位

安装RabbitMq
可以参照RabbitMq官网的安装教程(Installing on Debian and Ubuntu)，来进行安装。
这里我们使用apt-get来安装，就简单的几条命令：

1.由于rabbitMq需要erlang语言的支持，在安装rabbitMq之前需要安装erlang，执行命令：

apt-get install erlang-nox     # 安装erlang
erl    # 查看relang语言版本，成功执行则说明relang安装成功

2.添加公钥

wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -

3.更新软件包

apt-get update

4.安装 RabbitMQ

apt-get install rabbitmq-server  #安装成功自动启动

5.查看 RabbitMq状态

systemctl status rabbitmq-server   #Active: active (running) 说明处于运行状态

# service rabbitmq-server status 用service指令也可以查看，同systemctl指令

6.启动、停止、重启

service rabbitmq-server start    # 启动
service rabbitmq-server stop     # 停止
service rabbitmq-server restart  # 重启

执行了上面的步骤，rabbitMq已经安装成功。

7.启用 web端可视化操作界面，我们还需要配置Management Plugin插件

rabbitmq-plugins enable rabbitmq_management   # 启用插件
service rabbitmq-server restart    # 重启

此时，应该可以通过 http://localhost:15672 查看，使用默认账户guest/guest 登录。
注意：RabbitMQ 3.3 及后续版本，guest 只能在服务本机登录。
瞄了一眼官方文档，说的是默认会创建guest用户，但是只能服务器本机登录，建议创建其他新用户，授权，用来做其他操作。


8.查看用户

rabbitmqctl list_users

9.添加管理用户

rabbitmqctl add_user admin yourpassword   # 增加普通用户
rabbitmqctl set_user_tags admin administrator    # 给普通用户分配管理员角色

10.为用户分配资源权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
