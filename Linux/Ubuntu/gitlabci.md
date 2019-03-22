安装runner
1,添加GitLab的官方存储库
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```
2,安装指定版本的runner
```
apt-cache madison gitlab-runner
sudo apt-get install gitlab-runner=10.0.0
```
3,注册gitlab-runner
