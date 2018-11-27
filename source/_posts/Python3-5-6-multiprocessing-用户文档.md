title: Python3.5.6 multiprocessing 用户文档
categories:
  - Python
tags:
  - Python
  - documentation
  - Translation
date: 2018-11-16 11:44:02
---
最近在做 Python 内存优化时，发现 `multiprocessing` 可以充分利用 CPU 的多核特性，且可以在执行完任务后释放所有的资源，只保留需要的结果即可，可以避免 Python 的垃圾回收过慢的弊病，同时带来执行效率的提升。同时，之前的 Python 资料书籍更关注于 Python 多线程部分，而对多进程的讲解应用较少。于是开始阅读该部分文档和源码，本篇博客是其官网文档 `multiprocessing` 的翻译（[原文地址](https://docs.python.org/3.5/library/multiprocessing.html#)）。

 <!--more-->

### Introduction ###

`multiprocessing` 是一个支持产生新进程的 package，API 使用方式与 `threading` 模块相似。`multiprocessing` 可以提供本地/远程的并发，通过使用子进程而不是线程绕开 GIL 的约束（充分利用 CPU 多核提升效率）。因此，`multiprocessing` 模块可以支持程序充分利用给定计算机上的多个处理器。Unix/Windows 环境均可以运行。

`multiprocessing`同样引入了 `threading`中没有的 APIs。一个主要的例子如 `Pool`，提供了便捷的方法实现跨多个输入值的函数并行执行，可以跨进程分配输入的数据（数据并行）。下面的例子演示`Pool`在模块中定义这样一个函数的常见用法，以便子进程可以成功导入该模块，实现数据的并行性。

{% codeblock lang:python %}
from multiprocessing import Pool

def f(x):
    return x*x

if __name__ == '__main__':
    with Pool(5) as p:
        print(p.map(f, [1, 2, 3]))
{% endcodeblock %}
打印结果如下
{% codeblock lang:python %}
[1, 4, 9]
{% endcodeblock %}

#### Process 类 ####

`multiprocessing`通过创建一个 `Process` 对象调用其 `start()` 方法产生子进程。`Process` 的 API 与 `threading.Thread`一致，一个多进程的例子如下所示。
{% codeblock lang:python %}
from multiprocessing import Process

def f(name):
    print('hello', name)

if __name__ == '__main__':
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
{% endcodeblock %}

为了显示每个进程的独立 ID，一个扩展的程序实例如下。
{% codeblock lang:python %}
from multiprocessing import Process
import os

def info(title):
    print(title)
    prinnt('module name:', __name__)
    print('parent process:', os.getpid())
    print('process id:', os.getpid())

def f(name):
    info('function f')
    print('hello', name)

if __name__ == '__main__':
    info('main line')
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
{% endcodeblock %}

如果读者需要了解 `if __name__ == '__main__'` 为什么在这里是必须的，可以查看 [Programming Guidlines](#3)。

#### Context 与 start 方法 ####

依据运行环境的不同（Unix/Windows），`multiprocessing` 支持三种方式创建子进程，这三种方式是
* *spawn* 
父进程开启一个全新的 Python 解释器进程。子进程仅继承运行进程对象的 `run()` 方法所必须的资源。特别的是，不会从父进程中继承不需要的文件描述和文件句柄。与下面两种方法相比，此方法启动比较慢。
Unix 与 Windows 环境均存在该方法，Windows 下默认使用该方法。
* *fork*
父进程使用 `os.fork()` 复制 Python 解释器。子进程开始运行时实际是与父进程一样的。父进程的所有资源均会被子进程继承。注意拷贝一个多线程的进程可能会有问题。
仅 Unix 环境存在该方法，Unix 默认使用该方法。
* *forkserver*
当程序开始运行并选择了 `forkserver` 方法，将开启一个服务进程。从此时起，每当需要一个新进程时，父进程将连接到服务进程并请求它复制一个新进场。服务进程是单线程的因此使用 `os.fork()` 更安全。不必要的资源不会被继承。
仅支持 Unix 管道通信传输文件描述符的 Unix 平台支持该方法。

在 3.4 版本之后，所有的 Unix 环境支持了`spawn` 方法，部分 Unix 环境支持了 `forkserver`  方法，Windows 环境下子进程不再继承父进程的所有可继承的句柄。

Unix 环境下使用 `spawn` 或 `forkserver` 方法同样会开启一个信号量跟踪器进程跟踪由程序创建的未连接的命名信号量。当所有的进程退出时，信号量跟踪器取消链接任何剩余的信号量。一般情况下是没有的，但如果一个进程被信号量杀死，可能会有信号量泄漏。（没有链接的命名信号量是一个严重的问题，因为操作系统只允许有限数量的信号量且直至下次重启不会主动取消链接。）

{% note default %}
猜测：该方式只拷贝了父进程的信号量而没有拷贝文件描述符，因此是未连接的命名信号量，文件描述符或句柄由父进程的信号量连接。正常情况下，进程结束时，信号跟踪器会负责销毁该信号量，当意外退出或被 kill -9 杀死时，未能完成信号量的回收操作导致信号量泄漏。
{% endnote %}

可以在主函数的 `if __name__ == '__main__'` 从句中使用 `set_start_method()` 方法选择启动方法，如下实例。
{% codeblock lang:python %}
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
{% endcodeblock %}

程序中 `set_start_method()` 方法至多被使用一次，如像上述实例中，使用多次会报错

{% codeblock %}
RuntimeError: context has already been set.
{% endcodeblock %}

或者，可以使用 `get_context()` 获取一个 `context` 对象，`context` 对象作为 `multiprocessing` 模块的一员，具有和该模块同样的 API，并允许在同一程序中使用多次 `start()` 方法。
{% codeblock lang:python %}
    import multiprocessing as m  ctx = mp.get_context('spawn')
    q = ctx.Queue()
    p = ctx.Process(target=foo, args=(q,))
    p.start()
    print(q.get())
    p.join()
{% endcodeblock %}

注：与 `context` 相关的对象可能与不同 `context` 的进程不兼容。特别是使用 `fork()` 方法的 `context` 创建的锁不能传递给使用 `spawn` 或 `forkserver` 方法启动的进程。

使用特殊启动方式的库应该使用 `get_context()` 避免干扰库的使用者对启动方法的选择。

#### 进程间信息交换 ####

进程间 `multiprocessing` 支持两种类型的交流通道：`Queue` 和 `Pipe` 。

##### Queue ##### 
`Queue` 差不多克隆自 `queue.Queue`。代码实例如下
{% codeblock lang:python %}
from multiprocessing import Process, Queue

def f(q):
    q.put([42, None, 'hello'])

if __name__ == '__main__':
    q = Queue()
    p = Process(target=f, args=(q,))
    p.start()
    print(q.get())    # prints "[42, None, 'hello']"
    p.join()
{% endcodeblock %} `Queue` 是线程/进程安全的。

##### Pipes #####
`Pipe()` 函数返回一对由管道连接的连接对象（默认是双工的）。代码实例如下
{% codeblock lang:python %}
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
{% endcodeblock %} 两个连接对象代表了管道的两端。每一个连接对象都有 `send()` 和 `recv()` （彼此间）。

注：如果两个进程（或线程）同时读取或写入管道同一端的数据，则数据可能被损毁。当然，进程使用管道不同端不存在数据损坏的风险。

#### 进程间同步 ####
`multiprocessing` 模块包含 `threading` 所有同步原语的等价语句。例如我们可以使用 `lock` 保证同一时刻只有一个进程在进行标准输出。
{% codeblock lang:python %}
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
{% endcodeblock %} 如果不使用 `lock` ，来自不同进程的输出可能会混淆不清。

#### 进程间状态共享 ####
如上所述，在进行并发编程时，尽量不使用状态共享，多线程编程时尤其如此。
然而，如果你确实需要共享数据，`multiprocessing` 提供了两种方式实现。

##### 内存共享 #####
数据可以以 `Value` 或者 `Array` 的方式在内存中共享。代码实例如下
{% codeblock lang:python %}
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
{% endcodeblock %}

打印结果如下
{% codeblock lang:python %}
3.1415927
[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
{% endcodeblock %}

创建变量 `num` 和 `arr` 时使用的参数 'd' 和 'i' 指定数据类型的代码：'d' 表示双精度浮点型，'i' 表示有符号整型。共享对象是进程/线程安全的。

为了更灵活地使用内存共享，可以使用 `multiprocessing.sharedctypes` 模块，该模块支持从共享内存分配出来的任意 [ctypes](https://docs.python.org/3.5/library/ctypes.html) 对象。

##### 服务进程 #####
`Manager()` 返回的 `manager` 对象管理一个服务进程持有 Python 对象且允许其他进程使用代理操作这些对象，支持 `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` 和 `Array`。代码示例如下
{% codeblock lang:python %}
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
{% endcodeblock %}

打印结果如下
{% codeblock lang:python %}
{0.25: None, 1: '1', '2': 2}
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
{% endcodeblock %}

使用服务进程 `manager` 比使用内存共享更灵活，因为 `manager` 可以支持任意对象类型。此外，单个 `manager` 可以通过网络在不同计算机上实现进程共享。**但是**，`manager` 方式比使用内存共享慢。

#### 使用 Pool workers ####
`Pool` 对象表示一个进程池，其允许以几种不同的方式将任务装载到进程池。代码示例如下

{% codeblock lang:python %}
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
        for i in pool.imap_unordered(f, range((10)):
            print(i)
        # evaluate "f(20)" asynchroously
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
{% endcodeblock %}
注：`Pool` 的方法只能由创建它的进程使用。
{% blockquote %}
Tips: 该包中的功能要求 `__main__` 模块可由子项导入。在 Programming Guidelines 中有所涉及，但值得在此指出。这意味着某些例子，如下 `multiprocessing.pool.Pool` 的实例在交互式解释器中不起作用。
{% codeblock lang:python %}
​from multiprocessing import Pool
    
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
{% endcodeblock %}如果你尝试运行该代码，会以半随机的方式输出三个完整的回溯，然后不能不以某种方式停止主进程。
{% endblockquote %}

### Reference ###
`multiprocessing` 模块主要复制了 `threading` 模块的 API。

#### Process && Exceptions ####
multiprocessing.**Process**(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)

{% blockquote %}
`Process` 对象表示另一个进程中的活动。`Process` 类具有 `threading.Thread` 的所有方法的等价方法。

应该始终使用关键字参数调用构造函数。
参数 *group* 应该始终是 `None`，它仅为了与 `threading.Thread` 构造参数一致。
参数 *target* 是 `run()` 方法的调用函数，默认为 `None`，即不调用任何内容。
参数 *name* 是进程的名字，详细情况见下参数部分。
参数 *kwargs* 是目标函数调用的参数关键字字典，若提供了该参数，将关键字参数 `daemon` 设置为 `True/False`，若为默认值 `None`，则从创建该进程的进程继承该参数。
默认情况下，不会将任何参数传递给 *target*。

如果子类重写构造函数，必须确保在对进程执行任何动作之前先调用 `Process.__init__()` 方法。

版本 3.3 后添加了 `daemon` 参数。

{% note info %}
构造函数源码如下所示。
{% codeblock lang:python %}
# cpython 3.5.2 /Lib/multiprocessing/process BaseProcess 类
def __init__(self, group=None, target=None, name=None, args=(), kwargs={},
             *, daemon=None):
​    assert group is None, 'group argument must be None for now'
​    count = next(_process_counter)
​    self._identity = _current_process._identity + (count,)
​    self._config = _current_process._config.copy()
​    self._parent_pid = os.getpid()
​    self._popen = None
​    self._target = target
​    self._args = tuple(args)
​    self._kwargs = dict(kwargs)
​    self._name = name or type(self).__name__ + '-' + \
                 ':'.join(str(i) for i in self._identity)
​    if daemon is not None:
​        self.daemon = daemon
​    _dangling.add(self)
{% endcodeblock %} 通过该函数我们可以清晰了解到进程的命名规则，如下有详细介绍。
{% endnote %}

**run**()
表示进程动作（即进程的执行任务）的方法。
分别按顺序使用 *args* 和 *kwargs* 参数中的关键字参数。

**start**()
进程启动的方法。
该方法在每一个进程中至多被调用一次，其在一个单独进程中调用 `Process` 对象的 `run()` 方法。

**join**([*timeout*])
若可选参数 *timeout* 为 `None` （默认值），该方法会阻塞进程直至调用该方法的进程终止；
若可选参数 *timeout* 为整数 N，该方法会阻塞进程 N 秒；
注：若方法进程终止或超时，返回 `None`。可以通过检查进程的 `exitcode` 属性值确认进程是否终止。

**name**
进程名称。`name` 是字符串类型，仅用于识别目的，没有语义。多个进程可以使用相同的名字。
初识名字由构造器设置，若构造器没有提供显示名字，按照 "Process-{N<sub>1</sub>:N<sub>2</sub>:...:N<sub>k</sub>}" 形式构造名字，其中 "N<sub>k</sub>" 表示父进程的第 N<sub>k</sub> 个子进程。

**is_alive**()
返回进程是否活着。
粗略的说，从进程调用 `start()` 方法到进程终止之间进程状态时活着的。

**daemon**
进程守护进程运行标志，Boolean 类型。必须在调用 `start()` 方法之前设置 - 这样可以变为守护进程。
初始值继承自父进程（即创建进程）。
当进程退出时，它会尝试其终止其守护的所有子进程。
{% note info %}
注：守护进程不允许创建子进程。否则一个后台进程退出时会终止其子进程。当然，这些子进程不是 Unix 守护进程或服务，它们是可以正常终止的进程（不调用 `join()` 方法阻塞等待）。
{% endnote %}
{% endblockquote %}

除了 `threading.Thread` API 之外，`Process` 还支持如下属性或方法
{% blockquote %}
**pid**
返回进程 ID，进程生成之前返回 `None`。

**exitcode**
子进程退出码。
`None` 表示进程尚未终止；返回负整数 `-N` 表示进程被信号 `N` 终止；返回0表示没有错误正常退出；返回正数表示进程有错误，并以错误码为状态码退出。

 **authkey**
程序的身份认证秘钥是一个 byte 类型的字符串。
`multiprocessing` 初始化时，会使用 `os.urandom()` 方法为主进程分配一个随机字符串。
当创建 `Process` 对象时，它会继承父进程的 `authkey`，可以通过设置 `authkey` 来改变它。

**sentinel**
系统对象的数字句柄，当进程结束时状态变为 ready。
一次等待多个事件时可以使用 `multiprocessing.connection.wait()`，否则使用 `join()` 方法会更简单。
 Windows 环境下这是一个系统句柄调用 `WaitForSingleObject` 和 `WaitForMultipleObjects` API 族；Unix 环境下是一个文件描述符，与 `select` 模块中的原语配合使用。

**terminate**()
终止进程。Unix 环境下使用 `SIGTERM` 信号量完成；Windows 环境下调用 `TerminatedProcess()` 方法实现。但不会执行退出句柄和剩余子句。
注：进程的后裔进程不会被终止，后裔进程将称为孤儿进程。
{% note warning %} 警告：如果相互关联的进程正在使用管道或队列时，调用了该方法，管道或队列可能会被损毁且无法被其他进程使用。类似的，如果进程已获得了锁或者信号量，终止它可能导致其他进程的死锁。{% endnote %}
{% endblockquote %}

注：`start()`，`join()`，`is_alive()`，`terminate()` 和 `exitcode` 这些方法应该仅由父进程对象调用。

如下是一个 `Process` 方法的使用实例。

{% codeblock lang:python %}
import multiprocessing, time, signal
p = multiprocessing.Process(target=time.sleep, args=(1000,))
print(p, p.is_alive())
# <Process(Process-1, initial)> False
p.start()
print(p, p.is_alive())
# <Process(Process-1, started)> True
p.terminate()
time.sleep(0.1)
print(p, p.is_alive())
# <Process(Process-1, stopped[SIGTERM])> False
p.exitcode == -signal.SIGTERM
# True
{% endcodeblock %}

文档中这里介绍了 `multiprocessing` 中四种异常。
{% blockquote %}
*exception* multiprocessing.**ProcessError**
`multiprocessing` 异常类型的基类。

*exception* multiprocessing.**BufferTooShort**
当提供的缓存对读取的消息来说太小时，由 ``Connection.recv_bytes_into()`` 方法抛出的异常。

*exception* multiprocessing.**AuthenticationError**
认证失败抛出的异常。

*exception* multiprocessing.**TimeoutError**
超时抛出的异常。
{% endblockquote %}

#### Pipes && Queues ####
通常多进程之间通信应该使用消息传递，尽量避免使用任何同步原语（如锁）。

消息传递可以使用 `Pipe()` （两个进程间通信）或 `queue`（多个生产者和消费者之间通信）。
`Queue`、`SimpleQueue` 和 `JoinableQueue` 类型是基于标准库中 `queue.Queue` 类的基础上构建的多生产者，多消费者 FIFO 队列。不同之处在于 `Queue` 与 `queue.Queue` 相比缺少 `task_done()` 和 `join()` 方法（该方法在 Python 2.5 之后的版本引入）。

如果使用 `JoinableQueue` 则一定要为从队列中删除的每个任务调用 `JoinableQueue.task_done()`，否则可能导致用于统计未完成任务数量的信号量溢出，从而引发异常。

注：也可以通过 `Manager` 对象创建一个共享队列。

{% note info %}
Tip：`multiprocessing` 使用 `queue.Empty` 和 `queue.Full` 异常发出超时信号。这些不能在 `multiprocessing` 的命名空间中找到，因此需要从 `queue` 中引用进来。
{% endnote %}

{% note info %}
Tip：当一个对象被放入队列时，对象是被序列化了的，之后后台线程会把序列化之后的数据缓冲到一个底层管道。这可能是一个令人惊讶的结果，但不会造成任何困难。如果这个操作会影响到你的任务，可以使用  `manager`。
1. 将一个对象放入到空队列后，可能有一个无限小的延迟队列的 `empty()` 方法返回 `True` 。`get_nowait()` 方法可以在不抛出 `queue.Empty` 异常的情况下返回队列状态。
{% codeblock lang:python %}
# CPython 3.5.2 /Lib/multiprocessing/queues.py
#
# Queue type using a pipe, buffer and thread
#
class Queue(object):
    # ... ignore other methods
    def get(self, block=True, timeout=None):
        if block and timeout is None:
            with self._rlock:
                res = self._recv_bytes()
                self._sem.release()
        else:
            if block:
                deadline = time.time() + timeout
            if not self._rlock.acquire(block, timeout):
                raise Empty
            try:
                if block:
                    timeout = deadline - time.time()
                    if timeout < 0 or not self._poll(timeout):
                        raise Empty
                elif not self._poll():
                    raise Empty
                res = self._recv_bytes()
                self._sem.release()
            finally:
                self._rlock.release()
        # unserialize the data after having released the lock
        return ForkingPickler.loads(res)

    def get_nowait(self):
        return self.get(False)
{% endcodeblock %}

2. 如果多个进程同时向队列中放入对象，在另一端接收到的对象很可能是无序的。但是，由同一个进程放入队列中的对象始终按预期的顺序相互关联。
{% endnote %}

{% note warning %}
警告：如果一个进程正在使用一个队列的时候通过使用 `Process.terminate()` 或 `os.kill()` 终止了进程，队列中的数据很可能被损毁。这可能导致任何尝试使用该队列中数据的进程抛出异常。
{% endnote %}

{% note warning %}
警告：如上所述，子进程向队列中放入信息（且没有使用 `JoinableQueue.cancel_join_thread` 方法），进程在将数据缓冲到管道之前不会终止。
{% endnote %}

这意味着如果调用进程的 `join()` 方法在队列中仍有数据时可能造成死锁。类似地，若子进程是非后台进程，当父进程尝试调用 `join()` 方法等待它的非后台子进程终止时，父进程可能在退出时挂起。

通过使用 `manager` 创建的队列不会存在该问题。参与 [Programming guidelines](#3)。

有关使用队列在进程间通信的代码实例，可以查看 [Examples 部分](#4.3)。

multiprocessing.**Pipe**([*duplex*])
{% blockquote %}
返回一对代表 `connection` 对象通道两端的变量 `(conn1, conn2)`。
若参数 *dumplex* 为 `True` 通道是双工的（默认为 `True`），若该参数为 `False` 则通道是单向的，`conn1` 只能用来接收消息，`conn2` 只能用来发送消息。
{% endblockquote %}

multiprocessing.**Queue**([*maxsize*])
{% blockquote %}
返回一个进程，其使用管道和一些锁/信号量实现了共享队列。当进程首次将数据放入队列时，会启动将数据对象从缓存传输到队列的进给线程。
标准库 `queue` 模块中的 `queue.Empty` 和 `queue.Full` 异常会引发超时信号。
`Queue` 实现了 `queue.Queue` 中除 `task_done()` 和 `join()` 外的所有方法。

**qsize**()
返回队列的近似大小。因为多线程/多进程影响，该数字是不可靠的。
注：该方法在像 MacOSX 这样未实现 `sem_getvalue()` 的 Unix 平台可能引发 `NotImplementedError` 异常。

**empty**()
如果队列为空返回 `True` 否则返回 `False`。由于多线程/多进程影响，返回值不可靠。

**full**()
如果队满返回 `True` 否则返回 `False`。由于多线程/多进程影响，返回值不可靠。

**put**(*obj*[, *block*[, *timeout*]])
将对象放入队列。
若可选参数 *block* 是 `True`（默认值）且 *timeout* 参数为 `None`（默认值），若没有空间插入对象会发生阻塞直到可以插入对象 。若 *timeout* 参数是一个正整数 N，其会至多阻塞 N 秒还没有空间可以插入时抛出 `queue.Full`
异常。
若可选参数 *block* 是 `False` 会直接向队列中放入元素，若没有空间直接抛出 `queue.Full` 异常（该情况下 *timeout* 参数被忽略）。

**put_nowait**(*obj*)
等价于 `put(obj, False)` 方法。

**get**([*block*[, *timeout*]])
从队列中删除并返回一个数据对象。

**get_nowait**()
等价于 `get(False)` 方法。

`multiprocessing.Queue` 还有一些 `queue.Queue` 中没有的方法。这些方法在大多数情况下用不到。

**close**()
表示当前进程不会再将数据放入该队列。一旦后台线程将所有的数据缓冲到管道中，后台线程退出。当 `Queue` 实例被垃圾回收时自动调用该方法。

**join_thread**()
`join` 后台线程。该方法只能在调用 `close()` 方法后使用，将阻塞至后台线程的退出以确保缓存的所有数据均缓冲到管道中。
默认情况下，非 `Queue` 创建者的进程退出时会调用该方法，可以通过调用 `cancel_join_thread()` 使该操作失效。

**cancel_join_thread**()
阻止 `join_thread()` 方法阻塞。特别的是，可以阻止后台线程在进程中退出时进程对线程的 `join` 操作 - 参考上面 `join_thread()` 方法。
`allow_exit_without_flush()` 的名字更适合该方法，其可能会导致进入队列中的数据消失，因此基本不需要使用该方法。只有在当前进程需要立即退出，并不关心丢失的数据，不再等待后台线程将缓存数据缓冲到底层管道的情况下，使用该方法。
{% note info %}
注：该类的方法依赖于主机操作系统支持共享信号量功能。若不支持，该类中的方法将被禁用，尝试实例化 `Queue` 将会导致 `ImportError` 异常。参阅 [bpo-3770](https://bugs.python.org/issue3770) 获取更详细的信息。对下面列出的其他特殊 `Queue` 类型具有同样的要求。
{% endnote %}
{% endblockquote %}

multiprocessing.**SimpleQueue**
{% blockquote %}
类似于加锁 `Pipe` 类型的 `Queue` 简单实现。

**empty**()
若队列为空返回 `True` 否则返回 `False`。

**get**()
从队列中移除并返回数据对象。

**put**(*item*)
将数据对象放入队列。

{% endblockquote %}

multiprocessing.**JoinableQueue**([*maxsize*])
{% blockquote %}
`Queue` 类型的子类，添加了 `task_done()` 和 `join()` 方法。

**task_done**()
表明之前排队过的任务已经完成。由队列的消费者使用。对于每个使用 `get()` 方法获取任务的消费者后续调用该方法告知队列任务完成。
若调用 `join()` 方法，会阻塞至队列中的所有任务均被完成（即调用 `put()` 方法放入队列中的每个任务都收到了 `task_done` 回调）。
若该方法的调用次数超过了队列中放置的任务数量，会引发 `ValueError` 异常。

**join**()
阻塞至放入到队列中的所有任务均被处理完成。
每当有新的任务加入到队列中，未完成的任务数量就会增加。每当消费者调用了 `task_done()` 告知队列任务完成，未完成的任务数量就会下降。当未完成任务数量的计数将至零时，该方法的阻塞不再阻塞。
{% endblockquote %}

#### Miscellaneous ####
multiprocessing.**active_children**()
返回当前进程活跃的子进程列表。
使用该函数会对已经结束的子进程调用 `poll()` 方法的副作用。
{% codeblock lang:python %}
# CPython 3.5.2 /Lib/multiprocessing/queues.py
def active_children():
    children = current_process()._children
    for p in list(children):
        if not p.is_alive():
            children.pop(p, None)
    return list(children)
{% endcodeblock %}

multiprocessing.**cpu_count**()
返回系统的 CPU 数量，可能抛出 `NotImplementedError`。
{% note info %} 
可同时参阅 [`os.cpu_count()`](https://docs.python.org/3.5/library/os.html#os.cpu_count) 
{% endnote %}

multiprocessing.**current_process**()
返回与当前进程一致的进程对象。
与 `threading.current_thread()` 类似。

multiprocessing.**freeze_support**()
添加了冻结程序的多进程的支持以产生 Windows 可执行文件。（已使用 `py2exe`，`PyInstaller` 和 `cx_Freeze` 进行过测试。）
需要在主函数的 `if __name__ == '__main__':` 代码后调用该函数，实例如下
{% codeblock lang:python %}
from multiprocessing import Process, freeze_support

def f():
    print('hello world!')

if __name__ == '__main__':
    freeze_support()
    Process(target=f).start()
{% endcodeblock %} 若省略了 `freeze_support()` 行代码去执行冻结的可执行文件会抛出 `RuntimeError` 异常。
在 Windows 系统外任何其他系统上运行该函数无效；若 Windows 环境下 Python 解释器正常运行模块（程序没有被冻结），该函数无效。

multiprocessing.**get_all_start_methods**()
返回所有支持的 `start()` 方式列表，第一个是默认方式。可能的方式有 `fork`、`spawn` 和 `forkserver`。Windows 系统下只支持 `spawn` 方式。Unix 系统下支持 `fork` 和 `spawn` 方式，默认是 `fork` 方式。
版本 3.4 后添加。

multiprocessing.**get_context**(*method=None*)
返回与 `multiprocessing` 模块相同属性的 `context` 对象。
若参数 `method` 为 `None`，返回默认 `context`；否则参数 `method` 应该为 `fork`，`spawn`，`forkserver`。若 `start()` 的方式系统不支持会抛出 `ValueError` 异常。
版本 3.4 后添加。

multiprocessing.**get_start_method**(*allow_none=False*)
返回启动进程方式的名称。
若 `start()` 方式未确定且参数 *allow_none* 是 `False`，`start()` 启动方式被设置为默认值并返回默认方式的名称。若 `start()` 方式没有被确定且参数 *allow_none* 是 `True` 返回 `None`。
返回值可以是 `fork`，`spawn`，`forkserver` 或 `None`。Unix 默认 `fork`，Windows 默认 `spawn`。
版本 3.4 后添加。

multiprocessing.**set_executable**()
设置启动子进程时 Python 解释器路径。（默认使用 `sys.executable`）。嵌入式在子进程创建之前，很可能需要做类似于如下的事情。
{% codeblock lang:python %}
set_executable(os.path.join(sys.exec_prefix, 'pythonw.ext'))
{% endcodeblock %} 版本 3.4 之后 Unix 支持使用 `spawn` 方式启动子进程。

multiprocessing.**set_start_method**(*method*)
设置子进程的 `start()` 方式，可以是 `fork`，`spawn` 或者 `forkserver`。
注意，该函数至多被调用一次，且应该在主模块的 `if __name__ == '__main__':` 子句中进行保护。
版本 3.4 后添加。

{% note info %}
注：`multiprocessing` 不包含如下类似方法 `threading.active_count()`、`threading.enumerate()`、`threading.settrace()`、`threading.setprofile()`、`threading.Timer` 和 `threading.local`。
{% endnote %}

#### Connection Objects ####
`Connection` 对象可以发送和接收序列化的对象或字符串，它们可以被看做是面向消息的套接字。
`Connection` 对象一般使用 `Pipe()` 方法创建 - 参阅 [Listeners && Clients](#2.10)

*class* multiprocessing.**Connection**
{% blockquote %}
**send**(*obj*)
向连接的另一端发送一个对象，另一端应该使用 `recv()` 接收。
传输的对象一定可以被序列化。过大的序列化对象可能抛出 `ValueError` 异常（大约 32 MB+，取决于操作系统）。

**recv**()
返回由连接另一端使用 `send()` 方法发送来的对象。如果没有对象传输过来调用该方法会产生阻塞直至接收到对象。若没有任何对象发送给该端接收而另一发送端关闭，会抛出 `EOFError` 异常。

**fileno**()
返回连接使用的文件描述符或句柄。

**close**()
关闭连接。连接被垃圾回收时被自动调用。

**poll**([*timeout*])
返回是否有可以被读取的数据。
若 *timeout* 没有指定，则立即返回；若 *timeout* 是数字 N，则指定阻塞的时间最长为 N 秒；若 *timeout* 为 `None`，则阻塞的时间为无限长。
注：可以使用 `multiprocessing.connection.wait()` 方法一次轮训多个连接对象。

**send_bytes**(*buffer*[, *offset*[, *size*])
将字节类型对象作为完整消息发送。
若设置了 *offset* 参数将会缓存中该偏移位置读取数据。
若设置了 *size* 参数将会从缓存中读取多个字节。
太大的缓存区（约为 32MB+，取决于操作系统）可能抛出 `ValueError` 异常。

**recv_bytes**([*maxlength*])
将从连接另一端发送的字节类型的数据对象作为字符串返回。阻塞至有数据对象收到。若没有剩余可接收数据对象且连接发送端已经关闭会抛出 `EOFError` 异常。
若指定了 *maxlength* 参数且消息对象的长度大于该参数数值，将抛出 `OSError` 异常，且该连接不再可读。
版本 3.3 后变更：该方法引发的 `IOError` 异常是 `OSError` 异常的别称。

**recv_bytes_into**(*buffer*[, *offset*])
读入从连接发送端发送的字节类型数据对象完整消息并返回数据对象的字节数。阻塞至有数据对象收到。若没有剩余可接收数据对象且连接发送端已经关闭会抛出 `EOFError` 异常。
参数 *buffer* 必须是可写的字节类型对象。若给定 *offset* 参数，将会从缓存中该偏移位置写入数据，该参数是小于 *buffer* 长度的非负整数。
若缓冲区太小会引发 `BufferTooShort` 异常，且完整的消息可以从异常实例 `e.args[0]` 获取。
{% endblockquote %}

版本 3.3 后改动：`Connection` 对象可以在进程间使用 `Connection.send()` 和 `Connection.recv()` 传输。
版本 3.3 后添加：`Connection` 对象支持上下文管理协议。

Python 交互界面中，示例如下
{% codeblock lang:python %}
from multiprocessing import Pipe
a, b = Pipe()
a.send([1, 'hello', None])
b.recv()
# [1, 'hello', None]
b.send_bytes(b'thank you')
a.recv_bytes()
# b'thank you'
import array
arr1 = array.array('i', range(5))
arr2 = array.array('i', [0] * 10)
a.send_bytes(arr1)
count = b.recv_bytes_into(arr2)
assert count == len(arr1) * arr1.itemsize
arr2
# array('i', [0, 1, 2, 3, 4, 0, 0, 0, 0, 0])
{% endcodeblock %} {% note warning %}
警告：`connection.recv()` 方法会自动反序列化接收的数据对象，这可能带来安全风险，所以要确认发送消息的进程是值得信任的。因此，除使用 `Pipe()` 生成的连接对象，均应该进行身份验证后再使用 `recv()` 和 `send()` 方法。参与 [Authentication keys](#2.11)。{% endnote %}{% note warning %}
警告：杀死正在从管道中读取数据或向管道中写入数据的进程可能导致管道中的数据被破坏，因为无法确定数据边界。{% endnote %}

#### Synchronization primitives ####
通常，同步原语在多进程程序中不像多线程程序中那么必要。参阅 [`threading`](https://docs.python.org/3.5/library/threading.html#module-threading) 模块文档。
可以通过是使用 `manager` 对象创建同步原语 - 参考 [Manager](#2.7)。

*class* multiprocessing.**Barrier**(*parties*[, *action*[, *timeout*]])
克隆自 `threading.Barrier` 的屏障对象。
版本 3.3 后添加。

*class* multiprocessing.**BoundedSemaphore**([*value*])
类似于 `threading.BoundedSemaphore` 的屏障信号量。
与类似对象 `threading.BoundedSemaphore` 的细微不同：其 `acquire` 方法第一个参数是与 `Lock.acquire()` 方法一致的 `block`。
{% note info %}
MacOSX 系统上该对象与 `Semaphore` 无法区分，因为该平台未实现 `sem_getvalue()` 方法。
{% endnote %}

*class* multiprocessing.**Condition**([*lock*])
类似于 `threading.Condition` 的条件变量。
若指定了 *lock* 参数，参数应该是来自 `multiprocessing` 模块的 `Lock` 或 `RLock` 对象实例。
版本 3.3 后添加了 `wait_for()` 方法。

*class* multiprocessing.**Event**
克隆自 `threading.Event`。

*class* multiprocessing.**Lock**
{% blockquote %}
类似于 `threading.Lock` 的非递归锁。一旦进程/线程获得了锁，后续其他进程/线程尝试获取锁时会阻塞至锁被释放。任何进程/线程均可以释放它。除非另有说明，`threading.Lock` 对于线程的概念和行为同样适用于 `multiprocessing.Lock` 对于进程/线程。
`Lock` 对象支持上下文管理协议，因此可以与 `with` 语句一起使用。

**acquire**(*block=True, timeout=None*)
获取一个阻塞/非阻塞的锁。
*block* 参数被设置为 `True`（默认值）该方法会阻塞至锁处于解锁状态，将其设置为上锁状态返回 `True`。注意此处第一个参数的名字与 `threading.Lock.acquire()` 方法中的第一个参数名字不同。
若 *block* 参数设置为 `False`，该方法不会阻塞。若锁被上锁，返回 `Fasle`，否则将锁上锁返回 `True`。
当使用正的浮点型数值 F 设置 *timeout* 参数时，其阻塞至获得锁的最长等待时间 F 秒。F 为负数时等效于为 0 的调用。若该参数设置为 `None`（默认值），即阻塞时间没有上限。注意此处对超时的负值或 `None` 的表现形式已与 `threading.Lock.acquire()` 不同。若 *block* 参数被设置为 `False` 该参数因没有实际意义将被忽略。若获得锁返回 `True` 否则阻塞时间超时时返回 `False`。

**release**()
解锁。该方法不仅可以由当前占有锁的进程/线程调用，也可以由其他进程/线程调用。
除在未上锁的锁上调用该方法引发 `ValueError` 异常外，其行为与 `threading.Lock.release()` 一致。
{% endblockquote %}

*class* multiprocessing.RLock
{% blockquote %}
类似于 `threading.RLock` 的递归锁。递归锁必须由获取它的进程/线程释放它。一旦进程/线程已经获取了它，同一进程/线程可以不阻塞的再次获得它；该进程/线程每次获取它均需要释放。
注意 `RLock` 实际上是一个工厂函数，其返回使用默认上下文初始化的 `multiprocessing.synchronize.RLock` 实例。
`RLock` 对象支持上下文管理协议，因此可以与 `with` 语句一起使用。

**acquire**(*block=True, timeout=None*)
获取一个阻塞/非阻塞的锁。
*block* 参数被设置为 `True`（默认值）该方法会阻塞至锁处于解锁状态，除非锁被当前进程/线程所有。当前进程/线程获得了锁的所有权（若之前还没有所有权），锁的内部递归级别加一并返回 `True`。注意，与 `threading.RLock.acquire()` 的实现相比，第一个参数名称不同，这里是函数本身。
参数 *block* 设置为 `False` 时不阻塞。若锁已经被另一个进程/线程获取（即拥有），则当前进程/线程不会获得所有权，锁的内部递归级别也不会更改，返回 `False`。若锁处于解锁状态，则当前进程/线程获取锁的所有权且锁的内部递归级别加一，返回 `True`。
*timeout* 参数的用法和行为与 `Lock.acquire()` 相同。注意，这里的超时行为与 `threading.RLock.acquire()` 的实际表现略有不同。

**release**()
释放锁并将内部递归级别减一。在锁的递归级别变为零后，解锁（锁不为任何进程/线程拥有）。若有其他进程/线程在阻塞等待该锁，仅允许其中一个获得锁的所有权。若递归级别不为零，则锁保持上锁状态，并由当前调用进程/线程拥有。
仅在锁的拥有者的进程/线程调用该方法。由锁的非拥有者调用该方法或锁处于非锁定状态（无主状态）时调用该方法会引发 `AssertionError` 异常。注意，该情况下抛出的异常类型与 `threading.RLock.release()` 实现的行为不同。
{% endblockquote %}

*class* multiprocessing.Semaphore([*value*])
类似于 `threading.Semaphore` 的信号量对象。
差异在于其 `acquire` 方法的第一个参数名是与 `Lock.acquire()` 一样的 *block*。
{% note info %}
注：MacOS X 系统不支持 `sem_timedwait` 函数，因此带有 *timeout* 参数调用 `acquire()` 方法将使用休眠循环模拟该行为。
{% endnote %} {% note info %}注：若 `Ctrl-C` 信号量到达，而主线程在调用 `BoundedSemaphore.acquire()`、`Lock.acquire()`、`RLock.acquire()`、`Semaphore.acquire()`、`Condition.acquire()` 或 `Condition.wait()` 产生阻塞，会立即中断并抛出 `KeyboardInterrupt` 异常。
这里与 `threading` 模块中行为不同，`threading` 模块会忽略 SIGINT 信号。
{% endnote %}{% note info %}
注：该模块中的部分功能需要系统支持共享信号量。不支持的系统上会禁用 `multiprocessing.synchronize` 模块，继续引用会抛出 `ImportError` 异常。详细信息参阅 [bpo-3770](https://bugs.python.org/issue3770)。{% endnote %}


#### Shared ctypes Objects ####
可以使用共享内存创建共享对象，共享内存可以由子进程继承。

multiprocessing.**Value**(*typecode_or_type*, **args*, *lock=True*)
{% blockquote %}
返回从共享内存中分配的 ctypes 对象。默认返回的实际是对象的同步包装器。可以通过 `Value` 的 *value* 属性获取对象本身。
*typecode_or_type* 决定了返回对象的类型：它是 ctypes 类型或是由 `array` 模块使用的类型字符串。 **args* 被传递给对应类型的构造函数。
若参数 *lock* 为 `True`（默认值），会创建一个递归锁以同步对该值的访问。如果 *lock* 是 `Lock` 或 `RLock` 对象类型，将用于对同步值权限的保护。若 *lock* 是 `False`，则锁不会自动保护对返回对象的访问，因此将不是进程安全的。
例如像 `+=` 这样涉及读写的操作不是原子性的。因此，想以原子性方式递增共享值仅以如下方式将不能实现。
{% codeblock lang:python %}
counter.value += 1
{% endcodeblock %}加入相关锁是递归锁（默认情况）可以改为
{% codeblock lang:python %}
with counter.get_lock():
    counter.value += 1
{% endcodeblock %} 注意 *lock* 仅是关键字参数。
{% endblockquote %}

multiprocessing.**Array**(*typecode_or_type*, *size_or_initializer*, *, *lock=True*)
{% blockquote %}
返回从共享内存中分配的 ctypes 类型的数组。默认情况下，返回值实际是数组的同步包装器。
*typecode_or_type* 决定了返回对象的类型：它是 ctypes 类型或是由 `array` 模块使用的类型字符串。若 *size_or_initializer* 是一个整数，它决定数组长度，且数组初识值为零。否则， *size_or_initializer* 是一个用于初始化数组的序列，其长度决定了数组的长度。
若参数 *lock* 为 `True`（默认值），会创建一个锁以同步对该值的访问。如果 *lock* 是 `Lock` 或 `RLock` 对象类型，将用于对同步值权限的保护。若 *lock* 是 `False`，则锁不会自动保护对返回对象的访问，因此将不是进程安全的。
注意 *lock* 仅是关键字参数。
注意 `ctypes.c_char` 具有值和原始属性，允许用户存储和检测字符串。
{% endblockquote %}

##### `multiprocessing.sharedctypes` 模块 #####
`multiprocessing.sharedctypes` 模块提供了从共享内存分配 ctypes 类型对象的功能，这些对象可以被子进程继承。
{% note info %}
注：虽然可以在内存中存储指针，但要注意这将引用特定进程的地址空间中的位置。这样在其他进程的上下文环境中该指针可能无效，尝试在其他进程取消指针引用也可能导致进程的崩溃。
{% endnote %}

multiprocessing.sharedctypes.**RawArray**(*type_or_type*, *size_or_initializer*)
{% blockquote %}
返回从共享内存中分配的 ctypes 数组。

*typecode_or_type* 决定了返回数组的元素类型，其是一个 ctypes 类型或者 `array` 模块使用的一个类型字符串。若 *size_or_initializer* 是一个整数，它决定数组长度，且数组初识值为零。否则， *size_or_initializer* 是一个用于初始化数组的序列，其长度决定了数组的长度。

注：读写元素是非原子性的 - 使用 `Array()` 通过锁确保其实原子性的。
{% endblockquote %}

multiprocessing.sharedctypes.**RawValue**(*typecode_or_type*, **args*)
{% blockquote %}
返回从共享内存中分配的 ctypes 类型对象。

*typecode_or_type* 决定了返回数组的元素类型，其是一个 ctypes 类型或者 `array` 模块使用的一个类型字符串。**args* 参数将传递给该类型对象的构造函数。

注：读写元素是非原子性的 - 使用 `Value()` 通过锁确保其实原子性的。

注：`ctypes.c_char` 数组具有 `value` 和  `raw` 属性，可以通过该属性存储和检索字符串。
{% endblockquote %}

multiprocessing.sharedctypes.**Array**(*typecode_or_type*, *size_or_initializer*, *, *lock=True*)
{% blockquote %}
除需依赖锁类型变量确保线程安全，可以返回进程安全的修饰器而不是原始 ctypes 数组外，与 `RawArray()` 一样。

若参数 *lock* 为 `True`（默认值），会创建一个锁以同步对该值的访问。如果 *lock* 是 `Lock` 或 `RLock` 对象类型，将用于对同步值权限的保护。若 *lock* 是 `False`，则锁不会自动保护对返回对象的访问，因此将不是进程安全的。

注意 *lock* 仅是关键字参数。
{% endblockquote %}

multiprocessing.sharedctypes.**Value**(*typecode_or_type*, **args*, *lock=True*)
{% blockquote %}
除需依赖锁类型变量确保线程安全，可以返回进程安全的修饰器而不是原始 ctypes 类型对象外，与 `RawValue()` 一样。

若参数 *lock* 为 `True`（默认值），会创建一个锁以同步对该值的访问。如果 *lock* 是 `Lock` 或 `RLock` 对象类型，将用于对同步值权限的保护。若 *lock* 是 `False`，则锁不会自动保护对返回对象的访问，因此将不是进程安全的。

注意 *lock* 仅是关键字参数。
{% endblockquote %}

multiprocessing.sharedctypes.**copy**(*obj*)
{% blockquote %}
返回从共享内存分配的 ctypes 类型对象，共享内存中的对象是对象 *obj* 的一个副本。
{% endblockquote %}

multiprocessing.sharedctypes.**synchronized**(*obj*[, *lock*])
{% blockquote %}
返回一个 ctypes 类型的进程安全包装器对象，使用了 *lock* 同步访问权限。若 *lock* 是 `None`（默认值）将自动创建 `multiprocessing.RLock` 类型对象。

同步的包装器除包装的对象外还有两个方法：`get_obj()` 返回包装过的对象；`get_lock()` 返回用于同步的锁对象。
注：通过包装器访问 ctypes 对象比访问原始 ctypes 对象慢很多。
版本 3.5 后同步对象支持上下文管理协议。
{% endblockquote %}

下表比较了使用不同 ctypes 语法从共享内存创建 ctypes 类型对象的语法。（表中 `MyStruct` 是 `ctypes.Structure` 的子类。

| **ctypes** | **sharedctypes using type** | **sharedctypes using typecode** |
| :------ | :------ | :------ |
| c_double(2.4) | RawValue(c_double, 2.4) | RawValue('d', 2.4)
| MyStruct(4, 6) | RawValue(MyStruct, 4, 6) |  |
| (c_short * 7)() | RawArray(c_short, 7) | RawArray('h', 7)
| (c_int * 3)(9, 2, 8) | RawArray(c_int, (9, 2, 8)) | RawArray('i', (9, 2, 8)) |

下面是一个子进程改变 ctypes 类型数值的代码实例。
{% codeblock lang:python %}
from multiprocessing import Process, Lock
from multiprocessing.sharedctypes import Value, Array
from ctypes import Structure, c_double

class Point(Structure):
    _fields_ = [('x', c_double), ('y', c_double)]

def modify(n, x, s, A):
    n.value **= 2
    x.value **= 2
    s.value = s.value.upper()
    for a in A:
        a.x **= 2
        a.y **= 2

if __name__ == '__main__':
    lock = Lock()
    n = Value('i', 7)
    x = Value(c_double, 1.0/3.0, lock=False)
    s = Array('c', b'hello world', lock=lock)
    A = Array(Point, [(1.875,-6.25), (-5.75,2.0), (2.375,9.5)], lock=lock)
    p = Process(target=modify, args=(n, x, s, A))
    p.start()
    p.join()
    # print result
    print(n.value)
    print(x.value)
    print(s.value)
    print([(a.x, a.y) for a in A])
{% endcodeblock %}

打印结果如下。
{% codeblock %}
49
0.1111111111111111
HELLO WORLD
[(3.515625, 39.0625), (33.0625, 4.0), (5.640625, 90.25)]
{% endcodeblock %}

#### <span id='2.7'>Manager</span> ####

`Manager` 提供了一种在不同进程间共享创建的数据的方式，包括在不同计算机运行的进程之间通过网络共享。`Manager` 对象管理一个共享对象的服务进程，其他进程通过使用代理获取共享对象。

Multiprocessing.**Manager()**
{% blockquote %}
返回一个 `SyncManager` 对象，可以用来在进程间共享对象。返回的 `manager` 对象对应一个生成的子进程，该子进程会创建共享对象并返回相应的代理方法。
`manager` 进程一旦被垃圾回收或其父进程退出就会关闭。`manager` 类的定义位于 `multiprocessing.managers` 模块。
{% endblockquote %}

multiprocessing.managers.**BaseManager**([*address*[, *authkey*]])
{% blockquote %}
创建 `BaseManager` 对象。
创建后应该调用 `start()` 或 `get_server().serve_forever()` 方法保证 `manager` 对象引用的进程开启。
*address* 是 `manager` 进程监听的新连接的地址，若该参数为 `None` 则系统随机分配一个对应于某些空闲端口号的地址。
*authkey* 身份认证密钥，用于检查连接到服务进程的有效性。若该参数为 `None` 则使用 `current_process().authkey`。若使用该参数，其类型是 byte 字符串类型。

**start**([*initializer*[, *initargs*]])
`manager` 开启子进程。若构造函数不为空，启动时调用构造函数 `initializer(*initargs)`。

**get_server**()
返回 `manager` 对象控制下的真实服务器 `Server`，该类型支持 `serve_forever()` 方法。
{% codeblock lang:python %}
from multiprocessing.managers import BaseManager
manager = BaseManager(address=('', 50000), authkey=b'abc')
server = manaager.get_server()
server.serve_forever()
{% endcodeblock %} `Server` 类型具有 `address` 属性，由 `manager` 对象使用的地址。

**Connect**()
连接一个本地的 `manager` 对象到远程 `manager` 进程。
{% codeblock lang:python %}
from multiprocessing.managers import BaseManager
m = BaseManager(address=('127.0.0.1', 5000), authkey=b'abc')
m.connect()
{% endcodeblock %}

**shutdown**()
终止 `manager` 进程。该方法只有在服务进程执行过 `start()` 方法后才有效。
该方法可以被多次调用。

<span id='basemanager.register'>**register**(*typeid*[, *callable*[, *proxytype*[, *exposed*[, *method_to_typeid*[, *create_method*]]]]])</span>
类方法，用于注册类型或调用 `manager`。

*typeid* 是类型标识符，用于识别共享对象的类型，字符串类型。

*callable* 是用于创建该类型对象的调用。若 `manager` 实例使用 `connect()` 方法连接到服务器或设置 `create_method` 参数为 `False` 该参数可以设置为 `None`。

*proxytype* 是 `BaseProxy` 的子类，用于为该类型的共享对象创建代理。若该参数为 `None`，则自动创建代理。

*exposed* 用于指定允许使用 `BaseProxy.__callmethod()` 方法访问该类型共享对象的一系列方法的名字。（若该参数为 `None` 且 `proxytype._exposed_` 存在则使用后者替换前者。）
若未指定，则共享的对象的所有公开方法均可以被访问。（这里公开方法指 `__call__()` 方法与其他不以 '_' 为名字开头的方法。）

*method_to_typeid* 是指定了返回代理的公开方法应该返回的数据类型的映射。
它将方法的名字映射到类型字符串。（若该参数为 `None` 且 `proxytype._method_to_typeid_` 存在使用后者替换前者。）
若方法的名字在该参数中不存在或者该参数为 `None` 则方法返回的对象按值复制。

*create_method* 决定是否构造方法用于通知服务进程创建共享对象并返回代理方法，默认为 `True`。

`BaseManager` 实例有一个只读属性：

**address**
由 `manager` 使用的地址。
版本 3.3 后的变化：`manager` 对象支持上下文管理协议。
{% endblockquote %}


multiprocessing.managers.**SyncManager** 
{% blockquote %}
`BaseManager` 的子类，用于进程间同步。`multiprocessing.Manager()` 返回的对象类型。
同样支持创建共享 List 和 Dict。

**Barrier**(*parties*[, *action*[, *timeout*]])
创建共享 `threading.Barrier` 对象并返回其代理。3.3 版本后引入。

**BoundedSemaphore**([*value*])
创建共享 `threading.BoundedSemaphore` 对象并返回其代理。

**Condition**([*lock*])
创建共享 `threading.Condition` 对象并返回其代理。
若创建的共享对象类型包含 `lock`则返回值是 `threading.Lock` 或 `threading.RLock` 对象。
3.3 版本后添加了 `wait_for()` 方法。

**Event**()
创建共享 `threading.Event` 对象并返回其代理。

**Lock**()
创建共享 `threading.Lock` 对象并返回其代理。

**Namespace**()
创建共享 `Namespace` 对象并返回其代理。

**Queue**([*maxsize*])
创建共享 `queue.Queue` 对象并返回其代理。

**RLock**()
创建共享 `threading.RLock` 对象并返回其代理。

**Semaphore**([*value*])
创建共享 `threading.Semaphore` 对象并返回其代理。

**Array**(*typecode*, *sequence*)
创建一个 Array 并返回其代理。

**Value**(*typecode*, *value*)
创建带有可写属性 `value` 的对象并返回其代理。

**dict**() & **dict**(*mapping*) & **dict**(*sequence*)
创建共享 `dict` 对象并返回其代理。

**list**() & **list**(*sequence*)
创建共享 `list` 对象并返回其代理。
注：对 dict 或 list 的代理中可变值或可变项的修改可能不会通过 `manager` 传播，因为代理无法知道其值或项在何时被修改的。因此，要修改此类值或项，需要将修改后的对象重新赋值给 `manager` 包含的代理。
{% codeblock lang:python %}
# create a list proxy and append a mutable object (a dictionary)
lproxy = manager.list()
lproxy.append({})
# now mutate the dictionary
d = lproxy[0]
d['a'] = 1
d['b'] = 2
# at this point, the changes to d are not yet synced, but by
# reassigning the dictionary, the roxy is notified of the change
lproxy[0] = d
{% endcodeblock %}
{% endblockquote %}

mulltiprocessing.managers.**Namespace**
{% blockquote %}
可以注册 `SyncManager` 的类型。
一个 `Namespace` 对象没有公开方法，但有可写属性，其代表它属性的值。
然后，当使用 `Namespace` 对象的代理时，其以 '_' 开头名字的属性是代理的属性而不是引用对象的属性。
{% codeblock lang:python %}
manager = multiprocessing.Manager()
Global = manager.Namespace()
Global.x = 10
Global.y = 'hello'
Global._z = 12.3    # this is an attribute of the proxy
print(Global)
# Namespace(x=10, y='hello')
{% endcodeblock %}
{% endblockquote %}

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
QueueManager.r.Queue()
class QueueManager(BaseManager): pass
QueueManager.regiress=('', 50000), authkey=b'abracadabra')
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

```python
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
代理是一个对象，其指向不同进程间（可能）存在的共享对象。共享对象被认为是代理指向的对象。多个代理对象可能指向同一个共享对象。
代理对象有调用其引用对象方法的相应方法（虽然不是引用对象的所有方法均需通过代理调用）。代理通常可以如同引用对象一样使用，如下实例所示。
{% codeblock lang:python %}
from multiprocessing import Manager
manager = Manager()
l = manager.list([i*i for i in range(10)])
print(l)
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(repr(l))
# <ListProxy object, typeid 'list' at 0x...>
l[4]
# 16
l[2:5]
# [4, 9, 16]
{% endcodeblock %} 注意使用 `str()` 返回的是饮用对象的，而 `repr()` 返回的是代理对象自身的。
代理对象的一个重要特征是可序列化的因此可以在不同进程间传输。但是，如果将代理发送到对应的 manager 的进程反序列化将取消引用对象本身。这意味这一个共享对象可以包含第二个共享对象。
{% codeblock lang:python %}
a = manager.list()
b = manager.list()
a.append(b)         # referent of a now contains referent of b
print(a, b)
# [[]] []
b.append('hello')
print(a, b)
# [['hello']] ['hello']
{% endcodeblock %} {% note info %}
`multiprocessing` 中的代理类型不支持值比较。因此对如下实例有
{% codeblock lang:python %}manager.list([1, 2, 3]) == [1, 2, 3] # False {% endcodeblock %} 因此进行比较操作时应使用引用对象的拷贝。
{% endnote %}

*class* multiprocessing.managers.**BaseProxy**
{% blockquote %}
代理对象是 `BaseProxy` 子类的实例。

**_callmethod**(*methodname*[, *args*[, *kwds*]])
调用引用对象方法并返回执行结果。
`proxy` 是一个引用对象是 `obj` 的代理，表达式
{% codeblock lang:python %}proxy._callmethod(methodname, args, kwds){% endcodeblock %} 等价于 manager 进程中的表达式 {% codeblock lang:python %}getattr(obj, methodname)(*args, **kwds){% endcodeblock %}

返回值是调用结果的拷贝或者新共享对象的代理 - *method_to_typeid* 参数参阅 [BaseManager.register()](#basemanager.register)。

若调用引发异常，将由 `_call_method` 方法重新引发该异常。若在 manger 进程中引发了其他异常，将其转换为 `RemoteError` 异常并由 `_callmethod()` 引发。

特别注意：若参数 *methodname* 是引用对象的非公开方法将抛出异常。

使用该方法的代码实例。
{% codeblock lang:python %}
l = manager.list(range(10))
l._callmethod('__len__')
# 10
l._callmethod('__getitem__', (slice(2, 7),)) # equivalent to l[2:7]
# [2, 3, 4, 5, 6]
l._callmethod('__getitem__', (20,))          # equivalent to l[20]
# Traceback (most recent call last):
# ...
# IndexError: list index out of range
{% endcodeblock %}

**_getvalue**()
返回引用对象的拷贝。
若引用对象不可以反序列化将引发异常。

**__repr__**()
返回代理对象的表示。

**__str__**()
返回引用对象的表示。

{% endblockquote %}

##### Cleanup #####
代理对象使用的弱引用回调，当被垃圾回收时，会从拥有其引用的 manager 中注销自己。
共享对象不再被任何代理对象引用时，从 manager 进程中删除。

#### Process Pool ####
创建进程池，可以执行使用 `Pool` 类提交给它的任务。

class multiprocessing.pool.**Pool**([*processes*[, *initializer*[, *initargs*[, *maxtasksperchild*[, *context*]]]]])
{% blockquote %}
进程池对象，可以控制提交任务的工作进程池，支持异步获取超时或回调结果，具有并行映射的实现。
*processes* 是使用的工作进程数量。若该参数为 `None` 使用 `os.cpu_count()` 返回的数字。
若 *initializer* 不是 `None` 则在每个进程 `start()` 时调用该 `initializer(*initargs)` 函数。
*maxtasksperchild* 是一个工作进程在退出或使用新工作进程取代之前可以完成的任务数，以释放不再使用的资源。默认值是 `None` 意味着工作进程生命周期与进程池一样。
*context* 可用于指定工作进程 `start()` 的 context。通常通过使用 `multiprocessing.Pool()` 或者 context 对象的 `Pool()` 方法创建进程池。两种情景均可以使用 *context* 设置。
注：进程池对象的方法仅可以由创建进程池的进程调用。
版本 3.2 后添加了 *maxtasksperchild*。
版本 3.4 后添加了 *context*。
{% note info %}
注：工作进程的生命周期通常与进程池生命周期一直。在其他系统（如 Apache，mode_wsgi 等）存在频繁切换进程时，在进程退出或者新进程取代旧进程前，释放用于工作进程所拥有的资源的模式。*maxtasksperchild* 参数向用户公开了进程池的这种能力。
{% endnote %} **apply**(*func*[, *args*[, *kwds*]])
使用参数 *args* 和 关键字参数 *kwds* 调用 *func* 函数。会阻塞至任务执行结果就绪。`apply_async()` 方法更适合阻塞下的并行执行任务。此外，*func* 仅在进程池中的一个工作进程中执行。

**apply_async**(*func*[, *args*[, *kwds*[, *callback*[, *error_callback*]]]])
`apply()` 方法的一个变体，返回一个结果对象。
若指定了参数 *callback*，该参数值应该是带有一个参数的可调用函数。当结果就绪时，对结果调用该函数。若该情况下调用失败，`error_callback` 异常取代返回结果。
若指定了参数 *error_callback*，该参数值是带有一个参数的可调用函数。当目标函数执行实行，则异常实例作为参数调用该函数。
回调应该是很快完成的，否则处理结果线程将会阻塞。

**map**(*func*, *iterable*[, *chunksize*])
内置 `map()` 的并行等价物（只支持一个 *iterable* 参数）。阻塞至结果就绪。
该方法将迭代对象切分为多个块并将其作为单独任务提交到进程池。可以通过参数 *chunksize* 设置为正整数指定这些块的（近似）大小。

**map_async**(*func*, *iterable*[, *chunksize*[, *callback*[, *error_callback*]]])
`map()` 方法的一个变种，返回一个结果对象。
若指定了参数 *callback*，该参数值应该是带有一个参数的可调用函数。当结果就绪时，对结果调用该函数。若该情况下调用失败，`error_callback` 异常取代返回结果。
若指定了参数 *error_callback*，该参数值是带有一个参数的可调用函数。当目标函数执行实行，则异常实例作为参数调用该函数。
回调应该是很快完成的，否则处理结果线程将会阻塞。

**imap**(*func*, *iterable*[, *chunksize*])
一个懒惰版本的 `map()`。
参数 *chunksize* 与 `map()` 方法的使用相同。对于非常长的迭代对象使用一个较大的 *chunksize* 可以使任务比使用默认为 1 完成的快很多。
此外，若 *chunksize* 为 1，则该方法返回的迭代器 `next()` 具有可选参数 *timeout*，若在超时时间内（单位为秒）没有返回，`next(timeout)` 将抛出 `multiprocessing.TimeoutError` 异常。

**imap_unordered**(*func*, *iterable*[, *chunksize*])
除返回的迭代器被认为是无序的外，与 `imap()` 方法相同。（只有当只有一个工作进程时才能保证顺序是正确的）

**starmap**(*func*, *iterable*[, *chunksize*])
除 *iterable* 的非解包元素作为参数迭代外，与 `map()` 方法相同。
因此，`[(1, 2), (3, 4)]` 的 *iterable* 参数值迭代为 `[func(1, 2), func(3, 4)]`。
版本 3.3 后添加。

**starmap_async**(*func*, *iterable*[, *chunksize*[, *callback*[, *error_back*]]])
`starmap()` 和 `map_async()` 方法的组合。迭代 *iterable* 中的非解包元素并调用 *func* 返回结果对象。 
版本 3.3 后添加。

**close**()
防止更多任务被提交到进程池。提交到进程池的所有任务完成后，工作进程退出。

**terminate**()
立即终止工作进程而不再继续执行未完成的任务。进程池对象被垃圾回收时直接调用该方法。

**join**()
等待工作进程退出。必须在 `close()` 和 `terminate()` 方法调用前调用该方法。

版本 3.3 后进程池对象开始支持上下文管理协议。

{% endblockquote %}

class multiprocessing.pool.AsyncResult
{% blockquote %}
由 `Pool.apply_async()` 和 `Pool.map_async()` 返回结果的类。

**get**([*timeout*])
返回到达时的结果。若 *timeout* 非 `None` 且结果没有在 *timeout* 时间内返回将抛出 `multiprocessing.TimeoutError` 异常。若远程调用引发了异常，将由该方法重新抛出该异常。

**wait**([*timeout*])
等待至结果返回或超出 *timeout* 时间。

**ready**()
返回调用是否完成。

**successfult**()
返回调用完成是否抛出了异常。若结果还未就绪将抛出 `AssertionError` 异常。

{% endblockquote %}

下面的实例说明如何使用 `Pool`。
{% codeblock lang:python %}
from multiprocessing import Pool
import time

def f(x):
    return x*x

if __name__ == '__main__':
    # start 4 worker processes
    with Pool(processes=4) as pool:
        # evaluate "f(10)" asynchronously in a single process
        result = pool.apply_async(f, (10,))

        # prints "100" unless your computer is *very* slow
        print(result.get(timeout=1))
        
        # prints "[0, 1, 4,..., 81]"
        print(pool.map(f, range(10)))
        # prints "[0, 1, 4,..., 81]"

        it = pool.imap(f, range(10))
        # prints "0"
        print(next(it))

        # prints "1"
        print(next(it))

        # prints "4" unless your computer is *very* slow
        print(it.next(timeout=1))

        result = pool.apply_async(time.sleep, (10,))
        # raises multiprocessing.TimeoutError
        print(result.get(timeout=1))
{% endcodeblock %}

#### <span id='2.10'>Listener && Client</span> ####
通常进程间传递消息使用队列或由 `Pipe()` 方法返回的 `Connection` 对象。

此外，`multiprocessing.connection` 模块有更多的灵活性。它可以处理来自处理套接字或者 Windows 管道等 API 的更高级别消息。同样支持使用 `hmac` 模块进行数字认证，支持同时轮询多个连接。

multiprocessing.connection.**deliver_challenge**(*connection*, *authkey*)
向另一端发送一个随机产生的消息并等待回复。
若返回的回复使用秘钥作为键匹配数字认证，则向另一端发送欢迎消息。否则抛出 `AuthenticationError` 异常。

multiprocessing.connection.**answer_challenge**(*connection*, *authkey*)
接收消息，然后使用 *authkey* 作为键计算出消息摘要发送回去。
若没有收到欢迎消息将抛出 `AuthenticationError` 异常。

multiprocessing.connection.**Client**(*address*[, *family*[, *authenticate*[, *authkey*]]])
使用 *address* 地址尝试建立到 Listener 的连接，返回 `Connection` 对象。
连接的类型由参数 *family* 决定，但通常可以省略，因为可以通过地址的格式推断出来（参阅 [Address Formats](#2.10.1)）
若 *authenticate* 是 `True` 或者 *authkey* 是一个字节字符串，将使用摘要身份认证。用于认证的秘钥是 *authkey*（若参数为 `None` 则使用 `current_process().authkey`）。若认证失败抛出 `AuthenticationError` 异常。参阅 [Authentication keys](#2.11)。

class multiprocessing.connection.**Listener**([*address*[, *family[*, *backlog*[, *authenticate*[, *authkey*]]]]])
{% blockquote %}
正在监听连接的包装器，绑定在套接字或 Windows 管道上。

*address* 是 Listener 对象用来绑定套接字或管道的地址。
{% note info %} 地址 '0.0.0.0' 地址不能是 Windows 的可连接端点，应使用 '127.0.0.1'。{% endnote %} *family* 是使用的套接字（或管道）的类型，如下所示
1. 'AF_INET' 对应 TCP 套接字；
2. 'AF_UNIX' 对应 Unix 套接字；
3. 'AF_PIPE' 对应 Windows 管道。

其中只有第一个是保证可用的。
若该参数为 `None` 则会从地址格式推断。若 *address* 也是 `None` 选择默认值。默认值被假定为可以最快获取的。参阅 [Address Formats](#2.10.1)。
注意，若 *family* 是 'AF_UNIX' 且 *address* 为 `None` 将使用 `tempfile.mkstemp()` 在私有临时目录中创建套接字。
若 Listener 对象使用套接字，一旦绑定该套接字，将向套接字的 `listen()` 方法发送 *backlog*（默认为1）。
若 *authenticate* 为 `True`（默认为 `False`）或 *authkey* 不是 `None` 将使用摘要身份认证。
若 *authkey* 是一个字节字符串，其将被用作认证秘钥；否则一定为 `None`。
若 *authkey* 为 `None` 且 *authenticate* 为 `True` 将使用 `current_process().authkey` 作为认证秘钥。
若 *authkey* 为 `None` 且 *authenticate* 为 `False` 将不会做认证。若认证失败抛出 `AuthenticationError` 异常。参阅 [Authentication keys](#2.11)。

**accept**()
尝试连接到 Listener 对象绑定的套接字或管道上，返回 `Connection` 对象。若尝试认证且认证失败，将抛出 `AuthenticationError` 异常。

**close**()
关闭 Listener 对象绑定的套接字或管道。Listener 对象被垃圾回收时自动调用该方法。**但是**建议明确调用该方法。

Listener 对象还具有如下只读属性。
**address**
Listener 对象使用的地址。

**last_accepted**
上次连接成功使用的地址。若该地址不可用为 `None`。

版本 3.3 后 Listener 对象支持上下文管理协议。
{% endblockquote %}

multiprocessing.connection.**wait**(*object_list*, *timeout=None*)
等待 *object_list* 中的对象就绪。返回列表中准备就绪的对象的列表。若 *timeout* 是浮点数 F 该调用至多阻塞 F 秒。若 *timeout* 为 `None` 则阻塞时间没有上限。负数的参数等效于零。
对 Unix 和 Windows 而言，出现在 *object_list* 中的对象需满足如下要求
* 可读 `connection` 对象；
* 可连接、可读的 `socket.socket` 对象；
* `Process` 对象的 `sentinel` 属性。
当连接或套接字对象中的数据可以获取时，或者另一端已经关闭时，认为连接或套接字对象已就绪。

**Unix**：`wait(object_list, timeout)` 几乎等价于 `select.select(object_list, [], [], timeout)`。不同之处在于，`select.select()` 可以被信号中断，抛出错误码为 `EINTR` 的 `OSError` 异常，而 `wait()` 不会。
**Windows**：*object_list* 中的对象必须是一个可以等待的整数句柄（根据 Win32 函数 `WaitForMultipleObjects()` 使用文档中的定义）或者是一个带有 `fileno()` 方法（该方法返回一个套接字句柄或管道句柄）的对象。（注意管道句柄和套接字句柄是不是可等待的句柄。）

版本 3.3 后添加。

Examples
下面的服务端代码创建了一个 listener，其使用 `secret_password` 作为秘钥。
之后等待连接并向客户端发送数据。
{% codeblock lang:python %}
from multiprocessing.connection import Listener
from array import array

address = ('localhost', 6000)     # family is deduced to be 'AF_INET'

with Listener(address, authkey=b'secret password') as listener:
    with listener.accept() as conn:
        print('connection accepted from', listener.last_accepted)
        conn.send([2.25, None, 'junk', float])
        conn.send_bytes(b'hello')
        conn.send_bytes(array('i', [42, 1729]))
{% endcodeblock %} 下面的代码代表客户端连接到服务端并从服务端接收数据。
{% codeblock lang:python %}
from multiprocessing.connection import Client
from array import array

address = ('localhost', 6000)

with Client(address, authkey=b'secret password') as conn:
    print(conn.recv())                  # => [2.25, None, 'junk', float]
    print(conn.recv_bytes())            # => 'hello'
    arr = array('i', [0, 0, 0, 0, 0])
    print(conn.recv_bytes_into(arr))    # => 8
    print(arr)                          # => array('i', [42, 1729, 0, 0, 0])
{% endcodeblock %} 下面的代码使用 `wait()` 方法同时等待来自多个进程的消息。
{% codeblock lang:python %}
import time, random
from multiprocessing import Process, Pipe, current_process
from multiprocessing.connection import wait

def foo(w):
    for i in range(10):
        w.send((i, current_process().name))
    w.close()

if __name__ == '__main__':
    readers = []
    for i in range(4):
        r, w = Pipe(duplex=False)
        readers.append(r)
        p = Process(target=foo, args=(w,))
        p.start()
        # We close the writable end of the pipe now to be sure that
        # p is the only process which owns a handle for it.  This
        # ensures that when p closes its handle for the writable end,
        # wait() will promptly report the readable end as being ready.
        w.close()
    while readers:
        for r in wait(readers):
            try:
                msg = r.recv()
            except EOFError:
                readers.remove(r)
            else:
                print(msg)
{% endcodeblock %}


##### <span id='2.10.1'>Address Formats</span> #####
* `AF_INET` 地址是一个 `(hostname, port)` 格式的元组，*hostname* 是一个字符串，*port* 是一整数。
* `AF_UNIX` 地址是代表文件系统中一个文件名字的字符串。
* `AF_PIPE` 地址是一个 `r'\\.\pipe\PipeName` 形式的字符串。使用 `Client()` 连接到远程计算机中名为 *ServerName* 的管道应使用 `r'\\ServerName\pipe\PipeName` 替换。

注意任何以两个反斜杠开始的字符串会被默认为 `AF_PIPE` 地址而不是 `AF_UNIX` 地址。

#### <span id='2.11'>Authentication keys</span> ####
当使用 `Connection.recv` 时，接收到的数据将会自动反序列化。不幸地是当反序列化的数据来自于非信任源时就会有安全风险。因此，`Listener` 和 `Client()` 使用 [hmac](https://docs.python.org/3.5/library/hmac.html#module-hmac) 模块提供摘要认证。

一个可以当做密码的秘钥是一个字节类型的字符串：一旦连接建立，连接的双方均需要提供对方知道的秘钥。（即时两端使用相同的秘钥验证也不涉及通过连接发送秘钥）

若要求身份认证但没有提供秘钥，将使用当前进程的秘钥，该值自动继承创建该对象的当前进程的秘钥。这意味着（默认）所有的多进程程序的进程间通信使用同一个秘钥。

合适的秘钥可以通过使用 `os.urandom()` 产生。

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
尽可能的避免在进程之间传递大量数据。
进程间通信最好严格使用 `queue` 或 `pipe` 而不是使用级别较低的同步原语。

2. 序列化
保证代理方法的参数是可以序列化的。（可以被 pickle）

3. 代理是线程安全的
若代理对象来自多个线程，应该使用锁进行保护。（确保使用同一个代理的不同进程不会出现问题）

4. 僵尸进程的 `join()` 方法
Unix 中，当一个子进程完成退出没有调用 `join()` 方法而父进程在继续运行时，子进程成为僵尸进程。因为每一个新进程 `start()` （或调用 `active_children()` 方法），所有没有调用 `join()` 方法的终止进程将会 `join()`，所以僵尸进程的数量不会特别多。同事，调用一个已终止进程的 `Process.is_alive` 将会调用 `join()` 方法。即使如此，开始时明确进程的执行流程也是一个好习惯。
{% note info %}
**个人解读**
`join()` 方法具有清除僵尸进程的作用。通过下述源码我们可以发现，该方法通过子父进程的串行进程执行方式清除僵尸进程。`join()` 方法主要做了两件事情：① 通知父进程调用 `wait()` 方法；② 等待子进程终止或超时后，移除子进程。不带参数的 `join()` 方法
若调用带有参数的 `join(timeout)` 方法，在代码中父进程等待 `timeout` 时长后，开始唤醒父进程，此时子父进程开始同时执行，父进程首先执行`join(timeout)` 方法，若此时子进程还未结束，则变量 res 获取的进程退出信息为空，即不会清除子进程，然后继续执行父进程的后续逻辑，此时子父进程在并行执行。若子进程再次终止后父任务还未结束，则父进程因无法获取到子进程的退出信息而导致子进程沦为僵尸进程。

{% codeblock lang:python %}
# CPython 3.5.2 /Lib/multiprocessing/process.py BaseProcess 类
class BaseProcess(object):
    '''
    Process objects represent activity that is run in a separate process
    The class is analogous to `threading.Thread`
    '''
    # ... ignore other methods
    def _Popen(self):
        raise NotImplementedError
    def join(self, timeout=None):
        '''
        Wait until child process terminates
        '''
        assert self._parent_pid == os.getpid(), 'can only join a child process'
        assert self._popen is not None, 'can only join a started process'
        res = self._popen.wait(timeout)
        if res is not None:
            _children.discard(self)
{% endcodeblock %}

{% codeblock lang:python%}
# CPython 3.5.2 /Lib/multiprocessing/popen_fork.py Popen 类
class Popen(object):
    method = 'fork'
    # ... ignore other methods
    def poll(self, flag=os.WNOHANG):
        if self.returncode is None:
            while True:
                try:
                   pid, sts = os.waitpid(self.pid, flag)
                except OSError as e:
                    # Child process not yet created. See #1731717
                    # e.errno == errno.ECHILD == 10
                    return None
                else:
                ​    break
                if pid == self.pid:
                if os.WIFSIGNALED(sts):
                ​    self.returncode = -os.WTERMSIG(sts)
                else:
                ​    assert os.WIFEXITED(sts)
                ​    self.returncode = os.WEXITSTATUS(sts)
                return self.returncode
    def wait(self, timeout=None):
        if self.returncode is None:
            if timeout is not None:
                from multiprocessing.connectionimport wait
                if not wait([self.sentinel], timeoout):
        ​           return None
            # This shouldn't block if wait() returned successfully.
            return self.poll(os.WNOHANG if timeout == 0.0 else 0)
        return self.returncode
{% endcodeblock %}

消除僵尸进程的方法：
* 创建两次子进程
即将父进程创建子进程的方式改为父进程先创建子进程代理进程，由代理进程创建子进程执行任务。代理进程在完成子进程的创建任务后，调用代理进程的 `join()` 方法消除（因为代理进程的执行时间很短）；同时，执行任务的子进程变成了孤儿进程，无论执行多久，最终被 init 进程（即进程号为 1 的进程）所收养，由 init 进程对其完成状态收集工作，避免了僵尸进程的产生。

* 利用系统信号清除僵尸进程
基于 Linux 信号方式添加代码 `signal.signal(signal.SIGCHLD, signal.SIG_IGN)`。`signal.signal()` 函数定义在收到信号时执行自定义的处理程序。此处是指忽略创建的子进程的退出信号，
`signal.SIGCHLD` 在子进程状态改变后会产生此信号。
`sinal.SIG_IGN` 是标准信号处理程序，简单忽略给定信号；默认使用 `sinal.SIG_DFL`，代表不理会给定信号，但也不丢弃。修改后不保存子进程的状态使之成为孤儿进程被 init 进程回收。

* 此外，父进程退出时创建的僵尸进程也会被清除。
{% endnote %}

5. 继承比使用 `pickle/unpickle` 更好
使用 `spawn` 或者 `forkserver` 方式调用 `start()` 方法时，`multiprocessing` 中的许多类型需要被序列化给子进程使用它们。但是，通常应该避免使用 `pipe` 或 `queue` 将共享对象发送给其他进程。应该合理安排程序，使得需要访问其他进程创建的共享资源的进程通过从祖先进程继承的方式去访问。

6. 避免主动终止进程
使用 `Process.terminate` 方法可以终止进程，但可能导致进程当前使用的资源（如锁、信号量、管道和队列）被破坏或者不能被其他进程使用。因此，最好只有没有使用任何共享资源的进程上调用 `Process.terminate` 方法。

7. 使用 `queue` 通信的进程调用 `join()`
请记住，当一个进程调用 `join()` 方法时，Python 会检测被放入到 `queue` 中数据是否已经被全部删除（如 `queue.get()`，若没有被完全删除，进程将会一直等待，直到所有缓冲的数据由“feeder”线程将数据提供给底层管道。（子进程可以通过调用 `Queue.cancel_join_thread` 方法取消该行为。）这意味着，无论在什么时候使用 `queue` 都需要确保进程调用 `join()` 方法前放入到队列中的数据被全部清除。否则，无法确保将数据放入到队列中的进程已经终止。**切记，非后台进程将会自动调用 `join()` 方法。**
下面是一个会造成死锁的代码示例。
{% codeblock lang:python %}
from multiprocessing import Process, Queue

def f(q):
    q.put('X' * 100000100000100000100000 
if __name__ == '__main__':
    queue = Q = Process(target=f, args=(queue,))
    p.start()
    p.join()                    # this deadlocks
    obj = queue.get()
{% endcodeblock %}
一个改进办法是交换上述代码的最后两行（或简单删除 `p.join()` ）

8. 将资源明确的传递给子进程
在使用 `fork` 方式调用 `start()` 方法的 Unix 环境中，子进程可以使用父进程创建的全局资源中的所有共享资源，但最好将共享的对象作为参数传递给子进程的构造函数。
这样除了可以使得代码（可能）兼容 Windows 平台和其他调用 `start()` 方法的方式外，同时还可以确保共享的对象在子进程仍处于活动状态时不会在父进程被垃圾回收。如果父进程垃圾回收时释放某些共享资源，这种方式就非常重要。如下代码示例应该被改写为后者。
{% codeblock lang:python%}
# 有问题的方式
from multiprocessing import Process, Lock

def f():
    # ... do something using "lock" ...

if __name__ == '__main__':
    lock = Lock()
    for i in range(10):
        Process(target=f).start()

# 更好的写法
from multiprocessing import Process, Lock

def f(l):
    # ... do something using "l" ...

if __name__ == '__main__':
    lock = Lock()
    for i in range(10):
        Process(target=f, args=(lock,)).start()
{% endcodeblock %}

9. 用文件对象取代 `sys.stdin` 时要小心
`multiprocessing` 模块最初是无约束的调用 `os.close(sys.stdin.fileno())`。
在 `multiprocessing.Process._bootstrap()` 方法中这样会导致子进程出现问题，修改如下
{% codeblock lang:python%}
sys.stdin.close()
sys.stdin = open(os.open(os.devnull, os.O_RDONLY), closefd=False)
{% endcodeblock %}
这样解决了进程相互冲突导致的文件描述符错误的基本问题，但对于带有输出缓冲的文件对象取代 `sys.stdin()` 的应用程序引入了潜在危险。危险是：如果在多个进程中调用同一个文件对象的 `close()` 方法多次，可能导致相同的数据被多次刷新到文件对象中，从而导致了数据损毁。
如果需要自己实现文件对象及其缓存，则可以通过对缓存附加进程 PID 并在 PID 发生改变时丢弃缓存使其成为拷贝安全的。一个实现实例如下所示。
{% codeblock lang:python%}
@property
def cache(self):
    pid = os.getpid()
    if pid != self._pid:
        self._pid = pid
        self._cache = []
    return self._cache
{% endcodeblock %}
更详细的信息可以阅读 [bpo-5155](https://bugs.python.org/issue5155)，[pbo-5313](https://bugs.python.org/issue5313) 和 [bpo-5331](https://bugs.python.org/issue5331)。

#### spawn方式 && forkserver方式 ####

此外还有些不适用于 `fork` 方式 `start()` 的其他约束。

1. 更多序列化要求
保证传给 `Process.__init__()` 方法的参数都是可以序列化的。同时也应该保证 `Process` 子类调用 `Process.start()` 方法时其实例也可以被序列化。

2. 全局变量
请记住，如果在子进程中尝试访问全局变量，则子进程看到的变量值（如果有）可能与父进程中调用 `Process.start()` 方法时的值不同。

3. `main` 模块的引用安全
确保 Python 解释器可以安全的导入主模块而不会导致意外发生（如启动一个新进程时）。例如，使用 `spawn` 或 `forkserver` 方式调用 `start()` 方法运行下面的代码实例会导致 `RuntimeError`。
{% codeblock lang:python %}
from multiprocessing import Process

def foo():
    print('hello')

p = Process(target=foo)
p.start()
{% endcodeblock %}
应使用 `if __name__ == '__main__':` 来包含程序入口，如下代码所示。
{% codeblock lang:python %}
from multiprocessing import Process, freeze_support, set_start_method

def foo():
    print('hello')

if __name__ == '__main__':
    freeze_support()
    set_start_method('spawn')
    p = Process(target=foo)
    p.start()
{% endcodeblock %}
（若程序实体正常运行而不需要冻结可以省略 `freeze_support()` 行代码。）这允许新生成的 Python 解释器可以安全导入模块，然后运行模块中 `foo()` 函数。在主模块中创建了 `Pool` 或 `Manager` 具有类似约束。

### Examples ###

#### manager ####

如下是一个创建使用 `manager` 及其代理的实例。
{% codeblock lang:python %}
from multiprocessing import freeze_support
from multiprocessing.managers import BaseManager, BaseProxy
import operator


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


if __name__ == '__main__':
    freeze_support()
    test()
{% endcodeblock %}

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
        try:
            print(list(pool.imap(f, list(range(10))1110))ro             print('\tGot ZeroDiviionEionErrionErrrrionErriorrrrionError as expected from list(pool.imap())')
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

def pluus(a, b):
    time.sleep(0.5*random.random())
    return a + b


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
在完成内存优化工作的尾声才完成了这篇文档的翻译工作。原以为较为简单的工作，进行中发现读懂的基础上翻译为较为合适的中文还是有不小的难度及工作量的。
首先，需要通读完成理解统一概念之后才能对整篇文档有所了解；其次，翻译过程中，很多细节的概念或不常见的参数还需要花费时间查阅资料或阅读源码；最终初稿完成的基础上仍需多次阅读校正（该步骤还未完成，因此欢迎大家支持本文翻译的不当之处或错误，感谢~）。因此还是花费了较长的时间编辑本篇文档。
当然，该工作对自己了解 Python 的一些机制尤其是与进程相关的一些设计思路，功能考虑等有了更深的认知和理解，若有暇可在后续整理出来供大家参考，共同讨论和进步。


#### 注 ####
该篇博客参考文档为 Python 3.5.6 版本，源码版本为 CPython 3.5.2。

#### 致谢 ####
本篇工作的完成还来自很多朋友的支持和帮助，如僵尸进程，孤儿进程等相关资料来自于网上博客的参考，CSDN 平台的 [@逸辰杳](https://blog.csdn.net/u011675745/article/details/78604760)；不同系统平台的子进程开启方式部分受到同事 [@linquanisaac](https://github.com/linquanisaac) [@Zhiya Zang](https://github.com/simpleapples) [@Mtax](https://github.com/mtax) 的帮助；以及更多同事朋友的支持帮助~再次感谢他们。