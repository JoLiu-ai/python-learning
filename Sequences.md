
An Array of Sequences

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
