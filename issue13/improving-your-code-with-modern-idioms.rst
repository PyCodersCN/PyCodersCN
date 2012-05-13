使用现代风格改善你的代码
============================

原文： `<http://python3porting.com/improving.html>`_

译者： `TheLover_Z <http://zhuang13.de>`_ 

一旦你开始使用 Python 3，你就有机会接触新的特性来改善你的代码。这篇文章中提到的很多东西实际上在 Python 3 之前就已经被支持了。但我还是要提一下它们，因为知道了这些以后你的代码可以从中获益。我说的包括修饰器，在 Python 2.2 开始提供支持； ``sorted()`` 方法，在 Python 2.4 开始提供支持；还有上下文管理，在 Python 2.5 开始提供支持。

这里提及的其它新特性在 Python 2.6 或者 2.7 都提供了支持，所以说如果你不是在用 Python 2.5 和之前的版本的话，你可以使用这里提到的几乎全部的新特性。

使用 ``sorted()`` 来替代 ``.sort()``
--------------------------------------

在 Python 中，列表有一个 ``.sort()`` 方法可以进行排序。 ``.sort()`` 会影响列表的结构。下面这么写是因为在 Python 2.3 之前只能这么写。

::

    >>> infile = open('pythons.txt')
    >>> pythons = infile.readlines()
    >>> pythons.sort()
    >>> [x.strip() for x in pythons]
    ['Eric', 'Graham', 'John', 'Michael', 'Terry', 'Terry']

Python 2.4 开始加入了新的支持 ``sorted()`` ，它会返回一个排好序的列表并且接受和 ``.sort()`` 一样的参数。使用 ``sorted()`` 你可以避免改变列表的结构。它还可以接受迭代器作为输入而不只是列表，这样可以让你的代码看起来更棒。

::

    >>> infile = open('pythons.txt')
    >>> [x.strip() for x in sorted(infile)]
    ['Eric', 'Graham', 'John', 'Michael', 'Terry', 'Terry']

然而，如果你把 ``mylist.sort()`` 替换为 ``mylist = sorted(mylist)`` 是没有用的，而且还会消耗更多的内存。

``2to3`` 有时会把 ``.sort()`` 改为 ``sorted()`` 。

使用上下文管理器来编写代码
--------------------------

从 Python 2.5 开始你可以使用上下文管理器，它允许你创造和管理运行时内容。如果你觉得听起来有点儿抽象，那就对了。上下文管理器确实很抽象并且很灵活，很容易被误用，我这就教你怎么正确运用它。

上下文管理器被用来当作 ``with`` 的一部分，在 ``with`` 的代码块内都有效。在代码块结束的时候上下文管理器退出。这可能听起来不是那么令人激动，除非我告诉你你可以使用它来实现资源分配。你进入上下文的时候资源管理器分配资源，你退出的时候它释放资源。

最常用的例子是读写文件。在大多数的更面向底层的语言中你必须记得关闭已打开的文件，但在 Python 中你不需要这么做。然而有时候你必须确认你关掉了文件，比如说你在循环中打开了许多文件以至于你用完了文件名。

::

    >>> f = open('/tmp/afile.txt', 'w')
    >>> try:
    ...     n = f.write('sometext')
    ... finally:
    ...     f.close()

你也可以这么写，使用上下文管理器。

::

    >>> with open('/tmp/afile.txt', 'w') as f:
    ...     n = f.write('sometext')

当你使用上下文管理器的时候，代码块结束的时候文件就会自动关闭，就算是有错误发生也是这样。正如你所看到的那样，代码量少了很多，但是更重要的是程序看起来干净多了，也易读了。

另一个例子是如果你想要重定向标准输出。正如前面一样，你会使用 ``try/except`` 。那样也不错，如果你只使用一次的话。但是如果你有很多次这样的需求的话，上下文管理器是你不二的选择。

::

    >>> import sys
    >>> from StringIO import StringIO
    >>> class redirect_stdout:
    ...     def __init__(self, target):
    ...         self.stdout = sys.stdout
    ...         self.target = target
    ...
    ...     def __enter__(self):
    ...         sys.stdout = self.target
    ...
    ...     def __exit__(self, type, value, tb):
    ...         sys.stdout = self.stdout
    ...
    >>> out = StringIO()
    >>> with redirect_stdout(out):
    ...     print 'Test'
    ...
    >>> out.getvalue() == 'Test\n'
    True

碰到 ``with`` 语句以后 ``__enter__`` 方法被调用，退出的时候 ``__exit__()`` 被调用，包括引发错误。

上下文管理器在很多地方都可以使用。你的任何使用例外的代码最好确保资源或者全局变量没有被分配或者设置。

``contextlib`` 库有各种各样的函数帮助你使用上下文管理器。比如说，如果你有一个有 ``.close()`` 方法但不是上下文管理器的对象，你可以使用 ``closing()`` 函数来在 ``with`` 块结束的时候自动关闭它们。

::

    >>> from contextlib import closing
    >>> import urllib
    >>> 
    >>> book_url = 'http://python3porting.com/'
    >>> with closing(urllib.urlopen(book_url)) as page:
    ...     print len(page.readlines())
    117

高级字符串格式化
-----------------

在 Python 3 和 2.6 中，一种新的字符串格式支持被引进了。它更灵活并且有更聪明的语法。

旧的字符串格式：

::

    >>> 'I %s Python %i' % ('like', 2)
    'I like Python 2'

新的字符串格式：

::

    >>> 'I {0} Python {1}'.format('♥', 3)
    'I ♥ Python 3'

使用这些新特性你可以实现一些比较疯狂的小东西，但是玩过火的话你旧失去了它易读的优点：

