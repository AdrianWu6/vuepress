---
title: python基础知识
date: 2025-08-07
categories:
- Python
tags:
- Python
---

## 规范

文件名全小写

## 标识符

Python中标识符的命名规则如下。

1 区分大小写：Myname与myname是两个不同的标识符。

2 首字符可以是下画线（_）或字母，但不能是数字。   _name

3 除首字符外的其他字符必须是下画线、字母和数字。

4 **关键字**不能作为标识符。

5 不要使用Python的内置函数作为自己的标识符。

## 关键字

![image-20250608104308618](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608104308618.png)

## 变量

python是弱类型语言，直接申明即可，甚至于可以其他值也进行推导。

## 语句

1.语句结束时不要带分号，可以但不符合规范

2.可以链式赋值多个变量 a=b=c=10

## 注释

和shell语言差不多注释 开头#即可

与此同时 在第一行第二行 # coding=utf-8设置文件编码格式

## 模块

python一个文件就是一个模块，另一个模块调用需要先导入模块

### 三种方式

1.导入所有元素

![image-20250608110135200](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608110135200.png)

2.从某个模块导入某个个元素

![image-20250608110323265](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608110323265.png)

3.从某个模块导入某个个元素并且命名别称（适合与本模块变量相左的问题）

![image-20250608110705988](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608110705988.png)

## 数据类型

在Python中所有的数据类型都是类，每个数据值都是类的“实例”。

在Python中有6种主要的内置数据类型：**数字、字符串、列表、元组、集合和字典**。列表、元组、集合和字典可以容纳多项数据，在本书 中把它们统称为容器类型的数据。

```
print(type(hellp.x))
print(type(y))
print(type(z1))
```

可以用type()来判断数据类型

### 数字类型

Python中的**数字类型**有4种：**整数类型、浮点类型、复数类型（数学概念虚部与实部）和布尔类型**。需要注意的是，布尔类型也是数字类型，它事实上是整数类型 的一种。

隐式转换与Java类似，不赘述

显示转换



### 布尔类型

任何类型数据都可以通过bool()函数转换为布尔值，空值位false，其他为true

![image-20250608113622407](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608113622407.png)

### 容器类型

Python内置的数据类型如序列（列表、元组等）、集合和字典等可 以容纳多项数据，我们称它们为容器类型的数据。

序列：可迭代、元素有序的容器类型，包含字符串、列表、元组、字节序列。

#### 序列通用操作

* 索引操作

正向索引

| 元素 | 1    | 2    | 3    | 4    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 索引 | 0    | 1    | 2    | 3    | 4    |

反向索引

| 元素 | 1    | 2    | 3    | 4    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 索引 | -5   | -4   | -3   | -2   | -1   |

```python
# 最后一个元素
print(a[-1])
# 第一个元素
print(a[0])
# 最后一个元素
print(max(a))
# 第一个元素
print(min(a))
# 列表长度
print(len(a))
```

* 加乘操作

```python
# 两个字符串拼接
print(a+b)
# 同个字符生成3次
print(a * 3)
```

* 切片操作

语法[start: end : step]。start是开始索引，end是结束索引，step是步长。左闭右开。

![image-20250609162112687](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250609162112687.png)

![image-20250609162144745](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250609162144745.png)

#### 列表

可变序列 有点像arraylist

* **创建列表**有两种方法。

1、list（iterable）函数：参数iterable是可迭代对象（字符串、列表、 元组、集合和字典等）。

2、[元素1，元素2，元素3，⋯]：指定具体的列表元素，元素之间以 逗号分隔，列表元素需要使用中括号括起来

```python
a = [1,2,3,4,5]
b = list('hello')

print(type(a))
print(type(b))
```

* **追加元素**

追加单个元素：list.append(ele)

追加多个元素：list1+list2 list.extend(list)

```python
a = [1,2,3,4,5]
b = list('hello')
a.append(7)
c = a+b;

print(a)
print(c)
a.extend(b)
print(a)
```

