.. _toctree-directive:

Python 的闭包和装饰器
=====================

翻译： `TheLover_Z <http://zhuang13.de>`_

Part I
------

原文地址： `<http://blaag.haard.se/Python-Closures-and-Decorators--Pt--1/>`_

回想起来，当初我做出了错误的选择，把 Python 的课程削减到了4个小时以至于把装饰器的部分搞砸了，我答应大家我稍后会对闭包和装饰器做一个更好的解说 —— 我是这么打算的。

函数也是对象。实际上，在 Python 中函数是一级对象——也就是说，他们可以像其他对象一样使用而没有什么特别的限制。这给了我们一些有趣的选择，我会由浅到深解释这个问题。

关于函数就是对象的一个最常见的例子就是 C 中的函数指针；将函数传递到其他的将要使用它的函数。为了说明这一点，我们来看看一个重复函数的实现 —— 也就是，一个函数接受另外一个函数以及一个数字当作参数，并且重复调用指定函数指定次数：

::
    
    >>> #A very simple function
    >>> def greeter():
    …     print("Hello")
    … 
    >>> #An implementation of a repeat function
    >>> def repeat(fn, times):
    …     for i in range(times):
    …         fn()
    … 
    >>> repeat(greeter, 3)
    Hello
    Hello
    Hello
    >>>

这种模式在很多情况下都有用 —— 比如向一个排序算法传递比较函数，向一个语法分析器传递一个装饰器函数，通常情况下这些做法可以使一个函数的行为 **更专一化** ，或者向已经抽象了工作流的函数传递一个待办的特定部分（比如， ``sort()`` 知道怎么排序， ``compare()`` 知道怎么比较元素）。

函数也可以在其他函数的内部声明，这给了我们另一个很重要的工具。在一般情况下，这可以用来隐藏实用函数的实现细节：

::
    
    >>> def print_integers(values):
    …     def is_integer(value):
    …         try:
    …             return value == int(value)
    …         except:
    …             return False
    …     for v in values:
    …         if is_integer(v):
    …             print(v)
    … 
    >>> print_integers([1,2,3,"4", "parrot", 3.14])
    1
    2
    3

这可能是有用的，但它本身并不算是个强大的工具。相比函数可以当作参数被传递而言，我们可以将它们包装（wrap）在另外的函数中，从而向已经构建好的函数增加新的行为。一个简单的例子是向一个函数增加跟踪输出：

::
    
    >>> def print_call(fn):
    …     def fn_wrap(*args, **args): #take any arguments
    …         print ("Calling %s" % (fn.func_name))
    …         return fn(*args, **kwargs) #pass any arguments to fn()
    …     return fn_wrap
    … 
    >>> greeter = print_call(greeter) #wrap greeter
    >>> repeat(greeter, 3)
    Calling fn_wrap
    Hello
    Calling fn_wrap
    Hello
    Calling fn_wrap
    Hello
    >>>
    >>> greeter.func_name
    'fn_wrap'

正如你看到的那样，我们可以使用带日志的函数来替换掉现有函数相应的部分，然后调用原来的函数。在例子的最后两行，函数的名字已经反映出了它已经被改变，这个改变可能是我们想要的，也可能不是。如果我们想包装一个函数同时保持它原来的名字，我们可以增加一行 ``print_call`` 函数，代码如下：

::
    
    >>> def print_call(fn):
    …     def fn_wrap(*args, **kwargs): #take any arguments
    …         print("Calling %s" % (fn.func_name))
    …         return fn(*args, **kwargs) #pass any arguments to fn()
    …     fn_wrap.func_name = fn.func_name #Copy the original name
    …     return fn_wrap

因为这是一个很长的话题，我明天会来更新第二部分，我们会讲讲闭包，偏函数（partial），还有（终于到它了）装饰器。

至此，如果这些你之前全部没有接触过，可以先用 ``print_call`` 函数作为基础，来创建一个能够在正常调用函数之前先打印出这个函数名字的一个修饰器。

Part II
-------

原文地址： `<http://blaag.haard.se/Python-Closures-and-Decorators--Pt--2/>`_

在第一部分中，我们学习了以函数作为参数调用其他的函数，还有嵌套函数，最终我们把一个函数包装在另外的函数中。我们先把第一部分的答案给出：

::
    
    >>> def print_call(fn):
    …     def fn_wrap(*args, **kwargs):
    …         print("Calling %s with arguments: \n\targs: %s\n\tkwargs:%s" %fn.__name__, args, kwargs))
    …         retval = fn(*args, **kwargs)
    …         print("%s returning '%s'" % (fn.func_name, retval))
    …         return retval
    …     fn_wrap.func_name = fn.func_name
    …     return fn_wrap
    …
    >>> def greeter(greeting, what='world'):
    …     return "%s %s!" % (greeting, what)
    …
    >>> greeter = print_call(greeter)
    >>> greeter("Hi")
    Calling greeter with arguments:
        args: ('Hi',)
        kwargs:{}
    greeter returning 'Hi world!'
    'Hi world!'
    >>> greeter("Hi", what="Python")
    Calling greeter with arguments:
        args: ('Hi',)
        kwargs:{'what': 'Python'}
    greeter returning 'Hi Python!'
    'Hi Python!'
    >>>

