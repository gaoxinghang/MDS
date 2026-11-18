# Python 常用装饰器速查

装饰器用 `@` 写在函数、方法或类的定义上方，用于改变它的行为或添加功能。
本文介绍五种常用的类与方法装饰器。

[TOC]

## dataclass

**`@dataclass` 装饰类，适合主要用来保存数据的对象。**
它根据类型注解识别字段，默认生成 `__init__`、`__repr__` 和 `__eq__` 等方法。

| 写法 | 含义 |
| --- | --- |
| `@dataclass` | 使用默认设置生成常用方法 |
| `@dataclass(frozen=True)` | 阻止通过正常属性赋值修改字段，不会冻结字段内部的可变对象 |
| `field(default_factory=list)` | 每个实例生成自己的默认列表 |

```python
from dataclasses import dataclass, field


@dataclass
class Note:
  """表示一条带标签的笔记。

  Attributes:
      title: 笔记标题。
      tags: 笔记标签。
  """

  title: str
  tags: list[str] = field(default_factory=list)


first = Note("Python")
second = Note("Python")
assert first == second
first.tags.append("速查")
assert second.tags == []
```

> 类型注解帮助定义字段，但 `@dataclass` 不会自动检查或转换传入值的类型。

## property

**把方法暴露成属性：写 `obj.area`，不用写 `obj.area()`。**
适合读取简单的计算结果，或用 `@area.setter` 控制赋值。

```python
from dataclasses import dataclass


@dataclass
class Rectangle:
  """表示一个矩形。

  Attributes:
      width: 矩形的宽。
      height: 矩形的高。
  """

  width: int
  height: int

  @property
  def area(self) -> int:
    """返回当前宽和高计算出的面积。"""
    return self.width * self.height


assert Rectangle(3, 4).area == 12
```

> 不定义 setter 时，不能直接给该属性赋值。
> `@property` 不会缓存结果，每次读取都会调用 getter。

## staticmethod

**定义静态方法，调用时不自动传入 `self` 或 `cls`。**
适合不依赖实例或类、但逻辑与类密切相关的辅助函数。

```python
class User:
  """表示一个用户。"""

  @staticmethod
  def is_valid_name(name: str) -> bool:
    """检查用户名是否包含非空白字符。

    Args:
        name: 待检查的用户名。

    Returns:
        包含非空白字符时为 True。
    """
    return bool(name.strip())


assert User.is_valid_name("alice")
```

> 不需要先创建实例；通过类或实例调用时，都不会绑定额外的参数。
> 辅助函数如果和类关系不大，直接写成模块函数通常更清楚。

## classmethod

**定义类方法，调用时自动传入 `cls`，而不是实例 `self`。**
适合提供另一种构造方式，或者读取子类可以覆盖的配置。

```python
class User:
  """表示一个用户。

  Attributes:
      name: 用户名。
  """

  def __init__(self, name: str):
    """用用户名创建用户。

    Args:
        name: 用户名。
    """
    self.name = name


  @classmethod
  def from_email(cls, email: str):
    """用邮箱的 @ 前缀创建用户，不校验邮箱格式。

    Args:
        email: 邮箱地址。

    Returns:
        当前类的实例。
    """
    return cls(email.split("@", 1)[0])


class Admin(User):
  """表示继承了用户构造方式的管理员。"""


assert User.from_email("alice@example.com").name == "alice"
assert isinstance(Admin.from_email("alice@example.com"), Admin)
```

> 不需要先创建实例。通过实例调用时，`cls` 也是实例所属的类。
> 用 `cls(...)` 而不是写死 `User(...)`，子类调用时才能创建自己的实例。

## abstractmethod

**用 `@abstractmethod` 标记抽象方法，让具体子类提供实现。**
这个装饰器来自 `abc` 模块，通常配合继承 `ABC` 使用。

```python
from abc import ABC, abstractmethod


class Shape(ABC):
  """表示能计算面积的图形。"""

  @abstractmethod
  def area(self) -> int:
    """返回图形的面积。"""


class UnitSquare(Shape):
  """表示边长为 1 的正方形。"""

  def area(self) -> int:
    """返回单位正方形的面积。"""
    return 1


assert UnitSquare().area() == 1
```

> 仍有抽象方法的类不能实例化。与 `@classmethod` 或 `@property` 叠加时，
> 把 `@abstractmethod` 写在最靠近 `def` 的位置。
