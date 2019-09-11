Dockerfile
```
FROM openjdk:8-jdk-alpine

WORKDIR /app

ADD target/dispatch-service-1.0.0.jar ./
```

docker build -t dispatch-service:0.0.1 .
docker run -it dispatch-service:0.0.1 sh
然后用sh命令运行


docker-maven-plugin:1.2.0:build (default) @ shelf-service ---
[INFO] Using authentication suppliers: [ConfigFileRegistryAuthSupplier, FixedRegistryAuthSupplier]
[INFO] Copying /builds/spkitty/shelf-service/target/shelf-service-0.0.1-SNAPSHOT.jar -> /builds/spkitty/shelf-service/target/docker/shelf-service-0.0.1-SNAPSHOT.jar
[INFO] Building image registry-vpc.cn-beijing.aliyuncs.com/mshoufu/shelf-service:e617f65e
Step 1/9 : FROM freemanliu/openjre:1.8.0_181_font

Step 2/9 : ENV APP_ARGS ''

Step 3/9 : ENV JAVA_OPTS '-Xms1G -Xmx1G'

Step 4/9 : ENV TZ Asia/Shanghai

Step 5/9 : WORKDIR /app

Step 6/9 : ADD shelf-service-0.0.1-SNAPSHOT.jar .

Step 7/9 : EXPOSE 8080

Removing intermediate container 64ee5c9849e5
Step 8/9 : CMD java -XX:+UseG1GC             -XX:+AggressiveOpts             -XX:+UseFastAccessorMethods             -XX:+UseStringDeduplication             -XX:+UseCompressedOops             -XX:+OptimizeStringConcat             -XX:CICompilerCount=8 $JAVA_OPTS -jar shelf-service-0.0.1-SNAPSHOT.jar $APP_ARGS

Removing intermediate container 5c31582ec2a0
Step 9/9 : VOLUME /data/logs
