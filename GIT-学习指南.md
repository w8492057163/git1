# Git 学习指南

这个仓库是你的 Git 练习场。下面从核心模型讲到日常操作、撤销技巧和分支，最后是 10 个可以反复练的练习。

所有练习都可以安全回退 —— 文末的「救命命令」能找回你误删的提交。

> 仓库路径：`C:\Users\l\Desktop\git`
> 远程仓库：`https://github.com/w8492057163/git1.git`（公有）

---

## 1. 核心模型：三个区域 + 一个远端

理解这张图，Git 就懂了一半：

```
工作区                暂存区              本地仓库             远程仓库
(你的文件)   --add-->  (index)  --commit--> (.git)  --push-->  (GitHub)
              <--restore--      <--reset--          <--pull--
```

| 区域 | 是什么 | 怎么进去 | 怎么出来 |
|---|---|---|---|
| **工作区** | 文件夹里你能直接看到、编辑的文件 | 直接改文件 | — |
| **暂存区** | 准备提交的「快照清单」 | `git add` | `git restore --staged` |
| **本地仓库** | 已提交的历史，存在 `.git` 里 | `git commit` | `git reset` / `git revert` |
| **远程仓库** | GitHub 上的副本 | `git push` | `git pull` |

关键点：**`git add` 不是保存文件**，它只是把改动放进「下一次提交的清单」。文件本身一直都在磁盘上。

---

## 2. 日常四步：status → add → commit → push

这是 90% 的时间你只会用到的四个命令。

```powershell
git status                 # 我现在改了什么？
git add .                  # 把改动放进暂存区（. 表示当前目录全部）
git commit -m "说明"        # 存成一个历史快照
git push                   # 同步到 GitHub
```

`git status` 的三种输出要认识：

| 输出 | 含义 |
|---|---|
| `Untracked files`（红色 `??`） | 新文件，Git 还没管它 |
| `Changes not staged`（红色 `M`） | 改过了，但没 `add` |
| `Changes to be committed`（绿色 `M`） | 已 `add`，等 `commit` |

**提交信息怎么写**：用一句话说清「做了什么」。好的例子：

- `Add 用户登录功能`
- `Fix 分页在空列表时崩溃`
- `Update README 增加安装说明`

避免 `update`、`修改`、`fix bug` 这种看不出内容的。

---

## 3. 看历史：你这个仓库的现状

```powershell
git log --oneline --graph --all    # 图形化看所有分支
git log -3                          # 只看最近 3 条
git show d0d635a                    # 看某个提交具体改了什么
git diff                            # 工作区 vs 暂存区（还没 add 的改动）
git diff --staged                   # 暂存区 vs 上次提交（准备提交的改动）
```

`git log --oneline` 每行格式：`提交哈希 提交信息`。哈希是提交的唯一 ID，前几位就能用（如 `d0d635a`）。

---

## 4. 撤销：按「错到哪一步」选命令

这是新手最需要的一节。先问自己：**改坏的东西在哪个区域？**

### 4.1 只是文件改乱了，还没 add

```powershell
git restore README.md        # 丢弃修改，恢复成上次提交的样子
```

⚠️ 这个操作**不可恢复**，改的内容直接没了。

### 4.2 已经 add 了，想撤回暂存区

```powershell
git restore --staged README.md    # 从暂存区拿出，文件内容不变
```

### 4.3 已经 commit 了，但还没 push

```powershell
git reset --soft HEAD~1      # 撤销提交，改动保留在暂存区
git reset --mixed HEAD~1     # 撤销提交，改动保留在工作区（默认）
```

⚠️ 别用 `--hard`，它会连你的文件改动一起删掉。

**只想改提交信息**（最常见）：

```powershell
git commit --amend -m "新的提交信息"
```

### 4.4 已经 push 了

不要 `reset` 后强推，那会改写别人看到的历史。用反向提交：

```powershell
git revert d0d635a           # 生成一个新提交，抵消那次改动
git push
```

### 4.5 救命命令：什么都能找回来

只要曾经 commit 过，就算 reset 掉了也能救：

```powershell
git reflog                   # 列出你所有操作的记录
git reset --hard <哈希>       # 回到那个状态
```

**`git reflog` 是 Git 的安全网**，学会它你就敢乱试了。提交过的内容几乎丢不掉。

---

## 5. 分支：并行开发的基础

分支就是一条独立的开发线，互不干扰。

```powershell
git branch                    # 列出分支（当前分支带 *）
git switch -c 实验            # 新建并切换到「实验」分支
git switch main               # 切回 main
git merge 实验                # 把「实验」的改动合并进当前分支
git branch -d 实验            # 删除已合并的分支
```

先在这个仓库随便建个分支乱改，再切回 `main` —— 你会发现 `main` 完全不受影响。这就是分支的价值。

**合并冲突**：如果两个分支改了同一行，`git merge` 会停下来，文件里出现：

