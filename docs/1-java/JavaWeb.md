# JavaWeb

## 1、基本概念

### 1.1、前言

web开发：

- web，网页的意思

- 静态web

  。 html，css

  。提供给所有人看的数据始终会发生变化！

  

  动态web

  。淘宝，几乎所有的网站

  。 提供给所有人看的数据始终会发生变化！每个人在不同的时间，不同的地点看到的信息各不相同！

  。技术栈：Servlet/JSP,ASP,PHP

  在java中，动态资源开发的技术统称为javaweb

  

  

  

  

  ### 1.2、web应用程序

  web应用程序：可以提供浏览器访问的程序

  . a.html、b.html.......多个web资源，这些web资源可以被外界访问，对外界提供服务

  . 你们能访问到的任何一个页面或者资源，都存在这个世界的某一个角落的计算机上

  . URL

  . 这个统一的web资源会被放在同一个文件夹下,web应用程序-->Tomcat:服务器

  . 一个web应用由多个部分组成（静态web，动态web）

  。 html、csss、js\

  。jsp、servlet

  。java程序

  。jar包

  。配置文件（Properties）

  web应用程序编写完毕后，若想提供给外界访问，需要一个服务器来统一管理;

  ### 1.3、静态web

  。 *.html，*.htm这些都是网页的后缀，如果服务器上一直存在这些东西，我们就可以直接进行读取。通络

  

  ![image-20210712154635339](JavaWeb.assets/image-20210712154635339.png)

  - 静态web存在的缺点

    . web页面无法动态更新，所有用户看到的都是一个页面

    轮播图，点击特效：伪动态

    javaScript[实际开发中，它用的最多]

    VBScript

    它无法和数据库交互（数据无法持久化，用户无法交互）

    ### 1.4、动态web

页面会动态展示："web的页面展示的效果因人而异"

![image-20210712155643651](JavaWeb.assets/image-20210712155643651.png)

缺点

- 加入服务器的动态web资源出现问题，我们需要重新编写我们的后台程序，重新发布

   。停机维护

  优点：

  - web页面可以动态刷新，所有用户看到的都不是同一个页面
  - 他可以与数据库交互(数据持久化：注册，商品信息，用户信息等等)
  - 

![image-20210712160133905](JavaWeb.assets/image-20210712160133905.png)

## 2、web服务器

### 2.1、技术讲解

ASP：

- 微软：国内最早流行的就是asp

- 在html中嵌入了vb的脚本，asp+COM

- 在asp开发中，基本一个页面几千行的代码，页面及其混乱

- 维护成本高

- C#

- IIS

  

php

- PHP开发速度很快，功能强大，跨平台，代码很简单

- 无法承载大访问量 的情况（局限性）

   



jsp/Servlet

B/S:浏览器和复位器

C/S:客户端和服务器

- sun公司主推的B/S架构

- 基于java语言的（所有的大公司，或者一些开源的组件，都是用java写的）

- 可以承载三高问题带来的影响

- 语法像asp，asp->jsp,加强市场强度

  。。。。。

  

### 2.2、web服务器

服务器是一种被动的操作，用来处理一些请求和给用户一些响应信息

IIS

微软的：ASP，，，，windows中自带的

Tomcat

。。。

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由[Apache](https://baike.baidu.com/item/Apache/6265)、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，Tomcat 5支持最新的Servlet 2.4 和JSP 2.0 规范。因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用[服务器](https://baike.baidu.com/item/服务器)，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应[HTML](https://baike.baidu.com/item/HTML)（[标准通用标记语言](https://baike.baidu.com/item/标准通用标记语言/6805073)下的一个应用）页面的访问请求。实际上Tomcat是Apache 服务器的扩展，但运行时它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。

诀窍是，当配置正确时，Apache 为HTML页面服务，而Tomcat 实际上运行JSP 页面和Servlet。另外，Tomcat和[IIS](https://baike.baidu.com/item/IIS)等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态[HTML](https://baike.baidu.com/item/HTML)的能力不如Apache服务器。目前Tomcat最新版本为10.0.5**。**







# 3、Tomcat



## 3.1、安装Tomcat





## 3.2 Tomcat启动和配置

文件夹作用





![image-20210712165105882](JavaWeb.assets/image-20210712165105882.png)

启动和关闭

![image-20210712170134699](JavaWeb.assets/image-20210712170134699.png)

访问测试：

http://localhost:8080/

可能遇到的问题：

1.java环境变量没有配置

2.闪退问题：需要配置兼容性

3.乱码问题：配置文件设置



![image-20210712170523959](JavaWeb.assets/image-20210712170523959.png)





可以配置启动的端口号

- tomcat的默认端口号为：8080

- nysql:3306

- http:80

- https:443

  

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```


可以配置主机的名称

- 默认的主机名为：localhost->127.0.0.1
- 默认网站应用存放的位置为：webapps

```xml
  <Host name="localhost"  appBase="webapps"
        unpackWARs="true" autoDeploy="true">
```
高难度面试题：

请你谈谈网站是如何进行访问的！

1.输入域名,回车

2.检查本机的C:\Windows\System32\drivers\etc\hosts配置文件下有没有这个域名映射；

​	1、有：直接返回对应的ip地址，有我们需要访问的web程序，可以直接访问

​	2.没有：去DNS服务器找，找到的话返回，找不到就返回找不到





![image-20210712172627116](JavaWeb.assets/image-20210712172627116.png)

可以配置一下环境白你两（可选项）

### 3.4、发布一个web网站

不会就先模仿

将自己写的网站，放到服务器中指定的web应用的文件夹（webapps）下，就可以访问了

网站应该有的结构

```
--webapps:tomcat服务器的web目录
	-ROOT
	-jie:网站的目录名
		-WEB-INFO
			-classes：java程序
			-lib：web应用所依赖的jar包
			-web.xml
		-index.html 默认的首页
		-static
			-css
				-style.css
			-js
			-img
		......
```









