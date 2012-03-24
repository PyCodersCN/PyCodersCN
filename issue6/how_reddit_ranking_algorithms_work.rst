Reddit 评级算法的工作原理
==================================

原文： `<http://eli.thegreenplace.net/2012/03/15/processing-xml-in-python-with-elementtree/>`_

译者： `Tiezhen WANG <https://github.com/wangtz>`_ 

.. figure:: http://amix.dk/uploads/reddit.png
   :align: center
   :alt: Reddit Icon

这篇是 `Hacker News 评级算法的工作原理 <http://amix.dk/blog/post/19574>`_ 一文的姊妹篇。这回主要讲的是 `Reddit <http://www.reddit.com/>`_ 是如何对话题和回复进行排序的。Reddit 的评级算法本身是非常容易理解和实现的，这里我想做深入一些的探讨。

第一部分主要是对话题排名的讨论，比如 Reddit 是如何对话题进行排名的。接下来是评论排名的讨论，Reddit 对话题和评论使用了不同的评级算法 (这一点跟
`Hacker News <http://news.ycombinator.com/>`_ 不太一样), Reddit 的评论排名算法是很值得玩味的，它由 Randall
Munroe (`xkcd <http://xkcd.com/>`_ 的作者) 提出.

从话题排名算法的实现说开来
~~~~~~~~~~~~~~~~~~~~~~~~~~

Reddit 公开了他们的代码，很方便就能找到。Reddit 是用 Python 实现的，源代码在 `这里 <http://code.reddit.com/>`_. 排名算法使用了
`Pyrex <http://www.cosc.canterbury.ac.nz/greg.ewing/python/Pyrex/>`_ (一个用来写 Python 的 C 扩展的语言) 来提高性能。这里为了方便说明，我用 Python 写了 `他们的 Pyrex 代码 <http://code.reddit.com/browser/r2/r2/lib/db/_sorts.pyx>`_.

这个算法被称作热排名 (hot ranking),代码如下:

::

    #Rewritten code from /r2/r2/lib/db/_sorts.pyx

    from datetime import datetime, timedelta
    from math import log

    epoch = datetime(1970, 1, 1)

    def epoch_seconds(date):
        """Returns the number of seconds from the epoch to date."""
        td = date - epoch
        return td.days * 86400 + td.seconds + (float(td.microseconds) / 1000000)

    def score(ups, downs):
        return ups - downs

    def hot(ups, downs, date):
        """The hot formula. Should match the equivalent function in postgres."""
        s = score(ups, downs)
        order = log(max(abs(s), 1), 10)
        sign = 1 if s > 0 else -1 if s < 0 else 0
        seconds = epoch_seconds(date) - 1134028003
        return round(order + sign * seconds / 45000, 7)

一下是用数学语言对该算法的描述 (我是在 `SEOmoz <http://www.seomoz.org/blog/reddit-stumbleupon-delicious-and-hacker-news-algorithms-exposed>`_ 看到这个描述的，但是不确定这是否是他们原创的):

.. figure:: http://amix.dk/uploads/reddit_cf_algorithm.png
   :align: center
   :alt: Reddit 使用的算法

发表时间的影响
~~~~~~~~~~~~~~

发表时间和话题排名的影响可以被概括如下:

-  发表时间对排名有很大影响，该算法使得新的话题比旧的话题排名靠前
-  话题的得分不会因为时间的流失而减少，但是新的话题会比旧的话题得分高。这与 Hacker New 的算法不同 (随着时间的发展降低话题的得分)

下图展示了话题得分在好评和差评的数量不变时，随着时间而变化的情况：

.. figure:: http://amix.dk/uploads/reddit_score_time.png
   :align: center
   :alt: Reddit 的话题得分

对数关系
~~~~~~~~

Reddit 的热排序算法使用了对数函数来衡量前面的投票与其他投票的差距：

-  前十个好评和之后的100个，1000个投票有相同的权重。

参见下面的图:

.. figure:: http://amix.dk/uploads/reddit_log_function.png
   :align: center
   :alt: 对数函数产生的影响

去掉对数函数之后（译注：采用线性函数）的效果

.. figure:: http://amix.dk/uploads/reddit_without_log.png
   :align: center
   :alt: 如果没有对数函数

反对票的影响
~~~~~~~~~~~~~~~~~~~~

Reddit 是为数不多的几个可以投反对票的网站。正如上边代码所述，一个话题的得分被定义成：好评数 - 差评数

下边的图可以帮助我们理解：

.. figure:: http://amix.dk/uploads/reddit_up_down.png
   :align: center
   :alt: 差评的影响

这一点对那些同时有大量赞成和反对票的话题（比如说一些有争议的话题）有显著影响。这种话题的排名会比只有赞成票的话题低一些，这也就解释了为什么 kittens 和其他一些没有争议的话题排名如此靠前。

话题排名算法的结论
~~~~~~~~~~~~~~~~~~

-  发表时间是一个非常重要的参数，通常，新的话题要比旧的话题排名靠前
-  前10个好评跟接下来的100个有着同样的权重。比如一个有着10个好评的话题，跟有50个好评的话题有着相似的排名
-  有争议的话题（支持票和反对票的数量相近）要比支持票占大多数的话题排名靠后

