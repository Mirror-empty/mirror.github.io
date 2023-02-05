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


### 记录git提交到github中错误

错误原因：

```java
ssh: Could not resolve hostname github.com:Mirror-empty: Name or service not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
1. 将.ssh文件下known_hosts文件删掉
2. ``ssh-keygen -t rsa -C`` "邮箱"  重新生成ssh
3. 打开id_rsa.pub复制密钥到github上去
4. ``git remote set-url origin ssh连接``  如果提交不上去需要重新设置一下，``git remote -v``查看连接