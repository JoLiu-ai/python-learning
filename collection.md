![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/cf1f3a5b-8505-4452-8168-8f40ca94618c/bc128153-8ae9-4139-a2c6-d3c08e0a51bc/image.png)

collection
 → 基本接口
● Iterable要支持for、拆包和其他迭代方式；\__iter\__    
● Sized要支持内置函数\__len\__   
● Container要支持in运算符。\__contain\__   
 → 专用接口
● Sequence规范list和str等内置类型的接口；   
● Mapping被dict、collections.defaultdict等实现；   
● Set是set和frozenset两个内置类型的接口。   
