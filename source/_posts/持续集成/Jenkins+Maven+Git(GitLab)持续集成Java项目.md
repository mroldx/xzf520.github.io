---
title: Jenkins+Maven+Git(GitLab)持续集成Java项目
tags: [CI/CD]
date: 2020-05-01 14:59:51
permalink:
categories: 持续集成
description: CI/CD持续集成  Jenkins+Maven+Git(GitLab)持续集成Java项目
image:
---

<p class="description" ></p>

<!-- more -->
# Jenkins+Maven+Git(GitLab)持续集成Java项目

## 一、前言

### 1、本文主要内容

- Jenkins+SSH获取Gitlab代码
- Jenkins+Maven构建Java(Spring Boot)项目配置
- Jenkins发布Spring Boot项目：远程服务器端配置
- Jenkins发布Spring Boot项目：pom.xml编译配置
- Jenkins+SSH将构建输出结果发布到远程服务器并启动应用

### 2、环境信息

- 服务器

| 服务器名 | 操作系统 | IP             | 说明             |
| :------- | :------- | :------------- | :--------------- |
| GitLab   | CentOS 7 | 192.168.56.160 | 部署GitLab社区版 |
| Jenkins  | CentOS 7 | 192.168.56.150 | 部署Jenkins      |
| Server   | CentOS 7 | 192.168.88.155 | 部署Java项目     |

- 软件

| 工具/环境 | 版本             |
| :-------- | :--------------- |
| Jenkins   | 2.176.2          |
| Maven     | 3.6.1            |
| GitLab    | GitLab CE 12.1.2 |
| JDK       | 1.8.0_181        |

### 3、基础准备

- GitLab部署

参考：https://ken.io/note/centos7-gitlab-install-tutorial

配置GitLab访问地址为：`http://192.168.56.160`

- Jenkins部署

参考：https://ken.io/note/centos7-jenkins-install-tutorial

## 二、 Java应用部署服务器

### 1、部署JDK8

参考：https://ken.io/note/centos-java-setup

将`jdk1.8.0_181`部署在目录`/usr/java/`

部署完成后，jdk的根目录就是：`/usr/java/jdk1.8.0_181/`

### 2、开放端口

```js
#开放1000到9999的端口
sudo firewall-cmd --add-port=1000-9999/tcp --permanent
sudo firewall-cmd --reload
```

### 3、创建应用部署目录

```js
#创建目录
sudo mkdir -p /webroot

#授权
sudo chown -R app:app /webroot
```

## 三、Jenkins环境准备

在配置构建任务之前，我们需要在Jenkins服务器配置Maven、Git环境

### 1、Maven安装

- 下载&解压

```js
cd /home/downloads

#下载
sudo wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz

#解压到指定目录
sudo mkdir -p /usr/maven
sudo tar -zvxf apache-maven-3.6.1-bin.tar.gz -C /usr/maven
```

- 配置环境变量

```js
#修改/etc/profile
sudo vi /etc/profile

#在文件末尾写入以下内容
export MAVEN_HOME=/usr/maven/apache-maven-3.6.1
export PATH=$MAVEN_HOME/bin:$PATH

#使更改生效
source /etc/profile

#测试
mvn -version
```

- 配置Maven仓库

为了保证jar包的下载速度，修改maven配置使用国内镜像

```js
#进入Maven根目录
cd $MAVEN_HOME

#备份配置文件
sudo mv conf/settings.xml conf/settings.xml.bak

#新建配置文件
sudo vi settings.xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <mirrors>

    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/public/</url>
      <mirrorOf>*</mirrorOf>        
    </mirror>

    <mirror>
      <id>nexus-163</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus 163</name>
      <url>http://mirrors.163.com/maven/repository/maven-public/</url>
    </mirror>

    <mirror>
        <id>central</id>
        <name>Maven Repository Switchboard</name>
        <url>http://repo1.maven.org/maven2/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>

  </mirrors>

</settings>
```

### 2、Git Client安装

- 安装

```js
sudo yum install -y git
```

- 密钥准备

```js
#生成密钥
ssh-keygen -t rsa
```

- 将公钥添加到GitLab

```js
#查看公钥
cat ~/.ssh/id_rsa.pub
```

访问GitLab：`http://192.168.56.150:/profile/keys`，添加公钥

- 添加Git SSH凭据

后面配置Jenkins构建任务代码仓库时需要用到

```js
#查询SSH私钥
cat ~/.ssh/id_rsa
```

访问:`/credentials/store/system/domain/_/newCredentials` 直接进入凭据添加界面

类型选择：`SSH Username with private key`

