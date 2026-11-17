# Conda 常用命令与最佳实践速查手册

> **适用场景**：日常 Python 环境隔离、跨平台依赖管理、科学计算包安装及环境迁移复现。  
> **使用说明**：示例中的 `myenv` 为环境名称，`3.11` 为 Python 版本，使用时请按需替换。  
> **平台支持**：Linux / macOS (Terminal) / Windows (Anaconda Prompt / PowerShell)。

[TOC]

---

## 1. 核心概念与工作机制

在开始使用命令前，建立清晰的概念模型能避免 90% 的依赖冲突：

| 核心概念 | 作用与定位 | 关键注意点 |
| :--- | :--- | :--- |
| **Conda** | 跨语言的环境与二进制包管理器 | 不仅能管 Python 包，还能管理 C/C++ 依赖库、CUDA 驱动、R 等 |
| **环境 (Environment)** | 相互独立的文件目录集合 | 拥有独立的 Python 解释器及二进制库，彻底隔离不同项目依赖 |
| **base 环境** | 安装 Conda 时的默认基础环境 | ⚠️ **强烈建议保持 base 干净**，仅用于管理 Conda 本身，项目务必新建环境 |
| **渠道 (Channel)** | 软件包的托管仓库源 | 常用有官方 `defaults`、社区主力 `conda-forge`，以及国内镜像源 |
| **pip** | Python 官方生态包管理器 | 主要从 PyPI 安装纯 Python 轮子包，不负责管理 Python 解释器和系统级依赖 |

> 💡 **核心原则**：**“一个项目一个环境，装包先看当前环境，优先 Conda 后用 Pip。”**

---

## 2. 高频命令极速索引（80% 日常场景）

日常开发最常用的 12 条核心指令，可直接复制参考：

| 场景分类 | 目标操作 | 核心命令 | 简要说明 |
| :--- | :--- | :--- | :--- |
| **环境管理** | 创建独立环境 | `conda create -n myenv python=3.11` | 指定 Python 版本创建新环境 |
| | 查看所有环境 | `conda env list` | 带 `*` 号者为当前激活环境 |
| | 激活指定环境 | `conda activate myenv` | 终端提示符会变为 `(myenv)` |
| | 退出当前环境 | `conda deactivate` | 回退至 base 或上一层终端环境 |
| | 删除整个环境 | `conda env remove -n myenv` | 彻底清理该环境及其所有依赖 |
| **包管理** | 安装指定包 | `conda install numpy pandas` | 自动解析并安装依赖 |
| | 查看已装包 | `conda list` | 列出当前环境所有包及对应渠道 |
| | 更新指定包 | `conda update numpy` | 仅更新该包及其直接依赖 |
| | 卸载指定包 | `conda remove numpy` | 移除包及专属依赖 |
| **环境迁移** | 跨平台导出配置 | `conda env export --from-history > env.yml` | 仅导出显式安装的包（最推荐） |
| | 从配置重建环境 | `conda env create -f env.yml` | 根据 yml 文件自动化构建 |
| **日常维护** | 清理包与索引缓存 | `conda clean --all` | 释放磁盘空间（安全有效） |

---

## 3. 环境全生命周期管理

### 3.1 创建环境 (`conda create`)

```bash
# 1. 创建指定 Python 版本的纯净环境（最常用）
conda create -n myenv python=3.11

# 2. 创建环境并一并安装常用基础科学计算包
conda create -n myenv python=3.11 numpy pandas matplotlib

# 3. 指定从特定渠道（如 conda-forge）创建环境
conda create -n myenv -c conda-forge python=3.11

# 4. 指定磁盘绝对路径创建环境（适用于项目根目录本地隔离）
conda create -p ./env python=3.11
```

### 3.2 激活与退出环境 (`activate` / `deactivate`)

```bash
# 激活命名环境
conda activate myenv

# 激活通过路径创建的环境
conda activate ./env

# 退出当前激活的环境（返回上一层环境或 base）
conda deactivate
```

