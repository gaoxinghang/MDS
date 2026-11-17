# Python 异步基础速查

`async def` 定义协程函数，`await` 等待结果，`Future` 表示未来结果，
`Task` 包装并调度协程。本文的 Future 和 Task 均指标准库 `asyncio` 中的对象。

[TOC]

## async def

**用于定义协程函数。调用它会得到协程对象，不会立即执行函数体。**
以下讨论不含 `yield` 的普通协程函数。

| 写法 | 含义 |
| --- | --- |
| `def get_value():` | 定义普通函数，调用时执行函数体 |
| `async def get_value():` | 定义协程函数，调用时创建协程对象 |
| `get_value()` | 对协程函数而言，得到的是协程对象，不是最终结果 |
| `await get_value()` | 执行并等待协程，取得其返回值 |
| `asyncio.run(get_value())` | 在普通脚本中运行协程，取得其返回值 |

```python
import asyncio


async def get_value():
  """返回一个示例数值。

  Returns:
      整数 42。
  """
  return 42


assert asyncio.run(get_value()) == 42
```

> `async def` 不会自动创建线程，也不会把函数里的同步操作变成非阻塞操作。
> `asyncio.run()` 用于脚本入口；同一线程已有运行中的事件循环时不能再调用。

## await

**等待一个可等待对象完成，取得结果；等待期间可以让其他任务运行。**

| 写法 | 含义 |
| --- | --- |
| `result = await get_value()` | 等协程完成，将返回值赋给 result |
| `await asyncio.sleep(1)` | 异步等待 1 秒，不阻塞事件循环线程 |
| `result = await future` | 等 Future 完成，取得其结果 |

```python
import asyncio


async def main():
  """先完成异步等待，再返回结果。

  Returns:
      字符串 done。
  """
  await asyncio.sleep(0.01)
  return "done"


assert asyncio.run(main()) == "done"
```

> `await` 通常写在 `async def` 内；它等待的是协程、Task、Future 等可等待对象。
> 普通整数、字符串不能直接 `await`；被等待的操作失败时，异常会在此处抛出。
> 当前协程仍然按顺序执行，不会跳过等待继续执行下一行。
> 连续写两个 `await` 默认是顺序等待，不等于自动并发。
> 如果结果已经就绪，`await` 不一定会让出执行权。

## Future

**代表一个稍后才会确定的结果，可以理解为“结果的占位对象”。**
一方等待它，另一方在操作完成后填入结果或异常。

| 写法 | 含义 |
| --- | --- |
| `future = loop.create_future()` | 创建一个尚未完成的 Future |
| `future.set_result(value)` | 填入结果，将 Future 标记为完成 |
| `future.set_exception(error)` | 填入异常，等待它时会抛出该异常 |
| `result = await future` | 未完成就等待，完成后取得结果或抛出异常 |
| `future.done()` | 检查是否已结束，包括成功、异常和取消 |
| `future.result()` | 立即读取结果，不会等待；尚未完成时会报错 |

```python
import asyncio


async def main():
  """通过回调填入 Future 的结果，再等待并返回它。

  Returns:
      整数 42。
  """
  loop = asyncio.get_running_loop()
  future = loop.create_future()
  loop.call_soon(future.set_result, 42)

  return await future


assert asyncio.run(main()) == 42
```

`call_soon()` 安排回调稍后执行：当前协程等待 Future 时，回调填入 42，等待随之结束。

> Future 本身不负责执行工作；如果没有人完成或取消它，等待可能一直持续。
> 已结束的 Future 不能再次 `set_result()` 或 `set_exception()`。
> 日常代码通常直接等待库返回的协程或 Future，不需要手动创建 Future。
> Task 是一种 Future，会调度协程执行；普通 Future 不会自动运行协程。

## Task

**Task（任务）包装一个协程，交给事件循环安排执行，并记录状态和结果。**
创建 Task 后可以先做其他事，再通过 `await task` 等待结果。

| 写法 | 含义 |
| --- | --- |
| `coroutine = get_value()` | 只创建协程对象，尚未执行函数体 |
| `task = asyncio.create_task(coroutine)` | 包装协程，安排其执行，返回 Task |
| `result = await task` | 等任务完成，取得结果；失败时抛出异常 |
| `task.done()` | 检查任务是否已结束，包括成功、异常和取消 |
| `task.result()` | 立即读取结果，不等待；尚未完成时会报错 |
| `task.cancel()` | 请求取消任务，不代表已立即终止 |

```python
import asyncio


async def get_value():
  """异步等待后返回示例数值。

  Returns:
      整数 42。
  """
  await asyncio.sleep(0.01)
  return 42


async def main():
  """先安排协程执行，再等待任务结果。"""
  task = asyncio.create_task(get_value())
  print("任务已安排，可以先做其他事")

  result = await task
  assert result == 42
  assert task.done()


asyncio.run(main())
```

| 对象 | 核心区别 |
| --- | --- |
| 协程 | 可暂停、恢复的执行过程，创建后需要等待或调度 |
| Future | 未来结果的占位对象，本身不负责执行工作 |
| Task | 一种 Future，负责推进协程执行，并保存结果或异常 |

> `create_task()` 不会创建线程；默认调度下，任务在当前代码让出执行权后运行。
> 它需要当前线程有运行中的事件循环，通常在 `async def` 内调用。
> 普通 Future 可手动填入结果；Task 的结果由协程决定，不能手动 `set_result()`。
> 保留任务引用并等待它，避免任务无人管理或异常无人处理。
