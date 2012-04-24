Python FAQ：相等
===============================

原文： `<http://me.veekun.com/blog/2012/03/24/python-faq-equality/>`_ 

译者： `TheLover_Z <http://zhuang13.de>`_ 

这是 `Python 常见问题 <http://me.veekun.com/blog/2012/03/24/python-faq-equality/>`_ 的一部分。

**What does** ``is`` **do? Sould I use** ``is`` **or** ``==`` **?**

这些操作符让很多 Python 新手都很疑惑，也许是因为 ``is`` 在其它的语言中并不常用。一些 Python 标准实现的奇怪用法也让这个东西看起来很难理解。

简单来说就是：

- ``==`` 检查两个对象是不是有同样的值。

- ``is`` 检查两个对象是不是 *相同的* 对象。

“相同的值”是什么意思？它取决于对象的类型。通常情况下它意味着对同样的操作返回同样的值，但是准确来说是由它的类来决定的。

另一方面，不管两个对象多么多么的像， ``is`` 总是可以把它们区分开。你调用了两次 ``SomeClass()`` ？然后你就有两个对象，并且 ``a is b`` 这个的结果也会是 ``False`` 。

重载
----------

这是另一个很精细的差别： ``==`` **可以被重载，但是** ``is`` **不能** 。不管是 ``__eq__`` 还是 ``__cmp__`` 都允许一个类来判断对于它们自身来说相等意味着什么。

因为那些方法是很常见的，所以它们可以做许多事情。一个对象可能并不和自身相等。它也可能和什么都相等。它可能随机的相等或者不相等。它可能对于 ``==`` 和 ``!=`` 都返回 ``True`` 。（这段好文艺…）

这样很混乱，我们都不希望这样的情况发生，但是这样的情况就是 *有可能* 发生。记住， ``==`` 并不一定可信，但是 ``is`` 就不一样了。

当 Python 看到 ``a == b`` 的时候，它会做出下面的判断：

- 如果 ``type(b)`` 是一个 new-style class (没有翻到中文，求指点)， ``type(b)`` 是 ``type(a)`` 的一个子类，并且 ``type(b)`` 重写了 ``__eq__`` ，那么结果就是 ``b.__eq__(a)`` 。

- 如果 ``type(a)`` 重写了 ``__eq__`` (也就是说， ``type(a).__eq__`` 并不是 ``object.__eq__`` )，那么结果就是 ``a.__eq__(b)`` 。

- 如果 ``type(b)`` 重写了 ``__eq__`` ，那么结果就是 ``b.__eq__(a)`` 。

- 如果以上任何一种情况都不是，Python 就会查找 ``__cmp__`` ，如果存在的话当且仅当返回值是 0 的时候是相等的。

- 最后，Python 调用 ``object.__eq__(a, b)`` ，当且仅当 ``a`` 和 ``b`` 是同一个对象的时候返回 ``True`` 。 

注意：如果 ``a`` 或者 ``b`` 都没有重载 ``==`` ，那么 ``a == b`` 就和 ``a is b`` 一样。

如何在正确的时间使用正确的操作符
-------------------------------------------

确实，使用 ``is`` 的机会非常少。最常用的例子是这样的：

::

    def foo(arg=None):
        if arg is None:
            arg = []

        # ...

为什么在这里要用 ``is`` ？这样读起来挺像是英语，而且这样的话关于 ``None`` 就不会有歧义。一个更好的解释有点儿不太容易想到：操作符重载。如果 ``arg`` 恰好重载了，那么它就有可能和 ``None`` 相等。

有时候 ``None`` 可能会有特殊的含义，也许意味着 JSON 或者 SQL 里面的 ``null`` 。如果你写了像我刚才写的那个函数，没有人可以传递给它 ``None`` ，它会被默认替换掉。在 ``None`` 是个真实的值的时候怎么传递参数呢？这个时候 ``is`` 也可以起作用。

::

    unspecified = object()
    def foo2(arg=unspecified):
        if arg is unspecified:
            arg = make_default_object()

        # ...

在这里 ``unspecified`` 只是一个不包含数据也没有任何行为的对象。唯一有用的是 ``if arg is unspecified`` 这句，然后你就明白了 ``arg`` *必须是* 那个对象。它没有歧义，所以很安全。

当然， ``==`` 也一样，但它也有关于 ``arg == None`` 的一点儿小提示：不好的重载。使用 ``is`` 可以更好地表达你的 *意愿* ，可能你想要做一些特别的测试。

通常来说，你需要 ``==`` 。 ``is`` 只是在你想要完全确定两个对象一致的时候才有用。

is 和 内置函数
-------------------------

一个常见的陷阱就像这样：

::

    >>> 2 == 2
    True
    >>> 2 is 2
    True
    >>> "x" == "x"
    True
    >>> "x" is "x"
    True
    >>> int("133") is int("133")
    True

神马？！这里怎么回事？！这些单独的数字和单独的字符串，甚至单独调用了 ``int()`` ，为什么还是相同的对象？

