---
layout: cnblog_post
title:  "Python threading 模块"
date:   2017-10-13 06:34:39
categories: Python 翻译
---

threading 模块定义了基于线程的并行机制的一些基本工具，它的具体文档地址为[https://docs.python.org/3/library/threading.html](https://docs.python.org/3/library/threading.html)
在开篇即说明了threading模块是构建在[_thread](https://docs.python.org/3/library/_thread.html)这个低级模块之上的高级线程编程接口。

在这个模块中定义了实现多线程编程的对象类型Thread，以及实现线程同步和线程通信的对象类型Thread-Local、Lock、RLock、Condition、Semaphore、Event、Barrier，以及实现定时器对象类型Timer。

<!--Category-->
<div id="navCategory">
	<b>导航</b>
	<ul>
		<li><a href="#anchor1_0">Global Functions</a></li>
        <li><a href="#anchor2_0">Thread-Local Data</a></li>
        <li><a href="#anchor3_0">Thread Objects</a></li>
        <li><a href="#anchor4_0">Lock Objects</a></li>
        <li><a href="#anchor5_0">RLock Objects</a></li>
        <li><a href="#anchor6_0">Condition Objects</a></li>
        <li><a href="#anchor7_0">Semaphore Objects</a></li>
        <li><a href="#anchor8_0">Event Objects</a></li>
        <li><a href="#anchor9_0">Timer Objects</a></li>
        <li><a href="#anchor10_0">Barrier Objects</a></li>
        <li><a href="#anchor11_0">Using locks, conditions, and semaphores in the with statement</a></li>
	</ul>
</div>
<!--Category结束-->

<h4 id="anchor3_0">Thread</h4>

threading.settrace(func)
为所有threading模块启动的线程设置追踪函数。这个函数会在每个线程的run方法执行之前为其传递sys.settrace()。

threading.setprofile(func)
为所有threading模块启动的线程设置配置函数。这个函数会在每个线程的run方法执行之前为其传递sys.setprofile()

stack_size
返回当创建线程时已经使用的线程栈大小。可选的size参数指定了栈的大小随后创建的线程使用的栈大小，必须指定为0(使用平台或者默认的配置)或者至少为32,768 (32 KiB)的正整数值。如果大小没有指定，会使用0.如果不支持改变线程栈的大小，会引发ValueError并且栈大小不会修改。32 KiB是当前最小支持的栈大小值来，以此来为解释器本身保证充足的栈空间。注意一些平台可能对栈大小的值有特殊的限制，例如要求最小的栈大小 > 32 KiB 或者要求分配系统内存页大小的倍数---平台文档应该会指出更多信息(通常使用4 KiB页大小；如果没有更多的规格信息，建议使用4096倍数的栈大小)。API支持: Windows, 支持POSIX线程的系统.

<h4 id="anchor2_0">Thread-Local</h4>

Thread_local数据是线程指定的。要管理thread-local数据，仅仅创建一个local实例然后将属性放置其中就可以。

```python
mydata = threading.local()
mydata.x = 1
```
这个实例的值在不同的线程中不同。

`Thread-local方案主要用于同一个线程的不同作用域处进行数据共享。`

想要获取更多信息和扩展例子，参见_threading_local模块的文档。

<h4 id="anchor4_0">Lock Objects</h4>

锁是一个同步原语，当被上锁时不被指定的线程持有。在Python中，目前它是可用的最低级的同步原语，它是通过_thread扩展模块直接实现的。

锁是有两种状态：“上锁”和“解锁”。它在创建时处于解锁状态。它有两个基本方法，acquire()和release()。当状态是解锁时，acquire()改变状态为“上锁”并立即返回。当状态是锁定时，acquire()阻塞直到在其他线程中通过调用release()方法将状态改变为“解锁”，然后调用acquire()会重置为“上锁”状态并返回。release()仅当上锁的状态下调用，它将状态更改为解锁并立即返回。如果尝试release一个“解锁”状态下的锁，会引发RuntimeError。

Lock支持上下文管理协议，也就是说可以使用with语句进行方便使用。

当多个线程因为acquire()方法等待状态转为“解锁”而被阻塞时，若release被调用并将状态更改为“解锁”，则只有一个线程可以处理。

acquire() 和 release()方法的执行都是原子性的。

##### class threading.Lock
这个类实现了一个原始的锁对象。一旦线程获取一个锁，随后进行获取的尝试都会被阻塞，直到它被释放。任何线程都可以释放它。

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

<h4 id="anchor5_0">RLock Objects</h4>

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

<h4 id="anchor6_0">Condition Objects</h4>

条件变量总是和一些类型的锁关联，可以传递给锁，即使不传递，也会默认创建一个。传递值会在几个条件变量共享相同的锁时非常有用。锁是条件对象的一部分，你不必单独追踪它。

条件变量遵守上下文管理协议，使用with语句在语句块中获取关联的锁。acquire()和release()方法也会调用锁的相应方法。


其他方法必须使用锁的持有者调用。wait()方法释放锁，然后阻塞直到另一个线程通过调用notify()或者notify_all()唤醒。一旦唤醒，wait()会重新获取锁并返回。也可以指定超时。

notify() 方法唤醒一个正在等待条件变量的线程(如果有正在等待的线程)。
notify_all()方法唤醒所有等待条件变量的线程。

注意：notify()和notify_all()方法不会释放锁。这意味着：被唤醒的线程不会从他们的wait()调用中立即返回，但是仅当调用notify()或者notify_all()的线程最终释放了锁的持有关系时会返回。

传统的使用条件变量进行编程的方式是：使用锁同步访问一些共享状态，当修改状态的线程调用notify()或者notify_all()，对指定的状态感兴趣的线程重复调用wait()直到它们看到期待的状态，前提是修改状态的线程可以将状态修改为等待者所期待的状态。例如，下面的代码是一个通用的生产者-消费者问题(使用容量没有限制的buffer)：

```python
# Consume one item
with cv:
    while not an_item_is_available():
        cv.wait()
    get_an_available_item()

# Produce one item
with cv:
    make_an_item_available()
    cv.notify()
```

while循环检查条件是必要的，因为wait()会在一个较长的时间后返回,并且被调用notify()打断的条件不再保持true值。这个是多线程编程的内部实现。wait_for()方法用来使条件自动检查，同时方便对超时的计算。

```python
# Consume an item
with cv:
    cv.wait_for(an_item_is_available)
    get_an_available_item()
```
如何选择notify()或者notify_all(),应考虑是一个还是多个等待线程对状态的改变感兴趣。例如：在传统的生产者消费者问题中，添加一个产品到buffer中仅仅需要唤醒一个消费者线程。

##### class threading.Condition(lock=None)

这个类实现了条件变量对象。一个条件变量使一个或多个线程等待直到他们被另一个线程唤醒。

如果lock参数被指定了并且不为None，它必须是一个Lock或者RLock对象，并且基于它来实现整个过程。否则，一个新的RLock对象会被创建来充当它的作用。

3.3版本的更新：将工厂方法改变为类。

###### acquire(*args)
获取锁，整个方法调用锁的相应方法，返回值是锁方法的返回值。

###### release()
释放锁，整个方法调用锁的相应方法，没有返回值。

###### wait(timeout=None)

等待直到被notify或者直到超时。如果调用线程调用这个方法时没有获取锁，会产生RuntimeError。

这个方法释放锁，然后阻塞直到 其他线程针对同一条件通过调用notify()或者notify_all()将它唤醒。一旦唤醒或者超时，它会重新获取锁并返回。

当提供了超时参数并且不为None, 超时参数应该是一个浮点数，并以秒为单位指定了操作的超时。

当锁是一个RLock对象，它不会使用它的release()方法来释放，因为当锁被多次递归获取时使用releae不会解锁。而作为替代方案，一个RLock类的内部接口会被使用，它会真正实现解锁，即使是在锁被递归获取多次的情况下。然后，另一个内部接口会在锁被重新获取时恢复递归级别。

返回值是True，如果过了超时时间则返回False。

3.2版本中的更新：在这(3.2)之前，总是返回None。

###### wait_for(predicate, timeout=None)

等待直到条件判定为true。谓词(判断条件)应该是可调用的，并且能够返回一个布尔值。超时时间可以指定为最长的等待时间。

这个通用方法会重复调用wait(),直到谓词条件被满足，或者直到超时。返回值是谓词条件最后的返回值，若超时则返回False。

如果忽略超时特性，调用这个方法的过程与“写操作”类似：

```python
while not predicate():
    cv.wait()
```
所以，相同的规则也适用于wait()：当调用时锁必须被持有，并且在返回时锁被重新获取。这个谓词是依据锁的持有状态判定的。

这个方法在 Python 3.2首次增加。

###### notify(n=1)
默认情况，唤醒一个等待这个条件的线程(如果有的话)。如果调用该方法的线程没有获取锁，会引发RuntimeError。

这个方法唤醒最多n个正在等待条件的线程，如果没有线程在等待则没有任何操作。

当前的实现是：如果至少有n个线程在等待，则唤醒n个线程。但是，这个行为是不安全的。在将来(的实现中)，优化的实现可能会唤醒最多n个线程。

注意：一个唤醒的线程直到它可以获取锁才会从它的wait()调用中返回。因为notify()不会释放锁，它的调用者才可以(释放锁)。

###### notify_all()

唤醒所有的正在等待条件的锁。这个方法的行为与notify()类似，但是它唤醒所有正在等待的线程，而不是唤醒一个线程。如果调用这个方法的线程在调用它时还没有获取锁，会引发RuntimeError。

<h4 id="anchor7_0">Semaphore Objects</h4>

在计算机科学发展史中，这是最古老的同步原语，由荷兰计算机科学家Edsger W. Dijkstra（他使用P()和V()两个名字替代acquire()和release()）。

信号量机制管理着一个内部计数，它通过每次acquire()调用递减、通过release()调用递增。计数不会为0以下的值。当acquire()发现这个值为0,它会阻塞等待，直到其他线程调用release().

信号量也支持上下文管理协议。

##### class threading.Semaphore(value=1)
这个类实现了信号量对象。信号量管理着一个计数，这个计数代表着release()调用的次数减去acquire()调用的次数，再加上初始值。acquire()方法阻塞(如果有必要的话)直到它可以返回一个非负的值。如果不指定，默认值是1.

可选参数为内部计数指定了初始值，它的默认值是1.如果给定的值小于0，引发ValueError

版本3.3更新：将工厂方法改变为了类。

###### acquire(blocking=True, timeout=None)
获取一个信号量。

当不传递参数调用时：
如果一个内部计数大于0，减一并立即返回。
如果计数为0，阻塞，等待直到其他线程调用release()来它大于0.
这个工作会使用合适的内部锁，这样，如果多个acquire()调用阻塞，release()会唤醒他们中的一个。这个过程的实现会随机挑选一个，因此唤醒哪个被阻塞的线程的顺序是不确定的。返回true(或者阻塞)。

当调用时传递blocking参数为false，不会阻塞。
如果(其他处)没有参数的调用被阻塞了，立即返回false。否则，与不传参数时的行为相同，并返回true。

当调用时传递timeout参数一个非None值，它会阻塞最多timeout指定秒数。如果acquire没有成功完成，返回false；否则返回true。

版本3.2的更新:新增timeout参数。

###### release()
返回一个信号量，内部计数加一。当它的值为0并且其他线程正在等待它变为一个大于0的值时，唤醒其他线程。

##### class threading.BoundedSemaphore(value=1)
一个实现了有限信号量的类。有限信号量检查保证它的当前值不会超出它的初始值。如果超过了则引发ValueError。
在大多数情况下(这种)信号量被用来保护有限容量的资源。如果信号量release太多次数会引起bug。如果没有指定参数value默认是1.

版本3.3的更新：将工厂方法改为类。

##### Semaphore Example
信号量经常用来保护有限容量的资源，例如，一个数据库服务。在任何资源数不变的情况下，你应该使用有限信号量。在产生任何工作线程之前，在主线程初始化信号量。

```python
maxconnections = 5
# ...
pool_sema = BoundedSemaphore(value=maxconnections)
```

一旦产生(工作线程)，工作线程在当他们需要连接服务的时候调用信号量的 acquire 和 release 方法：

```python
with pool_sema:
    conn = connectdb()
    try:
        # ... use connection ...
    finally:
        conn.close()
```

无限信号量的使用减少了由于信号量release多于acquire而产生的程序错误,这种错误通常是不易发现的。

<h4 id="anchor8_0">Event Objects</h4>

这是一个最简单的线程通信机制:在一个线程上激活一个其他的线程等待着的事件。

事件对象管理一个内部标识，这个标识可以使用set()方法设置为true，使用clear()方法重置为false。wait()方法阻塞直到标识为true。

##### class threading.Event

实现了事件对象的类。事件管理了一个标识，可以使用set()方法设置为true，使用clear()方法重置为false。
wait()方法阻塞直到标识为true。标识的初始值为false。

版本3.3的更新：将工厂方法更新为类。

###### is_set()
返回true 当且仅当内部标识为true。
###### set()
设置内部标识为true。所有等待它变为true的线程都会被唤醒。
一旦标识设置为true，调用wait()的线程则不再阻塞。

###### clear()
重置内部标识为false。随后，调用wait()的线程会被阻塞，直到通过set()方法将内部标识再次设置为true。

###### wait(timeout=None)
阻塞直到内部标识为true。如果内部标识是true，立即返回。否则，阻塞直到其他线程调用set()将标识设置为true,或者直到可选超时时间已经过去。

当传递timeout参数，并且不为None,他被指定为操作等待的秒数。

这个方法返回true，当且仅当内部标识被设置为了true，不管是在wait调用之前还是在wait开始之后。因此，它会总是返回True，除非指定了超时时间并等过了超时时间。

版本3.1中的更新：在这之前，这个方法总是返回None

<h4 id="anchor9_0">Timer Objects</h4>
这个类代表这样一种行为：可以在指定的时间之后执行。
Timer是Thread的子类，因此具有像下面例子中一样创建自定义线程的功能。

Timer启动的方式和线程一样，调用start()方法即可。timer可以通过调用cancel方法来停止(需要在任务开始之前)。任务执行之前等待的时间可能会与用户通过interval参数指定的时间稍微不一致。

例如：

```python
def hello():
    print("hello, world")

t = Timer(30.0, hello)
t.start()  # after 30 seconds, "hello, world" will be printed
```

##### class threading.Timer(interval, function, args=None, kwargs=None)
创建一个timer，它会依据指定的args参数和kwargs参数值在过了interval指定秒数之后来执行任务。如果args为None，会使用空列表。如果kwargs为None会使用空字典。

版本3.3的更新：工厂方法更新为类。

###### cancel()
停止timer，并取消timer定义任务的执行。它仅在timer还处于等待状态时执行。

<h4 id="anchor10_0">Barrier Objects</h4>

在版本3.2新增。

这个类为需要互相等待的固定数目的线程提供了简单的同步原语。每个线程试图通过调用wait()方法传递barrier（可译为屏障），线程会阻塞直到所有的线程调用了他们的wait()方法。这意味着，所有的线程都release。

This class provides a simple synchronization primitive for use by a fixed number of threads that need to wait for each other. Each of the threads tries to pass the barrier by calling the wait() method and will block until all of the threads have made their wait() calls. At this point, the threads are released simultaneously.

barrier可以针对同样数目的线程多次重用。
The barrier can be reused any number of times for the same number of threads.
例如，这是一个用来来同步客户端和服务端线程的简单方式：
As an example, here is a simple way to synchronize a client and server thread:

```python
b = Barrier(2, timeout=5)

def server():
    start_server()
    b.wait()
    while True:
        connection = accept_connection()
        process_server_connection(connection)

def client():
    b.wait()
    while True:
        connection = make_connection()
        process_client_connection(connection)
```

##### class threading.Barrier(parties, action=None, timeout=None)
为多个线程创建一个barrier对象。action是一个可以调用的任务，会在线程release时，由其中一个线程调用。timeout是为wait()方法指定的参数，默认是None。

###### wait(timeout=None)
通过barrier。当所有的与barrier关联的线程调用这个方法，他们会同时release。如果提供了timeout参数，它会影响任务的执行。

返回值是一个整型，范围是0~关联线程数-1，每个线程会有不同。可以使用这个值选择一个线程来做一些特殊的整理工作。例如：
The return value is an integer in the range 0 to parties – 1, different for each thread. This can be used to select a thread to do some special housekeeping, e.g.:

```python
i = barrier.wait()
if i == 0:
    # Only one thread needs to print this
    print("passed the barrier")
```

如果构造器提供了action值，其中一个线程会在被释放之前调用这个方法。如果这个调用引发了错误，barrier会放进broken状态。

如果调用超时，这个barrier会放进broken状态。

这个方法会引发BrokenBarrierError异常，如果barrier被broken或者当一个线程正在等待时被重置。


###### reset()

将barrier置为默认的空状态。任何等待它的线程会收到BrokenBarrierError异常。

注意：如果有一些其他状态未知的线程,使用这个函数可以获取一些外部同步。如果barrier是broken状态，丢弃这个barrier创建一个新的barrier会更好些。

###### abort()
将barrier置为broken状态。这会使任何正在进行的或之后进行的对wait()的调用失败，引发一个BrokenBarrierError。要在需要中止的时候使用这个方法，来避免程序死锁。

它会使用合适的超时值简单地创建一个barrier来自动地保护线程防止出现问题。

###### parties
需要传递给barrier的线程数目。

###### n_waiting
当前正在等待barrier的线程数目。

###### broken
如果barrier处于broken状态返回True

##### exception threading.BrokenBarrierError
RuntimeError的子类，会在Barrier对象被重置或者broken时产生。

<h4 id="anchor11_0">Using locks, conditions, and semaphores in the with statement</h4>
这个模块中所有的有acquire()和release()方法的类都可以如同上下文管理器一样使用with语句。acquire()方法会在块语句进入的时候调用，并在块语句离开的的时候调用release()方法。因此，有下面的代码片段：

```python
with some_lock:
    # do something...
is equivalent to:

some_lock.acquire()
try:
    # do something...
finally:
    some_lock.release()
```
目前，Lock、RLock、Condition、Semaphore和BoundedSemaphore对象都可以使用with语句。