### 3.3 查看与环境信息确认 (`info` / `list`)

```bash
# 查看所有已创建的环境列表（带 * 号表示当前正激活的环境）
conda env list
# 或使用等价命令：
conda info --envs

# 验证当前正在调用的 Python 解释器实际绝对路径（排查解释器指向错误神器）
python -c "import sys; print(sys.executable)"

# 查看 Conda 全局诊断信息（包含平台架构、环境路径、缓存路径等）
conda info
```

### 3.4 克隆环境 (`--clone`)

在做大型库升级、测试新功能或调参前，建议先复制一份快照环境：

```bash
# 将现有环境 myenv 完整复制一份为 newenv（包含所有包）
conda create -n newenv --clone myenv
```

### 3.5 删除环境 (`env remove`)

```bash
# ⚠️ 第一步：如果当前正处于该环境中，务必先退出
conda deactivate

# 第二步：彻底删除指定环境及其包含的所有安装包
conda env remove -n myenv

# 删除按路径创建的环境
conda env remove -p ./env
```

> ⚠️ **高危提醒**：
> - 绝对不要执行 `conda env remove -n base`，base 环境损坏将导致整个 Conda 需要重装。
> - 切勿把代码工程直接放在 Conda 的环境目录中，代码与环境应当目录分离。

---

## 4. 软件包管理（安装 / 查询 / 更新 / 卸载）

> 📌 **作用域说明**：以下命令若不额外加 `-n <env_name>`，默认均针对**当前已激活的环境**。

### 4.1 安装软件包 (`conda install`)

```bash
# 1. 安装单个或多个包
conda install numpy
conda install numpy pandas scikit-learn

# 2. 安装指定具体版本（需在配置的渠道中存在）
conda install numpy=1.26.4

# 3. 模糊版本约束安装（必须加双引号，防止 shell 误解析 > 或 < 字符）
conda install "numpy>=1.24,<2.0"

# 4. 指定从特定渠道下载安装（本次生效）
conda install -c conda-forge opencv

# 5. 免激活直接向目标环境装包（适合批量脚本执行）
conda install -n myenv requests

# 6. 演练/试运行（仅预览依赖计算方案，不真正执行下载与安装）
conda install pytorch --dry-run
```

### 4.2 查询软件包 (`conda list` / `conda search`)

```bash
# 列出当前环境中已安装的所有包（展示名称、版本、构建号和安装渠道）
conda list

# 过滤搜索当前环境已装的特定包（支持模糊匹配）
conda list torch

# 在远程可用渠道中搜索特定包的所有可用版本及依赖信息
conda search numpy

# 查看某个包的详细元信息（平台支持、依赖依赖项等）
conda search numpy --info
```

### 4.3 更新软件包 (`conda update`)

```bash
# 1. 更新指定包到当前渠道允许的最高安全兼容版本
conda update numpy

# 2. 更新当前环境中的所有包（⚠️ 生产环境慎用，可能引起依赖大范围变动）
conda update --all

# 3. 仅更新 base 环境下的 Conda 自身程序
conda update -n base conda
```

### 4.4 卸载软件包 (`conda remove`)

```bash
# 卸载指定包（Conda 会一并分析移除不再被其他包需要的孤立依赖）
conda remove numpy

# 从指定环境中卸载包
conda remove -n myenv numpy

# 卸载预览（先检查会附带卸载哪些关联依赖）
conda remove numpy --dry-run
```

---

## 5. 渠道管理与镜像加速 (Channels)

### 5.1 常用渠道与规则

`conda-forge` 是社区维护最活跃、支持包最全面的渠道，建议作为主力来源。

```bash
# 查看当前生效的渠道列表及优先级次序
conda config --show channels

# 查看所有配置来源（包含具体生效的 .condarc 配置文件路径）
conda config --show-sources
```

### 5.2 渠道配置命令

