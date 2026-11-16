# Git 今天怎么学

> 目标不是今天硬凑 20 次 commit，而是把流程跑通。
> 今天完成第 1–5 次 commit + SSH + 分支 + 第一个 PR；剩下 15 次在接下来 3 天完成。

## 今天 90 分钟路线

| 时间 | 做什么 | 验收 |
| --- | --- | --- |
| 0–10 min | 检查 Git / SSH，配置用户名和邮箱 | `git config --global --list` 能看到 name/email |
| 10–30 min | 生成 SSH key，添加到 GitHub，测试连接 | `ssh -T git@github.com` 返回 `successfully authenticated` |
| 30–50 min | 初始化本地仓库，完成前 3 次 commit | `git log --oneline` 有 3 条记录 |
| 50–65 min | 创建分支、修改、合并 | `git log --graph` 能看到分支合并 |
| 65–90 min | 推到 GitHub，开第一个 PR 并合并 | GitHub 上能看到 1 个 merged PR |

---

## 0. 检查环境

打开 **Git Bash**（Windows 开始菜单搜索 Git Bash），逐行执行：

```bash
git --version
ssh -V
pwd
```

如果 `git` 或 `ssh` 找不到，先安装 Git for Windows：<https://git-scm.com/download/win>。

---

## 1. 配置 Git 身份

把名字和邮箱换成你自己的。邮箱建议用 GitHub 账号绑定的邮箱。

```bash
git config --global user.name "你的名字"
git config --global user.email "你的GitHub邮箱"
git config --global init.defaultBranch main
git config --global core.autocrlf true
git config --global pull.rebase false
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global --list
```

说明：

- `user.name` / `user.email` 会写进每一次 commit，必须配置。
- `init.defaultBranch main` 让新仓库默认分支叫 `main`。
- `core.autocrlf true` 适合 Windows 单机练习；如果以后和 macOS/Linux 协作，可以改成 `input`。
- `alias.lg` 是自定义快捷命令，之后可以用 `git lg` 看提交图。

---

## 2. 配置 SSH 并连接 GitHub

### 2.1 生成 SSH key

```bash
ls -la ~/.ssh
ssh-keygen -t ed25519 -C "你的GitHub邮箱"
```

一路按 Enter：

- 保存路径用默认的 `~/.ssh/id_ed25519`
- passphrase 可以留空（练习阶段），也可以设置一个自己能记住的

### 2.2 启动 ssh-agent 并添加 key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

`ssh-add -l` 能看到指纹，说明 key 已加载。

### 2.3 复制公钥

```bash
cat ~/.ssh/id_ed25519.pub
```

或者直接复制到剪贴板：

```bash
clip < ~/.ssh/id_ed25519.pub
```

### 2.4 添加到 GitHub

1. 打开 GitHub → 右上角头像 → **Settings**
2. 左侧 **SSH and GPG keys** → **New SSH key**
3. Title 填 `Windows Git Bash`，Key 粘贴刚才复制的内容
4. 点击 **Add SSH key**

### 2.5 测试连接

```bash
ssh -T git@github.com
```

第一次会问 `Are you sure you want to continue connecting?`，输入 `yes`。

成功时显示：

```text
Hi <你的用户名>! You've successfully authenticated, but GitHub does not provide shell access.
```

如果卡住或提示 `Connection timed out`，尝试 443 端口：

```bash
ssh -T -p 443 git@ssh.github.com
```

如果 443 能通，在 `~/.ssh/config` 里加：

```text
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```

然后再执行 `ssh -T git@github.com`。

---

## 3. 初始化本地仓库 + 前 3 次 commit

进入练习目录：

```bash
cd /c/Users/Administrator/Documents/deepseek-harness/default-workspace/git-practice
git init
git status
```

如果默认分支不是 `main`：

```bash
git branch -M main
```

### 第 1 次 commit：初始化仓库

```bash
git add README.md .gitignore
git status
git diff --cached
git commit -m "chore: 初始化 Git 学习日志"
```

### 第 2 次 commit：添加命令速查表

```bash
git add CHEATSHEET.md
git commit -m "docs: 添加 Git 常用命令速查表"
```

### 第 3 次 commit：添加每日笔记模板

先创建目录和文件（用你习惯的编辑器，或直接复制下面的内容）：

```bash
mkdir -p templates notes
```

`templates/daily-note.md`：

```markdown
# YYYY-MM-DD 学习记录

## 今天学了什么

## 我动手做了什么

## 卡在哪里

## 明天做什么
```

然后：

```bash
git add templates/daily-note.md
git commit -m "docs: 添加每日笔记模板"
git log --oneline
```

---

## 4. 分支 + 合并（不制造冲突）

```bash
git switch -c feature/day01-note
```

创建 `notes/day01.md`，写 5–10 行今天的学习记录，例如：

```markdown
# 第一天：Git 环境与第一次提交

- 配置了 user.name / user.email
- 生成了 SSH key，并添加到 GitHub
- 用 git init 创建了本地仓库
- 完成了前 3 次 commit

今天最大的收获：commit 前先用 git diff --cached 检查暂存区。
```

提交并合并：

```bash
git add notes/day01.md
git commit -m "docs: 添加第一天学习记录"
git switch main
git merge feature/day01-note
git branch -d feature/day01-note
git log --oneline --graph --all
```

验收：`git log --graph` 能看到一条合并线（这次通常是 fast-forward，没有分叉也正常）。

---

## 5. 推到 GitHub + 第一个 PR

### 5.1 在 GitHub 创建空仓库

