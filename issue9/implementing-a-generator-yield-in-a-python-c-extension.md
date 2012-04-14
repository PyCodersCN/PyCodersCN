# 使用 Python C 扩展实现生成器/yield

在 Python 中，生成器 (generator) 是一个返回迭代器 (iterator) 对象的函数。虽然有很多方法来实现，不过最优雅和常用的形式是使用 `yield` 语句。

举例来说，这是一个简单的例子：

	def pyrevgen(seq):
		for i, elem in enumerate(reversed(seq)):
			yield i, elem

这里的 `pyrevgen` 函数就是一个生成器。给定一个序列，它将会返回一个迭代器用以逆序输出这个串的元素并附上序号。比如说：

	>>> for i, e in pyrevgen(['a', 'b', 'c']):
	... print(i, e)
	...
	0 c
	1 b
	2 a

这篇文章的目的是说明使用 Python 的 C API ，换言之，在一个 C 的扩展模块中，如何实现相同的功能。我们主要关注的是 Python 3，对于 Python 2 来说原理是一样的，不过细节上可能会有一些差异。

`yield` 是 Python 中一个非常强大的东西，在 C 中没有与之对应的能力 (除非你使用一些协程宏函数，不过这不在我们这里的讨论范围之内)。因此我们不得不显式地返回一个迭代器对象，并且处理迭代的细节。

