# Git / GitHub 常用命令速查

> 先看懂 `git status`，再决定下一步。大多数“Git 出问题了”，其实是没看清当前状态。

## 1. 初始配置

```bash
git config --global user.name "你的名字"
git config --global user.email "你的GitHub邮箱"
git config --global init.defaultBranch main
git config --global core.autocrlf true
git config --global pull.rebase false
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global --list
```

查看某个配置：

```bash
git config --global user.name
git config --global user.email
```

## 2. SSH 连接 GitHub

```bash
# 1. 生成 key（如果已有 ~/.ssh/id_ed25519，可跳过）
ssh-keygen -t ed25519 -C "你的GitHub邮箱"

# 2. 启动 agent 并加载 key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l

# 3. 复制公钥
cat ~/.ssh/id_ed25519.pub
clip < ~/.ssh/id_ed25519.pub   # Git Bash 可直接复制到剪贴板

# 4. 测试
ssh -T git@github.com
```

成功输出：

```text
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

端口 22 超时，改用 443：

```bash
ssh -T -p 443 git@ssh.github.com
```

`~/.ssh/config`：

```text
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```

## 3. 每天最常用的 6 条

```bash
git status                 # 现在是什么状态
git diff                   # 工作区改了什么（未暂存）
git diff --cached          # 暂存区改了什么
git add <file>             # 把指定文件放进暂存区
git commit -m "type: 说明" # 提交
git log --oneline --graph --all
```

推荐流程：

```bash
git status
git diff
git add <file>
git diff --cached
git commit -m "docs: 添加第一天学习记录"
git log --oneline -5
```

## 4. 查看历史

```bash
git log --oneline
git log --oneline --graph --decorate --all
git lg                         # 如果配置了 alias
git show <commit-hash>
git diff HEAD~1 HEAD           # 最近两次提交的差异
git diff --stat                # 只看哪些文件变了
```

## 5. 分支与合并

```bash
git branch                     # 本地分支
git branch -a                  # 本地 + 远程分支
git switch -c feature/xxx      # 新建并切换
git switch main                # 切回 main
git merge feature/xxx          # 把 feature 合进当前分支
git branch -d feature/xxx      # 删除已合并的分支
git branch -D feature/xxx      # 强制删除未合并分支（确认不要了再用）
```

合并冲突时：

```bash
git status                     # 看哪些文件冲突
# 打开冲突文件，删除 <<<<<<< ======= >>>>>>> 标记，保留最终内容
git add <冲突文件>
git commit -m "merge: 解决冲突"
```

## 6. 远程仓库与 PR

```bash
git remote -v
git remote add origin git@github.com:<username>/<repo>.git
git push -u origin main
git push                       # 之后直接推当前分支
git pull                       # 拉取并合并远程更新
git fetch --prune              # 拉取远程信息并清理已删除分支
```

PR 流程：

```bash
git switch -c feature/xxx
# 修改文件
git add <file>
git commit -m "docs: 说明改动"
git push -u origin feature/xxx
```

然后到 GitHub：

1. **Compare & pull request**
2. base 选 `main`，compare 选 `feature/xxx`
3. 写标题和描述
4. **Create pull request**
5. **Merge pull request** → **Confirm merge**
6. 本地 `git switch main && git pull`
7. 删除本地/远程分支

## 7. 撤销与救急

### 还没 commit

```bash
git restore <file>             # 丢弃工作区改动（危险，确认不要了）
git restore --staged <file>    # 取消暂存，保留工作区改动
```

### commit 了但还没 push

```bash
git reset --soft HEAD~1        # 撤销最后一次 commit，改动保留在暂存区
git commit --amend -m "新说明" # 修改最后一次 commit 的 message
```

### 已经 push 了

```bash
git revert <commit-hash>       # 生成一个反向提交，安全
```

不要对已经 push 的提交使用 `git reset --hard` 后强推，除非你非常清楚后果。

### 临时保存

```bash
git stash                      # 保存当前未提交改动
git stash list
git stash pop                  # 恢复并删除最近一条 stash
```

### 找丢失的提交

```bash
git reflog
```

`reflog` 记录 HEAD 的移动历史，误删分支/回退后常用它找回。

## 8. commit message 约定

| 前缀 | 用途 | 例子 |
| --- | --- | --- |
| `feat` | 新功能 | `feat: 添加学习时长统计脚本` |
| `fix` | 修 bug | `fix: 修复统计脚本路径问题` |
| `docs` | 文档/笔记 | `docs: 添加第一天学习记录` |
| `style` | 格式调整 | `style: 统一 Markdown 格式` |
| `refactor` | 重构 | `refactor: 拆分统计函数` |
| `chore` | 杂项/初始化 | `chore: 初始化仓库` |
| `test` | 测试 | `test: 添加脚本测试` |

原则：一次 commit 只做一件事；标题用祈使句；不要写“update”“修改一下”。

## 9. .gitignore

```text
# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Logs
*.log

# Python
__pycache__/
*.pyc

# Build
build/
```

已经被 Git 跟踪的文件，加进 `.gitignore` 不会自动取消跟踪：

```bash
git rm --cached <file>
git commit -m "chore: 停止跟踪 <file>"
```

## 10. 常见错误

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `Please tell me who you are` | 没配 user.name/email | 执行第 1 节配置 |
| `Permission denied (publickey)` | SSH key 没加到 GitHub / agent 没加载 | `ssh-add -l`，重新添加 key |
| `Connection timed out` | 22 端口不通 | 用 443 端口配置 |
| `failed to push some refs` | 远程有本地没有的提交 | `git pull` 后再 push |
| `Your branch is behind` | 本地落后远程 | `git pull` |
| `Merge conflict` | 两个分支改了同一处 | 手动编辑冲突文件，`git add`，`git commit` |
| `detached HEAD` | 切到了某个 commit 而不是分支 | `git switch main` 或新建分支 |
| `nothing to commit` | 没有改动或没 `git add` | `git status` 检查 |

## 11. 四个区域

```text
工作区（Working Tree）
    │  git add
    ▼
暂存区（Index / Staging Area）
    │  git commit
    ▼
本地仓库（Local Repository）
    │  git push
    ▼
远程仓库（Remote / GitHub）
```

- `git diff`：工作区 vs 暂存区
- `git diff --cached`：暂存区 vs 最后一次 commit
- `git status`：三个区域的综合状态
- `git log`：本地仓库的提交历史

## 12. 今天的验收命令

```bash
git config --global --list
ssh -T git@github.com
git log --oneline --graph --all
git status
```
