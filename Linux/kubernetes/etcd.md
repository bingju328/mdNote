1. etcdctl的安装
etcdctl的二进制文件可以在 github.com/coreos/etcd/releases 选择对应的版本下载，例如可以执行以下install_etcdctl.sh的脚本，修改其中的版本信息。

#!/bin/bash
ETCD_VER=v3.3.4
ETCD_DIR=etcd-download
DOWNLOAD_URL=https://github.com/coreos/etcd/releases/download

# Download
mkdir ${ETCD_DIR}
cd ${ETCD_DIR}
wget ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar -xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz

# install
cd etcd-${ETCD_VER}-linux-amd64
cp etcdctl /usr/local/bin/

2. etcdctl V3
使用etcdctlv3的版本时，需设置环境变量ETCDCTL_API=3。

export ETCDCTL_API=3

或者在`/etc/profile`文件中添加环境变量
vi /etc/profile
...
ETCDCTL_API=3
...
source /etc/profile

查看当前etcdctl的版本信息etcdctl version。

[root@k8s-dbg-master-1 etcd]# etcdctl version
etcdctl version: 3.3.4
API version: 3.3
1
2
3
更多命令帮助可以查询etcdctl —help。

[root@k8s-dbg-master-1 etcd]# etcdctl --help
NAME:
    etcdctl - A simple command line client for etcd3.

USAGE:
    etcdctl

VERSION:
    3.3.4

API VERSION:
    3.3


COMMANDS:
    get         Gets the key or a range of keys
    put         Puts the given key into the store
    del         Removes the specified key or range of keys [key, range_end)
    txn         Txn processes all the requests in one transaction
    compaction      Compacts the event history in etcd
    alarm disarm        Disarms all alarms
    alarm list      Lists all alarms
    defrag          Defragments the storage of the etcd members with given endpoints
    endpoint health     Checks the healthiness of endpoints specified in `--endpoints` flag
    endpoint status     Prints out the status of endpoints specified in `--endpoints` flag
    endpoint hashkv     Prints the KV history hash for each endpoint in --endpoints
    move-leader     Transfers leadership to another etcd cluster member.
    watch           Watches events stream on keys or prefixes
    version         Prints the version of etcdctl
    lease grant     Creates leases
    lease revoke        Revokes leases
    lease timetolive    Get lease information
    lease list      List all active leases
    lease keep-alive    Keeps leases alive (renew)
    member add      Adds a member into the cluster
    member remove       Removes a member from the cluster
    member update       Updates a member in the cluster
    member list     Lists all members in the cluster
    snapshot save       Stores an etcd node backend snapshot to a given file
    snapshot restore    Restores an etcd member snapshot to an etcd directory
    snapshot status     Gets backend snapshot status of a given file
    make-mirror     Makes a mirror at the destination etcd cluster
    migrate         Migrates keys in a v2 store to a mvcc store
    lock            Acquires a named lock
    elect           Observes and participates in leader election
    auth enable     Enables authentication
    auth disable        Disables authentication
    user add        Adds a new user
    user delete     Deletes a user
    user get        Gets detailed information of a user
    user list       Lists all users
    user passwd     Changes password of user
    user grant-role     Grants a role to a user
    user revoke-role    Revokes a role from a user
    role add        Adds a new role
    role delete     Deletes a role
    role get        Gets detailed information of a role
    role list       Lists all roles
    role grant-permission   Grants a key to a role
    role revoke-permission  Revokes a key from a role
    check perf      Check the performance of the etcd cluster
    help            Help about any command

