## await vs yield

### 基本原则
- **普通函数** - 只能使用 `yield`
- **async 函数** - 只能使用 `await`

- `yield from` 后面可接 **可迭代对象**，也可接 **Future对象** / **协程对象**
- `await` 后面必须要接 **Future对象** / **协程对象**

### 示例代码

```python
# 1. yield 接可迭代对象
def gen_numbers():
    yield from [1, 2, 3]  # 生成器委托
    yield from range(4, 6)

# 2. yield 接协程
def old_style_coroutine():
    result = yield from some_coroutine()
    print(result)

# 3. yield 接 Future
def handle_future():
    result = yield from asyncio.Future()  # Python 3.5之前的写法
    print(result)

# await 接协程
async def await_coroutine():
    result = await some_coroutine()
    print(result)

# await 接 Future
async def await_future():
    future = asyncio.Future()
    result = await future
    print(result)
```

### 可迭代对象、Future对象、协程对象和可等待对象的区别与联系

#### 1. 可迭代对象 (Iterable)
```python
# 可迭代对象示例
my_list = [1, 2, 3]  # 列表是可迭代的
my_dict = {'a': 1, 'b': 2}  # 字典是可迭代的
my_generator = (x for x in range(3))  # 生成器是可迭代的

# 自定义可迭代对象
class MyIterable:
    def __iter__(self):
        return iter([1, 2, 3])
```
- **特点**：实现了 `__iter__` 方法
- **用途**：
  - 可以用于 `for` 循环和 `yield from`
  - **不能**用于 `await`

#### 2. Future对象
```python
# Future对象示例
async def future_demo():
    future = asyncio.Future()
    
    # Future可以被await
    await future
    
    # Future也可以被yield from (旧式写法)
    # yield from future  

    # Future可以设置结果
    future.set_result(42)
```
- **特点**：
  - 是**可等待对象**的一种
  - 代表一个异步操作的最终结果
  - 可以被 `await` 和 `yield from`
  - 是 `Task` 的基类

#### 3. 协程对象 (Coroutine)
```python
# 协程对象示例
async def my_coroutine():
    return 42

# 创建协程对象
coro = my_coroutine()  # 此时返回协程对象
```
- **特点**：
  - 是**可等待对象**的一种
  - 由 `async def` 函数创建
  - 可以被 `await` 和 `yield from`
  - 可以被包装成 `Task`

#### 4. 可等待对象 (Awaitable)
```python
# 自定义可等待对象
class MyAwaitable:
    def __await__(self):
        yield "waiting"
        return "result"

async def use_awaitable():
    # 可等待对象可以被await
    result = await MyAwaitable()
```
- **特点**：
  - 是最顶层的抽象概念
  - 包含 `Future`、协程和 `Task`
  - 必须实现 `__await__` 方法
  - 只能被 `await` 使用

### 使用场景对比

```python
# 1. 可迭代对象：用于for循环和yield from
for item in [1, 2, 3]:  # ✓
    print(item)

yield from [1, 2, 3]  # ✓
await [1, 2, 3]  # ✗ 错误！

# 2. Future/协程/Task：用于await和yield from
async def example():
    await asyncio.sleep(1)  # ✓
    yield from asyncio.sleep(1)  # ✓ (旧式写法)
    
    future = asyncio.Future()
    await future  # ✓
    yield from future  # ✓ (旧式写法)
```

