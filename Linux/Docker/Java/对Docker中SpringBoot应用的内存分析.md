我有一个运行嵌入式Tomcat服务器的Spring Boot应用程序，非常简单...
它运行在一个Docker容器中。 若我用$ docker run运行镜像（没有内存限制），它的资源占用情况是这样：

docker stats

一个相当简单的spring boot应用的容器就占了677MB？ are you kidding me？开始挖掘真相吧...

首先，我需要看到容器中实际运行的进程。

docker exec my-app top -m

 有一个Gradle进程正在运行，它报告的RSS（ Resident Set Size ）与我们的Spring Boot应用程序一样大。 看看Dockerfile，很容易看出原因
 ```dockerfile
 FROM anapsix/alpine-java:8_jdk

# Create app directory
RUN mkdir -p /app  
WORKDIR /app

# Bundle app source
COPY . /app

# Build the solution (using the gradle task)
RUN ./gradlew build

EXPOSE 8080

CMD ["./gradlew", "bootRun"]
 ```
 1 第一次优化
所以，第一次内存优化的目的很简单 - 干掉Gradle过程！ 我只需要从使用Gradle任务引导应用程序改为直接使用Java执行.jar文件。
CMD ["java", "-jar", "build/libs/{app name}.jar"]
现在让我们看看容器中的top
即使我对Spring和Java runtime不甚了解，我仍然认为我们可以做得更好。 一个没有流量的简单API居然要382MB内存，...我们肯定错过了什么。
使用-Xmx56m指定运行时的 heap size limit（堆栈大小） 。 我想每个Java开发人员都知道这一点。 将这个参数添加到我们的$ java -jar命令将会限制Java堆栈大小为56MB。 运行时将尝试通过运行垃圾回收来保持它低于这个数字。要设置堆栈大小，我们可以在JAVA_OPTS环境变量设置Xmx参数：
docker run -e "JAVA_OPTS=-Xmx52m" app-image