在随便一个 Python 程序里面都有许多字符串，比如说 ``__init__`` 。也有 *许多* 小数字，比如说 ``0`` 和 ``-1`` 。严格意义上来说，这些字符串和数字每次出现的时候，Python 都需要创造一个新的对象，然后消耗掉一部分内存。在一个类中找到一个方法需要一个字节一个字节的比较，这需要消耗时间。

所以 CPython 对此做了优化，叫做驻留。小整数和一些字符串进行缓存：整数 ``2`` 永远都指向同一个对象，不管它是哪儿来的。

驻留并 *不* 是严格意义上语言的一部分，其它 Python 实现可能有也可能没有。只要是不变的对象都可以驻留，否则不可以。所以说， **对于内置的不可变类型** 绝对 **不要使用** ``is`` 。

当 CPython 编译代码的时候，它会先创建对象来替代字面量。字面量就是有本地 Python 标示符 (比如数字，字符串，使用 ``[]`` 的列表) 的对象。至于数字和字符串，相同值的字面量就变成了 *相同的对象* ，不管是否驻留。

然后我们来看看下面这段代码，应该就清楚许多了。

::

    # Interned ints
    # 整数的驻留
    >>> 100 is 100
    True
    # Non-interned ints, but compiled together, so still the same object
    # 非驻留的整数，但是一起编译了，所以仍然是相同的对象
    >>> 99999 is 99999
    True
    # Non-interned ints, compiled /separately/, so different objects
    # 非主流的整数，独立的编译，所以不同
    >>> a = 99999
    >>> b = 99999
    >>> a is b
    False
    # Interned ints are the same object no matter where they appear
    # 驻留的整数不管在哪儿都是相同的
    >>> a = 3
    >>> b = 3
    >>> c = 6 / 2
    >>> a is b
    True
    >>> a is c
    True
    # Floats are never interned, but these are compiled together, so are still the same object
    # 浮点数永远不驻留，但是对于一起编译的浮点数，它们是相同的对象
    >>> 1.5 is 1.5
    True
    # Strings are similar to ints
    # 字符串的结果和整数的差不多
    >>> "foo" is "foo"
    True
    >>> a = "foo"
    >>> b = "foo"
    >>> a is b
    True
    >>> "the rain in spain falls mainly on the plain" is "the rain in spain falls mainly on the plain"
    True
    >>> a = "the rain in spain falls mainly on the plain"
    >>> b = "the rain in spain falls mainly on the plain"
    >>> a is b
    False
    # Two different lists; they're mutable so they can't be the same object
    # 两个不同的列表，他们都是可变的，所以不可能是同一个对象。
    >>> [] is []
    False
    # Two different dicts; same story
    # 两个不同的词典。和上面一样。
    >>> {} is {}
    False
    # Tuples are immutable, but their contents can be mutable, so they don't get same object
    # 元组不可变，但他们的内容是可变的，所以它们也不是同样的对象。
    >>> (1, 2, 3) is (1, 2, 3)
    False

(顺便说一下，如果你真的特别特别好奇的话。CPython 对 ``-5`` 到 ``256`` 之间的 ``int`` 全部做驻留处理，闭区间。)

结论
-----------

- 大多数时间你需要 ``==`` 。

- 当你有一个带有默认参数是 ``None`` 的函数的时候使用 ``arg is None`` 。这样就行了，因为只有唯一的一个 ``None`` 。

- 测试两个类，函数，或者模块是不是同一个对象的时候， ``is`` 不错。

- 处理 ``str`` ， ``int`` ， ``float`` ， ``complex`` 或者其它内置不可变类型的时候 **永远不要** 使用 ``is`` 。正因为驻留的存在，这样的比较毫无意义。

- 其它的关于 ``is`` 的使用比较稀少而且晦涩，比如说：

    - 如果我有一个大型的树结构，想要找到子树的位置， ``==`` 递归的检查值 (意思就是非常慢)，但是 ``is`` 在找到同样的结点的时候会告诉你。

    - 缓存设备可能想要把所有的对象都加以区别，而不需要考虑或者依赖 ``==`` 。 ``is`` 在这里可以很精确的实现。

    - 给初学者演示一下驻留只在用 ``is`` 的时候存在:)

再总结一下：不要使用 ``is`` ，除非你正在和 ``None`` 比较或者你真的真的明白你在做什么。

更深入的阅读
-----------------

- Python 语言参考的 `数据模型部分 <http://docs.python.org/reference/datamodel.html>`_ 说明了缓存不可变数值的可能性。 `__eq__ 怎么工作 <http://docs.python.org/reference/datamodel.html#object.__eq__>`_ ， 还有 `操作符重载怎么工作 <http://docs.python.org/reference/datamodel.html#coercion-rules>`_ 。

- `The Python C API <http://docs.python.org/c-api/int.html#PyInt_FromLong>`_ 是唯一一份关于什么整数被驻留的文档。

中英文对照：
------------------

- 驻留 - interning

- 字面量 - literals 