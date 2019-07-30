git密码缓存

git是没有记录密码用户名的，于是乎只能自己设置，在命令行设置咋设置，代码如下。。

git config --global user.name "myusername"
git config --global user.email "myusername@myemaildomain.com"
git config --global credential.helper cache


但是最后一行  cache不是windows 下的，于是乎运行总是报错，
git: 'credential-cache' is not a git command. See 'get --help'.
对就它，老报这个问题，咋整，要用win下边的，

有人用：
git config --global credential.helper winstore

用这个解决问题了，我的还是不行。就试了下边这个

git config --global credential.helper wincred




2、初始化

git init
3、自己要与origin master建立连接（下划线为远程仓库链接）
git remote add origin git@github.com:XXXX/nothing2.git

4、把远程分支拉到本地
git fetch origin dev（dev为远程仓库的分支名）

5、在本地创建分支dev并切换到该分支
git checkout -b dev(本地分支名称) origin/dev(远程分支名称)

6、把某个分支上的内容都拉取到本地
git pull origin dev(远程分支名称)
