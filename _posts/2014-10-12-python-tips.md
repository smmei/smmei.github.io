---
layout: post
title: "python 笔记"
category: tools
---

读《简明Python教程》的一点笔记

## 控制流

* 如果后面接一个语句块，则前面的条件语句后面要带冒号。比如if语句：

```python
if xxx:

elif xxx:

else xxx:
```

* while 语句有 else 从句，当 while 判断的条件为 False 时执行 else 从语里的内容。如：

```python
while running:
	...
else:
	...
```

* for..in 循环也有 else 从句，它总在for循环结束后执行一次，除非遇到 break 语句。

```python
for i in range(1, 5):
	print i
else:
	print 'The for loop is over'
```

* break 语句用来终止循环，当 for/while 被 break 终止，它们对应的 else 从句也不执行。


## 函数

* global 用于在函数内声明变量是全局的

* 函数名后紧跟着的 '''xxx''' 是函数的文档字符串，用__doc__可以打印出来。

## 模块

* import

* from..import

* 每个模块都有一个名称__name__。如果模块名称是 '__main__'，则说明模块被单独运行，否则就是被其他模块 import 的，此时 __name__ 为不带.py后缀的文件名。

* dir()函数返回一个模块的符号列表。模块里的符号可以用 del 删除。

## 数据结构

三种内建的数据结构：列表、元组、字典

* 列表中的项目包括在方括号中，项目之间以逗号分割。help(list)获得完整的功能描述，比如有 append, sort 等方法。`del`可以删除列靓号中的项目。

* 元组项目在圆括号中，用逗号分割。

* 字典里是key-value联系在一起的一对值，可以这样定义：`d = { key1 : value1, key2 : value2 }`

列表、元组、字典都属于序列，可以用方括号中的一个数来指定序列，这也称为是下标操作。

* shoplist[0]   : 抓取序列中的第一个元素
* shoplist[3]   : 抓取序列中的第四个元素
* shoplist[-1]  : 抓取序列中最后一个元素
* shoplist[-2]  : 抓取序列中倒数第二个元素
* shoplist[1:3] : 返回位置1、位置2的一个序列切片，不包括位置3
* shoplist[:]   : 返回整个序列的拷贝
* shoplist[:-1] : 返回除了最后一个元素外的序列切片
