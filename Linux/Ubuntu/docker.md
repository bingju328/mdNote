要安装Docker CE，您需要这些Ubuntu版本之一的64位版本
Cosmic 18.10
Bionic 18.04 (LTS)
Xenial 16.04 (LTS)
卸载旧版本
&sudo apt-get remove docker docker-engine docker.io containerd runc
保存/var/lib/docker/的内容，包括图像、容器、卷和网络


在第一次在新主机上安装Docker CE之前，需要设置Docker存储库。然后，您可以从存储库安装和更新Docker。
apt-get update
安装包以允许apt通过HTTPS使用存储库:
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
添加Docker的官方GPG密钥:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

通过搜索指纹的最后8个字符，验证您现在拥有指纹9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88的密钥。
apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
使用以下命令设置稳定的存储库。要添加夜间或测试存储库，请在下面的命令中在单词stable后面添加词语nightly或test(或两者都)。了解夜间和测试频道。
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

安装Docker ce
apt-get update  
apt-get install docker-ce docker-ce-cli containerd.io
