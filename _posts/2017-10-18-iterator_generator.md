---
layout: cnblog_post
title:  "迭代器和生成器"
date:   2017-10-18 06:34:39
categories: Python
---

##### 可迭代对象
可迭代对象是是这样一类对象<br/>
表象：可以使用for-in语句进行遍历，如list tuple dict set str等类型的对象<br/>
在本质上：可迭代对象是`实现了Iterable协议(或者说接口)的实例`，而Iterable中定义了可迭代对象协议的方法:\_\_iter\_\_。也就是说只要是实现了\_\_iter\_\_方法的类实例都是可迭代对象，在Python中Iterable定义为：

```python
class Iterable(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            return _check_methods(C, "__iter__")
        return NotImplemented
```

它本身就是一个抽象类、或者说是一个接口，在Python中没有像Java那样显式遵守一个接口的语法, 如

```java
class ConcreteClass implements Iterable {
    @Override
    public Iterator iterator() {
        // ........
    }
}
```
Python中若遵守了一个接口或者协议，只需要实现接口中定义的抽象方法就可以`隐式遵守`，只要遵守了，便可使用`isinstance(子类对象, 协议)`或者`issubclass(子类, 协议)`这两种方式检测对象(类)是否遵守了协议。而这个过程实际上是这样实现的：<br/>
isinstance(子类对象, 接口)会调用issubclass(type(子类对象), 接口)<br/>
而issubclass(子类, 接口(或父类))方法会返回接口(或父类)的\_\_subclasshook\_\_方法值。

如上面的Iterable接口，只是简单地判断了类型是否实现了\_\_iter\_\_方法，例如：
issubclass(list, Iterable)
最终的值为
_check_method(list, '\_\_iter\_\_')。

那么这个\_\_iter\_\_方法到底是是干什么用的：
这个方法实际上返回了一个当前对象的迭代器，对当前这个可迭代对象进行迭代的时候，实际上是从\_\_iter\_\_方法的返回值中取数据，也就是说\_\_iter\_\_返回了一个进行迭代的数据源，这个数据源通常称为迭代器。


##### 迭代器
迭代器实际上是遵守了迭代器协议Iterator的对象,在这个协议本身遵守并实现了Iterable协议:

```python
def __iter__(self):
    return self
```
另外增加了一个抽象方法：

```python
@abstractmethod
def __next__(self):
    'Return the next item from the iterator. When exhausted, raise StopIteration'
    raise StopIteration
```
可见在迭代器本身进行迭代的数据源就是本身，而`__next__`方法是一个从数据源中取下一个值的过程方法，并且在实现它的时候要注意当迭代完毕的时候要抛出一个StopIteration异常，意味着停止迭代。for-in循环在内部的处理过程就是使用这个异常停止迭代的。

Python内置的`next()`函数实际上是对参数对象调用了`__next__`方法，因此`next()`函数的参数必须是一个迭代器。

Python内置`iter()`函数可以将“可迭代对象”转为“迭代器对象”，这个方法有点像工厂方法，但实际上是直接调用了参数的`__iter__`方法:<br/>
如`iter([1, 2])`其实相当于`[1, 2].__iter__()`<br/>
它会根据传入对象的不同，而产生不同的迭代器，这个迭代器对象的地址不是确定的，每次调用都会生成一个新的对象：

```txt
list  --> list_iterator
dict  --> dict_keyiterator
tuple --> tuple_iterator
set   --> set_iterator
str   --> str_iterator
generator --> generator (自身)
```
for-in 遍历list的实现过程如下：

```python
list_a = [1, 2, 3]

# for-in
for value in list_a:
    print(value)

# while
generator_a = iter(list_a)
while True:
    try:
        value = next(generator_a)
        print(value)
    except StopIteration:
        break
```

##### 生成器

生成器是`迭代器的子协议`,在`__iter__`、`__next__`方法的基础上又增加了`send`, `throw`, `close`这三个抽象方法。

为什么要引入生成器的概念：如果有这样一个需求，创建一个有1_0000_0000个元素的集合，如果使用list来实现，那么这个list占用了大量的内存资源，而且每个元素生成后不会立即释放，这样长期放置在内存中着实浪费。生成器已经不再使用具体集合类型作为数据源，它创建元素和取元素都是动态进行的，这样以`以计算性能换取空间性能`，而且大部分情况下生成值的计算方法都是非常高效的，极大地提升了程序的性能。

如何创建生成器：<br/>
方法①：将列表推导式的`[`和`]`换成`(`和`)`

```python
generator_a = (x*2 for x in range(10))
print(type(generator_a)) # generator
```

方法②：yield语法<br/>
yield本意是生产的意思，在Python中一个yield语句就是为了生成一个值，它的具体使用时，在需要为生成器产生一个新值的时候使用`yield 新值`，最常见的用法是使用与函数定义的语法实现创建一个生成器的功能：

```python
def fibonacci(n):
    a, b = 0, 1
    i = 0
    while i < n:
        yield b
        a, b = b, a + b
```
此时fibonacci不再是一个函数，而是一个创建生成器的模板，如创建有5个元素的斐波那契数列：

```python
fibo = fibonacci(5)
print(fibo) # generator
for value in fibo:
    print(value) # 1 1 2 3 5
```

方法③：自定义生成器子类,与创建自定义的迭代器类的方式相同，只需要重写`__next__`方法实现值的生成规则即可，与实现自定义迭代器子类增多的工作是：需要重写另外两个抽象方法send()和throw()。

```python
class FibonacciGenerator(Generator):
    def __init__(self, n):
        self.n = n
        self.a = 0
        self.b = 1
        self.cur_index = 0

    def __next__(self):
        if self.cur_index >= self.n:
            raise StopIteration
        self.a, self.b = self.b, self.a + self.b
        self.cur_index += 1
        return self.a

    def send(self, value):
        super(FibonacciGenerator, self).send(value)

    def throw(self, typ, val=None, tb=None):
        super(FibonacciGenerator, self).throw(typ, val, tb)
```
测试：

```python
    for i in FibonacciGenerator(5):
        print(i) # 1 1 2 3 5
```

##### send()方法
send()方法为下一次`yield值语句`传递消息，作为这个语句的返回值，消息的类型可以是任意的，也可以是None。
send方法本身会完成两个工作：
1.产生下一个值并作为send()方法的返回值，这个功能与\_\_next\_\_()方法相同
2.将传递的信息作为本次yield语句的返回值，这个返回过程是在下一次调用\_\_next\_\_,也就是程序在继续之后的执行时才会完成。


```python
def test_send(n):
    i = 0
    while i < n:
        message = yield i
        print(message)
        i += 1

t = test_send(5)
print(t.send(None))
print('-----------')
print(t.send('hello'))
print('-----------')
print(t.send('hello'))
print('-----------')
```

输出结果

```txt
0
-----------
hello
1
-----------
hello
2
-----------
```

注意的是第一次取值的时候，如果使用send方法，要传递参数为None，否则会产生异常。
查阅Generator实现代码可知，\_\_next\_\_()方法实际上是调用了send(None)，也就是说生成器的取值都是通过调用send()方法完成的。

