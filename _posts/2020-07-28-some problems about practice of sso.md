---
layout: post
title: SSO单点登陆实战及遇到的问题
categories:
 - SSO
 - 分布式
tags:
 - SSO
 - CAS
---

### SSO单点登录
---
单点登录：Single Sign On，它的出现致力于提供分布式项目中的用户登录和注销操作，使之只需要一次登陆即可在所有其他系统中无需登陆。注销同理，在任一系统中注销其他系统也会退出登陆。这样各个系统无需有登陆退出功能，统一进行用户授权。  比如阿里系的**淘宝和天猫**，很明显地我们可以知道这是两个系统，但是你在使用的时候，登录了天猫，淘宝无需再进行登录，一次授权，多个应用使用。  

在SSO体系中，主要包括三部分：  
1. User用户（多个）  
2. Web 应用（多个）  
3. SSO 认证中心（ 1 个）  

<!-- more -->
而SSO的实现基本核心原则如下：  

- 所有的登录授权都在 SSO 认证中心进行  
- SSO 认证中心通过一些方法来告诉 Web 应用当前访问用户究竟是不是已通过认证的用户  
- SSO 认证中心和所有的 Web 应用建立一种信任关系， SSO 认证中心对用户身份正确性的判断会通过某种方法告之 Web 应用，而且判断结果必须被 Web 应用信任。  

#### 单系统回顾  
在学习SSO的实现之前，先对以往的单系统用户身份授权认证的实现进行简单回顾。  
首先，HTTP是无状态的协议，服务器无法直接确认用户的信息。于是乎，W3C就提出了：给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息。通行证就是Cookie。  

如果说Cookie是检查用户身上的”通行证“来确认用户的身份，那么Session就是通过检查服务器上的”客户明细表“来确认用户的身份的。Session相当于在服务器中建立了一份“客户明细表”。  

HTTP协议是无状态的，Session不能依据HTTP连接来判断是否为同一个用户。于是乎：服务器向用户浏览器发送了一个名为JESSIONID的Cookie，它的值是Session的id值。其实Session是依据Cookie来识别是否是同一个用户。  

所以，一般我们单系统实现登录会这样做：  

- **登录**：将用户信息保存在Session对象中  
如果在Session对象中能查到，说明已经登录;  
如果在Session对象中查不到，说明没登录（或者已经退出了登录）  
- **注销**（退出登录）：从Session中删除用户的信息  
- **记住我**（关闭掉浏览器后，重新打开浏览器还能保持登录状态）：配合Cookie来用  

### CAS实现  
---
紧接SSO单点登录，CAS全称为Central Authentication Service即中央认证服务，是一个企业多语言单点登录的解决方案，并努力去成为一个身份验证和授权需求的综合平台。  

CAS是由Yale大学发起的一个企业级的、开源的项目，旨在为Web应用系统提供一种可靠的**单点登录解决方法**（属于 Web SSO ），即它是SSO的一种实现。  

CAS协议至少涉及三方：客户端Web浏览器，请求身份验证的Web应用程序和CAS服务器。 它也可能涉及后端服务，如数据库服务器，它没有自己的HTTP接口，但与Web应用程序进行通信。  

