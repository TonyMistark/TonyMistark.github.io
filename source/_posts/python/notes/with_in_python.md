---
layout: python
title: 详解：python中的with
categories:
  - python
date: 2024-10-28 18:49:46
---

## 详解: python中的with
一、基本概念与语法结构
在Python中，with语句是一种用于简化资源管理的控制流语句，它的语法结构如下：
```python
with context_expression [as target(s)]: 
   ... with - body...
```
其中，context_expression可以是任意表达式，它的结果应该是一个支持上下文管理协议（Context Management Protocol）的对象。as target(s)是可选的部分，如果存在，with语句执行时会将__enter__()方法的返回值绑定到as后面指定的目标（可以是一个或多个变量）。

例如，在文件操作中，我们经常这样使用with：
```python
with open('example.txt',  'r') as f: 
    data = f.read() 
```
这里open('example.txt',  'r')就是context_expression，它返回一个文件对象（支持上下文管理协议），f就是as后面的目标变量，在with - body（这里是data = f.read() ）中就可以使用这个文件对象进行操作。

二、支持的对象类型
很多对象都可以支持with语句，只要它们实现了上下文管理协议，也就是定义了__enter__()和__exit__()方法。

* 文件对象

文件对象是最常见的支持with语句的对象类型。当使用open()函数打开一个文件时，它返回的文件对象就可以直接用于with语句中。这是因为文件操作需要在使用完毕后关闭文件以释放资源，with语句可以确保无论在操作文件过程中是否发生异常，文件都会被正确关闭。例如：
```python
try: 
    f = open('test.txt',  'w')
    f.write('Hello,  World!')
finally: 
    f.close() 
```
这种写法比较繁琐，而使用with语句就简洁得多：

```python
with open('test.txt',  'w') as f: 
    f.write('Hello,  World!')
```

* 锁对象（如在多线程编程中的锁）

在多线程编程中，为了防止多个线程同时访问共享资源而导致数据不一致等问题，我们会使用锁。with语句可以用于自动获取和释放锁，例如在threading模块中的Lock对象：
```python
import threading
lock = threading.Lock()
def thread_function(): 
    with lock: 
        # 这里是对共享资源的操作，只有获取到锁的线程才能执行
        print('Thread is working') 
```
这里，with lock会自动获取锁，当with - body中的代码执行完毕后，会自动释放锁，避免了手动获取和释放锁时可能出现的忘记释放锁的问题。

* 自定义对象

我们也可以自定义支持with语句的对象。例如：
```python
class MyContextManager: 
    def __enter__(self): 
        print('Entering the context')
        return self
    def __exit__(self, exc_type, exc_value, traceback): 
        print('Exiting the context')
with MyContextManager() as cm: 
    print('Inside the with block')
```
在这个例子中，MyContextManager类定义了__enter__()和__exit__()方法，所以可以用于with语句。__enter__()方法在进入with块时被调用，__exit__()方法在退出with块时被调用，无论是否发生异常。

## Python with工作原理
一、上下文管理协议的核心方法
* __enter__()方法
当执行with语句时，首先会调用context_expression所返回对象的__enter__()方法。这个方法的主要作用是进行一些初始化操作，并返回一个值（如果有as target(s)部分，这个值会被绑定到目标变量上）。例如在文件对象中，__enter__()方法可能会打开文件并返回文件对象本身。
以一个自定义的上下文管理器类为例：
```python
class MyContext: 
    def __enter__(self): 
        print('__enter__ method is called')
        return 'This is the return value'
    def __exit__(self, exc_type, exc_value, traceback): 
        print('__exit__ method is called')
with MyContext() as result: 
    print(result)
在这个例子中，当执行with MyContext()时，__enter__()方法被调用，它打印出__enter__ method is called并返回This is the return value，这个返回值被绑定到result变量上，然后在with - body中打印出来。
* __exit__()方法
__exit__()方法在with - body执行完毕后被调用，无论with - body中的代码是正常执行完毕还是发生了异常。它有四个参数：exc_type（异常类型，如果没有异常则为None）、exc_value（异常值，如果没有异常则为None）、traceback（异常的追溯信息，如果没有异常则为None）。
它的主要功能是进行一些清理操作，比如关闭文件、释放锁等。如果在__exit__()方法中返回True，那么with - body中发生的异常会被抑制，不会向上传播；如果返回False或者None（默认），异常会正常向上传播。例如：
```python
class SuppressError: 
    def __enter__(self): 
        pass
    def __exit__(self, exc_type, exc_value, traceback): 
        if exc_type is not None: 
            print('Suppressing the error:', exc_type)
            return True
