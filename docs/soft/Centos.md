## 查看系统版本

```
[root@jet local]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core) 
[root@jet local]# uname -a
Linux jet 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## 上传下载
lrzsz 着实是有点方便的

安装：`yum -y install lrzsz`

上传： **rz**，    
下载： **sz**

## JDK 安装

#### 检查 jdk
`java -verison`

```
[root@jet ~]# java -version
-bash: java: command not found
```

#### 下载 jdk
[下载地址](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)
![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation01.png)

#### 上传至服务器
```
[root@jet ~]# ll
total 177356
-rw-r--r-- 1 root root 181608904 Nov  3 10:32 jdk-11.0.9_linux-x64_bin.tar.gz
```

#### 解压

`tar -zxvf jdk-11.0.9_linux-x64_bin.tar.gz`
```
[root@iZuf69te1lxhy77fkuh7grZ ~]# mkdir /usr/java
[root@iZuf69te1lxhy77fkuh7grZ ~]# cp jdk-11.0.9_linux-x64_bin.tar.gz /usr/java/
[root@iZuf69te1lxhy77fkuh7grZ ~]# cd /usr/java/
[root@iZuf69te1lxhy77fkuh7grZ java]# ll
total 177356
-rw-r--r-- 1 root root 181608904 Nov  3 11:23 jdk-11.0.9_linux-x64_bin.tar.gz
[root@iZuf69te1lxhy77fkuh7grZ java]# tar -zxvf jdk-11.0.9_linux-x64_bin.tar.gz 
```

#### 修改环境变量
修改环境变量并在末尾添加配置即可
`vim /etc/profile`

```
# java environment
export JAVA_HOME=/usr/java/jdk-11.0.9
export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
```
![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation02.png)

#### 生效
`[root@jet jdk-11.0.9]# source /etc/profile`

然后检查一下，发现 jdk11 的环境已经装好了

```
[root@jet jdk-11.0.9]# java -version
java version "11.0.9" 2020-10-20 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.9+7-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.9+7-LTS, mixed mode)
```

## python

#### 检查
`python -V`

```
[root@jet local]# python -V
Python 2.7.5
```

#### 安装依赖包
`yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make`

#### 下载源码包

`wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz`

#### 解压
`tar -zxvf Python-3.7.0.tgz`
```
[root@jet soft]# ll
total 22220
drwxr-xr-x 18  501  501     4096 Jun 27  2018 Python-3.7.0
-rw-r--r--  1 root root 22745726 Jun 27  2018 Python-3.7.0.tgz
```

#### 安装
`cd Python-3.7.0`

先执行配置文件 `./configure`

然后再编译安装 `make&&make install`

编译安装的时候如果报错：**“ModuleNotFound：No module named '_ctypes'”** 

![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation03.png)

则需要安装模块：`yum install libffi-devel -y`

然后再执行：`make&&make install`

#### 软连接

此时 python3 便已经安装好了，默认情况下，python3.7 安装在 /usr/local/bin/，但是命令一般都放在 /usr/bin/ 文件夹下，所以我们需要添加一条软连，（记得先把原先的备份）：

```
[root@jet Python-3.7.0]# mv /usr/bin/python /usr/bin/python.bak
[root@jet Python-3.7.0]# ln -s /usr/local/bin/python3 /usr/bin/python
```

同样，pip 最好也替换成 pip3：（不换问题也不大）

```
[root@jet Python-3.7.0]# mv /usr/bin/pip /usr/bin/pip.bak
[root@jet Python-3.7.0]# ln -s /usr/local/bin/pip3 /usr/bin/pip
```

#### 验证

```
[root@jet Python-3.7.0]# python -V
Python 3.7.0
[root@jet Python-3.7.0]# pip -V
pip 10.0.1 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)
[root@jet Python-3.7.0]# python
Python 3.7.0 (default, Nov  3 2020, 13:12:08) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit();
[root@jet Python-3.7.0]# 
```

#### 修改 yum

yum 是依赖 python2.7 的，我们把 python 改成了 3.7 版本了，自然不好使了。但是不用担心，python2.7 还在你的系统里。只要修改一下 yum 里的相关依赖即可。

`vi /usr/libexec/urlgrabber-ext-down`

![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation04.png)

`vi /usr/bin/yum`

![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation05.png)

## git 安装

#### 检查
`git --version`

```
[root@jet Python-3.7.0]# git --version
-bash: git: command not found
```

#### 安装

cetos7 以及以上版本的，推荐使用 yum 安装，除了方便以外的重要的原因是：centos5 及以下的版本，yum 没有 git，centos6 的 yum 虽然有 git，但是版本是 1.7.1，而 github 需要的 git 版本最低都不能低于 1.7.2。

`yum -y install git`

```
[root@jet Python-3.7.0]# git --version
git version 1.8.3.1
```

> 移除 git：`yum remove git`

## nginx 安装

nginx 有三种类型的版本，[链接](http://nginx.org/en/download.html)：

- Mainline version（主力开发版本）
- Stable version （最新的稳定版本，建议生产使用）
- Legacy versions （遗留的老版本的稳定版）


#### 环境安装

Nginx 是 C 语言 开发，建议在 Linux 上运行，当然，也可以安装 Windows 版本，本篇则使用 CentOS 7 作为安装环境。

- gcc 安装
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
`yum install gcc-c++`

- PCRE pcre-devel安装
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
`yum install -y pcre pcre-devel`

- zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
`yum install -y zlib zlib-devel`

- OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
`yum install -y openssl openssl-devel`

#### 下载

目前最新的稳定版本是 1.18.0

`wget -c https://nginx.org/download/nginx-1.18.0.tar.gz`

#### 解压

`tar -zxvf nginx-1.18.0.tar.gz`

```
[root@jet soft]# tar -zxvf nginx-1.18.0.tar.gz
[root@jet soft]# cd nginx-1.18.0
```

#### 配置&安装

运行命令进行配置：`./configure`

编译：`make`

安装：`make install`

查看安装路径：`whereis nginx`

```
[root@jet soft]# whereis nginx
nginx: /usr/local/nginx
```

#### 启动

- 进入安装路径：`cd /usr/local/nginx/sbin/`
- 启动：`./nginx `
- 停止：`./nginx -s stop`
- 退出：`./nginx -s quit`
- 重新加载配置：`./nginx -s reload`

#### 开机自启动

在 rc.local 增加启动代码就可以了，记得授权。

`vi /etc/rc.local`

增加一行 `/usr/local/nginx/sbin/nginx`

![](http://blogsource.chenkaikai.com/uploads/2020/11/Centos-environment-installation07.png)

设置执行权限：`chmod 755 /etc/rc.local`