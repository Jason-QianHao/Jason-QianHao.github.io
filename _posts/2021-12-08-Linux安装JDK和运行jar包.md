---
title: Linux安装JDK和运行jar包
tags: 操作系统OS
---

# Linux安装JDK

1. 查看有无系统自带jdk 

   ```shell
   rpm -qa |grep java
   ```

   如果有可以进行批量卸载

   ```shell
   rpm -qa | grep java | xargs rpm -e --nodeps 
   ```

2. 查询yum可用的jdk版本

   ```shell
   yum list java*
   ```

   安装jkd1.8

   ```shell
   yum install java-1.8.0-openjdk* -y
   ```

3. 验证是否安装成功

   ```shell
   java -version
   ```

4. 查看当前java版本和安装位置

   ```shell
   alternatives --config java
   ```

5. 设置环境变量

   ```shell
   vim /etc/profile
   
   export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   export PATH=$JAVA_HOME/bin:$PATH
   
   . /etc/profile
   ```


# Linux后台运行Jar包

1. 查看防火墙是否开放了jar包运行端口

   ```shell
   firewall-cmd --zone=public --list-ports—查看开放端口
   firewall-cmd --zone=public --add-port=8081/tcp --permanent--开放
   firewall-cmd --zone=public --remove-port=8081/tcp --permanent--关闭
   firewall-cmd --reload --刷新配置
   systemctl stop firewalld.service  --关闭防火墙(安全隐患)
   firewall-cmd --state --查看防火墙状态
   ```

   **开放端口后，一定要使用`firewall-cmd --reload`命令刷新配置才能生效**

2. 查看当前java进程，如果之前启动过可以先关闭进程

   ```shell
   ps -ef|grep java #查看java相关进程
   kill -9 PID #关闭Jar包进程
   ```

3. 后台启动Jar包

   ```shell
   nohup java -jar jarName-0.0.1-SNAPSHOT.jar >msg.log 2>&1 &
   ```

   

