# Git 命令速查

按操作对象分为六类；分支、路径、`repo_url` 和 `commit_id` 按需替换。
⚠ 表示删除内容或改写历史；示例中的 `switch`、`restore` 需 Git 2.23+。

[TOC]

## 仓库与配置

**仓库保存项目的版本历史；配置决定提交署名等行为。**

| 命令 | 作用 |
| --- | --- |
| `git --version` | 查看 Git 版本 |
| `git init -b main` | 初始化仓库，初始分支为 main；需 Git 2.28+ |
| `git clone repo_url` | 克隆仓库并检出默认分支 |
| `git clone repo_url my_project` | 克隆到指定目录 |
| `git config user.name "Your Name"` | 设置当前仓库提交署名 |
| `git config user.email "you@example.com"` | 设置当前仓库提交邮箱 |
| `git config --global user.name "Your Name"` | 设置当前用户默认署名 |
| `git config --global user.email "you@example.com"` | 设置当前用户默认邮箱 |
| `git config --list --show-origin` | 查看配置及来源文件 |
| `git help restore` | 查看子命令帮助；restore 可替换 |

> 仓库内不加 `--global` 通常只修改当前仓库配置；署名配置不是远程登录凭据。

## 文件与提交

**工作区是正在编辑的文件，暂存区是下次提交的内容，提交是保存的版本快照。**

| 命令 | 作用 |
| --- | --- |
| `git status -sb` | 简洁查看分支和文件状态 |
| `git diff` | 查看未暂存改动，不含未跟踪文件内容 |
| `git diff --staged` | 查看已暂存、尚未提交的改动 |
| `git diff HEAD` | 查看已跟踪文件相对当前提交的改动 |
| `git add -- file.py` | 暂存指定文件的新增、修改或删除 |
| `git add .` | 暂存当前目录树中的变更 |
| `git add -A` | 暂存全仓库新增、修改和删除 |
| `git add -p` | 交互选择部分改动暂存 |
| `git commit -m "docs: update guide"` | 提交暂存区内容 |
| `git commit --amend --no-edit` | ⚠ 将暂存内容补入最近提交，保留说明 |
| `git mv old.py new.py` | 移动文件并暂存变更 |
| `git rm -- file.py` | ⚠ 删除工作区文件并暂存删除 |
| `git ls-files` | 列出已跟踪文件 |
| `git check-ignore -v -- .env` | 查看文件命中的忽略规则及来源 |
| `git rm --cached -- .env` | 停止跟踪并暂存删除，保留本地文件 |
| `git clean -nd` | 预览未跟踪文件和目录的清理 |
| `git clean -fd` | ⚠ 删除未跟踪文件和目录，保留忽略项 |

> `.gitignore` 用来忽略未跟踪文件；不会自动停止跟踪已提交的文件。
> `amend` 会替换最近提交，通常只用于尚未共享的提交。
> `clean` 删除的未跟踪内容通常无法通过 Git 恢复；先预览。
> 忽略规则和 `rm --cached` 不清除历史中的密钥；泄露后需撤销或轮换。

## 分支与合并

**分支指向一条开发线的最新提交；合并把另一条开发线的改动整合进当前分支。**

| 命令 | 作用 |
| --- | --- |
| `git branch -vv` | 查看本地分支、上游及已知领先落后情况 |
| `git branch -a` | 列出本地和远程跟踪分支 |
| `git switch -c feature/login` | 从当前位置创建并切换分支 |
| `git switch main` | 切换到已有分支 |
| `git switch -` | 返回上一个分支 |
| `git branch -m new_name` | 重命名当前本地分支 |
| `git branch -d feature/login` | 通过合并检查后删除本地分支 |
| `git branch -D feature/login` | ⚠ 跳过合并检查，强制删除本地分支 |
| `git merge feature/login` | 将指定分支合入当前分支 |
| `git rebase main` | ⚠ 将当前分支的独有提交重放到 main 上 |
| `git cherry-pick commit_id` | 将一个普通提交的变更应用到当前分支 |
| `git diff --name-only --diff-filter=U` | 列出尚未解决冲突的文件 |
| `git add -- file.py` | 手动解决冲突后，将文件标记为已解决 |
| `git merge --continue` | 解决冲突并暂存后，继续合并 |
| `git merge --abort` | 放弃本次合并 |
| `git rebase --continue` | 解决冲突并暂存后，继续变基 |
| `git rebase --abort` | 放弃本次变基 |

> 操作作用于当前分支：要把功能分支合入 main，先切到 main，再执行 merge。
> 以干净工作区开始整合；`rebase` 会改写提交，通常只用于未共享的提交。
> `cherry-pick` 遇到冲突时，同样使用其 `--continue` 或 `--abort`。

## 远程同步

**远程是服务器仓库的本地别名，常用 origin；上游是当前分支默认跟踪的远程分支。**

