#服务器git部署新项目
1.创建项目文件夹 mkdir projectName
2.拉取项目文件可以使用 【上传项目.git文件夹 -> git pull】【如果不同服务器则可以通过scp命令拷贝文件到当前服务器】
2.创建shell update git 文件【project_update.sh】
if [ -n "$1" ]
then
   BRANCH_NAME=$1
else
   #默认用master分支部署
   BRANCH_NAME="master"
fi

echo "deploy branch: $BRANCH_NAME"
cd [projectName path]
#git fetch
git checkout $BRANCH_NAME
git pull
mvn clean package -DskipTests -U -B
使用-U参数： 该参数能强制让Maven检查所有SNAPSHOT依赖更新，确保集成基于最新的状态，如果没有该参数，Maven默认以天为单位检查更新，而持续集成的频率应该比这高很多。
使用-B参数：该参数表示让Maven使用批处理模式构建项目，能够避免一些需要人工参与交互而造成的挂起状态。

3.创建shell执行启动项目文件
#!/bin/bash
sh project_update.sh $1
targetPid=`jps | grep goods-service.jar | awk '{print $1}'`
#goods-service.jar name为pom文件中打出的jar的name
if [ -n '$targetPid' ]
then
  echo "find goods-service pid $targetPid, try to kill it"
  kill -9 $targetPid
else  
  echo "no goods-service alive"  
fi  
#检测是否有good-service.jar的启动进程,如果有则kill -9 pid
cd /home/lin/dep/spkitty-goods
nohup java -agentlib:jdwp=transport=dt_socket,address=8001,server=y,suspend=n -Dspring.profiles.active=test -Dmanagement.port=18090 -Dserver.port=18191 -jar spkitty-goods-service/target/goods-service.jar >/home/lin/log/goods-service.log 2>/h
ome/lin/log/goods-service-err.log &
#设置端口address ,-Dspring.profiles.active=test 指定为测试环境 设置管理端口18190  设置启动端口18191  若未引入manager jar包，则可以不设置managent.prot端口

tail -200f /home/lin/log/goods-service.log
#打印日志
