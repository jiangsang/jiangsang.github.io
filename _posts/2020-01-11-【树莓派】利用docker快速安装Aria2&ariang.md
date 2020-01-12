---
layout: post
title: 【树莓派】利用docker快速安装Aria2&ariang
categories:
 - 好玩的
tags:
 - 树莓派
 - Docker
 - Aria2
---

### 前排提示

- 树莓派3B+
- 系统为Raspbian



### 安装Docker

---

树莓派的架构是Arm，与一般的Windows笔记本电脑不同，x86的容器是无法使用的。而树莓派3B之后的板子实际上是64为arm64v8的，但官方的Raspbian系统为了兼容性问题依旧是32位，故编译环境依旧是32位。以下网址是一些arm32v7可用的容器。

<!-- more -->

```
https://hub.docker.com/u/arm32v7
```

好的，接下来安装docker，安装前先更新一下软件

```bash
sudo apt-get update
```

然后使用阿里云的一键安装

```bash
sudo curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

> 说明：上述操作我使用的是Raspbian系统默认的源，之前修改成阿里云的源，总是下载失败，显示缺少依赖。于是重新安装，什么都不改，直接运行以上两个命令，然后成了！奇怪~







### 更换容器镜像源

我使用官方镜像源会连接不上，因此换成阿里云提供的镜像加速服务，免费的。可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://iikujkej.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```







### 安装Aria2&ariang

---

其实Raspbain系统默认已经安装了Aria2，运行`apt-cache show aria2`即可看到如下信息，但是命令行不方便~，so...

![](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/aria2.JPG)

搜索半天终于找到一个build好的arm32v7可以使用的aria2容器镜像，进入网址☞https://hub.docker.com/r/huangzulin/aria2-ui-pi可查看详细文档，输入以下命令回车进行下载。

```bash
sudo docker pull huangzulin/aria2-ui-pi
```

- 快速运行aria2-ui-pi

```bash
sudo docker run -d --name aria2-ui-pi -p 80:80 -p 6800:6800 huangzulin/aria2-ui-pi
```

进入Aria2 webui界面: http://yourip/ui
进入FileManger文件管理界面: http://yourip，默认账户：admin，密码：admin 。

![](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/aria2ui.JPG)

至此就安装完成啦！docker安装好了还是很快的。



### 参考

本文参考了以下教程：

- [树莓派上 Docker 的安装和使用](http://shumeipai.nxez.com/2019/05/20/how-to-install-docker-on-your-raspberry-pi.html)   --树莓派实验室
- [树莓派入坑系列 Part-3 Docker安装与配置](https://www.bilibili.com/video/av29938313)  --B站UP[speculatecat](https://space.bilibili.com/28436361?spm_id_from=333.788.b_765f7570696e666f.1)