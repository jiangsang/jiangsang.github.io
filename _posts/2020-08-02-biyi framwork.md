---
clayout: post
title: 比翼框架学习笔记一
categories:
 - 笔记
tags:
 - 比翼
---
今天开始学习公司开发平台：比翼开发平台，该平台是公司一个功能全面的研发平台，包括代码托管、开发框架、协同研发、微服务等能力，目前我需要学习接触的主要是开发框架的使用和了解。这里对自己的学习踩坑进行记录。
对于这个开发平台，首先就是需要在自己的电脑上把平台搭建起来，开发平台本身是一个前后端分离的项目，依次部署和配置好即可进行使用，为自己的开发赋能，提高开发效率，可以有效减少简单的前期配置和准备时间。

<!-- more -->

> 为了防止被攻击和其他问题，公司项目相关地址已移除

### 安装使用环境要求
---
- Git版本控制
- JDK 8及以上
- Maven 3及以上
- Mysql数据库——其他数据库也可，这个是默认的
- NVM（可选）——NodeJS的版本管理工具
- NodeJS 8——运行在服务端的 JavaScript，注意版本为8
- NPM——随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题

### 配置部署后端项目udp6
---
项目地址：  [http://xxx/udp_public/udp6.git](http://xxx/udp_public/udp6.git).   
> ⚠️项目放在公司内部自己搭建的gitlab上，需要使用内网访问，并且有很多依赖包maven的默认配置是下不了的。需要配置maven的setting.xml文件（maven目录->conf->setting.xml），同时也替换掉C:\users\\{用户名}\\.m2下的。

setting.xml ：注意修改本地仓库地址为自己的  

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
<!--  配置本地仓库存储地址 -->
<localRepository>D:\apache-maven-3.6.3\res</localRepository>
<!--  配置central镜像地址为阿里仓库 -->
<mirrors>
<mirror>
<id>alimaven</id>
<name>aliyun maven</name>
<url>
http://maven.aliyun.com/nexus/content/groups/public/
</url>
<mirrorOf>central</mirrorOf>
</mirror>
</mirrors>
<profiles>
<profile>
<id>nexus-public</id>
<repositories>
<repository>
<id>nexus-public</id>
<name>local private nexus release</name>
<url>
http://xxx/repository/maven-public/
</url>
</repository>
</repositories>
<pluginRepositories>
<pluginRepository>
<id>nexus-public</id>
<name>Team Nexus Repository</name>
<url>
http://xxx/repository/maven-public/
</url>
</pluginRepository>
</pluginRepositories>
</profile>
</profiles>
<!--  servers节点的属性是在向仓库发布时使用  -->
<servers></servers>
<!--  激活配置  -->
<activeProfiles>
<activeProfile>nexus-public</activeProfile>
</activeProfiles>
</settings>
```

接着打开IDE，我使用IDEA，导入项目后，修改一下项目的pom.xml文件，将其中的仓库地址改为： 

```xml
<repositories>
  <repository>
    <id>maven-public</id>
    <name>maven-public</name>
    <url>http://xxx/repository/maven-group/</url>
  </repository>
</repositories>
```

此时Maven会自动下载依赖，但是我的依赖包一直没下好，报错，最后发现是网络原因，我坐的位置太偏了，网络很慢。  还有一点就是仓库地址链接不是很稳定，多尝试几次就好。

依赖下好后，修改项目配置文件。  

1. src->main->resources->config->application-dev.yml:先新建一个新的数据库，我这里创建了一个名为biyi_base的空数据库。  

![](http://jianger-upic.test.upcdn.net/uPic/%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8B%E5%8D%882.42.52.png)
    
2. src->main->resources->config->generatorConfig.xml: 

```xml
<!--修改为自己的-->
<jdbcConnection driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/biyi_base?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false"
                userId="root"
                password="123456">
</jdbcConnection>
```

确认mysql服务已打开，此时直接点击运行，项目即可运行成功。后端展示出的页面是接口文档。

#### 重要配置的理解说明

1. src->main->resources->db->changeling->xxx.xml:数据库各种表结构和数据的初始化
2. src->main->java->com.ctsi.config->KaptchaConfig.java:可以自定义图片验证码的长度和各种样式
3. src->main->java->com.ctsi.config->RedisConfiguration.java:
4. src->main->java->com.ctsi.config->SecurityConfiguration.java:
5. src->main->java->com.ctsi.config->SpringMvcConfiguration.java:
6. src->main->java->com.ctsi.config->MethodSecurityConfiguration:

### 配置部署前端项目udp6_web
---

项目地址：  [http://xxx/udp_public/udp6_web.git](http://xxx/udp_public/udp6_web.git)

首先安装好NodeJS 8，在当前目录打开终端，依次执行以下命令:  

```bash
npm install -g nrm	#安装nrm
nrm add ctsi http://xxx/repository/npmall/	#添加仓库地址
nrm use ctsi	#切换集成npm源
npm install #下载项目所需依赖，时间较长

#最后无报错的情况下运行启动项目
npm run dev

```

最后前后端项目都部署启动成功的情况下，输入账号密码和验证码即可进入开发平台页面。

![](http://jianger-upic.test.upcdn.net/uPic/%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8B%E5%8D%883.32.48.png)

初始化`账号`：admin		`密码：`ctsi@123



`更新`：自己写了一个比翼框架的搭建和使用指南，有需要的可以给我留言

### 遇到的问题

1. idea 启动后端项目时控制台报错：源发行版 8 需要目标发行版 1.8
    - Project Structure里确认两个地方:Project sdk以及project language level
    - Project Structure->Modules里Sources里的Language level
    - settings->java Compiler ->target bytecode Version
    - 以上三处版本保持一

​    

### 发现的bug
- 上传头像时，当所裁剪的图片大小超过指定值时，无论点击`取消`还是`确定`按钮弹出框都无法关闭。  
- 登录日志：无论是否登录成功，只要发生了登录操作的请求，登录日志里都会显示登录成功。

