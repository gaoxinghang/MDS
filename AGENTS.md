# ==基本信息==

- 姓名：高兴航
- 性别：男
- 年龄：21 岁
- 就读情况：西安电子科技大学大四学生
- 电脑配置：MacBook Pro（MacBookPro18,3），Apple M1 Pro 芯片（8 核 CPU、14 核 GPU），16 GB 统一内存，512 GB SSD（系统报告容量约 500 GB），macOS 27.0.1。

# ==行为准则==

如果没有明确要求进行改动或执行某个操作，则不要自行执行，只把对应的命令提供给用户，由用户自行决定是否运行。

# ==语言风格==

以后回答都使用中文。

默认用 ELI5 的方式和我说话：用简单的白话，先说结论，再说必要的原因。关键风险、限制和验证结果直接告诉我；如果我要求，再深入讲解。

# ==代码风格与注释规范==

## 代码风格

编写、审查或重构代码前，先读取并遵循：
`/Users/gaoxinghang/.pi/agent/skills/karpathy-guidelines/SKILL.md`。

保持改动最小、易读、便于审查，避免无需求依据的防御检查、兜底逻辑和抽象。
保留真实失败场景和安全边界所需的校验，不为追求简短而掩盖错误。

**同类处理使用一致的写法。** 对多个值执行同一类处理时，尽量在同一个逻辑块中完成，再一起使用。
除非处理时机或用途确实不同，不要一个提前处理、另一个内联在调用参数里。
例如，两段文本都要截断时，先截断两段文本，再组装 JSON。

## 代码格式

- 缩进使用 2 个空格，不使用制表符（Tab）。
- 每行不超过 80 个字符。
- 变量和函数使用 snake_case，例如 `my_variable`。
- 类名使用 PascalCase。
- 函数与方法之间空两行。
- 逻辑块之间空一行。

## 注释

函数和类的说明使用文档字符串，写在定义的下一行，用三个双引号 `"""` 包起来。

第一行用一句话说明用途，不超过 80 个字符，以句号结尾。后面还有内容时，先空一行再写。

### 函数与方法

公开接口、逻辑不短、或看名字看不出用意的函数，都要写文档字符串。只看文档字符串就应能正确调用。

小节顺序为 `Args:`、`Returns:`（生成器用 `Yields:`）、`Raises:`。标题单独一行，以冒号结尾。参数说明缩进 4 个空格，同一文件内保持一致。

- `Args:` 每个参数一行，名字后加冒号和说明。没有类型标注时，说明里要写类型。
- `Returns:` 说明返回值的含义。只返回 `None` 时可以不写。
- `Raises:` 列出调用方需要知道的异常。

```python
def add(left: int, right: int) -> int:
    """计算两数之和。

    Args:
        left: 左操作数。
        right: 右操作数。

    Returns:
        两数之和。
    """
    return left + right
```

### 类

类的文档字符串说明这个实例代表什么。公开属性写在 `Attributes:` 里，格式与 `Args:` 相同。`__init__` 单独写文档字符串，参数放在它的 `Args:` 里。用 `@property` 声明的属性写在属性自己的文档字符串里。

```python
class Counter:
    """从给定起点累加的计数器。

    Attributes:
        value: 当前计数值。
    """

    def __init__(self, start: int = 0):
        """用起始值创建计数器。

        Args:
            start: 初始计数值。
        """
        self.value = start
```

### 行内注释

行内注释写在代码行末尾，使用 `#`，与代码至少隔两个空格。只说明代码这样写的原因。

### TODO 注释

TODO 必须写上负责人姓名，便于追踪。例如：`# TODO gaoxinghang: fix this`。

### 配置文件注释

以后编写或修改 config、.env、.gitignore 等配置类文件时，按功能使用三行横幅注释分块，不使用单行横幅标题。格式如下：

```text
##################################################
# 数据预处理
##################################################
```

- 上下分隔线各使用 50 个 `#`，中间使用 `# ` 加中文标题。
- 三行保持相同缩进，顶层注释不缩进；块与块之间留一个空行。
- 按功能分块，不为每个配置项单独添加横幅，每行不超过 80 个字符。
- 仅在支持 `#` 注释的文件中使用上述格式；其他格式使用其合法注释语法，JSON 等不支持注释的格式不添加注释。

# ==Git Commit 规范==

提交信息采用 Conventional Commits 风格，标题和正文均使用英文。

## 格式

```text
<type>[(scope)][!]: <description>
```

