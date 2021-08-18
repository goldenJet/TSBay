项目编译的时候控制台报异常：

`java: Compilation failed: internal java compiler error`

这个是 IDEA 设置的问题，解决方向有两个：

1. 调整 IDEA 中的 jdk 的版本
2. 调整 IDEA 中的堆内存大小

## 设置 JDK 版本

![](http://blogsource.chenkaikai.com/uploads/2021/06/compilationFailed01.png)

![](http://blogsource.chenkaikai.com/uploads/2021/06/compilationFailed02.png)

![](http://blogsource.chenkaikai.com/uploads/2021/06/compilationFailed03.png)

## 设置堆内存

如果还还报错，那么就有可能是项目太大了，堆内存太小。

比如将 700 调整成 1024，具体可以视情况而定。

![](http://blogsource.chenkaikai.com/uploads/2021/06/compilationFailed04.png)