```
<<<<<<< HEAD
你当前分支的内容
=======
要合并进来的内容
>>>>>>> 实验
```

手动改成你想要的结果，删掉 `<<<<<<<`、`=======`、`>>>>>>>` 三行标记，然后 `git add` + `git commit`。

---

## 6. 远程：和 GitHub 同步

```powershell
git remote -v                 # 看远程仓库地址
git fetch                     # 只下载远程信息，不改你的文件
git pull                      # 下载 + 合并到当前分支
git push                      # 上传你的提交
```

`fetch` 和 `pull` 的区别：`fetch` 是「去看一眼」，`pull` 是「看一眼并合并进来」。

### push 被拒绝怎么办

报错 `rejected (non-fast-forward)` 或 `Updates were rejected`，意思是远程有你本地没有的提交。解决：

```powershell
git pull                      # 先把远程的拉下来合并
git push                      # 再推
```

**不要**用 `git push --force` 强行覆盖，除非你清楚会丢掉什么。

---

## 7. 练习清单（10 个，从易到难）

按顺序做，每个都能撤销。做完在 [练习记录.md](练习记录.md) 里记一行。

| # | 任务 | 用到的命令 |
|---|---|---|
| 1 | 改一下 `练习记录.md`，提交并推送 | `status` `add` `commit` `push` |
| 2 | 新建一个文件 `notes.txt`，提交 | `add` `commit` |
| 3 | 看 `git log --oneline --graph --all` 的输出 | `log` |
| 4 | 用 `git show` 看第 1 个练习的提交内容 | `show` |
| 5 | 改文件后**不提交**，用 `git restore` 撤回 | `restore` |
| 6 | 改文件后 `add` 了，再用 `restore --staged` 撤回 | `restore --staged` |
| 7 | 提交一条信息写错的 commit，用 `--amend` 改掉 | `commit --amend` |
| 8 | 新建分支 `实验`，改文件提交，切回 `main` 确认互不影响 | `switch -c` |
| 9 | 把 `实验` 合并进 `main` | `merge` |
| 10 | 故意 reset 掉一个提交，用 `reflog` 找回来 | `reset` `reflog` |

**建议**：练习 5–10 之前先 `git push`，这样万一搞乱了，可以直接从 GitHub 重新 clone 一份对照。

---

## 8. 新手常见坑

| 坑 | 表现 | 解决 |
|---|---|---|
| 忘了 `add` | commit 后发现自己改的没进去 | `git add` 后 `--amend` |
| 提交了不该提交的文件 | `.env`、密码、大文件进了历史 | 加进 `.gitignore`，从暂存区移除 |
| detached HEAD | `git status` 说 `HEAD detached at ...` | `git switch main` 回到主分支 |
| 在错误的分支上改了 | 改动出现在不该出现的地方 | `git stash` → 切分支 → `git stash pop` |
| 递归删掉 `.git` | 整个历史没了 | **永远不要动 `.git` 文件夹** |

### 关于 `.gitignore`

这个仓库已有一个 `.gitignore`，里面写了 `.env`。规则是：**永远不要提交密码、密钥、Token**。

`.gitignore` 的写法：

```
.env              # 忽略这个名字的文件
*.log             # 忽略所有 .log
node_modules/     # 忽略整个目录
!important.log    # 感叹号表示例外，不忽略
```

注意：`.gitignore` 只对**还没被跟踪**的文件生效。已经提交过的文件，要先 `git rm --cached 文件名` 再提交。

---

## 9. 速查表

```powershell
# 日常
git status                          # 看状态
git add .                           # 暂存所有改动
git commit -m "说明"                 # 提交
git push                            # 推送
git pull                            # 拉取

# 查看
git log --oneline --graph --all     # 历史图
git show <哈希>                      # 某个提交的详情
git diff                            # 未暂存的改动

# 撤销
git restore <文件>                   # 丢弃未暂存的修改
git restore --staged <文件>          # 取消暂存
git commit --amend -m "新信息"       # 改上次提交信息
git revert <哈希>                    # 反向提交（已推送时用）
git reflog                          # 救命：所有操作记录

# 分支
git switch -c <名字>                 # 新建并切换
git switch main                     # 切换
git merge <名字>                     # 合并
git branch -d <名字>                 # 删除

# 远程
git remote -v                       # 看远程地址
git fetch                           # 只下载
```

---

## 10. 下一步学什么

1. **`git stash`** — 临时收起未提交的改动，切分支再恢复
2. **Pull Request 流程** — 在分支上开发，通过 PR 合并（团队协作的标准做法）
3. **`git rebase`** — 整理提交历史，让历史变成一条直线（进阶，先别碰）
4. **`git bisect`** — 二分查找哪个提交引入了 bug
5. **标签 `git tag`** — 给版本打标记，如 `v1.0.0`

学 Git 最好的方式是**用它管真实项目**。等你有了真实代码，上面这些自然就会了。
