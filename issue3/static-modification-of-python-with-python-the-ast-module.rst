AST 模块：用 Python 修改 Python 代码
====================================

原文地址： `<http://blueprintforge.com/blog/2012/02/27/static-modification-of-python-with-python-the-ast-module/>`_

翻译： `Upsuper <http://upsuper.org/>`_

修改代码在有时会变的十分有用，比如在进行测试和分析的时候。在这篇文章中，我们将看到如何使用 ``ast`` 模块对 Python 代码进行修改，同时还将看到一些使用了这个技术的工具。

CPython 的编译过程
------------------

.. image:: http://pyimg.fanhe.org/pep339.png
    :align: left

在开始之前，我们应该先看看 CPython 的编译过程，这个过程在 `PEP 339 <http://www.python.org/dev/peps/pep-0339/>`_ 中有详细的描述。

当然，在读这篇文章的时候，你并不需要对这个步骤有很深入的理解，不过这可以帮助你对整个过程有一个大体的了解。

首先，编译器会根据源代码生成一棵语法分析树 (Parse Tree)，随后，再根据语法分析树建立抽象语法树 (AST, Abstract Syntax Tree)。从 AST 中可以生成出控制流图 (CFG, Control Flow Graph)，最后再将控制流图编译为代码对象 (Code Object)。

图中标蓝的部分就是 AST 这一步，也就是我们今天所关注的部分。Python 从 2.6 开始就提供了现在这样的 ``ast`` 模块，它提供了一种访问和修改 AST 的简单方式。

通过这个，我们可以从 AST 中生成代码对象，也可以出于某些原因，根据修改过的 AST 重新生成源代码。

创建 AST
--------

先来写一点简单的代码，我们写一个叫做 ``add`` 的函数，然后观察它所生成的 AST。

::
    
    >>> import ast
    >>> expr = """
    ... def add(arg1, arg2):
    ...     return arg1 + arg2
    ... """
    >>> expr_ast = ast.parse(expr)
    >>> expr_ast
    <_ast.Module object at 0x10a7a09d0>

现在我们已经生成了一个 ``ast.Module`` 对象，我们来看看它的内容：

::
    
    >>> ast.dump(expr_ast)
    "Module(
        body=[
            FunctionDef(
                name='add', args=arguments(
                    args=[
                        Name(id='arg1', ctx=Param()),
                        Name(id='arg2', ctx=Param())
                        ],
                        vararg=None,
                        kwarg=None,
                        defaults=[]),
                body=[
                    Return(
                        value=BinOp(
                            left=Name(id='arg1', ctx=Load()),
                            op=Add(),
                            right=Name(id='arg2', ctx=Load())))
                    ],
                decorator_list=[])
        ])"

正如我们所见， ``Module`` 是父节点，它的 ``body`` 中包含了一个函数定义的元素，这个函数定义包含了函数名、参数列表和函数体。函数体又包含了一个单独的 ``Return`` 节点，节点中含有一个 ``Add`` 运算。

修改 AST
--------

我们如何修改这棵树以改变代码的作用呢？为了说明这个问题，我们来做点也许你永远也不会在你自己代码中做的疯狂的事情吧。我们将遍历这棵树，并且将 ``Add`` 运算修改为 ``Mult`` 运算。看，我说过这很疯狂吧！

我们要先建立一个 ``NodeTransformer`` 变换器的子类，并且定义 ``visit_BinOp`` 方法。每当这个变换器访问到一个二元运算符节点时，就会调用这个方法。

::
    
    class CrazyTransformer(ast.NodeTransformer):

        def visit_BinOp(self, node):
            print node.__dict__
            node.op = ast.Mult()
            print node.__dict__
            return node

现在我们已经定义好了我们这个奇怪的变换器，让我们看看将它应用于我们开始时写的那些代码会怎么样：

::
    
    >>> transformer = CrazyTransformer()
    >>> transformer.visit(expr_ast)
    {
        'op': <_ast.Add object at 0x10a8321d0>,
        'right': <_ast.Name object at 0x10a839390>,
        'lineno': 3, 'col_offset': 8,
        'left': <_ast.Name object at 0x10a839350>}
    {
        'op': <_ast.Mult object at 0x10a839510>,
        'right': <_ast.Name object at 0x10a839390>,
        'lineno': 3, 'col_offset': 8,
        'left': <_ast.Name object at 0x10a839350>}

你可以从输出的结果对比发现， ``Add`` 节点已经被替换成了一个 ``Mult`` 。我们有许多方法没有提到，比如访问子节点，不过这个例子已经足以刻画出它的基本原理。

编译和执行修改后的 AST
----------------------

我们在最初的代码后面加上一个调用，比如：

::
    
    print add(4, 5)

让我们看看这些代码是如何运行的：

::
    
    >>> unmodified = ast.parse(expr)
    >>> exec compile(unmodified, '<string>', 'exec')
    9
    >>> transformer = CrazyTransformer()
    >>> modified = transformer.visit(unmodified)
    >>> exec compile(modified, '<string>', 'exec')
    20

我们可以看到，未修改的和修改后的 AST 所编译出的代码，一个输出了9，一个输出了20。

重新翻译回源代码
----------------

最后，我们可以用 ``unparse`` 模块将修改后的代码转换回对应的源代码， ``unparse`` 模块可以在 `这里 <http://svn.python.org/projects/python/trunk/Demo/parser/unparse.py>`_ 找到。

::
    
    >>> unparse.Unparser(modified, sys.stdout)

    def add(arg1, arg2):
        return (arg1 * arg2)
    print add(4, 5)

正如我们所看到的， ``*`` 运算符取代了 ``+`` 。在这个反解析工具对于理解你的 AST 变换器如何修改代码很有帮助。

实践应用
--------

显然，我们上面的例子在实际应用中几乎没有意义。然而静态分析和修改代码却是十分有用的。

比如你可以为测试程序而注入一些代码。你可以看看 `这篇 PyCon 演讲 <http://www.tudou.com/programs/view/5IHp-wxyt3c/>`_ ( `origin <http://blip.tv/pycon-us-videos-2009-2010-2011/pycon-2011-what-would-you-do-with-an-ast-4898264>`_ ) 以理解如何使用一个节点转换器注入指令代码来测试程序。

除此之外， `Pythonscope 项目 <http://pythoscope.org/>`_ 也使用了 AST 访问器 (visitor) 来处理源代码并根据函数签名生成测试。

还有像 pylint 这样的项目使用 AST 步移法 (walking method) 来分析源代码。在 pylint 中，Logilab 还建立了一个模块专门用于：

    "提供一个通用的 Python 源代码基本表示方式以为如 pychecker、pyreverse 或 pylint 等项目的开发提供方便。"

你可以在 `这里 <http://www.logilab.org/project/logilab-astng>`_ 看到更多关于这个项目的信息。

引用
----

Matthew J Desmarais 的 `这篇 PyCon 演讲 <http://www.tudou.com/programs/view/5IHp-wxyt3c/>`_ ( `origin <http://blip.tv/pycon-us-videos-2009-2010-2011/pycon-2011-what-would-you-do-with-an-ast-4898264>`_ ) 以及 Eli Bendersky 的 `这篇博客 <http://eli.thegreenplace.net/2009/11/28/python-internals-working-with-python-asts/>`_ 对于本文的帮助是无可估量的。
