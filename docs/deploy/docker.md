## 基本操作


#### 安装

推荐 Ubuntu 系统，但是我是装在 Centos7 上面的，所以下文所有操作都是在 Centos 7上，在 6 上面安装会有很大的坑。

- 安装：`yum install -y docker`

- 重启服务：`service docker restart` 

- 查看版本：`docker version`

#### 基操

- 镜像列出所有镜像：`docker images`

- 删除镜像：`docker rmi [镜像名/image id]`

- 下载镜像：`docker pull mysql` 或者 `docker pull mysql:5.7`  mysql 后面可以加版本号和重命名

- 搜索镜像：`docker search mysql`    

- 容器查看所有容器：`docker ps -a`

- 查看正在运行的容器：`docker ps`

- 创建容器：`docker create –name percona -v /data/mysql-data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root percona`

> 说明：  
> –name 是命名这个容器，  
> -v 是指定数据卷（本机：容器），  
> -p 端口映射（本机实际：docker内），  
> -e 参数设置，最后一个percona是镜像的名字  

- 启动/停止/重启容器：`docker start/stop/restart [容器名]`

- 创建并启动容器：`docker run -d …`

- 进入容器：`docker exec -it [容器名] /bin/bash`

- 查看日志：`docker logs -f [容器名]`

- 文件复制出来：`docker cp XXX:文件地址 本机地址`

#### 镜像加速
默认 pull 的是从官方的仓库，在国内，可以找一个更快的国内源，如阿里云的：[https://dev.aliyun.com/search.html](https://dev.aliyun.com/search.html)

两种方式：

1. 直接从指定仓库 pull 如：`docker pull registry.cn-hangzhou.aliyuncs.com/acs-sample/mysql:5.7`  
2. 在阿里云的容器服务内（[https://cr.console.aliyun.com/#/accelerator](https://cr.console.aliyun.com/#/accelerator)），我们可以查到阿里云的docker镜像加速器的使用，以下是我个人的加速器配置，前提是，Docker 客户端版本大于 1.10.0 可以通过修改 daemon 配置文件 /etc/docker/daemon.json 来使用加速器，直接运行下面的代码即可：

```
sudo mkdir -p /etc/docker 
sudo tee /etc/docker/daemon.json <<-'EOF' 
{ 
  "registry-mirrors": ["https://6jbywkhi.mirror.aliyuncs.com"] 
} 
EOF 
sudo systemctl daemon-reload 
sudo systemctl restart docker
```

## 镜像制作

以一个最简单的 demo 为例：

需求：springboot 打包了一个 jar 程序，现在要使用 docker 制作一个镜像来运行这个程序。

实现：

1. 创建项目用的文件夹并上传 jar 包至此文件夹内

![](http://blogsource.chenkaikai.com/uploads/image/20171123/1511426103753590.png)

2. 制作关键文件 Dockerfile

`vim Dockerfile`

内容如下：

```
FROM java:8
COPY ./wechat-0.0.1-SNAPSHOT.jar /wechat/wechat-0.0.1-SNAPSHOT.jar
COPY ./app-entrypoint.sh /
RUN chmod +x /app-entrypoint.sh
EXPOSE 9092

ENTRYPOINT ["/app-entrypoint.sh"]
```

> 说明：
> FROM：基础镜像是 java 的 8 版本（因为我们运行 jar 程序需要 java 运行环境）
> COPY：将当前文件夹下的 jar 包 复制到 镜像中的 /wechat/ 目录下
> COPY：将当前目录下的部署程序的shell脚本上传到 / 目录下
> RUN： 赋予权限，然后运行我们上传的运行脚本
> EXPOSE：docker 容器暴露出来的 端口
> ENTRYPOINT： 入口，即docker 启动之后执行的脚本

3. 制作启动脚本

`vim app-entrypoint.sh`

内容如下：

```
#!/bin/bash
java -jar /wechat/wechat-0.0.1-SNAPSHOT.jar
```

> 说明：第一行么就是 sh 脚本必备的，第二行就是执行 jar 程序必须的操作

4. 关键：镜像制作

由于我们当前目录下只有上文提到的三个文件，如下图：

![](http://blogsource.chenkaikai.com/uploads/image/20171123/1511426788936616.png)

执行： `docker build -t wechat:1.0.0 .`

> 说明：
> wechat:1.0.0 是自定义的镜像的名字和版本
> . 就是代表当前文件夹下的所有文件

运行 `docker images` 便可以看到我们的镜像了

5. 创建容器并运行

```
docker create –name wechat -t -p 9092:9092 wechat:1.0.0  
docker start wechat && docker logs -f wechat
```

运行 `docker ps` 便可以查看所有正在允许的镜像

6. 镜像推送到仓库

## mysql 相关

- 最基本的创建 MySQL 容器：
```
sudo docker run --name mysql5.7 --restart always --privileged=true -p 4306:3306 -v /opt/mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf -v /opt/mysql/data:/var/lib/mysql -e MYSQL_USER="jet" -e MYSQL_PASSWORD="pwd123" -e MYSQL_ROOT_PASSWORD="rootpwd123" -d mysql:5.7
```

- 容器内默认的配置文件路径：`/etc/mysql/mysql.conf.d/mysqld.cnf`

## 踩坑

直接见 [http://www.jetchen.cn/docker%e5%b8%b8%e7%94%a8%e6%8a%80%e5%b7%a7/](http://www.jetchen.cn/docker%e5%b8%b8%e7%94%a8%e6%8a%80%e5%b7%a7/)