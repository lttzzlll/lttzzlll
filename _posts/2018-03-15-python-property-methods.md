---
layout: post
title: Python 自省
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Python, Property, 自省]
---

### Python 中用于自省的一些方法

### dir([object])

列出对象的大多数属性。
官方文档
（https://docs.python.org/3/library/functions.html#dir）说，dir 函数的目的
是交互式使用，因此没有提供完整的属性列表，只列出一组“重要的”属
性名。dir 函数能审查有或没有 __dict__ 属性的对象。dir 函数不会
列出 __dict__ 属性本身，但会列出其中的键。dir 函数也不会列出类
的几个特殊属性，例如 __mro__、__bases__ 和 __name__。如果没有
指定可选的 object 参数，dir 函数会列出当前作用域中的名称。

### getattr(object, name[, default])

从 object 对象中获取 name 字符串对应的属性。
获取的属性可能
来自对象所属的类或超类。如果没有指定的属性，getattr 函数抛出
AttributeError 异常，或者返回 default 参数的值（如果设定了这
个参数的话）

### hasattr(object, name)

如果 object 对象中存在指定的属性，或者能以某种方式（例如继
承）通过 object 对象获取指定的属性，返回 True。
文档（https://docs.python.org/3/library/functions.html#hasattr）说道：“这个函数
的实现方法是调用 getattr(object, name) 函数，看看是否抛出
AttributeError 异常。

### setattr(object, name, value)

把 object 对象指定属性的值设为 value，前提是 object 对象能接受那个值。
这个函数可能会创建一个新属性，或者覆盖现有的属性。

### vars([object])

返回 object 对象的 __dict__ 属性；如果实例所属的类定义了
__slots__ 属性，实例没有 __dict__ 属性，那么 vars 函数不能处理
那个实例（相反，dir 函数能处理这样的实例）。如果没有指定参数，
那么 vars() 函数的作用与 locals() 函数一样：返回表示本地作用域
的字典。


### Python 中用于获取、设置、删除和列出属性。

使用点号或内置的 getattr、hasattr 和 setattr 函数存取属性都会触发下述列表中相应的特殊方法。
但是，直接通过实例的 __dict__ 属性读写属性不会触发这些特殊方法——如果需要，通常会使用这种方式跳过特殊方法。

Python 文档“Data model”一章中的“3.3.9. Special method lookup”一节（https://docs.python.org/3/reference/datamodel.html#special-methodlookup）警告说：
对用户自己定义的类来说，如果隐式调用特殊方法，仅当特殊方法在对象所属的类型上定义，而不是在对象的实例字典中定义时，才能确保调用成功。也就是说要假定特殊方法从类上获取，即便操作目标是实例也是如此。因此，特殊方法不会被同名实例属性遮盖。在下述示例中，假设有个名为 Class 的类，obj 是 Class 类的实例，attr 是 obj 的属性。不管是使用点号存取属性，还是使用 19.6.2 节列出的某个内置函数，都会触发下述特殊方法中的一个。例如，obj.attr 和 getattr(obj, 'attr', 42) 都会触发Class.__getattribute__(obj, 'attr') 方法。

### __delattr__(self, name)

只要使用 del 语句删除属性，就会调用这个方法。例如，del obj.attr 语句触发 Class.__delattr__(obj, 'attr') 方法。

### __dir__(self)

把对象传给 dir 函数时调用，列出属性。例如，dir(obj) 触发Class.__dir__(obj) 方法。

### __getattr__(self, name)

仅当获取指定的属性失败，搜索过 obj、Class 和超类之后调用。表达式 obj.no_such_attr、getattr(obj, 'no_such_attr') 和 hasattr(obj, 'no_such_attr') 可能会触发 Class.__getattr__(obj, 'no_such_attr') 方法，但是，仅当在obj、Class 和超类中找不到指定的属性时才会触发。

### __getattribute__(self, name)

尝试获取指定的属性时总会调用这个方法，不过，寻找的属性是特殊属性或特殊方法时除外。点号与 getattr 和 hasattr 内置函数会触发这个方法。调用 __getattribute__ 方法且抛出 AttributeError异常时，才会调用 __getattr__ 方法。为了在获取 obj 实例的属性时不导致无限递归，__getattribute__ 方法的实现要使用super().__getattribute__(obj, name)。

### __setattr__(self, name, value)

尝试设置指定的属性时总会调用这个方法。点号和 setattr 内置函数会触发这个方法。例如，obj.attr = 42 和 setattr(obj, 'attr', 42) 都会触发 Class.__setattr__(obj, 'attr', 42) 方法。

其实，特殊方法 __getattribute__ 和 __setattr__ 不管怎样都会调用，几乎会影响每一次属性存取，因此比__getattr__ 方法（只处理不存在的属性名）更难正确使用。与定义这些特殊方法相比，使用特性或描述符相对不易出错。


摘自 **流畅的Python**
