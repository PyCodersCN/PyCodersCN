探索 Python 代码对象
====================

由于受到 `David Beazley <http://www.dabeaz.com/>`_ 在 `PyCon <https://us.pycon.org/2012/>`_ 上的 `Keynote <http://pyvideo.org/video/659/keynote-david-beazley>`_ 的启发，近来我四处学习与 Python 代码对象 (code object) 相关的内容。我并没有什么特别的利器，也没有专门的任务去解决 (至今为止？)，所以请将这篇文章看做一些也许有趣的记录和随笔 (如果没意思的话，抱歉)。

*免责声明：* 这篇文章是关于 CPython 2.7 的，虽然其中的大部分对于其他的 CPython 版本应该也是正确的 (包括 3.x)。但我不保证它在 PyPy、Jython、IronPython 等实现上是正确和适用的。

第0步：是什么？
---------------

所以首先，代码对象是什么呢？许多人 (特别是仇视 Python 的人) 声称 Python 是一个解释型语言，但是事实上你所有的 Python 代码在执行之前都被编译了，甚至于你在 Python 命令行程序中交互式输入的代码也是如此。CPython 实现了一个执行基于栈的字节码 (stack-based bytecode) 的虚拟机。在运行时，任何可执行的东西 (函数、方法、模块、类主体 (class body)、Lambda 式、语句、表达式等等) 都以字节码的形式由 Python 虚拟机执行。

然后，代码对象是用于表示字节码片段的 Python 对象，同时还附带了所有执行需要的东西：预期的参数名称和数量的声明、一个本地变量的列表 (不是字典！稍后会有更多说明)、字节码生成时与源代码相关的信息 (用于调试和输出栈跟踪) 等——哦，还有 (也许很显然地) 字节码本身，以 ``str`` 来保存 (在 Python 3 中为 ``bytes`` )。

虽然代码对象表示了一些可执行的代码，但它们本身并不能被直接调用。要运行一个代码对象，你必须使用 ``exec`` 关键字或者 ``eval()`` 函数。

第1步：制造一些代码
-------------------

在大部分时候，你在平常的 Python 编程中并不需要面对代码对象。这些时候，你不需要特别地注意代码对象，Python 会为你创建和管理它们。但在某些情况下，你可能想要自己创建代码对象，比如在这篇文章中我们就要用他们来进行实验：

::
    
    >>> code_str = """
    ... print "Hello, world"
    ... """
    >>> code_obj = compile(code_str, '<string>', 'exec')
    >>> code_obj
    <code object <module> at 0x1054c74b0, file "<string>", line 2>

    Woohoo，你的第一个代码对象！

传递给 `compile() <http://docs.python.org/library/functions.html#compile>`_ 的第一个参数，很明显是需要编译的 Python 代码字符串。第二个定义了代码的“文件名” (在这里，方便起见我们用了 ``<string>`` 来表示代码从交互命令行中获得)。第三个参数是编译的类型，这个参数在大多数情况下都是 ``exec`` 正如你在这里看到的。其他可能的模式还有适用于仅包含一条表达式的字符串的 ``eval`` ，和生成仅含有一条语句的代码对象的 ``singnle`` 。后者的返回值如果不是 ``None`` 的话将会被打印出来 (正如交互式命令行一样)。

当使用 ``eval`` 模式时，如果代码中含有语句 (如我们上面包含 ``print`` 语句的例子那样) 编译将会因为一个符号错误失败：

::
    
    >>> code_str = """
    ... print "Hello, world"
    ... """
    >>> code_obj = compile(code_str, '<string>', 'eval')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<string>", line 2
        print "Hello, world"
            ^
    SyntaxError: invalid syntax

当使用 ``single`` 的时候，只有一个语句将会被处理，更多的语句 (或者其他什么) 将会被忽略：

::
    
    >>> code_str = """
    ... print "Hello, world"
    ... print "Goodbye, world"
    ... """
    >>> code_obj = compile(code_str, '<string>', 'single')
    >>> exec code_obj
    Hello, world

    我的 “goodbye” 发生了什么？

在这篇文章的余下部分，我们将继续分析 ``exec`` ，它也是在你导入模块时 Python 为你做的编译类型。

第2步：打开它
-------------

让我们回到我们的第一个例子然后进入代码对象内部看看我们有些什么：

