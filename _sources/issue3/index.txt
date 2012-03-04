issue 3: Code Hard
==================

Hi Pythonistas,

本周的 PyCoder's Weekly 由 `SWIX <http://swixhq.com/>`_ 盛情赞助。

让我们开始吧，本周的关注热点有：使用 Python 进行 iOS 开发， Python 3 的一些新的改进等等。

我们保证下周的时候你们就能在直接在网站上看到之前所有的周刊了，我们刚刚结束了存档页面的设计。同时，像往常一样，欢迎给我们提建议，直接回复这封邮件就行了。

把我们 `分享`_ 给你的朋友吧！

Enjoy.

--
Mahdi and Mik

.. _分享: http://twitter.com/pycoders

新闻与开发动态
--------------

`What's New Celery 2.5`_
^^^^^^^^^^^^^^^^^^^^^^^^

.. _What's New Celery 2.5: http://docs.celeryproject.org/en/latest/whatsnew-2.5.html

任务队列管理库 Celery 在本周发布了新版本，去试试新功能吧！

`AppEngine Python 2.7`_
^^^^^^^^^^^^^^^^^^^^^^^

.. _AppEngine Python 2.7: http://googleappengine.blogspot.in/2012/02/announcing-general-availability-of.html

Python 2.7 运行时终于摘掉了 Beta 的帽子，从实验室毕业了，同时还支持多个流行库（比如 PIL、numpy、lxml 等等），是时候去折腾下GAE了！

讨论
----

`Haskell 与 Python 哪个代码可读性更高？`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Haskell 与 Python 哪个代码可读性更高？: http://www.reddit.com/r/programming/comments/q80nh/haskell_python_and_readability/

关于 Python 与 Haskell 代码可读性的讨论非常激烈。最近有人在 Github 上发了一篇文章，分别用 Haskell 与 Python 实现了一个基于 Trie 树的自动补全程序，这个讨论值得一看。

`Python 3 即将支持 Unicode 标示符`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Python 3 即将支持 Unicode 标示符: http://mail.python.org/pipermail/python-dev/2012-February/116995.html

Guido 批准在 Python 3 中提供 Unicode 标识符的功能，详细信息请在 PEP 414 中查看。

新项目
------

`Python 学习测验`_
^^^^^^^^^^^^^^^^^^

.. _Python 学习测验: http://www.bythemark.com/products/quiz-learn-python/

这是一个非常有爱 iOS应用，用来检测你的 Python 知识，这个应用是为那些想去学习 Python 的人所设计的。

`Travis CI 宣布支持 Python 程序`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Travis CI 宣布支持 Python 程序: http://about.travis-ci.org/blog/announcing_python_and_perl_support_on_travis_ci/

首先普及一下知识， Travis CI 是一个基于云的持续集成项目，就像 Jenkins 那样的。Travis 是最近才出现的新项目，现在他已经能够完美支持 Python 项目了。最近我们试用了下，偶尔会出现一些小问题但是基本上还是不错的。

博文
----

`Python For iOS`_
^^^^^^^^^^^^^^^^^

.. _Python For iOS: http://pythonforios.com/

这是 iOS 平台上的一个非常简洁的 Python IDE，不管你想用它来干啥，这个东西真的是酷！毙！了！Python for iOS 拥有完整的解释器，代码编辑器以及文档浏览器。快用你的 iPad 试试这个东西吧～

:doc:`微框架 Django <django-micro-framework>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇很棒的文章，我们喜欢 Django，但是我们经常不理解为什么很多人觉得 Django 对于小项目来说太笨重了，Software Maniacs 解决了这个问题，让 Django 也能灵活适用于一些小项目，读读看吧，或许会改变你对 Django 的看法。

:doc:`Python 应用程序包 <python-application-package>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

对于 Python 的包分发方式一直有很多的讨论，在这篇文章里 Ian 简单勾勒出了未来的应用开发与部署方案。

`httpie`_
^^^^^^^^^

.. _httpie: https://github.com/jkbr/httpie

这个东西真的很酷， httpie 是一个用 Python 写的类似 cURL 的程序，实际上他只是一个对于 python-request 的包装。基本你能用 cURL 做的事情它都可以做，而且实现方式比 cURL 更加优美，还有各种代码高亮，以及更多的牛逼功能，赶快把它加到你的工具库里面吧。

`脚本语言横向比较`_
^^^^^^^^^^^^^^^^^^^

.. _脚本语言横向比较: http://hyperpolyglot.org/scripting

这是一篇非常完整的关于在不同的语言中实现一种功能的比较报告，这个表格包含了很多代码片段，从显示语言版本到处理文件，无所不包。

`用 Trie 树实现你自己的自动补全程序`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _用 Trie 树实现你自己的自动补全程序: http://v1v3kn.tumblr.com/post/18238156967/roll-your-own-autocomplete-solution-using-tries

想知道自动补全功能是怎么做出来的么，读一下这篇文章你就知道了，这篇文章利用 Python 实现了一个类似 Google 搜索建议的自动补全程序，利用 Python 代码很清晰的展示出了 Trie 树是如何工作的。读完这篇文章后或许你可以看看 Trie 树的维基百科条目。

`PyCon 会议手机版指南`_
^^^^^^^^^^^^^^^^^^^^^^^

.. _PyCon 会议手机版指南: http://guidebook.com/g/pycon2012/

要参加今年位于米帝的 PyCon 会议么？ 今年的 PyCon US 提供了一个手机版的指南，提供 iOS Android 两种版本，你可以利用这个应用查询今年会议的时刻表，查看会场地图，向大会反馈意见等等。

`Clojure 的纯 Python 实现`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Clojure 的纯 Python 实现: https://github.com/halgari/clojure-py

这是一个很有意思的项目，作者认为静态虚拟机对于动态语言来说是非常不合适的，所以他开发了一个用纯 Python 编写的 Clojure 实现以让人们相信虚拟机可以拥有更好的性能，正如 pypy 那样。这个项目的发展方向很令人期待。

:doc:`Python 的闭包和装饰器 <python-closures-and-decorators>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

@fhaard 写的这一系列文章对 Python 的装饰器做了一个非常系统的解析，如果你想学习或复习关于 Python 装饰器的东西，可以看看这篇文章。

:doc:`AST 模块：用 Python 修改 Python 代码 <static-modification-of-python-with-python-the-ast-module>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇关于如何使用 Pyhton 的 AST 模块对 Python 代码进行静态分析和修改的文章。它同时也介绍了关于 CPython 的编译过程以及修改 AST 树的工作过程。在这篇文章的最后，还有一个很有价值的 PyCon 演讲视频，来看看吧！

.. toctree::
    :hidden:

    django-micro-framework
    python-application-package
    static-modification-of-python-with-python-the-ast-module
    python-closures-and-decorators