[1, 2, 3, 4, 5, 7]
[1, 2, 3, 4, 5, 7, 'h', 'e', 'l', 'l', 'o']
[1, 2, 3, 4, 5, 7, 'h', 'e', 'l', 'l', 'o']

* 插入元素

```python
a = [1,2,3,4,5]
a.insert(2,10)

[1, 2, 10, 3, 4, 5]
```

* 替换元素

```python
a = [1,2,3,4,5]
a[0] = 10

[10, 2, 3, 4, 5]
```

* 删除元素

可使用列表的list.remove（x）方法，如果 找到**匹配**的元素x，则删除该元素，如果找到**多个**匹配的元素，则只删 除第一个匹配的元素。

```python
a = [1,2,3,4,5]
a.remove(1)

[2,3,4,5]
```

#### 元组

不可变序列

创建元组时有两种方法。

1、tuple（iterable）函数：参数iterable是可迭代对象（字符串、列表 、元组、集合和字典等）。

2、小括号包起来(12,12,14,16)

* 元组拆包

```python
a=(1,2,3,4,5)
w,e,r,t,y = a;
print(w)

1
```

#### 集合

集合（set）是一种可迭代的、无序的、不能包含重复元素的容器类 型的数据。

我们可以通过以下两种方式创建集合。

1、set（iterable）函数：参数iterable是可迭代对象（字符串、列表、 元组、集合和字典等）。

2 、{元素1，元素2，元素3，⋯}：指定具体的集合元素，元素之间以 逗号分隔。对于集合元素，需要使用大括号括起来。

* add（elem）：添加元素
  如果元素已经存在，则不能添加，不会 抛出错误。

  ```python
  a={1,2,4}
  a.add(1)
  a.add(2)
  a.add(3)
  print(a)
  
  {1, 2, 3, 4}
  ```

* remove（elem）：删除元素。
  如果元素不存在，则抛出错误

* clear（）：清除集合。

#### 字典

字典（dict）是可迭代的、通过键（key）来访问元素的可变的容器 类型的数据。类似于Map。

我们可以通过以下两种方法创建字典。

1、 dict（）函数。

2 、{key1：value1，key2：value2，...，key_n：value_n}：指定具体 的字典键值对，键值对之间以逗号分隔，最后用大括号括起来。

![image-20250610151130560](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250610151130560.png)

* 取数

map[key]

* 删除

map.pop(key)

* 增加/修改

map[key] = value

* 遍历输出

```python

b={1:'202',2:'303',3:'404'}

# 输出键
for item in b.keys():
    print("键值是:"+str(item))
# 输出值
for item in b.values():
    print("值是:"+item)
# 输出元组
for item in b.items():
    print(item)

#输出元组中的值
for key,value in b.items():
    print("key:"+str(key) +" value:"+value)
    
    
键值是:1
键值是:2
键值是:3
值是:202
值是:303
值是:404
(1, '202')
(2, '303')
(3, '404')
key:1 value:202
key:2 value:303
key:3 value:404    
```

#### 成员测试

适用于所有容器

```python
b={1:'202',2:'303',3:'404'}

print(1 in b)
print(1 not in b)
True
False
```



## 运算符

其他都与Java差不多

`/`有区别 例如10/3 Python输出结果是3.3333 Java是3，若要取整Python需要 10//3

还有幂运算 2^5 可以 2**5

### 比较运算符

与Java一致

**数值类型**一定要能够隐式转换才行，否则会报错。比如 1>'aa' 就是不可以的

**字符类型**相比要逐一比较Unicode的大小，直到有结果结束

### 逻辑运算符

![image-20250608135532259](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608135532259.png)

Python的逻辑运算符是短路的

### 位运算

![image-20250608135726486](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608135726486.png)

### 赋值运算

![image-20250608135900909](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608135900909.png)

### 运算符优先级

![image-20250608140014852](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250608140014852.png)

## 程序控制语句

### 分支语句

与Java,C++不同 Python没有switch语句

#### if结构

if 条件:

​		语句组

注意英文冒号和缩进