在 Python 中写一个[迭代器](http://docs.python.org/dev/library/stdtypes.html#iterator-types)我们需要创建一个实现了 `__iter__` 和 `__next__` 特殊函数的类，C API 中与之对应的方法分别是 `tp_iter` 和 `tp_iternext`。

我们来创建一个叫做 `spam` 的新的扩展模块，他将导出一个对象——`revgen` 类型，这个类型可以像上面的 Python 代码一样被调用。换句话说，Python 可以这样使用它：

	import spam
	for i, e in spam.revgen(['a', 'b', 'c']):
		print(i, e)

我们开始吧 (在这篇文章的末尾会给出完整代码的链接)：

	PyTypeObject PyRevgen_Type = {
	    PyVarObject_HEAD_INIT(&PyType_Type, 0)
	    "revgen",                       /* tp_name */
	    sizeof(RevgenState),            /* tp_basicsize */
	    0,                              /* tp_itemsize */
	    (destructor)revgen_dealloc,     /* tp_dealloc */
	    0,                              /* tp_print */
	    0,                              /* tp_getattr */
	    0,                              /* tp_setattr */
	    0,                              /* tp_reserved */
	    0,                              /* tp_repr */
	    0,                              /* tp_as_number */
	    0,                              /* tp_as_sequence */
	    0,                              /* tp_as_mapping */
	    0,                              /* tp_hash */
	    0,                              /* tp_call */
	    0,                              /* tp_str */
	    0,                              /* tp_getattro */
	    0,                              /* tp_setattro */
	    0,                              /* tp_as_buffer */
	    Py_TPFLAGS_DEFAULT,             /* tp_flags */
	    0,                              /* tp_doc */
	    0,                              /* tp_traverse */
	    0,                              /* tp_clear */
	    0,                              /* tp_richcompare */
	    0,                              /* tp_weaklistoffset */
	    PyObject_SelfIter,              /* tp_iter */
	    (iternextfunc)revgen_next,      /* tp_iternext */
	    0,                              /* tp_methods */
	    0,                              /* tp_members */
	    0,                              /* tp_getset */
	    0,                              /* tp_base */
	    0,                              /* tp_dict */
	    0,                              /* tp_descr_get */
	    0,                              /* tp_descr_set */
	    0,                              /* tp_dictoffset */
	    0,                              /* tp_init */
	    PyType_GenericAlloc,            /* tp_alloc */
	    revgen_new,                     /* tp_new */
	};
	
	static struct PyModuleDef spammodule = {
	   PyModuleDef_HEAD_INIT,
	   "spam",                  /* m_name */
	   "",                      /* m_doc */
	   -1,                      /* m_size */
	};
	
	PyMODINIT_FUNC
	PyInit_spam(void)
	{
	    PyObject *module = PyModule_Create(&spammodule);
	    if (!module)
	        return NULL;
	
	    if (PyType_Ready(&PyRevgen_Type) < 0)
	        return NULL;
	    Py_INCREF((PyObject *)&PyRevgen_Type);
	    PyModule_AddObject(module, "revgen", (PyObject *)&PyRevgen_Type);
	
	    return module;
	}

这是创建一个新模块和一个新类型的标准代码。模块初始化函数 (`PyInit_spam`) 添加了一个叫做 `revgen` 的对象到模块，这个对象是类型 `PyRevgen_Type`。通过调用这个对象，用户可以创建这个类型的一个新实例。

下面的结构 (`PyObject` 的子类) 是将用来表示 `revgen` 的实例的：

	/* RevgenState - reverse generator instance.
	 *
	 * sequence: ref to the sequence that's being iterated
	 * seq_index: index of the next element in the sequence to yield
	 * enum_index: next enumeration index to yield
	 *
	 * In pseudo-notation, the yielded tuple at each step is:
	 *  enum_index, sequence[seq_index]
	 *
	*/
	typedef struct {
	    PyObject_HEAD
	    Py_ssize_t seq_index, enum_index;
	    PyObject *sequence;
	} RevgenState;

这里需要特别点出的一个非常有趣的东西是对我们在迭代的序列的引用。每当 `next` 被调用时，迭代器需要它来访问那个序列。

这里是用来创建新的实例的那个函数，它被赋给了 `tp_new`。注意我们没有给 `tp_init` 赋值，所以默认的“什么都不做”的初始化器将会被使用：

	static PyObject *
	revgen_new(PyTypeObject *type, PyObject *args, PyObject *kwargs)
	{
	    PyObject *sequence;
	
	    if (!PyArg_UnpackTuple(args, "revgen", 1, 1, &sequence))
	        return NULL;
	
	    /* We expect an argument that supports the sequence protocol */
	    if (!PySequence_Check(sequence)) {
	        PyErr_SetString(PyExc_TypeError, "revgen() expects a sequence");
	        return NULL;
	    }
	
	    Py_ssize_t len = PySequence_Length(sequence);
	    if (len == -1)
	        return NULL;
	
	    /* Create a new RevgenState and initialize its state - pointing to the last
	     * index in the sequence.
	    */
	    RevgenState *rgstate = (RevgenState *)type->tp_alloc(type, 0);
	    if (!rgstate)
	        return NULL;
	
	    Py_INCREF(sequence);
	    rgstate->sequence = sequence;
	    rgstate->seq_index = len - 1;
	    rgstate->enum_index = 0;
	
	    return (PyObject *)rgstate;
	}

这是个非常直观的 `tp_new` 实现，它保证了要迭代的对象是一个序列，同时初始化了状态，准备好第一次 `next` 调用时将要返回的最后一个元素。与之对应的销毁函数也并没有什么特别的：

	static void
	revgen_dealloc(RevgenState *rgstate)
	{
	    /* We need XDECREF here because when the generator is exhausted,
	     * rgstate->sequence is cleared with Py_CLEAR which sets it to NULL.
	    */
	    Py_XDECREF(rgstate->sequence);
	    Py_TYPE(rgstate)->tp_free(rgstate);
	}

现在剩下的就是看看 `tp_iter` 和 `tp_iternext` 的实现。因为我们的类型是一个迭代器，我们可以简单的将 `PyObject_SelfIter` 函数赋给 `tp_iter`。`tp_iternext` 才是发生有趣的事情的地方。它正是执行内置函数 `next` 时真正调用的东西，也是 `for` 循环使用迭代器时调用的：

	static PyObject *
	revgen_next(RevgenState *rgstate)
	{
	    /* seq_index < 0 means that the generator is exhausted.
	     * Returning NULL in this case is enough. The next() builtin will raise the
	     * StopIteration error for us.
	    */
	    if (rgstate->seq_index >= 0) {
	        PyObject *elem = PySequence_GetItem(rgstate->sequence,
	                                            rgstate->seq_index);
	        /* Exceptions from PySequence_GetItem are propagated to the caller
	         * (elem will be NULL so we also return NULL).
	        */
	        if (elem) {
	            PyObject *result = Py_BuildValue("lO", rgstate->enum_index, elem);
	            rgstate->seq_index--;
	            rgstate->enum_index++;
	            return result;
	        }
	    }
	
	    /* The reference to the sequence is cleared in the first generator call
	     * after its exhaustion (after the call that returned the last element).
	     * Py_CLEAR will be harmless for subsequent calls since it's idempotent
	     * on NULL.
	    */
	    rgstate->seq_index = -1;
	    Py_CLEAR(rgstate->sequence);
	    return NULL;
	}

在这里，应该被牢记的最重要的一点是迭代的状态应该被完全地保存在迭代器对象中。相比与 Python 实现，这需要多做许多工作。Python 的 `yield` 语句让我们可以用 Python 解析器自己来为我们保存这些状态。这也是为什么[在 Python 中协程如此强大](http://eli.thegreenplace.net/2009/08/29/co-routines-as-an-alternative-to-state-machines/)——几乎没有显式的状态需要手工保存。正如我在文章一开始就提到的，在 C 扩展中我们没有这样的奢侈品，所以我们不得不自己动手。由于现在这个例子还非常简单，而且是线性的，这还相对简单一些。在更复杂的例子中，为了正确地设计状态对象和 `tp_iternext` 函数，需要更用心。

这篇文章的完整代码以及简单的 Python 测试脚本，还有用于使用 distutils 构建这个扩展的 `setup.py` 都可以[在这里下载](http://eli.thegreenplace.net/wp-content/uploads/2012/04/generator_c_ext.tgz)。