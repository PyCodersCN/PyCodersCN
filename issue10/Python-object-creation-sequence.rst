Python对象创建过程
=====================

原文： `Python object creation sequence <http://eli.thegreenplace.net/2012/04/16/python-object-creation-sequence/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

[本文讨论的Python版本为3.x]

本文旨在探究Python中新对象的创建过程。正如我在 `前一篇文章 <http://eli.thegreenplace.net/2012/03/23/python-internals-how-callables-work/>`_ 中所解释的，对象的创建只是调用可调用对象的一种特例。考虑这样的一段Python代码：

::

    class Joe:
        pass

    j = Joe()

当j = Joe()被执行时发生了什么呢？Python把它看作对可调用的Joe的一次调用，并且将它路由到内部函数 ``PyObject_Call`` ，将Joe作为PyObject_Call的第一个参数。 ``PyObject_Call`` 根据其第一个参数的类型抽取这个参数类型的 ``tp_call`` 属性。

那么，Joe的类型是什么呢？无论何时我们定义一个新的Python类(class)，它的类型都是 ``type`` ，除非我们明确地为它指定一个 `metaclass <http://eli.thegreenplace.net/2011/08/14/python-metaclasses-by-example/>`_ 。因此，当 ``PyObject_Call`` 试图查看Joe的类型，将得到类型 ``type`` ，然后选择 ``type`` 的 ``tp_call`` 属性。换句话说，就是调用 ``Objects/typeobject.c`` 文件中的函数 ``type_call`` [1]。

这是一个有趣并且短小的函数，所以我将它整个地粘贴到这里：

::

    static PyObject *
    type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
    {
        PyObject *obj;

        if (type->to_new == NULL) {
            PyErr_Format(PyExc_TypeError,
                        "cannot create '%.100s' instances",
                        type->tp_name);
            return NULL;
        }

        obj = type->tp_new(type, args, kwds);
        if (obj != NULL) {
            /* Ugly exception: when the call was type(something),
                don't call tp_init on the result. */
            if (type == &PyType_Type &&
                PyTuple_Check(args) && PyTuple_GET_SIZE(args) == 1 &&
                (kwds == NULL ||
                    (PyDict_Check(kwds) && PyDict_Size(kwds) == 0)))
                return obj;
            /* If the returned object is not an instance of type,
                it won't be initialized. */
            if (!PyType_IsSubtype(Py_TYPE(obj), type))
                return obj;
            type = Py_TYPE(obj);
            if (type->tp_init != NULL &&
                type->tp_init(obj, args, kwds) < 0) {
                Py_DECREF(obj);
                obj = NULL;
            }
        }
        return obj; 
    }

在我们的例子中传给 ``type_call`` 的参数是什么呢？第一个是Joe自己---它是如何表示的呢？好吧，Joe是一个类(class)，因此它是一个 *类型(type)* (`Python3中所有类都是类型 <http://eli.thegreenplace.net/2012/03/30/python-objects-types-classes-and-instances-a-glossary/>`_)。而类型在CPython虚拟机内部是通过 ``PyTypeObject`` 对象来表示的[2]。

``type_call`` 首先调用给定类型的 ``tp_new`` 属性。然后，检测一个特殊的情况(为简单起见可忽视先)以确保 ``tp_new`` 返回的是预期类型的对象，然后调用 ``tp_init`` 。如果返回的是一个不同类型的对象，则不将其初始化。

从Python代码来看，就是发生了这些事情：如果你的类中定义了特殊方法 ``__new__`` ，当创建类的一个新实例时，首先调用这个特殊方法。这个方法必须返回某个对象。通常，返回的即是预期类型的对象，但是并非必须如此。所需类型的对象对其自身调用 ``__init__`` 。示例如下：

::

    class Joe:
        def __new__(cls, *args, **kwargs):
            obj = super(Joe, cls).__new__(cls)
            print ('__new__ called. got new obj id=0x%x' % id(obj))
            return obj

        def __init__(self, arg):
            print ('__init__ called (self=0x%x) with arg=%s' % (id(self), arg))
            self.arg=arg

    j = Joe(12)
    print(type(j))

输出如下：

::

    __new__ called. got new obj id=0x7f88e7218290
    __init__ called (self=0x7f88e7218290) with arg=12
    <class '__main__.Joe'>

自定义过程
^^^^^^^^^^^^

如上所示，Joe的类型为 ``type`` ，所以调用函数 ``type_call`` 定义Joe实例的创建过程。可以通过为Joe指定一个自定义的类型来改变这个过程---换句话来说，这种自定义的类型就是一个metaclass。让我们修改前面的示例来为Joe指定一个自定义的metaclass：

::

    class MetaJoe(type):
        def __call__(cls, *args, **kwargs):
            print('MetaJoe.__call__')
            return None

    class Joe(metaclass=MetaJoe):
        def __new__(cls, *args, **kwargs):
            obj = super(Joe, cls).__new__(cls)
            print('__new__ called. got new obj id=0x%x' % id(obj))
            return obj

        def __init__(self, arg):
            print('__init__ called (self=0x%x) with arg=%s' % (id(self), arg))
            self.arg = arg

    j = Joe(12)
    print(type(j))