![img](https://ask.qcloudimg.com/http-save/1381082/w39peslr9p.png?imageView2/2/w/1620)jenkins-credentials-gitlab-key

### 3、插件安装

- 插件列表

| 插件名                                                       | 版本   | 说明                           |
| :----------------------------------------------------------- | :----- | :----------------------------- |
| [Git](https://ken.io/note/centos7-gitlab-install-tutorial)   | 3.11.0 | 使用Git访问远程仓库            |
| [Maven Integration](https://ken.io/note/centos7-gitlab-install-tutorial) | 3.3    | 使用Maven进行编译等            |
| [Publish Over SSH ](https://ken.io/note/centos7-gitlab-install-tutorial) | 1.20.1 | 用于将编译结果发布到远程服务器 |

- 安装说明

访问：

```
http://192.168.56.150:8080/pluginManager/available
```

`Ctrl+F`搜索插件名，勾选后，进行安装

安装完成后，重启jenkins

```js
sudo systemctl restart jenkins
```

### 4、Jenkins插件/环境配置

在菜单：系统管理->全局工具配置中对插件相关工具进行配置

![img](https://img2018.cnblogs.com/blog/1733080/201909/1733080-20190930185843004-1191682955.png)

- Publish over SSH

然后在菜单：系统管理->系统设置对Publish over SSH进行设置

![image-20200501143558976](C:\Users\Old丶x\AppData\Roaming\Typora\typora-user-images\image-20200501143558976.png)

主要配置项说明：

| 配置项              | 说明                                        |
| :------------------ | :------------------------------------------ |
| Name                | 服务器名，随便写，方便记忆即可              |
| Hostname            | 服务器IP，或者可以被正常解析的服务器名/域名 |
| Username            | 用于登录的账号                              |
| Remote Dictionary   | 远程目录，绝对路径                          |
| Passphrase/Password | 密码                                        |
| Port                | SSH端口                                     |

配置完成后可以点击`Test Configuration`进行连接测试  

## 四、Jenkins构建任务

### 1、示例项目准备

> 如果已经有现成项目可忽略此步骤

访问：`http://192.168.56.150/projects/new`创建项目:`helloworld`

- 创建SpringBoot应用

参考：https://ken.io/note/springboot-course-basic-helloworld 创建SpringBoot应用

| 参数       | 值                     |
| :--------- | :--------------------- |
| Maven模板  | maven-archetype-webapp |
| GroupId    | io.ken.tutorial        |
| ArtifactId | helloworld             |
| Version    | 1.0                    |

- 配置编译选项

修改pom.xml，以满足编译要求

```js
<build>
    <finalName>helloworld</finalName>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>false</filtering>
      </resource>
    </resources>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

配置application.yml，配置应用端口为8081

```js
server:
  port: 8081
```

项目准备完成后，在GitLab账号`ken`下面创建项目`helloworld`并将刚才创建的文件提交上去

> 示例代码：https://gitlab.com/ken-io/springboot-helloworld

### 2、Jenkins任务创建


![img](https://ask.qcloudimg.com/http-save/1381082/smueltu4o1.png?imageView2/2/w/1620)jenkins-create-job-maven-springboot

选择：`构建一个maven项目`，然后确定即可

### 3、配置源代码管理

![img](https://ask.qcloudimg.com/http-save/1381082/r7v5evbqyb.png?imageView2/2/w/1620)jenkins-create-job-maven-springboot-sourcecode-git-ssh

这里我们选择Git，并配置SSH Git地址，选择之前创建好的凭据/密钥

### 4、Publish over SSH配置

![img](https://ask.qcloudimg.com/http-save/1381082/c8v0o8tfko.png?imageView2/2/w/1620)jenkins-create-job-maven-springboot-buildenv-publishoverssh

配置项说明：

| 配置项           | 值                                                         | 说明                                                         |
| :--------------- | :--------------------------------------------------------- | :----------------------------------------------------------- |
| Name             | appserver                                                  | SSH Server名称，根据之前配置选择即可                         |
| Source files     | target/*.jar                                               | 需要传输的文件，支持通配符，编译文件默认都在项目根目录下的target目录中 |
| Remove prefix    | target                                                     | 移除匹配到的文件路径的前缀，如果留空，会在远程服务器上创建对应的目录 |
| Remote directory | helloworld/                                                | 远程服务器上的项目目录，该目录会被创建在Publish over SSH配置的远程根目录中(/webroot) |
| Exec command     | [---](https://ken.io/note/centos7-gitlab-install-tutorial) | 文件传输到远程服务器后执行的命令                             |

命令示例：

```js
APP_NAME=helloworld.jar
cd /webroot/helloworld
mkdir -p logs

#找到包含AppName的进程
PROCESS=`ps -ef|grep $APP_NAME|grep -v grep  |awk '{ print $2}'`
#循环停用进程直到成功
while :
do
  kill -9 $PROCESS > /dev/null 2>&1
  if [ $? -ne 0 ];then
   break
  else
   continue
fi
done
echo 'Stop Successed'

#启动应用
nohup  /usr/java/jdk1.8.0_181/bin/java -jar $APP_NAME >>logs/start.log 2>>logs/startError.log &

#sleep等待15秒后，判断包含AppName的线程是否存在
sleep 15
if test $(pgrep -f $APP_NAME|wc -l) -eq 0
then
   echo "Start Failed"
else
   echo "Start Successed"
fi
```

### 5、构建

点击立即构建即可进行项目构建，构建完成后，构建记录的图标会根据构建结果不同显示成不同颜色。  

蓝色、黄色、红色分别表示：成功、未完成、失败

如果构建并没有成功，可以点击构建记录，在后在`控制台输出`中查看构建记录



## 五 ，视频演示成果



