---
layout: post
title: Jekyll·利用腾讯云COS为博客添加相册模块
categories:
 - Jekyll
tags:
 - 相册

---

### ![](https://me.idealli.com/images/blog/18122304.jpg)

<!-- more -->

个人博客小站刚建站不久，想着除了主题里的功能外再添加上相册模块，于是半搜索半摸索把相册模块搞出来了，最后采用了利用腾讯云对象存储作图床的方案，在国内速度也快，不用上传图片还来修改代码，只负责上传图片就好。参考了以下博文[给hexo静态博客添加动态相册功能](<https://me.idealli.com/post/73ad4183.html>)（用了他的图🐶）实现本模块需要两方面的设置：一是腾讯云对象存储；二是代码层添加相关页面；

### 腾讯云部分设置

腾讯云新用户使用对象存储每月有50G的免费额度，持续6个月。**但是！不包括下行流量等其他消费**，这只是存储额度，每月还是需要花点小钱的，对于个人博客来说花销不会很大。

1. 登录腾讯云账号后选择菜单云产品->对象存储

2. 点击使用，选择存储桶列表然后新建，输入名称，选择地域，其他一切默认就好

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E8%85%BE%E8%AE%AF%E4%BA%91%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A8.PNG)

3. 然后在存储桶里上传一些照片，此时复制访问域名到地址栏里看看是否可以访问**（该访问域名代码层需要用到，很重要）**

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E8%85%BE%E8%AE%AF%E4%BA%91%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A81.PNG)

4. 不出意外应该会如下图所示，因为没有设置相关权限，暂时无法直接访问

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E8%85%BE%E8%AE%AF%E4%BA%91%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A83.PNG)

5. 点击权限管理，然后如图所示选择添加即可，完成后再进行第三步操作应该没问题啦

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E8%85%BE%E8%AE%AF%E4%BA%91%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A82.PNG)

6. 最后再开启防盗链，这样别人就不能偷偷用你流量和图片还不用付钱啦，选择基础配置->防盗链设置进行编辑，如下图所示，重点是Referer，即只有这里填写的IP地址或域名才能使用，其他人无法访问

   ![](<https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E8%85%BE%E8%AE%AF%E4%BA%91%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A84.PNG>)

   

#### 注意事项

> - 上传照片前，先在存储桶中建立一个文件夹，也就是你的相册名字，当然你也可以新建多个文件夹。
> - 但是有一点需要需要注意的是，**不能直接上传一个文件夹**，那样会出bug，见完文件夹后往里面上传照片，文件夹里面不能再新建文件夹了（除非你自己改造下面的相应代码）
> - 照片一定放在一个文件夹中，有文件夹又有单独的照片，相册是不会显示的（跟代码有关）

### Jekyll配置

1. 在项目里添加一个相册目录album，然后添加index.html（根据代码层次可能略有差异，不知道的话先去了解各个模块的功能吧）。

2. 然后`_config.yml`配置文件中的菜单栏里也加上相册模块

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/Jekyll%E9%85%8D%E7%BD%AE.PNG)

