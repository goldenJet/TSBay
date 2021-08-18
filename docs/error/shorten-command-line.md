## 复现

IDEA 运行项目的时候报异常：

``` Java
Error running 'TestApplication':   
Command line is too long. Shorten command line for TestApplication or also for Spring Boot default configuration?
```

![](
http://blogsource.chenkaikai.com/uploads/2021/06/shortenCommandLine01.png)

## 原因


IDEA 启动项目的时候是使用命令启动的。

伴随着各种 classpath 参数传递给 JVM，所以这个命令会特别长。

在 Windows 系统中当超过 32767 个字符，就会报错！

所以 IDEA 建议切换到动态类路径。长类路径被写入文件，然后由应用程序启动器读取并通过系统类加载器加载。

 > 如果对实施细节感兴趣，可以查看 IDEA 社区版的源代码，JdkUtil.java 文件，setupJVMCommandLine 方法。

## 解决

- 老版本的 IDEA，去修改配置文件即可


修改项目下 `.idea\workspace.xml`，

找到标签 `<component name="PropertiesComponent">` ， 

在标签里加一行  `<property name="dynamic.classpath" value="true" />`

- 新版本的 IDEA，则直接在 UI 界面修改项目配置即可


![](
http://blogsource.chenkaikai.com/uploads/2021/06/shortenCommandLine02.png)