#### if-else 结构

if 条件:

​		语句组1

else:

​		语句组2

#### if-elif-else

if 条件1:

​	语句组1

elif 条件2:

​	语句组2

....

else:

​	语句组n+1

### 循环语句

Python只支持While和for循环，else在循环结束时输出

#### While

While 循环条件:

​		循环体语句组

else:

​		结束语句组

```python
i=0

while i<10:
    print(i)
    i=i+1
else:
    print('循环结束')
```

#### for

for 变量 in 可迭代对象:

​	循环体语句组

else:

​	结束语句组

```python
while i<10:
    print(i)
    i=i+1
else:
    print('循环结束')


for item in 'heloo':
    print(item)
else:
    print('for循环结束了')

a = [1,2,3,4,5,6]
for it in a:
    print(it)
else:
    print('for循环结束了')
```

## 函数

可以反复执行的代码块，称之为函数。函数分为：

1.**模块**中的函数叫**函数**

2.**函数**中的函数叫**嵌套函数**

3.**类**里的函数叫**方法**

### 格式

def 函数名(形参列表):

​		函数体

​		return 返回值

没数据可省略返回值

### 形参

python可以直接指定形参的名字，颠倒位置

```python
def add(a,b):
    return a+b

print(add(12,23))

print(add(b=12,a=11))
```

### 不可重载

Python不能做方法重载，但是可以设置默认值

```python
def coffee(name = '卡布奇诺'):
    print(name)

coffee()
coffee('拿铁')

卡布奇诺   
拿铁

```

### 可变参数

基于元组的可变参数 *ele

```python
def sum(*ele):
    total = 0
    for e in ele:
        total+=e
    return total

print(sum(10,20,30))
print(sum(10,20))
```

基于字典的可变参数**ele

```python
def println1(**ele):
    for k,v in ele.items():
        print(k,v)

println1(name="张三",age=12)
```

### 函数类型

每个函数都有其类型，就是函数类型

```python
def switch(char):
    if char == '+':
        return add
    else:
        return minus

def add(a,b):
    return a+b

def minus(a,b):
    return a-b

# 这里f1返回的是函数add
f1 = switch('+')
print(f1(12,22))

```

### filter()与map()函数

filter函数用来过滤容器并生成新的容器

形式 filter(function,iterable)

```python
def bigThan(x):
    return x >5

a = [1,2,3,4,5,6,7,8]

b= filter(bigThan,a)
# 需要用list转换回来 否则为filter类型
print(list(b))


[6, 7, 8]

```

Map函数用来加工生成新的容器

形式map(function,iterable)

```python
def sum(x):
    return x + 5

a = [1,2,3,4,5,6,7,8]

b= map(sum,a)
print(list(b))

[6, 7, 8, 9, 10, 11, 12, 13]
```

### lambda函数

匿名函数

函数形式: lambda 参数列表: lambda体

```python
a = [1,2,3,4,5,6,7,8]

b= map(lambda x:x+5,a)
print(list(b))
```

![image-20250611153112346](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250611153112346.png)

## 类

### 定义类

如果定义的本身就是父类，如果是父类**可省略**

class 类名(父类):

​		函数体

![image-20250611163747657](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250611163747657.png)

### 构造函数

类中的`__init__（）`方法是一个非常特殊的方法，用来创建和初始 化实例变量，这种方法就是“构造方法”。在定义`__init__（）`方法时， 它的第1个参数应该是self，之后的参数用来初始化实例变量。调用构造 方法时不需要传入self参数。

```python
# self表示当前对象属于实例
# __init__ 构造函数
class Dog():
    def __init__(self,name,age):
        self.name = name
        self.age = age

dog = Dog('张三',12)
print(dog.name)
print(dog.age)

```

### 实例变量、类变量、类方法、实例方法

self表示当前对象、cls代表当前类

