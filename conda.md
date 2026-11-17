# Conda 命令速查

按操作对象分为四类；示例环境 `myenv`、包名和路径按需替换。
包操作默认针对当前环境；⚠ 表示需谨慎操作。

[TOC]

## 环境

**环境是独立的 Python 和依赖集合，用来隔离不同项目。**

| 命令 | 作用 |
| --- | --- |
| `conda env list` | 列出环境；`*` 表示当前环境 |
| `conda create -n myenv python=3.11` | 创建命名环境并指定 Python 版本 |
| `conda create -p ./env python=3.11` | 在指定路径创建环境 |
| `conda activate myenv` | 进入命名环境 |
| `conda activate ./env` | 进入路径环境 |
| `conda deactivate` | 退出当前激活层，不删除环境 |
| `conda create -n newenv --clone myenv` | 将 myenv 克隆为 newenv |
| `conda run -n myenv python main.py` | 不激活环境，直接使用它运行脚本 |
| `conda env remove -n myenv --dry-run` | 预览环境删除 |
| `conda env remove -n myenv` | ⚠ 删除命名环境；先退出，不要删除 base |
| `conda env remove -p ./env` | ⚠ 删除指定路径的环境 |

> `-n` 指定环境名称，`-p` 指定环境路径，二选一。
> `activate` 影响当前终端；`run` 只为这次命令使用指定环境。

## 包

**包是安装在环境中的库或工具；先确认目标环境，再安装或卸载。**

| 命令 | 作用 |
| --- | --- |
| `conda list` | 查看当前环境已安装的包 |
| `conda list -n myenv` | 查看指定环境已安装的包 |
| `conda list numpy` | 按正则表达式过滤已安装包名 |
| `conda search numpy` | 搜索渠道中可安装的版本 |
| `conda install numpy pandas` | 安装多个包及所需依赖 |
| `conda install numpy=1.26.4` | 安装或降级到指定版本 |
| `conda install "numpy>=1.26,<2"` | 安装满足版本范围的包 |
| `conda install -n myenv numpy` | 向指定环境安装包 |
| `conda install numpy --dry-run` | 预览安装计划，不实际安装包 |
| `conda update numpy` | 更新到依赖约束允许的版本 |
| `conda update --all` | ⚠ 更新当前环境全部包，可能大幅调整依赖 |
| `conda remove numpy --dry-run` | 预览卸载及连带变更 |
| `conda remove numpy` | 卸载包，可能连带移除依赖它的包 |

Conda 中找不到所需包时，可以在目标环境中使用 Pip：

```bash
conda install -n myenv pip
conda run -n myenv python -m pip install rich
conda run -n myenv python -m pip check
```

> 先 Conda 后 Pip；通过目标环境的 `python -m pip` 避免装错环境。
> `--dry-run` 不实际安装或卸载，但仍可能更新索引缓存。

## 渠道与配置

**渠道（channel）是包的来源；配置决定渠道顺序及终端行为。**

| 命令 | 作用 |
| --- | --- |
| `conda config --show` | 查看合并后的有效配置 |
| `conda config --show-sources` | 查看配置文件来源及内容 |
| `conda config --show channels` | 查看渠道顺序 |
| `conda install -c conda-forge numpy` | 本次优先使用 conda-forge |
| `conda config --prepend channels conda-forge` | 持久添加高优先级渠道 |
| `conda config --set channel_priority strict` | 启用严格渠道优先级 |
| `conda config --remove channels conda-forge` | 删除指定渠道条目 |
| `conda config --validate` | 检查配置是否合法 |
| `conda init zsh` | 初始化 Zsh 的 Conda 支持；执行后重开终端 |
| `conda init zsh --dry-run` | 预览终端初始化修改 |
| `conda config --set auto_activate_base false` | 关闭新终端默认激活 base |

> `-c` 只影响本次操作；`conda config` 通常写入用户配置，影响后续操作。
> `-c conda-forge` 不等于只使用该渠道；只使用它需再加 `--override-channels`。
> 使用 Bash 或 PowerShell 时，将 `conda init zsh` 中的 zsh 换为对应名称。

## 迁移与维护

**迁移用于保存、重建环境；维护用于诊断、清理和回滚。**

| 命令 | 作用 |
| --- | --- |
| `conda env export -n myenv > env.yml` | 导出指定环境描述 |
| `conda env export --from-history > env.yml` | 仅导出显式请求的 Conda 依赖，不含 Pip |
| `conda env create -f env.yml` | 根据 YAML 文件创建环境 |
| `conda env update -n myenv -f env.yml` | 根据 YAML 文件更新已有环境 |
| `conda list --explicit > spec.txt` | 导出精确 Conda 构建，不含 Pip，通常限同平台 |
| `conda create -n newenv --file spec.txt` | 根据精确包清单重建环境 |
| `conda --version` | 查看 Conda 版本 |
| `conda info` | 查看平台、环境和缓存目录 |
| `conda install --help` | 查看子命令帮助；install 可替换 |
| `conda update -n base conda` | 更新 Conda 及所需依赖 |
| `conda clean --all --dry-run` | 预览缓存清理 |
| `conda clean --all` | ⚠ 清理索引、下载归档和未使用包缓存等 |
| `conda list -n myenv --revisions` | 查看环境的 Conda 修订记录 |
| `conda install -n myenv --revision 2 --dry-run` | 预览回滚到修订 2 |
| `conda install -n myenv --revision 2` | ⚠ 回滚 Conda 包状态，不撤销 Pip 操作 |

> `>` 会覆盖目标文件；分享前检查私有源、凭据和本地路径。
> 完整环境导出不保证跨平台兼容；`--from-history` 更精简，但不锁定完整依赖。
> 包缓存清理可能破坏通过符号链接引用缓存的环境；清理、回滚前先预览。
