---
layout: post
title: 使用Selenium-Python自动化购物下单
categories:
 - Python
tags:
 - Selenium
 - Python
 
---

Selenium是一款web应用自动化测试的工具库，Selenium测试直接运行在浏览器中，就像真正的用户在操作一样，主流的浏览器都支持selenium。那么除了将其用于测试上，我们还可以用来做一些频繁机械反复的工作，例如每隔一段时间查看购物平台的商品详情页面，查看到货状态，有货就加购然后下单，下单成功发邮件提醒付款。

[Selenium中文文档](https://selenium-python-zh.readthedocs.io/)

眼下就有一个需求：疫情当前，口罩稀缺，商家不定时上货，想蹲口罩，又不想整天蹲着页面看，机械而无聊。

<!-- more -->

### 前排提示

以下介绍的方法适用于以下情形：

- 平台商家不定时上货的情况，如果是定时上货，估计瞬时流量很大，利用该工具恐难胜任。
- 应用的网页偏复杂，不太容易使用简单的请求获取商品信息，毕竟如果一个请求就能获取到商品信息何必利用Selenium

**注意：本方法仅供个人娱乐学习**

**我的环境**：

- Windows10
- Python 3.6.6
- Chrome 78.0.3904.97（正式版本） （64 位）

### 安装浏览器+驱动

一般推荐Chrome+chromedriver或者Firefox+相应driver，我之前已经安装了Chrome，就用它了。如果你没安装过，百度一下！So Easy。



#### 安装驱动

---

- 安装驱动前先看chrome版本

  点击chrome右上侧菜单按钮->帮助->关于Google Chrome(G)查看浏览器版本

  ![](https://article-1300776923.file.myqcloud.com/python/chrome%E7%89%88%E6%9C%AC%E5%8F%B7.JPG)

- 打开[chromedriver下载](http://npm.taobao.org/mirrors/chromedriver/)，选择相应版本的驱动器下载。没有对应版本怎么办？找最近的或者去其他网站找到也行，我下载的驱动器版本为78.0.3904.70，得到后缀`.zip`的压缩包

- 将解压缩得到的chromedriver.exe程序放到Python的安装目录下。什么？不知道安装在哪了，问题不大。搜索框搜索python，然后鼠标右键选择打开文件所在位置，至此驱动安装完毕。

  ![](https://article-1300776923.file.myqcloud.com/python/python.JPG)

  

  

### 配置系统环境

---

选择我的电脑鼠标右键->属性->高级系统设置->环境变量->系统变量

![](https://article-1300776923.file.myqcloud.com/python/%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F.JPG)

添加变量Chrome，值为chrome的安装位置，然后保存退出即可。

![](https://article-1300776923.file.myqcloud.com/python/chrome%E4%BD%8D%E7%BD%AE.JPG)





### 安装Selenium模块

使用管理员权限打开cmd运行窗口，运行以下命令即可。

```
pip install selenium
```



### 开始写脚本

---

以京东平台的页面为例进行脚本撰写

```python
# coding=utf-8
from selenium import webdriver#导入库
from selenium.webdriver.chrome.options import Options
import smtplib
import time
import re
from email.mime.text import MIMEText
from email.utils import formataddr

#python2需要添加如下代码
# import sys
# reload(sys)
# sys.setdefaultencoding('utf-8')

'''
本脚本作者：@jianger
作者主页：https://jianger.space
版本：2.0
说明：脚本仅供业余学习辅助，请勿依赖
该脚本仅供在京东平台使用
'''

#有货则发送邮件
def sendmail(url):
    my_sender=''    # 发件人邮箱账号
    my_pass = ''    # 发件人邮箱密码
    my_user=''      # 收件人邮箱账号，我这边发送给自己
    texts='京东口罩已下单，赶紧付款吧！地址：'+url
    ret=True
    try:
        msg=MIMEText(texts,'plain','utf-8')
        msg['From']=formataddr(["xx",my_sender])  # 括号里的对应发件人邮箱昵称、发件人邮箱账号
        msg['To']=formataddr(["xx",my_user])              # 括号里的对应收件人邮箱昵称、收件人邮箱账号
        msg['Subject']="xx"                # 邮件的主题，也可以说是标题
        server=smtplib.SMTP_SSL("smtp.qq.com", 465)  # 发件人邮箱中的SMTP服务器，端口
        server.login(my_sender, my_pass)  # 括号中对应的是发件人邮箱账号、邮箱密码
        server.sendmail(my_sender,[my_user,],msg.as_string())  # 括号中对应的是发件人邮箱账号、收件人邮箱账号、发送邮件
        server.quit()  # 关闭连接
    except Exception:  # 如果 try 中的语句没有执行，则会执行下面的 ret=False
        ret=False
    return ret


class JD():
    def __init__(self,num,urls):
        self.status=True
        self.count=num
        self.urls=urls
        self.browser=webdriver.Chrome()

    #微信扫码登录
    def login(self):
        self.browser.get('https://passport.jd.com/uc/login')
        self.browser.find_element_by_css_selector('#kbCoagent > ul > li:nth-child(2) > a > span').click()

    #判断是否有货
    def isElementExist(self,element):
        ret=True
        try:
            self.browser.find_element_by_id(element)
            display=self.browser.find_element_by_id(element).get_attribute('style')
            cart=self.browser.find_element_by_id(element).get_attribute('class')
            pattern=r"btn-disable"
            m = re.search(pattern, cart)
            if display=='display: none;' and m is None:
                ret=False
        except:
            ret=True
        return ret

    #设置购买数量
    def set_count(self):
        buynum=self.browser.find_element_by_id("buy-num")
        buynum.click()
        buynum.clear()
        buynum.send_keys(self.count)

    #有货加购物车并购买
    def buy(self):
        if self.count>1:
            self.set_count()
            time.sleep(1)
        self.browser.find_element_by_id('InitCartUrl').click()
        self.browser.find_element_by_id('GotoShoppingCart').click()
        time.sleep(2)
        self.browser.find_element_by_css_selector('#cart-floatbar > div > div > div > div.options-box > div.toolbar-right.toolbar-right-new > div.normal > div > div.btn-area > a').click()
        time.sleep(2)        
        self.browser.find_element_by_css_selector('#order-submit').click()
        pageSource = self.browser.page_source
        # 断言页面源码中是否包含“商品已成功加入购物车”关键字，以此判断页面内容是否正确
        assert "订单提交成功，请尽快付款" in pageSource
        sendmail("https://cart.jd.com/")
        self.status=False


    #检查商品状态
    def check(self):
        for url in self.urls:
            self.browser.get(url)
            content=self.isElementExist('btn-notify')
            if content:
                print("还未到货，等待下一次抓取")
            else:
                sendmail(url)
                self.buy()


if __name__ == '__main__':
    urls=[
    "https://item.jd.com/xx.html",
    "https://item.jd.com/xx.html"
    ]   #商品url
    user=JD(1,urls)
    user.login()
    status=True
    num=1
    while status:
        time.sleep(40)
        print("第"+str(num)+"次查看商品状态")
        try:
            user.check()
            status=user.status
            num=num+1
        except:
            pass

```

**代码说明：**

main函数定义商品url的链接列表，一次登录，循环检查商品列表的到货状态，有一个商品买到了即程序结束。

**实现了以下功能：**

- [x] 用户登录 login()
- [x] 查看商品状态 check()
- [x] 加购物车并结算 buy()
- [x] 邮件通知付款 sendmail()

**有待完善添加：**

- [ ] 自动输入付款密码，京东使用了密码安全控件，无法直接获取到密码输入框的input元素
- [ ] 页面加载超时处理
- [ ] 页面加载失败重载处理
- [ ] 给每个商品添加状态，而不是一个完成即程序结束
- [ ] 。。。。。。