# 0 基础：先完成第一个 commit

你不需要先学 Git 才能开始。Git 不是编程语言，它更像“给文件拍快照”的工具。

## 先记住 5 个词

| 词 | 一句话解释 |
| --- | --- |
| Git | 本地版本管理工具，负责记录文件的历史快照 |
| commit | 一次“存档”，把当前改动记进本地历史 |
| GitHub | 存放 Git 仓库的网站，可以理解为“云端的代码仓库” |
| branch | 平行分支，用来在不影响 main 的情况下做实验 |
| merge | 把分支的改动合并回主线 |

注意：`git commit` 只是保存到**本地**；`git push` 才是上传到 GitHub。这两个动作不要混。

## 今天只做这一件事

在 `git-practice` 里完成第一个 commit，不碰 SSH、不碰 GitHub、不碰分支。

### 1. 配置身份

打开 Git Bash：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的GitHub邮箱"
git config --global init.defaultBranch main
git config --global --list
```

### 2. 进入练习目录

```bash
cd /c/Users/Administrator/Documents/deepseek-harness/default-workspace/git-practice
```

### 3. 初始化仓库

```bash
git init
git status
```

你会看到 `README.md`、`.gitignore` 等文件是红色的 **Untracked files**，意思是“Git 还没开始跟踪它们”。

### 4. 把文件放进暂存区

```bash
git add README.md .gitignore
git status
```

这时它们会变成绿色的 **Changes to be committed**，意思是“已经挑好，准备存档”。

### 5. 提交

```bash
git commit -m "chore: 初始化 Git 学习日志"
git status
git log --oneline
```

`git status` 显示 `nothing to commit, working tree clean`，说明当前没有未保存的改动。  
`git log --oneline` 会显示一行提交记录，前面有一串字符，那是 commit 的编号。

## 到这一步就够了

今天先感受这个循环：

```text
改文件 → git status 看状态 → git add 挑改动 → git commit 存档 → git log 看历史
```

等你觉得这个循环不陌生了，再去学 SSH、GitHub、branch、merge、PR。  
顺序反了容易懵，先让第一个 commit 成功。

## 常见问题

**Q：commit 是上传到 GitHub 吗？**  
不是。commit 只存在本地；push 才上传。

**Q：我改错了怎么办？**  
练习仓库随便折腾。还没 commit 可以用 `git restore <file>` 丢弃改动；已经 commit 可以用 `git log` 找到历史。

**Q：一定要用命令行吗？**  
初学建议用命令行，因为能看清每一步。熟悉后再用 VS Code 的 Git 面板。

**Q：我还没学 Linux，能用吗？**  
能。Git Bash 提供的就是一个类似 Linux 的命令行环境，够你学 Git 了。