```bash
# 1. 添加 conda-forge 为高优先级源
conda config --add channels conda-forge

# 2. 开启严格渠道优先级（极大减少跨渠道混用引起的依赖求解死锁）
conda config --set channel_priority strict

# 3. 移除指定渠道
conda config --remove channels conda-forge
```

### 5.3 国内高校镜像源配置参考（以清华源为例）

如果直连官方源下载较慢，可通过修改用户目录下的 `~/.condarc` 文件配置国内镜像：

```yaml
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
```

> 💡 **配置提示**：
> - 运行 `conda clean -i` 可清除索引缓存，使新配置的镜像源立即生效。
> - 同一个环境建议尽量固定使用统一的渠道体系，避免反复混合导致元数据冲突。

---

## 6. 环境导出、共享与跨平台迁移

环境共享是团队协作与论文代码复现的关键，不同导出方式适用场景差异极大：

```
                    ┌─ 1. conda env export > env.yml ───────► 包含具体构建号，跨机器/跨系统易报错
                    │
三种导出方式对比 ────┼─ 2. conda env export --no-builds ────► 去除构建哈希，同系统平台复现较好
                    │
                    └─ 3. conda env export --from-history ──► ⭐ 仅包含手动声明包，跨系统最稳健
```

### 6.1 导出方式详解

#### 方式 A：推荐跨平台共享（`--from-history`）⭐

仅记录显式输入的包名，让目标机器自行根据平台匹配依赖，兼容性最好：

```bash
# 导出当前环境的手动安装清单
conda env export --from-history > environment.yml

# 导出指定环境
conda env export -n myenv --from-history > environment.yml
```

> ⚠️ **注意**：`--from-history` 不会自动记录通过 `pip install` 安装的包，若使用了 pip 包需手动在 yml 中补充。

#### 方式 B：同操作系统平台完整导出（`--no-builds`）

```bash
# 导出所有依赖项，但去除平台专属的 build hash 字符串
conda env export --no-builds > environment.yml
```

#### 方式 C：绝对一致复现（精确 spec 文件，仅限同 OS、同 CPU 架构）

```bash
# 导出包含 URL 的精确二进制列表（速度极快但完全绑定平台与架构）
conda list --explicit > conda-spec.txt

# 在同架构目标机上一键复现
conda create -n newenv --file conda-spec.txt
```

### 6.2 从配置文件创建或同步环境

```bash
# 1. 根据 yml 文件创建新环境（默认使用 yml 中声明的 name）
conda env create -f environment.yml

# 2. 创建环境但自定义环境名称（覆盖 yml 内部的 name 字段）
conda env create -n custom_name -f environment.yml

# 3. 根据 yml 更新现有环境
conda env update -n myenv -f environment.yml

# 4. 更新并清理不再需要的旧包（--prune 会删除 yml 中未包含的旧依赖）
conda env update -n myenv -f environment.yml --prune
```

### 6.3 标准手写 `environment.yml` 模板（团队交付标准）

```yaml
name: project_env
channels:
  - conda-forge
  - nodefaults              # 排除默认源，完全由 conda-forge 统一管理
dependencies:
  - python=3.11
  - numpy>=1.26
  - pandas>=2.0
  - matplotlib
  - scikit-learn
  - pip
  - pip:
      - rich>=13.0          # 仅在 conda 中没有或需要最新补丁时使用 pip
      - fastapi
      - uvicorn
```

---

## 7. 在 Conda 中混用 Pip 的“黄金法则”

很多开发者的环境崩溃，根本原因在于错误地交叉混用 Conda 和 Pip。

### 7.1 正确操作步骤

```bash
# 1. 确保已激活目标环境
conda activate myenv

# 2. 检查环境中是否已正确安装 pip
conda list pip

# 3. 必须通过当前环境的 Python 解释器调用 pip（核心技巧，防止误调全局 pip）
python -m pip install package_name

# 4. 导出 pip 格式的依赖文件
python -m pip freeze > requirements.txt

# 5. 从 requirements.txt 批量安装
python -m pip install -r requirements.txt
```