::
    
    >>> code_str = """
    ... print "Hello, world"
    ... """
    >>> code_obj = compile(code_str, '<string>', 'exec')
    >>> dir(code_obj)
    # 处于可读性考虑，这里排除了无关的属性
    ['co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_lnotab', 'co_name','co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']

这些属性都在 `inspect <http://docs.python.org/library/inspect.html>`_ 模块的文档中都有写明，不过这里我将特别说一说其中几个很酷的属性：

首先，我们可以看看我们传递给 ``compile()`` 的第二个参数最后到了哪里：

::
    
    >>> code_obj.co_filename
    '<string>'

还有也许让人吃惊的，我们的代码表示了一个匿名模块 (以 ``exec`` 模式编译的代码总是被当做模块级的代码，所以理所当然地它可以包含函数或类的定义，或者其他任何有效的 Python 代码)：

::
    
    >>> code_obj.co_name
    '<module>'

并且正如我们所期望的，一个表示 Python 模块的代码对象是没有参数的 (事实上这是因为我们的代码串仅仅是由一系列在没有任何缩进，在最顶层的语句组成)：

::
    
    >>> code_obj.co_argcount
    0
    >>> code_obj.co_varnames
    ()

如果我们从函数中取出一段含有参数的代码，我们将看到它们：

::
    
    >>> def foo(x, y):
    ...     print x, y
    ... 
    >>> foo.func_code
    <code object foo at 0x1054b9830, file "<stdin>", line 1>
    >>> foo.func_code.co_varnames
    ('x', 'y')
    >>> foo.func_code.co_argcount
    2

如果你很好奇，你也可以看看 Python 虚拟机将会处理的原始字节码：

::
    
    >>> code_obj.co_code
    'd\x00\x00GHd\x01\x00S'

我不推荐尝试直接去阅读这个代码，我们有一些更简单的方式 (注释：看看下一小节)。

最后，在代码中我们有一个常量对象，即我们的代码输出的字符串 “Hello, world”：

::
    
    >>> code_obj.co_consts
    ('Hello, world', None)

等等，那个 ``None`` 是哪儿来的？

稍稍绕道到代码反编译
--------------------

`dis <http://docs.python.org/library/dis.html>`_ 模块可以将代码对象反汇编为人类可读的一系列字节码指令，我们可以用它来精确的观察我们的代码对象中到底有些什么：

::
    
    >>> dis.dis(code_obj)
      2           0 LOAD_CONST               0 ('hello, world')
                  3 PRINT_ITEM          
                  4 PRINT_NEWLINE       
                  5 LOAD_CONST               1 (None)
                  8 RETURN_VALUE        

阅读反汇编的 Python 代码需要一点经验，所以让我来带你探索它。 ``LOAD_CONST`` 指令从 ``co_consts`` 元组中读取一个值，并将其压入栈顶。 ``PRINT_ITEM`` 指令则弹出栈顶元素并输出其字符串表示。 ``PRINT_NEWLINE`` 指令的名字可以很好地说明自己的用途。

下面我们来看看神秘的 ``None`` 。这被证明是 CPython 虚拟机实现细节中的一个怪异之处。因为任何 Python 中的函数调用 (包括隐含的函数调用如 ``import`` 语句) 在 Python 虚拟机中被实现为 C 函数调用。模块事实上有返回值，这提醒 Python 虚拟机模块的运行已经结束，并且将控制权返回给调用者 (即 ``import`` 语句所在的模块)。我不想尝试在这个问题上做更深入的解释，如果你有兴趣的话，可以看看 `Larry Hastings <http://www.larryhastings.com/>`_ 在 PyCon 的演讲 `Steping through CPython <http://pyvideo.org/video/635/stepping-through-cpython>`_ 大约 44:22 的位置。这个视频说的是 Python 3.x，不过 Python 2.7 其实是一样的。如果你对整个实现细节都感兴趣，你肯定应该完整地看完这个视频以及 David Beazley 的 Keynote。

第3步：有趣的内部实现
---------------------

我们已经看过的那些特性显然对于 Python 虚拟机的执行是非常有用的，但是关于人的部分呢？如果我们想要交互地调试代码 (用 ``pdb`` 或者类似的工具)，获得有用的、可读的异常回溯信息，该怎么办？

可以证明，代码对象也同样支持这些。我们之前已经看到，代码对象可以给出它是依据哪个文件生成出来的，这显然可以帮助我们查看源代码。它同样可以指出它自己是从源代码的哪一行开始的：

