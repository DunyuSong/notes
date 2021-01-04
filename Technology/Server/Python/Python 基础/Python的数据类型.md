# Python的数据类型
## Number 数字型
- 语法: a = 3、a = True
- 包含：int, float, bool, complex

## Sequence 序列型
- 包含：列表、元组、字符串
- 序列中的每一个元素都会分配一个序号，都是**有序**的。
- 都可以使用'索引'来访问。比如[0], [3]
- 都可以使用'切片'来操作。比如:[0:3],[-1:],[0:8:2]
- 都可以使用'in/not in' 来进行判断一个元素是否在列表中。
- 都可以使用'len()、max()、min()'等函数

### str 字符串, 不可变对象
- 语法：'aaa' 或 "aaa"

### tuple 元组，不可变对象
- 语法1：(1,)
- 语法2：1,
- 语法3：tuple(序列)

### list 列表，可变对象
- 语法1：[1, 3,'hello world', true, ['hello', True]]
- 语法2: list(序列)
- 使用：可以对列表执行所有标准的序列操作。
- 赋值：list2[0] = 'new_value'
- 新增：list2.insert(3, 'new_value')
- 删除：del list2[0]
- 删除最后一个: list2.pop()
- 追加: list2.append('new_value')
- 清空：list2.clear()
- 复制：new_list = list2
- 计数：list2.count('l')
- index:方法index在列表中查找指定值第一次出现的索引.list2.index('l')

## set 集合
- 集合里面的元素都是**无序**的。
- 集合里面的元素都是**不重复**的。
- 集合里面的元素都是**不可变对象**类型的。
- 语法1：{1, 3, 2, 4, 6, 1, 3}
- 语法2: set({1, 3, 2, 4, 6, 1, 3}) 
- 使用元祖()创建集合：s = set((1, 3, 6, 2, 8 , 2))
- 使用列表[]创建集合：s = set([1, 3, 3, 2, 5, 7])

## dict 字典
- 字典也是一种集合，但是不同的是，字典是{key:value}形式（item）的，字典中的key不允许重复、无序且也必须是不可变对象类型。
- 语法1：d = {'age':11, 'name':"Jerry"}
- 语法2：d = dict({'age':12, 'name': 'Tom'})

### 操作
- 字典有些操作类似于序列。
- d[k]返回与键k相关联的值。
- d[k] = v将值v关联到键k。
- del d[k]删除键为k的项。
- k in d检查字典d是否包含键为k的项
- d.get(key):获取值
- d.item2():方法items返回一个包含所有字典项的列表，其中每个元素都为(key, value)的形式
- d.get('key'):方法get('key')返回对应key的值，如果没有则不返回。
- d.get('key', 'defaultValue')：返回对应key的值，如果没有则返回defaultValue.