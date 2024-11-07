- 当调用 repr(obj) 或直接在交互式解释器中输入对象名称时，Python 会调用 \__repr\__，如未定义__repr__方法，Vector实例在Python控制台中显示为<Vector object at 0x10e100070>形式。
- 当调用 str(obj) 或使用 print(obj) 输出对象时，Python 会调用 \__str\__。如果对象没有定义 \__str\__ 方法，Python 会回退使用 \__repr\__ 的返回值
-  \__repr \__ is for developers,  \__str \__ is for customers.   
https://stackoverflow.com/questions/1436703/what-is-the-difference-between-str-and-repr

  ```python
  class MyClass:
    def __init__(self, value):
        self.value = value

    def __repr__(self):
        return f"MyClass(value={self.value})"

    def __str__(self):
        return f"MyClass with value: {self.value}"

obj = MyClass(10)
print(str(obj))  # 调用 __str__，输出：MyClass with value: 10
print(obj)       # 同样调用 __str__，输出：MyClass with value: 10
obj              # MyClass(value=10)
```

### !r：
- 用法：在 f-string（格式化字符串字面量）和 str.format() 方法中使用。
- 作用：通过 !r 将对象的 __repr__ 表示插入字符串。
-  使用场景：适用于需要将对象的详细表示嵌入到字符串中。
```python
class MyClass:
    def __repr__(self):
        return "MyClass representation"

obj = MyClass()

# 使用 f-string 格式化
print(f"Object: {obj!r}")  # 输出: Object: MyClass representation

# 使用 str.format() 格式化
print("Object: {!r}".format(obj))  # 输出: Object: MyClass representation

```
