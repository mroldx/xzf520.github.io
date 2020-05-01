---
title: CentOS 7 下安装部署 Jenkins教程
tags: [CI/CD]
date: 2020-02-01 13:40:20
permalink:
categories: 持续集成
description:  CentOS 7 下安装部署  Jenkins教程
image:
---

<p class="description" ></p>

<!-- more -->


## 一、前言

![img](img/2.jpg)

### 1、 Jenkins是什么？

Jenkins是一个开源的支持自动化构建、部署等任务的平台。基本上可以说是持续集成（CI）、持续发布（CD）不可或缺的工具。

官网：https://jenkins.io/

### 2、本篇环境信息

| 工具/环境    | 版本      |
| ------------ | --------- |
| Linux Server | CentOS 7  |
| Jenkins      | 2.121.2   |
| JDK          | 1.8.0_181 |

### 3、准备工作安装JDK

#### 1、下载

```
#JDK下载首页
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
#JDK8历史版本
https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html
#（1）下载之后FTP/SFTP到服务器
#（2）获取到下载链接后，用wget命令下载
```

#### 2、解压到指定目录

```
sudo mkdir -p /usr/java 
sudo tar zvxf jdk-8u131-linux-x64.tar.gz -C /usr/java
```

#### 3、配置环境变量

```
vi /etc/profile
# 在export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL下添加

export JAVA_HOME=/usr/java/jdk1.8.0_131
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

#### 4、使环境变量生效

```
source /etc/profile
```

#### 5、检查是否配置成功

```
java -version
```

## 二、Jenkins安装

### 1、Yum安装

- yum源导入

```bash
#添加Yum源
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

#导入密钥
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

- 安装

```bash
sudo yum install -y jenkins
```

### 2、开放端口

Jenkins站点的默认监听端口是8080

```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

### 3、配置Java可选路径

因为Jenkins默认的java可选路径不包含我们部署的jdk路径，所以这里要配置一下，不然Jenkins服务会启动失败

```bash
#修改jenkins启动脚本
sudo vi /etc/init.d/jenkins

#修改candidates增加java可选路径：/usr/java/jdk1.8.0_181/bin/java
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/bin/java
/usr/java/jdk1.8.0_181/bin/java
"
```

### 4、启动Jenkins并设置Jenkins开机启动

```bash
#重载服务（由于前面修改了Jenkins启动脚本）
sudo systemctl daemon-reload

#启动Jenkins服务
sudo systemctl start jenkins

#将Jenkins服务设置为开机启动
#由于Jenkins不是Native Service，所以需要用chkconfig命令而不是systemctl命令
sudo /sbin/chkconfig jenkins on
```

浏览器输入 `http://你虚拟机的ip:8080` 访问Jenkins



### 5,第一次启动需要输入密码

密码默认在/root/.jenkins/ssecrets/initiaAdminPassword

### 6、插件安装修改默认插件源

**注：（vi /root/.jenkins/hudson.model.UpdateCenter.xml）**

打开 hudson.model.UpdateCenter.xml

把 http://updates.jenkins-ci.org/update-center.json

改成 :（以下其中一个）

http://mirror.xmission.com/jenkins/updates/update-center.json

https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

http://mirror.esuni.jp/jenkins/updates/update-center.json

，还是不行的话找到 updates 目录下的 default.json 把里面所有的谷歌地址改成百度的，即将 http://www.google.com/ 替换为 http://www.baidu.com/。输入密码后出现以下界面说明成功。

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930111616624-1110674914.png)

，还是不行的话找到 updates 目录下的 default.json 把里面所有的谷歌地址改成百度的，即将 http://www.google.com/ 替换为 http://www.baidu.com/。输入密码后出现以下界面说明成功。

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930110700695-1135323904.png)

配置完成出现以下界面说明url配置完成！

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930111616624-1110674914.png)



###  3.推荐插件安装

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930111842790-1822503375.png)



### 4.创建用户

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930115339754-873101586.png)



###  5.实例配置

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930115427702-1648350448.png)



###  6.开始使用

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930115513633-1327651068.png)



###  7.进入首页

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930115541147-1354444469.png)

 　至此、Jenkins的下载安装，完成