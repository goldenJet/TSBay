
## 卸载原系统中的mariadb
首先执行命令 `rpm -qa|grep mariadb` 查看是否有 mariadb 的安装包，没有可以无视

如果有的话执行 `rpm -e --nodeps mariadb-libs` 删除它

## 下载mysql 5.7 安装包
#### 下载
前往官方网站复制 yum 源链接 [【Mysql官网】](https://dev.mysql.com/downloads/repo/yum/)

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql01.png)

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql02.png)

执行 `wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm`（即你复制的下载链接）进行下载


#### 进行 yum 源安装
执行 `rpm -ivh mysql80-community-release-el7-3.noarch.rpm` 命令

接下来可以通过 `yum repolist all | grep mysql` 查看 yum 源中的 mysql 安装包

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql03.png)


## 进行 mysql 安装
在上图中可以看到 yum 源中默认启用的安装包版本为 MySQL8.0，如果需要切换为 5.7 ，需要运行以下命令；

```
yum-config-manager --disable mysql80-community
yum-config-manager --enable mysql57-community
```

如果提示命令不存在，则和 Python 版本有关系，（我的服务器上已经将原始的 2.x 版本的 Python 替换成了 3.x），懒得折腾，所以采用 planB，即直接修改配置文件：

`vim /etc/yum.repos.d/mysql-community.repo`

修改 mysql80-community 为 enabled=0
修改 mysql57-community 为 enabled=1

可以参考下图：

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql05.png)

我们再查看 yum 源中的 mysql 安装包，发现已经改成 5.7 版本了

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql04.png)


接下来可以开始进行安装步骤，执行命令
`yum install mysql-community-server`
进行安装，需要依赖安装时选择 y 就 ok 了

## 启动 mysql 服务
执行命令 `systemctl start mysqld.service` 来启动 mysql 服务，`systemctl status mysqld.service` 可查看 mysql 服务运行状态。

>小插曲，我一开始启动的时候是有报错的，不用慌，从日志里面找答案。
`less /var/log/mysqld.log`

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql06.png)

日志很明显，告诉我 3306 端口占用，查都不用查是谁占用了，想起来由于 docker 中部署的 percona 已经将宿主机的 3306 端口占用掉了，所以就退一步，改下配置文件，调整了一下端口。
修改配置文件：`vim /etc/my.cnf`
添加端口配置：`port = 3307`，注意，一定要在 [mysql] 下方添加

![](http://blogsource.chenkaikai.com/uploads/2019/11/mysql07.png)

**敲黑板：此处只修改了配置文件中的端口号，更多的优化配置下文会介绍**

## 登录 mysql

#### 初始密码
刚创建的 mysql 服务，会给 root 初始化一个随机的密码，可以在日志文件中找到，日志文件为：/var/log/mysqld.log
或者使用命令过滤出来：
`grep 'temporary password' /var/log/mysqld.log`

连接 mysql 服务：
`mysql -u root -p`
输入以上临时密码

#### 修改密码

修改密码的命令为：
`ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';`

需要注意的是：
MySQL 的 validate_password 插件会默认安装。而此插件会要求密码包含至少一个大写字母，一个小写字母，一个数字和一个特殊字符，并且密码总长度至少为8个字符。

如果你需要修改为简单密码，可以依次执行以下操作步骤:

```
set global validate_password_policy=0;
set global validate_password_length=1;
set global validate_password_mixed_case_count=2;
```

然后进行密码更改

`ALTER USER 'root'@'localhost' IDENTIFIED BY 'http://www.jetchen.cn';`

## 用户相关

#### 远程登录
默认情况下，root 用户是只能在本地登录的，如果要远程登录，则需要将 localhost 调整为 %
``` mysql
use mysql;
update user set host = "%" where user = "root";
FLUSH PRIVILEGES;
```