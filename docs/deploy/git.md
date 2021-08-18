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