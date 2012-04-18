Python装饰器入门
==================

原文: `A primer on Python decorators <http://www.thumbtack.com/engineering/a-primer-on-python-decorators/>`_

翻译: `youngsterxyf <http://xiayf.blogspot.com/>`_

Python允许你，作为程序员，使用函数完成一些很酷的事情。在Python中，函数是 `一等对象(first-class object) <http://en.wikipedia.org/wiki/First-class_function>`_ ，这就意味着你可以像使用字符串，整数，或者任何其他对象一样使用函数。例如，你可以将函数赋值给变量:

::

    >>> def square(n):
    ...     return n * n;
    >>> square(4)
    16
    >>> alias = square
    >>> alias(4)
    16

然而，一等函数的真正威力在于你可以把函数传给其他函数，或者从其他函数中返回函数。Python的内置函数map利用了这种能力：给map传个函数以及一个列表，它会依次以列表中每个元素为参数调用你传给它的那个函数，从而生成一个新的列表。如下所示的例子中应用了上面的那个square函数:

::

    >>> number = [1, 2, 3, 4, 5]
    >>> map(square, numbers)
    [1, 4, 9, 16, 25]

如果一个函数接受其他函数作为参数，以及/或者返回一个函数，那么它就被称为 `高阶函数 <http://en.wikipedia.org/wiki/Higher-order_function>`_ 。虽然map函数只是简单地使用了我们传给它的函数，而没有改变这个函数，但我们也可以使用高阶函数去改变其他函数的行为。

例如，假设有这样一个函数，会被调用很多次，以致运行代价非常昂贵:

::

   >>> def fib(n):
   ...      "Recursively (i.e., dreadfully) calculate the nth Fibonacci number."
   ...      return n if n in [0, 1] else fib(n - 2) + fib(n - 1)

我们一般会保存计算过程中每次递归调用的结果，这样，对于函数调用树中经常出现某个n，当需要计算n对应的结果时，就不需要重复计算了。有多种方式可以做到这点。例如，我们可以将这些结果存在一个字典中，当以某个值为参数调用fib函数时，就先到这个字典去查一下其结果是否已经计算出来了。

但这样的话，每次我们想要调用fib函数，都需要重复那段相同的字典检查样板式代码。相反，如果让fib函数自己在内部负责存储其结果，那么在其他代码中调用fib，就非常方便，只要简单地调用它就行了。这样一种技术被称为 `memoization <http://en.wikipedia.org/wiki/Memoization>`_ (注意没有字母r的哦)。

我们可以把这种memoization代码直接放入fib函数，但是Python为我们提供了另外一种更加优雅的选择。因为可以编写修改其他函数的函数，那么我们可以编写一个通用的memoization函数，以一个函数作为参数，并返回这个函数的memoization版本:

::

    def memoize(fn):
        stored_results = {}

        def memoized(*args):
            try:
                # try to get the cached result
                return stored_results[args]
            except KeyError:
                # nothing was cached for those args. let's fix that.
                result = stored_results[args] = fn(*args)
                return result
        return memoized

如上， ``memoize`` 函数以另一个函数作为参数，函数体中创建了一个字典对象用来存储函数调用的结果：键为被memoized包装后的函数的参数，值为以键为参数调用函数的返回值。 ``memoize`` 函数返回一个新的函数，这个函数会首先检查在 ``stored_results`` 字典中是否存在与当前参数对应的条目；如果有，对应的存储值会被返回；否则，就调用经过包装的函数，存储其返回值，并且返回给调用者。memoize返回的这种新函数常被称为"包装器"函数，因为它只是另外一个真正起作用的函数外面的一个薄层。

很好，现在有了一个memoization函数，我们可以把fib函数传给它，从而得到一个经过包装的fib，这个版本的fib函数不需要重复以前那样的繁重工作:

::

    def fib(n):
        return n if n in [0, 1] else fib(n - 2) + fib(n - 1)
    fib = memoize(fib)

通过高阶函数memoize，我们获得了memoization带来的好处，并且不需要对fib函数自己做出任何改变，以免夹杂着memoization的代码而模糊了函数的实质工作。但是，你也许注意到上面的代码还算有点别扭，因为我们必须写3遍fib。由于这种模式-传递一个函数给另一个函数，然后将结果返回给与原来那个函数同名的函数变量-在使用包装器函数的代码中极为常见，Python为其提供了一种特殊的语法：装饰器:

