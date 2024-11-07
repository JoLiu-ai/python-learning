
![image](https://github.com/user-attachments/assets/fd0c8918-6258-43df-8377-1178ec5af8fb)

### collection
#### 基本接口
● Iterable要支持for、拆包和其他迭代方式；\__iter\__    
● Sized要支持内置函数\__len\__   
● Container要支持in运算符。\__contain\__   
#### 专用接口
● Sequence规范list和str等内置类型的接口；   
● Mapping被dict、collections.defaultdict等实现；   
● Set是set和frozenset两个内置类型的接口。   
