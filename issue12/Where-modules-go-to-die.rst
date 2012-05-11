模块死于何处？
-------------

原文： `Where modules go to die <http://www.leancrew.com/all-this/2012/04/where-modules-go-to-die/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

当我几个星期之前写" `Python不能与其他程序良好协作 <http://www.leancrew.com/all-this/2012/04/python-doesnt-play-nicely-with-others/>`_ "一文时，并不知道我正在讨论的问题已被 `kenneth Reitz <http://www.kennethreitz.com/>`_  充分叙述过了。我本应该知道 `envoy库 <https://github.com/kennethreitz/envoy>`_ (那篇文章中提及过，并且已被我采用以执行外部过程)背后的程序员的主要兴趣在于使得Python函数库更加简单，常见用途更加明显。

除了envoy，Reitz还是以下代码库的主要开发者:

- `requests <https://github.com/kennethreitz/requests>`_ 简化HTTP通信

- `clint <https://github.com/kennethreitz/clint>`_ 帮助构建命令行工具

- `tablib <https://github.com/kennethreitz/tablib>`_ 处理表格数据，生成多种格式化输出。

他为一个题为"人性化Python"的演讲准备了 `一组幻灯片 <http://python-for-humans.heroku.com/#1>`_ ，是本月初他在一个Django会议所作演讲的简短版本。

.. video:: http://www.youtube.com/watch?feature=player_embedded&v=Q1pe6lHZeNs

在这个简短的演讲中，他大部分时间是在讲述他的request库以及他正试图解决的urllib2存在的问题。Reitz认为标准库中一些模块违背了 `Python之禅 <http://www.python.org/dev/peps/pep-0020/>`_ 的某些准则，特别是:

    ``美胜于丑(Beautiful is better than ugly)``

    ``凡值得说，必易于说(If the implementation is hard to explain, it's a bad idea.)``

以及

    ``唯一的显然是王道(There should be one --- and preferably one --- obvious way to do it)``

演讲中，Reitz描述了对于一个Python新手来说，如果想编写一个涉及HTTP通信的程序，必须筛选的几种可能选择。在我 `关于subprocess的一文 <http://www.leancrew.com/all-this/2012/04/python-doesnt-play-nicely-with-others/>`_ 中也有类似的叙述：标准库至少包含了三种方法用于在Python中调用外部程序。

我猜测，一些标准库之所以令人费解，是因为他们认为必须覆盖所有的情况，那么库的实现因为要处理所有想得到的非常规情况而变得越来越笨重，简单常见的用途反而被淹没在层层对象和方法参数列表之下了。

读者Carl， `评论 <http://www.leancrew.com/all-this/2012/04/ftp-v-ftplib/>`_ 我的另一篇标准库牢骚文时，说道：

    Python“内置电池”的哲学存在的问题是：大量的电池是在Python 2.0时候编写的，至今从未更新过，但是因为已经存在它们，所以实现同样功能的第三方库不会流行起来。

确实如此。标准库是Python的一大特性，但它们似乎要吸光所有的氧气，使得第三方库没有了生存的空间。程序员不情愿编写与标准库功能重复的库，但实现这些功能的标准库是多么低劣啊。只有少数人，比如Reitz，愿意编写与标准库进行竞争的模块。

演讲将近结束的时候，Reitz说，“模块死于标准库”。当然有些言过其实了，却是有道理的。一旦某个库被收入标准集，它就不能有大的改变，因为有太多的程序依赖于它保持稳定---和它的缺陷，特质以及混乱。

Hacker News上的几个Python程序员认为我关于subprocess模块的抱怨是没有事实依据的。我很高兴看到核心开发者 `Nick Coghlan <http://www.boredomandlaziness.org/>`_  认同我的看法，Reitz也是：

    一篇关于subprocess模块令人沮丧的使用体验的精彩文章： `leancrew.com/all-this/2012/… <http://www.leancrew.com/all-this/2012/04/python-doesnt-play-nicely-with-others/>`_
        --- Kenneth Reith( `@kennethreitz <http://twitter.com/#!/kennethreitz>`_ ) `Tue Apr 17 2012 <https://twitter.com/#!/kennethreitz/status/192391385737994240>`_

"精彩"一词有些夸张了，但是很高兴看到在Python社区有实质影响的人意识到了：对于普通程序员来说，语言以及它的函数库应该是易用的。
