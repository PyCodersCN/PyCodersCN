# Python 内部：可调用对象是如何工作的

*【这篇文章所描述的 Python 版本是 3.x，更确切地说，是 CPython 3.3 alpha。】*

在 Python 中，可调用对象 (callable) 的概念是十分基本的。当我们说什么东西是“可调用的”，马上可以联想到的显而易见的答案便是函数。无论是用户定义的函数 (你所编写的) 还是内置的函数 (经常是在 CPython 解析器内由 C 实现的)，他们总是用来被调用的，不是么？

当然，还有方法也可以调用，但他们仅仅是被限制在对象中的特殊函数而已，没什么有趣的地方。还有什么可以被调用呢？你可能知道，也可能不知道，只要一个对象所属的类定义了 `__call__` 魔术方法，它也是可以被调用的。所以对象可以像函数那样使用。再深入思考一点，类也是可以被调用的。终究，我们是这样创建新的对象的：

	class Joe:
		. . . [contents of class]
	
	joe = Joe()

在这里，我们“调用”了 `Joe` 来创建新的实例。所以说类也可以像函数那样使用！

可以证明，所有这些概念都很漂亮地在 CPython 被实现。在 Python 中，一切皆对象，包括我们在前面的段落中提到的每一个东西 (用户定义和内置函数、方法、对象、类)。所有这些调用都是由一个单一的机制来完成的。这一机制十分优雅，并且一点都不难理解，所以这很值得我们去了解。不过首先我们从头开始。

## 编译调用

CPython 经过两个主要的步骤来执行我们的程序：

1. Python 源代码被编译为字节码。
2. 一个虚拟机使用一系列的内置对象和模块来执行这些字节码。

在这一节中，我会粗略地概括一下第一步中如何处理一个调用。我不会深入这些细节，而且他们也不是我想在这篇文章中关注的真正有趣的部分。如果你想了解更多 Python 代码在编译器中经历的流程，可以阅读[这篇文章](http://eli.thegreenplace.net/2010/06/30/python-internals-adding-a-new-statement-to-python/)。

简单地来说，Python 编译器将表达式中的所有类似 `(参数 …)` 的结构都识别为一个调用 [[1]](#ref_id1)。这个操作的 AST 节点叫 `Call`，编译器通过 `Python/compile.c` 文件中的 `compiler_call` 函数来生成 `Call` 对应的代码。在大多数情况下会生成 `CALL_FUNCTION` 字节码指令。它也有一些变种，例如含有“星号参数”——形如 `func(a, b, *args)`，有一个专门的指令 `CALL_FUNCTION_VAR`，但这些都不是我们文章所关注的，所以就忽略掉好了，它们仅仅是这个主题的一些小变种而已。

## CALL_FUNCTION

于是 `CALL_FUNCTION` 就是我们这儿所关注的指令。这是[它做了什么](http://docs.python.org/dev/library/dis.html)：

> ### CALL_FUNCTION(argc)
> 调用一个函数。`argc` 的低字节描述了定位参数 (positional parameters) 的数量，高字节则是关键字参数 (keyword parameters) 的数量。在栈中，操作码首先找到关键字参数。对于每个关键字参数，值在键的上面。而定位参数则在关键词参数的下面，其中最右边的参数在最上面。在所有参数下面，是要被调用的函数对象。将所有的函数参数和函数本身出栈，并将返回值压入栈。

CPython 的字节码由 `Python/ceval.c` 文件的一个巨大的函数 `PyEval_EvalFrameEx` 来执行。这个函数十分恐怖，不过也仅仅是一个特别的操作码分发器而已。他从指定帧的代码对象中读取指令并执行它们。例如说这里是 `CALL_FUNCTION` 的处理器 (进行了一些清理，移除了跟踪和计时的宏)：

	TARGET(CALL_FUNCTION)
	{
		PyObject **sp;
		sp = stack_pointer;
		x = call_function(&sp, oparg);
		stack_pointer = sp;
		PUSH(x);
		if (x != NULL)
			DISPATCH();
		break;
	}

并不是很难——事实上它十分容易看懂。`call_function` 根本没有真正进行调用 (我们将在之后细究这件事)，`oparg` 是指令的数字参数，`stack_pointer` 则指向栈顶 [[2]](#ref_id2)。`call_function` 返回的值被压入栈中，`DISPATCH` 仅仅是调用下一条指令的宏。

`call_function` 也在 `Python/ceval.c` 文件。它真正实现了这条指令的功能。它虽然不算很长，但80行也已经长到我不可能把它完全贴在这儿了。我将会从总体上解释这个流程，并贴一些相关的小代码片段取而代之。你完全可以在你最喜欢的编辑器中打开这些代码。

## 所有的调用仅仅是对象调用

要理解调用过程在 Python 中是如何进行的，最重要的第一步是忽略 `call_function` 所做的大多数事情。是的，我就是这个意思。这个函数最最主要的代码都是为了对各种情况进行优化。完全移除这些对解析器的正确性毫无影响，影响的仅仅是它的性能。如果我们忽略所有的时间优化，`call_function` 所做的仅仅是从单参数的 `CALL_FUNCTION` 指令中解码参数和关键词参数的数量，并且将它们转给 `do_call`。我们将在后面重新回到这些优化因为他们很有意思，不过现在先让我们看看核心的流程。

`do_call` 从栈中将参数加载到 `PyObject` 对象中 (定位参数存入一个元组，关键词对象存入一个字典)，做一些跟综和优化，最后调用 `PyObject_Call`。

`PyObject_Call` 是一个极其重要的函数。它可以在 Python 的 C API 中被扩展。这就是它完整的代码：

	PyObject *
	PyObject_Call(PyObject *func, PyObject *arg, PyObject *kw)
	{
		ternaryfunc call;
		
		if ((call = func->ob_type->tp_call) != NULL) {
			PyObject *result;
			if (Py_EnterRecursiveCall(" while calling a Python object"))
				return NULL;
			result = (*call)(func, arg, kw);
			Py_LeaveRecursiveCall();
			if (result == NULL && !PyErr_Occurred())
				PyErr_SetString(
					PyExc_SystemError,
					"NULL result without error in PyObject_Call");
			return result;
		}
		PyErr_Format(PyExc_TypeError, "'%.200s' object is not callable",
					func->ob_type->tp_name);
		return NULL;
	}

抛开深递归保护和错误处理 [[3]](#ref_id3)，`PyObject_Call` 提取出对象的 `tp_call` 属性并且调用它 [[4]](#ref_id4)，`tp_call` 是一个函数指针，因此我们可以这样做。

先让它这样一会儿。忽略所有那些精彩的优化，*Python 中的所有调用*都可以浓缩为下面这些内容：

* Python 中一切皆对象 [[5]](#ref_id5)。
* 所有对象都有类型，对象的类型规定了对象可以做和被做的事情。
* 当一个对象是可被调用的，它的类型的 `tp_call` 将被调用。

作为一个 Python 用户，你唯一需要直接与 `tp_call` 进行的交互是在你希望你的对象可以被调用的时候。当你在 Python 中定义你的类时，你需要实现 `__call__` 方法来达到这一目的。这个方法被 CPython 直接映射到了 `tp_call` 上。如果你在 C 扩展中定义你的类，你需要自己手动给类对象的 `tp_call` 属性赋值。

我们回想起类本身也可以被“调用”以创建新的对象，所以 `tp_call` 也在这里起到了作用。甚至更加基本地，当你定义一个类时也会产生一次调用——在类的元类中。这是一个有意思的话题，我将会在未来的文章中讨论它。