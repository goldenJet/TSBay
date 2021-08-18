TReasure Bay


[【官网】](https://docsify.js.org/#/zh-cn/)

## 安装
安装客户端：
`npm i docsify-cli -g`

## 初始化
初始化项目，例如项目目录为TSBay：
`docsify init ./TSBay`

然后就会生成一个 TSBay 的文件夹，文件夹内文件为：
>index.html 入口文件 
README.md 会做为主页内容渲染 
.nojekyll 用于阻止 GitHub Pages 会忽略掉下划线开头的文件

## 运行
服务运行命令：
`docsify serve ./TSBay`

或者进入文件夹内：
`docsify serve`

或者进入目录下然后使用 python 启动：
2.7 版本：`python -m SimpleHTTPServer 3000`
3.x 版本：`python -m http.server 3000`
或 直接一行命令进入文件夹下并且保持后台运行
`cd "D:\project\mixed\temp\TSBay" & nohup python -m http.server 3000 &`

> 上述命令的应用场景：部署在 VPS 上

访问：
`http://localhost:3000`

## 部署

初始化本地 git 库：
`git init`

提交代码：
`git add .`
`git commit -m 'test commit'`

如果部署至 github page

先去 github 新建一个项目，如果要使用 github pages 的话，项目名需要取 xx.github.io，例如：TSBay.github.io

然后将本地的 git 仓库和远程的 git 仓库关联：
` git remote add origin 'https://github.com/goldenJet/TSBay.github.io.git'`

` git pull --rebase origin master`

提交到远程仓库：
` git push -u origin master`