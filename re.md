# Python re 正则基础速查

`re` 是 Regular Expression（正则表达式）模块，Python 内置，用于文本模式匹配。
按五类查阅：字符、次数、位置、分组与条件、常用接口。
规则通常写成 `r"..."`，让反斜杠交给正则引擎处理，如 `r"\d+"`。

[TOC]

## 字符

**决定“匹配什么字符”。单个字符规则默认只匹配一个字符。**

| 规则 | 含义 |
| --- | --- |
| `abc` | 连续匹配文本 abc |
| `.` | 任意一个字符，默认不含换行符 `\n` |
| `\.` | 字面量点号 |
| `\d` / `\D` | Unicode 十进制数字 / 非此类数字 |
| `\w` / `\W` | Unicode 字母数字或下划线 / 非此类字符 |
| `\s` / `\S` | 空白字符 / 非空白字符 |
| `[abc]` | a、b、c 中的任意一个 |
| `[a-z]` | 一个 ASCII 小写字母，默认区分大小写 |
| `[0-9]` | 一个 ASCII 数字 |
| `[^abc]` | 除 a、b、c 外的任意一个字符 |
| `\n` / `\t` | 换行符 / 制表符 |

```python
import re

assert re.findall(r"\d", "a12") == ["1", "2"]
assert re.findall(r"[ab]", "abc") == ["a", "b"]
assert re.search(r"\.", "a.txt").group() == "."
```

> `\w` 包括中文，不只是英文字母；`\s` 包括空格、制表符和换行等。
> 严格要求数字 0～9 时用 `[0-9]`，不要默认 `\d` 只匹配 ASCII 数字。

## 次数

**决定“前面的字符或分组重复几次”。**

| 规则 | 含义 |
| --- | --- |
| `a?` | a 出现 0 或 1 次 |
| `a*` | a 出现 0 次或多次 |
| `a+` | a 出现 1 次或多次 |
| `a{3}` | a 恰好出现 3 次 |
| `a{2,4}` | a 出现 2～4 次 |
| `a{2,}` | a 至少出现 2 次 |
| `.*` | 贪婪匹配：优先尽可能多，再按需回溯 |
| `.*?` | 非贪婪匹配：优先尽可能少，再按需扩展 |

```python
import re

assert re.findall(r"\d+", "a12 b345") == ["12", "345"]
assert re.fullmatch(r"[0-9]{6}", "123456") is not None
assert re.findall(r'".*"', '"a" "b"') == ['"a" "b"']
assert re.findall(r'".*?"', '"a" "b"') == ['"a"', '"b"']
```

> `\d` 匹配一个数字，`\d+` 匹配一串数字。
> `*` 允许零次，可能产生空匹配；要求至少一个字符时用 `+`。

## 位置

**决定“在哪里匹配”。位置规则本身不消耗字符。**

| 规则 | 含义 |
| --- | --- |
| `^` | 字符串开头 |
| `$` | 字符串末尾，或末尾换行符之前 |
| `\b` | 单词边界，如词字符与非词字符的交界 |
| `^hello` | 要求 hello 出现在开头 |
| `world$` | 要求 world 出现在结尾位置 |
| `\bcat\b` | 独立词 cat，不匹配 cats 或 cat_name |

```python
import re

assert re.search(r"^hello", "hello world") is not None
assert re.search(r"^hello", "say hello") is None
assert re.findall(r"\bcat\b", "cat cats cat_name") == ["cat"]
```

> 校验整个字符串优先用 `re.fullmatch()`，避免 `$` 对末尾换行的特殊处理。
> `\b` 依据 `\w` 判断边界，不是中文分词工具；注意使用原始字符串 `r"..."`。

## 分组与条件

**分组用于组织规则或提取内容；断言只检查相邻条件，不把条件纳入匹配。**

