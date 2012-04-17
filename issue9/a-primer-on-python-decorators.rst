Python装饰器入门
==================

原文: `A primer on Python decorators <http://www.thumbtack.com/engineering/a-primer-on-python-decorators/>`_

翻译: `youngsterxyf <http://xiayf.blogspot.com/>`_

Python允许作为程序员的你使用函数完成一些很酷的事情。在Python中，函数是一等对象(first-class object)，这就意味着你可以像使用字符串，整数，或者任何其他对象一样使用函数。例如，你可以将函数赋值给变量:
::

    >>> def square(n):
    ...     return n * n;
    >>> square(4)
    16
    >>> alias = square
    >>> alias(4)
    16

然而，一阶函数的真正威力在于你可以把它传给其他函数，或者从其他函数中返回函数。Python的内置函数map利用了这种能力：给map传个函数以及一个列表，它会在列表中每个元素上调用你传给它的那个函数，从而生成一个新的列表。如下所示的例子中应用了上面的那个square函数:
::

    >>> number = [1, 2, 3, 4, 5]
    >>> map(square, numbers)
    [1, 4, 9, 16, 25]

如果一个函数接受其他函数作为参数，以及/或者返回一个函数，那么它就被称为高阶函数。虽然map函数只是简单地使用了我们传给它的函数，而没有改变这个函数，但我们也可以使用高阶函数去改变其他函数的行为。

例如，假设有这样一个函数，会被调用很多次，以致运行代价非常昂贵:
::

   >>> def fib(n):
   ...      "Recursively (i.e., dreadfully) calculate the nth Fibonacci number."
   ...      return n if n in [0, 1] else fib(n - 2) + fib(n - 1)

我们一般会保存这个计算过程中的中间结果，这样，对于函数调用树中经常出现某个n，当需要计算n对应的结果时，就不需要重复计算了。有多种方式可以做到这点。例如，我们可以将这些中间结果存在一个字典中，当以某个值为参数调用fib函数时，就先到这个字典去查一下其结果是否已经计算出来了。

但这样的话，每次我们想要调用fib函数，都需要重复那段相同的字典检查样板代码。相反，如果让fib函数自己在内部负责存储中间结果，那么在其他代码中调用fib，就非常方便，只要简单地调用它就行了。这样一种技术被称为memoization(注意没有字母r的哦)。

我们可以把这种memoization代码直接放入fib函数，但是Python为我们提供了另外一种更加优雅的选择。因为可以编写修改其他函数的函数，那么我们可以编写一个通用的memoization函数，以一个函数作为参数，并返回这个函数的memoization版本:
::

    def memoize(fn):
        stored_results = {}

        def memoized(\*args):
            try:
                # try to get the cached result
                return stored_results[args]
            except KeyError:
                # nothing was cached for those args. let's fix that.
                result = stored_results[args] = fn(*args)
                return result
        return memoized


