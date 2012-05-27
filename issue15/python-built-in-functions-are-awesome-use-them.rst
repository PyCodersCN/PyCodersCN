Python 内置的那些牛逼闪亮的函数
=================================

原文： `<http://isbullsh.it/2012/05/05-Python-built-in-functions/>`_

译者： `TheLover_Z <http://zhuang13.de>`_ 

引言
--------

这篇文章我们来看几个很有用的 `Python 内置函数 <http://docs.python.org/release/2.7.2/library/functions.html>`_ 。这些函数简直是屌爆了，我认为每个 Pythoner 都应该知道这些函数。


对于每个函数，我会使用一个普通的实现来和内置函数做对比。

如果我直接引用了内置函数的文档，请理解，因为这些函数文档写的非常棒！

``all(iterable)``
-----------------------

如果可遍历的对象(数组，字符串，列表等，下同)中的元素都是 true (或者为空)的话返回 ``True`` 。

::

    _all = True
    for item in iterable:
        if not item:
            _all = False
            break
    if _all:
        # do stuff

更简便的写法是：

::

    if all(iterable):
        # do stuff

``any(iterable)``
---------------------

如果可遍历的对象中任何一个元素为 true 的话返回 ``True`` 。如果可遍历的对象为空则返回 ``False`` 。

::

    _any = False
    for item in iterable:
        if item:
            _any = True
            break
    if _any:
        # do stuff

更简便的写法是：

::

    if any(iterable):
        # do stuff

``cmp(x, y)``
------------------

比较两个对象 ``x`` 和 ``y`` 。 ``x < y`` 的时候返回负数， ``x ==y`` 的时候返回 0， ``x > y`` 的时候返回正数。

::

    def compare(x,y):
        if x < y:
            return -1
        elif x == y:
            return 0
        else:
            return 1

你完全可以使用一句 ``cmp(x, y)`` 来替代。

``dict([arg])``
-----------------

使用 ``arg`` 提供的条目生成一个新的字典。

``arg`` 通常是未知的，但是它很方便！比如说，如果我们想把一个含两个元组的列表转换成一个字典，我们可以这么做。

::

    l = [('Knights', 'Ni'), ('Monty', 'Python'), ('SPAM', 'SPAAAM')]
    d = dict()
    for tuple in l:
        d[tuple[0]] = tuple[1]
    # {'Knights': 'Ni', 'Monty': 'Python', 'SPAM': 'SPAAAM'}

或者这样：

::

    l = [('Knights', 'Ni'), ('Monty', 'Python'), ('SPAM', 'SPAAAM')]
    d = dict(l) # {'Knights': 'Ni', 'Monty': 'Python', 'SPAM': 'SPAAAM'}

``enumerate(iterable [,start=0])``
---------------------------------------

我真的是超级喜欢这个！如果你以前写过 C 语言，那么你可能会这么写：

::

    for i in range(len(list)):
        # do stuff with list[i], for example, print it
        print i, list[i]

噢，不用那么麻烦！你可以使用 ``enumerate()`` 来提高可读性。

::

    for i, item in enumerate(list):
        # so stuff with item, for example print it
        print i, item

``isinstance(object, classinfo)``
------------------------------------

如果 ``object`` 参数是 ``classinfo`` 参数的一个实例或者子类(直接或者间接)的话返回 ``True`` 。

当你想检验一个对象的类型的时候，第一个想到的应该是使用 ``type()`` 函数。

::

    if type(obj) == type(dict):
        # do stuff
    elif type(obj) == type(list):
        # do other stuff
    ...

或者你可以这么写：

::

    if isinstance(obj, dict):
        # do stuff
    elif isinstance(obj, list):
        # do other stuff
    ...

``pow(x, y [,z])``
---------------------

返回 ``x`` 的 ``y`` 次幂(如果 ``z`` 存在的话则以 ``z`` 为模)。

如果你想计算 x 的 y 次方，以 z 为模，那么你可以这么写：

::

    mod = (x ** y) % z

但是当 x=1234567， y=4567676， z=56 的时候我的电脑足足跑了 64 秒！

不要用 ``**`` 和 ``%`` 了，使用 ``pow(x, y, z)`` 吧！这个例子可以写成 ``pow(1234567, 4567676, 56)`` ，只用了 0.034 秒就出了结果！

``zip([iterable, ])``
-------------------------

这个函数返回一个含元组的列表，具体请看例子。

::

    l1 = ('You gotta', 'the')
    l2 = ('love', 'built-in')
    out = []
    if len(l1) == len(l2):
        for i in range(len(l1)):
            out.append((l1[i], l2[i]))
    # out = [('You gotta', 'love'), ('the', 'built-in)]

或者这么写：

::

    l1 = ['You gotta', 'the']
    l2 = ['love', 'built-in']
    out = zip(l1, l2) # [('You gotta', 'love'), ('the', 'built-in)]

如果你想得到倒序的话加上 ``*`` 操作符就可以了。

::

    print zip(*out)
    # [('You gotta', 'the'), ('love', 'built-in')]

结论
-------

Python 内置函数很方便，它们很快并且经过了优化，所以它们可能效率更高。

我真心认为每个 Python 开发者都应该好好看看内置函数的文档(引言部分)。

忘了说了，在 ``itertools`` 模块中有很多很不错的函数。再说一次，它们确实屌爆了。

中英文对照
------------

可遍历的对象 － iterable
