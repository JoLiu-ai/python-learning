- [事件循环](#事件循环)
  - [**相关术语**](#相关术语)
    - [1.Future](#1future)
    - [2. Task](#2-task)
    - [3. Future vs Task](#3-future-vs-task)
    - [4. 其他相关概念](#4-其他相关概念)
  - [**事件循环操作**](#事件循环操作)
    - [运行事件循环](#运行事件循环)
    - [停止事件循环](#停止事件循环)
    - [事件循环的时钟](#事件循环的时钟)
  - [实践](#实践)
    - [1. 在异步函数中使用 `asyncio.create_task`（推荐）](#1-在异步函数中使用-asynciocreate_task推荐)
    - [2. 创建多个任务并并发执行](#2-创建多个任务并并发执行)
    - [3. 使用 `loop.create_task` （如果需要在同步代码中使用）](#3-使用-loopcreate_task-如果需要在同步代码中使用)
    - [4. 任务取消和超时处理](#4-任务取消和超时处理)
    - [5. 带名字的任务（Python 3.8+）](#5-带名字的任务python-38)
    - [最佳实践](#最佳实践)

  
# 事件循环

在 Python 的 `asyncio` 模块中，**事件循环（Event Loop）** 是异步编程的核心机制。它负责调度和执行异步任务、回调、网络 I/O 操作以及子进程等。事件循环通过不断轮询 I/O 事件和任务队列，确保程序能够以非阻塞的方式高效运行。

关键注意点：
1. 总是使用 asyncio.run() 来运行主协程
2. 在异步函数内使用 asyncio.create_task()
3. 适当处理任务的取消和异常
4. 使用 asyncio.gather() 
来并发执行多个任务
5. 需要时可以设置超时和取消机制

## **相关术语**

### 1.Future
  - `asyncio.Future` 是一个低级别的对象，表示一个异步操作的结果。它类似于占位符，表示将来某个时刻会产生的结果。Future 对象通常由底层代码使用，应用层代码很少直接操作它们。
  > 最佳实践：  
  > - 在应用层代码中，优先使用 async/await 语法
  > - 只在实现底层异步框架或库时使用 Future
  > - 如果需要直接使用 Future，考虑是否可以用更高级的抽象（如 Task）代替
  
```python
import asyncio

# 模拟一个低级操作，直接使用 Future
async def low_level_operation():
    # 创建 Future 对象
    future = asyncio.Future()
    
    # 模拟异步操作
    async def set_result_later():
        await asyncio.sleep(2)  # 模拟耗时操作
        future.set_result('操作完成')
    
    # 启动异步操作
    asyncio.create_task(set_result_later())
    
    # 等待结果
    result = await future
    return result

# 对比：更常见的高级方式（使用 async/await）
async def high_level_operation():
    await asyncio.sleep(2)
    return '操作完成'

# 运行示例
async def main():
    # 使用低级 Future
    result1 = await low_level_operation()
    print(f"低级操作结果: {result1}")
    
    # 使用高级 async/await
    result2 = await high_level_operation()
    print(f"高级操作结果: {result2}")

asyncio.run(main())
```

1. Future 的实际应用场景
 实现自定义协议：
 ```python
 class MyProtocol(asyncio.Protocol):
    def __init__(self):
        self.future = asyncio.Future()

    def data_received(self, data):
        # 当收到数据时，设置 Future 的结果
        self.future.set_result(data)
```
2.实现回调转异步：
 ```python
 async def callback_to_async(callback_based_func):
    future = asyncio.Future()
    
    def callback(result):
        # 当回调发生时，设置 Future 的结果
        future.set_result(result)
    
    # 使用回调函数
    callback_based_func(callback)
    
    # 等待结果
    return await future
 ```

3. 并发操作的底层控制：
 ```python
async def complex_operation():
    future = asyncio.Future()
    
    # 创建多个任务
    tasks = [
        asyncio.create_task(some_task())
        for _ in range(3)
    ]
    
    # 当所有任务完成时设置 future 结果
    def done_callback(_):
        if all(t.done() for t in tasks):
            results = [t.result() for t in tasks]
            future.set_result(results)
    
    for task in tasks:
        task.add_done_callback(done_callback)
    
    return await future
 ```

### 2. Task
  - `asyncio.Task` 是 `Future` 的子类，专门用于管理协程的执行。Task 将协程包装起来，安排在事件循环中执行，并允许其他协程等待其结果。通过 `asyncio.create_task(coro)` 或 `loop.create_task(coro)` 创建 Task。  
  > 最佳实践：  
    > - 如果你使用 Python 3.7+，推荐使用 asyncio.create_task()  
    > - 如果你需要在特定的事件循环上创建任务，或者使用较早版本的 Python，则使用 loop.create_task()  
    ```python
    import asyncio

    async def say_hello():
        print("Hello")
        await asyncio.sleep(1)
        print("World")

    loop = asyncio.get_event_loop()
    task = loop.create_task(say_hello())
    loop.run_until_complete(task)
    ```

### 3. Future vs Task
- Task 是 Future 的子类，添加了额外的功能
- Task 自动执行协程，而 Future 需要手动设置结果
```python
# Future 使用
future = asyncio.Future()
future.set_result(42)  # 手动设置结果

# Task 使用
async def coro():
    return 42
task = asyncio.create_task(coro())  # 自动执行并设置结果
```

### 4. 其他相关概念

- **协程（Coroutine）：**
  - 使用 `async def` 定义的函数，表示一个异步操作。协程是异步编程的基本构建块，可以使用 `await` 关键字等待其他协程或异步操作的结果。

- **回调（Callback）：**
  - 在某些异步操作完成时，需要执行的函数。可以使用 `loop.call_soon(callback, *args)` 或 `loop.call_later(delay, callback, *args)` 注册回调函数。

- **执行器（Executor）：**
  - 用于在事件循环外执行阻塞的 I/O 操作或 CPU 密集型任务。可以使用 `loop.run_in_executor(executor, func, *args)` 将这些操作委托给线程池或进程池执行。

## **事件循环操作**

### 运行事件循环
  - `asyncio.run(coro)`: 在 Python 3.7 及更高版本中，`asyncio.run()` 是启动协程的高级接口。它会创建一个新的事件循环，运行指定的协程，直到其完成，并自动关闭事件循环。
    ```python
    import asyncio

    async def main():
        print("Hello, World!")

    asyncio.run(main())
    ```
  - `loop.run_until_complete(future)`: 获取当前事件循环，运行直到指定的 Future 对象完成。
    ```python
    import asyncio

    async def main():
        print("Hello, World!")

    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
    ```
  - `loop.run_forever()`: 使事件循环持续运行，直到调用 `loop.stop()` 停止。
    ```python
    import asyncio

    async def main():
        print("Running...")

    loop = asyncio.get_event_loop()
    loop.create_task(main())
    loop.run_forever()
    ```

###  停止事件循环
  - `loop.stop()`: 停止事件循环的运行。
    ```python
    import asyncio

    async def main():
        print("Stopping loop...")
        loop.stop()

    loop = asyncio.get_event_loop()
    loop.create_task(main())
    loop.run_forever()
    ```
  - `loop.is_running()`: 检查事件循环是否正在运行，返回布尔值。
  - `loop.is_closed()`: 检查事件循环是否已关闭，返回布尔值。
  - `loop.close()`: 关闭事件循环。关闭后，不能重新运行或添加新任务。

### 事件循环的时钟

- `loop.time()`: 返回事件循环内部的时钟时间，单位为秒。与系统时间无关，主要用于测量时间间隔。
  ```python
  import asyncio

  loop = asyncio.get_event_loop()
  start = loop.time()
  # 执行一些操作
  end = loop.time()
  print(f"Elapsed time: {end - start} seconds")
  ```

## 实践

### 1. 在异步函数中使用 `asyncio.create_task`（推荐）

```python
import asyncio

async def say_hello():
    await asyncio.sleep(1)
    print("Hello!")
    return "Hello Done"

async def main():
    # 创建任务
    task = asyncio.create_task(say_hello())
    
    # 等待任务完成
    result = await task
    print(f"Result: {result}")

# 运行
asyncio.run(main())
```

### 2. 创建多个任务并并发执行

```python
import asyncio

async def task_func(name, delay):
    await asyncio.sleep(delay)
    return f"Task {name} done"

async def main():
    # 创建多个任务
    tasks = [
        asyncio.create_task(task_func("A", 2)),
        asyncio.create_task(task_func("B", 1)),
        asyncio.create_task(task_func("C", 3))
    ]
    
    # 等待所有任务完成
    results = await asyncio.gather(*tasks)
    print(results)

asyncio.run(main())
```

### 3. 使用 `loop.create_task` （如果需要在同步代码中使用）

```python
import asyncio

async def background_task():
    while True:
        print("Background task running...")
        await asyncio.sleep(1)

def start_background_task():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    
    try:
        # 创建任务
        task = loop.create_task(background_task())
        loop.run_forever()
    finally:
        loop.close()

# 在单独的线程中运行
import threading
thread = threading.Thread(target=start_background_task)
thread.start()
```
>asyncio.create_task() vs loop.create_task()
> - asyncio.create_task() 是 Python 3.7+ 引入的新API
> - loop.create_task() 是较早的API写法
> - asyncio.create_task() 必须在协程函数内部使用
> - loop.create_task() 可以在同步代码中使用，但需要先获取事件循环

### 4. 任务取消和超时处理

```python
import asyncio

async def long_operation():
    try:
        await asyncio.sleep(10)
        return "Operation completed"
    except asyncio.CancelledError:
        print("Operation was cancelled")
        raise

async def main():
    # 创建任务
    task = asyncio.create_task(long_operation())
    
    try:
        # 设置超时
        result = await asyncio.wait_for(task, timeout=5)
    except asyncio.TimeoutError:
        print("Operation timed out")
        task.cancel()  # 取消任务
        try:
            await task  # 等待任务真正结束
        except asyncio.CancelledError:
            pass

asyncio.run(main())
```

### 5. 带名字的任务（Python 3.8+）

```python
import asyncio

async def named_task():
    await asyncio.sleep(1)
    return "Task completed"

async def main():
    # 创建带名字的任务
    task = asyncio.create_task(
        named_task(),
        name="my_special_task"
    )
    
    print(f"Task name: {task.get_name()}")
    result = await task
    print(result)

asyncio.run(main())
```

### 6. 最佳实践

- 在高层代码中，尽量使用 `asyncio.run()` 来管理事件循环，确保事件循环的创建和关闭由框架自动处理。
- 在需要更细粒度控制的场景下，可以直接获取事件循环，并使用其方法来调度任务和管理回调。
- 避免在同一线程中嵌套运行多个事件循环，以防止不可预期的行为。
