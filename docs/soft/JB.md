使用激活服务器激活JetBrains的软件，适用于JetBrains全系列软件所有新老版本，如：IntelliJ IDEA、DataGrip、GoLand、PhpStorm、PyCharm、Rider、WebStorm，支持Windows/Mac/Linux平台。

而且使用JetBrains激活服务器激活方法支持登录账号，支持在线更新，支持跨平台，支持最新版(正版激活)。

最近网上找到了一些激活服务器，分享给大家。当然，期望有能力的朋友还是支持一下正版软件。



### 1、NOVX晓星沉落提供的激活服务器

也是目前楼主在用的激活服务器。下面地址是他们发布激活服务器的页面：

[https://novx.org/jetbrains](https://novx.org/jetbrains)



### 2、Fanx大佬收集的激活服务器

下面页面会每天更新激活服务器的地址：

[https://rushb.pro/article/JetBrains-license-server.html](https://rushb.pro/article/JetBrains-license-server.html)



### 3、寻找激活服务器的方法

1. 打开网站：[Censys Search](https://search.censys.io/)

在搜索栏中输入下面代码：`services.http.response.headers.location: account.jetbrains.com/fls-auth`

 ![Snipaste_2023-07-07_10-55-15](http://blogsource.chenkaikai.com/uploads/2023/07/Snipaste_2023-07-07_10-55-15.png)


2. 点击搜索，在返回的结果随便找一个点进去，查找到 HTTP/302。
![Snipaste_2023-07-07_10-56-08](http://blogsource.chenkaikai.com/uploads/2023/07/Snipaste_2023-07-07_10-56-08.png)


3. 复制网址，运行JetBrains的软件选择许可证服务器/License server，粘贴刚刚复制的网址，激活。不行的话可以在搜索结果中再寻找一个。
