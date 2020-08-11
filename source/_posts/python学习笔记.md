---
title: python学习笔记
date: 2019-02-14 19:01:55
tags:
- python
categories: 
- python
---
<!-- more -->
只写过两个简单python脚本。最近碰到的项目都是用python搭建的后端服务，还是学学python吧。。


## 模块化

### 模块

python作为动态解释型语言，其模块的加载也是动态进行的，一个py文件就是一个模块，使用import语句导入模块或者模块中的某个方法。

```python

import  somemodule
from somemoduel import somefunction（，otherfunction）  #从模块中引入某个或几个方法，*匹配所有方法

```

如果文件作为主程序运行，编译器会给特殊变量`__name__`赋值为`__main__`，如果是作为模块运行，`__name__`的值就是模块的名字。因此也有如下写法保证某些代码只在作为主程序运行时执行：

```python

If __name__ == ‘__main__’ :
    # somecode...

```

### 包

一个包是是一个文件夹中的模块的集合。一个python包文件夹中必须要有一个`__init__.py`文件（可为空），这个文件的作用就是告诉编译器这是一个包，否则就会认为它是个普通文件夹忽略它。

也可以从一个包中导入某个模块

```python

from package import module

```

## 面向对象

### 类和实例

python也是面向对象的语言，类（class）和实例（instance）的概念不多赘述。定义类的时候可以通过定义一个特殊的`__init__`方法，在创建实例的时候赋予一些初始属性。`__init__`方法的第一个参数永远是self，self指向实例本身，这个有点类似js的this指针。有了`__init__`方法以后就需要在创建实例时传入除self以外与形参匹配的参数，解释器负责传入self参数。

```python

class Student():
    def __init__(self, name, score):
         self.name = name
         self.score = score

xcola = Student('xcola', 60)

```
### 访问控制

类的内部存在属性和方法，外部代码可以直接调用和更改实例的属性和方法，如果不希望内部属性被外部访问，可以将属性定义为私有变量。私有变量的变量名以双下划线开头，如 `__name`。私有变量无法从外部访问，这样就保证了外部代码不能随意更改对象内部的状态。

```python

class Student():
    def __init__(self, name, score):
        self.__name = name
        self.__score = score
    
    def get_name(self):
        return self.__name 

    def set_score(self):
        self.__score = score
        
xcola = Student('xcola', 59)
xcola.set_score(60)

```

### 继承和多态和鸭子类型

不同于js基于原型的原型链继承，python的继承是基于类的继承，天然支持多态。

```python

class Animal():
    def eat(self):
        print 'animal eat something'
        
class Bark():
    def barking(self):
        print 'barking...'

class Cat(Animal):
    def eat(self):
        print 'cat eat something'
  
    def sleep(self):
        print 'sleep and snoring...'
    
class Dog(Animal, Bark):  # 多重继承
    pass
    
cat = Cat()
dog = Dog()
dog.eat()  # animal eat something
dog.barking() # barking...
cat.eat() # cat eat something
cat.sleep() # sleep and snoring...

def feed_animal(animal):
    animal.eat()

feed_animal(cat) # cat eat something
feed_animal(dog) # animal eat something

```

子类通过继承直接获得父类的方法，也可以添加自己的方法，也可以重写父类的方法。python中可以通过多重继承的方式获得mixin的能力，比如Dog类同时继承了Animal类和Bark类，所以dog直接获得了animal的eat方法和bark的barking方法，能吃会叫。Cat也继承了animal，但它重写了父类的eat方法，所以执行的时候执行cat自己定义的eat方法。cat还添加了属于自己的sleep方法，告诉我们猫睡觉的时候打呼噜。

python和js一样是动态语言，feed_animal方法接收的参数不需要是animal类或他的子类，给一个会eat的对象就可以喂它。

## 进程和线程

> 对操作系统来说，一个任务就是一个进程。有的进程还会同时干多个事，还需要同时运行多个子任务，因此进程内还有线程。一个进程至少有一个线程，多线程的操作方式和多进程的操作方式相同，都由操作系统负责调度。

因此如果要让python程序同时执行多个任务，有三种解决方案——多进程和多线程和多进程多线程。

### 多进程

unix和linux提供了fork方法。fork方法可以把当前进程（父进程）复制一份出来（子进程），并分别在父进程和子进程中分别返回。子进程返回0，父进程返回子进程ID。

> 有了`fork`调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

```python
import os

print('Process (%s) start...' % os.getpid())
pid = os.fork()
if pid == 0:
    print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
else:
    print('I (%s) just created a child process (%s).' % (os.getpid(), pid))
    
# Process (876) start...
# I (876) just created a child process (877).
# I am child process (877) and my parent is 876.
```