| 规则 | 含义 |
| --- | --- |
| `(abc)` | 捕获组：将 abc 作为一组，并保存匹配内容 |
| `(?P<name>abc)` | 命名捕获组：将匹配内容保存到 name 组 |
| `\1` / `\2` | 反向引用：匹配第 1 / 第 2 个捕获组已捕获的相同文本 |
| `(?P=name)` | 反向引用：匹配命名组 name 已捕获的相同文本 |
| `(?:abc)` | 非捕获组：将 abc 作为一组，但不单独保存内容 |
| `(?:ab)+` | 将整体 ab 重复 1 次或多次 |
| `X(?=Y)` | 匹配 X，要求后面紧跟 Y；正向前瞻 |
| `X(?!Y)` | 匹配 X，要求后面不紧跟 Y；负向前瞻 |
| `(?<=Y)X` | 匹配 X，要求前面紧邻 Y；正向后顾 |
| `(?<!Y)X` | 匹配 X，要求前面不紧邻 Y；负向后顾 |

分支用 `|` 表示，例如 `cat|dog` 匹配 `cat` 或 `dog`。

```python
import re

match = re.search(r"name=(\w+)", "name=alice")
assert match.group(0) == "name=alice"
assert match.group(1) == "alice"

assert re.fullmatch(r"(?:cat|dog)", "dog") is not None
assert re.search(r"\d+(?=元)", "价格 100元").group() == "100"
assert re.search(r"(?<=ID:)\d+", "ID:42").group() == "42"
```

> `group(0)` 是完整匹配，`group(1)` 是第一个捕获组；编号从 1 开始。
> `(?:...)` 不捕获，但仍会消耗字符；`(?=...)` 只检查条件，不消耗字符。
> 完整前瞻语法是 `(?=...)`，不能单独写 `?=`。
> 标准库 `re` 的后顾要求固定长度，不能写 `(?<=a+)`。

反向引用复用的是**已经匹配到的文本**，不是重新执行一次组内规则：

```python
import re

assert re.fullmatch(r"(\w+)-\1", "abc-abc") is not None
assert re.fullmatch(r"(\w+)-\1", "abc-xyz") is None
assert re.fullmatch(r"(?P<word>\w+)-(?P=word)", "abc-abc") is not None
```

> 第一个例子中，`(\w+)` 捕获 abc，`\1` 就要求后面也必须是 abc。
> `(?:...)` 不保存捕获内容，也不占捕获组编号，不能通过编号引用它。

## 常用接口

**先选择匹配方式，再读取结果；替换和分割使用各自的接口。**
表中 `pattern` 为正则规则，`text` 为待处理文本，`repl` 为替换内容。

| 写法 | 含义 |
| --- | --- |
| `re.search(pattern, text)` | 任意位置找首个匹配，返回 Match 或 None |
| `re.match(pattern, text)` | 只从开头匹配，不要求覆盖整个字符串 |
| `re.fullmatch(pattern, text)` | 要求整个字符串符合规则，返回 Match 或 None |
| `re.findall(pattern, text)` | 返回全部非重叠匹配组成的列表 |
| `re.sub(pattern, repl, text)` | 替换匹配内容，返回新字符串 |
| `re.split(pattern, text)` | 按规则分割，返回列表 |
| `re.compile(pattern)` | 编译为可复用对象，如 `pattern.search(text)` |
| `match.group()` | 读取完整匹配文本，等价于 group(0) |
| `match.group(1)` | 读取第一个捕获组 |
| `match.groups()` | 返回全部捕获组组成的元组 |

```python
import re

match = re.search(r"\d+", "id=42")
if match is not None:
  assert match.group() == "42"

assert re.fullmatch(r"\d+", "id=42") is None
assert re.findall(r"\d+", "a12 b34") == ["12", "34"]
assert re.sub(r"\s+", " ", "a   b") == "a b"
assert re.split(r"[,;]", "a,b;c") == ["a", "b", "c"]

pattern = re.compile(r"hello", flags=re.I)
assert pattern.search("HELLO") is not None
```

| 常用选项 | 含义 |
| --- | --- |
| `re.I` | 忽略大小写，也可在模式开头写 `(?i)` |
| `re.M` | 让 `^`、`$` 也匹配每行开头和结尾 |
| `re.S` | 让 `.` 也匹配换行符 |

> 读取 `group()` 前先判断匹配是否为 None。
> `findall()` 无捕获组时返回完整匹配；一个捕获组时返回该组内容；多个则返回元组。
> 只需组织规则时用 `(?:...)`，避免意外改变 `findall()` 的结果形状。
