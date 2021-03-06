---
layout: post
title: 利用Feed43为网站自制RSS源
categories:
 - 好玩的
tags:
 - RSS

---

### 什么是RSS，它可以做什么

快2020年了，RSS日渐式微，我也是去年机缘巧合下才开始使用的，以前只是听说过。RSS，全称Really Simple Syndication，又称[简易信息聚合](https://baike.baidu.com/item/%E7%AE%80%E6%98%93%E4%BF%A1%E6%81%AF%E8%81%9A%E5%90%88)（也叫聚合内容）是一种RSS基于[XML](https://baike.baidu.com/item/XML)标准，在互联网上被广泛采用的内容包装和投递协议。RSS搭建了[信息](https://baike.baidu.com/item/%E4%BF%A1%E6%81%AF/111163)迅速传播的一个技术平台，使得每个人都成为潜在的信息提供者。它简洁直接无广告，只提供内容！

<!-- more -->

说人话就是采用一种某种标准的信息数据，广泛应用于网上新闻，博客，不用打开相关网站，有新内容就会推送（当然，前提是需要有一个RSS阅读器），其实RSS用途不仅于此，你可以使用RSS订阅一切内容，例如如下场景：

- 某个资源网站更新了想看的信息，自动推送相关信息给你
- 喜欢的爱豆在各个平台上有新动态了，第一时间get~
- 关注的歌手在附近有新演唱会啦，买买买~
- 淘宝上看中的宝贝降价啦，立马提醒你
- 明天会下雨，提前温馨通知
- ......

------

### 可以直接食用的订阅源

简单推荐一些官方的源和各位大佬们做好的

| 网站                               | 简述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| [FeedX](https://feedx.co/)         | 主要提供国内外新闻平台的RSS源，很良心                        |
| [RSSHub](https://docs.rsshub.app/) | DIYGod大佬的开源项目，提供了很多RSS源，种类很多，目前还在不断更新中 |
| [cnbeta](https://www.cnbeta.com/)  | 主要提供科技数码方面的新闻，官方源https://www.cnbeta.com/backend.php，但是不提供全文。 |
| [少数派](https://sspai.com/)       | 官方源https://sspai.com/feed                                 |

### 获取全文输出

有的官方源不支持全文输出，只有简述，例如cnbeta，将其转成有全文输出的办法也简单，利用相关网站。以下推荐一些

- [freefullrss](https://www.freefullrss.com/) 免费RSS全文输出网站，但是最高限定10条

  ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/freefullrss.PNG)

- [fivefilters](https://fivefilters.org/content-only/)

  ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/fivefilters.PNG)

------

### 给指定网站制作RSS源

> [ Feed43](https://feed43.com/) 国外老牌RSS定制网站，已运营多年，它能将网页转换为标准格式的 RSS 源。

优点

- 免费（也有收费套餐，但一般用不上）
- 容易上手，无需编程基础

缺点

- 国外服务，不是很稳定
- 可自定义程度略低
- 有些网站不能抓取，报403或404
- 免费版只会每6小时抓取一次

---

### 开始炮制

1. 如果你想以后继续用或者更改你的feed，可以注册一个账号，不注册登录也可以用

2. 点击create your first rss feed开始

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_1.PNG)

3. 这里以1905电影网为例，Step 1. Specify source page address (URL)在Address框输入网址，然后点击reload加载，如果出现乱码，试试Encoding框输入UTF-8

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_2.PNG)

4. 如果不幸显示`404 Not Found`或者`403 Forbid`，那么说明该网站无法转换，你可以换个姿势再试试

5. 如果成功你就可以看到Page Source框里的html代码，Step 2. Define extraction rules，定义抓取规则。

6. Global Search Pattern是可选的，一般留空即可，重点在Item (repeatable) Search Pattern

7. 我要抓取1905网里的电影资讯，可以看到资讯代码如下`<a href="https://www.1905.com/news/20191225/1428107.shtml" target="_blank" data-hrefexp="fr=homepc_news_kx">奥斯卡热门影片《别告诉她》聚焦社会话题 文化差异引碰撞</a>`

8. 那么在Item (repeatable) Search Pattern框下输入`<a href="\{\%\}" target="_blank" data-hrefexp="fr=homepc_news_kx">\{\%\}</a>`

   即可，\{\%\}表示你要获取的内容，点击<kbd>Extract</kbd>然后网页中符合这个模板的内容都会被抓取到。

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_5.PNG)

9. Step 3. Define output format,定义输出格式，重点在RSS item properties下的Item Title Template（标题）、Item Link Template（链接）、Item Content Template（全文内容），将第二步获取到的内容输入，\{\%数字\}的形式

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_3.PNG)

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_4.PNG)

   最后点击Preview即可，然后你就能看到最后的RSS源

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/feed43_6.PNG)

   Feed URL就是RSS源，你还可以修改成简单好记的名称。

### 全文输出

利用Feed43做的RSS源无法获取到全文内容，那么利用上文提到的相关网站就可以啦，下面我使用[freefullrss](https://www.freefullrss.com/)进行操作。

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/freefullrss_1.PNG)

在输入框输入相关信息，然后点击<kbd>Creat Full Text RSS</kbd>，成功就会显示如下成果：

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/freefullrss_2.PNG)

最后将该网页网址复制添加到RSS阅读器即可，大功告成！