- `type` 和可选的 `scope` 使用小写英文；描述以小写英文动词开头。
- 描述使用祈使句，例如 `add`、`fix`、`update`，不使用过去式。
- 标题简洁明确，不以句号结尾，尽量不超过 72 个字符。
- `scope` 按需填写，表示受影响的模块，例如 `config`、`cli`。
- 正文按需添加，与标题空一行，说明改动原因和必要背景。
- 破坏性变更在类型或作用域后加 `!`，并在正文后的页脚中使用
  `BREAKING CHANGE: ...` 说明影响及迁移方式。

## 类型

按变更的实际目的选择类型，不将所有改动都归为 `chore`。

- `feat`：新增功能。
- `fix`：修复缺陷。
- `docs`：仅文档变更。
- `style`：仅格式、空白或注释排版调整，不改变程序行为。
- `refactor`：代码重构，不新增功能或修复缺陷。
- `perf`：性能优化。
- `test`：新增或调整测试。
- `build`：构建系统、打包或依赖变更。
- `ci`：持续集成与自动化流水线变更。
- `chore`：其他维护工作，例如编辑器配置、忽略规则等。
- `revert`：回退已有提交。

## 提交原则与示例

每次提交围绕一个目的；互不相关的改动尽量拆分提交。
配置文件变更也按目的分类，例如修复配置错误用 `fix`，仅调整注释分块用 `style`。

```text
feat(weather): add hourly forecasts
fix(config): handle missing api keys
style(config): standardize section comments
chore(git): ignore vscode settings
```

# ==下载文件归类==

以后下载或保存到本机的文件，按类型放进家目录对应的文件夹，不要堆在家目录根目录。访达里显示的是中文名，实际路径用下面的英文目录。

- 视频（mp4、mov、mkv、webm 等）放到 `~/Movies`（影片）。
- 图片（png、jpg、jpeg、gif、webp、heic 等）放到 `~/Pictures`（图片）。
- 音频（mp3、wav、flac、m4a 等）放到 `~/Music`（音乐）。
- Markdown 文档（md）放到 `~/Documents/MDS`（文稿/MDS）。
- 其他文档（pdf、doc、docx、ppt、xls、txt 等）放到 `~/Documents`（文稿）。
- 对不上上面几类的文件放到 `~/Downloads`（下载）。

用户指定了保存位置时，按用户说的位置放。项目里的代码和仓库文件不按这条归类。

# ==浏览器操作==

浏览器操作使用 `playwright-browser` Skill 和本机 Playwright CLI 包装脚本。
先读技能说明；默认直接连接我已打开的 Chrome，复用当前个人资料和登录状态，
不要另开独立浏览器，除非我明确要求。

1. 确认 Chrome 正在运行，当前个人资料已启用 Playwright MCP Bridge 扩展。
   Pi 从启动它的 zsh 继承 `~/.zshrc` 导出的
   `PLAYWRIGHT_MCP_EXTENSION_TOKEN`。用扩展连接，不需要每次点「允许」：

   ```bash
   pw="$HOME/.pi/agent/npm/node_modules/pi-playwright/skills"
   pw="$pw/playwright-browser/scripts/pw.js"
   node "$pw" -s=my-chrome attach --extension=chrome
   ```

2. 后续命令沿用 `-s=my-chrome`；每个新 shell 先设好上面的 `pw`。
   用 `tab-list` 查标签页，`tab-select <编号>` 切换。目标页没打开时，
   用 `tab-new <网址>` 在当前 Chrome 开新标签页，不覆盖无关页面。
3. 用 `snapshot` 读取页面和元素引用，再用实际引用执行
   `click e4`、`fill e5 "文本"` 等。页面更新后重新取快照，操作后核对结果。
   只操作本次任务涉及的页面。
4. 做完用 `node "$pw" -s=my-chrome detach` 断开，不用 `close`，
   保留 Chrome 和标签页。
5. 如果 Pi 没继承变量，在启动 Pi 的终端执行 `source ~/.zshrc`，
   退出并重新启动 Pi。只检查变量是否存在，不打印、索要或记录 token。
   如果仍连接失败，确认扩展在当前 Chrome 个人资料内启用；个人资料可在
   `chrome://version` 核对。若报 `Operation not permitted`，检查
   macOS「隐私与安全性 → 文件与文件夹」里终端的「Google Chrome」权限
   （若有），不要默认重装扩展或开启「完全磁盘访问」。

# ==电脑操作==

以后涉及电脑桌面操作，都使用 `agent-desktop` Skill。
操作前先读取并遵循：
`/Users/gaoxinghang/.pi/agent/skills/agent-desktop/SKILL.md`。
浏览器操作仍按上面的「浏览器操作」规范执行。

