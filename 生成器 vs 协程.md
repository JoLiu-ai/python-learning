# 生成器 vs 协程
## 历史演进与联系
1. **技术基础**：
   - 协程在Python中是基于生成器机制实现的
   - 两者都使用了栈帧暂停和恢复的底层机制
   - 协程可以看作是生成器的高级应用形式

## 共同特点
   - 都支持暂停和恢复执行
   - 都维护了自己的状态
   - 都实现了迭代器协议

## 关键区别
**生成器（Generator）**:
   - 主要用于数据生成和迭代
   - 使用`yield`语句产生值
   - 单向通信：只能从生成器向调用者发送数据

**协程（Coroutine）**:
   - 用于异步编程和并发
   - 使用`async/await`语法
   - 双向通信：可以接收和发送数据
   - 能够在等待I/O操作时释放控制权

1. **设计目的**：
```python
# 生成器：用于数据生成和迭代
def number_generator():
    for i in range(5):
        yield i

# 协程：用于异步并发
async def async_operation():
    await asyncio.sleep(1)
    return "completed"
```

2. **控制流方向**：
```python
# 生成器：单向流动（生成器 -> 调用者）
def gen_data():
    for i in range(3):
        yield f"数据 {i}"

# 协程：双向流动（可以接收外部数据）
async def process_data():
    data = await get_external_data()
    result = await process(data)
    return result
```

3. **数据处理方式**：
```python
# 生成器：主要用于产生数据序列
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 协程：主要用于处理异步操作
async def fetch_urls(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return responses
```

4. **异步特性**：
```python
# 生成器：同步执行
def sync_generator():
    for i in range(3):
        time.sleep(1)  # 阻塞操作
        yield i

# 协程：异步执行
async def async_coroutine():
    for i in range(3):
        await asyncio.sleep(1)  # 非阻塞操作
        yield i
```

5. **上下文切换**：
```python
# 生成器：手动切换
def manual_switch():
    value = yield 1
    print(f"收到: {value}")
    value = yield 2
    print(f"收到: {value}")

# 协程：事件循环管理
async def event_loop_managed():
    while True:
        data = await queue.get()
        await process_data(data)
```

## 实际应用场景对比

1. **生成器最适合的场景**：
```python
# 1. 处理大数据集
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

# 2. 无限序列生成
def infinite_counter(start=0):
    while True:
        yield start
        start += 1
```

2. **协程最适合的场景**：
```python
# 1. 并发网络请求
async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in urls:
            task = asyncio.create_task(session.get(url))
            tasks.append(task)
        return await asyncio.gather(*tasks)

# 2. I/O密集型操作
async def process_files(files):
    tasks = []
    for file in files:
        task = asyncio.create_task(async_process_file(file))
        tasks.append(task)
    await asyncio.gather(*tasks)
```

## 性能特征对比

1. **内存使用**：
```python
# 生成器：惰性求值，内存效率高
def large_dataset():
    for i in range(1_000_000):
        yield i

# 协程：并发执行，可能需要更多内存
async def concurrent_operations():
    tasks = [asyncio.create_task(operation()) 
             for _ in range(1_000)]
    await asyncio.gather(*tasks)
```

2. **CPU使用**：
```python
# 生成器：单线程顺序执行
def cpu_bound_gen():
    for i in range(1000):
        result = heavy_computation()
        yield result

# 协程：I/O等待时释放CPU
async def io_bound_coroutine():
    while True:
        await asyncio.sleep(0)  # 让出CPU
        await process_next_item()
```


## 生成器与协程的演进关系
1. **基础生成器**：
```python
def simple_generator():
    yield 1
    yield 2
    yield 3

# 使用方式
gen = simple_generator()
print(next(gen))  # 1
print(next(gen))  # 2
```

2. **生成器的send方法**：
```python
def generator_with_send():
    value = yield 1
    print(f"接收到: {value}")
    value = yield 2
    print(f"接收到: {value}")

gen = generator_with_send()
print(next(gen))      # 1
print(gen.send(10))   # 打印"接收到: 10"，然后返回2
```

3. **yield from的引入**：
```python
def sub_generator():
    yield 1
    yield 2

def main_generator():
    yield 'a'
    yield from sub_generator()  # 委托给子生成器
    yield 'b'

# 使用方式
for item in main_generator():
    print(item)  # 打印: a, 1, 2, b
```

4. **协程的最终形态**：
```python
async def modern_coroutine():
    result = await some_async_operation()
    return result
```

## 关键词**await**与**yield**对比
1. **yield关键字**:
   - 用于生成器函数
   - 产生一个值并暂停执行
   - 主要用于迭代和数据流

2. **await关键字**:
   - 只能在async函数中使用
   - 等待一个协程或可等待对象完成
   - 用于异步操作的控制流