::
    
    >>> code_obj.co_firstlineno
    2

还有神秘的 ``co_lnotab`` 属性。为了描述它的用途，我们需要更长一点的代码段：

::
    
    >>> code_str = """
    ... x = 1
    ... y = 1
    ... print x + y
    ... """
    >>> code_obj = compile(code_str, '<string>', 'exec')
    >>> code_obj.co_lnotab
    '\x06\x01\x06\x01'

嗯，然后我们能从中知道些什么呢？也许 ``dis`` 模块在这儿能够帮点忙：

::
    
    >>> dis.dis(code_obj)
      2           0 LOAD_CONST           0 (1)
                  3 STORE_NAME           0 (x)
    
      3           6 LOAD_CONST           1 (2)
                  9 STORE_NAME           1 (y)
    
      4          12 LOAD_NAME            0 (x)
                 15 LOAD_NAME            1 (y)
                 18 BINARY_ADD
                 19 PRINT_ITEM
                 20 PRINT_NEWLINE
                 21 LOAD_CONST           2 (None)
                 24 RETURN_VALUE

在一些行的最左边是代码对象所对应的 Python 源代码的行号 (注意到这里的2正对应了 ``code_obj.co_firstlineno`` 的值)。后面一列是这个字节码指令在代码中的偏移位置，0字节处是第一条指令，3字节处是第二条这样。第三列是指令名称本身。而如果指令有参数的话，第四列就是参数，参数后面的括号内是参数的值。

现在我们可以将这些东西与 ``co_lnotab`` (其实就是“行号表 (line number table)”的意思) 放到一起来看看 Python 是如何将代码对象与原始的源代码联系起来的：

::
    
    >>> code_obj.co_lnotab
    '\x06\x01\x06\x01'

在经过一些尝试和错误之后，我明白了这是一串字节对：第一个字节是字节码的偏移量 (这里是6字节，加上这之后就到达了我们反汇编之后的第二个 ``LOAD_CONST`` )，后面是源代码中跳过的行。

我们可以稍稍修改我们的源代码来验证这个理论：

::
    
    >>> code_str2 = """
    ... x = 1
    ... 
    ... y = 2
    ... print x + y
    ... """
    >>> code_obj2 = compile(code_str2, '<string>', 'exec')
    >>> code_obj2.co_lnotab
    '\x06\x02\x06\x01'

我们将第二条赋值语句向下移了一行，于是我们就看到 ``co_lnotab`` 的第二个字节由1变成了2，表示从当前行数向下移2行。

我们同样可以验证这两个稍稍有些不同的源代码生成的字节码是相同的：

::
    
    >>> code_obj2.co_code == code_obj.co_code
    True

由于字节码偏移和行号偏移都是一个无符号字节，于是你或许会想，如果比如说在 Python 代码中的两个语句之间我有 257 (或更多) 的空行会怎么样？让我们来看看：

::
    
    >>> thousand_blanks = '\n' * 1000
    >>> code_str = """
    ... x = 1
    ... """ + thousand_blanks + """
    ... y = 2
    ... print x + y
    ... """
    >>> code_obj = compile(code_str, '<string>', 'exec')
    >>> code_obj.co_lnotab
    '\x06\xff\x00\xff\x00\xff\x00\xec\x06\x01'

由于字节码便宜和行数偏移终归都是偏移，大的空白仅仅意味着另一边是一个长度为0的便宜。于是我们在字节码上有6字节的偏移，随后跟着一个源代码上 255 行的偏移 (原文写 256 是错误的，译注)，然后在字节码上0字节的偏移，之后又是一个 255 行的源代码偏移，还有一个0字节的字节码偏移和一个 255 行的源码偏移，还有一个0字节的字节码偏移和最后一个 236 行的源代码偏移 (接下去就是对于最后一个 ``print`` 语句的正常的6个字节的字节码偏移和1行的源代码偏移)。如此优雅！

结语
----

我先对这篇文章的散乱表示抱歉，不过我希望它的内容能让你感到很有趣。敬请继续关注对 ``exec`` 特性的探索以及在不久的将来其在 `Keystone <http://late.am/post/2011/11/27/keystone-a-simple-python-web-framework`_ 中的应用 (这是作者开发的一个有趣的 Python 网站框架，值得关注一下，译注)。
