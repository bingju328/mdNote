mysql
下载安装
安装的方式很简单：更新软件包索引，安装mysql-server软件包，然后运行附带的安全脚本即可。
sudo apt-get update
sudo apt-get install mysql-server
sudo mysql_secure_installation

按上边方式安装完成后，MySQL应该已经开始自动运行了。要测试它，请检查其状态。

systemctl status mysql.service
您将看到类似于以下内容的输出：


mysql.service - MySQL Community Server
Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: en Active: active (running) since Wed 2016-11-23 21:21:25 UTC; 30min ago Main PID: 3754 (mysqld) Tasks: 28 Memory: 142.3M CPU: 1.994s CGroup: /system.slice/mysql.service └─3754 /usr/sbin/mysqld
如果MySQL没有运行，您可以启动它：

sudo systemctl mysql start
如果额外的检查，您可以尝试使用该 mysqladmin 工具连接到数据库，该工具是允许您运行管理命令的客户端。例如，该命令表示以 root（-u root）方式连接到 MySQL ，提示输入密码（-p）并返回版本。

mysqladmin -p -u root version

sudo netstat -tap | grep mysql

通过上述命令检查之后，如果看到有mysql 的socket处于 listen 状态则表示安装成功。

//通过地址进入控制台
mysql -h 127.0.0.1 -u root -p

登陆mysql数据库可以通过如下命令：

mysql -u root -p 登录
show databases; 查看当前的数据库
use mysql 选则库
show tables 显示当前数据库的表单
desc user; 显示行

查看端口
netstat -an|grep 3306

修改访问权限
进入目录“etc/mysql/mysql.conf.d/
sudo vim mysqld.cnf
By default we only accept connections from localhost”，这几句话的意思是说“在默认情况下我们只允许本地服务访问MySQL”，所以我们需要注释掉下方那条配置，直接在它前面加上一个井号即可：

# bind-address = 127.0.0.1
开放root账户的访问权限
在第三步中，我们仅仅只是取消了本地访问限制，但是我们还是没有对账户权限进行设置。
重启MySQL服务，并进入MySQL控制台：

service mysql stop
service mysql start
mysql -h 127.0.0.1 -u root -p
//把root用户改成非本地访问
update user set host='%' where user='root';

---------------------

一. 创建用户
命令:
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
说明：
username：你将创建的用户名
host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符%
password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
例子：
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'pig'@'192.168.1.101_' IDENDIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '';
CREATE USER 'pig'@'%';
二. 授权:
命令:
GRANT privileges ON databasename.tablename TO 'username'@'host'
说明:
privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
databasename：数据库名
tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*
例子:
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
GRANT ALL ON maindataplus.* TO 'pig'@'%';
注意:
用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
三.设置与更改用户密码
命令:
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
如果是当前登陆用户用:

SET PASSWORD = PASSWORD("newpassword");
例子:
SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
四. 撤销用户权限
命令:
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
说明:
privilege, databasename, tablename：同授权部分

例子:
REVOKE SELECT ON *.* FROM 'pig'@'%';
注意:
假如你在给用户'pig'@'%'授权的时候是这样的（或类似的）：GRANT SELECT ON test.user TO 'pig'@'%'，则在使用REVOKE SELECT ON *.* FROM 'pig'@'%';命令并不能撤销该用户对test数据库中user表的SELECT 操作。相反，如果授权使用的是GRANT SELECT ON *.* TO 'pig'@'%';则REVOKE SELECT ON test.user FROM 'pig'@'%';命令也不能撤销该用户对test数据库中user表的Select权限。

具体信息可以用命令SHOW GRANTS FOR 'pig'@'%'; 查看。

五.删除用户
命令:
DROP USER 'username'@'host';




[client]
port=3306
[mysql]
no-beep
default-character-set=utf8
[mysqld]
server-id=2
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin
slave-skip-errors=all #跳过所有错误
skip-name-resolve

port=3306
datadir="D:/mysql-slave/data"
character-set-server=utf8
default-storage-engine=INNODB
sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

log-output=FILE
general-log=0
general_log_file="WINDOWS-8E8V2OD.log"
slow-query-log=1
slow_query_log_file="WINDOWS-8E8V2OD-slow.log"
long_query_time=10

# Binary Logging.
# log-bin

# Error Logging.
log-error="WINDOWS-8E8V2OD.err"


# 整个数据库最大连接（用户）数
max_connections=1000
# 每个客户端连接最大的错误允许数量
max_connect_errors=100
# 表描述符缓存大小，可减少文件打开/关闭次数
table_open_cache=2000
# 服务所能处理的请求包的最大大小以及服务所能处理的最大的请求大小(当与大的BLOB字段一起工作时相当必要)  
# 每个连接独立的大小.大小动态增加
max_allowed_packet=64M
# 在排序发生时由每个线程分配
sort_buffer_size=8M
# 当全联合发生时,在每个线程中分配
join_buffer_size=8M
# cache中保留多少线程用于重用
thread_cache_size=128
# 此允许应用程序给予线程系统一个提示在同一时间给予渴望被运行的线程的数量.
thread_concurrency=64
# 查询缓存
query_cache_size=128M
# 只有小于此设定值的结果才会被缓冲  
# 此设置用来保护查询缓冲,防止一个极大的结果集将其他所有的查询结果都覆盖
query_cache_limit=2M
# InnoDB使用一个缓冲池来保存索引和原始数据
# 这里你设置越大,你在存取表里面数据时所需要的磁盘I/O越少.  
# 在一个独立使用的数据库服务器上,你可以设置这个变量到服务器物理内存大小的80%  
# 不要设置过大,否则,由于物理内存的竞争可能导致操作系统的换页颠簸.  
innodb_buffer_pool_size=1G
# 用来同步IO操作的IO线程的数量
# 此值在Unix下被硬编码为4,但是在Windows磁盘I/O可能在一个大数值下表现的更好.
innodb_read_io_threads=16
innodb_write_io_threads=16
# 在InnoDb核心内的允许线程数量.  
# 最优值依赖于应用程序,硬件以及操作系统的调度方式.  
# 过高的值可能导致线程的互斥颠簸.
innodb_thread_concurrency=9

# 0代表日志只大约每秒写入日志文件并且日志文件刷新到磁盘.  
# 1 ,InnoDB会在每次提交后刷新(fsync)事务日志到磁盘上
# 2代表日志写入日志文件在每次提交后,但是日志文件只有大约每秒才会刷新到磁盘上
innodb_flush_log_at_trx_commit=2
# 用来缓冲日志数据的缓冲区的大小.  
innodb_log_buffer_size=16M
# 在日志组中每个日志文件的大小.  
innodb_log_file_size=48M
# 在日志组中的文件总数.
innodb_log_files_in_group=3
# 在被回滚前,一个InnoDB的事务应该等待一个锁被批准多久.  
# InnoDB在其拥有的锁表中自动检测事务死锁并且回滚事务.  
# 如果你使用 LOCK TABLES 指令, 或者在同样事务中使用除了InnoDB以外的其他事务安全的存储引擎  
# 那么一个死锁可能发生而InnoDB无法注意到.  
# 这种情况下这个timeout值对于解决这种问题就非常有帮助.
innodb_lock_wait_timeout=30
# 开启定时
event_scheduler=ON

查看Mysql存储引擎情况：
mysql>show engines;
查看当前变量
mysql> show variables like "%innodb%";

show global VARIABLES like 'innodb_flush_log_at_trx_commit'
