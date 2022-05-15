## 基本操作

#### curl 请求过程：

`curl -o /dev/null -s -w %{http_code}:%{time_namelookup}:%{time_redirect}:%{time_pretransfer}:%{time_connect}:%{time_starttransfer}:%{time_total}:%{speed_download} http://www.jetchen.cn/`

#### crontab 日志

`tail -n 50 /var/spool/mail/root` 查看最后50行执行日志


#### 统计文件夹大小

***查看当前路径下占用磁盘空间最大的 10 个文件/文件夹：*** `du -hsx * | sort -rh | head -20`


***查看指定目录下的所有子目录的大小：*** `du -h /home --max-depth=1`

