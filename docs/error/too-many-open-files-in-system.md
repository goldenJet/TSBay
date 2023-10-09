## 起因

最近突然在某些负载比较高的任务里遇到 MacOS 报错 Too many open files in system，就想着直接给解决掉，但是 Mac 这边和 Linux 不太一样 ~~废话~~ 。就去苹果论坛转了圈，找到了解决办法。

> 推荐使用方法二，方便快捷一劳永逸。

## 办法一
直接在终端中输入即可暂时解除

```sudo launchctl limit maxfiles <soft limit> <hard limit> ```

如：

```sudo launchctl limit maxfiles 102400 102400```

输入 `launchctl limit` 即可看到当前的限制：

``` 
❯ launchctl limit
  cpu         unlimited      unlimited      
  filesize    unlimited      unlimited      
  data        unlimited      unlimited      
  stack       8388608        67104768       
  core        0              unlimited      
  rss         unlimited      unlimited      
  memlock     unlimited      unlimited      
  maxproc     1392           2088           
  maxfiles    102400         102400  
```

但是这样解决的话不是一劳永逸的，所以就有了方法二

## 方法二
在 `/Library/LaunchDaemons` 下创建一个名为  `limit.maxfiles.plist` 的文件。

写入以下内容：

``` xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"  
        "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">  
  <dict>
    <key>Label</key>
    <string>limit.maxfiles</string>
    <key>ProgramArguments</key>
    <array>
      <string>launchctl</string>
      <string>limit</string>
      <string>maxfiles</string>
      <string>SOFT LIMIT</string>
      <string>HARD LIMIT</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>ServiceIPC</key>
    <false/>
  </dict>
</plist> 
```

记得替换上面的 `SOFT LIMIT` 和 `HARD LIMIT` 为你自己想要的值，如果你不知道填什么，无脑写 102400 就行。

然后使用 `plutil` 来验证你的 plist 文件有没有问题，不出意外，你应该可以看到如下结果：

```
❯ plutil /Library/LaunchDaemons/limit.maxfiles.plist
/Library/LaunchDaemons/limit.maxfiles.plist: OK
```

然后输入以下代码并重启电脑即可完成更改

```
sudo chown root:wheel /Library/LaunchDaemons/limit.maxfiles.plist
# 更改权限
sudo launchctl load -w /Library/LaunchDaemons/limit.maxfiles.plist
# 加载文件
```
