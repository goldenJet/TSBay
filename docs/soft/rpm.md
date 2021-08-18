## rpm 包的安装      
1. 安装一个包　

`rpm -ivh xxx`

2. 升级一个包　

`rpm -Uvh xxx`

3. 移走一个包　　

`rpm -e xxx`

4. 安装参数　　

- --force 即使覆盖属于其它包的文件也强迫安装
- --nodeps 如果该RPM包的安装依赖其它包，即使其它包没装，也强迫安装。　　

5. 查询一个包是否被安装　　

`rpm -q < rpm package name>`

6. 得到被安装的包的信息　　

`rpm -qi < rpm package name>`

7. 列出该包中有哪些文件　　

`rpm -ql < rpm package name>`

8. 列出服务器上的一个文件属于哪一个RPM包　

`rpm -qf xxx`

9. 可综合好几个参数一起用　　

`rpm -qil < rpm package name>`

10. 列出所有被安装的rpm package　　

`rpm -qa`

11. 列出一个未被安装进系统的RPM包文件中包含有哪些文件？　　

`rpm -qilp < rpm package name>`

## rpm 包的卸载

- `rpm -qa | grep 包名`     这个命令是为了把包名相关的包都列出来          

- `rpm -e 文件名`    这个命令就是你想卸载的软件，后面是包名称，最后的版本号是不用打的   

例如：     

```
[root@jet ~]#  rpm -qa | grep mysql
mysql80-community-release-el7-3.noarch
```

卸载 mysql 的包只需要执行：`rpm -e mysql`

## yum 安装     

`yum install 包名`

## yum 卸载       

`yum -y remove 包名`