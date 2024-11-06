### 比较 \__mul\__ 和 \__rmul\__
>  \__mul\__：用于对象在左边的情况，即 a * b 时的 a.\__mul\__(b)。  
> \__rmul\__：用于对象在右边的情况，即 a * b 时如果 a 没有实现合适的 \__mul\__，则调用 b.\__rmul\__(a)。  
#### 典型应用场景
> 当实现一个类的乘法运算时，可以定义 \__mul\__来支持对象在左侧进行乘法运算。如果希望同时支持对象在右侧的乘法（比如 int * MyNumber 的情况），则需要同时实现 \__rmul\__。

举个综合示例

```python
class MyNumber:
    def __init__(self, value):
        self.value = value

    def __mul__(self, other):
        if isinstance(other, (int, float)):
            return MyNumber(self.value * other)
        return NotImplemented

    def __rmul__(self, other):
        # 反向乘法逻辑，通常与 __mul__ 相同
        if isinstance(other, (int, float)):
            return MyNumber(self.value * other)
        return NotImplemented

    def __repr__(self):
        return f"MyNumber({self.value})"

a = MyNumber(10)
print(a * 3)     # 调用 __mul__，输出 MyNumber(30)
print(3 * a)     # 调用 __rmul__，输出 MyNumber(30)
```
这样就可以同时支持 a * 3 和 3 * a 的操作。
