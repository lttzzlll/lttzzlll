---
layout: post
title: Python - Python 自省
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Python, 自省]
---

### Python 中用于自省的一些方法

### 1.  dir([object])

> 列出对象的大多数属性。
官方文档
（https://docs.python.org/3/library/functions.html#dir）说，dir 函数的目的
是交互式使用，因此没有提供完整的属性列表，只列出一组“重要的”属
性名。dir 函数能审查有或没有 __dict__ 属性的对象。dir 函数不会
列出 __dict__ 属性本身，但会列出其中的键。dir 函数也不会列出类
的几个特殊属性，例如 __mro__、__bases__ 和 __name__。如果没有
指定可选的 object 参数，dir 函数会列出当前作用域中的名称。

### 2. getattr(object, name[, default])

> 从 object 对象中获取 name 字符串对应的属性。
获取的属性可能
来自对象所属的类或超类。如果没有指定的属性，getattr 函数抛出
AttributeError 异常，或者返回 default 参数的值（如果设定了这
个参数的话）

### 3. hasattr(object, name)

> 如果 object 对象中存在指定的属性，或者能以某种方式（例如继
承）通过 object 对象获取指定的属性，返回 True。
文档（https://docs.python.org/3/library/functions.html#hasattr）说道：“这个函数
的实现方法是调用 getattr(object, name) 函数，看看是否抛出
AttributeError 异常。

### 4. setattr(object, name, value)

> 把 object 对象指定属性的值设为 value，前提是 object 对象能接受那个值。
这个函数可能会创建一个新属性，或者覆盖现有的属性。

### 5. vars([object])

> 返回 object 对象的 __dict__ 属性；如果实例所属的类定义了
__slots__ 属性，实例没有 __dict__ 属性，那么 vars 函数不能处理
那个实例（相反，dir 函数能处理这样的实例）。如果没有指定参数，
那么 vars() 函数的作用与 locals() 函数一样：返回表示本地作用域
的字典。


摘自 **流畅的Python**
