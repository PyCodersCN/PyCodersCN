Issue #16: 规则胜于特例
=======================

Hi Pythonistas,

本周的新闻并不是很多，不过贴纸的事情倒是有很多要说的，我们这星期会收到新设计的贴纸，所以赶快给我们寄来回邮吧，我们的地址是：

    44 Byward Market Square, Suite 210
    Ottawa, Ontario Canada 
    K1P 7A2

RSS 订阅和存档可以在 `这里 <http://feeds.feedburner.com/pycodersweekly>`_ 和 `这里 <http://pycoders.com/archive.html>`_ 找到。

`戳这里 <https://twitter.com/#!/pycoders>`_ 和我们签订契约成为魔法少女（少年）。 (◕‿‿◕)

--
Mahdi and Mike 

新闻与开发动态
--------------

`Light Table <http://www.kickstarter.com/projects/ibdknox/light-table?ref=users>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

虽然这个东西在之前的邮件中已经说过了，但是现在还是有必要再说一下， Light Table 现在已经筹集到足够的资金来开发 Python 支持，我们很高兴能够看到像视频中一样的 `Flask <http://flask.pocoo.org/>`_ 支持。

`Radio Free Python - 第八期 <http://radiofreepython.com/episodes/8/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

最新一期的 Radio Free Python ( Python 每月 Podcast )，本期采访了来自冰岛 CCP 公司（以制作 `Eve Online <http://www.eveonline.com/>`_ & `DUST 514 <http://www.dust514.com/>`_ 知名的游戏公司）的 `Kristjan Valur Jonsson <http://blog.ccpgames.com/kristjan/>`_ ，讨论了关于 `Stackless Python <http://www.stackless.com/>`_ 以及其应用在 Eve Online 中的话题。

`Python 3.3.0 Alpha 4 发布 <http://blog.python.org/2012/06/python-33-alpha-4-released.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Python 3.3.0 仍在继续开发，在5月31日发布了 Alpha 4 版本，包含了许多重量级的新特性与漏洞修复，赶快去试一下吧，同时可以在 `bugs.python.org <http://bugs.python.org>`_ 报告发现的漏洞。

讨论
----

`Python 与 C <http://www.reddit.com/r/Python/comments/u9by4/coming_to_python_from_c_writing_this_finally_made/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当一个 C 程序员开始学习 Python 的时候会出现什么样的情况呢？在这篇讨论里，作者贴了一段很简单的从 C 语言转换到 Python 的代码，从而引出了一系列对这段代码的优化，从这篇讨论中，我们或许能够了解如何用更加 Python 的方式来解决问题，如果你刚开始学习 Python，认真的读一下这篇讨论吧。

项目
----

`workflow <https://github.com/mdipierro/workflow>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Workflow 是一个单文件的小项目，用于解决某些周期性的工作，比如删除或移动文件，运行脚本之类的。

`webxray <https://github.com/hackasaurus/webxray>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这个项目的文档中写道："Web X-Ray 眼睛为非技术宅提供了一个简单的方式去剖析页面，了解他们是如何运作的"， webxray 提供了一个书签，在你需要的页面点一下书签就可以方便快捷的得到关于这个页面的所有信息了。

`django-discover-runner <https://github.com/jezdez/django-discover-runner>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

django-discover-runner 是一个基于 unittest2 的 Django 单元测试模块，It solves the problem with the django test runner that it runs tests based on the structure of your Django project. django-discover-runner allows you to place your tests outside the structure of a django project and still allow you to run your test suite.

`dnspython <https://github.com/rthalley/dnspython>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

dnspython 是一个用 Python 写的 DNS 工具箱，换言之， dnspython 是一个一站式的 DNS 解决方案，它几乎支持所有的记录类型，同时也可以被用作解析，区域转移 (zone transfers)和动态更新(Dynamic updates)等。

博文
-----

`Mongrel2 & Circus = 完全控制你的网络堆栈 <http://blog.ziade.org/2012/05/31/mongrel2-amp-circus-full-control-of-your-web-stack/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

由于配置 `Gunicorn <http://gunicorn.org/>`_ 失败.. Tarek Ziadé 决定用 Mongrel2 & Circus 来建立一个 WSGI 栈，自己的进程管理器。这篇文章介绍了如何建立一个网络堆栈，如果你并不想使用传统的 Gunicorn ，可以读一下这篇文章。

`不要使用散点图！ <http://www.chrisstucchio.com/blog/2012/dont_use_scatterplots.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在这篇文章里，作者讨论了为什么散点图对于可视化数据来说很不好，并且建议应该绘制密度图而不是散点图，同时作者也提供了一些生成图形的 Python 代码，这些代码对你的数据展示工作或许会有帮助。

`奇怪的 Python 本地线程 <http://emptysquare.net/blog/pythons-thread-locals-are-weird/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这篇文章很好的解释了 Python 中本地线程的奇怪之处，作者比较了新版本与旧版本 Python 的本地线程的不同之处，读一下吧。

`RQ Tips <http://nvie.com/posts/introducing-rq/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RQ 是最近发布的一个任务队列处理库，对于任务队列我们更常用 Celery ，不过 RQ 仍然是一个不错的选择，因为他仅仅依赖 Redis ，所以比起 Celery ， RQ 要更易于管理，这篇文章中介绍了许多关于在生产环境中部署 RQ 的提示。