现在Joe的类型不是 ``type`` ，而是 ``MetaJoe`` 。因此，当 ``PyObject_Call`` 为 ``j = Joe(12)`` 选择要执行的调用函数，它选择的是 ``MetaJoe.__call__`` 。后者先打印一条关于自己的提示，然后返回None，所以我们根本不要期望调用Joe的方法 ``__new__`` 和 ``__init__`` 。

事实上，输出是这样的：

::

    MetaJoe.__call__
    <class 'NoneType'>

更深的挖掘 - tp_new
^^^^^^^^^^^^^^^^^^^^^

很好，现在我们对于对象创建过程有了一个更好的理解，但是这一问题的一个关键部分还没有得到解释。虽然我们几乎总会为类定义方法 ``__init__`` ，但却很少定义 ``__new__`` [3]。此外，快速浏览一下代码就能明显地发现从某种程度上 ``__new__`` 更为重要。这个方法是被用来创建对象的。每个实例仅调用它一次。另一方面，调用 ``__init__`` 时已经得到了一个构造好的对象，且 ``__init__`` 可能根本不会被调用；而且它也可以被调用多次。

在我们的例子中，传递给 ``type_call`` 的参数type是Joe，而Joe并没有自定义的 ``__new__`` 方法，那么 ``type->tp_new`` 的工作将交予基本类型(the base type)的结构成员(slot) ``tp_new`` 。Joe( `以及所有其他的Python对象 <http://eli.thegreenplace.net/2012/04/03/the-fundamental-types-of-python-a-diagram/>`_ ，除了object自己)的基本类型是object。CPython内部是通过Objects/typeobject.c中的object_new函数来实现object.tp_new结构成员的。

``object_new`` 实际上非常简单：先检查某些参数，核实正尝试实例化的类型不是 `抽象 <http://docs.python.org/dev/library/abc.html>`_ 的，然后：

::

    return type->tp_alloc(type, 0)

``tp_alloc`` 是CPython内部类型对象的一个低层次结构成员，不可以在Python代码中直接访问它，但是C扩展开发人员应该对它比较熟悉。C扩展程序中的自定义类型(custom type)可能会重载这个结构成员从而为自己的实例提供一个自定义的内存分配方案。然而，大多数的C扩展类型会将其实例的内存分配工作交予 ``PyType_GenericAlloc`` 函数完成。

这个函数是CPython的公共C API的一部分，也恰好将它赋值给了object的结构成员 ``tp_alloc`` (在Objects/typeobject.c中定义)。它先算出新对象需要多少内存空间[4]，从CPython的内存分配器中分配一个内存块，将分配得的所有内存单元都初始化为0，然后仅初始化基本的PyObject域(类型与引用计数)，做些垃圾收集簿记(GC bookkeeping)的工作并返回。其结果是一个刚分配的实例。

结论
^^^^^

为了避免只见树木不见森林，让我们一起回顾一下文章开始的那个问题。当CPython执行j = Joe()时发生了什么？

    >> 由于Joe没有明确的metaclass，type就是它的类型，因此调用type的tp_call接口，即是，type_call。

    >> type_call一开始就调用Joe的tp_new接口：
        
        >> 由于Joe没有明确的基类(base class)，它的基类(base)就是object(译注:意思就是Joe继承自object)，因此，调用object_new。
        
        >> 由于Joe是一个Python代码定义的类，它没有自定义的tp_alloc接口。因此，object_new调用PyType_GenericAlloc。

        >> PyType_GenericAlloc分配并初始化一块足够大的内存空间用来存储Joe。

    >> 然后type_call继续执行并在刚创建对象上调用Joe.__init__。

        >> 由于Joe没有定义__init__，所以调用它的基类的__init__，即object_init。
        
        >> object_init啥事都没干。

    >> 从type_call返回新的对象并将其绑定到名字j。

如上，就是创建一个类的对象的乏味的流程，这个类没有自定义的metaclass，没有明确的基类，也没有定义它自己的__new__和__init__方法。然而，本文应该已经解释清楚了如何插入自定义功能从而改变对象创建过程。正如你所见的，Python灵活得令人惊讶，几乎可以自定义上述过程的每个步骤，甚至对于Python代码实现的用户定义的类型也是如此。C扩展中实现的类型可以进行更多的自定义，比如：用于创建类型实例的确切的内存分配策略。

------

[1] type的PyTypeObject结构定义即为Objects/typeobject.c中的PyType_Type。你可以看到type_call被赋值给了它的结构成员tp_call。

[2] 以后的文章会解说:当创建一个新类时这是怎么实现的。

[3] 甚至当在类中明确地重载了__new__，我们也几乎可以肯定实际的对象创建被推迟到基类的__new__。

[4] 任何类型的PyObject头部都有这个信息。

------

译注:

1. sequence直译成"序列，顺序"，可能会导致不容易理解文章的意思，所以翻译成" **过程** "更合适一些。

2. CPython中用C语言的结构(struct)来定义对象(类型也是对象)，一个对象所具有的方法或者属性都是结构的成员，所以把slot翻译成" **结构成员** "应该比较合适。
