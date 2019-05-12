ubuntu A 登录 B，C
A上
1，
ssh-keygen -t rsa  # 生成RSA加密的密钥
2，
ssh-copy-id 用户名@ip地址  # 上传公钥到服务器

ssh登录的配置，即/etc/ssh/sshd_config文件，修改为允许root登录，可以执行命令

sudo vim /etc/ssh/sshd_config
