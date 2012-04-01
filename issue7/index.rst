Issue #7 : __dunder__
=====================

Hi Pythonistas,

本周有大量的有趣项目出现，所以本期的项目小节会十分丰富，以及，邮件列表的新设计将在几周内出现～

现在我们提供 `RSS 订阅 <http://feeds.feedburner.com/pycodersweekly>`_ (需要翻墙， `隐藏秘籍 <http://feeds2.feedburner.com/pycodersweekly>`_ 什么的人家才不会告诉你) 了，同时也有 `往期存档 <http://pycoders.com/archive.html>`_ 可以查阅。

想和我们签订契约成为魔法少女（少年）么， `戳这里 <https://twitter.com/#!/pycoders>`_ 。 (◕‿‿◕)

以及 `这里 <http://wiki.python.org/moin/DunderAlias>`_ 是关于本期标题的解释。

新闻与开发动态
--------------

`Legit 0.1 版本发布 <http://www.git-legit.org/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

想要一个更好的 git 命令行界面么？试试 Legit 吧，这是 Kenneth Reitz 的最新作品，试图将 Github for Mac 为 GUI 用户做的事情移植到命令行界面上。

`Heroku 现在支持插件形式的 Sentry 了 <https://addons.heroku.com/sentry>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sentry，一个由 `David Cramer <https://twitter.com/#!/zeeg>`_ 开发的错误记录工具现在以插件形式在 `Heroku <http://heroku.com/>` 上提供，无需手工搭建及配置，只需启用这个插件就可以了。

`PyCharm 2.5 公开测试版开放下载 <http://blog.jetbrains.com/pycharm/2012/03/pycharm-2-5-public-beta-available-for-download/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

即使我们是 Vim 和 Sublime Text 使用者，但是还是要说一下 PyCharm 发布了 2.5 的公开测试版，如果你在寻找一个全能的 Python IDE，可以去试试 PyCharm。

讨论
----

`像老头滚动条和 Minecraft 这样的复杂游戏能够用 Python 写么？ <http://www.reddit.com/r/Python/comments/rcm78/is_a_game_as_complex_as_something_like_skyrim_or/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

简答： 像老滚那样的游戏是不可能了，但是如果你不介意为了性能写一些 C 代码的话， Minecraft 这样的游戏还是可行的，这篇讨论含金量很高。

`Hacker News 程序员投票 <http://attractivechaos.github.com/HN-prog-lang-poll.png>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是在本周 Hacker News 上做的关于你最喜欢与最不喜欢的编程语言投票的结果，我觉得说 Python 碾压全场完全不过分，这些数据很值得一番研究。

项目
----

`Cuisine <https://github.com/sebastien/cuisine>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

是否希望能够拥有有 Chef 功能的 fabric？ 或者在执着你的 fab 脚本的时候有类似经历？不用再找了， Cuisine 是 Fabric 中一系列函数的子集。

`CLPython <http://common-lisp.net/project/clpython/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在 issue 3 中我们介绍了一个用 Python 实现的 Lisp 方言 Clojure ，而这个项目恰好相反，用 Lisp 实现了一个 Python 语言，不过这并不是它的卖点，实际上他允许你通过 Lisp 访问 Python 库，或者在 Python 中访问 Lisp 库，很棒吧，快去试试～

`Beets <http://beets.radbox.org/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

完美的，不能再完美的管理音乐库的工具， Beets 的最终目的是能够让你的音乐库变的整齐有序，它能够帮助你把音乐归类，通过 MusicBrainz 数据库更新 ID3 信息等。

`RQ - 简单的任务队列 <http://nvie.github.com/rq/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

喜欢 Celery ？但是又讨厌它的复杂？ RQ (Redis Queue) 是一个简单的任务调度 Python 库，能够在后台调度任务并且处理他们，后端采用 Redis ，能够轻松的融合入你的 Web 项目中。

`SciPy Central <http://scipy-central.org/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你使用 Python 进行一些科学研究的话， SciPy Central 提供了很多使用 SciPy 处理科学问题以及相关 Python 工具的代码片段，模块，链接等。

`assertEquals <https://github.com/whit537/assertEquals>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在经过许多争议之后， Testosterone 改名为 assertEquals 了， assertEquals 是一个 “史诗级别” 的 Python 测试接口，提供命令行（CLI/Curses）界面，希望这次改变能够使这个项目更有爱 =v=

博文
----

`使用 Python 建立一个自动玩网页游戏的机器人 <http://active.tutsplus.com/tutorials/workflow/how-to-build-a-python-bot-that-can-play-web-games/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一篇不错的关于如何制作一个如何玩网页游戏机器人的文章，即时你是初学者也能轻松学习这篇文章所教的内容。

:doc:`探索 Python 代码对象 <exploring-python-code-objects>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Python code objects are Python objects that are used to represent Python bytecode. This article is a great introduction to code objects and a nice guided tour to getting started with inspecting them to learn more about Python and its internals.

`Simple <https://github.com/orf/simple>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Simple 是一个使用 Flask 写的 Obtvse 的克隆项目，This saw quite of bit excitement in the entire web development community! Dustin Curtis, made it invite only service; which made the community wanting, this clone was up in less than 24 hrs.

`用 Python 写的贝叶斯网络模型 <http://derandomized.com/post/20009997725/bayes-net-example-with-python-and-khanacademy>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

贝叶斯网络在很多地方都能用到，这是一篇很棒的关于 `Khan Academy <http://derandomized.com/post/20009997725/bayes-net-example-with-python-and-khanacademy>`_ 是如何利用贝叶斯网络的文章，通过贝叶斯网络给用户推荐适合的课程。

`Tornado 中异步 MongoDB 连接 <http://www.10gen.com/presentations/webinar/Asynchronous-MongoDB-with-Python-and-Tornado>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是 A. Jesse Jiryu Davis 发表的一些关于 Python 实时应用程序（ Tornado 与 MongoDB ）的一些视频和幻灯片，如果你对建造实时应用程序感兴趣的话，这篇文章会对你很有帮助。

`使用 Python 与 Phantomjs 建造书签服务 <http://charlesleifer.com/blog/building-bookmarking-service-python-and-phantomjs/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇关于如何使用 Flask 与 Phantom.js 架设你自己的书签服务的文章，这篇文章一步一步的带着你完成整个网站，所以到最后你会在本地得到一个实用的书签服务。
