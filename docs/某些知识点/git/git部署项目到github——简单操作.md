# git部署项目到github——简单操作

提交到mian分支


```git
git init

git checkout -b main

git branch -D master

git branch

git add .

git commit -m " "

git remote add origin 仓库名称

git push -u origin main
```

如果git push报main->main 错误，执行下面命令将远程分支上的文件拉到本地上来，然后在重新git push

```git
git pull origin main --allow-unrelated-histories
```