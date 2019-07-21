Dockerfile
```
FROM openjdk:8-jdk-alpine

WORKDIR /app

ADD target/dispatch-service-1.0.0.jar ./
```

docker build -t dispatch-service:0.0.1 .
docker run -it dispatch-service:0.0.1 sh
然后用sh命令运行
