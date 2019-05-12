nexus3下载部署
环境是Ubuntu16.04 + JDK8 + MAVEN3.3.9,首先需要安装 JDK8 和 Maven3
1,下载解压
去官网下载最新的https://www.sonatype.com/download-oss-sonatype
->Nexus Repository Manager OSS 3.x - Unix

解压(/usr/nexus3)并修改目录名为nexus-3
```
>/usr/nexus3# tar -zxvf nexus-3.15.2-01-unix.tar.gz
>mv /usr/nexus3/nexus-3.15.2-01 nexus-3
```
2,修改nexus的运行用户为root
内容修改为:run_as_user="root"
3,修改nexus启动时要使用的jdk版本
新增如下内容:INSTALL4J_JAVA_HOME_OVERRIDE=/usr/jdk8
```
>vi /usr/nexus3/nexus-3/bin/nexus.rc
```
4,修改nexus默认端口(可选),以及允许远程机器访问
```
>vi usr/nexus3/nexus-3/etc/nexus-default.properties
```
application-port=18081
application-host=0.0.0.0

5,启动nexus服务
```
>./usr/nexus3/nexus-3/bin/nexus run &
```
启动中看到下图就表示成功了，在root用户下启动会有警告直接忽略就可以了（警告是不推荐用root用户启动）
-------------------------------------------------

Started Sonatype Nexus OSS 3.15.2-01

-------------------------------------------------
参考文档:https://jingyan.baidu.com/article/ff42efa9d526e4c19e220215.html

Nexus默认的端口是8081，可以在etc/nexus-default.properties配置中修改。
Nexus默认的用户名密码是admin/admin123
当遇到奇怪问题时，重启nexus，重启后web界面要1分钟左右后才能访问。
Nexus的工作目录是sonatype-work（路径一般在nexus同级目录下），日志文件也在这里。
---------------------

//打包上传 -s 指定配置
mvn deploy -s C:\Users\Administrator.000\.m2\settings328.xml
//注意 idel里面指定的配置只能在idel右边工具栏里面用，命令行默认用的是settings.xml
//如有特殊需要指定配置文件