### 7.2 六大铁律（避免环境被玩坏）

1. **先后顺序铁律**：永远**先用 Conda 安装所有底层基础库**（Python、CUDA、NumPy 等），最后才用 Pip 安装纯 Python 库。
2. **命令调用铁律**：永远使用 `python -m pip install`，严禁直接输入 `pip install`，防止 PATH 错乱污染全局。
3. **隔离权限铁律**：在虚拟环境中**绝对不要使用 `pip install --user`**，这会把包安装至全局用户目录，击穿虚拟环境的隔离屏障。
4. **单向不可逆**：一旦在环境中运行了 pip 安装，**不要再回头去执行 `conda install` 大改基础包**，极易破坏依赖链接。如需重大修改，建议直接重建环境。
5. **记录区分**：`pip freeze` 会把 Conda 预装的 Python 库也当成 pip 包导出；记录纯 pip 依赖时需手动筛查。
6. **Conda 主导**：`requirements.txt` 无法记录 Python 解释器版本和 C 运行时依赖，不能完全替代 `environment.yml`。

---

## 8. 免激活执行与自动化脚本 (`conda run`)

在 Shell 脚本、CI/CD 流水线、Cron 定时任务或第三方 IDE 中，无需显式 `conda activate` 即可直接执行：

```bash
# 1. 使用指定环境的 Python 打印版本
conda run -n myenv python --version

# 2. 免激活直接运行指定脚本
conda run -n myenv python /path/to/main.py

# 3. 实时无缓冲输出（适合耗时长的模型训练或批处理任务）
conda run --no-capture-output -n myenv python train.py
```

---

## 9. 终端初始化与 Shell 行为控制 (`conda init`)

### 9.1 初始化终端

如果新装 Conda 后执行 `conda activate` 报 `Run 'conda init' before activate` 错误，根据当前 Shell 执行一次初始化：

```bash
# macOS 默认 zsh
conda init zsh

# Linux 常见 bash
conda init bash

# Windows PowerShell
conda init powershell
```

> ⚠️ 执行后需**关闭终端窗口并重新打开**生效。

### 9.2 关闭终端默认自动激活 `base`

每次打开新终端自动加载 `(base)` 会拖慢终端启动速度，推荐关闭：

```bash
# 关闭终端启动时自动激活 base 环境（推荐）
conda config --set auto_activate_base false

# 重新开启自动激活
conda config --set auto_activate_base true
```

---

## 10. 瘦身维护与版本回滚 (Clean & Revisions)

### 10.1 磁盘清理 (`conda clean`)

Conda 长时间使用会积累大量 tar.bz2 压缩包和未使用的索引，常占用数十 GB 空间：

```bash
# 1. 演练预览有哪些可以清理的内容
conda clean --all --dry-run

# 2. 一键安全清理：未使用的包缓存、下载的压缩包、索引缓存及历史日志
conda clean --all

# 3. 仅清理下载的包压缩包（.tar.bz2 / .conda）
conda clean --tarballs
```

### 10.2 环境历史版本追踪与后悔药回滚 (`conda list --revisions`)

Conda 拥有版本快照功能，能精准回退到某个历史安装状态：

```bash
# 1. 查看当前环境的历史变更记录（每次 install/update 会生成 rev 编号）
conda list --revisions

# 2. 状态回滚：撤销误操作，回到指定的 revision 编号（如回滚到 rev 2 状态）
conda install --revision 2
```

> 💡 **局限**：回滚仅对 Conda 安装的包有效，无法自动追踪或撤销 pip 执行的操作。

---

## 11. 常见疑难杂症与排错指南

### 诊断 1：安装成功但代码运行报 `ModuleNotFoundError`

