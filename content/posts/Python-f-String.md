---
title: Python f-String
date: 2018-07-09 12:41:09
tags:
 - Python
categories:
 - 技术
---
<!--more-->
> [原文, 知乎 - 刘哈哈](https://zhuanlan.zhihu.com/p/39128162)

### 使用方法

#### 字符串 

```Python
>>> name = 'Python'
>>> age = 18
>>> f"hi, {name}, are you {age}"
'hi, Python, are you 18'
>>> F"hi, {name}, are you {age}"
'hi, Python, are you 18'
```

#### 运算
```Python
>>> f"{ 2 * 3 + 1}"
'7'
```


#### 函数

```Python
>>> def test(input):
...     return input.lower()
...
>>> name = "Python"
>>> f"{test(name)} is handsome."
'python is handsome.'
```

```Python
>>> f"{name.lower()} is handsome."
'python is handsome.'
```

#### 在类中使用

```Python
>>> class Person:
...     def __init__(self,name,age):
...         self.name = name
...         self.age = age
...     def __str__(self):
...         return f"{self.name} is {self.age}"
...     def __repr__(self):
...         return f"{self.name} is {self.age}. HAHA!"
...
>>> Python = Person("Python",18)
>>> f"{Python}"
'Python is 18'
>>> f"{Python!r}"
'Python is 18. HAHA!'
>>> print(Python)
Python is 18
>>> Python
Python is 18. HAHA!
```

#### 多行

```python
>>> name = 'Python'
>>> age = 18
>>> message = {
...     f'hi {name}.'
...     f'you are {age}.'
... }
>>>
>>> message
{'hi Python.you are 18.'}
```

这里需要注意，每行都要加上 f 前缀，否则格式化会不起作用：

```python
>>> message = {
...     f'hi {name}.'
...     'you are learning {status}.'
... }
>>> message
{'hi Python.you are learning {status}.'}
```

#### 速度对比

```Python
from timeit import timeit

print(timeit("""name = "Python"
age = 18
'%s is %s.' % (name, age)""", number = 10000))

print(timeit("""name = "Python"
age = 18
'{} is {}.'.format(name, age)""", number = 10000))

print(timeit("""name = "Python"
age = 18
f'{name} is {age}.'""", number = 10000))
```

结果

```Bash
$ python fstring.py
0.002238000015495345
0.004068000009283423
0.0015349999885074794
```

#### 注意事项

可以在字符串中使用各种引号，只要保证和外部的引号不重复即可。

以下使用方式都是没问题的：

```python
>>> f"{'Python'}"
'Python'
>>> f'{"Python"}'
'Python'
>>> f"""Python"""
'Python'
>>> f'''Python'''
'Python'
```
那如果字符串内部的引号和外部的引号相同时呢？那就需要 \ 进行转义：

```python
>>> f"You are very \"handsome\""
'You are very "handsome"'
```

##### 括号的处理

```python
>>> f"{{74}}"
'{74}'

>>> f"{{{74}}}"
'{74}'
```

可以看出，使用三个括号包裹效果一样。


##### 反斜杠

```python
>>> f"You are very \"handsome\""
'You are very "handsome"'
>>> f"{You are very \"handsome\"}"
  File "<stdin>", line 1
SyntaxError: f-string expression part cannot include a backslash
```
你可以先在变量里处理好待转义的字符，然后在表达式中引用变量：

```python
>>> name = '"handsome"'
>>> f'{name}'
'"handsome"'
```

##### 注释符号

```python
>>> f"Python is handsome # really"
'Python is handsome # really'
>>> f"Python is handsome {#really}"
  File "<stdin>", line 1
SyntaxError: f-string expression part cannot include '#'
```