再来谈一下 Reddit 的评论排名算法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Reddit 评论排名算法是由xkcd 的 `Randall Munroe <http://xkcd.com/>`_ 提出的。他的这篇博客清楚地解释了排名算法:

-  `reddit 新的评论排名算法 <http://blog.reddit.com/2009/10/reddits-new-comment-sorting-system.html>`_

内容可以归纳如下：

-  热排名算法不适合应用于评论，因为它更偏向于早期发表的评论
-  评论的评级算法应该与发表时间无关
-  Edwin B. Wilson 在1927年提出了一个解决方案，叫做威尔逊得分区间 (Wilson score interval)，它可以被用于信心排序 (The confidence sort)
-  信心排序把投票数当作一个假象的对所有人投票情况的抽样统计，类似于一个投票选举
-  `怎么样避免根据投票平均值排序 <http://www.evanmiller.org/how-not-to-sort-by-average-rating.html>`_ 概述了如何进行信心排序，非常推荐阅读。

深入评论评级代码
~~~~~~~~~~~~~~~~

`\_sorts.pyx <http://code.reddit.com/browser/r2/r2/lib/db/_sorts.pyx>`_ 实现了信心排序算法。我已经用纯 Python 重新实现了原来的 Pyrex 代码，缓存优化相关的代码也被省略了。

::

    #Rewritten code from /r2/r2/lib/db/_sorts.pyx

    from math import sqrt

    def _confidence(ups, downs):
        n = ups + downs

        if n == 0:
            return 0

        z = 1.0 #1.0 = 85%, 1.6 = 95%
        phat = float(ups) / n
        return sqrt(phat+z*z/(2*n)-z*((phat*(1-phat)+z*z/(4*n))/n))/(1+z*z/n)

    def confidence(ups, downs):
        if ups + downs == 0:
            return 0
        else:
            return _confidence(ups, downs)

信心排序使用了 `威尔逊得分区间 <http://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval>`_
数学记法如下：

.. figure:: http://amix.dk/uploads/wilsons_score_interval.png
   :align: center
   :alt: 威尔逊得分区间

公式中参数意义如下：

-  p 是观察到的支持票所占百分比
-  n 是总投票数
-  z\ :sub:`α/2`\ 是 (1-α/2) 位数的标准正态分布

我们把上边的讨论总结如下：

-  信心排序把投票数当作一个假象的对所有人投票情况的抽样统计，类似于一个投票选举
-  信心排序给出的排名有 85% 的可信度
-  投票数越多，85%的信息得分越接近于真实得分
-  威尔逊得分区间对小数量的尝试和极端的概率有着很好的特性  (?)

Randall `在他的一篇博客里 <http://blog.reddit.com/2009/10/reddits-new-comment-sorting-system.html>`_ 有一个很好的例子解释了信心排序是如何给评论作排名的：

    如果一个评论有1个好评，没有差评，它的支持率是100%，但是由于数据量过小，系统还是会把它放到底部。 但如果，它有10个好评，1个差评，系统可能会有足够的信息把他放到一个有着40个好评，20个差评的评论之前。因为我们基本确认当它有了40个好评的时候，它收到的差评会少于20个。最好的一点是，一旦这个算法出错了（算法有15%的失效概率），它会很快拿到更多的数据，因为它被排到了前面。(?)

发表时间的影响：一点儿都没有
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

信心排序的有点在于它跟发表时间无关 (与热排名和 Hacker New 的排序不同)。评论是通过信心和数据采样来评级的。比如，投票数越多，得分也就越准确。

可视化
~~~~~~

现在我们来通过图表看一下信心排序是如何对评论排名的。我们可以借用 Randall 给出的例子:

.. figure:: http://amix.dk/uploads/reddit_confidence_sort.png
   :align: center
   :alt: Reddit 的信心排序

正如你看到的，信心排序并不关心一个评论的投票数，而关心好评数和投票总数或采样大小的相对关系！

排名之外的应用
~~~~~~~~~~~~~~

正如 `Evan
Miller <http://www.evanmiller.org/how-not-to-sort-by-average-rating.html>`_
所说，威尔逊得分区间不仅仅用于排名，他举了3个例子：

-  垃圾邮件检测：看到这个内容并将它标记成垃圾邮件的百分比有多少？
-  创建精华列表：看到这个内容并将它加星标件的百分比有多少？
-  创建最受欢应列表：看到这个内容并将它转发给朋友的百分比有多少？

使用这个算法，只要知道两点：

-  投票的总数
-  投赞成票的数量

知道了这个算法的威力和易用性之后，再想到大部分网站仍在使用最朴素的评级方法就会觉得很吃惊。即使是几十亿美元的大公司，诸如亚马逊 `Amazon.com <http://amazon.com/>`_ 的评级公式也是很简单：
平均得分 = 好评数 / 投票总数。

结论
~~~~

我希望这篇文章对你有用，如果有任何问题或是建议，请留下您的回复。

Happy hacking as always :)

相关阅读
~~~~~~~~

-  `Reddit 的评论排名算法 <http://possiblywrong.wordpress.com/2011/06/05/reddits-comment-ranking-algorithm/>`_,
   讨论了Reddit排序系统的一个bug

