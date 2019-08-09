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



Ubuntu 18.04 LTS设置固定ip

最近新装的Ubuntu 18.04 LTS搞起来还是略不习惯啊，相比之前的SUSE和CentOS差别还是比较大的。这不，想要配置个固定IP还搞了大半天。。。

总结一下踩坑过程吧。

系统版本：

root@ubuntu:/# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04 LTS
Release:    18.04
Codename:   bionic
root@ubuntu:/#

之前的版本网卡配置信息配置在/etc/network/interfaces文件，可以如下配置，

auto ens33
iface ens33 inet static
address 192.168.0.111
netmask 255.255.255.0
gateway 192.168.0.1

在18.04上也是可以用的，只是要重启才能生效。通过service networking restart无效。

下面介绍一下在18.04上新采用的netplan命令。网卡信息配置在/etc/netplan/01-network-manager-all.yaml文件，需做如下配置，

# Let NetworkManager manage all devices on this system
network:
  version: 2
  # renderer: NetworkManager
  ethernets:
          ens33:
                  addresses: [192.168.0.111/24]
                  gateway4: 192.168.0.1
                  nameservers:
                        addresses: [192.168.0.1]
然后使用以下命令使配置即时生效，

netplan apply
以上操作均在root用户下进行，如在普通用户，请自行加上sudo。
