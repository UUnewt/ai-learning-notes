# 03 · Git 与 GitHub 速查

> 这篇是「做过一遍之后」的笔记，命令都是我在 `D:\ai-learning-notes` 里真实跑过的。

## 1. 先分清三个东西

| 名字 | 是什么 | 类比 |
|---|---|---|
| **Git** | 装在你电脑上的**版本管理工具**，离线也能用 | 你家的保险柜 |
| **GitHub** | 一个网站，把 Git 仓库放到云上，方便备份和协作 | 银行的公共保险库（托管商） |
| **GitLab** | 同类网站，可以自己搭在公司内网，私有化更彻底 | 另一家银行（可自建） |

关键：**没有 GitHub，Git 照样能用**（只是别人看不到你的代码）；**没有 Git，GitHub 就是一排空文件夹**。

## 2. 四个区：理解 Git 的核心

```
工作区            暂存区            本地仓库           远端仓库
(你看到的文件) → (待提交清单)  →  (带历史快照)  →  (GitHub 上)
              git add         git commit      git push
              ←──────────────────────────────  git pull
```

- **工作区**：`D:\ai-learning-notes` 里你能直接编辑的文件
- **暂存区**：一个"这次要提交哪些改动"的清单，`git add` 往里放
- **本地仓库**：`.git` 隐藏目录，存着每一次快照
- **远端仓库**：GitHub 上那份

> 为什么要有暂存区？因为一次改动可能涉及 5 个文件，但你只想提交其中 3 个。暂存区让你「挑着提交」。

## 3. 四个必会命令

### init —— 让文件夹变成仓库

```bash
git init -b main
```

`-b main` 指定初始分支叫 `main`（GitHub 现在默认也叫 main，保持一致省事）。

跑完会多出一个 `.git` 目录。**这个目录不要删、不要手动改**，删了历史就没了。

### add —— 把改动放进暂存区

```bash
git add .                  # 全部改动
git add README.md          # 只加一个文件
git status --short         # 看现在谁在暂存区（A=新增 M=修改）
```

### commit —— 封存一个快照

```bash
git commit -m "docs: init AI learning notes repo"
```

`-m` 后面是提交说明。约定俗成的写法是 `类型: 说明`，常用类型：

| 前缀 | 含义 |
|---|---|
| `feat` | 新功能 |
| `fix` | 修 bug |
| `docs` | 只改文档 |
| `refactor` | 重构（不改功能） |
| `chore` | 杂活（配置、依赖） |

每次 commit 会得到一个 40 位哈希（比如 `3559d59...`），这就是那次快照的身份证。

### push / pull —— 与远端同步

```bash
git remote add origin git@github.com:UUnewt/ai-learning-notes.git   # 第一次要告诉它推到哪
git push -u origin main        # 推上去，-u 记住这个对应关系，以后直接 git push
git pull                       # 把远端的改动拉下来（多人协作必用）
```

## 4. 我第一次遇到的两个问题

### ① `Repository not found`

先 `git push` 才去建仓库，一定报这个错。**GitHub 上的仓库要先存在，push 才有地方去**。

### ② 中文文件名显示成 `\346\257\257...`

Git 默认会把非 ASCII 字符转义显示。关掉它：

```bash
git config --global core.quotepath false
```

## 5. SSH 还是 HTTPS？

两种连 GitHub 的方式，我选了 SSH：

| | SSH | HTTPS |
|---|---|---|
| 地址形式 | `git@github.com:用户名/仓库.git` | `https://github.com/用户名/仓库.git` |
| 认证方式 | 密钥对，配一次永久有效 | 账号密码 / Personal Access Token |
| 输密码 | 不用 | Token 过期要重配 |
| 适合 | 自己电脑长期用 | 临时机器、公司电脑 |

配 SSH 的三步（我已经配好了）：

```bash
# 1. 生成密钥对（敲完一路回车，口令留空）
ssh-keygen -t ed25519 -C "你的邮箱"

# 2. 把公钥内容贴到 GitHub → Settings → SSH and GPG keys
#    C:\Users\uu\.ssh\id_ed25519.pub  ← 只有这个能公开

# 3. 验证
ssh -T git@github.com
# 成功会回：Hi UUnewt! You've successfully authenticated...
```

**私钥 `id_ed25519`（没有 .pub）永远不外传**，它等于你的身份证。

## 6. 日常只用这五条

```bash
git status              # 我现在改了什么？（最常用，随时敲）
git add .               # 收进暂存区
git commit -m "..."     # 封存
git push                # 上云
git pull                # 同步别人的改动
```

## 7. 我的仓库

<https://github.com/UUnewt/ai-learning-notes>
