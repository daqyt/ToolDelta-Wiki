## Utils 类

**Utils 提供了一些框架内常用的实用方法。**

#### 获取方法
```python
from tooldelta.utils import Utils
```

以下的所有方法都是 Utils 类中的成员。

### createThread {#createThread}
```python
def createThread(
    func: Callable,
    args: tuple,
    usage: str = "",
    **kwargs
) -> ToolDeltaThread
```

创建一个特殊线程，这个线程受ToolDelta主框架控制和收集。和普通线程类不一样的是，它能使用 stop() 中断运行。
- 参数：  
    - **func** (Callable)： 线程函数  
    - **args** (tuple)： 函数传入的参数组，默认为空  
    - **usage** (str)： 线程的用途，建议填写，默认为空  
    - **kwargs**： 线程函数的关键字参数，默认为空
- 返回：`ToolDeltaThread`  

可以使用 `.stop()` 以中止这个线程的执行（底层原理：在线程内下一条代码指令处引发专有异常）。

:::details `Utils.createThread` 用法示例
```python
def hello_forever(greet, name):
    while 1:
        print(f"{greet}, {name}")

thread = Utils.createThread(
    hello_forever, ("Hello", "ToolDelta"), "一直说 hello"
)
while 1:
    if input() == "quit":
        thread.stop()
```
:::

### simple_fmt {#simple_fmt}
```python
def simple_fmt(kw: dict[str, Any], sub: str) -> str
```

对字符串以简单的方法进行格式化替换。
- 参数：  
    - **kw** (dict[str, Any])： 替换标记与替换内容的映射字典   
    - **sub** (str)： 将被格式化的字符串
- 返回：
格式后的字符串

:::details `Utils.simple_fmt` 用法示例
```python
>>> print(
    Utils.simple_fmt(
        {
            "[arg1]": "Super",
            "[arg2]": 2
        },
        "i am [arg1] and i'm [arg2] yrs old"
    )
)
"i am Super and i'm 2 yrs old"
```
:::


### thread_func {#thread_func}
```python
@Utils.thread_func(usage)
```

作为装饰器使用， 放在任意堵塞函数上，使函数成为线程函数，在被执行的时候自身作为一个新线程启动，不会阻塞代码的继续运行。
- 参数：  
    - **usage** (str)： 此线程用途

:::details `@Utils.thread_func` 用法示例
```python
@Utils.thread_func("一个简单的方法")
def say_hi(name):
    for i in range(5):
        print(f"hi {name}")
        time.sleep(1)

say_hi("Super")
```
:::

### thread_gather {#thread_gather}
```python
def thread_gather(funcs_and_args: list[tuple[Callable, tuple]])
```
并行获取多个阻塞函数的返回值。

- 参数：  
    - **funcs_and_args** (list[tuple[Callable, tuple]])： 阻塞函数以及其参数的列表

- 返回：  
这些函数的返回的列表 （按传入函数的顺序返回其结果）

:::details `Utils.thread_gather` 示例用法
```python
import requests
from tooldelta import Utils, game_utils

(
    baidu_resp,
    bili_resp,
    douyin_resp,
    cmd_is_success
) = Utils.thread_gather(
    [
        requests.get, ("https://www.baidu.com",),
        requests.get, ("https://www.bilibili.com",),
        requests.get, ("https://www.douyin.com",)
        game_utils.isCmdSuccess, ("/testfor @a[m=0]")
    ]
)


```
:::

### try_int {#try_int}
```python
def try_int(n: Any) -> int
```

传入任意值（e.g.字符串）， 尝试转换为整数并返回，否则返回None

### fill_list_index {#fill_list_index}
```python
def fill_list_index(lst: list[VT], default: list[VT]) -> None
```
当列表 lst 长度比 default 小时， 使用 default 中的剩余内容填充 lst 的剩余部分。
- 参数：  
    - **lst** (list)：待填充的列表
    - **default** (list)： 用于填充默认值的列表
- 无返回

:::details `Utils.fill_list_index` 示例用法
```python
>>> list_a = [1, 2]
>>> Utils.fill_list_index(list_a, [1, 2, 3])
>>> list_a
[1, 2, 3]

>>> list_a = [-2, -1]
>>> Utils.fill_list_index(list_a, [1, 2, 3, 4])
>>> list_a
[-2, -1, 3, 4]

>>> list_a = ["我", "的"]
>>> Utils.fill_list_index(list_a, ["你", "好", "世", "界"])
>>> list_a
["我", "的", "世", "界"]
```
:::

### to_player_selector {#to_player_selector}
```python
def to_player_selector(playername: str) -> str
```
将玩家名转化为目标选择器。
传入目标选择器时，不会产生任何效果。
- 参数：   
    - **playername** (str)： 玩家名
- 返回：转化结果

