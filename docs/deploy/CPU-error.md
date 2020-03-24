## 背景

最近测试服出现了CPU异常高的情况，占用率接近 100%，所以写篇文章简单地记录下碰到这种情况，该如何去定位导致CPU异常的代码，下文介绍了几种比较常用的工具。

下文均基于测试代码。

## 准备
我们先准备一个测试项目，此处使用的是一个简单的 springboot 的 web 项目，直接跑去官网初始化一个，地址：
[地址](https://start.spring.io/)，然后写了段简单的示例代码，见下图。

![示例代码](http://blogsource.chenkaikai.com/uploads/2020/03/CPU14.png)

打包后放到我本地的虚拟机上运行：`nohup java -jar cputest-0.0.1-SNAPSHOT.jar &`

在虚拟机上进行本地访问，运行死循环的那段逻辑：`curl 127.0.0.1:7777/api/test`

此时使用 `top -c` 查看服务器的 CPU 信息，发现已经使用了 97%，而且是被我们刚才启动的 java 进程占用的，**PID 为 5008**。

> COMMAND 注释就是 ‘java -jar cputest-0.0.1-SNAPSHOT.jar’，显而易见。

![CPU详情](http://blogsource.chenkaikai.com/uploads/2020/03/CPU03.png)

下面就介绍几种简单的方法来定位有问题的代码。

## jstack 查看堆栈信息

此处原生的方法其实就是使用 jdk 自带的工具（**需要注意的是：Linux自带的 openJdk 是没有的**）。

这是最基础并且最有效的方法，因为很多其它的工具其实还是使用的 openJdk 的这一系列命令，只不过封装得更为友好了而已。

上文已经查出来是5008这个进程有问题。

所以我们就先查看下5008下面是哪些线程在搞鬼：`top -Hp 5008`

![线程PID查询](http://blogsource.chenkaikai.com/uploads/2020/03/CPU04.png)

从面板上我们乍一看，发现了三个异常的线程，PID分别为5010、5092、5093。

要使用 jstack 查看他们的堆栈信息，首先要将 PID 转为 16进制：`printf "%x\n" 5010 5092 5093`

![线程PID转16进制](http://blogsource.chenkaikai.com/uploads/2020/03/CPU05.png)

接下来就可以查看堆栈信息了，下面以5092这个为例（**注意：16进制要在最前面添加0x**）：`jstack 5008 | grep '0x13e4' -A10`

![jstack查看异常代码](http://blogsource.chenkaikai.com/uploads/2020/03/CPU06.png)

问题代码立马就找到了。

> 另外，我么也可以将堆栈信息打印下来：`jstack 5008 > cputest.dump`  
> 然后使用 cat 等命令工具进行过滤查看：`cat -n cputest.dump | grep -A10 '0x13e4'`  

GC 信息也是有问题的：`jstat -gcutil 5008 2000 5`

> 比如老年代（O）使用率达到了 84%，  
> 元数据区（M）使用率达到了 94%，  
> 老年代回收次数（FGC）达到了 540 多  

![GC信息](http://blogsource.chenkaikai.com/uploads/2020/03/CPU09.png)

## Arthas（阿尔萨斯）

上文的方法比较繁琐，下面就使用目前特别火的工具来排查下问题。

Arthas 是阿里巴巴开源的一个针对 java 程序的线上诊断工具，全平台支持，有命令行，也有web端，功能非常强大，官网：[地址](https://alibaba.github.io/arthas/install-detail.html)。

我们先安装基础工具：`curl -O https://alibaba.github.io/arthas/arthas-boot.jar`

> 也可以从 gitee 下载：`curl -O https://arthas.gitee.io/arthas-boot.jar`

运行：`java -jar arthas-boot.jar`

> Arthas 会罗列出当前服务器上面的所有 Java 进程，我们选择一个我们想要查看的就行，比如此处输入1即可。  
> 另外，首次运行是需要下载一些依赖工具的，如果无法下载，也可以使用全量安装。  
> 此时左下角也已经变成了 [arthas@5008]，这个5008 就是我们想要查看的程序继进程

![开启Arthas](http://blogsource.chenkaikai.com/uploads/2020/03/CPU10.png)

控制台输入 `dashboard` 即可查看：

![Arthas面板](http://blogsource.chenkaikai.com/uploads/2020/03/CPU11.png)

从上面的面板上，其实很容易发现有两个异常的线程，ID 分别为 28 和 27，另外还有异常的 GC 信息。

输入 `Ctrl c` 退出面板，接下来我们就来为ID为27的这个线程把把脉，输入 `thread 27` 查看线程的详细信息：

![Arthas查看异常代码](http://blogsource.chenkaikai.com/uploads/2020/03/CPU12.png)

问题代码就这样被定为了，so cool ！

输入 `quite` 即可退出 Arthas。

> 另外，Arthas 的功能十分强大，远不限于此，比如自带 jad 反编译工具等，此处暂不赘述


## show-busy-java-threads

下面的这个工具，也是很常用并且很强大的，而且 show-busy-java-threads 只是 
useful-scripts 工具包中的工具之一。

下载：[地址](https://github.com/oldratlee/useful-scripts/releases)

上传 bin 文件夹下的 show-busy-java-threads 脚本到服务器上去。

赋予权限：`chmod +x show-busy-java-threads`

运行：`./show-busy-java-threads`

![Arthas查看异常代码](http://blogsource.chenkaikai.com/uploads/2020/03/CPU13.png)

从上图中就看到了异常代码的出处。

> 其他命令：
>1. show-busy-java-threads -c 3：3为n，指定显示最耗cpu使用率前3的线程。
>2. show-busy-java-threads -c 3 -p 17376：展示进程17376耗费CPU组多的3个线程。


## 总结

此文简单介绍了三种排查服务器CPU负载高的方案，其实使用体验都还是比较简单的，但是有两点是万变不离其宗的，那就是**底层工具的使用**和**解决问题的流程**。

从打印的堆栈信息中我们就可以看出，信息都长的差不多，所以也可以看出，底层所使用的工具都一样，其实都是 jdk 的 jstack 等工具。

还有，此类问题一般的解决流程是：

> 1. 查看 CPU 高负载的进程
> 2. 查看具体的线程
> 3. 找到对应进程的堆栈信息
> 4. 在上述的堆栈信息中过滤出对应线程的信息