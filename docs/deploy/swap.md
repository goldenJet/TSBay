虚拟缓存 和 分页文件其实是不太一样的，但是我们普通人就认为是一个东西就行了，即简单理解成：在内存不够使用的时候，作为内存暂存的地方。

> 本文是基于 Centos 7.6

---

## 查看 swap 大小

一般的 swap 大小是实体内存的 1-2 倍，本文的案例是修改为原内存的 1.5 倍。


```
[root@jet swap]# free -h
              total        used        free       shared     buff/cache   available
Mem:       3.6G        97M        1.1G        424K        2.3G          3.2G
Swap:        0            0             0
```

上面显示 Swap 是0，即没有配置 swap

## 设置 swap 文件大小

#### 创建 /usr/swap 文件，并进入该文件

```
[root@jet ~]# mkdir /usr/swap && cd /usr/swap
```

#### 创建 6g 大小的缓存文件

```
[root@jet swap]# dd if=/dev/zero of=swapfile bs=1G count=6
6+0 records in
6+0 records out
6442450944 bytes (6.4 GB) copied, 53.456 s, 121 MB/s
```
> if 表示 infile，of 表示 outfile，bs=1G 代表增加的模块大小，count=6 代表 6 个模块，也就是 6G 的空间

#### 查看文件大小

```
[root@jet swap]# ll
total 6291460
-rw-r--r-- 1 root root 6442450944 Nov  2 14:51 swapfile
```

## 设置为 swap 文件格式

```
[root@jet swap]# mkswap /usr/swap/swapfile
Setting up swapspace version 1, size = 6291452 KiB
no label, UUID=0c021631-9ae1-4d21-b191-f9c45017ee7f
```

## 激活 swap 文件立即使用

```
[root@jet swap]# swapon /usr/swap/swapfile
swapon: /usr/swap/swapfile: insecure permissions 0644, 0600 suggested.
```

此时运行 free -h 我们可以看到 swap 已经生效了。

```
[root@jet swap]# free -h
                 total          used        free       shared    buff/cache   available
Mem:         3.6G         97M         1.1G        424K        2.3G          3.2G
Swap:         6.0G          0B           6.0G
```

**但是这样设置只是临时的 swap，reboot 后会便会失效，要想永久生效，需要再配置 fstab**

## 永久生效
删除原来 swap 类型行数据
新增一行 /usr/swap/swapfile swap swap defaults 0 0

`vi /etc/fstab`

```
#
# /etc/fstab
# Created by anaconda on Thu Jul 11 02:52:01 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=1114fe9e-2309-4580-b183-d778e6d97397 /                       ext4    defaults        1 1
/usr/swap/swapfile swap swap defaults 0 0
```

保存后即完成 swap 调整。

## 查看依赖度

```
[root@jet swap]# cat /proc/sys/vm/swappiness
0
```

swappiness 值的范围为 0-100，值越高代表对 swap 依赖程度越高，但是 swap 是基于文件储存的缓存交换机制，所以效率明显低于物理内存，swappiness 值过高的情况下容易导致物理内存远远没有耗尽便开始使用 swap；一般来说 swappiness 值可以设置为 10-60，ssd 可以设置的高一点。

如果要修改当前 swappiness 的值，可以执行：`sysctl vm.swappiness=15`，但是同样也是临时方案，重启之后便失效，要想永久有效，可以更改系统配置 `echo "vm.swappiness = 15" >> /etc/sysctl.conf`