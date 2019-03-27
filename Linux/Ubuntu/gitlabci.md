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

    Run the following command:

     sudo gitlab-runner register

    Enter your GitLab instance URL:

     Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
     https://gitlab.com

    Enter the token you obtained to register the Runner:

     Please enter the gitlab-ci token for this runner
     xxx

    Enter a description for the Runner, you can change this later in GitLab’s UI:

     Please enter the gitlab-ci description for this runner
     [hostame] my-runner

    Enter the tags associated with the Runner, you can change this later in GitLab’s UI:

     Please enter the gitlab-ci tags for this runner (comma separated):
     my-tag,another-tag

    Enter the Runner executor:

     Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
     docker

    If you chose Docker as your executor, you’ll be asked for the default image to be used for projects that do not define one in .gitlab-ci.yml:

     Please enter the Docker image (eg. ruby:2.1):
     alpine:latest
