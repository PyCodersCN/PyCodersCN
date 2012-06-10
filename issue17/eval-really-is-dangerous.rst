Eval是邪恶的
==============
原文: `<http://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html>`_

Python中有一个eval()内置函数，用来对一个python代码的字符串进行求值。


::

    assert eval("2 + 3 * len('hello')") == 17

这种特性非常强大，但是如果在参数中接收了不被信任的输入的话也会非常危险。假设被求值的字符串是 ``"os.system('rm -rf /')"`` 会有怎样的情况发生？这条命令真的会开始删除你计算机中的所有文件。(在接下来的例子中，我会用'clear'来代替'rm -rf /' 来防止意外的自残 )

有些人会说，你可以通过给它的globals参数传一个空来进行安全的计算操作。eval()会接收一个全局参数用来进行计算。如果你不提供这个global的字典的话，eval会使用目前的全局函数，所以这个时候"os"就可能会可用。如果你提供一个空字典进去的话，那么就不会有全局参数了。执行上面的操作，会抛出一个NameError的异常，"name 'os' is not defined":

::

  eval("os.system('clear')", {})
   
但是我们仍然可以通过内建函数__import__来导入模块并且使用他们。这样会成功执行

::

  eval("__import__('os').system('clear')", {})

接下来让我们的操作安全的尝试是，禁止访问内建的函数。原因是像 __import__ 和open这样的名字在Python 2中是可用的，因为他们是 __builtins__ 的全局函数。我们可以通过将__builtins__置为一个空的字典来明确的指名没有内置的函数。现在这样会抛出一个异常:

::

  eval("__import__('os').system('clear')", {'__builtins))':{}})

现在安全了吗？有些人说， `是的 <http://lybniz2.sourceforge.net/safeeval.html>`_ ，但是有些事情是不正确的。为了证明这件事情，我们在CPython下来跑下面的代码将会引发一个编译器的段错误。

::

    s = """
    (lambda fc=(
        lambda n: [
            c for c in 
                ().__class__.__bases__[0].__subclasses__() 
                if c.__name__ == n
            ][0]
        ):
        fc("function")(
            fc("code")(
                0,0,0,0,"KABOOM",(),(),(),"","",0,""
            ),{}
        )()
    )()
    """
    eval(s, {'__builtins__':{}})


我们现在来剖析一下，看看到底发生了什么。在中间我们发现了这个:

::

    ().__class__.__bases__[0]

这是一个解释"object"的很好的例子。元组的基类是一个"object"。记住，我们不能简单的说"object"，因为我们没有内置方法。我们可以用字面的语法来创建对象，然后使用这些属性。一下是用来显示一个object的所有子类。

::

    ().__class__.__bases__[0].__subclasses__()

换句话说，在程序中已经被实例化的类。我们最终会返回到这个点上。我把这些类简化做 ALL_CLASSED。然后这是一个易理解的列表来检查所有的类找到一个命名的n:

::

  [c for c in ALL_CLASSES if c.__name__ == n][0]

我们将这样通过名字找到类，因为我们需要用它两次。我们给它创建一个函数Z:

::

    lambda n: [c for c in ALL_CLASSES if c.__name__ == n][0]

但是我们这是一个eval里面，所以我们不能用def语句,或者赋值语句来给这个方法一个名字。但是对函数的默认参数也是赋值的一种形式，lambda可以有默认的参数。所以我们将剩下的代码放在一个lambda函数来进行一个默认赋值。

::

    (lambda fc=(
        lambda n: [
            c for c in ALL_CLASSES if c.__name__ == n
            ][0]
        ):
        # code goes here...
    )()

现在我们有了我们的"找类"的方法fc，我们用它来做什么？我们可以做一个对象！很简单，需要对构造器提供12个参数，但是大部分的是很简单的默认值。

::

    fc("code")(0,0,0,0,"KABOOM",(),(),(),"","",0,"")

这个"KABOOM"字符串是真实的字节码，你大概可以猜到，"KABOOM"不是一个有效的字节码串。是加上，这些串中的每个都已经足够，这些二进制操作符将会尝试去在一个空的操作栈上操作，将会使CPython产生段错误。"KABOOM"只是更有趣一些。感谢  `lvh <http://b.lvh.cc/>`_ 。

这将会给我们一个对象: fc("code")帮我们找到"code"类，然后我们通过十二个参数来调用它。你不能直接去调用这个code对象，但是你可以创建一个函数:

::

    fc("function")(CODE_OBJECT, {})

当然，一点你有了一个函数，你可以调用它。将会让这个代码运行在自己的对象中。在本例中，将会运行一个伪造的字节码，将Cpython的编译器引发段错误。以下是危险的字符串。

::
    (lambda fc=(lambda n: [c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == n][0]):
    fc("function")(fc("code")(0,0,0,0,"KABOOM",(),(),(),"","",0,""),{})()
    )()

所以eval并不安全，即使你已经将所有的globals和builtins移除掉。
我们用了一系列object的子类来创建一个code对象和一个函数。你当然可以找到其他的类并且使用他们。你能找到哪些类取决于eval()这个函数在哪里调用。在实际的程序中，在eval()函数调用时将会有很多类被创建，他们都将会在我们的ALL_CLASSES中，以下是例子:

::

    s = """
    [
        c for c in 
        ().__class__.__bases__[0].__subclasses__() 
        if c.__name__ == "Quitter"
        ][0](0)()
    """

标准的站点模块中定义了一个类叫做Quitter，与"quit"这个功能绑定，所以你可以在解释器中输入输入quit()来离开解释器。所以在eval中我们找到Quitter，来举例，调用它。这个字符串实际上退出了Python的解释器。

当然，在一个实际的系统中，将会有一系列强大的类，这些类可能被eval来解释和调用。eval带来的破坏性无可止境。

所有这些带来的问题试图调用eval()。明确的删除某个东西将会是很危险的。如果其中有一只漏网之鱼的话，就可以攻击整个系统。

当我在这个话题中尝试的时候，我在Python的受限制的执行模式中踌躇，我们尝试去通过lambda访问代码object,发现我们是不被允许的。

::

    >>> eval("(lambda:0).func_code", {'__builtins__':{}})
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<string>", line 1, in <module>
    RuntimeError: function attributes not accessible in restricted mode

受限模式是一个明确的访问黑名单指名是"危险"的属性访问。如果你的内建函数不是官方的内建函数的话，代码执行时候他会明确的被激活。在 `Tav's blog <http://tav.espians.com/paving-the-way-to-securing-the-python-interpreter.html>`_ 中有更多的细节对此话题的劳伦。如我们所见，这种首先模式对于防止恶作剧也是不够的。

所以，eval可以安全调用吗？很难说，此刻我最好的猜测就是如果你不使用任何重复强调的话将不会有任何上海，所以也许排除这些你会是安全的。也许。。