with SuppressError(): 
    1/0
```
在这个例子中，__exit__()方法检查到发生了ZeroDivisionError异常（exc_type不为None），然后打印出抑制错误的信息并返回True，所以这个异常不会导致程序崩溃。
二、与try - finally的等价关系
正常执行情况
在语义上，with语句等价于一个包含try - finally结构的代码块。例如：
```python
try: 
    context_manager = context_expression
    value = type(context_manager).__enter__(context_manager)
    try: 
        target = value
        # with - body 
    finally: 
        type(context_manager).__exit__(context_manager, None, None, None)
except: 
    pass
```
当with - body正常执行时，__enter__()方法首先被调用，然后执行with - body中的代码，最后__exit__()方法被调用且传入的exc_type、exc_value、traceback都为None。
* 发生异常情况
如果在with - body中发生了异常，__exit__()方法会被调用，并且传入异常的类型、值和追溯信息。如果__exit__()方法返回True，异常被抑制，就好像在try - finally结构中在finally块中处理了异常并且不再向上传播；如果__exit__()方法返回False或者None，异常会像在普通的try - except结构中一样向上传播，由外部的异常处理机制来处理。例如：
```python
try: 
    context_manager = context_expression
    value = type(context_manager).__enter__(context_manager)
    try: 
        target = value
        # with - body中发生异常
        raise ValueError('This is an error')
    except: 
        exc = False
        if not type(context_manager).__exit__(context_manager, *sys.exc_info()):  
            raise
        else: 
            exc = True
    finally: 
        if exc: 
            type(context_manager).__exit__(context_manager, None, None, None)
```
## Python with常见应用场景举例
一、文件操作
* 读取文件
在读取文件时，使用with语句可以确保文件在读取完毕后被正确关闭，即使在读取过程中发生了异常。例如：
```python
with open('data.txt',  'r') as f: 
    lines = f.readlines() 
    for line in lines: 
        print(line.strip()) 
```
如果不使用with语句，我们需要使用try - finally结构来确保文件关闭：
```python
f = open('data.txt',  'r')
try: 
    lines = f.readlines() 
    for line in lines: 
        print(line.strip()) 
finally: 
    f.close() 
```
很明显，使用with语句的代码更加简洁。
* 写入文件
同样，在写入文件时，with语句也很方便。例如：
```python
with open('output.txt',  'w') as f: 
    f.write('This  is a test.\n')
```
这里，如果写入过程中出现问题（比如磁盘空间不足等），__exit__()方法会被调用，文件会被关闭，避免资源泄漏。
二、数据库连接
* 连接数据库并执行查询
在使用数据库连接时，如sqlite3数据库，我们可以这样使用with语句：
```python
import sqlite3
with sqlite3.connect('example.db')  as conn: 
    cursor = conn.cursor() 
    cursor.execute('SELECT  * FROM users')
    results = cursor.fetchall() 
    for row in results: 
        print(row)
```
当with - body执行完毕后，数据库连接会被正确关闭，释放相关资源。如果在查询过程中发生异常（比如数据库文件损坏等），__exit__()方法也会被调用进行清理操作。
* 事务处理
在数据库事务处理中，with语句也很有用。例如：
```python
with sqlite3.connect('example.db')  as conn: 
    cursor = conn.cursor() 
    try: 
        cursor.execute('BEGIN  TRANSACTION')
        cursor.execute('INSERT  INTO users (name, age) VALUES (?,?)', ('John', 30))
        cursor.execute('COMMIT') 
    except: 
        cursor.execute('ROLLBACK') 
```
这里，with语句确保了数据库连接的正确管理，而在with - body中通过try - except结构处理事务的提交和回滚，如果没有异常则提交事务，如果发生异常则回滚事务。
三、多线程中的资源锁定
* 线程安全的计数器
考虑一个多线程环境下的计数器的例子。我们使用threading.Lock来确保计数器的操作是线程安全的：
```python
import threading
counter = 0
lock = threading.Lock()
def increment(): 
    global counter
    with lock: 
        counter += 1
threads = []
for _ in range(10): 
    t = threading.Thread(target = increment)
    threads.append(t) 
    t.start() 
for t in threads: 
    t.join() 
print('Counter value:', counter)
```
在increment函数中，with lock会自动获取锁，使得只有一个线程能够执行counter += 1这个操作，从而保证了计数器在多线程环境下的正确性。当线程完成操作后，锁会被自动释放。
* 共享资源的访问控制
假设我们有一个共享的列表，多个线程可能会对其进行读写操作。我们可以使用with语句和锁来确保共享资源的正确访问：
```python
import threading
shared_list = []
lock = threading.Lock()
def add_to_list(item): 
    with lock: 
        shared_list.append(item) 