![](http://jianger-upic.test.upcdn.net/uPic/CAS%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

如上图所示，我们可以将认证分为两类：未登录认证和已认证。

**未认证**：用户通过浏览器发起请求，因为未认证，发起的请求没有附带有效的sessionid信息，应用端重定向到cas服务端去验证，显然会失败，所以cas服务端会返回用户认证页面给浏览器，用户输入账号信息后认证成功后，cas服务端再重定向到应用端，并附带用户认证后的sessionid。这里就完成了图中的步骤1至4。

**已认证**：紧接着，应用端将刚才获得的sessionid信息再去cas客户端进行验证，验证通过获得用户信息。这里完成了图中的步骤5至6。

> 以上的步骤中使用sessionid说明是为了更好理解，实际使用的类似它的封装

从结构上看，CAS 包含两个部分： CAS 服务端和 CAS 客户端。CAS服务端需要独立部署，主要负责对用户的认证工作；CAS 客户端负责处理对客户端受保护资源的访问请求，需要登录时，重定向到 CAS 服务端。上图是 CAS 最基本的协议过程。

CAS Ticket（票据），在CAS1.0和2.0中有不同的术语概念：  

**CAS1.0**：

- TGT：Ticket Granting Ticket，TGT是CAS为用户签发的登录票据，拥有了TGT，用户就可以证明自己在CAS成功登录过。TGT封装了Cookie值以及此Cookie值对应的用户信息。用户在CAS认证成功后，CAS生成cookie（叫TGC），写入浏览器，同时生成一个TGT对象，放入自己的缓存，TGT对象的ID就是cookie的值。当HTTP再次请求到来时，如果传过来的有CAS生成的cookie，则CAS以此cookie值为key查询缓存中有无TGT ，如果有的话，则说明用户之前登录过，如果没有，则用户需要重新登录。
- ST：Service Ticket，ST是CAS为用户签发的访问某一service的票据。用户访问service时，service发现用户没有ST，则要求用户去CAS获取ST。用户向CAS发出获取ST的请求，如果用户的请求中包含cookie，则CAS会以此cookie值为key查询缓存中有无TGT，如果存在TGT，则用此TGT签发一个ST，返回给用户。用户凭借ST去访问service，service拿ST去CAS验证，验证通过后，允许用户访问资源。
- TGC：Ticket Granting Cookie，存放用户身份认证凭证的cookie，在浏览器和CAS Server间通讯时使用，并且只能基于安全通道传输（Https），是CAS Server用来明确用户身份的凭证。

**CAS2.0**：

- PGT：Proxy Granting Ticket，Proxy Service的代理凭据。用户通过CAS成功登录某一Proxy Service后，CAS生成一个PGT对象，缓存在CAS本地，同时将PGT的值（一个UUID字符串）回传给Proxy Service，并保存在Proxy Service里。Proxy Service拿到PGT后，就可以为Target Service（back-end service）做代理，为其申请PT。
- PT：Proxy Ticket，PT是用户访问Target Service（back-end service）的票据。如果用户访问的是一个Web应用，则Web应用会要求浏览器提供ST，浏览器就会用cookie去CAS获取一个ST，然后就可以访问这个Web应用了。如果用户访问的不是一个Web应用，而是一个C/S结构的应用，因为C/S结构的应用得不到cookie，所以用户不能自己去CAS获取ST，而是通过访问proxy service的接口，凭借proxy service的PGT去获取一个PT，然后才能访问到此应用。 
- PGTIOU：Proxy Granting Ticket I Owe You，PGTIOU是CAS协议中定义的一种附加票据，它增强了传输、获取PGT的安全性。
    PGT的传输与获取的过程：Proxy Service调用CAS的serviceValidate接口验证ST成功后，CAS首先会访问pgtUrl指向的https url，将生成的 PGT及PGTIOU传输给proxy service，proxy service会以PGTIOU为key，PGT为value，将其存储在Map中；然后CAS会生成验证ST成功的xml消息，返回给Proxy Service，xml消息中含有PGTIOU，proxy service收到Xml消息后，会从中解析出PGTIOU的值，然后以其为key，在map中找出PGT的值，赋值给代表用户信息的Assertion对象的pgtId，同时在map中将其删除。

**TGT、ST、PGT、PT之间关系**

1. ST是TGT签发的。用户在CAS上认证成功后，CAS生成TGT，用TGT签发一个ST，ST的ticketGrantingTicket属性值是TGT对象，然后把ST的值redirect到客户应用。
2. PGT是ST签发的。用户凭借ST去访问Proxy service，Proxy service去CAS验证ST（同时传递PgtUrl参数给CAS），如果ST验证成功，则CAS用ST签发一个PGT，PGT对象里的ticketGrantingTicket是签发ST的TGT对象。
3. PT是PGT签发的。Proxy service代理back-end service去CAS获取PT的时候，CAS根据传来的pgt参数，获取到PGT对象，然后调用其grantServiceTicket方法，生成一个PT对象。

### CAS服务端

### 实践中的问题  

- 单点登出返回的页面仍旧显示为登陆成功的，刷新后才会跳转到cas服务端登陆页面.  

### 参考资料  
[什么是单点登录](https://zhuanlan.zhihu.com/p/66037342)	-知乎·[Java3y](https://www.zhihu.com/people/Java_3y)  
未完待续。。。