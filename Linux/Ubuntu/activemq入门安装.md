演示环境： Centos7、jdk8、activemq5.15.8
下载地址： http://activemq.apache.org/activemq-5158-release.html
解压： tar -zxvf apache-activemq-5.15.8-bin.tar.gz -C /var
修改目录名称 mv /var/apache-activemq-5.15.8/ /var/activemq/

启动： ./bin/activemq start
停止：./bin/activemq stop

做成系统服务

1、创建一个systemd服务文件：vi  /usr/lib/systemd/system/activemq.service

2、 放入内容

[Unit]
Description=ActiveMQ service
After=network.target
[Service]
Type=forking
ExecStart=/var/activemq/bin/activemq start
ExecStop=/var/activemq/bin/activemq stop
User=root
Group=root
Restart=always
RestartSec=9
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=activemq
[Install]
WantedBy=multi-user.target

3、 找到java命令所在的目录 whereis java

4、设置activemq配置文件/var/activemq/bin/env中的JAVA_HOME

# Location of the java installation
# Specify the location of your java installation using JAVA_HOME, or specify the
# path to the "java" binary using JAVACMD
# (set JAVACMD to "auto" for automatic detection)
JAVA_HOME="/usr/local/java/jdk1.8.0_181"
JAVACMD="auto"

5、 通过systemctl管理activemq启停
•启动activemq服务: systemctl start activemq
•查看服务状态: systemctl status activemq
•创建软件链接：ln -s /usr/lib/systemd/system/activemq.service /etc/systemd/system/multi-user.target.wants/activemq.service
•开机自启: systemctl enable activemq
•检测是否开启成功(enable)： systemctl list-unit-files |grep activemq

6、 防火墙配置，Web管理端口默认为8161，通讯端口默认为61616
•添加并重启防火墙
firewall-cmd --zone=public --add-port=8161/tcp --permanent

firewall-cmd --zone=public --add-port=61616/tcp --permanent

systemctl restart firewalld.service