| 命令 | 作用 |
| --- | --- |
| `git remote -v` | 查看远程别名和 URL |
| `git remote add origin repo_url` | 添加远程地址，不会创建服务器端仓库 |
| `git remote set-url origin repo_url` | 修改已有远程地址 |
| `git branch -u origin/main` | 设置当前分支的上游 |
| `git fetch origin` | 获取远程更新，不合并当前分支 |
| `git fetch --prune origin` | 获取更新并按 refspec 清理失效远程跟踪引用 |
| `git pull --ff-only` | 抓取上游并快进，分叉时停止 |
| `git pull --rebase` | ⚠ 抓取上游并重放本地提交 |
| `git push -u origin feature/login` | 推送分支并设置上游 |
| `git push` | 按推送配置上传提交 |
| `git push --dry-run` | 预览推送，不实际更新远程引用 |
| `git push origin --delete feature/login` | ⚠ 删除远程分支 |
| `git push --force-with-lease origin feature/login` | ⚠ 带引用检查的强推 |

> `fetch` 只取回更新；`pull` 还会尝试整合到当前分支；`push` 上传本地提交。
> 强推仍可能覆盖远程历史；先确认远程变更并协调协作者。
> 默认 `--force-with-lease` 的保护可能受后台 fetch 影响，不是绝对安全保证。

## 历史与恢复

**历史用于查看发生过什么；恢复操作要区分撤销文件、撤回提交和生成反向提交。**

| 命令 | 作用 |
| --- | --- |
| `git log --oneline -10` | 查看最近 10 条提交 |
| `git log --oneline --graph --decorate --all` | 查看本地已知分支的提交图 |
| `git show commit_id` | 查看提交说明及变更 |
| `git show commit_id:file.py` | 查看指定提交中的文件内容 |
| `git log --follow -- file.py` | 查看单文件历史，尝试跨重命名追溯 |
| `git blame -- file.py` | 查看每行最近修改的提交和作者 |
| `git restore --staged -- file.py` | 取消该文件的暂存，保留工作区内容 |
| `git restore -- file.py` | ⚠ 从暂存区恢复文件，丢弃未暂存修改 |
| `git restore --source=commit_id -- file.py` | ⚠ 从指定提交覆盖工作区文件 |
| `git reset --soft HEAD~1` | ⚠ 撤回最近提交，保留暂存区和工作区 |
| `git reset --mixed HEAD~1` | ⚠ 撤回最近提交并重置暂存区，保留工作区 |
| `git reset --hard commit_id` | ⚠ 重置当前分支、暂存区和工作区，丢弃相关改动 |
| `git revert commit_id` | 用新提交撤销一个普通提交的效果，不删除历史 |
| `git reflog` | 查看本地 HEAD 移动记录，辅助定位丢失的提交 |
| `git branch rescue commit_id` | 为找到的提交建立分支，不切换工作区 |

临时切换工作、但还不想提交时，可以使用 stash：

| 命令 | 作用 |
| --- | --- |
| `git stash push -u -m "wip: login"` | 保存已跟踪和未跟踪文件改动，不含忽略项 |
| `git stash list` | 列出本地 stash 条目 |
| `git stash show -p "stash@{0}"` | 查看最新条目的已跟踪部分补丁 |
| `git stash apply "stash@{0}"` | 恢复条目，保留记录 |
| `git stash pop` | 恢复最新条目；成功后删除记录，冲突时保留 |
| `git stash drop "stash@{0}"` | ⚠ 删除指定保存记录 |

> `HEAD` 通常指当前提交，`HEAD~1` 指沿第一父链向前一代，要求存在父提交。
> 已共享的提交通常用 `revert` 撤销，避免用 reset 改写共享历史。
> `revert` 遇到冲突时，解决后用 `--continue` 继续，或用 `--abort` 放弃。
> `reset --hard` 还可能删除阻碍检出的未跟踪路径；reflog 不保证找回未提交内容。
> stash 不是长期备份，也不会随普通 push 上传。

## 标签与多工作区

**标签标记某个提交；多工作区让同一仓库在不同目录中分别检出不同分支。**

| 命令 | 作用 |
| --- | --- |
| `git tag --list` | 列出本地标签 |
| `git show v1.0.0` | 查看标签及关联提交 |
| `git tag -a v1.0.0 -m "release: publish v1.0.0"` | 为当前提交创建附注标签 |
| `git push origin v1.0.0` | 推送指定标签 |
| `git tag -d v1.0.0` | 删除本地标签，不删除对应提交 |
| `git push origin --delete refs/tags/v1.0.0` | ⚠ 删除远程标签 |
| `git worktree list` | 列出工作目录及对应分支 |
| `git worktree add -b hotfix ../hotfix main` | 从 main 新建分支，在另一目录检出 |
| `git worktree remove ../hotfix` | 删除干净的关联工作目录，不删除分支 |
| `git worktree prune` | 清理失效工作区管理记录 |

> 普通分支推送不保证包含标签；需要发布标签时显式推送。
> 多工作区共享仓库历史，但各有工作区和暂存区；同一分支通常不能同时检出两次。
