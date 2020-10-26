---
layout: post
title: SSO单点登陆实战及遇到的问题
categories:
 - SSO
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

#### CAS流程

我们可以将认证分为两类情况：未登录CAS和已登录CAS。

**未登录**：用户通过浏览器对应用A发起请求，因为未认证，发起的请求没有附带有效的sessionid信息，应用A302重定向到cas服务端，CAS服务端返回登录表单，用户输入账号信息后，cas服务端保存用户信息TGT，对用户浏览器设置TGC，302跳转到应用A，并附带发给该应用端的ST。应用A收到请求拿着ST去服务端验证，验证成功后应用A给浏览器设置cookie并跳转到受限页面。

![](http://image.jianger.space/uPic/CAS%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

**已登录**：

- 如果是请求的是相同应用A，应用端通过前端请求的sessionid信息验证用户session，然后直接返回该应用下的受限页面；

- 如果请求的不同应用B，应用B会302重定向到CAS服务端，服务端通过保存的TGT信息与浏览器的TGC信息进行验证，然后302重定向到应用B，并发放ST给应用B，然后应用B会拿着ST去服务端再验证，验证成功后应用B给浏览器设置cookie并跳转到受限页面。



下面是官方给出的详细流程，图源：[Apereo](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol.html)

![](http://image.jianger.space/uPic/cas_flow_diagram.png)



#### CAS Ticket

CAS Ticket（票据），在CAS1.0和2.0中有不同的术语概念：  

**CAS1.0**：

- TGT：Ticket Granting Ticket，TGT是CAS为用户签发的登录票据，拥有了TGT，用户就可以证明自己在CAS成功登录过。TGT封装了Cookie值以及此Cookie值对应的用户信息。用户在CAS认证成功后，CAS生成cookie（叫TGC），写入浏览器，同时生成一个TGT对象，放入自己的缓存，TGT对象的ID就是cookie的值。当HTTP再次请求到来时，如果传过来的有CAS生成的cookie，则CAS以此cookie值为key查询缓存中有无TGT ，如果有的话，则说明用户之前登录过，如果没有，则用户需要重新登录。
- ST：Service Ticket，ST是CAS为用户签发的访问某一service（应用）的票据。用户访问service时，service发现用户没有ST，则要求用户去CAS获取ST。用户向CAS发出获取ST的请求，如果用户的请求中包含cookie，则CAS会以此cookie值为key查询缓存中有无TGT，如果存在TGT，则用此TGT签发一个ST，返回给用户。用户凭借ST去访问service，service拿ST去CAS验证，验证通过后，允许用户访问资源。
- TGC：Ticket Granting Cookie，存放用户身份认证凭证的cookie，在浏览器和CAS Server间通讯时使用，并且只能基于安全通道传输（Https），是CAS Server用来明确用户身份的凭证。

**CAS2.0**：

- PGT：Proxy Granting Ticket，Proxy Service的代理凭据。用户通过CAS成功登录某一Proxy Service后，CAS生成一个PGT对象，缓存在CAS本地，同时将PGT的值（一个UUID字符串）回传给Proxy Service，并保存在Proxy Service里。Proxy Service拿到PGT后，就可以为Target Service（back-end service）做代理，为其申请PT。
- PT：Proxy Ticket，PT是用户访问Target Service（back-end service）的票据。如果用户访问的是一个Web应用，则Web应用会要求浏览器提供ST，浏览器就会用cookie去CAS获取一个ST，然后就可以访问这个Web应用了。如果用户访问的不是一个Web应用，而是一个C/S结构的应用，因为C/S结构的应用得不到cookie，所以用户不能自己去CAS获取ST，而是通过访问proxy service的接口，凭借proxy service的PGT去获取一个PT，然后才能访问到此应用。 
- PGTIOU：Proxy Granting Ticket I Owe You，PGTIOU是CAS协议中定义的一种附加票据，它增强了传输、获取PGT的安全性。
    PGT的传输与获取的过程：Proxy Service调用CAS的serviceValidate接口验证ST成功后，CAS首先会访问pgtUrl指向的https url，将生成的 PGT及PGTIOU传输给proxy service，proxy service会以PGTIOU为key，PGT为value，将其存储在Map中；然后CAS会生成验证ST成功的xml消息，返回给Proxy Service，xml消息中含有PGTIOU，proxy service收到Xml消息后，会从中解析出PGTIOU的值，然后以其为key，在map中找出PGT的值，赋值给代表用户信息的Assertion对象的pgtId，同时在map中将其删除。

**TGT、ST，PGT、PT之间关系**

1. ST是TGT签发的。用户在CAS上认证成功后，CAS生成TGT，用TGT签发一个ST，ST的ticketGrantingTicket属性值是TGT对象，然后把ST的值redirect到客户应用。
2. PGT是ST签发的。用户凭借ST去访问Proxy service，Proxy service去CAS验证ST（同时传递PgtUrl参数给CAS），如果ST验证成功，则CAS用ST签发一个PGT，PGT对象里的ticketGrantingTicket是签发ST的TGT对象。
3. PT是PGT签发的。Proxy service代理back-end service去CAS获取PT的时候，CAS根据传来的pgt参数，获取到PGT对象，然后调用其grantServiceTicket方法，生成一个PT对象。

> 💡：TGT用于浏览器与CAS服务端建立信任，ST既用于应用端与服务端建立信任，也用于应用端与浏览器建立信任

### 总结

- CAS认证鉴权仍旧使用的是浏览器cookie和服务器session的验证机制，只是封装了相关信息
- 在统一认证平台登录后，首次访问该系统下的任一应用都需要重定向到CAS服务端去验证cookie，然后给颁发给用户ST，用户拿着ST访问应用，该应用拿到ST后还得去CAS服务端验证一遍
- 如果用户已在统一认证平台登录，并且非首次访问该系统下的任一应用，则验证只需要在浏览器和应用端进行。-存疑

#### 疑问

- **为什么已经有了TGC，还需要ST？**

    暂时还不知道啊

- **如何避免sessionId 冲突？**

    要避免冲突很简单，**在应用给前端设置的cookie 的key 上加入服务名作为前缀**，比如分别写成a_sessionId 和b_sessionId。

- **假设a 与b 使用同样的单点登录认证Server，有没有可能出现a 应用登录过期，b 应用没有过期的情况？**

    不会。在业务实现中，CAS Client 会定期和CAS Server 进行通信，如果用户一直在操作，那么CAS Server 就会相应延长TGC 的过期时间，最终对于a 和b 来说，**TGC 的过期时间一定是相同的**。所以哪怕两边的session 设置过期时长不一致，认证状态最多走到CAS Server 处通过TGC 的检测就能完成，而不会出现a 需要登录，b 不需要登录的情况。

### 参考资料  
[什么是单点登录](https://zhuanlan.zhihu.com/p/66037342)	-知乎·[Java3y](https://www.zhihu.com/people/Java_3y)  

[前后端鉴权二三事](https://zhuanlan.zhihu.com/p/157324388)  - 知乎