3. 在相册页面的index.html添加如下代码，其中JS代码是重点，页面布局可以看个人喜好啦：

   ```html
   <style type="text/css">
   			.main {
   			padding-bottom: 150px;
   			margin-top: 0px;
   			}
   			body {
   				background-size: 100%;
   				/*background-position: top left;*/
   			}
   			.imgbox{
   			 width: 100%;
   			 overflow: hidden;
   			}
   			.box{
   				visibility: visible;
   				overflow: auto; 
   				zoom: 1;
   			}
   			.box li{
   			float: left;
   			width: 25%;
   			position: relative;
   			overflow: hidden;
   			text-align: center;
   			list-style: none;
   			margin: 0;
   			/*display: inline;*/
   			padding: 0;
   			}
   			.box li span{
   			display: block;
   			font-size: 16px;
   			font-weight: 600;
   			font-family: fantasy;
   			-webkit-box-sizing: border-box;
   			box-sizing: border-box;
   			}
   			img.imgitem{
   				padding: unset;
   				padding: unset;
   				border: unset;
   				position: relative;
   				padding: 0px;
   				height: auto;
   				width: 100%;
   			}
   		.v{
   			width: 95%;
   			padding-top: 40px;
       		margin: 0 auto;
   		}
   		div#posts.posts-expand {
   			border: unset;
   			padding: unset;
   			margin-bottom: 10px;
   		}
   		.posts-expand .post-body img{
   			padding: 12px !important;
   		}
   		.box p{
   			display: block;
   			font-size: 12px;
   			font-family: 'SwisMedium';
   			-webkit-box-sizing: border-box;
   			box-sizing: border-box;
   			text-align: center;
   		}
   		.box span strong{
   			background: rgba(0,0,0,0.4);
   			padding: 20px;
   		}
   		.posts-expand .post-title {
   			display: none;
   		}
   		.title{
   			width: 40%;
   			margin: 0 auto;
   			text-align: center;
   		}
   		.title h2{
   			border: double;
   		    color: white;
   		    width: 70%;
   		    margin: 0 auto;
   		    background-color: tomato;
   		}
   		.title hr{
   			height:2px;
   		}
   		.btn-more-posts{
   			display: inline-block;
   			vertical-align: middle;
   			font: 85px/250px 'ChaletComprimeMilanSixty';
   			color: #000;
   			width: 100%;
   			text-align: center;
   			border: unset;
   			height: 400px;
   			background-color: #121212;
   			-webkit-box-sizing: border-box;
   			box-sizing: border-box;
   		}
   		@media (max-width: 767px){
   			.box li {
   			width: 100%;
   		}
   		.posts-expand .post-body img{
   			padding: 0px !important;
   		}
   		.box span {
   			border-right: unset;
   			font-size: 17px;
   		}
   		.box p{
   			border-right: unset;
   			font-size: 12px;
   		  
   		}
   		.posts-expand {
   			margin: unset;
   		}
   			div#comments.comments.v {
   			width: 96%;
   			padding-top: 50px;
   		}
   		}
   		@media (min-width: 1600px){
   			.container .main-inner{
   				width: 100%;
   			}
   		}
   		.footer{
   			background-color: #121212 !important;
   		}
   		.v .vwrap .vmark .valert .vcode {
   			background: #00050b !important;
   		}
   		.nav>li {
   			float: left;
   		}
   		.imgbox img{
   			width: 95%;
   		}
   		.container-fluid a{
   			border-bottom-style: none;
   			color: #555;
   		}
   		</style>
   		</head>
   		<nav class="navbar navbar-default" role="navigation">
   		<div class="container-fluid"> 
   		<div class="navbar-header">
   			<button type="button" class="navbar-toggle" data-toggle="collapse"
   					data-target="#example-navbar-collapse">
   				<span class="sr-only">切换导航</span>
   				<span class="icon-bar"></span>
   				<span class="icon-bar"></span>
   				<span class="icon-bar"></span>
   			</button>
   			<a class="navbar-brand" href="https://jiangsang.github.io/">Jianger's Blog</a>
   		</div>
   		<div class="collapse navbar-collapse" id="example-navbar-collapse">
   			<ul class="nav navbar-nav">
   				<li><a href="https://jiangsang.github.io/"><i class="menu-item-icon fa fa-fw fa-home"></i>首页</a></li>
   				<li><a href="https://jiangsang.github.io/about"><i class="menu-item-icon fa fa-fw fa-user"></i>关于</a></li>
   				<li><a href="https://jiangsang.github.io/archives/"><i class="menu-item-icon fa fa-fw fa-archive"></i>归档</a></li>
   				<li><a href="https://jiangsang.github.io/tags/"><i class="menu-item-icon fa fa-fw fa-tags"></i>标签</a></li>
   				<li><a href="https://jiangsang.github.io/life/"><i class="menu-item-icon fa fa-fw fa-apple"></i>生活</a></li>
   				<li class="active"><a href="https://jiangsang.github.io/life/"><i class="menu-item-icon fa fa-fw fa-camera-retro"></i>相册</a></li>
   			</ul>
   		</div>
   		</div>
   	</nav>
   		<div id="box" class="box"></div>
   		<script type="text/javascript">
   		function loadXMLDoc(xmlUrl) 
   		{
   			try //Internet Explorer
   			{
   				xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
   			}
   			catch(e)
   			{
   			  try //Firefox, Mozilla, Opera, etc.
   				{
   				  xmlDoc=document.implementation.createDocument("","",null);
   				}
   			  catch(e) {alert(e.message)}
   			}
   			
   			try 
   			{
   				  xmlDoc.async=false;
   				  xmlDoc.load(xmlUrl);
   			}
   			catch(e) {
   				try //Google Chrome  
   				  {  
   					var chromeXml = new XMLHttpRequest();
   					chromeXml.open("GET", xmlUrl, false);
   					chromeXml.send(null);
   					xmlDoc = chromeXml.responseXML.documentElement; 				
   					//alert(xmlDoc.childNodes[0].nodeName);
   					//return xmlDoc;    
   				  }  
   				  catch(e)  
   				  {  
   					  alert(e.message)  
   				  }  		  	
   			}
   			return xmlDoc; 
   		}
   		//xmllink就是你的相册存储桶的访问域名
   		xmllink="https://album-1300776923.cos.ap-shanghai.myqcloud.com/";
   		xmlDoc=loadXMLDoc(xmllink);
   		var urls=xmlDoc.getElementsByTagName('Key');
   		var date=xmlDoc.getElementsByTagName('LastModified');
   		var showNum=12; //每个相册一次展示多少照片
   		if ((window.innerWidth)>1200) {wid=(window.innerWidth*3)/18;}
   		var box=document.getElementById('box');
   		var i=0;
   		var content=new Array();
   		var tmp=0;
   		var kkk=-1;
   		for (var t = 0; t < urls.length ; t++) {
   			var bucket=urls[t].innerHTML;
   			var length=bucket.indexOf('/');
   			if(length===bucket.length-1){
   				kkk++;
   				content[kkk]=new Array();
   				content[kkk][0]={'url':bucket,'date':date[t].innerHTML.substring(0,10)};
   				tmp=1;
   			}
   			else {
   				content[kkk][tmp++]={'url':bucket.substring(length+1),'date':date[t].innerHTML.substring(0,10)};
   			}
   		}
   		for (var i = 0; i < content.length; i++) {
   			var conBox=document.createElement("div");
   			conBox.id='conBox'+i;
   			conBox.style='display: inline-block;';
   			box.appendChild(conBox);
   			var item=document.createElement("div");
   			var title=content[i][0].url;
   			item.innerHTML="<div class=title style=margin-top:20px;><h2>"+title.substring(0,title.length-1)+"</h2><hr/></div>";
   			conBox.appendChild(item);
   			for (var j = 1; j < content[i].length && j < showNum+1; j++) {
   				var con=content[i][j].url;
   				var item=document.createElement("li");
   				item.innerHTML="<div class=imgbox id=imgbox><img class=imgitem src="+xmllink+'/'+title+con+" alt="+con+"></div><span>"+con.substring(0,con.length-4)+"</span><p>上传于"+content[i][j].date+"</p>";
   				conBox.appendChild(item);
   			}
   			if(content[i].length > showNum){
   				var moreItem=document.createElement("button");
   				moreItem.className="btn-more-posts";
   				moreItem.id="more"+i;
   				moreItem.value=showNum+1;
   				let cur=i;
   				moreItem.onclick= function (){
   					moreClick(this,cur,content[cur],content[cur][0].url);
   				}
   				moreItem.innerHTML="<span style=display:inline;><strong style=color:#f0f3f6;>加载更多</strong></span>";
   				conBox.appendChild(moreItem);
   			}
   		}
   ```

   再往cos存储桶里上传照片，就可以了！效果如下

   ![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/%E7%9B%B8%E5%86%8C%E9%A1%B5%E9%9D%A2.PNG)

   

   ### 相册布局界面

   > 主题有默认布局，如果不想要在默认布局的基础上放置内容页面需要在layout文件下添加布局代码album.html，然后album目录下的index.html需要添加`layout:album`代码，意思便是使用album布局喽
   >
   > 这个很重要，每个人喜好不同，花点时间打造成自己喜欢的相册空间吧！