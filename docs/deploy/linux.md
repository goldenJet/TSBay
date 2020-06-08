## 基本操作

#### curl 请求过程：

`curl -o /dev/null -s -w %{http_code}:%{time_namelookup}:%{time_redirect}:%{time_pretransfer}:%{time_connect}:%{time_starttransfer}:%{time_total}:%{speed_download} http://www.jetchen.cn/`
