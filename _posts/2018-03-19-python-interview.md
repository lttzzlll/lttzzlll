### 简述Python中is和==的区别

is 用来比对两个引用是否指向同一个内存对象
== 用来比较两个对象的内容(值)是否一致

解释下面代码的输出结果并简述原理

```Python
a = 1
b = 1
print(a == b)
print(a is b)
# True
# True

# print(a is b) 输出True类似于C# Java中String对象常量池的预分配，Python会自动创建一些常用的小数字，把他们预先存储起来，用的时候直接加一个引用就ok了。这里a和b同时指向1，所以 a is b == True

class A(object):
    def __eq__(self, other):
        return True

a = A()
b = A()
print(a is b)
print(a == b)
# False
# True # 对象自定义[重写]了 __eq__ 方法 返回 True

class A(object):
    def __eq__(self, other):
        return False
a = A()
b = A()
print(a is b)
print(a == b)
# False
# False # 对象自定义[重写]了 __eq__ 方法 返回 False
```

关于is 和 id 更准确地描述:

每个变量都有标识、类型和值。对象一旦创建，它的标识绝不会变；你可以把标识理解为对象在内存中的地址。is 运算符比较两个对象的标识；id() 函数返回对象标识的整数表示。

对象 ID 的真正意义在不同的实现中有所不同。在 CPython 中，id() 返回对象的内存地址，但是在其他 Python 解释器中可能是别的值。关键是，ID 一定是唯一的数值标注，而且在对象的生命周期中绝不会变。

其实，编程中很少使用 id() 函数。标识最常使用 is 运算符检查，而不是直接比较 ID。

== 运算符比较两个对象的值（对象中保存的数据），而 is 比较对象的
标识。is 运算符比 == 速度快，因为它不能重载，所以 Python 不用寻找并调用特殊方法，而是直接比较两个整数 ID。而 a == b 是语法糖，等同于a.__eq__(b)。继承自 object 的 __eq__ 方法比较两个对象的 ID，结果与 is 一样。但是多数内置类型使用更有意义的方式覆盖了 __eq__方法，会考虑对象属性的值。

### 如何理解元组的相对不可变形性

元组的不可变性其实是指 tuple 数据结构的物理内容（即保存的引用）不可变，与引用的对象无关。

元组的值会随着引用的可变对象的变化而变。元组中不可变的是元素的标识。

```Python
a = 1
b = "123"
class C:
    a = 1
c = C()
d = (a, b, c)

# d[0] = babala ERR
# d[1] = babala ERR
# d[2] = babala ERR
```

### 使用装饰器来统计一个函数[非异步与协程]的运行时间

```Python
from functools import wraps
import time

def timeit(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        res = func(*args, **kwargs)
        end = time.time()
        print('{} cost {:.2f}s'.format(func.__name__, end - start))
        return res
    return wrapper

@timeit
def add(a, b):
    time.sleep(1)
    return a + b

print(add.__name__) # 如果有@wraps(func) add 没有的话 wrapper
print(add(1, 2))

# add cost 1.00s
# 3
```

考察对装饰器的理解, 类似C# Java中的注解，由于Python中一切都是对象，所以比静态语言灵活一些。
装饰器在什么时候执行：模块加载的时候,在装饰的对象(函数)定义之后立即运行。

```Python
# demo.py
registry = []

def register(func):
    print('running register(%s)' % func)
    registry.append(func)
    return func


@register
def f1():
    print('running f1()')

@register
def f2():
    print('running f2()')


def f3():
    print('running f3()')


def main():
    print('running main()')
    print('registry ->', registry)
    f1()
    f2()
    f3()


if __name__ == '__main__':
    main()

# running register(<function f1 at 0x000000D4F4B07C80>)
# running register(<function f2 at 0x000000D4F4B07D08>)
# running main()
# registry -> [<function f1 at 0x000000D4F4B07C80>, <function f2 at # 0x000000D4F4B07D08>]
# running f1()
# running f2()
```

### 如何理解Python中的函数闭包

