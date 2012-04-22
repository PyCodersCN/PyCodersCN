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


