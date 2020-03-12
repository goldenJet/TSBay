## 进程相关

#### 端口被占用
1. 根据端口查看 PID

`netstat -ano | findstr "1080"`

2. 根据pid查看任务，或者直接起任务管理器杀掉这个进程

`tasklist | findstr "21152"`

`taskkill /f /pid 21152`

#### 查看本地的公网ip

`curl ifconfig.me`
或
`curl ipinfo.io`
