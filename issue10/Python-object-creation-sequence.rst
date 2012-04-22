Python对象创建顺序
=====================

[本文讨论的Python版本为3.x]

本文旨在探究Python中新对象的创建过程。正如我在 `前一篇文章 <http://eli.thegreenplace.net/2012/03/23/python-internals-how-callables-work/>`_ 中所解释的，对象的创建只是调用可调用对象的一种特例。考虑这样的一段Python代码：

::

    class Joe:
        pass

    j = Joe()

当j = Joe()被执行时发生了什么呢？Python把它看作对可调用的Joe的一次调用，并且将它路由到内部函数 ``PyObject_Call`` ，将Joe作为PyObject_Call的第一个参数。 ``PyObject_Call`` 根据其第一个参数的类型抽取这个参数的 ``tp_call`` 属性。

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

在我们的例子中传给 ``type_call`` 的参数是什么呢？第一个是Joe自己---它是如何表示的呢？好吧，Joe是一个class，因此它是一个 *type* (`Python3中所有的class都是type <http://eli.thegreenplace.net/2012/03/30/python-objects-types-classes-and-instances-a-glossary/>`_)。而type在CPython内部是通过 ``PyTypeObject`` 对象来表示的[2]。

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

自定义顺序
^^^^^^^^^^^^

如上所示，Joe的类型为 ``type`` ，所以调用函数 ``type_call`` 定义Joe实例的创建顺序。可以通过为Joe指定一个自定义的类型来改变这个顺序---换句话来说，这种自定义的类型就是一个metaclass。让我们修改前面的示例来为Joe指定一个自定义的metaclass：

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

很好，现在
