https://about.gitlab.com/install/#ubuntu
1,安装并设置依赖
```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
```
2.官方的建议是使用脚本直接执行安装
```
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce
```
如果你能通过以上方式安装，恭喜你的网络很好，但一般因为大墙的存在这个方式很多时候并不能成功，所以我们要通过手动下载包的方式进行安装。

```
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/trusty/gitlab-ce_10.0.0-ce.0_amd64.deb/download.deb
sudo dpkg -i gitlab-ce-XXX.deb
```
以上是示例，具体版本需要进行替换，在 https://packages.gitlab.com/gitlab/gitlab-ce 中找到适合自己的 GitLab 版本，从 Download 获取到下载地址。
3，安装
```
sudo dpkg -i gitlab-ce_10.0.0-ce.0_amd64.deb
```
使用以上命令进行安装。

打开/etc/gitlab/gitlab.rb,将external_url = 'http://git.example.com'修改为自己的域名地址：http://example.com，默认为80端口，如要使用其他端口后面加上端口号，如：http://127.0.0.1:8080。

然后执行：

```
sudo gitlab-ctl reconfigure
```
启动完成后浏览器访问配置好的地址，应该出现重置管理员密码的界面。


汉化
1.下载社区提供的汉化包，在 https://gitlab.com/xhang/gitlab/ 中找到相应的汉化分支。

sudo wget wget -cO gitlab-9.0_zh.tar.gz https://gitlab.com/xhang/gitlab/repository/archive.tar.gz?ref=9-0-stable-zh
2.解压包

sudo tar zxvf gitlab-9.0_zh.tar.gz
3.停止 GitLab 服务

sudo gitlab-ctl stop
4.备份 gitlab-rails 目录，该目录下主要是web应用部分，也是当前项目仓库的起始版本，也是汉化包要覆盖的目录。

sudo tar zcvf /opt/gitlab/embedded/service/gitlab-rails-bak.tar.gz gitlab-rails
5.将解压后的汉化补丁覆盖原来的

sudo cp -rf gitlab-9-0-stable-zh/* gitlab-rails/
6.启动服务

sudo gitlab-ctl start
7.重新执行配置命令

sudo gitlab-ctl reconfigure
汉化完成

一些界面设置
进入界面后关掉一些我们可能用不到的设置，在 「管理区域」的设置中进行更改


gitlab ssh key 配置
git客户端生成ssh key，然后配置在gitlab里，而后使用ssh协议进行提交和拉取git远程仓库的代码
使用如下命令生成ssh公钥和私钥对
```
ssh-keygen -t rsa -C 'xxxx@163.com'
```
一路回车
然后在用户目录下.ssh 文件下面生成两个文件 id_rsa id_rsa.pub