::

    @memoize
    def fib(n):
        return n if n in [0, 1] else fib(n - 2) + fib(n -1)

这里，我们说memoize函数装饰了fib函数。需要注意的是这仅是一种语法上的简便写法(译注：就是我们常说的"语法糖")。这段代码与前面的代码片段做的是同样的事情：定义一个名为fib的函数，把它传给memoize函数，将返回结果存为名为fib的函数变量。特殊的(看起来有点奇怪的)@语法只是减少了冗余。

你可以将多个装饰器堆叠起来使用，它们会自底向上地逐个起作用。例如，假设我们还有另一个用来帮助调试的高阶函数:

::

    def make_verbose(fn):
        def verbose(*args):
            # will print (e.g.) fib(5)
            print '%s(%s)' % (fb.__name__, ', '.join(repr(arg) for arg in args))
            return fn(*args)   # actually call the decorated function

        return verbose

下面的两个代码片段做的是同样的事情:

::

    @memoize
    @make_verbose
    def fib(n):
        return n if n in [0, 1] else fib(n - 2) + fib(n - 1)

::

    def fib(n):
        return n if n in [0, 1] else fib(n - 2) + fib(n - 1)
    fib = memoize(make_verbose(fib))

有趣的是，Python并没有限制你在@符号后只能写一个函数名：你也可以调用一个函数，从而能够高效地传递参数给装饰器。假设我们并不满足于简单的memoization，还想将函数的结果存储到 `memcached <http://memcached.org/>`_ 中。如果你已经写了一个 ``memcached`` 装饰器函数，那么可以(例如)传递一个服务器地址给它:

::

    @memcached('127.0.0.1:11211')
    def fib(n):
        return n if n in [0, 1] else fib(n - 2) + fib(n - 1)

非装饰器语法的写法会如下展开:

::

    fib = memcached('127.0.0.1:11211')(fib)

Python配备有一些作为装饰器使用的非常有用的函数。例如，Python有一个 ``classmethod`` 函数，可以创建大致类似于java的静态方法:

::

    class Foo(object):
        SOME_CLASS_CONSTANT = 42

        @classmethod
        def add_to_my_constant(cls, value):
            # Here, `cls` will just be Foo, buf if you called this method on a
            # subclass of Foo, `cls` would be that subclass instead.
            return cls.SOME_CLASS_CONSTANT + value

    Foo.add_to_my_constant(10)  # => 52

    # unlike in Java, you can also call a classmethod on an instance
    f = Foo()
    f.add_to_my_constant(10)    # => 52

旁注：文档字符串
=================

Python函数可以包含更多的信息，而不仅仅是代码：它们也包含有用的帮助信息，比如函数名称，文档字符串:

::

    >>> def fib(n):
    ...     "Recursively (i.e., dreadfully) calculate the nth Fibonacci number."
    ...     return n if n in [0, 1] else fib(n - 2) + fib(n - 1)
    ...
    >>> fib.__name__
    'fib'
    >>> fib.__doc__
    'Recursively (i.e., dreadfully) calculate the nth Fibonacci number.'

Python内置函数 `help <http://docs.python.org/library/functions.html#help>`_ 输出的就是这些信息。但是，当函数被包装之后，我们看到就是包装器函数的名称和文档字符串了:

::

    >>> fib = memoized(fib)
    >>> fib.__name__
    'memoized'
    >>> fib.__doc__

那样的信息并没有什么用处。幸运的是，Python包含一个名为 ``functools.wraps`` 的助手函数，能够把函数的帮助信息拷贝到其包装器函数:

::

    import functools
    def memoize(fn):
        stored_results = {}
        
        @functools.wraps(fn)
        def memoized(*args):
            # (as before)

        return memoized

使用装饰器帮助你编写装饰器会使很多事情令人非常满意。现在，如果使用更新过的memoize函数重试前面的代码，我们将会看到得到保留的文档:

::

    >>> fib = memoized(fib)
    >>> fib.__name__
    'fib'
    >>> fib.__doc__
    'Recursively (i.e., dreadfully) calculate the nth Fibonacci number.'

