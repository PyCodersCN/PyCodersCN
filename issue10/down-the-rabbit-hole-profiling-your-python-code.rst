对你的 Python 代码进行性能分析
==============================

原文： `<http://reinout.vanrees.org/weblog/2012/04/18/profiling-python.html>`_ 

译者： `TheLover_Z <http://zhuang13.de>`_ 

从一个输入请求到给出输出结果，这中间有很多的处理过程。有一部分是关于你的代码，还有一部分则是关于内置库。你可能不太关心这个过程，你关心的是给客户的最终产出结果。

我知道一点一点剖析你程序的性能有点儿无趣。 **性能分析** 意味着运行你的代码，让 Python 解释器对你 **所有** 的请求 (耗费的时间，内存等) 做出一份统计。在生产环境的时候不要这么做，因为影响的因素实在是太多太多了。但是这样一份统计可以让你对你的 Python 代码如何运行有更深入的了解，这是非常可贵的。

关于性能分析最有趣的一点就是 **可轻松实现的目标** 。通常情况是有那么几个函数你可以很容易就改进性能：你只需要一点点的努力就可以得到很高的性能提升。但是当你在一个很麻烦的问题上面耗费了大量的时间，换来的只是 2% 的性能提升，这时候性能分析就不是那么有用了。

Python 有很多 `工具 <http://docs.python.org/library/profile.html>`_ 可以用。最广为人知的是 *cProfile* 。 *行分析器* 可以给出每一行的运行时间。

cProfile 这样用：

::

    import cProfile
    cProfile.run('your_method()')

你也可以这么用：

::

    python -m cProfile your_script.py -o your_script.profile

用 ``-o`` 选项你可以得到一个输出文件，然后你可以用 Python 的 `pstats <http://docs.python.org/library/profile.html#module-pstats>`_ 模块来得到一份统计结果。

如果想得到一份可视的结果，你可以用 `run snake run <http://www.vrplumber.com/programming/runsnakerun/>`_ ，它可以呈现出一份树图给你。kcachegrind 也可以，但是你还需要用 ``pyprof2calltree`` 来把 Python 的性能信息转换成 kcachegrind 的。

你需要注意什么：

- 你以前并不太期望的东西。也许你就偶然发现了一个次优的或者看起来有点儿奇怪的东西，然后你可以深入研究一下。

- 如果在某个函数中耗费了大量的时间。那么这个函数可能就是一个可轻松实现的目标。

- 对同一个函数的大量调用。

还有一些你需要关注的：

- 缓存。

- 在内层循环中进行的操作。

- 删除记录信息。尤其注意对 Django 数据库对象的记录：你操作的对象的 ``__inocode()__`` 所调用的可能比你想要的要多，比如说 ``self.parent.xyz...`` 。

除了代码性能分析 (CPU/IO) 以外还有内存性能分析。关于 Django 的小贴士：在调试模式 (缓存查询) 的时候它有 (故意的) 内存泄露。并不是什么大问题，有个印象就可以了。

关于内存性能分析的工具： heapy 和 `meliea <https://launchpad.net/meliae>`_ 。Meliea 很不错，你可以在服务器上面运行，然后拷贝到你的本地电脑上和 *run sname run* 做一个评估。

性能分析很有用并且有时候挺有趣的，但是在用于生产的服务器上面环境不同，怎么进行性能分析？你 *可能* 有数个运行于性能分析模式的 wsgi 进程中的一个，比如一个对于单个 wsgi 进程只产生少量结果的负荷平衡器(with a load balancer that only trickles a few results to that single wsgi process)。

或者你也可以试试 Boaz Leskes 的 `pycounters <http://pycounters.readthedocs.org/>`_ 。

写在最后： **你应该知道这些知识** 。它应该成为你职业工具的一部分。并且，也应该在 IDE 中集成。Komodo 已经集成了，但是 PyCharm 呢？我希望这篇博文可以让 IDE 供应商们明白这个功能的必要性。

一些附加内容：

- Django 有可以让你进行性能分析的中间件，你可以转换到带 GET 参数的特定请求。

- WSGI 也有中间件(比如 `dozer <http://pypi.python.org/pypi/Dozer/>`_)。


中英文对照表
---------------------

- `性能分析 <http://zh.wikipedia.org/wiki/%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90>`_ - profiling

- 可轻松实现的目标 - low hanging fruit

- 中间件 - middleware

- 负荷平衡器 - load balancer