def remove_from_list(): 
    with lock: 
        if shared_list: 
            return shared_list.pop() 
threads = []
# 创建一些添加元素的线程
for i in range(5): 
    t = threading.Thread(target = add_to_list, args = (i,))
    threads.append(t) 
    t.start() 
# 创建一些删除元素的线程
for _ in range(3): 
    t = threading.Thread(target = remove_from_list)
    threads.append(t) 
    t.start() 
for t in threads: 
    t.join() 
print('Shared list:', shared_list)
```
这里，add_to_list和remove_from_list函数中的with lock确保了在对共享列表进行操作时的线程安全性。
## Python with与其他类似机制的比较
一、与try - finally的比较
* 语法简洁性
with语句比try - finally更加简洁。例如在文件操作中，使用try - finally来确保文件关闭的代码如下：
```python
f = open('test.txt',  'r')
try: 
    data = f.read() 
    # 其他操作
finally: 
    f.close() 
而使用with语句则是：
with open('test.txt',  'r') as f: 
    data = f.read() 
    # 其他操作
```
可以明显看出，with语句不需要显式地编写try和finally块，减少了代码的冗余。
* 异常处理逻辑
在try - finally中，如果想要在finally块中处理异常并根据情况决定是否重新抛出异常，逻辑会比较复杂。而在with语句中，通过__exit__()方法可以很方便地处理异常。例如：
```python
class MyContext: 
    def __enter__(self): 
        pass
    def __exit__(self, exc_type, exc_value, traceback): 
        if exc_type is not None: 
            print('Caught an exception in __exit__:', exc_type)
            # 根据情况返回True或False来决定是否抑制异常
            return False
with MyContext(): 
    raise ValueError('This is an error')
```
在这个with语句的例子中，__exit__()方法可以直接处理异常，而在try - finally中，需要在finally块中额外的逻辑来处理异常并决定是否重新抛出。
* 代码可读性
with语句的代码可读性更高，因为它将资源管理和操作代码更加紧密地结合在一起。try - finally结构中，资源管理（finally块）和正常操作（try块）的代码是分开的，对于复杂的逻辑，可能会使代码的理解变得困难。例如在数据库连接的操作中，with语句使得连接的获取、操作和释放更加直观：
```python
with sqlite3.connect('db.sqlite')  as conn: 
    cursor = conn.cursor() 
    cursor.execute('SELECT  * FROM table')
```
而使用try - finally结构会使代码的结构更加分散。
二、与装饰器的比较（在资源管理方面）
* 功能侧重点
装饰器主要用于在不修改原函数代码的情况下，对函数的功能进行扩展，如添加日志记录、性能检测等功能。而with语句主要用于资源管理，确保资源的正确获取和释放。例如，一个简单的日志记录装饰器：
```python
def log_decorator(func): 
    def wrapper(*args, **kwargs): 
        print(f'Calling function {func.__name__}')
        result = func(*args, **kwargs)
        print(f'Function {func.__name__} finished')
        return result
    return wrapper
@log_decorator
def my_function(): 
    print('This is my function')
my_function()
```
这个装饰器主要是在函数调用前后添加了日志记录功能，与资源管理没有直接关系。而with语句则专注于资源的生命周期管理，如文件、数据库连接等资源。
* 使用方式
装饰器是通过在函数定义前添加@符号和装饰器函数名来使用的，它会自动包装被装饰的函数。而with语句是在需要管理资源的代码块前直接使用的。例如，对于一个需要在函数执行前后进行资源管理的情况，如果使用装饰器，需要定义一个专门的装饰器来处理资源管理，而使用with语句则可以直接在函数内部相关代码块使用：
```python
def resource_heavy_function(): 
    with open('data.txt',  'r') as f: 
        data = f.read() 
    # 对读取的数据进行操作
```
如果使用装饰器来实现类似的资源管理，会更加复杂，需要更多的代码来处理资源的获取、释放和函数的执行逻辑。
## Python with错误处理方法
一、__exit__()方法中的异常处理
接收异常信息
当with - body中发生异常时，__exit__()方法会接收到异常的类型（exc_type）、异常的值（exc_value）和异常的追溯信息（traceback）。例如：
```python
class ErrorHandler: 
    def __enter__(self): 
        pass
    def __exit__(self, exc_type, exc_value, traceback): 
        if exc_type is not None: 
            print(f'Caught an exception: {exc_type}, {exc_value}')
with ErrorHandler(): 
```