windows没有fork调用，想让python跨平台多进程需要用multiprocessing支持。multiprocessing的Process类即代表一个进程对象。start方法启动进程，join方法等待子进程结束。

```python
from multiprocessing import Process
import os

def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()
    p.join()
    print('Child process end.')
```



#### 进程间通信

multiprocessing提供Queue（队列）和Pipe（管道）等方式交换数据，Queue是基于Pipe的进一步实现。

##### pipe

首先，pipe是半双工的，仅仅适用于两个进程一读一写的情况。

![Pipe](https://segmentfault.com/img/bVIeWr?w=960&h=540)

当主进程创建Pipe的时候，Pipe的两个Connections连接的的都是主进程。当主进程创建子进程后，Connections也被拷贝了一份。此时有了4个Connections。此后再关闭主进程的一个Out Connection，关闭一个子进程的一个In Connection，那么就建立好了一个输入在主进程，输出在子进程的管道。

```python
from multiprocessing import Pipe, Process

def son_process(x, pipe):
    _out_pipe, _in_pipe = pipe
    # 子进程关闭fork过来的输入端
    _in_pipe.close()
    while True:
        try:
            msg = _out_pipe.recv()
            print msg
        except EOFError:
            # 当out_pipe接受不到输出的时候且输入被关闭的时候，会抛出EORFError，可以捕获并且退出子进程
            break

if __name__ == '__main__':
    out_pipe, in_pipe = Pipe(True) # 新建一个Pipe(duplex)的时候，如果duplex为True，那么创建的管道是双向的；如果duplex为False，那么创建的管道是单向的。
    son_p = Process(target=son_process, args=(100, (out_pipe, in_pipe)))
    son_p.start()

    # pipe已被fork，关闭主进程的输出端
    out_pipe.close()
    # Pipe一端连接着主进程的输入，一端连接着子进程的输出口
    for x in range(1000):
        in_pipe.send(x)
    in_pipe.close()
    son_p.join()
    print "主进程结束"
```

##### queue

> Queue的使用主要是一边put(),一边get().但是Queue可以是多个Process 进行put操作，也可以是多个Process进行get()操作。

```python
from multiprocessing import Queue, Process
from Queue import Empty as QueueEmpty
import random

def getter(name, queue):
    print 'Son process %s' % name
    while True:
        try:
            value = queue.get(True, 10)
            # 取出数据 get(obj[, block[, timeout]])
            # block为True,就是如果队列中无数据了。
            # block 为False，如果队列中无数据，就抛出Queue.Empty异常
            # 若timeout是默认None，那么就会一直等下去，否则超时抛出Queue.Empty异常
            print "Process getter get: %f" % value
        except QueueEmpty:
            break

def putter(name, queue):
    print "Son process %s" % name
    for i in range(0, 1000):
        value = random.random()
        queue.put(value)
        # 放入数据 put(obj[, block[, timeout]])
        # 若block为True，队列是空的：
        # 若block是False，队列满了，直接抛出Queue.Full
        # 若timeout是默认None，那么就会一直等下去，否则超时抛出Queue.Full异常
        print "Process putter put: %f" % value

if __name__ == '__main__':
    queue = Queue()
    getter_process = Process(target=getter, args=("Getter", queue))
    putter_process = Process(target=putter, args=("Putter", queue))
    getter_process.start()
    putter_process.start()
```

补充记录同步问题的经典模型——生产者/消费者模型

> 有两个进程：一组生产者进程和一组消费者进程共享一个初始为空、固定大小为n的缓存（缓冲区）。生产者的工作是制造一段数据，只有缓冲区没满时，生产者才能把消息放入到缓冲区，否则必须等待，如此反复; 同时，只有缓冲区不空时，消费者才能从中取出消息，一次消费一段数据（即将其从缓存中移出），否则必须等待。由于缓冲区是临界资源，它只允许一个生产者放入消息，或者一个消费者从中取出消息。

问题的核心是：

* 要保证不让生产者在缓存还是满的时候仍然要向内写数据
* 不让消费者试图从空的缓存中取出数据。

解决思路：

- 对于生产者，如果缓存是满的就去睡觉。消费者从缓存中取走数据后就叫醒生产者，让它再次将缓存填满。若消费者发现缓存是空的，就去睡觉了。下一轮中生产者将数据写入后就叫醒消费者。
  不完善的解决方案会造成“死锁”，两个进程都在“睡觉”，等着对方来“唤醒”。

### 多线程

采用threading模块支持多线程编程，启动一个线程就是创建`Thread`实例，并把一个函数传入，然后调用`start()`开始执行。每个进程默认有一个主线程，主线程可以启动新的子线程。

```python
import time, threading

def loop():
    print('thread %s is running...' % threading.current_thread().name)
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)

print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='Thread') # 创建子线程，子线程执行loop函数，线程名字为Thread
t.start() # 启动线程
t.join()  # 等待线程结束
print('thread %s ended.' % threading.current_thread().name)
```

Python解释器由于设计时有GIL（Global Interpreter Lock）全局锁，任何Python线程执行前，必须先获得GIL锁，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行，所以多线程在Python中只能交替执行，导致了多线程无法利用多核，只能利用一个核心。

#### 锁

多线程最大的不同于多进程的地方在于，多进程间同一个变量在不同进程间各有拷贝互不影响，多线程中所有变量都由所有线程共享，所以任何一个变量都能被任意一个线程修改，当多个线程同时修改同一个变量的时候就会造成尴尬的情况——输出的结果有可能是不可预期的。所以必须确保某个线程修改一个变量的时候别的线程不能修改这个变量。

```python
balance = 0
lock = threading.Lock()
def change_it(n):
    global balance
    balance = balance + n
    balance = balance - n
    
def run_thread(n):
    for i in range(100000):
        # 先要获取锁:
        lock.acquire()
        try:
            # 有锁可以更改
            change_it(n)
        finally:
            # 改完了要释放锁:
            lock.release()

```

当多个线程同时lock.acquire()的时候，只有一个线程能获得锁，其他线程只能等到获得锁为止才能继续向下执行。获得锁的线程也必须在执行完后及时释放锁，否则别的线程将一直等待成为死进程。

#### ThreadLocal

线程可以保有局部变量，但是在函数调用的时候每层都得传参。线程处理不同的对象，而使用全局变量会导致所有线程共享一个变量。ThreadLocal本地线程对象可以帮助给每个线程保存变量状态。

```python
import threading

# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Ashe',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```

> 全局变量`local_school`就是一个`ThreadLocal`对象，每个`Thread`对它都可以读写`student`属性，但互不影响。你可以把`local_school`看成全局变量，但每个属性如`local_school.student`都是线程的局部变量，可以任意读写而互不干扰，也不用管理锁的问题，`ThreadLocal`内部会处理。
>
> `ThreadLocal`最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

## 网络编程

网络编程涉及的通信协议大致为tcp和udp两种协议。

### tcp

tcp是面向连接点对点的协议，大多数连接都是TCP连接。创建一个TCP连接要经过三次握手，断开一个TCP连接需要四次挥手，主动发起连接的叫客户端，被动响应连接的叫服务器。

#### 创建一个tcp客户端

```python
import socket

# 创建一个socket:
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # AF_INET为ipv4，AF_INET6为ipv6，SOCK_STREAM为面向流的tcp协议
# 建立连接:
s.connect(('blog.xcola.top', 80))
# 发送数据:
s.send(b'GET / HTTP/1.1\r\nHost: blog.xcola.top\r\nConnection: close\r\n\r\n')
# 接收数据:
buffer = []
while True:
    # 每次最多接收1k字节:
    d = s.recv(1024)
    if d:
        buffer.append(d)
    else:
        break
data = b''.join(buffer)
# 关闭连接:
s.close()
header, html = data.split(b'\r\n\r\n', 1)
print(header.decode('utf-8'))
# 把接收的数据写入文件:
with open('sina.html', 'wb') as f:
    f.write(html)
```

#### 创建一个tcp服务器

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 监听端口:
s.bind(('0.0.0.0', 4000))
s.listen(5)

def tcplink(sock, addr):
    print('Accept new connection from %s:%s...' % addr)
    sock.send(b'Welcome!')
    while True:
        data = sock.recv(1024)
        time.sleep(1)
        if not data or data.decode('utf-8') == 'exit':
            break
        sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
    sock.close()
    print('Connection from %s:%s closed.' % addr)
    
while True:
    # 接受一个新连接:
    sock, addr = s.accept()
    # 创建新线程来处理TCP连接:
    t = threading.Thread(target=tcplink, args=(sock, addr))
    t.start()
```



### udp

udp是无连接可以一对一一对多多对一多对多的协议，只保证把数据包尽可能的丢出去，不关心数据包能不能到也不关心数据包到达的顺序，尽最大努力交付，但不保证可靠交付。

#### 创建一个udp客户端

```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

for data in [b'Genji', b'76', b'Hanzo']:
    # 发送数据:
    s.sendto(data, ('127.0.0.1', 4000))
    # 接收数据:
    print(s.recv(1024).decode('utf-8'))
s.close()
```



#### 创建一个udp服务器

```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 绑定端口:
s.bind(('0.0.0.0', 4000))

while True:
    # 接收数据:
    data, addr = s.recvfrom(1024) # recvfrom返回数据和客户端的地址与端口
    print('Received from %s:%s.' % addr)
    s.sendto(b'Hello, %s!' % data, addr)
```



学习过程参阅了[廖雪峰的python教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)以及几个SegmentFault上的回答。

