# Ubuntu 下使用 Docker 安装 Nginx

## 1、安装 nginx 镜像

```shell
sudo docker search nginx
sudo docker pull nginx
#查看镜像信息 docker inspect 容积或镜像ID
sudo docker inspect 5a3221f0137b
#push 到harbor
sudo docker tag nginx:latest 192.168.0.245:18071/common/nginx:v1.17.3
sudo docker push 192.168.0.245:18071/common/nginx:v1.17.3
#删除nginx:latest  sudo docker image rm 镜像名/ID
sudo docker image rm nginx:latest
```

## 2、在主机中创建挂载目录

- 挂载 nginx 的静态文件目录 `mkdir /var/data/nginx/html`
- 挂载 nginx 的配置目录 `mkdir /etc/conf/nginx/conf.d`
- 挂载 nginx 的日志目录 `mkdir /var/data/nginx/log`

## 3、配置文件

创建nginx.conf  /etc/conf/nginx

```shell
user  root;
worker_processes  2;
 
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
 
 
events {
    worker_connections  10240;
}
 
 
http {
    # Tomcat服务器集群
    #upstream tomcat_servers {
    #    server 192.168.0.245:80;
    #    server 192.168.0.246:80;
    #}
    #将所有请求交给Tomcat集群去处理
    #location / {
    #    proxy_set_header Host $http_host;
    #    proxy_pass http://tomcat_servers;
    #}

    include  /etc/nginx/mime.types;

    default_type  application/octet-stream;
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log  /var/log/nginx/access.log  main;
 
    #sendfile  on;

    #tcp_nopush  on;
 
    keepalive_timeout  65;
 
    autoindex  on;
    
    gzip  on;
 
    client_max_body_size 100M;
 
    client_header_buffer_size  128k;

    large_client_header_buffers  4  128k;

    include  /etc/nginx/conf.d/*.conf;
}
```

将 default.conf 配置文件放在 /etc/conf/nginx/conf.d 下。

```shell
server {

    listen  80;

    server_name  localhost;

    charset koi8-r;

    access_log  /var/log/nginx/access.log  main;

    location / {
        root  /usr/share/nginx/html;
        index  index.html;
    }

    error_page  500 502 503 504  /50x.html;

    location = /50x.html {
        root  /usr/share/nginx/html;
    }
}
```

## 4、启动容器

将容器 80 端口映射到主机 80 端口。

```shell
docker run \
    --name docker-nginx \
    --restart always \
    -p 80:80 \
    -v /etc/conf/nginx/nginx.conf:/etc/nginx/nginx.conf \
    -v /etc/conf/nginx/conf.d:/etc/nginx/conf.d \
    -v /var/data/nginx/html:/usr/share/nginx/html \
    -v /var/data/nginx/log:/var/log/nginx \
    192.168.0.245:18071/common/nginx:v1.17.3
```

