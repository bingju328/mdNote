Redis下载安装
下载redis
https://redis.io/download

下载解压
wget http://download.redis.io/releases/redis-5.0.4.tar.gz
tar -zxvf redis-5.0.4.tar.gz
cd redis-5.0.4
在make前需要安装gcc和一些库函数
如没装make可先运行 sudo apt-get install make
sudo apt-get install build-essential
编译安装
make
修改redis.conf
将daemonize no改为 daemonize yes
启动方式改为后台启动
启动
./redis-server

解决redis远程连接不上的问题

redis现在的版本开启redis-server后，redis-cli只能访问到127.0.0.1，因为在配置文件中固定了ip，因此需要修改redis.conf（有的版本不是这个文件名，只要找到相对应的conf后缀的文件即可）文件以下几个地方。

1.bind 127.0.0.1改为 #bind 127.0.0.1
protected-mode no

2.protected-mode yes 改为 protected-mode no

3.加入 daemonize no(这个是是否在后台启动不占用一个主程窗口)

4.设置密码为  redis123
requirepass redis123

把redis做成系统服务
创建一个systemd服务文件：vi  /usr/lib/systemd/system/redis.service
/usr/redis/redis-5.0.4这个是我的redis下载解压目录
```
[Unit]
Description=Redis service
After=network.target
[Service]
Type=forking
ExecStart=/usr/redis/redis-5.0.4/src/redis-server /usr/redis/redis-5.0.4/redis.conf
ExecStop=/usr/redis/redis-5.0.4/src/redis-cli -p 6379 shutdown
Restart=always
User=root
Group=root
RestartSec=5
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=redis

[Install]
WantedBy=multi-user.target
```
启动、查看状态、停止服务
systemctl start redis
systemctl status redis
systemctl stop redis
•创建软件链接：ln -s /usr/lib/systemd/system/redis.service /etc/systemd/system/multi-user.target.wants/redis.service
•开机自启: systemctl enable redis
•关闭开机自启: systemctl disable redis
•检测是否开启成功(enable)： systemctl list-unit-files |grep redis

启动服务后
打开客户端输入密码
```
>./redis-cli
>auth redis123
```

redis集群
# 配置文件进行了精简，完整配置可自行和官方提供的完整conf文件进行对照。端口号自行对应修改
#后台启动的意思
daemonize yes
 #端口号
port 6381
# IP绑定，redis不建议对公网开放，直接绑定0.0.0.0没毛病
bind 0.0.0.0
# redis数据文件存放的目录
dir /usr/local/redis/data
# 开启AOF
appendonly yes
 # 开启集群
cluster-enabled yes
# 会自动生成在上面配置的dir目录下
cluster-config-file nodes-6381.conf
cluster-node-timeout 5000
# 这个文件会自动生成
pidfile /var/run/redis_6381.pid

2，启动每一个redis
3,创建cluster
./redis-cli -a redis --cluster create node328:6379 node203:6379 node512:6379 --cluster-replicas 1

redis哨兵高可用(这个需要在关闭集群的情况下搭建)
否则会报这个 (error) ERR REPLICATION not allowed in cluster mode.
配置文件如下：
cluster-enabled no
# 配置文件进行了精简，完整配置可自行和官方提供的完整conf文件进行对照。端口号自行对应修改
#后台启动的意思
daemonize yes
 #端口号(如果同一台服务器上启动，注意要修改为不同的端口)
port 6380
# IP绑定，redis不建议对公网开放，直接绑定0.0.0.0没毛病
bind 0.0.0.0
# 这个文件会自动生成(如果同一台服务器上启动，注意要修改为不同的端口)
pidfile /var/run/redis_6379.pid
replicaof node328 6379
masterauth

1、启动三个Redis
2、配置为 1主2从
./redis-cli -a redis123 -p 6379 slaveof node328 6379
./redis-cli -a redis123 -p 6379 slaveof node328 6379
3、检查集群
./redis-cli -a redis123 -p 6379 info replication

准备哨兵配置文件

# 配置文件：sentinel.conf，在sentinel运行期间是会被动态修改的
# sentinel如果重启时，根据这个配置来恢复其之前所监控的redis集群的状态
# 绑定IP
bind 0.0.0.0
# 后台运行
daemonize yes
# 默认yes，没指定密码或者指定IP的情况下，外网无法访问
protected-mode no
# 哨兵的端口，客户端通过这个端口来发现redis
port 26380
# 哨兵自己的IP，手动设定也可自动发现，用于与其他哨兵通信
# sentinel announce-ip
# 临时文件夹
dir /tmp
# 日志
logfile "/usr/local/redis/logs/sentinel-26380.log"
# sentinel监控的master的名字叫做mymaster,初始地址为 192.168.100.241 6380,2代表两个及以上哨兵认定为死亡，才认为是真的死亡
sentinel monitor mymaster 192.168.100.241 6380 2
# 发送心跳PING来确认master是否存活
# 如果master在“一定时间范围”内不回应PONG 或者是回复了一个错误消息，那么这个sentinel会主观地(单方面地)认为这个master已经不可用了
sentinel down-after-milliseconds mymaster 1000
# 如果在该时间（ms）内未能完成failover操作，则认为该failover失败
sentinel failover-timeout mymaster 3000
# 指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长
sentinel parallel-syncs mymaster 1
#密码
sentinel auth-pass mymaster redis123

启动哨兵
./src/redis-server sentinel.conf --sentinel
./src/redis-server sentinel.conf --sentinel
./src/redis-server sentinel.conf --sentinel

查看哨兵状态
./src/redis-cli -a redis123 -h node203 -p 26379 info Sentinel
$ redis-cli -p 26379
127.0.0.1:5000> sentinel master mymaster


测试
# 停掉master，主从切换过程
启动哨兵(客户端通过哨兵发现Redis实例信息)
哨兵通过连接master发现主从集群内的所有实例信息
哨兵监控redis实例的健康状况
哨兵一旦发现master不能正常提供服务，则通知给其他哨兵
当一定数量的哨兵都认为master挂了
选举一个哨兵作为故障转移的执行者
执行者在slave中选取一个作为新的master
将其他slave重新设定为新master的从属
