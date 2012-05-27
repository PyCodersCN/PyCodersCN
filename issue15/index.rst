Issue #15: Readability counts.
===============================

Hi, Pythonistas,

在今天早上我们刚刚决定了我们的贴纸设计，你可以在你拿到贴纸之前在 `这里 <http://bit.ly/KfZdUK>`_ 围观贴纸的样子。我们很高兴能在本周收到前几封快递，嘛，“快递”的速度你懂的。

另外一个问题就是上周我们发送的邮件被 Gmail 判定成垃圾邮件了，现在我们做了强力的保护措施，以防止这种事情再次发生，另外我们非常感谢你能在我们发送的邮件上点 “这不是垃圾邮件”。

如果你想要一份贴纸的话，请寄给我们一份回邮到以下的地址：

44 Byward Market Square, Suite 210
Ottawa, Ontario Canada 
K1P 7A2

RSS 订阅和存档可以在 `这里 <http://feeds.feedburner.com/pycodersweekly>`_ 和 `这里 <http://pycoders.com/archive.html>`_ 找到。

`戳这里 <https://twitter.com/#!/pycoders>`_和我们签订契约成为魔法少女（少年）。 (◕‿‿◕)

--
Mahdi and Mike

新闻与开发动态
--------------

`Python 重设计 <http://pythonorg-redesign.readthedocs.org/en/latest/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Python 软件基金会正在征集关于 Python 官方网站的结构，设计，开发以及维护的建议。

`Light Table  <http://www.kickstarter.com/projects/ibdknox/light-table?ref=users>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我相信你已经看过这个牛逼闪闪的 IDE Light Table 的视频了，Light Table 基于一个非常简单的想法：我们需要一个 **真正** 的写代码的环境，不仅仅是一个编辑器或者是一个代码浏览器，太棒了！

`PEP 405 (在 Virtualenvs 中构建) 已经被接纳 <http://www.python.org/dev/peps/pep-0405/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个很好的消息，可选的轻量级 virtualenvs 将在 Python 3.3 中包含。 virtualenvs 是对于那些需要同时进行多个 Python 项目的人所设计的，终于在核心包中见到它了。

讨论
----

`语法错误与结构错误? <http://www.reddit.com/r/Python/comments/twg56/3rd_degree_black_belts_in_python_what_are_the/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇关于在 Python 中新手常犯的错误的讨论，你是否写过这篇讨论中说的错误呢，也许你可以翻翻你以前的代码找到这些错误并且改掉，并且别忘了对照 `PEP 8 <http://www.python.org/dev/peps/pep-0008/>`_ 检查一遍。

`内建命名空间包 <http://www.reddit.com/r/Python/comments/u3cf1/pep_420_is_accepted_and_init_py_wont_be_required/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一篇对于 `PEP 420 <http://www.reddit.com/r/Python/comments/u3cf1/pep_420_is_accepted_and_init_py_wont_be_required/>`_ 决定 ``__init__.py`` 不再成为 Python 包必须包含的文件的讨论，Packages can be created by making sub-directories of directories on sys.path, and will be additive if multiple subdirectories with the same name  are specified. ``__init__.py`` can still be used but will not be required. 

项目
----

`0bin <https://github.com/sametmax/0bin>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

0bin 是一个简单的 Pastebin 服务，你可以利用它很轻松的搭建出来一个允许各种内容上传的 Pastebin 服务，这个想法产生于反对 pastebin 对于上传内容的 `和谐 <http://www.zdnet.com/blog/security/pastebin-to-hunt-for-hacker-pastes-anonymous-cries-censorship/11336>`_ ，在现今各种 Pastebin 站都被要求自我审查的时候，这个项目出现的真是时候。

`libsass <https://github.com/ducksboard/libsaas>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

libsass 是一个针对众多 SaaS 平台的 API 库，它在现有的 SaaS API 上建立了一层抽象层，它会处理所有的与 SaaS 平台进行交互的事情，使你可以把更多的精力放在更重要的事情上，去试试它吧。

`gmvault <https://github.com/gaubert/gmvault>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

gmvault 是一个用于备份你的 Gmail 账户的工具，跨平台，还有其他诸如邮件恢复等实用功能。

`Stallion <http://perone.github.com/stallion/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

"Stallion 是一个为新手提供'简单易用'的 Python 包管理层的工具"， Stallion 的安装过程非常方便，并且提供了一个很不错的 Python 包管理界面。

博文
-----

`造一个“红色大按钮”<http://blog.metricfire.com/2012/05/building-the-big-red-button/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我一直都很想在硬件中使用 Python ，这篇文章简单的介绍了如何通过一些器件和几行代码来建立一个你自己的“红色大按钮”。

`Python 核心开发：如何提交一个补丁 <http://www.blog.pythonlibrary.org/2012/05/22/core-python-development-how-to-submit-a-patch/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

是否对 Python 语言开发感兴趣呢？如果感兴趣的话，这篇文章对你来说是一个很好的指引，作者简短扼要的一步步教你如何创建一个补丁并提交给 Python 开发者。

`My Road to the Python Commit Bit <http://hynek.me/articles/my-road-to-the-python-commit-bit/>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a great post by Hynek Schlawack about his road to getting the commit bit(committer permissions) on CPython. This is a great article to give you a taste of what it takes to get the commit bit in the Python community. Interested? What are you waiting for, start contributing!

`Python 内置的那些牛逼闪亮的函数 <python-built-in-functions-are-awesome-use-them>`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们热爱 Python 的原因之一就是因为 Python 有足够的内置函数去解决你遇到的所有问题，这篇文章重点介绍了那些并不为人所知的一些内置函数。

`Python Fundamentals Tutorial <http://marakana.com/bookshelf/python_fundamentals_tutorial/index.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a very extensive tutorial of Python fundamentals. It starts right from opening the interpreter, and proceeds to touch on topics such as variable, control flow, code organization, regular expressions and lots of other things. Definitely worth reading for the beginner or someone looking to refresh their knowledge on a particular area of Python fundamentals.

`Python Vs. Processing as a first language <http://compscigail.blogspot.ca/2012/05/python-vs-processing-as-first-language.html>`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is an interesting blog post about what is more appropriate as a first programming language, Python or Processing. While the author may be leaning towards processing, MIT would likely disagree with her assessment. 
