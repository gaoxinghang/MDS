# Python 类型注解速查

`typing` 提供类型注解工具，主要供 IDE 和静态类型检查器使用。
Python 默认不会根据注解在运行时检查值的类型。本文示例使用 Python 3.11+。

[TOC]

## Any

**表示暂不限制类型，类型检查器通常不会检查对该值的操作。**
适合逐步补全旧代码的注解，或暂时接入类型不明确的接口。

| 写法 | 含义 |
| --- | --- |
| `value: Any` | 可接收任意值，类型检查器通常放行属性访问和方法调用 |
| `value: object` | 也可接收任意值，但使用类型特有的方法前需要先缩小类型范围 |

```python
from typing import Any

value: Any = "python"
assert value.upper() == "PYTHON"
```

> 如果 `value` 实际是整数，调用 `value.upper()` 仍会在运行时报错。
> 只知道值的类型不确定、但仍想让检查器拦住不安全的操作时，用 `object`。

## Optional

**`Optional[T]` 等同于 `T | None`，表示值可以是 `None`。**
能不能省略参数，由有没有默认值决定，与 `Optional` 无关。

| 写法 | 含义 |
| --- | --- |
| `name: Optional[str]` | 必须传入参数，可以传字符串或 `None` |
| `name: Optional[str] = None` | 可以不传参数，默认用 `None` |

Python 3.10+ 也可以写 `name: str | None` 或 `name: str | None = None`。

```python
from typing import Optional


def greet(name: Optional[str] = None) -> str:
  """返回一条问候语。

  Args:
      name: 要问候的名字；不提供时使用通用称呼。

  Returns:
      问候语。
  """
  return "你好，访客" if name is None else f"你好，{name}"


assert greet() == "你好，访客"
assert greet("小明") == "你好，小明"
```

## Literal

**把值限制在几个具体选项中，适合标注模式、状态和固定命令。**
`Literal["r", "w"]` 允许的是字符串 `"r"` 和 `"w"`，而不是任意字符串。

```python
from typing import Literal

mode: Literal["r", "w"] = "r"
```

> 如果给 `mode` 赋值 `"x"`，静态类型检查器会报错，Python 本身不会阻止赋值。

## Annotated

**`Annotated[T, 元数据]` 给类型 `T` 附加信息，不改变它的基础类型。**
元数据可以供框架或自定义工具读取，例如字段说明、校验规则或参数来源。

```python
from typing import Annotated

Age = Annotated[int, "年龄，单位为年"]
age: Age = 18
```

> `Annotated` 不会自行处理元数据或校验值。比如 Pydantic 会读取
> `Annotated[int, Field(ge=0)]` 中的 `Field()` 并执行非负校验。
> 用 `get_type_hints(..., include_extras=True)` 可以保留元数据；默认会去掉它。

## TypedDict

**描述字典的固定键，以及各个键对应的值类型。**
适合标注结构已知的字典，例如接口返回的数据。运行时它仍然是普通字典。

```python
from typing import NotRequired, TypedDict


class User(TypedDict):
  """表示一个用户字典。

  Attributes:
      name: 用户名，必填。
      email: 邮箱地址，可以省略。
  """

  name: str
  email: NotRequired[str]


user: User = {"name": "Alice"}
assert user["name"] == "Alice"
```

> 默认键是必填的。`NotRequired[str]` 表示键可以缺失，
> `str | None` 表示值可以是 `None`，并不表示键可以缺失。

## Callable

**`Callable[[参数类型, ...], 返回类型]` 用来标注回调等可调用对象。**
例如 `Callable[[int], str]` 表示接收一个整数并返回字符串。

```python
from collections.abc import Callable


def format_number(value: int, formatter: Callable[[int], str]) -> str:
  """用指定的回调格式化整数。

  Args:
      value: 待格式化的整数。
      formatter: 接收整数并返回字符串的回调。

  Returns:
      回调返回的字符串。
  """
  return formatter(value)


assert format_number(42, str) == "42"
```

> 新代码可从 `collections.abc` 导入 `Callable`；旧代码里常见 `typing.Callable`。
> `Callable[[], str]` 表示不接收参数；`Callable[..., str]` 不限制参数签名。

## TypeVar

**定义类型变量，用来表达多个注解之间的类型关系。**
例如，参数是 `T`，返回值是 `list[T]`，类型检查器就能根据传入值推断列表元素类型。

```python
from typing import TypeVar

T = TypeVar("T")


def repeat(value: T, count: int) -> list[T]:
  """把一个值重复放入列表。

  Args:
      value: 要放入列表的值。
      count: 重复次数。

  Returns:
      重复指定次数得到的列表。
  """
  return [value] * count


numbers = repeat(2, 3)
words = repeat("hi", 2)
assert numbers == [2, 2, 2]
assert words == ["hi", "hi"]
```

> 类型检查器会推断 `numbers` 是 `list[int]`，`words` 是 `list[str]`。
> 如果把返回类型写成 `list[Any]`，就会丢掉这种输入与输出的类型联系。
