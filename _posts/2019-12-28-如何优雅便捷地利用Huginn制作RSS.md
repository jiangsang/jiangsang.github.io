---
layout: post
title: 如何优雅快速地利用Huginn制作专属RSS
categories:
 - 好玩的
tags:
 - RSS
 - Docker
 - Huginn

---

###  前言

本篇博文接[利用Feed43为网站自制RSS源](https://jianger.space/%E5%88%A9%E7%94%A8Feed43%E4%B8%BA%E7%BD%91%E7%AB%99%E8%87%AA%E5%88%B6RSS%E6%BA%90/)一文，上一篇讲解了RSS的简介以及利用Feed43自制专属RSS，Feed43有其优势，缺陷也很明显，不能高度自定义、有的网站无法使用。那么此时，一个更为牛X的工具出场了，它就是Huginn。本篇就带你一探Huginn，并从安装到制作全程详细讲解制作专属RSS。

<!-- more -->

### 前排提示

> Huginn需要安装在云主机/云服务器上使用，如果你还没有，赶快购置一台吧
>
> 需要了解：<u>CSS基础；Linux基础命令；利用浏览器开发者工具找到对应内容代码</u>
>
> 我的使用环境：腾讯云主机centos7.5



### Huginn为何物

[Huginn](https://github.com/huginn/huginn) 是一个创建代理的系统，它为你在线执行自动化的任务。它主要的用途就是发送HTTP请求获得相关数据，然后你可以选择进行如下处理：

- 将获得的数据进行相应的格式处理输出，例如RSS；
- 将获得的数据通过第三方接口（huginn官方支持不少）进行发出，例如发邮件；
- 将获得的数据作为相关参数用作另一事件event的参数，例如将获得的天气信息作为Email Agent的内容。

不难发现，通过以上三方面的组合操作，Huginn可玩性颇高，它不仅仅可以用来制作RSS源，是一款自动化的效率利器，你可以把它看作是自己服务器上的 IFTTT 或 Zapier （注：两款皆是自动化工具）的破解版本。解锁更多姿势，请参阅[官方文档](https://github.com/huginn/huginn) 。



### Huginn的安装

---

官方提供了很多种安装方式，我采用了Docker的安装方式，Docker是一个轻量的实现虚拟化的利器，不用一个一个安装需要的应用，个人用足以，使用方法不详述了，贴一个学习地址[Docker教程](http://www.baidu.com/link?url=sGzkYAT0i-FJpZobSUTkUzYlXWlmLEg82ZchlUAiznx1JcFsoELMKKrlXRsuiXyGIoMuOgpKazt7sEtfky1ZL_&wd=&eqid=eed6dca6000ef600000000065e0759d5)，本文也用不上太多，会用就好。下面开始安装，说明一下我的云服务器为腾讯云Centos 7.5，以下操作均基于此系统。

1. 使用连接客户端连接腾讯云，并输入如下命令回车

   ```
   yum install docker -y
   ```

2. 不一会儿，若看到`Complete!`就安装完成啦，接着启动docker服务，输入如下命令并回车

   ```
   systemctl start docker
   ```

3. 安装容器前先替换成阿里云镜像，官方源太慢了！

   ```
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://iikujkej.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

4. 安装运行测试容器，输入如下命令并回车，如果80端口已占请改用其他端口

   ```
   docker run -d -p 80:80 httpd
   ```

5. 安装成功后在浏览器输入http://你的服务器IP，如果页面显示`It Works`，说明docker运行起来了,可以继续下一步

6. 虽然安装Huginn时附带有一个轻量的数据库，但Huginn一删里面的数据就没了，所以接下来安装通过docker安装MySQL，直接连接外部的MySQL也可以，不过麻烦些

   ```
   docker pull mysql:5.6
   ```

   我这里指定了MySQL版本为5.6，未指定的话，默认是最新的，但是最新版容易出问题，so.

7. 完成后创建并启动一个MySQL容器，MYSQL_ROOT_PASSWORD就是密码，可以自行设置

   ```
   docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:5.6
   ```

8. 接着进入MySQL容器修改权限

   ```
   docker exec -it mysql bash
   mysql -u root -p
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
   flush privileges;
   ```

9. 终于要安装Huginn了！输入如下命令，HUGINN_DATABASE_PASSWORD就是上面设置的密码，MYSQL_PORT_3306_TCP_ADDR对应docker的本地IP，你可以使用`ifconfig`命令查看

   ```
   docker pull huginn/huginn 
   
   docker run  --name huginn -p 3000:3000 -e MYSQL_PORT_3306_TCP_ADDR=172.18.0.1 -e HUGINN_DATABASE_NAME=huginn -e HUGINN_DATABASE_USERNAME=root -e HUGINN_DATABASE_PASSWORD=123456 huginn/huginn
   ```

10. 最后在浏览器中输入http://你的服务器IP:3000并访问，出现如下页面即安装成功！

    ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn.PNG)

    点击Login登录，初始账号：admin，密码：password，登录进去可自行修改。





### 开始定制专属RSS

------

> 以下以制作电影天堂的[最新电影](https://www.dytt8.net/html/gndy/dyzz/index.html)信息RSS为例

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn1.PNG)



#### 新建第一个Agent，获得标题和对应链接

------

初始化已经有一些Agents，你可以从里中学习到一些使用方法。点击<kbd>+ New Agent</kbd>添加第一个Agent，Type选择Website Agent。

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn2.PNG)



![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn3.PNG)

Name框输入名称，Schedule下拉框选择执行的间隔时间，其他默认即可

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn4.PNG)

Options参数最为关键，右侧都有英文说明的，字段简要说明如下：

> url:网址链接
>
> type:返回的数据类型，支持xml,、html、json、text，此处填写html
>
> mode:抓取模式，可选`all`, `on_change`,  `merge`，这里填写on_change，表示页面有变化才会抓取
>
> extra:表示抓取规则，
>
> - url和title表示抓取字段的名称，可随意命名；（后面用得着，作为参数传给其他Agent）
> - css表示抓取内容的**css路径**，value表示抓取的值，@href表示抓取对应css标签的href属性值，还有@src,@title等等；
> - 如果要抓取对应标签的值，可填`.`(包括html代码的全部内容）,`string(.)`（只包含对应标签的值），text()等同string(.)



![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn5.PNG)

填写完毕后点击<kbd>Dry Run</kbd>，如上图显示抓取到了数据表明有效，然后点击save保存，否则请修改extract下的参数再试。

保存后run一下，然后就会有生产出很多events，就是获取到的数据。如果没有获取到可能是数据库的问题。

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/events.PNG)





#### 新建第二个Agent，获取全文输出

---

同样的，Type选择Website Agent，Sources选中第一个Agent，下面的框一定勾选上

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn6.PNG)

{{url}}即第一个Agent传过来的超链接参数，这里mode一定填写merge，这样两个Agent的字段就组合到一起了，同样的选择一个接受到的event测试一下

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn7.PNG)

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn8.PNG)

把第一个的events都删除然后重新run一下可以发现第二个Agent也自动执行了,第二步完毕。

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn9.PNG)





#### 新建第三个Agent，输出成RSS

---

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn10.PNG)

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn11.PNG)

Type选择Data output Agent,Sources选择第二个Agent，secrets填写RSS地址自定义的末尾名称，item下就是RSS中的每一条信息了，填写上对应参数，其他默认即可。最后点击保存

至此，一个专属RSS源就已生成，点击第三个Agent就能看到下图所示，把它添加到RSS阅读器上去吧。

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Huginn%E5%88%B6%E4%BD%9CRSS/Huginn12.PNG)



### 参考

---

- [Huginn实现自动通过slack推送豆瓣高分电影](https://www.cnblogs.com/tiantianbyconan/p/8719444.html)	--[天天_byconan](https://www.cnblogs.com/tiantianbyconan)