```python
# __init__ 构造函数
class Dog():
    #类对象 类可直接访问
    rate = 12.0

    def __init__(self,name,age):
        self.name = name
        self.age = age
    def run(self):
        print('狗在跑')

    def speak(self,language):
        print(language)

    #类方法
    @classmethod
    def add(cls,a,b):
         print(a+b)


dog = Dog('张三',12)
print(dog.name)
print(dog.age)
dog.run()
dog.add(12,23)
print(Dog.rate)
dog.speak('汪汪汪')

```

### 封装

封装性是面向对象重要的基本特性之一。封装隐藏了对象的内部细节，只保留有限的对外接口，外部调用者不用关心对象的内部细节，使得操作对象变得简单。

* 私有变量

`__variable` 变量前面加两个下划线

类的内部可以访问私有变量

类的外部不可以访问

* 私有方法

`def __functionName():`

方法名前加两个下划线

### 使用属性

```python
class Dog():
    def __init__(self,name,address):
        #创建和私有化变量 name、address
        self.__name = name
        self.__addres = address

    def getName(self):
        return self.__name

    def setName(self,name):
        self.__name = name

dog = Dog('旺财',"上海东路")
print(dog.getName())

dog.setName('小花')
print(dog.getName())
```

#### 特殊方式

```python
class Dog():
    def __init__(self,name,address):
        self.__name = name
        self.__addres = address

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self,name):
        self.__name = name

dog = Dog('旺财',"上海东路")
print(dog.name)

dog.name = '小花'
print(dog.name)
```

@property 定义get方法

@pro.setter 定义set方法

方法名=字段名

### 继承

类的括号里写父类类型，和其他语言基本类似

```python
class animal:
    def __init__(self,name):
        self.__name = name

    def setName(self,name):
        __name = name


    def getName(self):
        return self.__name;
    def run(self):
        print("动物跑")


class dog(animal):
    def __init__(self,name,age):
        super().__init__(name)
        self.__age = age

dog = dog('狗狗',15);
print(dog.getName())
dog.run()
```

#### 多继承

Python支持多继承，当出现同名字段或者方法时，以最左面为准

```python
class obj:
    def run(self):
        print("物质运动")

class animal:
    def __init__(self,name):
        self.__name = name

    def setName(self,name):
        __name = name


    def getName(self):
        return self.__name;
    def run(self):
        print("动物跑")


class dog(obj,animal):
    def __init__(self,name,age):
        super().__init__(name)
        self.__age = age

dog = dog('狗狗',15);
print(dog.getName())
dog.run()

狗狗
物质运动
```

#### 方法重写

子类重写父类方法

```python
class obj:
    def run(self):
        print("物质运动")

class animal:
    def __init__(self,name):
        self.__name = name

    def setName(self,name):
        __name = name


    def getName(self):
        return self.__name;
    def run(self):
        print("动物跑")


class dog(animal):
    def __init__(self,name,age):
        super().__init__(name)
        self.__age = age
    def run(self):
        print("狗狗跑")


dog = dog('狗狗',15);
print(dog.getName())
dog.run()


狗狗
狗狗跑
```

#### 多态

```python
class Animal:
    def speak(self):
        print("说话")

class Dog(Animal):
    def speak(self):
        print("汪汪汪")

class Cat(Animal):
    def speak(self):
        print("喵喵喵")

ob = Dog()
ob.speak()

ob1 = Cat()
ob1.speak()
```

## 异常处理

和其他语言异常处理差不多 try-except-finally

```python
a = input()
try:
    res = 10000/int(a)
    print(res)
except:
    print('不能为0')
finally:
    print('finally结束')
```

**except [] 后面可以指定异常类型，什么也不写所有异常全部拦截，写几个拦截几个**

```python
a = input()
try:
    res = 10000/int(a)
    print(res)
except (ZeroDivisionError,ValueError) as e:
    print(e)
finally:
    print('啊啊啊啊')
```

## 常用内置模块

### 数学模块

![image-20250701164448896](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250701164448896.png)

```python
import math

print(math.ceil(2.3))
```

### 时间模块

时间模块包含几个类

