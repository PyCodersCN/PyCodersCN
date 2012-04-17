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


