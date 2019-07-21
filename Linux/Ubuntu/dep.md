1.使用dpkg -i安装deb包
   语法:
   dpkg -i package-file-name
   如下所示，你可以使用dpkg -l +名称 来验证安装
  $ dpkg -l | grep 'tcl'
2.使用kpkg -r来删除deb包

   dpkg 加上 -r参数，用于卸载已安装好的软件包
   $ dpkg -r tcl8.4
   以下命令表示彻底卸载软件包（包括配置文件）.
     $ dpkg -P tcl8.4



dpkg安装deb缺少依赖包的解决方法
【先贴出解决方案（基于Ubuntu）】：

使用dpkg -i   *.deb 的时候出现依赖没有安装

使用apt-get -f -y install  解决依赖问题后再执行dpkg安装deb包

=====================1.下面是遇到的依赖没有安装===========================

问题

horizon@horizon-pc ~/下载 $ sudo dpkg -i youdao-dict_1.1.0-0-ubuntu_amd64.deb
[sudo] password for horizon:
Selecting previously unselected package youdao-dict.
(正在读取数据库 ... 系统当前共安装有 163525 个文件和目录。)
Preparing to unpack youdao-dict_1.1.0-0-ubuntu_amd64.deb ...
Unpacking youdao-dict (1.1.0-0~ubuntu) ...
dpkg: dependency problems prevent configuration of youdao-dict:
 youdao-dict 依赖于 python3-pyqt5；然而：
  未安装软件包 python3-pyqt5。
 youdao-dict 依赖于 python3-requests；然而：
  未安装软件包 python3-requests。
Processing triggers for hicolor-icon-theme (0.13-1) ...
Processing triggers for gnome-menus (3.10.1-0ubuntu2) ...
Processing triggers for desktop-file-utils (0.22-1ubuntu1) ...
Processing triggers for mime-support (3.54ubuntu1) ...
在处理时有错误发生：
 youdao-dict


===============2.解决依赖========================

执行命令：

horizon@horizon-pc ~/下载 $ sudo apt-get -f -y install

==============3.重新使用dpkg安装deb包=============

horizon@horizon-pc ~/下载 $ sudo dpkg -i youdao-dict_1.1.0-0-ubuntu_amd64.deb
