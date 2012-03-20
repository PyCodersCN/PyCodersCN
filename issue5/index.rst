Issue #5 : Jam Packed
=====================

Hi Pythonistas,

本周真是疯狂的一周，本期 PyCoders 充满了各种各样的关于 Python 的新闻。我们尽可能的把在 PyCon 上发生的一切都告诉给你。

我们正在征集一些关于 PyCoders 的推荐语放在我们的首页上，如果你希望在我们的主页上出现你的名字，以及想对我们说一些什么的话，请在 Twitter `@pycoders <https://twitter.com/#!/pycoders>`_ 。我们很期待听到你的声音。

让我们继续吧。

--
Mahdi and Mike

新闻与开发动态
--------------

`Django 的未来与 Python 3 <https://www.djangoproject.com/weblog/2012/mar/13/py3k/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这篇在 Django 网站上的文章表明了 Django 转向 Python 3 的计划与进程，Django 将在 1.5 版本中加入对 Python 3 的支持。

`Django 1.4 RC2 发布 <https://www.djangoproject.com/weblog/2012/mar/14/14rc2/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Django 1.4 RC2 按照开发计划表如期的发布了，这只是一个候选发布版本，建议你可以去试试用一下他，看能不能帮助找到一些 BUG 。

`IronPython 2.7.2 发布 <http://ironpython.codeplex.com/releases/view/74478>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

IronPython 新版添加了 Sqlite3 支持，以及修复了关于 zip 的BUG，以及其他更多的 BUG 修复。

项目
----

`Avalanche: 一个致力于复用性与测试性的 Web 框架 <http://packages.python.org/avalanche/overview.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一个基于 webapp2 构建的 Python Web 框架，使用 jinja2 作为模板语言。这个框架致力于构建可高重用与可测性的代码，我们已经迫不及待的想看看这个框架在现实中应用的情景了。

`EasyDB: 简单的 SQL 持久性框架 <http://www.coderholic.com/easydb-simple-sql-persistence-for-python/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你曾经被数据库的模式，链接，事务性等概念弄得焦头烂额过么？或许你只是需要一个简单的 K-V 数据库存储。试一下 `Ben Dowling <https://twitter.com/#!/coderholic>`_ 开发的 EasyDB 吧，轻松去除烦恼，帮助你集中精神解决程序问题。

`python-modernize <https://github.com/mitsuhiko/python-modernize>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们都不希望接受这个事实，但是，真的是时候去使用 Python 3 了。python-modernize 帮助你转换代码到最新的 Python 版本，基于 2to3 构造，作者是 `Armin Ronacher <https://twitter.com/#!/mitsuhiko>`_ .

讨论
----

`为什么用 C++ 从 stdin 中读入字符串比 Python 还要慢？ <http://stackoverflow.com/questions/9371238/why-is-reading-lines-from-stdin-much-slower-in-c-than-python>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这个讨论从比较 C++ 与 Python 的读入速度开始，又讨论了更多的关于在什么时候 Python 比 C++ 快，以及应该如何优化 C++ 与 Python 程序。

`Python 中的 Goto 语句 <http://stackoverflow.com/questions/6959360/goto-in-python>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在这个讨论中，你会见到 Python 中各种形式的 Goto 语句实现，同样建议可以读一下 `这个版本的 Goto <https://github.com/albertz/playground/blob/master/py_goto.py>`_ 以及在 `Hacker News 上的相关讨论 <http://news.ycombinator.com/item?id=3691472>`_

`为什么我们需要在浏览器中使用 Python <http://www.reddit.com/r/Python/comments/qxmaq/why_we_need_python_in_the_browser/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇很不错的关于在浏览器中添加 Python 支持的文章，我们认为这件事情真的很不错，然后我们就再也听不到 Javascript 开发者的声音了。文章同时也推荐了一些实现这个想法的项目，比如 `Python Webkit <http://www.gnu.org/software/pythonwebkit/>`_ 。

博文
-----

`PEP 20 (Python之禅) 的实例 <http://artifex.org/~hblanks/talks/2011/pep20_by_example.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这篇有趣的文章展示了一个Python之禅 (Zen of Python) 的实例。 值得去试试! 

:doc:`Unicode 之痛 <unipain>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇关于处理在 python 应用中的 Unicode 错误的文章（内含幻灯片）。 `Ned <http://pycoders.us4.list-manage.com/track/click?u=9735795484d2e4c204da82a29&id=cf26dea230&e=33300bf8fc>`_ 阐述了一些 Unicode 已经存在的问题，同时还给予了一些解决你的文本问题的提示。

`Django 1.4 作弊条 <http://media.revsys.com/images/django-1.4-cheatsheet.pdf>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

REVSYS 小组为即将到来的 Django 1.4 准备的很酷的作弊条。

`PyVideo <http://pyvideo.org/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

来看看 PyVideo 吧。在这里你能找到所有过去的和现在的关于 Python 会议的视频。如果视频被放出，那它也许就在这儿。

`Guido 在 2012 PyData 大会上的讨论（视频） <http://marakana.com/s/2012_pydata_workshop_panel_with_guido_van_rossum,1091/index.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

(译者：视频 1.2GB ..正在拖，然后上传 Youku )

如果你对 Python 和在科学界使用 Python 感兴趣，你可以来看看这个由 Google 主办的在 2012 PyData 大会上的讨论视频。

`Kivy 对 iOS 支持 <https://groups.google.com/forum/?fromgroups#!topic/kivy-users/EoujXCpMSSk>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Kivy <http://pycoders.us4.list-manage.com/track/click?u=9735795484d2e4c204da82a29&id=3bf54f489f&e=33300bf8fc>`_ 是一个迅速发展的，为快速开发应用准备的，支持许多创新界面例如多点触控的应用环境。Kivy 现在允许你把应用直接打包成 iOS 应用。Kivy 支持5个平台，所以你可以随意的部署你的应用而不用去更改任何一行代码。

`伟大的创意是令人恐惧的 <http://www.36kr.com/p/89586.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

(译注： 本文已经由 `36氪 <http://www.36kr.com/>`_ 翻译过，不做重复工作)

在 Paul Graham 发表了他的 PyCon 幻灯片之后，他写了这篇和幻灯片中态度一致的博文。

:doc:`你所需要知道的 datetimes <what-you-need-to-know-about-datetimes>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是 Taavi Burns’s slides 在 PyCon 上关于 python datetimes 的发言的幻灯片。处理时间问题很麻烦，所以这个幻灯值得你去看看。

`Stepping Through CPython [视频] <http://www.youtube.com/watch?v=XGF3Qu4dUqk>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

(译者：等着俺慢慢拖吧... )

这是 Larry Hastings 做的一个关于 Cpython 的内部细节的演讲。演讲开始于一个简单的程序，并贯穿着 Cpython 循序渐进。这次演讲的目的主要通过向你展示你所需要了解并且付诸行动的事情，来让你成为 Python 核心开发者。
