
内存使用情况
1.

命令行输入：top

2.

安装命令如下：sudo apt-get install htop

安装完后，直接输入命令：htop

使用pmap查看进程内存
运行命令
使用pmap可以查看某一个进程（非java的也可以）的内存使用使用情况，
命令格式：
pmap 进程id
示例说明
例如运行：
pmap 12358

使用jmap查看Java进程对象使用情况
运行命令
使用jmap可以查看某个Java进程中每个对象有多少个实例，占用多少内存，
命令格式：
jmap -histo 进程id
示例说明
例如运行：
jmap -histo  12538
