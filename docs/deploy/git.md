## 修改 Commit 记录的用户名 Name 和邮箱 Email

背景是在 github 上面提交代码的时候，个人首页的日历图上面会显示绿色，但是前提是提交人的邮箱是在 github 上面配置过的邮箱，但是如果出现提交记录的邮箱不在 github 配置的邮箱列表中的时候，首页的日历图上面就没有绿色小点点，此时怎么办？我们可以通过修改下历史的提交记录的邮箱来实现。

查看记录：`git log`

配置本地的邮箱：`git config user.email 'chen_kaikai777@126.com'`

编写 `email.sh` 脚本：

记得修改脚本中的邮箱

``` bash
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="jet.chen@wailianvisa.com"
CORRECT_NAME="jet.chen"
CORRECT_EMAIL="chen_kaikai777@126.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

执行脚本，然后提交到远程：`git push origin --force --all`

## 阿里云服务器连接不上 github

不整翻墙的工具，直接修改 host 文件临时解决下。

1. 在网站：https://github.com.ipaddress.com/ 上面分别查询出 **github.com** 和 **github.global.ssl.fastly.net** 的 ip，
2. vim /etc/hosts，修改 hosts 文件，添加：

``` bash
140.82.114.4 github.com
199.232.69.194 github.global.ssl.fastly.net
```

## git 提交代码统计

- 仓库提交者排名前五：

``` bash
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```


- 统计每个人的增删行数：

``` bash
git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```


- git log 参数说明

``` bash
--author   指定作者
--stat   显示每次更新的文件修改统计信息，会列出具体文件列表
--shortstat    统计每个commit 的文件修改行数，包括增加，删除，但不列出文件列表：  
--numstat   统计每个commit 的文件修改行数，包括增加，删除，并列出文件列表：
   
-p 选项展开显示每次提交的内容差异，用-2 则仅显示最近的两次更新,例如：git log -p  -2
--name-only 仅在提交信息后显示已修改的文件清单
--name-status 显示新增、修改、删除的文件清单
--abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符
--relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）
--graph 显示 ASCII 图形表示的分支合并历史
--pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）,例如： git log --pretty=oneline ; git log --pretty=short ; git log --pretty=full ; git log --pretty=fuller
--pretty=tformat:   可以定制要显示的记录格式，这样的输出便于后期编程提取分析
       例如：git log --pretty=format:""%h - %an, %ar : %s""
       下面列出了常用的格式占位符写法及其代表的意义。                   
       选项       说明                  
       %H      提交对象（commit）的完整哈希字串               
       %h      提交对象的简短哈希字串               
       %T      树对象（tree）的完整哈希字串                   
       %t      树对象的简短哈希字串                    
       %P      父对象（parent）的完整哈希字串               
       %p      父对象的简短哈希字串                   
       %an     作者（author）的名字              
       %ae     作者的电子邮件地址                
       %ad     作者修订日期（可以用 -date= 选项定制格式）                   
       %ar     作者修订日期，按多久以前的方式显示                    
       %cn     提交者(committer)的名字                
       %ce     提交者的电子邮件地址                    
       %cd     提交日期                
       %cr     提交日期，按多久以前的方式显示              
       %s      提交说明  
       
--since  限制显示输出的范围，
       例如： git log --since=2.weeks    显示最近两周的提交
       选项 说明                
       -(n)    仅显示最近的 n 条提交                    
       --since, --after 仅显示指定时间之后的提交。                    
       --until, --before 仅显示指定时间之前的提交。                  
       --author 仅显示指定作者相关的提交。                
       --committer 仅显示指定提交者相关的提交。
```




