
An Array of Sequences

### Built-In Sequences

- Container sequences
		 hold items of different types 存放的是所包含对象的引用，对象可以是任何类型  
		list, tuple, and collections.deque
- Flat sequences
		Hold items of one simple type 在自己的内存空间中存储所含内容的值，而不是各自不同的Python对象  
		str, bytes, and array.array
![image](https://github.com/user-attachments/assets/77c7d1aa-7ea5-474f-932b-a1e7dd04a9cb)


- Mutable sequences  
i.e. list, bytearray, array.array, and collections.deque.  
- Immutable sequences  
i.e. For example, tuple, str, and bytes

![image](https://github.com/user-attachments/assets/ffdadf9a-4146-46d9-be58-c88268561242)


任何Python对象在内存中都有一个包含元数据的标头。最简单的Python对象，例如一个float，内存标头中有一个值字段和两个元数据字段。  
● ob_refcnt：对象的引用计数。  
● ob_type：指向对象类型的指针。  
● ob_fval：一个C语言double类型值，存放float的值。  


### List Comprehensions and Generator Expressions
Generator Expressions 生成器表达式使用迭代器协议逐个产出项，而不是构建整个列表提供给其他构造函数。

### Tuples
- immutable lists
   - 元组的不可变性仅针对元组中的引用而言。元组中的引用不可删除、不可替换。倘若引用的是可变对象，改动对象之后，元组的值也会随之变化
![image](https://github.com/user-attachments/assets/e76eb349-10f0-42bf-b511-ba3e75b12848)

```python
>>> a = (10, 'alpha', [1, 2])
>>> b = (10, 'alpha', [1, 2])
>>> a == b
True
>>> b[-1].append(99)
>>> a == b
False
>>> b
(10, 'alpha', [1, 2, 99])
```
> 只有值永不可变的对象才是可哈希的。不可哈希的元组不能作为字典的键，也不能作为集合的元素。
> 

- as records with no field names.

```python
lax_coordinates = (33.9425, -118.408056)
city, year, pop, chg, area = ('Tokyo', 2003, 32_450, 0.66, 8014)
traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ('ESP', 'XDA205856')]
for passport in sorted(traveler_ids):
    print(passport)
    print('%s/%s' % passport)

for country, _ in traveler_ids:
    print(country)
```


