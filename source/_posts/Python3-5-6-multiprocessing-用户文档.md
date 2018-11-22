---
title: Python3.5.6 multiprocessing 用户文档
date: 2018-11-16 11:44:02
categories:
- "Python"
tags:
- "Python"
- "documentation"
- "Translation"
---

最近在做 Python 内存优化时，发现 `multiprocessing` 可以充分利用 CPU 的多核特性，且可以在执行完任务后释放所有的资源，只保留需要的结果即可，可以避免 Python 的垃圾回收过慢的弊病，同时带来执行效率的提升。同时，之前的 Python 资料书籍更关注于 Python 多线程部分，而对多进程的讲解应用较少。于是开始阅读该部分文档和源码，本篇博客是其官网文档 `multiprocessing` 的翻译（[原文地址](https://docs.python.org/3.5/library/multiprocessing.html#)）。

 <!--more-->

### Introduction ###

`multiprocessing` 是一个支持产生新进程的 package，API 使用方式与 `threading` 模块相似。`multiprocessing` 可以提供本地/远程的线程并发，通过使用子进程而不是线程绕开 GIL 的约束（充分利用 CPU 多核提升效率）。因此，`multiprocessing` 模块可以支持程序充分利用给定计算机上的多个处理器。Unix/Windows 环境均可以运行。

`multiprocessing`同样引入了 `threading`中没有的 APIs。一个主要的例子如 `Pool`，提供了便捷的方法实现跨多个输入值的函数并行执行，可以跨进程分配输入的数据（数据并行）。下面的例子演示`Pool`在模块中定义这样一个函数的常见用法，以便子进程可以成功导入该模块，实现数据的并行性。

```python
from multiprocessing import Pool

def f(x):
    return x*x

if __name__ == '__main__':
    with Pool(5) as p:
        print(p.map(f, [1, 2, 3]))
```

打印结果如下

```python
[1, 4, 9]
```

#### Process 类 ####

`multiprocessing`通过创建一个 `Process` 对象调用其 `start()` 方法产生子进程。`Process` 的 API 与 `threading.Thread`一致，一个多进程程序的例子如下所示。

```python
from multiprocessing import Process

def f(name):
    print('hello', name)

if __name__ == '__main__':
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```

为了显示每个进程的独立 ID，一个扩展的程序实例如下。

```python
from multiprocessing import Process
import os

def info(title):
    print(title)
    print('module name:', __name__)
    print('parent process:', os.getppid())
    print('process id:', os.getpid())

def f(name):
    info('function f')
    print('hello', name)

if __name__ == '__main__':
    info('main line')
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```

如果读者需要了解 `if __name__ == '__main__'`，查看 [Programming Guidlines](#3)。

#### Context 与 start 方法 ####

依据运行环境的不同（Unix/Windows），`multiprocessing` 支持三种方式创建子进程，这三种方式是

* *spawn* 

  > 父进程开启一个全新的 Python 解释器进程。子进程仅继承运行进程对象的 `run()` 方法所必须的资源。特别的是，不会从父进程中继承不需要的文件描述和文件句柄。与下面两种方法相比，此方法启动比较慢。

  > Unix 与 Windows 环境均存在该方法，Windows 下默认使用该方法。

* *fork*

  > 父进程使用 `os.fork()` 复制 Python 解释器。子进程开始运行时实际是与父进程一样的。父进程的所有资源均会被子进程继承。注意拷贝一个多线程的进程可能会有问题。

  > 仅 Unix 环境存在该方法，Unix 默认使用该方法。

* forkserver

  > 当程序开始运行并选择了 `forkserver` 方法，

  > 仅支持 Unix 管道通信传输文件描述符的 Unix 平台支持该方法。

在 3.4 版本之后，所有的 Unix 环境支持了`spawn` 方法，部分 Unix 环境支持了 `forkserver`  方法，Windows 环境下子进程不再继承父进程的所有可继承的句柄。

Unix 环境下使用 `spawn` 或 `forkserver` 方法同样会开启一个信号量跟踪器进程跟踪由程序创建的未连接的命名信号量。当所有的进程退出时，信号量跟踪器取消链接任何剩余的信号量。一般情况下是没有的，但如果一个进程被信号量杀死，可能会有信号量泄漏。（没有链接的命名信号量是一个严重的问题，因为操作系统只允许有限数量的信号量且直至下次重启不会主动取消链接。）

> 猜测：该方式只拷贝了父进程的信号量而没有拷贝文件描述符，因此是未连接的命名信号量，文件描述符或句柄由父进程的信号量连接。正常情况下，进程结束时，信号跟踪器会负责销毁该信号量，当意外退出或被 kill -9 杀死时，未能完成信号量的回收操作导致信号量泄漏。

可以在主函数的 `if __name__ == '__main__'` 从句中使用 `set_start_method()` 方法选择启动方法，如下实例。

```python
import multiprocessing as mp

def foo(q):
    q.put('hello')

if __name__ == '__main__':
    mp.set_start_method('spawn')
    q = mp.Queue()
    p = mp.Process(target=foo, args=(q,))
    p.start()
    print(q.get())
    p.join()
```

程序中 `set_start_method()` 方法至多被使用一次，如像上述实例中，使用错误会报错

```
RuntimeError: context has already been set.
```

或者，可以使用 `get_context()` 获取一个 `context` 对象，`context` 对象作为 `multiprocessing` 模块的一元，具有同样的设置开启子进程方式的方法，并允许在同一程序中使用多个启动方法。

```python
import multiprocessing as mp

def foo(q):
    q.put('hello')

if __name__ == '__main__':
    ctx = mp.get_context('spawn')
    q = ctx.Queue()
    p = ctx.Process(target=foo, args=(q,))
    p.start()
    print(q.get())
    p.join()
```

注：与 `context` 相关的对象可能与不同 `context` 的进程不兼容。特别是使用 `fork()` 方法的 `context` 创建的锁不能传递给使用 `spawn` 或 `forkserver` 方法启动的进程。

使用特别启动方法的库应该使用 `get_context()` 避免干扰库使用者对启动方法的选择。

#### 进程间信息交换 ####

进程间 `multiprocessing` 支持两种类型的交流通道：`Queue` 和 `Pipe` 。

`Queue` 是差不多克隆自 `queue.Queue`。代码实例如下

```python
from multiprocessing import Process, Queue

def f(q):
    q.put([42, None, 'hello'])

if __name__ == '__main__':
    q = Queue()
    p = Process(target=f, args=(q,))
    p.start()
    print(q.get())    # prints "[42, None, 'hello']"
    p.join()
```

`Queue` 是线程/进程安全的。

`Pipe()` 函数返回一对由管道连接的连接对象（默认是双工的）。代码实例如下

```python
from multiprocessing import Process, Pipe

def f(conn):
    conn.send([42, None, 'hello'])
    conn.close()

if __name__ == '__main__':
    parent_conn, child_conn = Pipe()
    p = Process(target=f, args=(child_conn,))
    p.start()
    print(parent_conn.recv())   # prints "[42, None, 'hello']"
    p.join()
```

两个连接对象代表了管道的两端。每一个连接对象都有 `send()` 和 `recv()` （彼此间）。

注：如果两个进程（或线程）同时读取或写入管道同一端的数据，则数据可能被损毁。当然，进程使用管道不同端的不存在数据损坏的风险。

#### 进程间同步 ####

`multiprocessing` 模块包含 `threading` 所有同步原语的等价语句。例如我们可以使用 `lock` 保证同一时刻只有一个进程在进行标准输出。

```python
from multiprocessing import Process, Lock

def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()

if __name__ == '__main__':
    lock = Lock()

    for num in range(10):
        Process(target=f, args=(lock, num)).start()
```

如果不使用 `lock` ，来自不同进程的输出可能会混淆不清。

#### 进程间状态共享 ####

如上所述，在进行并发编程时，尽量不适用状态共享，多线程编程时尤其如此。

然而，如果你确实需要共享数据，`multiprocessing` 提供了两种方式实现。

##### 内存共享 #####

数据可以以 `Value` 或者 `Array` 的方式在内存中共享。代码实例如下

```python
from multiprocessing import Process, Value, Array

def f(n, a):
    n.value = 3.1415927
    for i in range(len(a)):
        a[i] = -a[i]

if __name__ == '__main__':
    num = Value('d', 0.0)
    arr = Array('i', range(10))

    p = Process(target=f, args=(num, arr))
    p.start()
    p.join()

    print(num.value)
    print(arr[:])
```

打印结果如下

```
3.1415927
[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
```

创建变量 `num` 和 `arr` 时实用的参数 'd' 和 'i' 指定数据类型的代码：'d' 表示双精度浮点型，'i' 表示有符号整型。共享对象是进程/线程安全的。

为了更灵活地使用内存共享，可以使用 `multiprocessing.sharedctypes` 模块，该模块支持从共享内存分配出来的任意 [ctypes](https://docs.python.org/3.5/library/ctypes.html) 对象。

##### 服务进程 #####

`Manager()` 返回的 `manager` 对象管理一个服务器进程持有 Python 对象且允许其他进程使用代理操作这些对象，支持 `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` 和 `Array`。代码示例如下

```python
from multiprocessing import Process, Manager

def f(d, l):
    d[1] = '1'
    d['2'] = 2
    d[0.25] = None
    l.reverse()

if __name__ == '__main__':
    with Manager() as manager:
        d = manager.dict()
        l = manager.list(range(10))

        p = Process(target=f, args=(d, l))
        p.start()
        p.join()

        print(d)
        print(l)
```

打印结果如下

```
{0.25: None, 1: '1', '2': 2}
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

使用服务进程 `manager` 比使用内存共享更灵活，因为 `manager` 可以支持任意对象类型。此外，单个 `manager` 可以通过网络在不同计算机上实现进程共享。**但是**，`manager` 方式比使用内存共享慢。

#### 使用 Pool workers ####

`Pool` 对象表示一个进程池，其允许以几种不同的方式将任务装载到进程池。代码示例如下

```python
from multiprocessing import Pool, TimeoutError
import time
import os

def f(x):
    return x*x

if __name__ == '__main__':
    # start 4 worker processes
    with Pool(processes=4) as pool:

        # print "[0, 1, 4,..., 81]"
        print(pool.map(f, range(10)))

        # print same numbers in arbitrary order
        for i in pool.imap_unordered(f, range(10)):
            print(i)

        # evaluate "f(20)" asynchronously
        res = pool.apply_async(f, (20,))      # runs in *only* one process
        print(res.get(timeout=1))             # prints "400"

        # evaluate "os.getpid()" asynchronously
        res = pool.apply_async(os.getpid, ()) # runs in *only* one process
        print(res.get(timeout=1))             # prints the PID of that process

        # launching multiple evaluations asynchronously *may* use more processes
        multiple_results = [pool.apply_async(os.getpid, ()) for i in range(4)]
        print([res.get(timeout=1) for res in multiple_results])

        # make a single worker sleep for 10 secs
        res = pool.apply_async(time.sleep, (10,))
        try:
            print(res.get(timeout=1))
        except TimeoutError:
            print("We lacked patience and got a multiprocessing.TimeoutError")

        print("For the moment, the pool remains available for more work")

    # exiting the 'with'-block has stopped the pool
    print("Now the pool is closed and no longer available")
```

注：`Pool` 的方法只能由创建它的进程使用。

> Tips: 该包中的功能要求 `__main__` 模块可由子项导入。在 Programming Guidelines 中有所涉及，但值得在此指出。这意味着某些例子，如下 `multiprocessing.pool.Pool` 的实例在交互式解释器中不起作用。
    ```python
        from multiprocessing import Pool
        
        def f(x):
            return x*x
        
        p = Pool(5)
        p.map(f, [1,2,3])
        # Process PoolWorker-1:
        # Process PoolWorker-2:
        # Process PoolWorker-3:
        # Traceback (most recent call last):
        # AttributeError: 'module' object has no attribute 'f'
        # AttributeError: 'module' object has no attribute 'f'
        # AttributeError: 'module' object has no attribute 'f'
    ```
>
> 如果你尝试运行该代码，会以半随机的方式输出三个完整的回溯，然后不能不以某种方式停止主进程。

### Reference ###

`multiprocessing` 模块主要复制了 `threading` 模块的 API。

#### Process && Exceptions ####

multiprocessing.**Process**(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)

>  `Process` 对象表示另一个进程中的活动。`Process` 类具有 `threading.Thread` 的所有方法的等价方法。
>
>  应该始终使用关键字参数调用构造函数。参数 `group` 应该始终是 `None`，它仅为了与 `threading.Thread` 构造参数一致。参数 `target` 是 `run()` 方法的调用函数，默认为 `None`，即不调用任何内容。参数 `name` 是进程的名字，详细情况见下参数部分。`kwargs` 是目标函数调用的参数关键字字典，若提供了该参数，将关键字参数 `daemon` 设置为 `True/False`，若为默认值 `None`，则从创建该进程的进程继承该参数。
>
>  默认情况下，不会将任何参数传递给 `target`。
>
> 如果子类重写构造函数，必须确保在对进程执行任何动作之前先调用 `Process.__init__()` 方法。
>
> 3.3版本后添加了 `daemon` 参数。
>
> 构造函数源码如下所示。
>
> ```python
> # cpython 3.5.2 /Lib/multiprocessing/process BaseProcess 类
> def __init__(self, group=None, target=None, name=None, args=(), kwargs={},
>              *, daemon=None):
>     assert group is None, 'group argument must be None for now'
>     count = next(_process_counter)
>     self._identity = _current_process._identity + (count,)
>     self._config = _current_process._config.copy()
>     self._parent_pid = os.getpid()
>     self._popen = None
>     self._target = target
>     self._args = tuple(args)
>     self._kwargs = dict(kwargs)
>     self._name = name or type(self).__name__ + '-' + \
>     			 ':'.join(str(i) for i in self._identity)
>     if daemon is not None:
>         self.daemon = daemon
>     _dangling.add(self)
> ```
>
> **run()**
>
> > 表示进程动作（即进程的执行任务）的方法。
> >
> > 可以在子类中重写该方法。标准 `run()` 方法会调用构造函数参数 `target` 传递进来的目标函数（如果有），分别按顺序使用 `args` 和 `kwargs` 参数中的关键字参数。
>
>  **start()**
>
> > 进程启动的方法。
> >
> > 该方法在每一个进程中至多被调用一次，其在一个单独进程中调用 `Process` 对象的 `run()` 方法。
>
>  **join([timeout])**
>
> > 若可选参数 `timeout` 为 `None` （默认为 `None` ），该方法会阻塞进程直至调用该方法的进程终止；
> >
> > 若可选参数 `timeout` 为整数，该方法会阻塞进程 `timeout` 秒；
> >
> > 注：若方法进程终止或超时，返回 `None`。可以通过检查进程的 `exitcode` 属性值确认进程是否终止。
>
> **name**
>
> > 进程名称。`name` 是字符串类型，仅用于识别目的，没有语义。多个进程可以使用相同的名字。
> >
> > 初识名字由构造器设置，若构造器没有提供显示名字，按照 "Process-{N<sub>1</sub>:N<sub>2</sub>:...:N<sub>k</sub>}" 形式构造名字，其中 "N<sub>k</sub>" 表示父进程的第 N<sub>k</sub> 个子进程。
>
> **is_alive()**
>
> > 返回进程是否活着。
> >
> > 粗略的说，从进程调用 `start()` 方法到进程终止之间进程状态时活着的。
>
> **daemon**
>
> > 进程后台运行标志，Boolean 类型。必须在调用 `start()` 方法之前设置 - 这样可以后台运行进程。
> >
> > 初始值继承自父进程（即创建进程）。当进程退出时，它会尝试其终止其进程的所有子进程。
> >
> > 注：后台运行的进程不允许创建子进程，否则子进程会变成孤儿进程。此外，后台进程不能是 Unix 守护进程或服务，当非守护进程退出时，它们可以正常退出。

除了 `threading.Thread` API 之外，`Process` 还支持如下属性或方法

>  **pid**
>
> > 返回进程 ID，进程生成之前返回 `None`。
>
> **exitcode**
>
> > 子进程退出码。
> >
> > `None` 表示进程尚未终止；返回负整数 `-N` 表示进程被信号 `N` 终止；返回0表示没有错误正常退出；返回正数表示进程有错误，并以错误码为状态码退出。
>
> **authkey**
>
> > 程序的身份认证秘钥是一个 byte 类型的字符串。
> >
> > `multiprocessing` 初始化时，会使用 `os.urandom()` 方法为主进程分配一个随机字符串。
> >
> > 当创建 `Process` 对象时，它会继承父进程的 `authkey`，可以通过设置 `authkey` 来改变它。
>
> **sentinel**
>
> > 系统对象的数字句柄，当进程结束时状态变为 ready。
> >
> > 一次等待多个事件时可以使用 `multiprocessing.connection.wait()`，否则使用 `join()` 方法会更简单。
> >
> > Windows 环境下这是一个系统句柄调用 `WaitForSingleObject` 和 `WaitForMultipleObjects` API 族；Unix 环境下是一个文件描述符，与 `select` 模块中的原语配合使用。
>
> **terminate()**
>
> > 终止进程。Unix 环境下使用 `SIGTERM` 信号量完成；Windows 环境下调用 `TerminatedProcess()` 方法实现。但不会执行退出句柄和剩余子句。
> >
> > 注：进程的后裔进程不会被终止，后裔进程将称为孤儿进程。
> >
> > 警告：如果相互关联的进程正在使用管道或队列时，调用了该方法，管道或队列可能会被损毁且无法被其他进程使用。类似的，如果进程已获得了锁或者信号量，终止它可能导致其他进程的死锁。

注：`start()`，`join()`，`is_alive()`，`terminate()` 和 `exitcode` 这些方法应该仅由父进程对象调用。

如下是一个 `Process` 方法的使用实例。

```python
>>> import multiprocessing, time, signal
>>> p = multiprocessing.Process(target=time.sleep, args=(1000,))
>>> print(p, p.is_alive())
<Process(Process-1, initial)> False
>>> p.start()
>>> print(p, p.is_alive())
<Process(Process-1, started)> True
>>> p.terminate()
>>> time.sleep(0.1)
>>> print(p, p.is_alive())
<Process(Process-1, stopped[SIGTERM])> False
>>> p.exitcode == -signal.SIGTERM
True
```

*exception* multiprocessing.**ProcessError**

> `multiprocessing` 异常类型的基类。

*exception* multiprocessing.**BufferTooShort**

> 当提供的缓存对读取的消息来说太小时，由 ``Connection.recv_bytes_into()`` 方法抛出的异常。

*exception* multiprocessing.**AuthenticationError**

> 认证失败抛出的异常。

*exception* multiprocessing.**TimeoutError**

> 超时抛出的异常。

#### Pipes && Queues ####

通常多进程之间通信应该使用消息传递，尽量避免使用任何同步原语（如锁）。

消息传递可以使用 `Pipe()` （两个进程间通信）或 `queue`（多个生产者和消费者之间通信）。

`Queue`、`SimpleQueue` 和 `JoinableQueue` 类型是基于标准库中 `queue.Queue` 类的基础上构建的多生产者，多消费者 FIFO 队列。不同之处在于 `Queue` 与 `queue.Queue` 相比缺少 `task_done()` 和 `join()` 方法（该方法在 Python 2.5 之后的版本引入）。

如果使用 `JoinableQueue` 则一定要为从队列中删除的每个任务调用 `JoinableQueue.task_done()`，否则可能导致用于统计未完成任务数量的信号量溢出，从而引发异常。

注：也可以通过 `Manager` 对象创建一个共享队列。

> Tip：`multiprocessing` 使用 `queue.Empty` 和 `queue.Full` 异常发出超时信号。这些不能在 `multiprocessing` 的命名空间中找到，因此需要从 `queue` 中引用进来。

> Tip：当一个对象被放入队列时，对象是被序列化了的，之后后台线程会把序列化之后的数据缓冲到一个底层管道。这可能是一个令人惊讶的结果，但不会造成任何困难。如果这个操作会影响到你的任务，可以使用  `manager`。
>
> 1. 将一个对象放入到空队列后，可能有一个无限小的延迟队列的 `empty()` 方法返回 `True` 。`get_nowait()` 方法可以在不抛出 `queue.Empty` 异常的情况下返回队列状态。
>
>   ```python
>   # CPython 3.5.2 /Lib/multiprocessing/queues.py
>   #
>   # Queue type using a pipe, buffer and thread
>   #
>   class Queue(object):
>       # ... ignore other methods
>       def get(self, block=True, timeout=None):
>           if block and timeout is None:
>               with self._rlock:
>                   res = self._recv_bytes()
>               self._sem.release()
>           else:
>               if block:
>                   deadline = time.time() + timeout
>               if not self._rlock.acquire(block, timeout):
>                   raise Empty
>               try:
>                   if block:
>                       timeout = deadline - time.time()
>                       if timeout < 0 or not self._poll(timeout):
>                           raise Empty
>                   elif not self._poll():
>                       raise Empty
>                   res = self._recv_bytes()
>                   self._sem.release()
>               finally:
>                   self._rlock.release()
>           # unserialize the data after having released the lock
>          return ForkingPickler.loads(res)
>   ```

    	def get_nowait(self):
            return self.get(False)
    ```
> 2. 如果多个进程同时向队列中放入对象，在另一端接收到的对象很可能是无序的。但是，由同一个进程放入队列中的对象始终按预期的顺序相互关联。

> 警告：如果一个进程正在使用一个队列的时候通过使用 `Process.terminate()` 或 `os.kill()` 终止了进程，队列中的数据很可能被损毁。这可能导致任何尝试使用该队列中数据的进程抛出异常。

> 警告：如上所述，子进程向队列中放入信息（且没有使用 `JoinableQueue.cancel_join_thread` 方法），进程在将数据缓冲到管道之前不会终止。
>
> 这意味着如果调用进程的 `join()` 方法在队列中仍有数据时可能造成死锁。类似地，若子进程是非后台进程，当父进程尝试调用 `join()` 方法等待它的非后台子进程终止时，父进程可能在退出时挂起。
>
> 通过使用 `manager` 创建的队列不会存在该问题。参与 [Programming guidelines](#3)。

有关使用队列在进程间通信的代码实例，可以查看 [Examples 部分](#4.3)。

multiprocessing.**Pipe**([duplex])

>返回一对代表 `connection` 对象通道两端的变量 `(conn1, conn2)`。
>
>若参数 *dumplex* 为 `True` 通道是双工的（默认为 `True`），若该参数为 `False` 则通道是单向的，`conn1` 只能用来接收消息，`conn2` 只能用来发送消息。

multiprocessing.**Queue**([maxsize])

> 返回



#### Miscellaneous ####

On the way!

#### Connection Objects ####

On the way!

#### Synchronization primitives ####

On the way!

#### Shared ctypes Objects ####

On the way!

#### Manager ####

`Manager` 提供了一种在不同进程间共享创建的数据的方式，包括在不同计算机运行的进程之间通过网络共享。`Manager` 对象管理一个共享对象的服务进程，其他进程通过使用代理获取共享对象。

Multiprocessing.**Manager()**

> 返回一个 `SyncManager` 对象，可以用来在进程间共享对象。返回的 `manager` 对象对应一个生成的子进程，该子进程会创建共享对象并返回相应的代理方法。

`manager` 进程一旦被垃圾回收或其父进程退出就会关闭。`manager` 类的定义位于 `multiprocessing.managers` 模块。

multiprocessing.managers.**BaseManager**([address[, authkey]])

> 创建 `BaseManager` 对象。
>
> 创建后应该调用 `start()` 或 `get_server().serve_forever()` 方法保证 `manager` 对象引用的进程开启。
>
> *address* 是 `manager` 进程监听的新连接的地址，若该参数为 `None` 则系统随机分配一个对应于某些空闲端口号的地址。
>
> *authkey* 身份认证密钥，用于检查连接到服务进程的有效性。若该参数为 `None` 则使用 `current_process().authkey`。若使用该参数，其类型是 byte 字符串类型。
>
> **start**([initializer[, initargs]])
>
> > `manager` 开启子进程。若构造函数不为空，启动时调用构造函数 `initializer(*initargs)`。
>
> **get_server**()
>
> > 返回 `manager` 对象控制下的真实服务器 `Server`，该类型支持 `serve_forever()` 方法。
> >
> > ```python
> > from multiprocessing.managers import BaseManager
> > manager = BaseManager(address=('', 50000), authkey=b'abc')
> > server = manager.get_server()
> > server.serve_forever()
> > ```
> >
> > `Server` 类型具有 `address` 属性，由 `manager` 对象使用的地址。
>
> **Connect**()
>
> > 连接一个本地的 `manager` 对象到远程 `manager` 进程。
> >
> > ```python
> > from multiprocessing.managers import BaseManager
> > m = BaseManager(address=('127.0.0.1', 5000), authkey=b'abc')
> > m.connect()
> > ```
>
> **shutdown**()
>
> > 终止 `manager` 进程。该方法只有在服务进程执行过 `start()` 方法后才有效。
> >
> > 该方法可以被多次调用。
>
> **register**(typeid[, callable[, proxytype[, exposed[, method_to_typeid[, create_method]]]]])
>
> > classmethod，用于注册类型或调用 `manager`。
> >
> > *typeid* 是类型标识符，用于识别共享对象的类型，字符串类型。
> >
> > *callable* 是用于创建该类型对象的调用。若 `manager` 实例使用 `connect()` 方法连接到服务器或设置 `create_method` 参数为 `False` 该参数可以设置为 `None`。
> >
> > *proxytype* 是 `BaseProxy` 的子类，用于为该类型的共享对象创建代理。若该参数为 `None`，则自动创建代理。
> >
> > *exposed* 用于指定允许使用 `BaseProxy.__callmethod()` 方法访问该类型共享对象的一系列方法的名字。（若该参数为 `None` 且 `proxytype._exposed_` 存在则使用后者替换前者。）若未指定，则共享的对象的所有公开方法均可以被访问。（这里公开方法指 `__call__()` 方法与其他不以 '_' 为名字开头的方法。）
> >
> > *method_to_typeid* 是指定了返回代理的公开方法应该返回的数据类型的映射。它将方法的名字映射到类型字符串。（若该参数为 `None` 且 `proxytype._method_to_typeid_` 存在使用后者替换前者。）若方法的名字在该参数中不存在或者该参数为 `None` 则方法返回的对象按值复制。
> >
> > *create_method* 决定是否构造方法用于通知服务进程创建共享对象并返回代理方法，默认为 `True`。
>
> `BaseManager` 实例有一个只读属性：
>
> **address**
>
> > 由 `manager` 使用的地址。
>
> 版本 3.3 后的变化：`manager` 对象支持上下文管理协议。

multiprocessing.managers.**SyncManager** 

> `BaseManager` 的子类，用于进程间同步。`multiprocessing.Manager()` 返回的对象类型。
>
> 同样支持创建共享 List 和 Dict。
>
> **Barrier**(parties[, action[, timeout]])
>
> > 创建共享 `threading.Barrier` 对象并返回其代理。3.3 版本后引入。
>
> **BoundedSemaphore**([value])
>
> > 创建共享 `threading.BoundedSemaphore` 对象并返回其代理。
>
> **Condition**([lock])
>
> > 创建共享 `threading.Condition` 对象并返回其代理。
> >
> > 若创建的共享对象类型包含 `lock`则返回值是 `threading.Lock` 或 `threading.RLock` 对象。
>
> 3.3 版本后添加了 `wait_for()` 方法。
>
> **Event**()
>
> > 创建共享 `threading.Event` 对象并返回其代理。
>
> **Lock**()
>
> > 创建共享 `threading.Lock` 对象并返回其代理。
>
> **Namespace**()
>
> > 创建共享 `Namespace` 对象并返回其代理。
>
> **Queue**([maxsize])
>
> > 创建共享 `queue.Queue` 对象并返回其代理。
>
> **RLock**()
>
> > 创建共享 `threading.RLock` 对象并返回其代理。
>
> **Semaphore**([value])
>
> > 创建共享 `threading.Semaphore` 对象并返回其代理。
>
> **Array**(typecode, sequence)
>
> > 创建一个 Array 并返回其代理。
>
> **Value**(typecode, value)
>
> > 创建带有可写属性 `value` 的对象并返回其代理。
>
> **dict**() & **dict**(mapping) & **dict**(sequence)
>
> > 创建共享 `dict` 对象并返回其代理。
>
> **list**() & **list**(sequence)
>
> > 创建共享 `list` 对象并返回其代理。
>
> > 注：对 dict 或 list 的代理中可变值或可变项的修改可能不会通过 `manager` 传播，因为代理无法知道其值或项在何时被修改的。因此，要修改此类值或项，需要将修改后的对象重新赋值给 `manager` 包含的代理。
> >
> > ```python
> > # create a list proxy and append a mutable object (a dictionary)
> > lproxy = manager.list()
> > lproxy.append({})
> > # now mutate the dictionary
> > d = lproxy[0]
> > d['a'] = 1
> > d['b'] = 2
> > # at this point, the changes to d are not yet synced, but by
> > # reassigning the dictionary, the proxy is notified of the change
> > lproxy[0] = d
> > ```
>
> multiprocessing.managers.**Namespace**
>
> > 可以注册 `SyncManager` 的类型。
> >
> > 一个 `Namespace` 对象没有公开方法，但有可写属性，其代表它属性的值。
> >
> > 然后，当使用 `Namespace` 对象的代理时，其以 '_' 开头名字的属性是代理的属性而不是引用对象的属性。
> >
> > ```python
> > manager = multiprocessing.Manager()
> > Global = manager.Namespace()
> > Global.x = 10
> > Global.y = 'hello'
> > Global._z = 12.3    # this is an attribute of the proxy
> > >>> print(Global)
> > Namespace(x=10, y='hello')
> > ```

##### 定制 manager #####

可以通过创建 `BaseManager` 的子类创建自己的定制化 `manager`，使用 `register()` 类方法注册新类型或调用该类。实例如下

```python
from multiprocessing.managers import BaseManager

class MathsClass:
    def add(self, x, y):
        return x + y
    def mul(self, x, y):
        return x * y

class MyManager(BaseManager):
    pass

MyManager.register('Maths', MathsClass)

if __name__ == '__main__':
    with MyManager() as manager:
        maths = manager.Maths()
        print(maths.add(4, 3))         # prints 7
        print(maths.mul(7, 8))         # prints 56
```

##### 使用远程 manger #####

可以在一台计算机上运行 `manager` 服务，另一台计算机上使用客户端使用该服务（假设防火墙允许）。

运行如下代码创建一个 `manager` 服务建立一个允许远程客户端访问的共享队列。

```python
from multiprocessing.managers import BaseManager
import queue
queue = queue.Queue()
class QueueManager(BaseManager): pass
QueueManager.register('get_queue', callable=lambda:queue)
m = QueueManager(address=('', 50000), authkey=b'abracadabra')
s = m.get_server()
s.serve_forever()
```

客户端获取服务实例代码如下

```python
from multiprocessing.managers import BaseManager
class QueueManager(BaseManager): pass
QueueManager.register('get_queue')
m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
m.connect()
queue = m.get_queue()
queue.put('hello')
```

另一个客户端可以获取上述客户端放入到队列中的信息，如下代码所示

```
from multiprocessing.managers import BaseManager
class QueueManager(BaseManager): pass
QueueManager.register('get_queue')
m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
m.connect()
queue = m.get_queue()
queue.get()
# output: 'hello'
```

本地也可以使用上述客户端的代码远程访问该队列

```python
from multiprocessing import Process, Queue
from multiprocessing.managers import BaseManager

class Worker(Process):
    
    def __init__(self, q):
        self.q = q
        super(Worker, self).__init__()
        
    def run(self):
        self.q.put('local hello')

queue = Queue()
w = Worker(queue)
w.start()

class QueueManager(BaseManager): 
    pass

QueueManager.register('get_queue', callable=lambda: queue)
m = QueueManager(address=('', 50000), authkey=b'abracadabra')
s = m.get_server()
s.serve_forever()
```

#### Proxy ####

On the way!

#### Process Pool ####

On the way!

#### Listener && Client ####

On the way!

#### Authentication keys ####

On the way!

#### Logging ####

提供了部分日志支持。但要注意，`logging` 模块不使用进程共享锁，所以来自不同进程的信息可能混淆（取决于处理程序类型）。

multiprocessing.**get_logger**()

> 返回一个 `multiprocessing` 模块使用的日志记录器。若有必要，将会新建一个。
>
> 首次创建的日志记录器日志级别为 `logging.NOTSET` 且没有默认处理程序。日志信息默认不会传递到根日志记录器。
>
> 注：Windows 环境下子进程只会继承父进程的日志级别而不会继承任何其他定义。

multiprocessing.**log_to_stderr**()

> 该方法返回一个添加了日志处理程序的 `get_logger()` 方法调用，该日志处理程序使用 `"[%(levelname)s/%(processName)s] %(message)s"` 格式将消息发送到 `sys.stderr`。

下面是打开日志记录的一个代码实例。

```python
import multiprocessing, logging
logger = multiprocessing.log_to_stderr()
logger.setLevel(logging.INFO)
logger.warning('doomed')
# [WARNING/MainProcess] doomed
m = multiprocessing.Manager()
# [INFO/SyncManager-...] child process calling self.run()
# [INFO/SyncManager-...] created temp directory /.../pymp-...
# [INFO/SyncManager-...] manager serving at '/.../listener-...'
del m
# [INFO/MainProcess] sending shutdown message to manager
# [INFO/SyncManager-...] manager exiting with exitcode 0
```

有关日志记录级别的详细信息，请参阅 [logging](https://docs.python.org/3.5/library/logging.html#module-logging)。

#### **multiprocessing.dummy** ####

该模块复制了 `multiprocessing` 的 API，仅仅是 `threading` 模块的修饰器。

### <span id="3">Programming Guidelines</span> ###

如下是使用 `multiprocessing` 模块时的一些指导原则和习惯用法。

#### Start() ####

如下原则适用于所有的 `start()` 方法。

1. 避免内存共享

> 尽可能的避免在进程之间传递大量数据。
>
> 进程间通信最好严格使用 `queue` 或 `pipe` 而不是使用级别较低的同步原语。

2. 序列化

> 保证代理方法的参数是可以序列化的。（可以被 pickle）

3. 代理是线程安全的

> 若代理对象来自多个线程，应该使用锁进行保护。（确保使用同一个代理的不同进程不会出现问题）

4. 僵尸进程的 `join()` 方法

> Unix 中，当一个子进程完成退出没有调用 `join()` 方法而父进程在继续运行时，子进程成为僵尸进程。因为每一个新进程 `start()` （或调用 `active_children()` 方法），所有没有调用 `join()` 方法的终止进程将会 `join()`，所以僵尸进程的数量不会特别多。同事，调用一个已终止进程的 `Process.is_alive` 将会调用 `join()` 方法。即使如此，开始时明确进程的执行流程也是一个好习惯。

> **个人解读**
>
> `join()` 方法具有清除僵尸进程的作用。通过下述源码我们可以发现，该方法通过子父进程的串行进程执行方式清除僵尸进程。`join()` 方法主要做了两件事情：① 通知父进程调用 `wait()` 方法；② 等待子进程终止或超时后，移除子进程。不带参数的 `join()` 方法
>
> 若调用带有参数的 `join(timeout)` 方法，在代码中父进程等待 `timeout` 时长后，开始唤醒父进程，此时子父进程开始同时执行，父进程首先执行`join(timeout)` 方法，若此时子进程还未结束，则变量 res 获取的进程退出信息为空，即不会清除子进程，然后继续执行父进程的后续逻辑，此时子父进程在并行执行。若子进程再次终止后父任务还未结束，则父进程因无法获取到子进程的退出信息而导致子进程沦为僵尸进程。
>
> ```python
> # CPython 3.5.2 /Lib/multiprocessing/process.py BaseProcess 类
> class BaseProcess(object):
>     '''
>     Process objects represent activity that is run in a separate process
>     The class is analogous to `threading.Thread`
>     '''
>     # ... ignore other methods
>     def _Popen(self):
>         raise NotImplementedError
>     def join(self, timeout=None):
>         '''
>         Wait until child process terminates
>         '''
>         assert self._parent_pid == os.getpid(), 'can only join a child process'
>         assert self._popen is not None, 'can only join a started process'
>         res = self._popen.wait(timeout)
>         if res is not None:
>             _children.discard(self)
> ```
>
> ```python
> # CPython 3.5.2 /Lib/multiprocessing/popen_fork.py Popen 类
> class Popen(object):
>     method = 'fork'
>     # ... ignore other methods
>     def poll(self, flag=os.WNOHANG):
>         if self.returncode is None:
>             while True:
>                 try:
>                     pid, sts = os.waitpid(self.pid, flag)
>                 except OSError as e:
>                     # Child process not yet created. See #1731717
>                     # e.errno == errno.ECHILD == 10
>                     return None
>                 else:
>                     break
>             if pid == self.pid:
>                 if os.WIFSIGNALED(sts):
>                     self.returncode = -os.WTERMSIG(sts)
>                 else:
>                     assert os.WIFEXITED(sts)
>                     self.returncode = os.WEXITSTATUS(sts)
>         return self.returncode
> 
>     def wait(self, timeout=None):
>         if self.returncode is None:
>             if timeout is not None:
>                 from multiprocessing.connection import wait
>                 if not wait([self.sentinel], timeout):
>                     return None
>             # This shouldn't block if wait() returned successfully.
>             return self.poll(os.WNOHANG if timeout == 0.0 else 0)
>         return self.returncode
> ```
>
> 消除僵尸进程的方法：
>
> * 创建两次子进程
>
> 即将父进程创建子进程的方式改为父进程先创建子进程代理进程，由代理进程创建子进程执行任务。代理进程在完成子进程的创建任务后，调用代理进程的 `join()` 方法消除（因为代理进程的执行时间很短）；同时，执行任务的子进程变成了孤儿进程，无论执行多久，最终被 init 进程（即进程号为 1 的进程）所收养，由 init 进程对其完成状态收集工作，避免了僵尸进程的产生。
>
> * 利用系统信号清除僵尸进程
>
> 基于 Linux 信号方式添加代码 `signal.signal(signal.SIGCHLD, signal.SIG_IGN)`。`signal.signal()` 函数定义在收到信号时执行自定义的处理程序。此处是指忽略创建的子进程的退出信号，
>
> `signal.SIGCHLD` 在子进程状态改变后会产生此信号。
>
> `sinal.SIG_IGN` 是标准信号处理程序，简单忽略给定信号；默认使用 `sinal.SIG_DFL`，代表不理会给定信号，但也不丢弃。修改后不保存子进程的状态使之成为孤儿进程被 init 进程回收。
>
> * 此外，父进程退出时创建的僵尸进程也会被清除。

5. 继承比使用 `pickle/unpickle` 更好

> 使用 `spawn` 或者 `forkserver` 方式调用 `start()` 方法时，`multiprocessing` 中的许多类型需要被序列化给子进程使用它们。但是，通常应该避免使用 `pipe` 或 `queue` 将共享对象发送给其他进程。应该合理安排程序，使得需要访问其他进程创建的共享资源的进程通过从祖先进程继承的方式去访问。

6. 避免主动终止进程

>使用 `Process.terminate` 方法可以终止进程，但可能导致进程当前使用的资源（如锁、信号量、管道和队列）被破坏或者不能被其他进程使用。因此，最好只有没有使用任何共享资源的进程上调用 `Process.terminate` 方法。

7. 使用 `queue` 通信的进程调用 `join()`

> 请记住，当一个进程调用 `join()` 方法时，Python 会检测被放入到 `queue` 中数据是否已经被全部删除（如 `queue.get()`，若没有被完全删除，进程将会一直等待，直到所有缓冲的数据由“feeder”线程将数据提供给底层管道。（子进程可以通过调用 `Queue.cancel_join_thread` 方法取消该行为。）这意味着，无论在什么时候使用 `queue` 都需要确保进程调用 `join()` 方法前放入到队列中的数据被全部清除。否则，无法确保将数据放入到队列中的进程已经终止。**切记，非后台进程将会自动调用 `join()` 方法。**
>
> 下面是一个会造成死锁的代码示例。
>
> ```python
> from multiprocessing import Process, Queue
> 
> def f(q):
>     q.put('X' * 1000000)
> 
> if __name__ == '__main__':
>     queue = Queue()
>     p = Process(target=f, args=(queue,))
>     p.start()
>     p.join()                    # this deadlocks
>     obj = queue.get()
> ```
>
> 一个改进办法是交换上述代码的最后两行（或简单删除 `p.join()` ）

8. 将资源明确的传递给子进程

>  在使用 `fork` 方式调用 `start()` 方法的 Unix 环境中，子进程可以使用父进程创建的全局资源中的所有共享资源，但最好将共享的对象作为参数传递给子进程的构造函数。
>
> 这样除了可以使得代码（可能）兼容 Windows 平台和其他调用 `start()` 方法的方式外，同时还可以确保共享的对象在子进程仍处于活动状态时不会在父进程被垃圾回收。如果父进程垃圾回收时释放某些共享资源，这种方式就非常重要。如下代码示例应该被改写为后者。
>
> ```python
> # 有问题的方式
> from multiprocessing import Process, Lock
> 
> def f():
>     ... do something using "lock" ...
> 
> if __name__ == '__main__':
>     lock = Lock()
>     for i in range(10):
>         Process(target=f).start()
> 
> # 更好的写法
> from multiprocessing import Process, Lock
> 
> def f(l):
>     ... do something using "l" ...
> 
> if __name__ == '__main__':
>     lock = Lock()
>     for i in range(10):
>         Process(target=f, args=(lock,)).start()
> ```

9. 用文件对象取代 `sys.stdin` 时要小心

> `multiprocessing` 模块最初是无约束的调用 `os.close(sys.stdin.fileno())`。
>
> 在 `multiprocessing.Process._bootstrap()` 方法中这样会导致子进程出现问题，修改如下
>
> ```python
> sys.stdin.close()
> sys.stdin = open(os.open(os.devnull, os.O_RDONLY), closefd=False)
> ```
>
> 这样解决了进程相互冲突导致的文件描述符错误的基本问题，但对于带有输出缓冲的文件对象取代 `sys.stdin()` 的应用程序引入了潜在危险。危险是：如果在多个进程中调用同一个文件对象的 `close()` 方法多次，可能导致相同的数据被多次刷新到文件对象中，从而导致了数据损毁。
>
> 如果需要自己实现文件对象及其缓存，则可以通过对缓存附加进程 PID 并在 PID 发生改变时丢弃缓存使其成为拷贝安全的。一个实现实例如下所示。
>
    ```python
    @property
    def cache(self):
        pid = os.getpid()
        if pid != self._pid:
            self._pid = pid
            self._cache = []
        return self._cache
    ```
> 更详细的信息可以阅读 [bpo-5155](https://bugs.python.org/issue5155)，[pbo-5313](https://bugs.python.org/issue5313) 和 [bpo-5331](https://bugs.python.org/issue5331)。

#### spawn方式 && forkserver方式 ####

此外还有些不适用于 `fork` 方式 `start()` 的其他约束。

1. 更多序列化要求

> 保证传给 `Process.__init__()` 方法的参数都是可以序列化的。同时也应该保证 `Process` 子类调用 `Process.start()` 方法时其实例也可以被序列化。

2. 全局变量

> 请记住，如果在子进程中尝试访问全局变量，则子进程看到的变量值（如果有）可能与父进程中调用 `Process.start()` 方法时的值不同。

3. `main` 模块的引用安全

确保 Python 解释器可以安全的导入主模块而不会导致意外发生（如启动一个新进程时）。例如，使用 `spawn` 或 `forkserver` 方式调用 `start()` 方法运行下面的代码实例会导致 `RuntimeError`。

```python
from multiprocessing import Process

def foo():
    print('hello')

p = Process(target=foo)
p.start()
```

应使用 `if __name__ == '__main__':` 来包含程序入口，如下代码所示。

```python
from multiprocessing import Process, freeze_support, set_start_method

def foo():
    print('hello')

if __name__ == '__main__':
    freeze_support()
    set_start_method('spawn')
    p = Process(target=foo)
    p.start()
```

（若程序实体正常运行而不需要冻结可以省略 `freeze_support()` 行代码。）这允许新生成的 Python 解释器可以安全导入模块，然后运行模块中 `foo()` 函数。在主模块中创建了 `Pool` 或 `Manager` 具有类似约束。

### Examples ###

#### manager ####

如下是一个创建使用 `manager` 及其代理的实例。

```python
from multiprocessing import freeze_support
from multiprocessing.managers import BaseManager, BaseProxy
import operator

##

class Foo:
    def f(self):
        print('you called Foo.f()')
    def g(self):
        print('you called Foo.g()')
    def _h(self):
        print('you called Foo._h()')

# A simple generator function
def baz():
    for i in range(10):
        yield i*i

# Proxy type for generator objects
class GeneratorProxy(BaseProxy):
    _exposed_ = ['__next__']
    def __iter__(self):
        return self
    def __next__(self):
        return self._callmethod('__next__')

# Function to return the operator module
def get_operator_module():
    return operator

##

class MyManager(BaseManager):
    pass

# register the Foo class; make `f()` and `g()` accessible via proxy
MyManager.register('Foo1', Foo)

# register the Foo class; make `g()` and `_h()` accessible via proxy
MyManager.register('Foo2', Foo, exposed=('g', '_h'))

# register the generator function baz; use `GeneratorProxy` to make proxies
MyManager.register('baz', baz, proxytype=GeneratorProxy)

# register get_operator_module(); make public functions accessible via proxy
MyManager.register('operator', get_operator_module)

##

def test():
    manager = MyManager()
    manager.start()

    print('-' * 20)

    f1 = manager.Foo1()
    f1.f()
    f1.g()
    assert not hasattr(f1, '_h')
    assert sorted(f1._exposed_) == sorted(['f', 'g'])

    print('-' * 20)

    f2 = manager.Foo2()
    f2.g()
    f2._h()
    assert not hasattr(f2, 'f')
    assert sorted(f2._exposed_) == sorted(['g', '_h'])

    print('-' * 20)

    it = manager.baz()
    for i in it:
        print('<%d>' % i, end=' ')
    print()

    print('-' * 20)

    op = manager.operator()
    print('op.add(23, 45) =', op.add(23, 45))
    print('op.pow(2, 94) =', op.pow(2, 94))
    print('op._exposed_ =', op._exposed_)

##

if __name__ == '__main__':
    freeze_support()
    test()
```

#### Pool ####

使用 ``Pool`` 的实例

```python
import multiprocessing
import time
import random
import sys

#
# Functions used by test code
#

def calculate(func, args):
    result = func(*args)
    return '%s says that %s%s = %s' % (
        multiprocessing.current_process().name,
        func.__name__, args, result
        )

def calculatestar(args):
    return calculate(*args)

def mul(a, b):
    time.sleep(0.5 * random.random())
    return a * b

def plus(a, b):
    time.sleep(0.5 * random.random())
    return a + b

def f(x):
    return 1.0 / (x - 5.0)

def pow3(x):
    return x ** 3

def noop(x):
    pass

#
# Test code
#

def test():
    PROCESSES = 4
    print('Creating pool with %d processes\n' % PROCESSES)

    with multiprocessing.Pool(PROCESSES) as pool:
        #
        # Tests
        #

        TASKS = [(mul, (i, 7)) for i in range(10)] + \
                [(plus, (i, 8)) for i in range(10)]

        results = [pool.apply_async(calculate, t) for t in TASKS]
        imap_it = pool.imap(calculatestar, TASKS)
        imap_unordered_it = pool.imap_unordered(calculatestar, TASKS)

        print('Ordered results using pool.apply_async():')
        for r in results:
            print('\t', r.get())
        print()

        print('Ordered results using pool.imap():')
        for x in imap_it:
            print('\t', x)
        print()

        print('Unordered results using pool.imap_unordered():')
        for x in imap_unordered_it:
            print('\t', x)
        print()

        print('Ordered results using pool.map() --- will block till complete:')
        for x in pool.map(calculatestar, TASKS):
            print('\t', x)
        print()

        #
        # Test error handling
        #

        print('Testing error handling:')

        try:
            print(pool.apply(f, (5,)))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from pool.apply()')
        else:
            raise AssertionError('expected ZeroDivisionError')

        try:
            print(pool.map(f, list(range(10))))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from pool.map()')
        else:
            raise AssertionError('expected ZeroDivisionError')

        try:
            print(list(pool.imap(f, list(range(10)))))
        except ZeroDivisionError:
            print('\tGot ZeroDivisionError as expected from list(pool.imap())')
        else:
            raise AssertionError('expected ZeroDivisionError')

        it = pool.imap(f, list(range(10)))
        for i in range(10):
            try:
                x = next(it)
            except ZeroDivisionError:
                if i == 5:
                    pass
            except StopIteration:
                break
            else:
                if i == 5:
                    raise AssertionError('expected ZeroDivisionError')

        assert i == 9
        print('\tGot ZeroDivisionError as expected from IMapIterator.next()')
        print()

        #
        # Testing timeouts
        #

        print('Testing ApplyResult.get() with timeout:', end=' ')
        res = pool.apply_async(calculate, TASKS[0])
        while 1:
            sys.stdout.flush()
            try:
                sys.stdout.write('\n\t%s' % res.get(0.02))
                break
            except multiprocessing.TimeoutError:
                sys.stdout.write('.')
        print()
        print()

        print('Testing IMapIterator.next() with timeout:', end=' ')
        it = pool.imap(calculatestar, TASKS)
        while 1:
            sys.stdout.flush()
            try:
                sys.stdout.write('\n\t%s' % it.next(0.02))
            except StopIteration:
                break
            except multiprocessing.TimeoutError:
                sys.stdout.write('.')
        print()
        print()


if __name__ == '__main__':
    multiprocessing.freeze_support()
    test()
```

#### <span id="4.3"> queue </span> ####

下面的实例介绍了如何使用 `queue` 将任务分配给多个任务进程并收集任务结果。

```python
import time
import random

from multiprocessing import Process, Queue, current_process, freeze_support

#
# Function run by worker processes
#

def worker(input, output):
    for func, args in iter(input.get, 'STOP'):
        result = calculate(func, args)
        output.put(result)

#
# Function used to calculate result
#

def calculate(func, args):
    result = func(*args)
    return '%s says that %s%s = %s' % \
        (current_process().name, func.__name__, args, result)

#
# Functions referenced by tasks
#

def mul(a, b):
    time.sleep(0.5*random.random())
    return a * b

def plus(a, b):
    time.sleep(0.5*random.random())
    return a + b

#
#
#

def test():
    NUMBER_OF_PROCESSES = 4
    TASKS1 = [(mul, (i, 7)) for i in range(20)]
    TASKS2 = [(plus, (i, 8)) for i in range(10)]

    # Create queues
    task_queue = Queue()
    done_queue = Queue()

    # Submit tasks
    for task in TASKS1:
        task_queue.put(task)

    # Start worker processes
    for i in range(NUMBER_OF_PROCESSES):
        Process(target=worker, args=(task_queue, done_queue)).start()

    # Get and print results
    print('Unordered results:')
    for i in range(len(TASKS1)):
        print('\t', done_queue.get())

    # Add more tasks using `put()`
    for task in TASKS2:
        task_queue.put(task)

    # Get and print some more results
    for i in range(len(TASKS2)):
        print('\t', done_queue.get())

    # Tell child processes to stop
    for i in range(NUMBER_OF_PROCESSES):
        task_queue.put('STOP')


if __name__ == '__main__':
    freeze_support()
    test()
```

### 后记 ###

On the way!

#### 注 ####

该篇博客参考文档为 Python 3.5.6 版本，源码版本为 CPython 3.5.2。
