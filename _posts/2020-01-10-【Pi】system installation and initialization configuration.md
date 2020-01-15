---
layout: post
title: 【树莓派】系统安装与初始化配置
categories:
 - 好玩的
tags:
 - 树莓派
---

![](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/IMG_0634.GIF)

<!-- more -->

### 前排提示

以下必须：

- 树莓派一块，我使用的3B+
- SD卡一枚（最好内存16G以上）
- 读卡器，用于录入镜像到SD卡
- 写卡软件，我使用的BalenaEtcher



### 下载系统并写入

---

#### 选择镜像

官方[下载地址](https://www.raspberrypi.org/downloads)，Raspbian为官方推荐系统，基于Debian，看到还有很多第三方系统，刚入手，先用官方推荐的。Raspbian系统又分为轻量版和桌面版两种。桌面版有图形化显示界面，并且预装了一些软件，占用内存更大，我下载了有图形化的，建议下载torrent,然后通过迅雷下载，速度更快。

![](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/raspbian.JPG)
![third-party](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/third-party.JPG)

下载完成后解压缩，有一个img后缀的镜像文件，就是下一步写入SD卡要用的。

#### 安装BalenaEtcher

这是一个写入镜像到SD卡的软件，[官方网站](https://www.balena.io/etcher)有macos和windows版两种，但是下载太慢了，这里贴一个windows版地址，下载速度飞起：http://soft.u-jingling.com/201912/Etcher_V1.5.70_XiTongZhiJia.zip。安装完成后打开即可。

#### 系统录入

将SD卡插入读卡器，然后插入电脑接口，点击select选中刚才解压得到的后缀img的镜像文件，软件会自动选中SD卡，当然也可以选择，如果内存足够，Flash点击等待完成后即可关闭软件。（SD卡先别弹出！）

![BalenaEtcher](https://article-1300776923.file.myqcloud.com/%E6%A0%91%E8%8E%93%E6%B4%BE/BalenaEtcher.JPG)







### 系统初始化配置

---

此时查看sd卡，可以看到一个名字为boot的分区，这个`boot`分区就是系统的`/boot`文件夹

- #### 开启SSH

  系统默认SSH是关闭的，在boot下新建一个名为`ssh`的文件夹SSH便开启，开启SSH就可以连接树莓派了。

- #### 设置WiFi连接

  在boot分区下新建`wpa_supplicant.conf`文件，并将如下内容保存
  其中ssid 为wifi 名称（必需），psk 为wifi密码（必需）,priority为可选参数，设置优先级的，并且WiFi连接可设置多个

  ```
  country=CN
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  
  network={
      ssid="wifiname"
      psk="password"
      priority=1
  }
  ```





### 连接树莓派并配置

---

插上充电器，树莓派即启动，并且指示灯会亮起，在3b+上，红色灯亮起时，为接通电源状态；绿色灯亮起时，为系统在对SD卡做读写操作。树莓派启动后，就可以通过SSH连接登录系统。



#### 获取树莓派IP并连接

- 获取IP：树莓派启动后，便会自动连接WiFi，这时登录路由器后台便可以看到树莓派的IP地址，设备名称为raspberry就是了

- 连接树莓派：连接工具有很多，比如putty、xshell等，我使用的xshell连接，输入主机IP，其他默认即可。

  该系统ssh初始账户为`pi`，密码为`raspberry`



#### 校正时区，时间

在命令行输入第二行代码并回车，然后选择ShangHai即可

```bash
# 打开时区设置
sudo dpkg-reconfigure tzdata
# 选择Asia  ---> ShangHai
```



#### 修改系统软件源

官方的源在国内下载很慢，这里换成阿里云镜像源

```
# 树莓派系统默认使用nano做为编辑器，修改完成后使用ctrl+x退出，退出时会提示是否保存文件, 按Y即可

sudo nano /etc/apt/sources.list

deb http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi
```



#### 设置ROO用户密码

刚登录进来是以pi用户的身份的，有时候会有权限问题，因此使用ROOT用户操作更顺畅，输入以下命令后根据提示输入两次密码即可

```bash
#设置root
sudo passwd root
```

然后输入`su - root` 命令切换到ROOT用户。



#### 安装vim和配置

```csharp
# 更新软件源
sudo apt-get update
# 安装vim
sudo apt-get install vim

```

配置vim（可选）：

```csharp
# 修改配置
sudo vim /etc/vim/vimrc

# 语法高亮
syntax on
# 显示行号
set nu
```





### 可能遇到的问题

- 烧制完系统后，发现SD卡分成了两个分区，但是加起来只剩下7G左右。问题不大，这是由于在安装系统的时候给TF卡分了Windows无法识别的分区。

  ```
  df -h
  ```

  进入树莓派输入如上命令查看分区内存大小，如果内存仍旧很小，就需要其他操作扩容分区了。

  ![](C:\Users\jianger\Desktop\分区.JPG)

- 相关问答与教程

  [树莓派新烧入系统之后，SD卡分区大小怎么变小了？](https://www.v2ex.com/t/635916)  --V2EX

  [恢复安装过树莓派相关操作系统的TF卡容量](bbs.shumeipaiba.com/thread-26-1-1.html)  --树莓派论坛

贴上一个B站安装与初始化视频，未完待续~

<iframe src="//player.bilibili.com/player.html?aid=64914801&cid=112679504&page=1" scrolling="no" border="0" width="100%" height="450px" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>


