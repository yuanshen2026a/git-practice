## 今天做了什么

- 在 GitHub 网页修改 README，产生远程提交 acd9019
- 用 git fetch 观察 origin/main 前进
- 用 git pull --ff-only 快进同步
- 创建 temp-remote 跟踪 origin/main，练习后删除

## fetch 和 pull 的区别

- fetch：只下载远程更新，不改工作区
- pull：fetch + 合并/快进

## 远程分支

- origin/main 是远程跟踪分支
- 本地 main 跟踪 origin/main
- temp-remote 是从 origin/main 创建的临时本地分支
[200~- git branch -r 看到 origin/main
- git switch -c temp-remote origin/main 创建本地分支
- git branch -vv 查看分支和上游
- git switch main 切回
- git branch -d temp-remote 删除临时分支
