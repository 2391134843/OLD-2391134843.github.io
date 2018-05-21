---
title: localhost与127.0.0.1
date: 2018-5-20 20:12:10
categories:
- web
# comments: true
---
# 关于localhost 与 127.0.0.1的介绍，以及一些小技巧
## localhost与 127.0.0.1的关系

localhost：n. 本地主机；本地服务器
其实
__localhost与127.0.0.1是一种映射关系 ：__

在计算机网络中，localhost意为"本地主机"，指"这台计算机"，是给回路网络接口(loopback)的一个标准主机名，相对应的IP地址为127.0.0.1这个名称也是一个保留域名(RFC 2606) ，为了避免同狭义定义主机名混淆而单独列出。
在可用其他方式使用计算机主机名称的地方，可以指定主机为localhost。例如，将web服务器上安装的web浏览器指向http://localhost，将会显示运行这个浏览器的计算机上所服务的网站的主页，但是只有当web服务器配置至服务回路接口时才能显示。
## 解释
使用Windows系统的朋友可以打开目录：C:\Windows\System32\drivers\etc
在这里面有一个host文件，用记事本，或者notepad++打开文件:可以看见localhost与127.0.0.1是一种映射关系。
![](http://p8i28834i.bkt.clouddn.com/localhost-1.png)
## 小拓展
我们可以修改这个host文件来达成某些有意思的功能__（修改此文件需要管理员权限，Windows下直接确认就OK）__：
举个简单例子：假如不想别人访问某个网址你可以这样修改：
![](http://p8i28834i.bkt.clouddn.com/localhost-2.png)
这样修改有什么用呢？
只要这样修改之后，你这台电脑就不能访问这个网页了，哈哈~
![](http://p8i28834i.bkt.clouddn.com/localhost-4.png)
![](http://p8i28834i.bkt.clouddn.com/localhost-5.png)
这样你就再也不用担心双十一会“剁手了~哈哈，同样也可以让别人访问不了别的网站，
__加#可以注释，也可以直接删掉这句话，这样网页又会回来的~__
## 再拓展
当我们输入网址之后，浏览器是怎么访问的？
+ 先去本地host文件中找有没有相对应的ip地址 (当我们输入localhost时候，先去host文件中找到127.0.0.1)
+ 在本地找不到之后再去外网发布消息，去DNS服务器中解析域名获得IP地址再进行访问，比如你可以这样试一下：
ping出京东的ip地址，这样我们就可以跳过DNS服务端直接去访问ip地址
![](http://p8i28834i.bkt.clouddn.com/localhost-6.png)
同样也是成功的：
![](http://p8i28834i.bkt.clouddn.com/localhost7.png)