::

    >>> import sys
    >>> 'Python {0.version_info[0]:!<9.1%}'.format(sys)
    'Python 300.0%!!!'

更详细的文档请参考 `Common String Operations <http://docs.python.org/library/string.html#format-string-syntax>`_ 。

旧的字符串格式基于 ``%`` 的这个特性可能最终会被移除，不过最终日期还没有定。

类修饰器
-------------

修饰器在 Python 2.4 的时候被支持，然后有了内置的修饰器比如说 ``@property`` 和 ``@classmethod`` ，修饰器开始变的流行。Python 2.6 引入了类修饰器。

类修饰器可以用来包裹类或者修饰类。一个例子就是 ``functools.total_ordering`` ，可以让你实现最小的富比较操作符，然后增加到你的类。它们可以作为元类，类修饰器的例子就是修饰器可以把类变成一个单独的类。 ``zope.interface`` 类修饰器可以注册一个作为特定接口的类。

集合
----------

Python 3 中引入了一种新的集合语法。相对于 ``set([1, 2, 3])`` 你可以使用更干净语法的 ``{1, 2, 3}`` 。两种语法在 Python 3 中都可以工作，但是更建议使用新的语法。

::

    >>> set([1,2,3])
    {1, 2, 3}

``yield`` 和 生成器
-----------------------

就像浮点除法操作符和 ``.sort()`` 的 ``key`` 参数，生成器已经在不知不觉深入了我们的编码生活。虽然不多见，但它们还是非常实用的，可以帮你节省内存，简化代码。我们来看看这个例子：

::

    >>> def allcombinations(starters, endings):
    ...    result = []
    ...    for s in starters:
    ...         for e in endings:
    ...             result.append(s+e)
    ...     return result

这么写就优雅多了：

::

    >>> def allcombinations(starters, endings):
    ...     for s in starters:
    ...         for e in endings:
    ...             yield s+e

生成器在 Python 2.2 开始加入支持，但是 Python 2.4 进行了一些改进。看起来很像是列表表达式，但并不返回列表而是返回表达式。它们在有列表表达式的地方几乎都可以使用。

::

    >>> sum([x*x for x in xrange(2000000)])
    2666664666667000000L

可以写作：

::

    >>> sum(x*x for x in xrange(2000000))
    2666664666667000000L

更多的推导式
-------------

在 Python 3 和 2.6 中，生成器推导式被引进。它就是简单的一个带括号的生成器表达式，可以和列表推导式一样工作，返回一个生成器而不是列表。

::

    >>> (x for x in 'Silly Walk')
    <generator object <genexpr> at ...>

在 Python 3 中生成器推导式不仅仅是一个新的漂亮的特性，而是一个重要的改变，因为生成器推导式现在是其它所有内置推导式的基础。在 Python 3 中列表推导式只是一个给 ``list`` 类型的构造器提供生成器表达式的语法糖。

::

    >>> list(x for x in 'Silly Walk')
    ['S', 'i', 'l', 'l', 'y', ' ', 'W', 'a', 'l', 'k']

    >>> [x for x in 'Silly Walk']
    ['S', 'i', 'l', 'l', 'y', ' ', 'W', 'a', 'l', 'k']

这也意味着循环变量再也不会掺入附近的命名空间了。

生成器推导式也可以用 Python 2.6 及其以后版本的 ``dict()`` 和 ``set()`` 构造器生成。但是在 Python 3 还有 Python 2.7 中，你可以用新的语法来定义字典和列表推导式：

::

    >>> department = 'Silly Walk'
    >>> {x: department.count(x) for x in department}
    {'a': 1, ' ': 1, 'i': 1, 'k': 1, 'l': 3, 'S': 1, 'W': 1, 'y': 1}

    >>> {x for x in department}
    {'a', ' ', 'i', 'k', 'l', 'S', 'W', 'y'}

新的模块
----------

还有许多新的模块值得你一看。在这里我就不多说了，因为大多数如果你不重写软件的话可能获益不多，但你应该知道它们存在。你可以翻看一下 Python 文档来了解一下。

``abc`` 
~~~~~~~~~~~~~~~~

``abc`` 模块包含了对生成抽象的基础类的支持，你可以 **标记** 一个基础类的方法或者属性为“抽象”，意思是你必须在子类中进行实现，否则无法实例化。

抽象基础类也可以创建没有实体方法的类，用于定义接口。

``abc`` 模块在 Python 2.6 及其以后的版本被支持。

``multiprocessing`` 和 ``future``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``multiprocessing`` 是一个新的模块，用于进行多进程操作，它允许你拥有进程队列和使用锁，还有用于同步进程的 `信号标 <http://zh.wikipedia.org/wiki/%E4%BF%A1%E8%99%9F%E6%A8%99>`_ 。

``multiprocessing`` 在 Python 2.6 以后被加入支持。在 2.4 和 2.5 你可以使用 `CheeseShop <http://pypi.python.org/pypi/multiprocessing>`_ 。

如果你要做并发你可以看一下 ``future`` 模块，在 Python 3.2 引入了这个模块，在 Python 2.5 及以后的版本可以用 `参考这里 <http://pypi.python.org/pypi/futures/>`_ 。

``numbers`` 和 ``fractions``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python 3 加入了这个库。大多数情况下你不会注意到它，但是很有趣的是 ``fractions`` 模块，在 Python 2.6 被支持。

::

    >>> from fractions import Fraction
    >>> Fraction(3,4) / Fraction('2/3')
    Fraction(9, 8)

还有 ``numbers`` 模块，包含支持所有数字类型的抽象基础类。如果你正在实现你自己的数字类型的话，那么它非常有用。


中英文对照
-----------

生成器推导式 - generator comprehension

列表推导式 － list comprehension

生成器 － generator

抽象的基础类 － abstract base classes
