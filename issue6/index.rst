issue #6: 春分
==============

Hi Pythonistas,

正如我们承诺过的，我们已经建立了往期 PyCoder's Weekly 的存档页，你可以在我们的 `主页 <http://pycoders.com/>`_ 的 ``archives`` 中找到它。我们正在策划让邮件列表变的更方便一些，如果你有什么建议，请直接回复这封邮件告诉我们。同时不要忘记把 `我们 <https://twitter.com/#!/pycoders>`_ 分享给你的朋友～

Enjoy.

--
Mahdi and Mike 

新闻与开发动态
--------------

`在计算机科学或编程教学中使用 Python <http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-00sc-introduction-to-computer-science-and-programming-spring-2011/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MIT 对在计算机科学教学中使用 Python 作了一个介绍。

`Pyramid 网页框架 1.3 版本发布 <http://readthedocs.org/docs/pyramid/en/1.3-branch/whatsnew-1.3.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pyramid 网页框架发布了 1.3 版本，这个版本已经对 Python 3 完美兼容，去他们的网站看看更多的更新内容吧。

`Django 之旅 <http://pressedweb.com/django-djourney/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我很清楚的记得当年为了学习 Web 框架在网上找了好久的教程。不用再找了， `Cory <https://twitter.com/#!/PressedWeb>`_ 给初学者们写了一系列的 Django 相关文章，并且他也是一个初学 Django 的人。

项目
----

`discoball <https://github.com/bkad/discoball>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Discoball 是一个通过正则表达式匹配染色的工具，最初是用 Ruby 写的，后来只用了两次提交就有了很大的提升，第一步删掉所有 Ruby 代码，然后用 Python 写一遍。在日常工作中给你的命令输出加上颜色真的是很棒，比如高亮栈输出中的行号等等。

`Airy: 网页应用框架 <http://airy.letolab.com/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个与众不同的 Web 框架， Airy 并不遵循普通的 HTTP 请求方式，取而代之的是使用 WebSockets （通过 Socket.io ），可惜现在并不支持 SQL 查询，希望这点能够改进下。

`Django Twilio <https://github.com/rdegges/django-twilio>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有没有想过在你的 Django 程序中方便的添加 Twilio 支持？不用在找其他的了， `Randall Degges <https://twitter.com/#!/rdegges>`_ 已经把这个做出来了。

`Email Pie <http://emailpie.com/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这个东西非常有用，特别是在制作一个邮件表单的时候（比如创建一个邮件列表登录页？）。 Email Pie 提供一个 JSON 的 API 接口去验证邮件地址是否合法，而且还会检查比如邮件格式， MX 记录，以及一些明显的拼写错误。

`Django 1.4 最好的模板系统 <https://github.com/xenith/django-base-template>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你正要开始一个全新的 Django 1.4 项目，这将会对你很有帮助。这是一个 Django 1.4 的基本模板，所以你基本上只需要运行 ``django-admin startproject`` 并且将它指定为项目的模板，你的 Django 项目就会基于这个模板。这个项目是基于 `Mozilla 的 Playdoh <https://github.com/mozilla/playdoh>`_ 的工作。即使它不能完全适合你的需要，这儿也是个好地方让你可以 fork 并创建你自己的项目模板。

`pyprocessing - Python 图形开发的类 Processing 环境 <http://code.google.com/p/pyprocessing/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

译注： ``Processing`` 指一门图形开发相关的语言，更多资料参见 `enwp:Processing_(programming_language) <http://en.wikipedia.org/wiki/Processing_(programming_language)>`_

这个项目提供了一个创建类似于 Processing 系统的图形开发环境的 Python 包。 pyprocessing 的后端基于 OpenGL 和 Pyglet 开发，同时也提供动画图形渲染。

讨论
----

`Python 3 转移之痛，要考虑 Python 2/3 都兼容的感觉实在是太糟糕了 <http://www.reddit.com/r/Python/comments/r3qv2/python_3_transition_gripes_im_writing_a_py23/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个关于写一个同时兼容 Python 2/3 程序以及如何达到完整兼容的讨论。如果你打算或者需要维护一个 Python 2 和 3 写的项目，这篇讨论对你会很有帮助。

博文
----

:doc:`Python 魔术方法指南 <a-guide-to-pythons-magic-methods>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇很翔实的关于 Python 内部探究的文章，建议都读一下这篇文章。


:doc:`Reddit 评级算法的工作原理 <how-reddit-ranking-algorithms-work>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

想知道 Reddit 的评级算法是如何工作的么，这篇文章很好的解释了它的工作原理，并且还有一个用我们非常喜爱的语言实现的代码样例。

`Ajax 与 Django <http://brack3t.com/ajax-and-django-views.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

看到有很多人在 Ajax 与 Django 的整合中遇到问题， `Bracket <http://brack3t.com/pages/about-us.html>`_ 的一群家伙们决定写以下应该如何完成这件事的文章，稍微有些长，但是确实是一篇不错的介绍。

:doc:`Python 内部探究：调用的工作原理 <python-internals-how-callables-work>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Eli Bendersky <https://plus.google.com/103282573189025907018>`_ 写了这篇关于在 Python 3.3 中，调用是如何工作的文章，如果你希望能够深入了解 Python 的内部实现细节，这篇文章会很有帮助。

:doc:`在 DotCloud 部署 TurboGears 应用 <deploying-a-turbogears-project-to-dotcloud>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇对于如何在 DotCloud 上搭建 TurboGears 项目的快速指南，如果你是一个 TurboGears 用户这篇文章或许对你很有用。

:doc:`用 ElementTree 在 Python 中解析 XML <processing-xml-in-python-with-element-tree>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这也是一篇由 `Eli Bendersky <https://plus.google.com/103282573189025907018>`_ 写的文章，对于 XML 分析处理方面，这篇文章或许会给你带来新的看法，并且还包含了许多例子供你思考。

`自定义 Bash 中 Fabric 任务的自动补全功能 <http://foobar.lu/wp/2012/03/20/custom-bash-completion-for-fabric-tasks/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Fabric <http://docs.fabfile.org/en/1.4.0/index.html>`_ 是一个对于 Python Hacker 来说非常有用的工具，这里是一个简单的 Bash 自动补全脚本，能够让 Bash 自动补全 Fabric 文件中的命令名字。

.. toctree::
    :hidden:

    how-reddit-ranking-algorithms-work
    processing-xml-in-python-with-element-tree
    python-internals-how-callables-work
    deploying-a-turbogears-project-to-dotcloud
    a-guide-to-pythons-magic-methods
