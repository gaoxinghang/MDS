# Git 命令速查

分支、路径、`repo_url` 和 `commit_id` 按需替换；⚠ 表示删除内容或改写历史。

[仓库与配置](#仓库与配置) · [状态与差异](#状态与差异) · [暂存与提交](#暂存与提交)

[历史与搜索](#历史与搜索) · [分支](#分支) · [整合与冲突](#整合与冲突)

[远程同步](#远程同步) · [临时保存](#临时保存) · [撤销与恢复](#撤销与恢复)

[标签](#标签) · [忽略与清理](#忽略与清理) · [多工作区](#多工作区)

## 仓库与配置

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
| `git rev-parse --show-toplevel` | 查看工作区根目录 |
| `git help restore` | 查看子命令帮助；restore 可替换 |

## 状态与差异

| 命令 | 作用 |
| --- | --- |
| `git status` | 查看分支和文件状态 |
| `git status -sb` | 简洁显示分支及文件状态 |
| `git diff` | 查看未暂存改动，不含未跟踪文件内容 |
| `git diff --staged` | 查看已暂存改动；等价于 `--cached` |
| `git diff HEAD` | 查看已跟踪文件相对当前提交的改动 |
| `git diff --stat` | 查看未暂存改动的文件和行数统计 |
| `git diff --name-status` | 查看未暂存改动的路径和状态 |
| `git diff -- file.py` | 仅查看指定文件的未暂存改动 |
| `git diff main feature/login` | 比较两个分支尖端的文件快照 |
| `git diff main...feature/login` | 比较共同祖先与功能分支的快照 |
| `git diff --staged --check` | 检查暂存内容的空白错误和冲突标记 |

## 暂存与提交

| 命令 | 作用 |
| --- | --- |
| `git add -- file.py` | 暂存指定文件的新增、修改或删除 |
| `git add -- src/` | 暂存指定目录中的变更 |
| `git add .` | 暂存当前目录树中的变更 |
| `git add -A` | 暂存全仓库新增、修改和删除 |
| `git add -u` | 仅暂存已跟踪文件的修改和删除 |
| `git add -p` | 交互选择补丁块暂存 |
| `git mv old.py new.py` | 移动文件并暂存变更 |
| `git rm -- file.py` | 删除工作区文件并暂存删除 |
| `git commit` | 用编辑器填写说明并提交暂存内容 |
| `git commit -m "docs: update guide"` | 指定说明并提交暂存内容 |
| `git commit -am "fix: correct typo"` | 暂存已跟踪文件变更并提交，不含新文件 |
| `git commit --amend -m "docs: clarify setup"` | ⚠ 替换最近提交及说明，也会纳入暂存内容 |
| `git commit --amend --no-edit` | ⚠ 将暂存内容补入最近提交，保留说明 |

## 历史与搜索

| 命令 | 作用 |
| --- | --- |
| `git log --oneline -10` | 查看最近 10 条提交 |
| `git log --oneline --graph --decorate --all` | 查看本地已知分支的提交图 |
| `git show commit_id` | 查看提交说明和变更 |
| `git show commit_id:file.py` | 查看指定提交中的文件内容 |
| `git show HEAD~2` | 查看沿第一父链向前两代的提交 |
| `git show HEAD^2` | 查看当前合并提交的第二个父提交 |
| `git log --follow -- file.py` | 查看单文件历史，尝试跨重命名追溯 |
| `git log --oneline --grep="login"` | 按提交说明搜索 |
| `git log -S "old_api" -- file.py` | 查找字符串出现次数变化的提交 |
| `git blame -- file.py` | 查看每行最近修改的提交和作者 |
| `git grep -n "TODO"` | 搜索已跟踪文件并显示行号 |
| `git log --oneline main..feature/login` | 查看功能分支有、main 没有的提交 |

## 分支

| 命令 | 作用 |
| --- | --- |
| `git branch --show-current` | 查看当前分支名；分离 HEAD 时为空 |
| `git branch -vv` | 查看本地分支、上游及已知领先落后情况 |
| `git branch -a` | 列出本地和远程跟踪分支 |
| `git switch -c feature/login` | 从 HEAD 新建并切换分支 |
| `git switch -c feature/login main` | 从 main 新建并切换分支 |
| `git switch main` | 切换到已有分支 |
| `git switch -` | 返回上一个分支 |
| `git switch --track origin/feature/login` | 创建跟踪远程分支的本地分支 |
| `git branch -m new_name` | 重命名当前本地分支 |
| `git branch -d feature/login` | 通过合并检查后删除本地分支 |
| `git branch -D feature/login` | ⚠ 跳过合并检查，强制删除本地分支 |
| `git switch --detach commit_id` | 检出指定提交，进入分离 HEAD 状态 |
| `git switch -c rescue` | 从当前位置建分支，保留分离 HEAD 上的提交 |

## 整合与冲突

| 命令 | 作用 |
| --- | --- |
| `git merge feature/login` | 将指定分支合入当前分支 |
| `git merge --no-ff feature/login` | 合并时禁用快进，保留合并节点 |
| `git merge --ff-only origin/main` | 仅允许快进，分叉时停止 |
| `git rebase origin/main` | ⚠ 将当前分支提交重放到指定基底 |
| `git rebase -i HEAD~3` | ⚠ 交互整理最近 3 个提交 |
| `git cherry-pick commit_id` | 将一个普通提交的变更应用到当前分支 |
| `git diff --name-only --diff-filter=U` | 列出尚未解决冲突的文件 |
| `git add -- file.py` | 编辑冲突后，标记该文件已解决 |
| `git merge --continue` | 继续合并 |
| `git merge --abort` | 放弃本次合并 |
| `git rebase --continue` | 继续变基 |
| `git rebase --abort` | 放弃本次变基 |
| `git cherry-pick --continue` | 继续挑选提交 |
| `git cherry-pick --abort` | 放弃本次挑选 |
| `git revert --continue` | 继续反向提交 |
| `git revert --abort` | 放弃本次反向提交 |

> 以干净工作区开始整合；amend、rebase 默认仅用于未共享的提交。

## 远程同步

| 命令 | 作用 |
| --- | --- |
| `git remote -v` | 查看远程别名和 URL |
| `git remote add origin repo_url` | 添加远程地址，不会创建服务器端仓库 |
| `git remote set-url origin repo_url` | 修改已有远程地址 |
| `git remote show origin` | 查询远程分支及跟踪详情 |
| `git branch -u origin/main` | 设置当前分支的上游 |
| `git fetch origin` | 获取远程更新，不合并当前分支 |
| `git fetch --prune origin` | 获取更新，并按 refspec 清理失效远程跟踪引用 |
| `git pull --ff-only` | 抓取上游并快进，分叉时停止 |
| `git pull --rebase` | ⚠ 抓取上游并重放本地提交 |
| `git push -u origin feature/login` | 首次推送分支并设置上游 |
| `git push` | 按推送配置上传提交 |
| `git push origin main` | 推送本地 main 分支 |
| `git push --dry-run` | 预览推送，不实际更新远程引用 |
| `git push origin --delete feature/login` | ⚠ 删除远程分支 |
| `git push --force-with-lease origin feature/login` | ⚠ 带引用检查的强推，可能覆盖远程历史 |

> 强推前确认远程变更并协调协作者；默认 lease 的保护可能受后台 fetch 影响。

## 临时保存

| 命令 | 作用 |
| --- | --- |
| `git stash push -m "wip: login"` | 保存已跟踪文件改动并清理现场 |
| `git stash push -u -m "wip: login"` | 额外保存未跟踪文件，不含忽略项 |
| `git stash list` | 列出本地 stash 条目 |
| `git stash show -p "stash@{0}"` | 查看最新条目的已跟踪部分补丁 |
| `git stash apply "stash@{0}"` | 恢复条目，保留记录 |
| `git stash apply --index "stash@{0}"` | 尝试同时恢复暂存状态 |
| `git stash pop` | 恢复最新条目；成功后删除记录，冲突时保留 |
| `git stash drop "stash@{0}"` | ⚠ 删除指定保存记录 |

## 撤销与恢复

| 命令 | 作用 |
| --- | --- |
| `git restore --staged -- file.py` | 从 HEAD 恢复暂存区文件，保留工作区内容 |
| `git restore --staged .` | 取消当前目录树的暂存，保留工作区内容 |
| `git restore -- file.py` | ⚠ 从暂存区恢复文件，丢弃未暂存修改 |
| `git restore --source=commit_id -- file.py` | ⚠ 从指定提交覆盖工作区文件，不自动暂存 |
| `git restore --staged --worktree -- file.py` | ⚠ 将文件的暂存区和工作区都恢复为 HEAD |
| `git reset --soft HEAD~1` | ⚠ 撤回最近提交，保留暂存区和工作区 |
| `git reset --mixed HEAD~1` | ⚠ 撤回最近提交并重置暂存区，保留工作区 |
| `git reset --hard commit_id` | ⚠ 重置分支、暂存区和工作区，丢弃相关改动 |
| `git revert commit_id` | 用新提交撤销一个普通提交的效果，不删除历史 |
| `git reflog` | 查看本地 HEAD 移动记录，定位误删提交 |
| `git branch rescue commit_id` | 为找回的提交建立分支，不切换工作区 |

> `switch` / `restore` 需 Git 2.23+；`HEAD~1` 要求存在父提交。
> `reset --hard` 还可能删除阻碍检出的未跟踪路径；reflog 不保证找回未提交内容。

## 标签

| 命令 | 作用 |
| --- | --- |
| `git tag --list` | 列出本地标签 |
| `git show v1.0.0` | 查看标签及关联提交 |
| `git tag -a v1.0.0 -m "release: publish v1.0.0"` | 为当前提交创建附注标签 |
| `git push origin v1.0.0` | 推送指定标签 |
| `git fetch origin --tags` | 抓取标签及通常的分支更新 |
| `git tag -d v1.0.0` | 删除本地标签 |
| `git push origin --delete refs/tags/v1.0.0` | ⚠ 删除远程标签 |

## 忽略与清理

| 命令 | 作用 |
| --- | --- |
| `git ls-files` | 列出已跟踪文件 |
| `git ls-files -- .env` | 检查文件是否已被跟踪 |
| `git check-ignore -v -- .env` | 查看忽略规则及来源 |
| `git status --ignored -s` | 显示忽略项，标记为 `!!` |
| `git rm --cached -- .env` | 停止跟踪并暂存删除，保留本地文件 |
| `git rm -r --cached -- build/` | 停止跟踪目录，保留本地目录 |
| `git clean -nd` | 预览未跟踪文件和目录的清理 |
| `git clean -id` | ⚠ 交互选择并删除未跟踪项 |
| `git clean -fd` | ⚠ 删除未跟踪文件和目录，保留忽略项 |
| `git clean -ndX` | 预览仅清理忽略项 |
| `git clean -fdX` | ⚠ 删除忽略项，可能包含 .env 和本地环境 |

> clean 删除的未跟踪内容通常无法通过 Git 恢复；先预览。
> 忽略规则和 `rm --cached` 不清除历史中的密钥；泄露后需撤销或轮换。

## 多工作区

| 命令 | 作用 |
| --- | --- |
| `git worktree list` | 列出工作目录及对应分支 |
| `git worktree add -b hotfix ../hotfix main` | 从 main 新建分支，在另一目录检出 |
| `git worktree add ../review feature/login` | 在另一目录检出尚未被占用的分支 |
| `git worktree remove ../hotfix` | 删除干净的关联工作目录，不删除分支 |
| `git worktree prune` | 清理失效工作区管理记录 |