```python
datetime:包含时间和日期 年、月、日、小时、分钟、秒
date:只包含时间    年、月、日
time:只包含时间	  时、分、秒
timedelta:计算时间跨度    
tzinfo:时区信息
```

```python
import datetime

print(datetime.datetime.now())
print(datetime.datetime(2022,12,1))
print(datetime.datetime.today())
print(datetime.date(12, 1, 1))

# 格式化
print(datetime.datetime.now().strftime('%Y-%m-%d'))

# 时间差
t = datetime.timedelta(5)
t += datetime.datetime.now()
print(t)
```

strftime参数

![image-20250702091020472](https://md-img-market.oss-cn-beijing.aliyuncs.com/img/image-20250702091020472.png)

### 正则表达式模块

后续研究



## 文件读写

### 打开文件

语法：

```
open(file,mode='r',encoding=None,errors=None)
```

open（）函数中的参数很多，这里介绍4个常用参数，这些参数的含义如下。

1.file参数 file参数用于表示要打开的文件，可以是字符串或整数。如果file是**字符串**，则表示文件名，文件名既可以是当前目录的相对路径，也可以是绝对路径；如果file是**整数**，则表示一个已经打开的文件。

2.mode参数 mode参数用于设置文件打开模式，用字符串表示，例如rb表示以只 读模式打开二进制文件。用于设置文件打开模式的字符串中的每一个字 符都表示不同的含义，对这些字符的具体说明如下。

t：以文本文件模式打开文件。

b：以二进制文件模式打开文件。

r：以只读模式打开文件。

w：以只写模式打开文件，不能读内容。如果文件不存在，则创建文件；如果文件存在，则覆盖文件的内容。

x：以独占创建模式打开文件，如果文件不存在，则创建并以写入模式打开；如果文件已存在，则引发FileExistsError异常。

a：以追加模式打开文件，不能读内容。如果文件不存在，则创建文件；如果文件存在，则在文件末尾追加。

+：以更新（读写）模式打开文件，必须与r、w或a组合使用，才能设置文件为读写模式。 这些字符可以进行组合，以表示不同类型的文件的打开模式

3.encoding参数 encoding用来指定打开文件时的文件编码，默认是UTF-8编码，主 要用于打开文本文件。 4.errors参数 errors参数用来指定在文本文件发生编码错误时如何处理。推荐erro rs参数的取值为'ignore'，表示在遇到编码错误时忽略该错误，程序会继 续执行，不会退出。

追加写入

```
f = open('a.txt','a+')
f.write('233333')
```

### 关闭文件

读取 -- 写入 -- 关闭流程

```python
try:
    # 可能出现not found异常
    f = open('a.txt','r+')
    print(f.read())
    # 可能出现IO异常
    f.write('66666')
    print(f.read())
except (FileNotFoundError,IOError) as e:
    print(e)
finally:
    if f is not None:
        f.close()
```

还有一种类似于java try-catch-resource的方法 可以自动管理资源

```python
# 可能出现not found异常
with open('a.txt','r+') as f:
    print(f.read())
    # 可能出现IO异常
    f.write('66666')
    print(f.read())
```

### 读文件

```python
read（size=-1）：从文件中读取字符串，size限制读取的字符数，size=-1指对读取的字符数没有限制。
readline（size=-1）：在读取到换行符或文件尾时返回单行字符串。如果已经到文件尾，则返回一个空字符串。size是限制读取的字符数，size=-1表示没有限制。
readlines（）：读取文件数据到一个字符串列表中，每一行数据都是列表的一个元素。
write（s）：将字符串s写入文件中，并返回写入的字符数。
writelines（lines）：向文件中写入一个字符串列表。不添加行分隔符，因此通常为每一行末尾都提供行分隔符。
flush（）：刷新写缓冲区，在文件没有关闭的情况下将数据写入文件中。
```

### 二进制复制文件

```python
with open('a.txt','rb') as f:
    b = f.read()
    cpy_name = 'b.txt'
    with open(cpy_name,'wb') as copy_f:
        copy_f.write(b)
```


