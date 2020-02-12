## 基本操作

- 进入 docker 容器内：`docker exec -it /bin/bash`
- 文件复制出来：`docker cp XXX:文件地址 本机地址`


## mysql 相关

- 最基本的创建 MySQL 容器：
```
sudo docker run --name mysql5.7 --restart always --privileged=true -p 4306:3306 -v /opt/mysql/config/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf -v /opt/mysql/data:/var/lib/mysql -e MYSQL_USER="jet" -e MYSQL_PASSWORD="pwd123" -e MYSQL_ROOT_PASSWORD="rootpwd123" -d mysql:5.7
```

- 容器内默认的配置文件路径：`/etc/mysql/mysql.conf.d/mysqld.cnf`