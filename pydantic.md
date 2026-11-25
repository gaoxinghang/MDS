# Pydantic v2 基础速查

Pydantic 是第三方数据校验库。`BaseModel` 定义模型，`ConfigDict` 设置模型默认行为，
`Field()` 和 `StringConstraints` 配置字段，校验装饰器用于编写自定义规则。

[TOC]

## BaseModel

**用类型注解声明字段，创建模型时校验输入，并在允许的情况下转换类型。**

| 写法 | 含义 |
| --- | --- |
| `User(age=18)` | 用关键字参数创建并校验模型 |
| `User.model_validate(data)` | 从字典等输入创建并校验模型 |
| `user.model_dump()` | 将模型导出为字典 |
| `user.model_dump_json()` | 将模型导出为 JSON 字符串 |

```python
from pydantic import BaseModel, ValidationError


class User(BaseModel):
  """表示带有年龄的用户。

  Attributes:
      age: 用户年龄。
  """

  age: int


user = User.model_validate({"age": "18"})
assert user.age == 18
assert user.model_dump() == {"age": 18}

try:
  User(age="abc")
except ValidationError as error:
  assert error.errors()[0]["loc"] == ("age",)
```

> 默认允许部分类型转换；不想把字符串 `"18"` 转为整数时，
> 可用 `User.model_validate({"age": "18"}, strict=True)` 启用严格校验。

## Field

**`Field()` 是字段配置函数，不是装饰器。**
适合声明默认值、简单约束、输入别名等，不用为每条规则都写校验器。

| 写法 | 含义 |
| --- | --- |
| `age: int = Field(ge=0)` | 必填，且不能为负数 |
| `age: int = Field(default=0, ge=0)` | 可省略，默认值为 0 |
| `tags: list[str] = Field(default_factory=list)` | 为每个实例创建默认列表 |
| `name: str = Field(min_length=1, max_length=20)` | 字符串长度为 1～20 |
| `name: str = Field(alias="user_name")` | 默认可用 `user_name` 提供输入 |
| `Field(default=0, validate_default=True)` | 也校验默认值 |

```python
from pydantic import BaseModel, Field, ValidationError


class UserInput(BaseModel):
  """接收用户输入，并校验年龄。

  Attributes:
      age: 非负的用户年龄。
  """

  age: int = Field(default=0, ge=0)


assert UserInput().age == 0

try:
  UserInput(age=-1)
except ValidationError as error:
  assert error.errors()[0]["loc"] == ("age",)
```

> `Field()` 只声明规则，由 Pydantic 执行校验。默认情况下不校验默认值；
> 需要校验时显式设置 `validate_default=True`。

## ConfigDict

**`ConfigDict` 用来设置整个模型的默认行为。**

| 写法 | 含义 |
| --- | --- |
| `model_config = ConfigDict(...)` | 在模型类中设置默认行为 |
| `str_strip_whitespace=True` | 默认去掉字符串两端的空白 |
| `str_min_length=1` | 默认要求字符串至少有一个字符 |
| `frozen=True` | 实例创建后不允许给字段重新赋值 |

```python
from pydantic import BaseModel, ConfigDict


class Note(BaseModel):
  """表示一条有标题的笔记。

  Attributes:
      title: 笔记标题。
  """

  model_config = ConfigDict(
    str_strip_whitespace=True, str_min_length=1, frozen=True,
  )
  title: str


assert Note(title="  Python  ").title == "Python"
```

> `title` 不需要单独写去空白规则，会直接沿用模型配置。

## StringConstraints

**`StringConstraints` 配合 `Annotated`，给单个字符串字段加规则。**

| 写法 | 含义 |
| --- | --- |
| `Annotated[str, StringConstraints(...)]` | 字段仍是 `str`，由 Pydantic 执行附加规则 |
| `pattern=r"^[A-Za-z0-9_]+$"` | 整个字符串只能含英文字母、数字和下划线，且不能为空 |
| `strip_whitespace=False` | 当前字段不去掉首尾空白，覆盖模型的默认设置 |

```python
from typing import Annotated

from pydantic import BaseModel, ConfigDict, StringConstraints, ValidationError


class MemoryRecord(BaseModel):
  """表示一条带标识和简介的记忆。

  Attributes:
      name: 只含英文字母、数字和下划线的标识。
      description: 记忆的简短介绍。
  """

  model_config = ConfigDict(str_strip_whitespace=True)

  name: Annotated[
    str,
    StringConstraints(strip_whitespace=False, pattern=r"^[A-Za-z0-9_]+$"),
  ]
  description: str


record = MemoryRecord(name="Python_Style2", description=" 缩进偏好 ")
assert record.description == "缩进偏好"

try:
  MemoryRecord(name=" Python_Style2 ", description="缩进偏好")
except ValidationError as error:
  assert error.errors()[0]["loc"] == ("name",)
else:
  raise AssertionError("name 不应该接受首尾空白")
```

> `pattern` 不会自动要求匹配整个字符串；这里用 `^` 和 `$` 限定开头和结尾。
> `description` 沿用默认规则，会去掉首尾空白；`name` 保留空白再校验，因此会拒绝带空白的输入。

## validator

**校验单个字段用 `@field_validator`，校验字段之间的关系用 `@model_validator`。**
能用 `Field()` 表达的简单约束优先用 `Field()`；需要自定义逻辑时再用校验器。

| 写法 | 何时运行 | 适用场景 |
| --- | --- | --- |
| `@field_validator("name", mode="before")` | 字段类型校验前 | 处理原始输入 |
| `@field_validator("name")` | 字段类型校验后，默认 `mode="after"` | 检查或调整该字段的值 |
| `@model_validator(mode="after")` | 模型完成字段校验后 | 通过 `self` 检查多个字段的关系 |

```python
from pydantic import BaseModel, field_validator, model_validator


class Booking(BaseModel):
  """表示一天内的一条预约。

  Attributes:
      name: 预约人姓名。
      start: 开始小时。
      end: 结束小时。
  """

  name: str
  start: int
  end: int

  @field_validator("name")
  @classmethod
  def strip_name(cls, value: str) -> str:
    """去掉姓名两端的空白。

    Args:
        value: 已通过类型校验的姓名。

    Returns:
        去掉两端空白后的姓名。
    """
    return value.strip()


  @model_validator(mode="after")
  def check_order(self):
    """检查结束时间是否晚于开始时间。

    Returns:
        校验通过的当前实例。

    Raises:
        ValueError: 结束时间不晚于开始时间。
    """
    if self.end <= self.start:
      raise ValueError("结束时间必须晚于开始时间")
    return self


assert Booking(name=" Alice ", start=9, end=10).name == "Alice"
```

> `@field_validator` 无法依赖已构造完成的 `self`，不能写成实例方法。
> 常见写法是把 `@classmethod` 放在它下面；不需要 `cls` 时也可以用静态方法。
> 字段校验器返回处理后的字段值，`mode="after"` 的模型校验器返回 `self`。
