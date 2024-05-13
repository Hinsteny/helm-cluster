# ccr.ccs.tencentyun.com/doctorwork/skywalking-agent:latest是包含jre以及skywalking相关jar包基础容器
FROM openjdk:11
LABEL maintainer="Hinsteny.Hisoka"

# 容器的工作目录统一/app
WORKDIR /app
EXPOSE 20883/tcp

ENV seata.address=seata-server
ENV registry.address=zookeeper://zookeeper-service:2181
ENV mysql.address=stock-mysql-db-service

# 复制应用压缩文件
COPY app.jar /app

# 容器启动后执行的脚本
ENTRYPOINT ["java", "-jar", "app.jar", "-Djava.net.preferIPv4Stack=true"]