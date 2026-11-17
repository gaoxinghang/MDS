# 从零创建本地 Git 仓库并关联 GitHub

> **适用场景**：将本地尚未进行版本控制的项目目录，初始化为 Git 仓库并完整推送到 GitHub 新建的空仓库中。

> [!IMPORTANT]
> 文中所有命令出现的占位符（如 `/你的/项目目录`、`你的用户名`、`你的 GitHub 邮箱`、`my-project`），请替换为你本机的真实信息。

---

## ⚡ 极速速查（TL;DR）

如果你已熟悉流程，仅需查阅完整命令序列，可直接参考下方操作流：

```bash
# 1. 进入目录并初始化
cd "/你的/项目目录"
git init
git branch -m main

# 2. 配置提交者信息（单仓库配置）
git config user.name "你的名字"
git config user.email "你的 GitHub 邮箱"

# 3. 暂存并提交（确保已有 .gitignore 与文件）
git add .
git status
git commit -m "Initial commit"

# 4. 关联远程仓库并推送到 main 分支
git remote add origin https://github.com/你的用户名/my-project.git
git push -u origin main
```

---

## 目录

- [第一阶段：本地仓库初始化与配置](#第一阶段本地仓库初始化与配置)
  - [1. 检查 Git 是否安装](#1-检查-git-是否安装)
  - [2. 准备并进入项目目录](#2-准备并进入项目目录)
  - [3. 初始化 Git 仓库并命名分支](#3-初始化-git-仓库并命名分支)
  - [4. 设置提交者信息](#4-设置提交者信息)
  - [5. 配置忽略规则（.gitignore）](#5-配置忽略规则gitignore)
  - [6. 创建第一次提交](#6-创建第一次提交)
- [第二阶段：远程仓库创建与认证](#第二阶段远程仓库创建与认证)
  - [7. 在 GitHub 创建远程空仓库](#7-在-github-创建远程空仓库)
  - [8. 配置 GitHub 身份认证（二选一）](#8-配置-github-身份认证二选一)
- [第三阶段：关联推送与日常管理](#第三阶段关联推送与日常管理)
  - [9. 关联远程仓库并完成首次推送](#9-关联远程仓库并完成首次推送)
  - [10. 日常代码更新标准流程](#10-日常代码更新标准流程)
- [常见报错与排查指南](#常见报错与排查指南)

---

## 第一阶段：本地仓库初始化与配置

### 1. 检查 Git 是否安装

打开终端执行以下命令：

```bash
git --version
```

- **正常输出**：例如 `git version 2.x.x`，说明 Git 已就绪。
- **macOS 提示安装**：如果系统弹出安装提示，点击「安装命令行开发者工具」完成安装即可。

### 2. 准备并进入项目目录

根据你的项目现状选择执行：

```bash
# 场景 A：已有项目目录
cd "/你的/项目目录"

# 场景 B：从零创建全新目录
mkdir -p ~/workspace/my-project
cd ~/workspace/my-project
```

> [!NOTE]
> 后续所有 Git 命令均在该项目根目录下执行。

### 3. 初始化 Git 仓库并命名分支

```bash
git init
git branch -m main
```

| 命令 | 作用说明 |
| :--- | :--- |
| `git init` | 在当前目录下创建隐藏的 `.git` 文件夹，正式开启版本控制。 |
| `git branch -m main` | 将当前默认分支重命名为 `main`（GitHub 默认主分支名称）。 |

> [!TIP]
> 常见教程常写 `git branch -M main`，大写 `-M` 属于强制重命名（若存在同名分支将直接覆盖）。新仓库使用小写 `-m` 更安全规范。
> 
> 若该目录此前已经初始化过，请先运行 `git status` 与 `git remote -v` 检查状态，切勿重复初始化或强行覆盖。

### 4. 设置提交者信息

用于记录每一笔 Commit 的署名，方便在 GitHub 提交历史中识别身份。

```bash
# 仅对当前项目生效（推荐）
git config user.name "你的名字"
git config user.email "你的 GitHub 邮箱"

# 或者：对当前电脑的所有 Git 项目全局生效
git config --global user.name "你的名字"
git config --global user.email "你的 GitHub 邮箱"
```

> [!NOTE]
> - **并非登录认证**：此配置仅作为日志展示，不等于登录 GitHub。
> - **邮箱建议**：请使用 GitHub 账号中已验证的邮箱，提交记录才能正确点亮 GitHub 头像与贡献绿墙；若不想公开真实邮箱，可前往 GitHub 设置页面启用并填写官方提供的 `noreply` 邮箱。

### 5. 配置忽略规则（.gitignore）

在项目根目录下新建 `.gitignore` 文件，过滤不需要版本控制的临时文件、依赖包与本地密钥：

```gitignore
# 本地环境变量及密钥文件（严禁提交！）
.env
.env.*
!.env.example

# 项目依赖与运行环境
node_modules/
.venv/
__pycache__/

# 系统与编辑工具缓存
.DS_Store
Thumbs.db
.idea/
.vscode/
```

> [!WARNING]
> - `.env.example` 仅用于放置变量名模板，**切勿写入真实密钥**。
> - **避免上传泄漏**：即便创建的是 Private（私有）私有仓库，也坚决不要提交密钥、Token、数据库密码等敏感信息。
> - **已被跟踪的文件**：`.gitignore` 仅对未被 Git 跟踪的新文件生效。若敏感文件已在历史提交中，单从目录删除并不能抹除历史，必须轮换或撤销密钥。

### 6. 创建第一次提交

Git 默认不会跟踪空文件夹。如果当前是空目录，请先创建一个文件（如 `README.md`）：

```bash
# 1. 将文件添加到暂存区
git add .

# 2. 检查暂存区状态（核对有无误加的敏感文件）
git status

# 3. （可选）查看本次将要提交的具体代码差异
git diff --cached

# 4. 确认无误后，创建初次提交
git commit -m "Initial commit"
```

---

## 第二阶段：远程仓库创建与认证

### 7. 在 GitHub 创建远程空仓库

1. 登录 [GitHub](https://github.com/)，点击右上角 **＋ → New repository**。
2. **Repository name**：填写仓库名称（例如 `my-project`，无需与本地文件夹同名）。
3. **Visibility**：按需选择 **Public**（公开）或 **Private**（私有）。
4. **关键步骤**：**不要勾选** Add a README file、Add .gitignore 或 Choose a license（务必保持仓库为完全空白）。
5. 点击 **Create repository**。
6. 复制生成的 HTTPS 地址：
   ```text
   https://github.com/你的用户名/my-project.git
   ```

### 8. 配置 GitHub 身份认证（二选一）

> [!IMPORTANT]
> GitHub 早已废除使用账号密码进行终端 Git 推送的方式，必须通过 **GitHub CLI** 或 **Personal Access Token (PAT)** 认证。

#### 方式一：使用 GitHub CLI（最推荐，一次配置永久省心）

如果尚未安装 `gh`，macOS 用户可通过 Homebrew 一键安装：

```bash
brew install gh
```

登录并配置 Git 凭证：

```bash
# 1. 发起登录（根据提示选择 GitHub.com -> HTTPS -> 网页授权登录）
gh auth login

# 2. 将 GitHub CLI 配置为 Git 的凭据管理器
gh auth setup-git
```

#### 方式二：使用个人访问令牌（Personal Access Token, PAT）

1. 进入 GitHub：**Settings → Developer settings → Personal access tokens**。
2. 推荐创建 **Fine-grained tokens**（细粒度令牌）：
   - **Repository access**：勾选目标仓库。
   - **Repository permissions**：将 **Contents** 设为 **Read and write**。
3. 生成并妥善保存 Token 字符串。
4. 在终端执行推送时，按提示输入：
   - **Username**：输入你的 GitHub 用户名。
   - **Password**：粘贴你的 **PAT 令牌**（非 GitHub 网页登录密码）。

---

## 第三阶段：关联推送与日常管理

### 9. 关联远程仓库并完成首次推送

在本地项目根目录下执行：

```bash
# 1. 添加名为 origin 的远程仓库地址
git remote add origin https://github.com/你的用户名/my-project.git

# 2. 验证关联的远程地址是否准确
git remote -v

# 3. 推送本地 main 分支到远程，并设置上游追踪分支
git push -u origin main
```

- `-u` 参数（`--set-upstream`）：将本地 `main` 与远程 `origin/main` 建立绑定，后续推送只需简写 `git push` 即可。
- 推送成功后，刷新 GitHub 仓库页面，即可查看上传的所有文件与提交记录。

### 10. 日常代码更新标准流程

后续日常开发和修改代码后，标准提交与同步循环如下：

```bash
# 1. 添加修改到暂存区
git add .

# 2. 检查暂存状态与差异
git status
git diff --cached

# 3. 提交修改并附带清晰的说明
git commit -m "feat: 描述本次更新的内容"

# 4. 推送到远程 GitHub 仓库
git push
```

---

## 常见报错与排查指南

| 常见错误提示 | 问题根因 | 解决办法 |
| :--- | :--- | :--- |
| `fatal: remote origin already exists.` | 本地已配置过名为 `origin` 的远程地址。 | 先执行 `git remote -v` 查看现状。<br>如需更换地址，执行：<br>`git remote set-url origin https://github.com/你的用户名/新仓库.git` |
| `error: src refspec main does not match any` | 本地尚未产生任何 Commit，或当前分支名称不叫 `main`。 | 1. 执行 `git status` 确认是否有文件已 Commit。<br>2. 执行 `git branch --show-current` 查看当前分支名。 |
| `error: failed to push some refs... (Updates were rejected)` | 远程仓库非空（创建仓库时勾选了 README/License），导致本地与远程历史脱节。 | 1. 最简单方式：删除该 GitHub 仓库，重新建一个**纯空**仓库。<br>2. 若需保留远程文件，拉取并合并历史：<br>`git pull origin main --allow-unrelated-histories`<br>解决冲突后再次 `git push`。 |
| `Permission denied` / `Repository not found` | 1. 仓库 URL 拼写有误。<br>2. 账号无该仓库权限。<br>3. PAT 权限不足或已过期。 | 1. 检查 `git remote -v` 中的组织名/用户名和仓库拼写。<br>2. 检查 PAT 是否拥有 `Contents: Read and write` 权限。<br>3. 若使用 GitHub CLI，执行 `gh auth status` 查看认证状态。 |
