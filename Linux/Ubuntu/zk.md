zk zoo.conf中
集群配制
server.1=127.0.0.1:8880:7770
server.2=127.0.0.1:8881:7771
server.3=127.0.0.1:8882:7772

这里的1，2，3是每个zk的mypid 文件中的id
127.0.0.1 ip
8880:7770 这个只要是没有被占用的端口就可以
