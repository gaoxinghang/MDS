# Conda 命令速查

示例环境 `myenv`、包名和路径按需替换；包操作默认针对当前环境。⚠ 表示需谨慎操作。

[查询](#查询) · [环境](#环境) · [安装](#安装) · [更新与卸载](#更新与卸载)

[渠道与配置](#渠道与配置) · [导出与复现](#导出与复现) · [运行](#运行)

[终端](#终端) · [清理与回滚](#清理与回滚) · [Pip](#pip) · [诊断](#诊断)

## 查询

| 命令 | 作用 |
| --- | --- |
| `conda --version` | 查看 Conda 版本 |
| `conda info` | 查看平台、环境和缓存目录 |
| `conda env list` | 列出环境；`*` 为当前环境 |
| `conda list` | 列出当前环境的包 |
| `conda list -n myenv` | 列出指定环境的包 |
| `conda list numpy` | 按正则表达式过滤已安装包名 |
| `conda list --full-name numpy` | 精确匹配已安装包名 |
| `conda list --show-channel-urls` | 显示包的来源渠道 |
| `conda search numpy` | 搜索渠道中的可用版本 |
| `conda search "numpy=1.26.4"` | 搜索指定版本 |
| `conda search numpy --info` | 查看包详情与依赖 |
| `conda search -c conda-forge numpy` | 优先搜索指定渠道 |
| `conda install --help` | 查看子命令参数；install 可替换 |

## 环境

| 命令 | 作用 |
| --- | --- |
| `conda create -n myenv python=3.11` | 创建指定 Python 版本的环境 |
| `conda create -n myenv python numpy pandas` | 创建环境并安装包 |
| `conda create -p ./env python=3.11` | 按路径创建环境 |
| `conda activate myenv` | 激活命名环境 |
| `conda activate ./env` | 激活路径环境 |
| `conda deactivate` | 退出当前激活层，不删除环境 |
| `conda create -n newenv --clone myenv` | 克隆环境 |
| `conda env remove -n myenv --dry-run` | 预览环境删除 |
| `conda env remove -n myenv` | ⚠ 删除命名环境；先退出，不要删除 base |
| `conda env remove -p ./env` | ⚠ 删除指定路径的环境 |

## 安装

| 命令 | 作用 |
| --- | --- |
| `conda install numpy pandas` | 安装包及所需依赖 |
| `conda install numpy=1.26.4` | 安装或降级到指定版本 |
| `conda install "numpy>=1.26,<2"` | 安装满足版本范围的包 |
| `conda install -n myenv numpy` | 向命名环境安装 |
| `conda install -p ./env numpy` | 向路径环境安装 |
| `conda install -n myenv python=3.11` | 调整 Python 版本并重新求解依赖 |
| `conda install -c conda-forge numpy` | 本次优先使用指定渠道 |
| `conda install --override-channels -c conda-forge numpy` | 本次仅使用指定渠道 |
| `conda install numpy --dry-run` | 预览安装计划；仍可能更新索引缓存 |
| `conda install numpy -y` | 自动确认安装 |

## 更新与卸载

| 命令 | 作用 |
| --- | --- |
| `conda update numpy` | 更新到依赖约束允许的版本 |
| `conda update -n myenv numpy` | 更新指定环境的包 |
| `conda update -n myenv --all` | ⚠ 更新全部包，可能大幅调整依赖 |
| `conda update -n base conda` | 更新 Conda 及所需依赖 |
| `conda remove numpy --dry-run` | 预览卸载及连带变更 |
| `conda remove numpy` | 卸载包，可能连带移除依赖它的包 |
| `conda remove -n myenv numpy` | 从指定环境卸载 |

## 渠道与配置

| 命令 | 作用 |
| --- | --- |
| `conda config --show` | 查看合并后的有效配置 |
| `conda config --show-sources` | 查看配置文件及其内容 |
| `conda config --show channels` | 查看渠道顺序 |
| `conda config --prepend channels conda-forge` | 添加高优先级渠道 |
| `conda config --append channels conda-forge` | 添加低优先级渠道 |
| `conda config --set channel_priority strict` | 启用严格渠道优先级 |
| `conda config --env --set channel_priority strict` | 仅写入当前已激活环境的配置 |
| `conda config --remove channels conda-forge` | 删除指定渠道条目 |
| `conda config --remove-key channels` | 删除配置键；仍可能继承默认值或其他配置 |
| `conda config --describe channel_priority` | 查看配置项说明 |
| `conda config --validate` | 校验配置 |

## 导出与复现

| 命令 | 作用 |
| --- | --- |
| `conda env export --from-history > env.yml` | 导出显式请求的 Conda 依赖，不含 Pip |
| `conda env export -n myenv > env.yml` | 导出指定环境的完整描述 |
| `conda env export --no-builds > env.yml` | 导出时去掉构建号，不保证跨平台兼容 |
| `conda list --explicit > spec.txt` | 导出精确 Conda 构建，不含 Pip，通常限同平台 |
| `conda env create -f env.yml` | 按 YAML 创建环境 |
| `conda env create -n newenv -f env.yml` | 按 YAML 创建并覆盖环境名 |
| `conda env update -n myenv -f env.yml` | 按 YAML 更新已有环境 |
| `conda env update -n myenv -f env.yml --prune` | ⚠ 同步并移除不再需要的 Conda 包 |
| `conda create -n newenv --file spec.txt` | 从精确包清单重建环境 |

> `>` 覆盖目标文件；分享前检查私有源、凭据和本地路径。

## 运行

| 命令 | 作用 |
| --- | --- |
| `conda run -n myenv python --version` | 使用目标环境查看 Python 版本 |
| `conda run -n myenv python main.py` | 免激活运行脚本 |
| `conda run -p ./env python main.py` | 使用路径环境运行脚本 |
| `conda run -n myenv --no-capture-output python -u main.py` | 直出日志并禁用缓冲 |
| `conda list -n myenv --json` | 以 JSON 输出包信息 |

## 终端

| 命令 | 作用 |
| --- | --- |
| `conda init zsh` | 初始化 Zsh；执行后重开终端 |
| `conda init bash` | 初始化 Bash |
| `conda init powershell` | 初始化 PowerShell |
| `conda init zsh --dry-run` | 预览初始化修改 |
| `conda init --reverse zsh` | 撤销 Zsh 初始化配置 |
| `conda config --set auto_activate_base false` | 关闭默认环境自动激活 |
| `conda config --set auto_activate_base true` | 开启默认环境自动激活 |

## 清理与回滚

| 命令 | 作用 |
| --- | --- |
| `conda clean --all --dry-run` | 预览缓存清理 |
| `conda clean --all` | ⚠ 清理索引、归档、未使用包缓存等 |
| `conda clean --tarballs` | 清理下载归档 |
| `conda clean --index-cache` | 清理索引缓存 |
| `conda clean --packages` | ⚠ 清理判定为未使用的解压包缓存 |
| `conda list -n myenv --revisions` | 查看 Conda 事务及修订号 |
| `conda install -n myenv --revision 2 --dry-run` | 预览回滚到修订 2 |
| `conda install -n myenv --revision 2` | ⚠ 回滚 Conda 包状态，不撤销 Pip 操作 |

> 包缓存清理可能破坏通过符号链接引用缓存的环境；先预览再清理。

## Pip

| 命令 | 作用 |
| --- | --- |
| `conda install -n myenv pip` | 为目标环境安装 Pip |
| `conda run -n myenv python -m pip --version` | 查看目标环境 Pip 的版本和路径 |
| `conda run -n myenv python -m pip install rich` | 向目标环境安装 Pip 包 |
| `conda run -n myenv python -m pip check` | 检查 Python 包依赖冲突 |
| `conda run -n myenv python -m pip freeze > requirements.txt` | 导出包版本清单 |
| `conda run -n myenv python -m pip install -r requirements.txt` | 按 Pip 清单安装 |

> 先 Conda 后 Pip；`freeze` 也可能包含 Conda 安装的 Python 包。

## 诊断

| 命令 | 作用 |
| --- | --- |
| `conda config --show solver` | 查看求解器 |
| `conda config --set solver libmamba` | 持久设置求解器 |
| `conda install numpy --solver libmamba` | 本次使用 libmamba 安装 |
| `conda install numpy --dry-run -vv` | 预览安装并输出调试日志 |
| `conda run -n myenv python -c "import sys; print(sys.executable)"` | 查看解释器路径 |