闭包指延伸了作用域的函数，其中包含函数定义体中引用、但是不在定义体中定义的非全局变量。函数是不是匿名的没有关系，关键是它能访问定义体之外定义的非全局变量。
闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。


数字、字符串、元组等不可变类型来说，只能读取，不能更新

```Python
func.__code__.co_freevars
func.__closure__[0].cell_contents
```

### Python 函数闭包的应用

```Python
def make_averager():
    count = 0
    total = 0
    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count
    return averager

```

### Python 装饰器的功能

装饰器的典型行为：把被装饰的函数替换成新函数，二者接受相同的参数，而且（通常）返回被装饰的函数本该返回的值，同时还会做些额外操作。


### Python中如何实现单例模式

1. 使用 __new__

```Python
class Singleton(object):
    def __new__(cls, *args, **kw):
        if not hasattr(cls, '_instance'): #Method resultion order
            orig = super(Singleton, cls) # MRO [MyClass, Singleton, object, type]
            cls._instance = orig.__new__(cls, *args, **kw)
        return cls._instance

class MyClass(Singleton):
    a = 1

a = MyClass()
b = MyClass()

print(a is b)
# True
```

2. 使用装饰器

```Python
def singleton(cls, *args, **kw):
    instances = {}
    def getinstance():
        if cls not in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]
    return getinstance

@singleton
class MyClass:
    a = 1

MyClass = singleton(MyClass)

a = MyClass()
b = MyClass()

print(a is b)
# True
```

3. import

Python的模块是天然的单例模式

```Python
# singleton.py
class Singleton(object):
    a = 1

singletion = Singleton()
```

```Python
# a.py
from singleton import singletion
```

```Python
# b.py
from singleton import singletion
```

```Python
# c.py
from a import singletion as aSingleton
from b import singletion as bSingleton

print(aSingleton is bSingleton)
# True
```

考察对装饰器及类和实例及模块的理解

### Python 中深拷贝与浅拷贝

下面代码的输出结果是什么

```Python
import copy

a = [1, 2, 3, 4, ['a', 'b']]
b = a 
c = copy.copy(a)
d = copy.deepcopy(a)

a.append(5)
a[4].append('c')

print('a = ', a)
print('b = ', b)
print('c = ', c)
print('d = ', d)

# a =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
# b =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
# c =  [1, 2, 3, 4, ['a', 'b', 'c']]
# d =  [1, 2, 3, 4, ['a', 'b']]
```

考察对Python中深拷贝与浅拷贝的理解

```Python
>>> l1 = [3, [55, 44], (7, 8, 9)]
>>> l2 = list(l1) # 1
>>> l2
[3, [55, 44], (7, 8, 9)]
>>> l2 == l1 # 2
True
>>> l2 is l1 # 3
False
```

1 list(l1) 创建 l1 的副本。

2 副本与源列表相等。

3 但是二者指代不同的对象。对列表和其他可变序列来说，还能使用简洁的 l2 = l1[:] 语句创建副本。

然而，构造方法或 [:] 做的是浅复制（即复制了最外层容器，副本中的元素是源容器中元素的引用）。如果所有元素都是不可变的，那么这样没有问题，还能节省内存。但是，如果有可变的元素，可能就会导致意想不到的问题。

```Python
l1 = [3, [66, 55, 44], (7, 8, 9)]
l2 = list(l1) # 1
l1.append(100) # 2
l1[1].remove(55) # 3
print('l1:', l1)
print('l2:', l2)
l2[1] += [33, 22] # 4
l2[2] += (10, 11) # 5
print('l1:', l1)
print('l2:', l2)

#l1: [3, [66, 44], (7, 8, 9), 100]
#l2: [3, [66, 44], (7, 8, 9)]
#l1: [3, [66, 44, 33, 22], (7, 8, 9), 100]
#l2: [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
```

shallow copy and deepcopy

