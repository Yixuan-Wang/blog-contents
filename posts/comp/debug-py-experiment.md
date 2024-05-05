---
title: 调试 Python 实验代码
date: 2024-02-17
category: comp
tags:
  - python
keywords:
  - productivity
  - academia
---

面向实验代码的 Python 调试技巧。

<!-- more -->

## 断点

除却集成开发环境提供的断点调试界面，Python 提供了一个[内置函数 `breakpoint`](https://docs.python.org/3/library/functions.html#breakpoint) 启动断点。底层此函数将调用 [`sys.breakpointhook`](https://docs.python.org/3/library/sys.html#sys.breakpointhook)，默认指向 `pdb.set_trace`。所以，可以使用更简短的代码启动调试器：

```python
# 不使用 breakpoint 函数
import pdb; pdb.set_trace()

# 之后
breakpoint()
```

如果设置了环境变量 `PYTHONBREAKPOINT`，Python 解释器会调用该变量指向的函数。因此可以自由地更换调试器，比如更换到 [`ipdb`](#调试器)。

```python
os.environ["PYTHONBREAKPOINT"] = "ipdb.set_trace"
```

如果设为空，任何断点将会被忽略。可以利用这个特性，避免在服务器集群的脚本上触发断点：

```python
os.environ["PYTHONBREAKPOINT"] = "ipdb.set_trace" if sys.stdout.isatty() else ""
```

## 内嵌终端

[IPython](https://ipython.org/) 是一个交互式的 Python REPL，提供了代码高亮和补全等功能。可以用以下代码在程序中启动一个**内嵌的交互终端**，其全局命名空间将继承调用处的全局和局部命名空间。

```python
from IPython import embed; embed(using=False)
```

注意，`using` 是一个[没在文档里出现的参数](https://github.com/ipython/ipython/blob/e60c06a57de9f1e53fdd4b3a16c6a16c530e4f2a/IPython/terminal/embed.py#L398)，默认效果是关闭高亮和异步运行时功能，如此可以启用高亮。

特别注意：内嵌交互终端的“全局命名空间”实际上并不是真正的全局命名空间，因此诸如闭包、在终端内定义全局变量、嵌套的生成式都不能使用，即使声明 `global` 也不行。

详细接口可以查看[其文档](https://ipython.readthedocs.io/en/stable/api/generated/IPython.terminal.embed.html)。可以将这个内嵌终端设作默认的断点钩子。

```python
os.environ=["PYTHONBREAKPOINT"] = "IPython.embed"
breakpoint(using=False)
```

内嵌终端有两个特殊的 IPython 魔法指令：`%exit_raise` 将**关闭终端并抛出一个异常，终止外部程序**；而 `%kill_embedded` 则停止此处调用后续的终端唤起（比如在循环里启动内嵌终端的时候），同时加上选项 `-y` 可以关闭确认退出提示 。

## 调试器

调试器提供了检视堆栈帧，列出源码列表等高级功能，这是内嵌终端所不具备的。

一些常用的指令：

```yaml
u(p): 上一层堆栈
d(own): 下一层堆栈
interact: 启动交互终端

s(tep): 单步调试
n(ext): 当前函数内单步调试  # 不会停止在调用的下层函数中
c(ontinue): 继续运行
r(eturn): 继续运行到函数返回
b(reak): 在某一行增加断点

l(ist): 列出源代码
j(ump): 跳到某一行执行

q(uit): 退出
```

[`ipdb`](#IPDB) 提供了一个包含代码高亮和补全，类似 IPython 的调试器，**`interact` 指令将默认启用 IPython 内嵌终端**，同时暴露了一些方便调试的接口。这是单行的用法：

```python
import ipdb; ipdb.set_trace()
```

同时提供自动捕获异常的装饰器和上下文管理器，可以利用它们简化调试。

```python
with ipdb.launch_ipdb_on_exception():
    ...

@ipdb.iex
def foo():
    ...
```

## 调试常量

Python 内部有一个常量 `__debug__`用于标记当前是否为调试状态，**默认为真**。使用 `python -O`(O for Optimize) 或修改 `PYTHONOPTIMIZE` 环境变量可以关闭调试状态，此时 `assert` 语句和依赖 `__debug__` 的代码将不会运行。目前它不会影响断点，但可以利用这个功能修改环境变量，手动关闭断点。

## 断言

`assert` 语句接受第二个参数，会作为 `AssertionError` 的参数被调用，可在调试代码时用于说明异常缘由。

```python
assert (s := input_ids.shape[0]) == BATCH_SIZE, f"Got batch size {s}, {BATCH_SIZE} wanted"
```

对于实验代码来说，断言比较简便（实验中通常不会捕获异常），且自己写的异常缘由会比库提供的底层错误更易读。如果对自己的代码没有把握，就加上断言。

## 日志

~~除了 JavaScript 以外~~使用 `print` 进行调试并不理想。它不具备输出程序元信息的功能，也不方便批量关闭（而日志可以通过修改全局日志级别关闭低级别的信息）。对于在集群上运行（甚至长时间运行）的程序来说，写入文件甚至直接向手机推送通知可能比打印到标准输出更高效且有用😏。

但 Python 内置的 [`logging`](https://docs.python.org/3/library/logging.html) 模块过于复杂，配置需要写大量模版代码，~~甚至 API 的大小写都不符合 Python 的惯例~~？[`loguru`](https://github.com/Delgan/loguru) 是一个不错的替代品。

`loguru` 一般使用包顶级的 `logger` 单例暴露的 API，比如 `logger.info` 和 `logger.warning`。创建新的日志 Sink 只需调用 `logger.add`：

```python
# 文件
# - 带时间戳的
logger.add("file_{time}.log")
# - 自动旋转
logger.add("file.log", rotation="1 week")

# 标准输出流（默认启用）
logger.add(sys.stdout)

# 其他标准 Handler
logger.add(NotificationHandler(...))
```

可以使用以下 `loguru` 提供的代码劫持标准库的 `logging` 处理器：

```python
class InterceptHandler(logging.Handler):
    def emit(self, record: logging.LogRecord) -> None:
        # Get corresponding Loguru level if it exists.
        level: str | int
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno

        # Find caller from where originated the logged message.
        frame, depth = inspect.currentframe(), 0
        while frame and (depth == 0 or frame.f_code.co_filename == logging.__file__):
            frame = frame.f_back
            depth += 1

        logger.opt(depth=depth, exception=record.exc_info).log(level, record.getMessage())

logging.basicConfig(handlers=[InterceptHandler()], level=0, force=True)
```

此外，`loguru` 还提供了一些高级 API 用于复用：

```python
# 添加上下文
# - 绑定风味
binded_logger = logger.bind(ip="192.168.0.1", user="someone")
# - 上下文管理器风味
with logger.contextualize(task=task_id):
  logger.info(...)

# 动态上下文
patched_logger = logger.patch(lambda record: record["extra"].update(utc=datetime.utcnow()))
  
# 懒加载
logger.opt(lazy=True).debug("If sink level <= DEBUG: {x}", x=lambda: expensive_function())
```

## 实时通知

Discord Webhook 可以提供开箱可用的 bot 通知体验。[`notifiers`]([liiight/notifiers: The easy way to send notifications (github.com)](https://github.com/liiight/notifiers)) 没有提供 Discord API，但是提供了 Slack API，可以搭配 Discord Webhook 的 Slack 兼容层使用。以下是创建标准库 `logging` 的处理器的代码：

```python
from notifiers.logging import NotificationHandler

handler = NotificationHandler("slack", defaults={
  "webhook_url": "{}/slack".format(os.environ["DISCORD_WEBHOOK_URL"]),
})
```

标准库可以使用 `Logger.addHandler` 注册这一处理器，而 `loguru` 可以使用下列代码：

```python
logger.add(handler, level="INFO", colorize=True, format=(
    f"```ansi\n"
    "{time:MM-DD HH:mm:ss} <level>{level: <8}</level> {name}:{function}:{line}\n"
    "<level>{message}</level>\n```"
))
```

Discord 支持利用 `ansi` 代码块格式化文本——相比 Embed 块来说代码块视觉负担要小一些，在密密麻麻的日志里更容易看清。记得要手动打开着色选项 。同时，可以把输出等级调高到 `INFO` 防止被 `DEBUG` 和 `TRACE` 消息淹没。
