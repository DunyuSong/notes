# Python字符串
## 基本操作
### 格式化字符串
1. 使用format()方法进行格式化
```python
# 每个替换字段都用花括号括起
>>> "hello {}, My name is {}".format("world", "Tom")
'hello world, My name is Tom'
```
2. 使用format()添加索引控制值的顺序
```python
>>> "{3} {0} {2} {1} {3} {0}".format("be", "not", "or", "to")
'to be or not to be'
```
3. 使用命名字段赋值
```python
>>> from math import pi
>>> "{name} is approximately {value:.2f}.".format(value=pi, name="π")
'π is approximately 3.14.'
```
### 操作字符串
- find():查找字符串,''.find('要查找的字符串', [开始索引], [结束索引])。返回找到的字符串开始的索引位置，没找到返回-1。
```python
>>> 'With a moo-moo here, and a moo-moo there'.find('moo', 0, 100)
7
```
- join():合并序列的元素。str1.join(str2)，将str1字符串按照索引拆分开来，然后将str2插入到str1索引的每个元素中间。作用与split相反。
```python
# 比如'abc'.join('***'),那么就是将'***'拆分为三个元素，然后将str1分别插入到两个**之间，结果就是*abc*abc*
>>> 'abc'.join('***')
'*abc*abc*'
>>> str1 = 'helloworld'
>>> str2 = ' '
>>> str2.join(str1)
'h e l l o w o r l d'
```
- replace():将指定子串都替换为另一个字符串，并返回替换后的结果。
```python
>>> 'hello Tom'.replace('Tom', 'jerry')
'hello jerry'
```
- split():用于将字符串拆分为序列，返回一个新字符串。
```python
>>> str1 = '1, 2, 3, 4, 5, 6'
>>> str2 = str1.split(',')
>>> str2
['1', ' 2', ' 3', ' 4', ' 5', ' 6']
>>> str1
'1, 2, 3, 4, 5, 6'
>>> 
```
- strp(' '):根据规则删除符合规则的字符串开头和结尾。
```python
>>> '*** SPAM * for * everyone!!! ***'.strip(' *!')
'SPAM * for * everyone'
>>> 
```