```Python
class Bus:

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)

>>> import copy
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)
>>> bus3 = copy.deepcopy(bus1)
>>> id(bus1), id(bus2), id(bus3)
(4301498296, 4301499416, 4301499752) 1
>>> bus1.drop('Bill')
>>> bus2.passengers
['Alice', 'Claire', 'David'] 2
>>> id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
(4302658568, 4302658568, 4302657800) 3
>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David'] 4
```

1 使用 copy 和 deepcopy，创建 3 个不同的 Bus 实例。

2 bus1 中的 'Bill' 下车后，bus2 中也没有他了。

3 审查 passengers 属性后发现，bus1 和 bus2 共享同一个列表对象，因为 bus2 是 bus1 的浅复制副本。

4 bus3 是 bus1 的深复制副本，因此它的 passengers 属性指代另一个列表。

一般来说，深复制不是件简单的事。如果对象有循环引用，那么这个朴素的算法会进入无限循环。deepcopy 函数会记住已经复制的对象，因此能优雅地处理循环引用。

```Python
>>> a = [10, 20]
>>> b = [a, 30]
>>> a.append(b)
>>> a
[10, 20, [[...], 30]]
>>> from copy import deepcopy
>>> c = deepcopy(a)
>>> c
[10, 20, [[...], 30]]
```

### 简述Python中的垃圾回收机制

Python GC主要使用引用计数（reference counting）来跟踪和回收垃圾。在引用计数的基础上，通过“标记-清除”（mark and sweep）解决容器对象可能产生的循环引用问题，通过“分代回收”（generation collection）以空间换时间的方法提高垃圾回收效率。

1. 引用计数

PyObject是每个对象必有的内容，其中ob_refcnt就是做为引用计数。当一个对象有新的引用时，它的ob_refcnt就会增加，当引用它的对象被删除，它的ob_refcnt就会减少.引用计数为0时，该对象生命就结束了。
优点:简单 实时性
缺点:维护引用计数消耗资源

问题: 对象之间存在循环引用

2. 标记-清除机制

基本思路是先按需分配，等到没有空闲内存的时候从寄存器和程序栈上的引用出发，遍历以对象为节点、以引用为边构成的图，把所有可以访问到的对象打上标记，然后清扫一遍内存空间，把所有没标记的对象释放。

3. 分代技术

分代回收的整体思想是：将系统中的所有内存块根据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集频率随着“代”的存活时间的增大而减小，存活时间通常利用经过几次垃圾回收来度量。
Python默认定义了三代对象集合，索引数越大，对象存活时间越长。
举例： 当某些内存块M经过了3次垃圾收集的清洗之后还存活时，我们就将内存块M划到一个集合A中去，而新分配的内存都划分到集合B中去。当垃圾收集开始工作时，大多数情况都只对集合B进行垃圾回收，而对集合A进行垃圾回收要隔相当长一段时间后才进行，这就使得垃圾收集机制需要处理的内存少了，效率自然就提高了。在这个过程中，集合B中的某些内存块由于存活时间长而会被转移到集合A中，当然，集合A中实际上也存在一些垃圾，这些垃圾的回收会因为这种分代的机制而被延迟。

现代的GC技术一般就这种思路

### del和垃圾回收

del 语句删除名称，而不是对象。del 命令可能会导致对象被当作垃圾回收，但是仅当删除的变量保存的是对象的最后一个引用，或者无法得到对象时。 重新绑定也可能会导致对象的引用数量归零，导致对象被销毁。


在 CPython 中，垃圾回收使用的主要算法是引用计数。实际上，每个对象都会统计有多少引用指向自己。当引用计数归零时，对象立即就被销毁：CPython 会在对象上调用 __del__ 方法（如果定义了），然后释放分配给对象的内存。CPython 2.0 增加了分代垃圾回收算法，用于检测引用循环中涉及的对象组——如果一组对象之间全是相互引用，即使再出色的引用方式也会导致组中的对象不可获取。Python 的其他实现有更复杂的垃圾回收程序，而且不依赖引用计数，这意味着，对象的引用数量为零时可能不会立即调用 __del__ 方法。
