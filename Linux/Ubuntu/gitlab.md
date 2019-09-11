<https://about.gitlab.com/install/#ubuntu>
1,安装并设置依赖

    sudo apt-get update
    sudo apt-get install -y curl openssh-server ca-certificates

2.官方的建议是使用脚本直接执行安装

    curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
    sudo apt-get install gitlab-ce

我是直接用下面的方式安装的

    <!-- wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/trusty/gitlab-ce_10.0.0-ce.0_amd64.deb/download.deb -->
    wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/debian/stretch/gitlab-ce_12.2.1-ce.0_amd64.deb
    sudo dpkg -i gitlab-ce-XXX.deb

以上是示例，具体版本需要进行替换，在 <https://packages.gitlab.com/gitlab/gitlab-ce> 中找到适合自己的 GitLab 版本，从 Download 获取到下载地址。
3，安装

    sudo dpkg -i gitlab-ce_10.0.0-ce.0_amd64.deb

使用以上命令进行安装。

打开/etc/gitlab/gitlab.rb,将external_url = '<http://git.example.com'修改为自己的域名地址：http://example.com，默认为80端口，如要使用其他端口后面加上端口号，如：http://127.0.0.1:8080。>

然后执行：

    sudo gitlab-ctl reconfigure

启动完成后浏览器访问配置好的地址，应该出现重置管理员密码的界面。

汉化
1.下载社区提供的汉化包，在 <https://gitlab.com/xhang/gitlab/> 中找到相应的汉化分支。

sudo wget wget -cO gitlab-9.0_zh.tar.gz <https://gitlab.com/xhang/gitlab/repository/archive.tar.gz?ref=9-0-stable-zh>
2.解压包

sudo tar zxvf gitlab-9.0_zh.tar.gz
3.停止 GitLab 服务

sudo gitlab-ctl stop
4.备份 gitlab-rails 目录，该目录下主要是web应用部分，也是当前项目仓库的起始版本，也是汉化包要覆盖的目录。

sudo tar zcvf /opt/gitlab/embedded/service/gitlab-rails-bak.tar.gz gitlab-rails
5.将解压后的汉化补丁覆盖原来的

sudo cp -rf gitlab-9-0-stable-zh/\* gitlab-rails/
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

    ssh-keygen -t rsa -C 'xxxx@163.com'

一路回车
然后在用户目录下.ssh 文件下面生成两个文件 id_rsa id_rsa.pub
id_rsa是私钥，id_rsa.pub是公钥
我们需要把id_rsa.pub文件中的内容拷贝一下
进入你自己的github，进入Settings->SSHand GPG keys->New SSH key,然后在Key那栏下面将id_rsa.pub粘贴进去就可以了，最后点击 Add SSH key按钮添加

本地配置多个ssh key
大多数时候，我们的机器上会有很多的git host,比如公司gitlab、github、oschina等，那我们就需要在本地配置多个ssh key，使得不同的host能使用不同的ssh key ,做法如下（以公司gitlab和github为例）：

为公司生成一对秘钥ssh key

ssh-keygen -t rsa -C 'yourEmail@xx.com' -f ~/.ssh/gitlab-rsa
为github生成一对秘钥ssh key

ssh-keygen -t rsa -C 'yourEmail2@xx.com' -f ~/.ssh/github-rsa
在~/.ssh目录下新建名称为config的文件（无后缀名）。用于配置多个不同的host使用不同的ssh key，内容如下：

# gitlab

Host gitlab.com
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab_id-rsa

# github

Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id-rsa
  ​

# 配置文件参数

# Host : Host可以看作是一个你要识别的模式，对识别的模式，进行配置对应的的主机名和ssh文件

# HostName : 要登录主机的主机名

# User : 登录名

# IdentityFile : 指明上面User对应的identityFile路径

按照上面的步骤分别往gitlab和github上添加生成的公钥gitlab_id-rsa.pub和github_id-rsa.pub
OK，大功告成，再次执行git命令验证是不是已经不需要再次验证权限了。

再次查看~/..ssh目录下的文件,会有gitlab_id-rsa、gitlab_id-rsa.pub和github_id-rsa、github_id-rsa.pub四个文件