OPTIONS:
      --cacert=""               verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""                 identify secure client using this TLS certificate file
      --command-timeout=5s          timeout for short running command (excluding dial timeout)
      --debug[=false]               enable client-side debug logging
      --dial-timeout=2s             dial timeout for client connections
  -d, --discovery-srv=""            domain name to query for SRV records describing cluster endpoints
      --endpoints=[127.0.0.1:2379]      gRPC endpoints
      --hex[=false]             print byte strings as hex encoded strings
      --insecure-discovery[=true]       accept insecure SRV records describing cluster endpoints
      --insecure-skip-tls-verify[=false]    skip server certificate verification
      --insecure-transport[=true]       disable transport security for client connections
      --keepalive-time=2s           keepalive time for client connections
      --keepalive-timeout=6s            keepalive timeout for client connections
      --key=""                  identify secure client using this TLS key file
      --user=""                 username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"          set the output format (fields, json, protobuf, simple, table)


3. etcdctl 常用命令
3.1. 指定etcd集群
HOST_1=10.240.0.17
HOST_2=10.240.0.18
HOST_3=10.240.0.19
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

etcdctl --endpoints=$ENDPOINTS member list

3.2. 增删改查
1、增

etcdctl --endpoints=$ENDPOINTS put foo "Hello World!"
1
2、查

etcdctl --endpoints=$ENDPOINTS get foo
etcdctl --endpoints=$ENDPOINTS --write-out="json" get foo
1
2
基于相同前缀查找

etcdctl --endpoints=$ENDPOINTS put web1 value1
etcdctl --endpoints=$ENDPOINTS put web2 value2
etcdctl --endpoints=$ENDPOINTS put web3 value3

etcdctl --endpoints=$ENDPOINTS get web --prefix

3、删

etcdctl --endpoints=$ENDPOINTS put key myvalue
etcdctl --endpoints=$ENDPOINTS del key

etcdctl --endpoints=$ENDPOINTS put k1 value1
etcdctl --endpoints=$ENDPOINTS put k2 value2
etcdctl --endpoints=$ENDPOINTS del k --prefix

3.3. 集群状态
集群状态主要是etcdctl endpoint status 和etcdctl endpoint health两条命令。

etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status

+------------------+------------------+---------+---------+-----------+-----------+------------+
|     ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+------------------+------------------+---------+---------+-----------+-----------+------------+
| 10.240.0.17:2379 | 4917a7ab173fabe7 | 3.0.0   | 45 kB   | true      |         4 |      16726 |
| 10.240.0.18:2379 | 59796ba9cd1bcd72 | 3.0.0   | 45 kB   | false     |         4 |      16726 |
| 10.240.0.19:2379 | 94df724b66343e6c | 3.0.0   | 45 kB   | false     |         4 |      16726 |
+------------------+------------------+---------+---------+-----------+-----------+------------+

etcdctl --endpoints=$ENDPOINTS endpoint health

10.240.0.17:2379 is healthy: successfully committed proposal: took = 3.345431ms
10.240.0.19:2379 is healthy: successfully committed proposal: took = 3.767967ms
10.240.0.18:2379 is healthy: successfully committed proposal: took = 4.025451ms

3.4. 集群成员
跟集群成员相关的命令如下：

    member add          Adds a member into the cluster
    member remove       Removes a member from the cluster
    member update       Updates a member in the cluster
    member list         Lists all members in the cluster

例如 etcdctl member list列出集群成员的命令。

etcdctl --endpoints=http://172.16.5.4:12379 member list -w table

+-----------------+---------+-------+------------------------+-----------------------------------------------+
|       ID        | STATUS  | NAME  |       PEER ADDRS       |                 CLIENT ADDRS                  |
+-----------------+---------+-------+------------------------+-----------------------------------------------+
| c856d92a82ba66a | started | etcd0 | http://172.16.5.4:2380 | http://172.16.5.4:2379,http://172.16.5.4:4001 |
+-----------------+---------+-------+------------------------+-----------------------------------------------+

文章参考：

https://coreos.com/etcd/docs/latest/demo.html
---------------------
作者：胡伟煌
来源：CSDN
原文：https://blog.csdn.net/huwh_/article/details/80225902
版权声明：本文为博主原创文章，转载请附上博文链接！