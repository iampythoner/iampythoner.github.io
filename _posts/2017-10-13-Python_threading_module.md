---
layout: cnblog_post
title:  "Python threading 模块"
date:   2017-10-13 06:34:39
categories: Python 翻译
---

threading 模块定义了基于线程的并行机制的一些基本工具，它的具体文档地址为[https://docs.python.org/3/library/threading.html](https://docs.python.org/3/library/threading.html)
在开篇即说明了threading模块是构建在[_thread](https://docs.python.org/3/library/_thread.html)这个低级模块之上的高级线程编程接口。

在这个模块中定义了实现多线程编程的对象类型Thread，以及实现线程同步和线程通信的对象类型Thread-Local、Lock、RLock、Condition、Semaphore、Event、Barrier，以及实现定时器对象类型Timer。

#### Thread

threading.settrace(func)
为所有threading模块启动的线程设置追踪函数。这个函数会在每个线程的run方法执行之前为其传递sys.settrace()。

threading.setprofile(func)
为所有threading模块启动的线程设置配置函数。这个函数会在每个线程的run方法执行之前为其传递sys.setprofile()

stack_size
返回当创建线程时已经使用的线程栈大小。可选的size参数指定了栈的大小随后创建的线程使用的栈大小，必须指定为0(使用平台或者默认的配置)或者至少为32,768 (32 KiB)的正整数值。如果大小没有指定，会使用0.如果不支持改变线程栈的大小，会引发ValueError并且栈大小不会修改。32 KiB是当前最小支持的栈大小值来，以此来为解释器本身保证充足的栈空间。注意一些平台可能对栈大小的值有特殊的限制，例如要求最小的栈大小 > 32 KiB 或者要求分配系统内存页大小的倍数---平台文档应该会指出更多信息(通常使用4 KiB页大小；如果没有更多的规格信息，建议使用4096倍数的栈大小)。API支持: Windows, 支持POSIX线程的系统.

#### Thread-Local

Thread_local数据是线程指定的。要管理thread-local数据，仅仅创建一个local实例然后将属性放置其中就可以。

```python
mydata = threading.local()
mydata.x = 1
```
这个实例的值在不同的线程中不同。

`Thread-local方案主要用于同一个线程的不同作用域处进行数据共享。`

想要获取更多信息和扩展例子，参见_threading_local模块的文档。

####  Lock Objects

(原始)锁是一个同步工具，当被上锁时其他线程不能持有。在Python中，目前它是可用的最低级的同步工具，它是通过_thread扩展模块直接实现的。

(原始)锁是有两种状态：“上锁”和“解锁”。它在创建时处于解锁状态。它有两个基本方法，acquire()和release()。当状态是解锁时，acquire()改变状态为“上锁”并立即返回。当状态是锁定时，acquire()阻塞直到在其他线程中通过调用release()方法将状态改变为“解锁”，然后调用acquire()会重置为“上锁”状态并返回。release()仅当上锁的状态下调用，它将状态更改为解锁并立即返回。如果尝试release一个“解锁”状态下的锁，会引发RuntimeError。

Lock支持设备管理协议，也就是说可以使用with语句进行方便使用。

当多个线程因为acquire()方法等待状态转为“解锁”时而被阻塞时，release被调用并将状态更改为“解锁”时，只有一个线程可以处理。

acquire() 和 release()方法的执行都是原子性的。

##### class threading.Lock
这个类实现了一个原始的锁对象。一旦线程获取一个锁，随后进行获取的尝试都会被阻塞，知道它被释放。任何线程都可以释放它。

注意Lock实际上是一个工厂方法，会返回一个平台支持的最高效版本的的具体的Lock类的实例。

###### acquire(blocking=True, timeout=-1)
获取一个锁，阻塞或者非阻塞。

当调用时将blocking参数设置为True(默认参数)，当前线程阻塞直到锁被“解锁”，然后设置它为“锁定”并返回True。

当调用时将blocking参数设置为False，则不会阻塞。如果其他设置True的地方被阻塞了，会立即返回False，否则，设置这个锁为上锁状态并返回True。

当调用时为浮点型的超时参数传递了一个正值，当锁不能获取时则最多阻塞指定的秒数。超时参数如果指定为-1，则会一直等待。当blocking参数设置为False时，禁止设置超时时间。

如果成功获取了锁则返回True，否则返回False(比如，指定超时时间过期的情况)。

3.2版本的更改：新增超时时间参数

3.2版本更改：POSIX中，锁的获取现在可以被信号打断。

###### release()
释放一个锁。可以在任何线程中调用，不仅限于已经获取锁的线程。

当锁被“上锁”时，重置为“解锁”状态并返回。如果任何其他线程阻塞等待锁被“解锁”，允许他们中的一个来处理。

当对一个“未上锁”的锁调用时，会引发RuntimeError。

没有返回值。

##### RLock Objects

重入锁是一个同步工具允许在相同线程中多次上锁。在内部，它除了有传统锁的上锁、解锁状态外，还使用了“持有线程”和“递归级别”的概念。上锁状态下，一些线程持有这个锁，在解锁状态下，没有线程持有它。

调用acquire()方法来上锁；一旦线程持有锁这个方法就返回。
调用release()方法来解锁。
acquire()、release()调用对可以嵌套，只有最后的release()（最外层的release()）重置锁为“解锁”，而且使其他线程在acquire()时阻塞。

重入锁也支持上下文管理协议。

##### class threading.RLock
这个类实现了可重入锁对象。一个重入锁必须通过acquired它的线程释放。一旦线程获取了一个重入锁，这个线程可以再次获取它而不会阻塞，每次的获取它都要有对应的释放(acquire和release配对使用)。

注意RLock实际上是一个工厂函数，它会返回一个平台支持的最高效版本的的具体的RLock类的实例。


###### acquire(blocking=True, timeout=-1)
获取一个锁，阻塞或则非阻塞。

当不使用参数调用：如果这个线程已经持有锁，将递归级别增加1，并立即返回。否则，如果其他线程持有锁，则会阻塞，知道锁被解锁。一旦锁被解锁(不被其他任何线程持有),会建立持有关系，设置递归级别为1并返回。如果多于一个线程被阻塞等待锁被解锁，在这个时刻只有一个线程可以建立持有关系。这种情况下没有返回值。

当传递blocking参数为True时，和没有参数时的执行过程相同，返回true。

当传递blocking参数为false时，不会阻塞。如果一次没有参数的调用被阻塞了，则立即返回false，否则，和不传递参数时的执行过程相同，返回true。

当调用时为浮点型的超时参数传递了一个正值，当锁不能获取时则最多阻塞指定的秒数。如果获取了锁返回true，如果过了超时时间仍没获取锁则返回false。

版本3.2的更新：新增超时参数。

###### release()
释放一个锁，将递归级别递减。如果在递减之后为0，重置锁为“解锁”状态(不再被任何线程持有)，如果任何其他线程正在阻塞等待解锁，会让他们中的一个来处理。如果在递归级别递减之后仍然不为0，锁会保持“锁定”状态，仍然被调用的线程持有。

当且仅当调用的线程持有锁时调用这个方法。如果所在“解锁”状态下调用这个方法会引发RuntimeError。

没有返回值。

##### Condition Objects

条件变量总是和一些类型的锁关联，

A condition variable is always associated with some kind of lock; this can be passed in or one will be created by default. Passing one in is useful when several condition variables must share the same lock. The lock is part of the condition object: you don’t have to track it separately.

A condition variable obeys the context management protocol: using the with statement acquires the associated lock for the duration of the enclosed block. The acquire() and release() methods also call the corresponding methods of the associated lock.

Other methods must be called with the associated lock held. The wait() method releases the lock, and then blocks until another thread awakens it by calling notify() or notify_all(). Once awakened, wait() re-acquires the lock and returns. It is also possible to specify a timeout.

The notify() method wakes up one of the threads waiting for the condition variable, if any are waiting. The notify_all() method wakes up all threads waiting for the condition variable.

Note: the notify() and notify_all() methods don’t release the lock; this means that the thread or threads awakened will not return from their wait() call immediately, but only when the thread that called notify() or notify_all() finally relinquishes ownership of the lock.

The typical programming style using condition variables uses the lock to synchronize access to some shared state; threads that are interested in a particular change of state call wait() repeatedly until they see the desired state, while threads that modify the state call notify() or notify_all() when they change the state in such a way that it could possibly be a desired state for one of the waiters. For example, the following code is a generic producer-consumer situation with unlimited buffer capacity:



