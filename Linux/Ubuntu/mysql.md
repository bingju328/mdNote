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