```bash
# 第一步：核实当前激活的环境究竟是哪一个
conda env list

# 第二步：核实实际调用的 Python 解释器是否为你以为的环境
python -c "import sys; print(sys.executable)"

# 第三步：检查包是否实际装进了当前环境
conda list 包名

# 第四步：检查调用的是否为当前环境的 pip
python -m pip --version
```

- **常见根因**：
  - VS Code / PyCharm 右下角选择的 Python 解释器仍为系统 Python 或 base 环境。
  - Jupyter Notebook 未注册当前虚拟环境内核（需安装 `ipykernel` 并注册：`python -m ipykernel install --user --name myenv`）。
  - 包安装名称与 Python 导入名称不同（如 `conda install scikit-learn`，导入是 `import sklearn`；`opencv-python` 导入是 `import cv2`）。

---

### 诊断 2：依赖解析缓慢 (Solving environment 极慢)

早期版本的经典求解器处理复杂环境常耗费数十分钟甚至死锁卡住：

1. **升级为 Libmamba 快速求解器（强烈推荐）**：
   ```bash
   # 查看当前求解器
   conda config --show solver

   # 为全局启用全新 libmamba 高性能求解器
   conda config --set solver libmamba

   # 或者单次命令指定使用 libmamba
   conda install numpy --solver=libmamba
   ```
2. **规避建议**：
   - 避免无休止地往一个老旧环境里堆放不兼容的库，遇到大版本升级果断新建环境。
   - 启用严格渠道优先级：`conda config --set channel_priority strict`。

---

## 12. 常用命令行参数速查表

| 短参数 | 全称参数 | 作用说明 | 典型范例 |
| :--- | :--- | :--- | :--- |
| `-n` | `--name` | 指定目标环境名称 | `conda list -n myenv` |
| `-p` | `--prefix` | 指定目标环境的物理绝对路径 | `conda create -p ./env python=3.11` |
| `-c` | `--channel` | 临时指定下载渠道 | `conda install -c conda-forge rich` |
| `-y` | `--yes` | 自动回答 yes（脚本无交互安装必备） | `conda install numpy -y` |
| `-f` | `--file` | 指定依赖清单文件 | `conda env create -f environment.yml` |
| | `--dry-run` | 仅模拟计算求解过程，不实际变更磁盘 | `conda remove scipy --dry-run` |
| | `--from-history` | 仅导出用户主动声明过的依赖包 | `conda env export --from-history` |
| | `--no-builds` | 导出时移除系统特定的构建字符串 | `conda env export --no-builds` |
| `-h` | `--help` | 查看命令或子命令的详细帮助说明 | `conda install --help` |

---

## 13. 推荐的标准工程协作流水线

在项目实际交付或新开项目时，建议严格遵循此标准工作流：

```bash
# 1. 终端进入项目根目录
cd ~/projects/my_awesome_project

# 2. 为项目量身定制独立干净的环境
conda create -n my_awesome_project python=3.11 -y

# 3. 激活新环境
conda activate my_awesome_project

# 4. 优先通过 Conda 安装核心科学计算库
conda install numpy pandas matplotlib scikit-learn -y

# 5. 特殊纯 Python 库通过当前解释器的 pip 补充
python -m pip install python-dotenv tqdm

# 6. 验证环境解释器指向与代码可运行性
python -c "import sys; print(sys.executable)"
python main.py

# 7. 工作完成/准备分享时，导出轻量跨平台环境配置
conda env export --from-history > environment.yml

# 8. 退出环境
conda deactivate
```

---

## 14. 官方权威参考资料

- [Conda 官方核心文档](https://docs.conda.io/projects/conda/en/stable/)
- [Conda 命令行完整参数手册 (Commands Reference)](https://docs.conda.io/projects/conda/en/stable/commands/)
- [Conda 环境管理官方指南 (Managing Environments)](https://docs.conda.io/projects/conda/en/stable/user-guide/tasks/manage-environments.html)
- [Conda-forge 官方社区文档](https://conda-forge.org/docs/)