:::details `Utils.to_player_selector` 示例用法
```python
>>> Utils.to_player_selector("SkyblueSuper")
'@a[name="SkyblueSuper"]'

>>> Utils.to_player_selector("2334567")
'@a[name="2334567"]'

>>> Utils.to_player_selector('@a[name="chfwd"]')
'@a[name="chfwd"]`
```
:::

### create_result_cb {#create_result_cb}
```python
def create_result_cb() -> (getter, setter)
```
创建一对回调锁（获取方法和设置方法）。

:::details `Utils.create_result_cb` 示例用法
```python
from uuid import uuid4

callback_handlers = {}

def callback_handler(uuid, data):
    callback_handlers[uuid](data)

def wait_result():
    timeout = 5 # 等待回调的超时时间
    uuid_gen = str(uuid4())
    getter, setter = Utils.create_result_cb()
    callback_handlers[uuid_gen] = setter
    # 
    # 如果超时将返回None; 否则返回回调结果
    result = getter(timeout)
    del callback_handlers[uuid_gen]
    return result

```
:::

---

### TMPJson 类

**Utils.TMPJson：** 提供了加载、卸载、读取和写入JSON文件到**缓存区**的方法的类，让JSON的读取写入更加迅速，磁盘操作更少。

获取方法：
```python
from tooldelta import Utils

TMPJson = Utils.TMPJson
```

#### 内含方法
以下方法都是 TMPJson 类的成员。

```python
def loadPathJson(path, needFileExists=False)
```

从磁盘路径加载json文件到缓存区，用于快速读写。  
在缓存文件已加载的情况下，再使用一次该方法不会有任何作用。  
一般情况下， 强烈建议使用 `read_as_tmp()` 代替此方法
- 参数：  
    - **path** (str)： 文件路径，同时作为这个文件的磁盘路径和虚拟路径  
    - **needFileExists** (bool)： 是否需要文件存在，为False则自动创建一个内容为null的json文件。默认为True  

```python
def unloadPathJson(path)
```

将json从缓冲区卸载并存盘，之后不能对此文件进行读写。
- 参数：  
    - **path** (str)： 文件路径，同时作为这个文件的磁盘路径和虚拟路径
- 返回：  
`bool` 是否卸载成功（没有加载文件时卸载视为不成功）
- 报错：  
`ValueError` json虚拟路径未载入，无法读取

---

```python
def read(path: str)
```

对虚拟路径中的json进行读取。返回一个缓存区经过深拷贝的json对象。
- 参数：
    - **path** (str)： json文件的虚拟路径
- 返回：
`Any` json对象

---

```python
def write(path: str, obj)
```

对虚拟路径中的json进行覆写操作（注意是覆写之前的内容）。
- 参数：  
    - **path** (str)： json文件的虚拟路径  
    - **obj** (Any)： 待写入的任意json对象  
- 报错：  
`ValueError` json虚拟路径未载入，无法写入

---

```python
def read_as_tmp(path: str, needFileExists=False, default=None, timeout=30)
```

对虚拟路径中的json进行读取。如果不存在，则尝试从磁盘打开对应路径的json文件。在限定时间后自动存盘卸载。
- 参数：  
    - **path** (str): json文件的虚拟路径  
    - **needFileExists** (bool): 当从磁盘读取文件时，是否在文件不存在时自动创建一个 json 文件。默认为 False  
    - **default** (Any): 当文件不存在时，向空文件写入的默认内容，也将作为返回值返回
    - **timeout** (int)： 经过多长时间（秒）后自动将打开的json文件从缓存区卸载，默认为 30

```python
def write_as_tmp(path, obj, needFileExists=False, timeout=30)
```

对虚拟路径中的json进行覆写。如果不存在，则尝试从磁盘打开对应路径的json文件。在限定时间后自动存盘卸载。  
- 参数：  
    - **path** (str)： json文件的虚拟路径  
    - **obj** (Any)： 待写入的任意json对象  
    - **needFileExists** (bool)： 当从磁盘读取文件时，是否在文件不存在时自动创建一个内容为null的json文件。默认为False  
    - **timeout** (int)： 经过多长时间（秒）后自动将打开的json文件从缓存区卸载，默认为30

:::details TMPJson 各方法综合使用示例
```python
from tooldelta import Utils
TMPJson = Utils.TMPJson

# 最初的缓冲json文件使用方法 (现在不推荐使用)
file_path = "测试.json"
# 将文件装载到缓存区
TMPJson.loadPathJson(file_path, False)
# 此时可以对 file_path 指向的json文件进行快速读写
content = TMPJson.read(file_path)
# 对文件内容 (content) 做点什么
# if content is None:
#     content = {}
# content["age"] = 18
TMPJson.write(file_path, content)
# 将文件从缓存区卸载, 写入磁盘
TMPJson.unloadPathJson(file_path)


# 最新的缓冲json文件使用方法 (推荐)
# 会自动 loadPathJson 和 unloadPathJson
file_path = "测试.json"
content = TMPJson.read_as_tmp(file_path, False)
# 对文件内容 (content) 做点什么
# if content is None:
#     content = {}
# content["age"] = 18
# 写文件
TMPJson.write_as_tmp(file_path, content)
# 如果你需要立刻更新文件内容到磁盘:
TMPJson.flush(file_path)

```
:::
