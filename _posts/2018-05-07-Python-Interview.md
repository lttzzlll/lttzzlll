---
layout: post
title: Python Interview
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Python, Backend]
---

### 整理的一些Python面试题

Python语法以及其他基础部分

1.可变与不可变类型； 

###### 不可变(immutable)对象类型
* int
* float
* decimal
* complex
* bool
* string
* tuple
* range
* frozenset
* bytes

###### 可变(mutable)对象类型
* list
* dict
* set
* bytearray
* user-defined classes (unless specifically made immutable)

怎么记忆这些可变或者不可变的对象类型呢？很简单，容器（containers）和自定义（user-defined）类型都是可变的，最典型的就是list，列表类型相当于一个容器，数据可以放进去，可以取出来，所以是可变对象类型。标量类型（scalar type）的对象基本上都是不可变，比如，int，string。但是，有一个例外，tuple是一种不可变对象类型，其实也很好理解，元组声明完成之后，就无法再进行修改，自然是不可变对象类型。

refer [Python对象：可变和不可变对象类型](http://www.blue7wings.com/python/Python-objects-mutable-vs-immutable.html)

2.浅拷贝与深拷贝的实现方式、区别；deepcopy如果你来设计，如何实现；

*** 浅拷贝和深拷贝的不同仅仅是对组合对象来说，所谓的组合对象就是包含了其它对象的对象，如列表，类实例。而对于数字、字符串以及其它“原子”类型，没有拷贝一说，产生的都是原对象的引用。 ***

refer [Python FAQ2：赋值、浅拷贝、深拷贝的区别？](https://songlee24.github.io/2014/08/15/python-FAQ-02/)

deepcopy设计中需要注意的问题主要是对象自己是否存在循环引用以及如何处理这个问题。
如果要把deepcopy设计为一个通用的模块，则必须要处理这个问题，如果是自定义的类型需要深拷贝，则应该根据自定义数据类型的特点实现obj.deepcopy方法。
尽量不要使用Python自带的copy模块，因为deepcopy的代价太大了，费时费力。

详细内容参考这篇文章[为了 1% 情形，牺牲 99% 情形下的性能:蜗牛般的 Python 深拷贝](http://www.algorithmdog.com/slow-python-deepcopy)。

3.__new__() 与 __init__()的区别；

Use __new__ when you need to control the creation of a new instance. Use __init__ when you need to control initialization of a new instance.

__new__ is the first step of instance creation. It's called first, and is responsible for returning a new instance of your class. In contrast, __init__ doesn't return anything; it's only responsible for initializing the instance after it's been created.

In general, you shouldn't need to override __new__ unless you're subclassing an immutable type like str, int, unicode or tuple.

简单说，__new__ 用来创建对象(控制对象的创建行为), __init__ 用来初始化对象。从这两者的功能上来讲，先执行 __new__ 后执行 __init__ , 即 __new__ 的返回值作为 __init__(self, ..) 的默认第一个参数。在自定义类的时候通常都需要编写 __init__ 方法, 因为对象通常都是由状态的, 所以需要一些变量来表示这些状态。而 __new__ 则经常和元类在一起使用，用来控制类对象(特别是继承后的子类)的创建行为，从而实现一些高级一点的东西，比如 ORM。

refer [Why is __init__() always called after __new__()?](https://stackoverflow.com/questions/674304/why-is-init-always-called-after-new)

4.你知道几种设计模式； 

- 单例模式

Python的module是天然的单例模式

```Python
# singleton.py
class Singleton:
    pass
singleton = Singleton()
```

```Python
# a.py
from singleton import singleton
```

```
# b.py
from singleton import singleton
```

```Python
from a import singleton as SA
from b import singleton as SB

assert(SA is SB)
```

```Python
class Singleton(object):
    def __new__(cls, *args, **kw):
        if not hasattr(cls, '_instance'):
            orig = super(Singleton, cls)
            cls._instance = orig.__new__(cls, *args, **kw)
        return cls._instance

class MyClass(Singleton):
    a = 1
```

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
    pass
```

```Python
class Singleton(type):
    def __init__(self, *args, **kwargs):
        self.__instance = None
        super().__init__(*args, **kwargs)

    def __call__(self, *args, **kwargs):
        if self.__instance is None:
            self.__instance = super().__call__(*args, **kwargs)
            return self.__instance
        else:
            return self.__instance

# Example
class Spam(metaclass=Singleton):
    def __init__(self):
        print('Creating Spam')
```

一种更好的实现单例模式的装饰器模式

```Python
class _SingletonWrapper:
    """
    A singleton wrapper class. Its instances would be created
    for each decorated class. 
    """

    def __init__(self, cls):
        self.__wrapped__ = cls
        self._instance = None

    def __call__(self, *args, **kwargs):
        """Returns a single instance of decorated class"""
        if self._instance is None:
            self._instance = self.__wrapped__(*args, **kwargs)
        return self._instance

def singleton(cls):
    """
    A singleton decorator. Returns a wrapper objects. A call on that object
    returns a single instance object of decorated class. Use the __wrapped__
    attribute to access decorated class directly in unit tests
    """
    return _SingletonWrapper(cls)

```


refer [Python中实现单例模式的几种方法](https://stackoverflow.com/questions/6760685/creating-a-singleton-in-python)

refer [WHY YOU DON'T NEED DESIGN PATTERNS IN PYTHON?](https://ep2017.europython.eu/media/conference/slides/why-you-dont-need-design-patterns-in-python.pdf)

5.编码和解码你了解过么?

Python2中的编码是一个坑, 也正因为如此, 才会有不向后兼容的Python3的出现, Python3中, str 与 byte 泾渭分明, 两者之间的关系如下.

```Python
# Python 3
str.encode() -> bytes
bytes.decode() -> str

str: a sequence of code points (unicode)

bytes: a sequence of bytes
```

Python 3中 str 与 bytes之间的转换必须是显示转换, 不能隐式转换, 否则报错.

[Python 开发中编码遵循三明治原则](https://nedbatchelder.com/text/unipain/unipain.html#35):

```
bytes -> str 解码输出的字节序列

str 100% 业务逻辑只处理文本

str -> bytes 编码输出的文本
```

refer [Unicode之痛](http://pycoders-weekly-chinese.readthedocs.io/en/latest/issue5/unipain.html)

refer [Pragmatic Unicode](https://nedbatchelder.com/text/unipain/unipain.html#1)

Python 2 中: 

```
str: a sequence of bytes

unicode: a sequence of code points (unicode)
```

所以才会有unicode这一单独的类型. Python3中的str可以理解为Python2中的unicode,而 bytes 则是Python2中的bytes.


6.列表推导list comprehension和生成器的优劣； 

列表推导优点: 简洁而强大; 缺点: 列表推导立即执行, 如果列表内元素较多需要占用较大的内存空间.
生成器优点: 由于是先计算后产出值并且是一次产出一个值, 所以内存占用空间少, 并且可以表示无限序列; 缺点: 比普通函数运行慢很多.但这几乎不成问题,Python3中越来越多的接口默认返回的迭代器都是生成器类型.

7.什么是装饰器；如果想在函数之后进行装饰，应该怎么做； 

一个装饰器就是一个函数, 它接受一个函数作为参数并返回一个新的函数.

```Python
from functools import wraps
def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        # decorated after the func
        print('function {} completed.'.format(func.__name__))
        return res
    return wrapper

@log
def add(a, b):
    return a + b

res = add(1, 2)
print(res)
# function add completed.
# 3
```

8.手写个使用装饰器实现的单例模式； 

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
    pass

```

9.使用装饰器的单例和使用其他方法的单例，在后续使用中，有何区别； 

it makes impossible a direct access to the class object without decorator. So you cannot call methods using a class name in unit tests:

```Python
@singleton
class YourClass:
    def method(self):
        ...
YourClass.method(...)
```

this code would not work because YouClass actually contains a wrapper function but not your class object. Also this approach causes another problem if your tests require separate instances of the objects, so a singleton pattern could break an isolation of different tests.


你想要的都在这里[singleton-decorator 1.0.0](https://pypi.python.org/pypi/singleton-decorator/1.0.0).

所以有下面的单利模式装饰器

```Python
class _SingletonWrapper:
    """
    A singleton wrapper class. Its instances would be created
    for each decorated class. 
    """

    def __init__(self, cls):
        self.__wrapped__ = cls
        self._instance = None

    def __call__(self, *args, **kwargs):
        """Returns a single instance of decorated class"""
        if self._instance is None:
            self._instance = self.__wrapped__(*args, **kwargs)
        return self._instance

def singleton(cls):
    """
    A singleton decorator. Returns a wrapper objects. A call on that object
    returns a single instance object of decorated class. Use the __wrapped__
    attribute to access decorated class directly in unit tests
    """
    return _SingletonWrapper(cls)

```

10.手写：正则邮箱地址； 

[^@]+@[^@]+\.[^@]+

refer [Python check for valid email address?](https://stackoverflow.com/questions/8022530/python-check-for-valid-email-address)

11.介绍下垃圾回收：引用计数/分代回收/孤立引用环； 

1. 引用计数

PyObject是每个对象必有的内容，其中ob_refcnt就是做为引用计数。当一个对象有新的引用时，它的ob_refcnt就会增加，当引用它的对象被删除，它的ob_refcnt就会减少.引用计数为0时，该对象生命就结束了。

优点:
- 简单
- 实时性

缺点:
- 维护引用计数消耗资源
- 循环引用


2 标记-清除机制

基本思路是先按需分配，等到没有空闲内存的时候从寄存器和程序栈上的引用出发，遍历以对象为节点、以引用为边构成的图，把所有可以访问到的对象打上标记，然后清扫一遍内存空间，把所有没标记的对象释放。

3. 分代回收:

分代回收的整体思想是：将系统中的所有内存块根据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集频率随着“代”的存活时间的增大而减小，存活时间通常利用经过几次垃圾回收来度量。

Python默认定义了三代对象集合，索引数越大，对象存活时间越长。

举例： 当某些内存块M经过了3次垃圾收集的清洗之后还存活时，我们就将内存块M划到一个集合A中去，而新分配的内存都划分到集合B中去。当垃圾收集开始工作时，大多数情况都只对集合B进行垃圾回收，而对集合A进行垃圾回收要隔相当长一段时间后才进行，这就使得垃圾收集机制需要处理的内存少了，效率自然就提高了。在这个过程中，集合B中的某些内存块由于存活时间长而会被转移到集合A中，当然，集合A中实际上也存在一些垃圾，这些垃圾的回收会因为这种分代的机制而被延迟。

孤立引用环:

- 两个对象可能相互引用，从而构成所谓的引用环。
- 即使是一个对象，只需要自己引用自己，也能构成引用环。

像这样的对象，由于直接或者间接相互引用，导致其引用计数不为0，虽然已经不再使用，但是没有办法被垃圾回收机制回收，所以称为孤立引用环。


refer [Garbage collection in Python: things you need to know](https://rushter.com/blog/python-garbage-collector/)

refer [Java 中的垃圾回收](http://imtangqi.com/2016/06/12/garbage-collection-in-java/) 这个实际上作者讲解的并不清楚，从文章的组织能够看出作者还没有将GC的机制完全掌握。

这里有一篇很好的关于Python Ruby的垃圾回收机制简介[Ruby 画说 Ruby 与 Python 垃圾回收](https://ruby-china.org/topics/28127)


12.多进程与多线程的区别；CPU密集型适合用什么； 

进程是一个程序的执行实例。每一个进程提供执行程序所需的所有资源。（进程本质上是资源的集合）

一个进程有一个虚拟的地址空间、可执行的代码、操作系统的接口、安全的上下文（记录启动该进程的用户和权限等等）、唯一的进程ID、环境变量、优先级类、最小和最大的工作空间（内存空间），还要有至少一个线程。

每一个进程启动时都会最先产生一个线程，即主线程。然后主线程会再创建其他的子线程。

与进程相关的资源包括:

内存页（同一个进程中的所有线程共享同一个内存空间）
文件描述符(e.g. open sockets)
安全凭证（e.g.启动该进程的用户ID）

线程是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。一个线程是一个execution context（执行上下文），即一个cpu执行时所需要的一串指令。

进程与线程区别
1.同一个进程中的线程共享同一内存空间，但是进程之间是独立的。
2.同一个进程中的所有线程的数据是共享的（进程通讯），进程之间的数据是独立的。
3.对主线程的修改可能会影响其他线程的行为，但是父进程的修改（除了删除以外）不会影响其他子进程。
4.线程是一个上下文的执行指令，而进程则是与运算相关的一簇资源。
5.同一个进程的线程之间可以直接通信，但是进程之间的交流需要借助中间代理来实现。
6.创建新的线程很容易，但是创建新的进程需要对父进程做一次复制。
7.一个线程可以操作同一进程的其他线程，但是进程只能操作其子进程。
8.线程启动速度快，进程启动速度慢（但是两者运行速度没有可比性）。

refer [搞定python多线程和多进程](http://www.cnblogs.com/whatisfantasy/p/6440585.html)

多进程 multiprocessing

多线程 multithreading

如果是CPU密集型的任务适合用多进程。最好的办法还是编写C扩展，将计算型任务leftout出去，这样不会占用Python控制的资源。


13.进程通信的方式有几种；

（1） 管道（pipe）：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有血缘关系的进程间使用。进程的血缘关系通常指父子进程关系。

（2）有名管道（named pipe）：有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间通信。

（3）信号量（semophore）：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它通常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

（4）消息队列（message queue）：消息队列是由消息组成的链表，存放在内核中 并由消息队列标识符标识。消息队列克服了信号传递信息少，管道只能承载无格式字节流以及缓冲区大小受限等缺点。

（5）信号（signal）：信号是一种比较复杂的通信方式，用于通知接收进程某一事件已经发生。

（6）共享内存（shared memory）：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问，共享内存是最快的IPC方式，它是针对其他进程间的通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量配合使用，来实现进程间的同步和通信。

（7）套接字（socket）：套接口也是一种进程间的通信机制，与其他通信机制不同的是它可以用于不同及其间的进程通信。

几种线程间的通信机制

1)、锁机制

     1.1 互斥锁：提供了以排它方式阻止数据结构被并发修改的方法。

     1.2 读写锁：允许多个线程同时读共享数据，而对写操作互斥。

     1.3 条件变量：可以以原子的方式阻塞进程，直到某个特定条件为真为止。对条件测试是在互斥锁的保护下进行的。条件变量始终与互斥锁一起使用。

2)、信号量机制：包括无名线程信号量与有名线程信号量

3)、信号机制：类似于进程间的信号处理。

线程间通信的主要目的是用于线程同步，所以线程没有象进程通信中用于数据交换的通信机制。

14.介绍下协程，为何比线程还快； 

协程，又称微线程，纤程。英文名Coroutine。

子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。

协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

协程有何优势？

最大的优势就是协程极高的执行效率。**因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。**

第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。


15.range和xrange的区别（他妹的我学的py3…）； 

Python 2 中的概念, Python 3中已经移除xrange, Python 3 中的 range 就是 Python 2 中的 xrange.

```
range creates a list

xrange is a sequence object that evaluates lazily. xrange(x).__iter__() is a generator.
```

这里有你想要的一切[What is the difference between range and xrange functions in Python 2.X?](https://stackoverflow.com/questions/94935/what-is-the-difference-between-range-and-xrange-functions-in-python-2-x?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

16.由于我有C/C++背景，因此要求用C来手写：将IP地址字符串（比如“172.0.0.1”）转为32位二进制数的函数。


### 算法排序部分

1.手写快排；堆排；几种常用排序的算法复杂度是多少；快排平均复杂度多少，最坏情况如何优化； 

###### quick sort

```Python
import unittest
import random


def partation(nums, low, high):
    pivot = nums[random.randrange(low, high + 1)]
    while low <= high:
        while low <= high and nums[low] < pivot:
            low += 1

        while low <= high and pivot < nums[high]:
            high -= 1
        if low <= high:
            nums[low], nums[high] = nums[high], nums[low]
            low += 1
            high -= 1
    return low


def qsort(nums, low, high):
    pivot = partation(nums, low, high)
    if low < pivot - 1:
        qsort(nums, low, pivot - 1)
    if pivot < high:
        qsort(nums, pivot, high)


def quick_sort(nums):
    if nums is None or len(nums) == 1:
        return
    return qsort(nums, 0, len(nums) - 1)


class QucikSortTest(unittest.TestCase):
    def setUp(self):
        self.nums = random.sample(range(0, 1000), 20)

    def test_quicksort(self):
        sorted_nums = sorted(self.nums)
        qsort_nums = self.nums[:]
        quick_sort(qsort_nums)
        self.assertListEqual(sorted_nums, qsort_nums)
        print(sorted_nums)
        print(qsort_nums)

    def tearDown(self):
        self.nums = None


if __name__ == '__main__':
    unittest.main()

```

平均时间复杂度: O(log(n)), 最坏时间复杂度: O(n^2)


###### merge sort

```Python
import random


def merge_sort(ints):
    length = len(ints)
    if length < 2:
        return ints
    mid = length >> 1
    left, right = merge_sort(ints[:mid]), merge_sort(ints[mid:])
    left_count, right_count = len(left), len(right)
    res = []
    i, j = 0, 0
    while i < left_count and j < right_count:
        if left[i] < right[j]:
            res.append(left[i])
            i += 1
        else:
            res.append(right[j])
            j += 1
    if i == left_count:
        res += right[j:]
    else:
        res += left[i:]
    return res


def test():
    ints = random.sample(range(1, 100), 10)
    print(ints)
    ints = merge_sort(ints)
    print(ints)
    for i in range(len(ints) - 1):
        if ints[i] > ints[i + 1]:
            print('Error')
            break


def main():
    for _ in range(1000):
        test()


if __name__ == '__main__':
    main()
```

平均时间复杂度: O(log(n)), 最坏时间复杂度: O(log(n))


###### heap sort

```Python
import random
import unittest


def siftdown(heap, top, end):
    pos = end
    newitem = heap[pos]
    while pos > top:
        parentpos = (pos - 1) >> 1
        parent = heap[parentpos]
        if newitem < parent: #
            heap[pos] = parent
            pos = parentpos
        else:
            break
    heap[pos] = newitem


def siftup(heap, top):
    end = len(heap)
    pos = top
    newitem = heap[pos]
    childpos = (pos << 1) + 1
    while childpos < end:
        rightpos = childpos + 1
        if rightpos < end and heap[rightpos] < heap[childpos]: #
            childpos = rightpos
        heap[pos] = heap[childpos]
        pos = childpos
        childpos = (pos << 1) + 1

    heap[pos] = newitem
    siftdown(heap, top, pos)


def heapify(heap):
    end = len(heap)
    for i in reversed(range(end >> 1)):
        siftup(heap, i)


def heappush(heap, newitem):
    heap.append(newitem)
    siftdown(heap, 0, len(heap) - 1)


def heappop(heap):
    lastitem = heap.pop()
    if heap:
        returnitem = heap[0]
        heap[0] = lastitem
        siftup(heap, 0)
        return returnitem
    return lastitem


def heapreplace(heap, newitem):
    if heap and heap[0] < newitem: #
        heap[0] = newitem
        siftup(heap, 0)


def nlargest(n, iterable):
    heap = [None for i in range(n)]
    for i, item in enumerate(iterable):
        if i < 10:
            heap[i] = item
        elif i == 10:
            heapify(heap)
            heapreplace(heap, item)
        else:
            heapreplace(heap, item)

    res = [None for i in range(n)]
    for i in reversed(range(n)):
        res[i] = heappop(heap)
    return res


def heap_sort(nums):
    res = []
    heapify(nums)
    while nums:
        res.append(heappop(nums))
    return res


class HeapSortTest(unittest.TestCase):
    def setUp(self):
        self.nums = random.sample(range(0, 100), 10)

    def test_heapsort(self):
        print('before sort:', self.nums)
        sortednums = sorted(self.nums)
        print('after sort:', sortednums)
        heapnums = heap_sort(self.nums[:])
        print('heap sort:', heapnums)
        self.assertListEqual(sortednums, heapnums)

    def test_nlargest(self):
        nums = random.sample(range(0, 10000000), 10000)
        top10nums = sorted(nums, reverse=True)[:10]
        print('top 10 largest element: ', top10nums)
        heap10nums = nlargest(10, nums)
        print('top 10 largest elemtn: ', heap10nums)
        self.assertListEqual(top10nums, heap10nums)

    def tearDown(self):
        self.nums = None


if __name__ == '__main__':
    unittest.main()
```

平均时间复杂度: O(log(n)), 最坏时间复杂度: O(log(n))


2.手写：已知一个长度n的无序列表，元素均是数字，要求把所有间隔为d的组合找出来，你写的解法算法复杂度多少； 
3.手写：一个列表A=[A1，A2，…,An]，要求把列表中所有的组合情况打印出来； 
4.手写：用一行python写出1+2+3+…+10**8 ； 
5.手写python：用递归的方式判断字符串是否为回文； 
6.单向链表长度未知，如何判断其中是否有环； 
7.单向链表如何使用快速排序算法进行排序； 
8.手写：一个长度n的无序数字元素列表，如何求中位数，如何尽快的估算中位数，你的算法复杂度是多少； 
9.如何遍历一个内部未知的文件夹（两种树的优先遍历方式）

网络基础部分

1.TCP/IP分别在模型的哪一层； 

3, 4

```
OSI: 
物理层：EIA/TIA-232, EIA/TIA-499, V.35, V.24, RJ45, Ethernet, 802.3, 802.5, FDDI, NRZI, NRZ, B8ZS 
数据链路层：Frame Relay, HDLC, PPP, IEEE 802.3/802.2, FDDI, ATM,  IEEE 802.5/802.2 
网络层：IP，IPX，AppleTalk DDP 
传输层：TCP，UDP，SPX 
会话层：RPC,SQL,NFS,NetBIOS,names,AppleTalk,ASP,DECnet,SCP 
表示层:TIFF,GIF,JPEG,PICT,ASCII,EBCDIC,encryption,MPEG,MIDI,HTML 
应用层：FTP,WWW,Telnet,NFS,SMTP,Gateway,SNMP 
```

refer [TCP/IP每层及OSI七层对应协议](http://thinking80s.iteye.com/blog/566438)

2.socket长连接是什么意思； 

短连接
连接->传输数据->关闭连接
HTTP是无状态的，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束就中断连接。
也可以这样说：短连接是指SOCKET连接后发送后接收完数据后马上断开连接。
 
长连接
连接->传输数据->保持连接 -> 传输数据-> 。。。 ->关闭连接。
***长连接指建立SOCKET连接后不管是否使用都保持连接，但安全性较差。***
 
http的长连接
HTTP也可以建立长连接的，使用Connection:keep-alive，HTTP 1.1默认进行持久连接。HTTP1.1和HTTP1.0相比较而言，最大的区别就是增加了持久连接支持(貌似最新的 http1.0 可以显示的指定 keep-alive),但还是无状态的，或者说是不可以信任的。
 
什么时候用长连接，短连接？
 长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况，。每个TCP连接都需要三步握手，这需要时间，如果每个操作都是先连接，再操作的话那么处理速度会降低很多，所以每个操作完后都不断开，下次处理时直接发送数据包就OK了，不用建立TCP连接。例如：数据库的连接用长连接， 如果用短连接频繁的通信会造成socket错误，而且频繁的socket 创建也是对资源的浪费。
 
而像WEB网站的http服务一般都用短链接，因为长连接对于服务端来说会耗费一定的资源，而像WEB网站这么频繁的成千上万甚至上亿客户端的连接用短连接会更省一些资源，如果用长连接，而且同时有成千上万的用户，如果每个用户都占用一个连接的话，那可想而知吧。所以并发量大，但每个用户无需频繁操作情况下需用短连好。

refer [TCP/IP，http，socket，长连接，短连接](https://my.oschina.net/OutOfMemory/blog/95803)

3.select和epoll你了解么，区别在哪； 
4.TCP UDP区别；三次握手四次挥手讲一下；

### [TCP(Transimission Control Protocol) 传输控制协议](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)

### [UDP(User Datagram Protocol) 用户数据报协议](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%8A%A5%E5%8D%8F%E8%AE%AE)

TCP与UDP基本区别
  1.TCP面向连接, UDP无连接
  2.TCP要求系统资源较多，UDP较少； 
  3.UDP程序结构较简单，只需需要8个字节的头信息；TCP的头部信息则需要20个字节，开销比较大。 
  4.(可靠的)(字节)流模式（TCP）与(不可靠的)数据报模式(UDP); 
  5.TCP保证数据正确性，UDP可能丢包 
  6.TCP保证数据顺序，UDP不保证 


TCP与UDP区别总结：
1、TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接
2、TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保证可靠交付
3、TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流;UDP是面向报文的。UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）
4、每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信。
5、TCP首部开销20字节;UDP的首部开销小，只有8个字节。
6、TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道。

refer [TCP和UDP的最完整的区别](https://blog.csdn.net/li_ning_/article/details/52117463)

###### 三次握手
所谓三次握手(Three-way Handshake)，是指建立一个 TCP 连接时，需要客户端和服务器总共发送3个数据包。

###### 四次挥手
TCP 的连接的拆除需要发送四个数据包，因此称为四次挥手(Four-way handshake)，也叫做改进的三次握手。

refer [https://hit-alibaba.github.io/interview/basic/network/TCP.html#%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B%E4%B8%8E%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B](https://hit-alibaba.github.io/interview/basic/network/TCP.html)

refer [通俗大白话来理解TCP协议的三次握手和四次分手](https://github.com/jawil/blog/issues/14)

5.TIME_WAIT过多是因为什么； 

###### TIME_WAIT状态

根据TCP协议，主动发起关闭的一方，会进入TIME_WAIT状态，持续2*MSL(Max Segment Lifetime)，缺省为240秒，在这个post中简洁的介绍了为什么需要这个状态。

值得一说的是，对于基于TCP的HTTP协议，关闭TCP连接的是Server端，这样，Server端会进入TIME_WAIT状态，可想而知，对于访问量大的Web Server，会存在大量的TIME_WAIT状态，假如server一秒钟接收1000个请求，那么就会积压240*1000=240，000个TIME_WAIT的记录，维护这些状态给Server带来负担。当然现代操作系统都会用快速的查找算法来管理这些TIME_WAIT，所以对于新的TCP连接请求，判断是否hit中一个TIME_WAIT不会太费时间，但是有这么多状态要维护总是不好。

refer [如何解决TIME_WAIT过多的解决办法（附Socket中的TIME_WAIT状态详解）](http://blog.51cto.com/johnsteven/817224)

6.http一次连接的全过程：你来说下从用户发起request——到用户接收到response； 
7.http连接方式。get和post的区别，你还了解其他的方式么； 
8.restful你知道么； 
9.状态码你知道多少，比如200/403/404/504等等；

refer [HTTP协议详解以及URL具体访问过程](http://www.cnblogs.com/phpstudy2015-6/p/6810130.html)

数据库部分

1.MySQL锁有几种；死锁是怎么产生的；

refer [MySQL锁详解](http://www.cnblogs.com/luyucheng/p/6297752.html)

2.为何，以及如何分区、分表；

refer [mysql分表，分区的区别和联系](http://blog.51yip.com/mysql/1029.html)

refer [MySQL的分区、分表、集群、优化](https://www.jianshu.com/p/65ffb9b58633)

3.MySQL的char varchar text的区别； 

refer [Mysql 中 char 、varchar 、text的区别](http://www.cnblogs.com/cnmenglang/p/6394888.html)

4.了解join么，有几种，有何区别，A LEFT JOIN B，查询的结果中，B没有的那部分是如何显示的（NULL）； 

- 内连接
内连接(INNER JOIN)使用比较运算符进行表间某(些)列数据的比较操作，并列出这些表中与连接条件相匹配的数据行。根据所使用的比较方式不同，内连接又分为等值连 接、自然连接和不等连接三种。 

- 外连接
外连接分为左外连接(LEFT OUTER JOIN或LEFT JOIN)、右外连接(RIGHT OUTER JOIN或RIGHT JOIN)和全外连接(FULL OUTER JOIN或FULL JOIN)三种。与内连接不同的是，外连接不只列出与连接条件相匹配的行，而是列出左表(左外连接时)、右表(右外连接时)或两个表(全外连接时)中所有符合搜索条件的数据行。

- 交叉连接
交叉连接(CROSS JOIN)没有WHERE 子句，它返回连接表中所有数据行的笛卡尔积，其结果集合中的数据行数等于第一个表中符合查询条件的数据行数乘以第二个表中符合查询条件的数据行数。

refer [图解SQL的JOIN](https://coolshell.cn/articles/3463.html)

refer [图解 SQL 里的各种 JOIN](http://mazhuang.org/2017/09/11/joins-in-sql/)

refer [SQL中有几种连接？有什么区别？（左连右连内连和外连？）](https://blog.csdn.net/ruixj/article/details/5479828)

5.索引类型有几种，BTree索引和hash索引的区别（我没答上来这俩在磁盘结构上的区别）； 

- FULLTEXT
全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE INDEX创建FULLTEXT索引，要比先为一张表建立FULLTEXT然后再将数据写入的速度快很多。

全文索引并不是和MyISAM一起诞生的，它的出现是为了解决WHERE name LIKE “%word%"这类针对文本的模糊查询效率较低的问题。在没有全文索引之前，这样一个查询语句是要进行遍历数据表操作的，可见，在数据量较大时是极其的耗时的，如果没有异步IO处理，进程将被挟持，很浪费时间

- HASH
（1）Hash 索引仅仅能满足"=","IN"和"<=>"查询，不能使用范围查询。 
由于 Hash 索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。 
（2）Hash 索引无法被用来避免数据的排序操作。 
由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算； 
（3）Hash 索引不能利用部分索引键查询。 
对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。 
（4）Hash 索引在任何时候都不能避免表扫描。 
前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。 
（5）Hash 索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。 
对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。

- BTREE
BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中，相信学过数据结构的童鞋都对当初学习二叉树这种数据结构的经历记忆犹新，反正愚安我当时为了软考可是被这玩意儿好好地折腾了一番，不过那次考试好像没怎么考这个。如二叉树一样，每次查询都是从树的入口root开始，依次遍历node，获取leaf。

BTREE在MyISAM里的形式和Innodb稍有不同

在 Innodb里，有两种形态：一是primary key形态，其leaf node里存放的是数据，而且不仅存放了索引键的数据，还存放了其他字段的数据。二是secondary index，其leaf node和普通的BTREE差不多，只是还存放了指向主键的信息.

而在MyISAM里，主键和其他的并没有太大区别。不过和Innodb不太一样的地方是在MyISAM里，leaf node里存放的不是主键的信息，而是指向数据文件里的对应数据行的信息.

- RTREE
RTREE在mysql很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

相对于BTREE，RTREE的优势在于范围查找.

（1）对于BTREE这种Mysql默认的索引类型，具有普遍的适用性

（2）由于FULLTEXT对中文支持不是很好，在没有插件的情况下，最好不要使用。其实，一些小的博客应用，只需要在数据采集时，为其建立关键字列表，通过关键字索引，也是一个不错的方法，至少愚安我是经常这么做的。

（3）对于一些搜索引擎级别的应用来说，FULLTEXT同样不是一个好的处理方法，Mysql的全文索引建立的文件还是比较大的，而且效率不是很高，即便是使用了中文分词插件，对中文分词支持也只是一般。真要碰到这种问题，Apache的Lucene或许是你的选择。

（4）正是因为hash表在处理较小数据量时具有无可比拟的素的优势，所以hash索引很适合做缓存（内存数据库）。如mysql数据库的内存版本Memsql，使用量很广泛的缓存工具Mencached，NoSql数据库redis等，都使用了hash索引这种形式。当然，不想学习这些东西的话Mysql的MEMORY引擎也是可以满足这种需求的。


refer [MySQL的btree索引和hash索引的区别](https://www.cnblogs.com/vicenteforever/articles/1789613.html)

refer [MySQL BTree索引和hash索引的区别](https://blog.csdn.net/oChangWen/article/details/54024063)


refer [B-Tree索引与Hash索引的比较 ](https://segmentfault.com/a/1190000003493173)

6.手写：如何对查询命令进行优化；

###### Mysql 优化策略

1.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

2.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，

Sql 代码 : select id from t where num is null;
可以在 num 上设置默认值 0,确保表中 num 列没有 null 值，然后这样查询：

Sql 代码 : select id from t where num=0;
3.应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

4.应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，

Sql 代码 : select id from t where num=10 or num=20;
可以这样查询：

Sql 代码 : select id from t where num=10 union all select id from t where num=20;
5.in 和 not in 也要慎用，否则会导致全表扫描，如：

Sql 代码 : select id from t where num in(1,2,3);
对于连续的数值，能用 between 就不要用 in 了：

Sql 代码 : select id from t where num between 1 and 3;
6.下面的查询也将导致全表扫描：

Sql 代码 : select id from t where name like '%c%';
若要提高效率，可以考虑全文检索。

7.如果在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变量，但优 化程序不能将访问计划的选择推迟到运行时;它必须在编译时进行选择。然 而，如果在编译时建立访问计 划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

Sql 代码 : select id from t where num=@num ;
可以改为强制查询使用索引：

Sql 代码 : select id from t with(index(索引名)) where num=@num ;
8.应尽量避免在 where 子句中对字段进行表达式操作， 这将导致引擎放弃使用索引而进行全表扫描。

Sql 代码 : select id from t where num/2=100;
可以这样查询：

Sql 代码 : select id from t where num=100*2;
9.应尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：

Sql 代码 : select id from t where substring(name,1,3)='abc';#name 以 abc 开头的 id
应改为：

Sql 代码 : select id from t where name like 'abc%';
10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用 索引。

11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件 时才能保证系统使用该索引， 否则该索引将不会 被使用， 并且应尽可能的让字段顺序与索引顺序相一致。

12.不要写一些没有意义的查询，如需要生成一个空表结构：

Sql 代码 : select col1,col2 into #t from t where 1=0;
这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：

Sql 代码 : create table #t(…);
13.很多时候用 exists 代替 in 是一个好的选择：

Sql 代码 : select num from a where num in(select num from b);
用下面的语句替换：

Sql 代码 : select num from a where exists(select 1 from b where num=a.num);
14.并不是所有索引对查询都有效，SQL 是根据表中数据来进行查询优化的，当索引列有大量数据重复时， SQL 查询可能不会去利用索引，如一表中有字段 ***,male、female 几乎各一半，那么即使在 *** 上建 了索引也对查询效率起不了作用。

15.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。

16.应尽可能的避免更新 clustered 索引数据列， 因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

17.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并 会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言 只需要比较一次就够了。

18.尽可能的使用 varchar/nvarchar 代替 char/nchar , 因为首先变长字段存储空间小， 可以节省存储空间， 其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

19.任何地方都不要使用 select * from t ,用具体的字段列表代替“*”,不要返回用不到的任何字段。

20.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限(只有主键索引)。

21.避免频繁创建和删除临时表，以减少系统表资源的消耗。

22.临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用 表中的某个数据集时。但是，对于一次性事件， 最好使用导出表。

23.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table,避免造成大量 log ,以提高速度;如果数据量不大，为了缓和系统表的资源，应先 create table,然后 insert.

24.如果使用到了临时表， 在存储过程的最后务必将所有的临时表显式删除， 先 truncate table ,然后 drop table ,这样可以避免系统表的较长时间锁定。

25.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过 1 万行，那么就应该考虑改写。

26.使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更 有效。

27.与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。

28.在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ,在结束时设置 SET NOCOUNT OFF .无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

29.尽量避免大事务操作，提高系统并发能力。 sql 优化方法使用索引来更快地遍历表。 缺省情况下建立的索引是非群集索引，但有时它并不是最佳的。在非群集索引下，数据在物理上随机存放在数据页上。合理的索引设计要建立在对各种查询的分析和预测上。一般来说：

a.有大量重复值、且经常有范围查询( > ,< ,> =,< =)和 order by、group by 发生的列，可考虑建立集群索引;

b.经常同时存取多列，且每列都含有重复值可考虑建立组合索引;

c.组合索引要尽量使关键查询形成索引覆盖，其前导列一定是使用最频繁的列。索引虽有助于提高性能但 不是索引越多越好，恰好相反过多的索引会导致系统低效。用户在表中每加进一个索引，维护索引集合就 要做相应的更新工作。

30.定期分析表和检查表。

分析表的语法：ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tb1_name[, tbl_name]...
以上语句用于分析和存储表的关键字分布，分析的结果将可以使得系统得到准确的统计信息，使得SQL能够生成正确的执行计划。如果用户感觉实际执行计划并不是预期的执行计划，执行一次分析表可能会解决问题。在分析期间，使用一个读取锁定对表进行锁定。这对于MyISAM，DBD和InnoDB表有作用。

例如分析一个数据表：analyze table table_name
检查表的语法：CHECK TABLE tb1_name[,tbl_name]...[option]...option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
检查表的作用是检查一个或多个表是否有错误，CHECK TABLE 对MyISAM 和 InnoDB表有作用，对于MyISAM表，关键字统计数据被更新

CHECK TABLE 也可以检查视图是否有错误，比如在视图定义中被引用的表不存在。

31.定期优化表。

优化表的语法：OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tb1_name [,tbl_name]...
如果删除了表的一大部分，或者如果已经对含有可变长度行的表(含有 VARCHAR、BLOB或TEXT列的表)进行更多更改，则应使用OPTIMIZE TABLE命令来进行表优化。这个命令可以将表中的空间碎片进行合并，并且可以消除由于删除或者更新造成的空间浪费，但OPTIMIZE TABLE 命令只对MyISAM、 BDB 和InnoDB表起作用。

例如： optimize table table_name
注意： analyze、check、optimize执行期间将对表进行锁定，因此一定注意要在MySQL数据库不繁忙的时候执行相关的操作。

补充：

1、在海量查询时尽量少用格式转换。

2、ORDER BY 和 GROPU BY:使用 ORDER BY 和 GROUP BY 短语，任何一种索引都有助于 SELECT 的性能提高。

3、任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移 至等号右边。

4、IN、OR 子句常会使用工作表，使索引失效。如果不产生大量重复值，可以考虑把子句拆开。拆开的子 句中应该包含索引。

5、只要能满足你的需求，应尽可能使用更小的数据类型：例如使用 MEDIUMINT 代替 INT

6、尽量把所有的列设置为 NOT NULL,如果你要保存 NULL,手动去设置它，而不是把它设为默认值。

7、尽量少用 VARCHAR、TEXT、BLOB 类型

8、如果你的数据只有你所知的少量的几个。最好使用 ENUM 类型

9、正如 graymice 所讲的那样，建立索引。

10、合理用运分表与分区表提高数据存放和提取速度。


refer [30多条mysql语句级优化方法,千万级数据库记录查询轻松解决](https://blog.csdn.net/u013614451/article/details/48902437)


7.NoSQL了解么，和关系数据库的区别；redis有几种常用存储类型；

NoSQL 一般是指非关系型数据库(RDBMS),常用的用redis, mongodb.

Redis相比其它的KV数据库，其一大特点是支持丰富的数据类型.

1. String:

简介：Strings数据类型是最常用、简单的key-value类型，普通的key/ value 存储都可以归为此类。value不仅可以是字符串，也可以是数字。因为是二进制安全的，所以你完全可以把一个图片文件的内容作为string来存储。Redis的string可以完全实现目前 Memcached的功能，并且效率更高。除了提供与 Memcached 一样的get、set、incr、decr 等操作外，Redis还额外提供了下面一些操作：

- 获取字符串长度
- 往字符串append内容
- 设置和获取字符串的某一段内容
- 设置及获取字符串的某一位（bit）
- 批量设置一系列字符串的内容

常用命令:  set,get,decr,incr,mget 等。


应用场景：

- 应用 Memcached和CKV的所有场景。字符串和数字直接存取。结构化数据需要先序列化，再set到value；相应的，get到value后需要反序列化。

- 可以利用redis的INCR、INCRBY、DECR、DECRBY等指令来实现原子计数的效果。即可以用来实现业务上的统计计数需求。也可用于实现idmaker，即生成全局唯一的id。

- 存放session key，实现一个分布式session系统。Redis的key可以方便地设置过期时间，用于实现session key的自动过期。验证skey时先根据uid路由到对应的redis，如取不到skey，则表示skey已过期，需要重新登录；如取到skey且校验通过则升级此skey的过期时间即可。

- Set nx或SetNx，仅当key不存在时才Set。可以用来选举Master或实现分布式锁：所有Client不断尝试使用SetNx master myName抢注Master，成功的那位不断使用Expire刷新它的过期时间。如果Master挂掉了key就会失效，剩下的节点又会发生新一轮抢夺。

- 借助redis2.6开始支持的lua脚本，可以实现更安全的2种分布式锁：一种适用于各进程竞争但总是单个进程获取锁并处理的场景。除非原处理进程挂掉因而锁过期才会被其它进程获取到锁。无须主动解锁。通过get、expire/pexpire、set nx ex| px的lua脚本实现；一种适用于各进程竞争获取锁并处理的场景。通过set nx ex| px获取锁，用完需要通过先get判断再del释放锁，否则在锁过期之前不能获取到锁。

- GetSet，设置新值，返回旧值。比如实现一个计数器，可以用GetSet获取计数并重置为0。

- GetBit/SetBit/BitOp/BitCount， BitMap的玩法，比如统计今天的独立访问用户数时，每个注册用户都有一个offset，他今天进来的话就把他那个位设为1，用BitCount就可以得出今天的总人数。

- Append/SetRange/GetRange/StrLen，对文本进行扩展、替换、截取和求长度，对特定数据格式非常有用。

 
实现方式

String在redis内部存储默认就是一个字符串，被redisObject所引用，当遇到incr,decr等操作时会转成数值型进行计算，此时redisObject的encoding字段为int。

 

2. Hash

简介：Hash存的是字符串和字符串值之间的映射。Hash将对象的各个属性存入Map里，可以只读取/更新对象的某些属性。这样有些属性超长就让它一边呆着不动，另外不同的模块可以只更新自己关心的属性而不会互相并发导致覆盖冲突。

常用命令：hget,hset,hgetall 等。


应用场景：

l  存放结构化数据，比如用户信息。在Memcached或CKV中，对于用户信息比如用户的昵称、年龄、性别、积分等，我们需要先序列化后存储为一个字符串的值，这时候在需要修改其中某一项时，通常需要将所有值取出反序列化后，修改某一项的值，再序列化存储回去。这样不仅增大了开销，也不适用于一些可能并发操作的场合（比如两个并发的操作都需要修改积分）。而Redis的Hash结构可以使你像在数据库中Update一个属性一样只修改某一项属性值。如下图：

Key是用户ID, value是一个Map，这个Map的key是成员的属性名，value是属性值，这样对数据的修改和存取都可以直接通过其内部Map的Key(Redis里称内部Map的key为field), 也就是通过 key(用户ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题。

不过这里需要注意，Redis提供了接口(hgetall)可以直接取到全部的属性数据,但是如果内部Map的成员很多，那么涉及到遍历整个内部Map的操作，由于Redis单线程模型的缘故，这个遍历操作可能会比较耗时，而对其它客户端的请求完全不响应，这点需要格外注意。

可用来建索引。比如User对象，除了id有时还要按name来查询，可以建一个Key为user:name:id的Hash，在插入User对象时(set user:101 {"id":101,"name":"calvin"})，顺便往这个hash插入一条(hset user:name:id calvin 101)，这时calvin作为hash里的一个key，值为101。按name查询的时候，用hget user:name:id calvin 就能从名为calvin的key里取出id。假如需要使用多种索引来查找某条数据时可以使用，一个hash key搞定，避免使用多个string key存放索引值。

l  HINCRBY同样可用于实现idmaker。相对string类型的idmaker每一个类型需要一个key，hash类型的用一个key即可。

 

实现方式：

Redis Hash对应Value内部实际就是一个HashMap，这里会有2种不同实现，这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap,当成员数量增大时会自动转成真正的HashMap,此时encoding为ht。

 

3. List

简介：List是一个双向链表，支持双向的Pop/Push，江湖规矩一般从左端Push，右端Pop——LPush/RPop，而且还有Blocking的版本BLPop/BRPop，客户端可以阻塞在那直到有消息到来。还有RPopLPush/ BRPopLPush，弹出来返回给client的同时，把自己又推入另一个list，LLen获取列表的长度。还有按值进行的操作：LRem(按值删除元素)、LInsert(插在某个值的元素的前后)，复杂度是O(N)，N是List长度，因为List的值不唯一，所以要遍历全部元素，而Set只要O(log(N))。

按下标进行的操作：下标从0开始，队列从左到右算，下标为负数时则从右到左。LSet ，按下标设置元素值。LIndex，按下标返回元素。LRange，不同于POP直接弹走元素，只是返回列表内一段下标的元素，是分页的最爱。LTrim，限制List的大小，比如只保留最新的20条消息。复杂度也是O(N)，其中LSet的N是List长度，LIndex的N是下标的值，LRange的N是start的值+列出元素的个数，因为是链表而不是数组，所以按下标访问其实要遍历链表，除非下标正好是队头和队尾。LTrim的N是移除元素的个数。

常用命令：lpush,rpush,lpop,rpop,lrange等。

 

应用场景：

- 各种列表，比如twitter的关注列表、粉丝列表等，最新消息排行、每篇文章的评论等也可以用Redis的list结构来实现。

- 消息队列，可以利用Lists的PUSH操作，将任务存在Lists中，然后工作线程再用POP操作将任务取出执行。这里的消息队列并没有ack机制，如果消费者把任务给Pop走了又没处理完就死机了怎么办？解决方法之一是加多一个sortedset，分发的时候同时发到list与sorted set，以分发时间为score，用户把任务做完了之后要用ZREM消掉sorted set里的job，并且定时从sorted set中取出超时没有完成的任务，重新放回list。另一个做法是为每个worker多加一个的list，弹出任务时改用RPopLPush，将job同时放到worker自己的list中，完成时用LREM消掉。如果集群管理(如zookeeper)发现worker已经挂掉，就将worker的list内容重新放回主list。

- 利用LRANGE可以很方便的实现list内容分页的功能。

- 取最新N个数据的操作：LPUSH用来插入一个内容ID，作为关键字存储在列表头部。LTRIM用来限制列表中的项目数最多为5000。如果用户需要的检索的数据量超越这个缓存容量，这时才需要把请求发送到数据库。


实现方式：

Redis list的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销，Redis内部的很多实现，包括发送缓冲队列等也都是用的这个数据结构。
 

4. Set

简介：是一种无序的集合，集合中的元素没有先后顺序，不重复。将重复的元素放入Set会自动去重。

常用命令：

sadd,spop,smembers,sunion 等。

应用场景：

- 某些需要去重的列表，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

- 可以存储一些集合性的数据，比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。又比如QQ有一个社交功能叫做“好友标签”，大家可以给你的好友贴标签，比如“大美女”、“土豪”、“欧巴”等等，这里也可以把每一个用户的标签都存储在一个集合之中。

- 想要知道某些特定的注册用户或IP地址，他们到底有多少访问了某个页面，可以这样实现：SADD page:day1:<page_id> <user_id> 。想知道特定用户的数量，使用SCARD page:day1:<page_id>。 需要测试某个特定用户是否访问了这个页面？SISMEMBERpage:day1:<page_id>。


实现方式：

set 的内部实现是一个 value永远为null的HashMap，实际就是通过计算hash的方式来快速排重的，这也是set能提供判断一个成员是否在集合内的原因。


5. SortedSet

简介：有序集合，相比set，元素放入集合时还要提供该元素的分数，可根据分数自动排序。

常用命令：

zadd,zrange,zrem,zcard等

使用场景：

- 存放一个有序的并且不重复的集合列表，比如twitter 的public timeline可以以发表时间作为score来存储，这样获取时就是自动按时间排好序的。

- 可以做带权重的队列，比如普通消息的score为1，重要消息的score为2，然后工作线程可以选择按score的倒序来获取工作任务。让重要的任务优先执行。

- 排行榜相关：ZADD leaderboard <score>  <username> 。 得到前100名高分用户很简单：ZREVRANGE leaderboard 0 99。用户的全球排名也相似，只需要执行：ZRANK leaderboard <username>。

- 新闻按照用户投票和时间排序，ZADD时的score = points / time^alpha， 这样用户的投票会相应的把新闻挖出来，但时间会按照一定的指数将新闻埋下去。

- 过期项目处理：使用unix时间作为关键字，用来保持列表能够按时间排序。对current_time和time_to_live进行检索，完成查找过期项目的艰巨任务。另一项后台任务使用ZRANGE...WITHSCORES进行查询，删除过期的条目。


实现方式：

Redis sorted set的内部使用HashMap和跳跃表(SkipList)来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序依据是HashMap里存的score,使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。
 

refer [Redis数据类型及使用场景](https://blog.csdn.net/stonenie/article/details/53509663)

refer [redis 五种数据类型及其使用场景](http://www.cleey.com/blog/single/id/808.html)

Linux部分

1.讲一下你常用的Linux/git命令和作用；

```bash
ls cd mv rm echo top systemctl service
```

```Git
git add, commit, remote add, log, fetch, branch, checkout reset head
```

2.查看当前进程是用什么命令，除了文件相关的操作外，你平时还有什么操作命令； 

```bash
ps top
```


refer [ps 进程查看器](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/ps.html#)


django项目部分

1.都是让简单的介绍下你在公司的项目，不管是不是后端相关的，主要是要体现出你干了什么；

工具。

2.你在项目中遇到最难的部分是什么，你是怎么解决的； 

google, 然后看别人的代码。

大文件的读写，分块。使用Python的生成器，协程。

3.你看过django的admin源码么；看过flask的源码么；你如何理解开源； 

简单的一部分。

4.MVC / MTV；

MVC: Model, View,     Controller;
MTV: Model, Template, Controller;


5.缓存怎么用； 

redis

6.中间件是干嘛的； 

中间件是介于操作系统和在其上运行的应用程序之间的软件。中间件实质上充当隐藏转换层，实现了分布式应用程序的通信和数据管理。它有时被称为管道，因为它将两个应用程序连接在一起，使数据和数据库可在“管道”间轻松传递。 通过中间件，用户可执行很多请求，例如在 Web 浏览器上提交表单，或者允许 Web 服务器基于用户的配置文件返回动态网页。

常见的中间件示例包括数据库中间件、应用程序服务器中间件、面向消息的中间件、Web 中间件和事务处理监视器。每个程序通常都会提供消息传递服务，让不同的应用程序可使用简单对象访问协议 (SOAP)、Web 服务、表述性状态转移 (REST) 和 JavaScript 对象表示法 (JSON) 等消息传递框架进行通信。虽然所有中间件都执行通信功能，但公司选用的类型将取决于要使用的服务以及需要传达的信息类型。这可包括安全身份认证、事务管理、消息队列、应用程序服务器、Web 服务器和目录。中间件还可用于实时发生的操作的分布式处理，而不是来回发送数据。

refer [什么是中间件](https://azure.microsoft.com/zh-cn/overview/what-is-middleware/)

refer [中间件](https://zh.wikipedia.org/zh-hans/%E4%B8%AD%E9%97%B4%E4%BB%B6)

7.CSRF是什么，django是如何避免的；XSS呢； 

跨站请求伪造（英语：Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。[1] 跟跨网站脚本（XSS）相比，**XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。**

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web中用户身份验证的一个漏洞：**简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。**

防御措施:

- 检查Referer字段

HTTP头中有一个Referer字段，这个字段用以标明请求来源于哪个地址。在处理敏感数据请求时，通常来说，Referer字段应和请求的地址位于同一域名下。以上文银行操作为例，Referer字段地址通常应该是转账按钮所在的网页地址，应该也位于www.examplebank.com之下。而如果是CSRF攻击传来的请求，Referer字段会是包含恶意网址的地址，不会位于www.examplebank.com之下，这时候服务器就能识别出恶意的访问。

这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。但这种办法也有其局限性，因其完全依赖浏览器发送正确的Referer字段。虽然http协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其Referer字段的可能。

- 添加校验token

由于CSRF的本质在于攻击者欺骗用户去访问自己设置的地址，所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在cookie中，并且攻击者无法伪造的数据作为校验，那么攻击者就无法再执行CSRF攻击。这种数据通常是表单中的一个数据项。服务器将其生成并附加在表单中，其内容是一个伪乱数。当客户端通过表单提交请求时，这个伪乱数也一并提交上去以供校验。正常的访问时，客户端浏览器能够正确得到并传回这个伪乱数，而通过CSRF传来的欺骗性攻击中，攻击者无从事先得知这个伪乱数的值，服务器端就会因为校验token的值为空或者错误，拒绝这个可疑请求。

refer [CSRF](https://zh.wikipedia.org/zh/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)


Django 是如何预防CSRF的:


refer [Django CSRF 原理分析](https://blog.csdn.net/u011715678/article/details/48752873)

refer [CSRF攻击原理&Django的应用方法](https://blog.csdn.net/u012556900/article/details/57412707)

refer [Django: csrf防御机制](https://www.jianshu.com/p/a178f08d9389)

refer [Django-CSRF](https://docs.djangoproject.com/en/2.0/ref/csrf/)


8.如果你来设计login，简单的说一下思路； 

9.session和cookie的联系与区别；session为什么说是安全的； 

###### Session:
1. IDU is stored on server (i.e. server-side)
2. Safer (because of 1)
3. Expiration can not be set, session variables will be expired when users close the browser. (nowadays it is stored for 24 minutes as default in php)

###### Cookies:
1. IDU is stored on web-browser (i.e. client-side)
2. Not very safe, since hackers can reach and get your information (because of 1)
3. Expiration can be set (see setcookies() for more information)

Session is preferred when you need to store short-term information/values, such as variables for calculating, measuring, querying etc.

Cookies is preferred when you need to store long-term information/values, such as user's account (so that even when they shutdown the computer for 2 days, their account will still be logged in)

The concept is storing persistent data across page loads for a web visitor. Cookies store it directly on the client. Sessions use a cookie as a key of sorts, to associate with the data that is stored on the server side.

It is preferred to use sessions because the actual values are hidden from the client, and you control when the data expires and becomes invalid. If it was all based on cookies, a user (or hacker) could manipulate their cookie data and then play requests to your site.

refer [Cookies vs. sessions](https://stackoverflow.com/questions/6253633/cookies-vs-sessions?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)


10.uWSGI和Nginx的作用；

一种情况，本地有多个 web 服务，有 Python、php、java 编写的，都想监听 80 端口，这个时候就必须有一个负责转发的服务了。如果本机确定只跑这一个服务，但是 uwsgi 和 gevent 对于静态资源处理的并不是很好，一是性能问题，二是各种 HTTP 请求缓存头，处理的也没有 Nginx 完善。然后还有一些安全问题，Nginx 作为专业服务器，暴露在公网相对比较安全（虽然有著名的心血漏洞），uwsgi 和 gevent 的话，漏洞恐怕只比 Nginx 多而不是少。再来就是支持的协议，uwsgi 和 gunicon 早期是不支持 https 的，只能提供 http 给浏览器访问。虽然现在这两者都支持了，但是以后的 spdy 和http2，恐怕也是 nginx 跟进更快一些。还有一些运维优势，比如服务器被人 CC，这是一种非常常见的情况，nginx 可以比较方便的把一些 IP 加入黑名单，直接改配置文件就好了。要是 uwsgi 或者 gunicorn，恐怕还要修改自己应用的代码，把 IP 过滤写进去。

- Nginx更安全
- Nginx能更好地处理静态资源（通过一些http request header）
- Nginx也可以缓存一些动态内容
- Nginx可以更好地配合CDN
- Nginx可以进行多台机器的负载均衡
- 不需要在wsgi server那边处理keep alive让
- Nginx来处理slow client
- 还有一个更隐蔽的区别是，像uWSGI支持的是wsgi协议，Nginx支持的是http协议，它们之间是有区别的。


refer [Why do I need nginx when I have uWSGI](https://serverfault.com/questions/590819/why-do-i-need-nginx-when-i-have-uwsgi?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

refer [使用了Gunicorn或者uWSGI,为什么还需要Nginx？](https://www.zhihu.com/question/30560394)
