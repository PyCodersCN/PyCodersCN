issue 4: 我们没有跳舞机器人
===========================

Hi Pythonistas,

本期 PyCoders 依然由 `SWIX <http://swixhq.com/>`_ 特别赞助。

嘛，PyCon 现在正在举行中，如果你在会场的话，告诉某两个人 `PyCoders <http://pycoders.com/>`_ 的话，我们说不定会为你打开那个跳舞机器人哟，如果你真的想这么做，就来加入我们吧～本周依然有很多的好东西，比如作为 Python 中的 Rake 的项目 Shovel，以及更多的新鲜玩意儿～

同时不要忘记把 `我们 <https://twitter.com/#!/pycoders>`_ 推荐给你的朋友～

P.S. 如果你真的很想知道那个跳舞机器人是啥，偷偷地告诉你：就是那个今年 PyCon 开场的会跳舞的机器人哟。(译者注：本届 PyCon 大会的开场视频可以在 `这里 <http://pyvideo.org/video/622/introduction-and-welcome>`_ 看到)

--
Mahdi and Mike 

新闻与开发动态
--------------

`PyCon US 本周开始举行了！ <https://us.pycon.org/2012/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Yay~

`Mezzanine 1.0 发布 <https://groups.google.com/forum/?fromgroups#!topic/django-users/x5hBMZe28ps>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Mezzanine 是一个 Django 为框架的 CMS 开源系统，最近发布了 1.0 版本，Mezzanine 是一个非常成熟的 CMS 系统，你可以看到有 `这么多 <http://mezzanine.readthedocs.org/en/latest/overview.html#sites-using-mezzanine>`_ 网站都选择了 Mezzanine ，并且你可以整合 Cartridge 使 Mezzanine 变成一个电子商务平台。 `Mezzanine 官方网站 <http://mezzanine.jupo.org/>`_

`历年 PyCon 视频 (2009-2011) <http://blip.tv/pycon-us-videos-2009-2010-2011>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本周是 PyCon 周，来这里看看你有没有错过什么 PyCon 上的好东西吧 (如果你都看过了，无视这个好了……)

讨论
----

`PyCon 实时讨论 <https://pycon.disqus.com/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

PyCon 实时讨论是基于 Disqus 搭建的一个 web 应用，你可以在这里问问题，向大会反馈意见以及找到志同道合的朋友。找到一个喜欢的话题进去讨论吧，或者建立一个你感兴趣的话题。

`Stack Overflow: 给人类看的解释：元类 (metaclass) 是什么? <http://www.reddit.com/r/Python/comments/qkg58/so_what_is_a_metaclass_for_humans/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我相信你们应该都在 Stack Overflow 上看过这篇文章了，SO (Stack Overflow，下同) 上曾经也对 Python 的 `修饰器 <http://stackoverflow.com/questions/739654/understanding-python-decorators/1594484#1594484>`_ ， `yield <http://stackoverflow.com/questions/231767/the-python-yield-keyword-explained/231855#231855>`_ 进行过很好的解释。我们认为这篇文章对于 Python 中元类的解释很有价值，同时在 Reddit 上的讨论也很不错，去看看吧。

项目
----

`Shovel <https://github.com/seomoz/shovel>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Python 中的 Rake ，轻松的在命令行中执行或调用 Python 函数，最近还添加了一项新特性：在浏览器中来执行这些任务 (给懒人的特性)。

`Cyclone <http://cyclone.io/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

底层的 HTTP 处理工具包，支持 HTTP 1.1 协议，提供和 Tornado 类似的 API 调用，但是实际上是用 twisted 写的，将两个项目融合在了一起，他们说性能和易用性都有极大的提升。

`Siri Server Core <https://github.com/Eichhoernchen/SiriServerCore>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一个 Siri 服务器 (不是代理) 的纯 python 实现。不幸的是，只有 iPhone 4S 可以跟苹果服务器进行通讯。这个项目尝试用 Google 语音识别 API 来重造这个 Siri 服务器。

`Sublime 自动刷新插件 <https://github.com/gcollazo/BrowserRefresh-Sublime>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
安装这个插件之后，按下 ``command + shift + r`` 键会自动切换到 Google Chrome 中并且自动刷新当前页，如果你在按之前没有保存文件，这个插件会替你保存然后再刷新。

博文
----

:doc:`Django 新的密码验证方法评测 <a-review-of-Django-s-new-password-authentication>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一个关于 Django 的新密码的哈希能力的简要概述，和使用开源或私人文档 (源代码) 的关于使用在 1.4 上的默认哈希算法差异的一些有趣的评论。

:doc:`Django 生产开发最佳设置 <django-settings-for-production-and-development-best-practices>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Django 设置是很困难的一件事，每个人都似乎有不同的方式做到这一点，这些方法也都有自己的优缺点。这篇文章中，作者列出了3种不同的方式为你的不同的环境做环境特定设置，他还列出了大量的能让你作出判断的材料。这很值得一试，即使你已经做了设置，它总是能够让你在解决相同问题上发现不同观点。

:doc:`Python 阅读书目推荐 <a-python-reading-list>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个能够令 Python 开发者和 Python 新手扩充知识的很棒的书单。这个书单相当完整，我强烈推荐你看看这个书单。这本书单唯一的遗漏是一本侧重于最佳实践，优化，管理代码和一些不同的编程范式教学的叫做 `Python 高级编程 <http://www.packtpub.com/expert-python-programming/book>`_ 的书。

`pip 并行下载 <https://gist.github.com/1971720>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这段代码漂亮整洁的展示了 pip 并行化下载的可能性。其中任务调度是基于 gevent 的。这个想法令我们眼前一亮，我们希望 pip 官方能够实现这一特性，这样就可以在有着繁多继承关系的时候更快的部署系统。

:doc:`在 Python 中使用模糊匹配根据发音搜索 <using-fuzzy-matching-to-search-by-sound-with-python>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Doug Hellmann <http://pycoders.us4.list-manage.com/track/click?u=9735795484d2e4c204da82a29&id=ef5272fe0d&e=33300bf8fc>`_ 在这篇文章中介绍的各种方法和算法可以用来解决数据库内搜索相同发音单词的要求。文章中包含一些示例代码，所以，通过这篇文章，你能够搞定问题。


`Python 程序员的发展历程 <https://gist.github.com/289467>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在这份 newsletter 中，你也许不会发现太多幽默的文章，虽然这篇文章在几年前就已经发表了，但是真的很有意思，所以我们认为你值得看一看。

:doc:`如何过渡至 Python 3 <how-python3-should-have-worked>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Aaron Shwartz <https://twitter.com/#!/aaronsw>`_ 是 Reddit 的创始人和 `web.py <http://webpy.org/>`_ 的作者。他有一些关于如何实现推广 Python 3 的有趣想法。他的主要观点是 Python 3 的过渡方式应该同 Python 2.x 系列保持一致，令 Python 3 支持文件所以你可以只用 ``from __future__ import python3`` 。我们不认为这些想法能够改变什么，但是绝对值得你一读。

目录译者： `wangtz <https://github.com/wangtz>`_ & `Yueh <http://yueh.me>`_ & `Zeray Rice <http://www.fanhe.org>`_ 

感谢 `bcxx <http://youknowmymind.com/>`_ 的校对工作。

.. toctree::
    :hidden:
    
    a-review-of-Django-s-new-password-authentication
    django-settings-for-production-and-development-best-practices
    a-python-reading-list
    using-fuzzy-matching-to-search-by-sound-with-python
    how-python3-should-have-worked
