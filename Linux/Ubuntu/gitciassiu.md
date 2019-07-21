1. cannot validate certificate for x.x.x.x because it doesn't contain any ip sans
报错信息：error: registering runner... failed runner=xxxxxxx status=couldn't execute post against https://x.x.x.x/api/v4/runners: post https://x.x.x.x/api/v4/runners: x509: cannot validate certificate for x.x.x.x because it doesn't contain any ip sans

原因：gitlab使用自签名证书时，注册时需要使用对应的ca根证书验证。

解决方案：注册时，使用"--tls-ca-file"参数，指定自签名的ca根证书。

2. certificate signed by unknown authority
报错信息：post https://x.x.x.x/api/v4/runners: x509: certificate signed by unknown authority

原因：注册runner时，如果设置了"--tag-list"，则"--run-untagged"默认为"false"，同时间.gitlab-ci.yml中的job未指定tag触发此报错。

解决方案：注册时，"--run-untagged"参数设置为"true"；或者在已注册的runner中修改勾选" indicates whether this runner can pick jobs without tags"；或者.gitlab-ci.yml中的job指定tag。

3. peer's certificate issuer is not recognized.
报错信息：fatal: unable to access 'https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab.x.com/root/cmop.git/': peer's certificate issuer is not recognized.



原因：gitlab-runner拉取代码时，使用https协议访问gitlab，需要验证。

解决方案：

# 参考：https://www.jianshu.com/p/fa71d97dcde0
# 因runner运行时的执行者是gitlab-runner账户，需要在gitlab-runner账号下设置访问https类网站时，免验证
[root@gitlab-runner ~]# su - gitlab-runner
[gitlab-runner@gitlab-runner ~]$ git config --global http."sslverify" false

# 查看
[gitlab-runner@gitlab-runner ~]$ cat /home/gitlab-runner/.gitconfig
[http]
    sslverify = false
4. dial unix /var/run/docker.sock: connect: permission denied
报错信息：got permission denied while trying to connect to the docker daemon socket at unix:///var/run/docker.sock: get http://%2fvar%2frun%2fdocker.sock/v1.27/info: dial unix /var/run/docker.sock: connect: permission denied

原因：gitlab-runner账号权限不足，不能访问/var/run/docker.sock。

解决方案：

# 将gitlab-runner用户加入docker组
[root@gitlab-runner ~]# usermod -ag docker gitlab-runner

# 查看
[root@gitlab-runner ~]# groups gitlab-runner
5. couldn't resolve host 'gitlab.x.com'
报错信息：fatal: unable to access 'https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab.cmop.chinamcloud.com/root/cmop.git/': couldn't resolve host 'gitlab.x.com'

原因：executor = "docker"时，执行环境是1个容器，由于验证用的gitlab域名不能被dns解析，导致无法连接。

解决方案：

在注册时使用"--docker-volumes /etc/hosts:/etc/hosts"，将运行gitlab-runner服务主机的hosts文件映射到执行容器内；
注册时还可使用参数"--clone-url 地址覆盖域名，执行容器使用ip地址直接访问gitlab。参考：
ps：使用ip覆盖域名时，可能会带来其他问题，如果使用的是自签名的证书，需要明确ip地址是否也被自签名的ca机构认证。

6. ssl certificate problem: unable to get local issuer certificate
报错信息：fatal: unable to access 'https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@100.64.135.200/root/cmop.git/': ssl certificate problem: unable to get local issuer certificate



原因：注册时，为使执行容器可访问不能被dns解析的gitlab域名，使用了参数"--clone-url https://x.x.x.x"覆盖了原域名，但ca机构（自签名的ca证书）只对域名做了认证，导致使用ip访问时不能认证。

解决方案：注册时，将运行gitlab-runner服务主机的hosts映射到执行容器内，使其可通过被ca机构认证的域名访问gitlab，而非ip地址

7.
*** WARNING: Service runner-118b6b82-project-10-concurrent-0-docker-0 probably didn't start properly.

Health check error:
ContainerStart: Error response from daemon: Cannot link to a non running container: /runner-118b6b82-project-10-concurrent-0-docker-0 AS /runner-118b6b82-project-10-concurrent-0-docker-0-wait-for-service/service

Service container logs:
2018-09-10T06:02:08.161544784Z mount: permission denied (are you root?)
2018-09-10T06:02:08.161686312Z Could not mount /sys/kernel/security.
2018-09-10T06:02:08.161695928Z AppArmor detection and --privileged mode might break.
2018-09-10T06:02:08.164169464Z mount: permission denied (are you root?)

*********

出现的原因在于，在你的宿主机使用root用户安装docker（执行docker命令需要加上sudo），这种情况下，需要配置runner的privileged为true。
gitlab-runner的配置文件在宿主机上的默认位置为/srv/gitlab-runner/config/config.toml，直接修改该文件中的配置项即可。