1. GitHub → **New repository**
2. 名称：`git-practice`
3. 选择 Public 或 Private
4. **不要**勾选 “Add a README file”“Add .gitignore”“Choose a license”
5. 点击 **Create repository**
6. 复制 SSH 地址：`git@github.com:<你的用户名>/git-practice.git`

### 5.2 关联远程并推送

```bash
git remote add origin git@github.com:<你的用户名>/git-practice.git
git remote -v
git push -u origin main
```

刷新 GitHub 页面，应该能看到 4 次 commit。

### 5.3 新建分支，做一个改动

```bash
git switch -c feature/readme-usage
```

在 `README.md` 末尾加一段“如何使用这个仓库”，然后：

```bash
git add README.md
git commit -m "docs: 补充仓库使用说明"
git push -u origin feature/readme-usage
```

### 5.4 在 GitHub 开 PR

1. 打开仓库页面，点击 **Compare & pull request**
2. base 选 `main`，compare 选 `feature/readme-usage`
3. 标题写 `docs: 补充仓库使用说明`
4. 描述写：`补充了仓库的使用方法和学习顺序。`
5. 点击 **Create pull request**
6. 点击 **Merge pull request** → **Confirm merge**
7. 删除远程分支（可选）：**Delete branch**

### 5.5 同步回本地

```bash
git switch main
git pull
git branch -d feature/readme-usage
git log --oneline --graph --all
```

---

## 6. 今天的验收

```bash
git config --global --list
ssh -T git@github.com
git log --oneline --graph --all
git status
```

你应该能回答：

- 工作区、暂存区、本地仓库、远程仓库分别是什么？
- `git add` 和 `git commit` 各做了什么？
- `git switch -c` 和 `git merge` 各做了什么？
- PR 合并后，为什么本地要 `git pull`？

---

## 20 次 commit 清单（4 天完成）

| # | 做什么 | commit message | 完成 |
| --- | --- | --- | --- |
| 1 | 提交 README + .gitignore | `chore: 初始化 Git 学习日志` | ☐ |
| 2 | 提交 CHEATSHEET.md | `docs: 添加 Git 常用命令速查表` | ☐ |
| 3 | 创建每日笔记模板 | `docs: 添加每日笔记模板` | ☐ |
| 4 | 写第一天学习记录 | `docs: 添加第一天学习记录` | ☐ |
| 5 | 写第二天学习记录 | `docs: 添加第二天学习记录` | ☐ |
| 6 | 写第三天学习记录 | `docs: 添加第三天学习记录` | ☐ |
| 7 | README 增加“目录” | `docs: 在 README 添加目录` | ☐ |
| 8 | CHEATSHEET 补充 SSH 排错 | `docs: 补充 SSH 连接 GitHub 的排错步骤` | ☐ |
| 9 | 第一天笔记补充“常见错误” | `docs: 补充第一天笔记的常见错误` | ☐ |
| 10 | 新建 FAQ.md | `docs: 添加 Git 常见问题 FAQ` | ☐ |
| 11 | 统一笔记标题格式 | `docs: 统一笔记标题格式` | ☐ |
| 12 | 写一个统计笔记数量的小脚本 | `feat: 添加学习笔记统计脚本` | ☐ |
| 13 | 记录分支练习过程 | `docs: 添加分支练习记录` | ☐ |
| 14 | 修复脚本在子目录运行的问题 | `fix: 修复统计脚本的路径问题` | ☐ |
| 15 | 创建周复盘模板 | `docs: 添加周复盘模板` | ☐ |
| 16 | README 补充“使用方法” | `docs: 更新 README 的使用方法` | ☐ |
| 17 | 创建学习资源清单 | `docs: 添加学习资源清单` | ☐ |
| 18 | 创建 PR 检查清单 | `docs: 添加 PR 检查清单` | ☐ |
| 19 | 统一 Markdown 空行和标点 | `style: 统一 Markdown 格式` | ☐ |
| 20 | 写第一阶段复盘 | `docs: 完成第一阶段复盘` | ☐ |

建议节奏：

- **Day 1（今天）**：1–5 + SSH + 分支 + PR
- **Day 2**：6–10
- **Day 3**：11–15 + 冲突练习
- **Day 4**：16–20 + 最终 push

---

## 冲突练习（Day 3 做）

目标：故意让两个分支修改同一行，然后手动解决冲突。

```bash
git switch -c feature/readme-goal
```

把 README 的“目标”改成：

```markdown
- 每天学习 1 小时 Git
```

提交：

```bash
git add README.md
git commit -m "docs: 调整学习目标为每天 1 小时"
git switch main
```

在 `main` 上把同一行改成：

```markdown
- 每天学习 2 小时 Git
```

提交：

```bash
git add README.md
git commit -m "docs: 调整学习目标为每天 2 小时"
git merge feature/readme-goal
```

这时会出现冲突标记：

```text
<<<<<<< HEAD
- 每天学习 2 小时 Git
=======
- 每天学习 1 小时 Git
>>>>>>> feature/readme-goal
```

把这三行改成你最终想要的内容，例如：

```markdown
- 每天学习 1.5 小时 Git
```

然后：

```bash
git add README.md
git commit -m "merge: 解决 README 学习目标冲突"
git log --oneline --graph --all
```

---

## 卡住时先查这三条

```bash
git status
git diff
git log --oneline --graph --all
```

- `git status` 告诉你现在在哪个分支、哪些文件改了、哪些已暂存。
- `git diff` 看工作区还没暂存的改动；`git diff --cached` 看已经暂存的改动。
- `git log --oneline --graph --all` 看提交历史和分支关系。

不要一上来就搜“Git 怎么回退”；先看清楚状态，再决定下一步。