这稍微有那么点儿用了，但它可以变的更好！你可能听说过或者没有听说过*闭包*，你可能听说过成千上万种闭包定义中的某一种或者某几种 —— 我不会那么挑剔，我只是说闭包就是一个捕捉了（或者关闭）非本地变量（自由变量）的代码块（比如一个函数）。如果你不清楚我在说什么，你可能需要进修一下 CS 的相关课程，但是不要担心 —— 我会给你演示例子。闭包的概念很简单：一个可以引用在函数闭合范围内变量的函数。

比如说，看一下这个代码：

::
    
    >>> a = 0
    >>> def get_a():
    …     return a
    …
    >>> get_a()
    0
    >>> a = 3
    >>> get_a()
    3

正如你看到的那样， ``get_a`` 函数可以取得 ``a`` 的值，并且可以读取更新后的值。然而这里有一个限制 —— 被捕获的变量（captured variable，下同）不能被写入。

::
    
    >>> def set_a(val):
    …     a = val
    …
    >>> set_a(4)
    >>> a
    3

为什么会这样？由于闭包不能写入任何被捕获的变量， ``a = val`` 这个语句实际上写入了本地变量 ``a`` 从而隐藏了模块级别的 ``a`` ，这正是我们想写入的内容。为了解决这个限制（也许这并不是一个好主意），我们可以用一个容器类型：

::
    
    >>> class A(object): pass
    …
    >>> a = A()
    >>> a.value = 1
    >>> def set_a(val):
    …     a.value = val
    …
    >>> a.value
    1
    >>> set_a(5)
    >>> a.value
    5

因此，我们已经知道了函数从它的闭合范围内捕捉变量，我们最终可以接触到有趣的东西了，我们先实现一个偏函数（partial，下同）。一个偏函数是一个你已经填充了部分或者全部参数的函数的实例；比如说你有一个存储了用户名和密码的会话，和一个查询后端的函数，这个函数有不同的参数但是*总是*需要身份验证。与其说每次都手动传递身份验证信息，我们可以用偏函数来预填充那些信息。

::
    
    >>> #Our 'backend' function
    … def get_stuff(user, pw, stuff_id):
    …     """Here we would presumably fetch data using the
    …     credentials and id"""
    …     print("get_stuff called with user: %s, pw: %s, stuff_id: %s" % (user, pw, stuff_id))
    >>> def partial(fn, *args, **kwargs):
    …     def fn_part(*fn_args, **fn_kwargs):
    …         kwargs.update(fn_kwargs)
    …         return fn(*args + fn_args, **kwargs)
    …     return fn_part
    …
    >>> my_stuff = partial(get_stuff, 'myuser', 'mypwd')
    >>> my_stuff(3)
    get_stuff called with user: myuser, pw: mypwd, stuff_id: 3
    >>> my_stuff(67)
    get_stuff called with user: myuser, pw: mypwd, stuff_id: 67

偏函数可以用在许多地方来消除代码的重复。当然，你没有必要自己手动实现它，只需要 ``from functools import partial`` 就可以了。

最后，我们来看看函数装饰器（未来可能有类装饰器）。函数装饰器接收一个函数作为参数然后返回一个新的函数。听起来很熟悉吧？我们已经实现过一个 ``print_call`` 装饰器了。

::
    
    >>> @print_call
    … def will_be_logged(arg):
    …     return arg*5
    …
    >>> will_be_logged("!")
    Calling will_be_logged with arguments:
        args: ('!',)
        kwargs:{}
    will_be_logged returning '!!!!!'
    '!!!!!'

使用@符号标记是一个很方便的方法。

::
    
    >>> def will_be_logged(arg):
    …     return arg*5
    …
    >>> will_be_logged = print_call(will_be_logged)

但是如果我们想要确定装饰器的参数呢？在这种情况下，作为装饰器的函数会接收参数，并且返回一个包装（wrap）了装饰器函数的函数。

::
    
    >>> def require(role):
    …     def wrapper(fn):
    …         def new_fn(*args, **kwargs):
    …             if not role in kwargs.get('roles', []):
    …                 print("%s not in %s" % (role, kwargs.get('roles', [])))
    …                 raise Exception("Unauthorized")
    …             return fn(*args, **kwargs)
    …         return new_fn
    …     return wrapper
    …
    >>> @require('admin')
    … def get_users(**kwargs):
    …     return ('Alice', 'Bob')
    …
    >>> get_users()
    admin not in []
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 7, in new_fn
    Exception: Unauthorized
    >>> get_users(roles=['user', 'editor'])
    admin not in ['user', 'editor']
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 7, in new_fn
    Exception: Unauthorized
    >>> get_users(roles=['user', 'admin'])
    ('Alice', 'Bob')

就是这样。你现在会写装饰器了，也许你会用这些知识去写面向方面（aspect-oriented）的编程。加入 ``@cache``, ``@trace``, ``@throttle`` 都是微不足道的（在你添加 ``@cache`` 之前，一定要检查 ``functools`` ，如果你用的是 Python 3 的话！）
