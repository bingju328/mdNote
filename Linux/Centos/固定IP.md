1. 设置路由器： 首先我们要有路由器的登录账号和密码，登录路由器后设定mac地址与IP地址绑定，这样每次开机路由器都会给我们分配一个固定的IP地址。这种方式是最稳定可靠的方式，但很多情况下我们并没有登录路由器的权限，尤其是在办公区，并不十分通用，这里不做详细说明。

　　2. 配置系统： 我们都知道window系统有自动获取IP和手动配置IP地址两种方式，linux也支持手动配置。(以下操作我都是在管理员权限下完成) 首先在linux系统下获取网卡名,终端下输入ifconfig或者ipaddr
我这里网卡名为：enp2s0，同时记录下掩码地址，下面会用到这两个参数。

　　终端输入vi /etc/network/命令编辑配置文件,增加如下内容： 　　　　
　　　　auto enp2s0
　　　　iface enp2s0 inet static
　　　　address 192.168.1.211
　　　　netmask 255.255.255.0
　　　　gateway 192.168.1.1
　　　　iface enp2s0 inet6 auto
sudo /etc/init.d/networking restart 
---------------------
作者：lzhitsh
来源：CSDN
原文：https://blog.csdn.net/lzhitwh/article/details/82773335
版权声明：本文为博主原创文章，转载请附上博文链接！
