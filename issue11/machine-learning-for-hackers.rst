Will it Python？ － 机器学习 第一章第一部分：加载数据
=====================================================

原文： `<http://slendrmeans.wordpress.com/2012/04/14/will-it-python-machine-learning-for-hackers-chapter-1-part-1-loading-the-data/>`_ 

译者： `TheLover_Z <http://zhuang13.de>`_ 

序言
------

这是我《Will it Python？》系列的第一篇。这个系列记录了我把一个项目从 R 语言(还真是第一次听说这个语言⋯)转移到 Python 的一些经验。我从刚出版的 `Machine Learning for Hackers (MLFH) <http://shop.oreilly.com/product/0636920018483.do>`_ 来讲起，这本书是由 `Drew Conway <http://www.drewconway.com/>`_ 和 `John Miles White <http://johnmyleswhite.com/>`_ 撰写的。

介绍
-------

MLFH 第一章对关于数据的载入、处理、绘图做了一个简短的介绍。为了让这个教程不是那么枯燥，作者使用了 `UFO 目击纪录 <https://github.com/johnmyleswhite/ML_for_Hackers/tree/master/01-Introduction/data/ufo>`_ 的相关资料。

因为这一章主要讲的是数据的载入和处理，他们使用了 `Pandas <http://pandas.pydata.org/>`_ 来进行模拟。尽管这一章并没有太多有趣的东西，但是探索基本的数据处理任务如何在 Python 中被完成还是挺棒的。结果是，尽管是个很简单的玩意儿，但是在 R 语言和 Python 中还是有一些很有趣的差别。

在这一篇中，我会着重讲解在工作环境中调用数据。全部的代码在 `这里 <https://github.com/carljv/Will_it_Python/tree/master/MLFH/CH1>`_ 。

不稳定的数据处理：报错还是忽略？
---------------------------------

未经处理的数据是由 tab 分隔开的文件，作者使用 R 语言的 ``read.delim()`` 来读取数据。数据看起来很平稳的读入了，没有错误也没有警告。数据没有文件头所以作者设置了 ``headers`` 参数为 ``FALSE`` ，并且在载入以后命名了数据框的列。

在 Python 中同样的过程可以用 Pandas 里的 ``read_table()`` 来实现：

::

    ufo = read_table('data/ufo/ufo_awesome.tsv', sep = '\t',
        na_values = '', header = None)

然而这样会引发一个错误，告诉你 “wrong number of columns”(列数有错误)。但是 R 语言却没有这样的提示，这是怎么回事？

事实证明 ``read_table()`` 是正确的。我们来使用 Python 基本的文件输入输出来读取文件的每一行，然后按照 tab 来把每一行分割成列。我们期望每一行都有六列。只要我们发现有一行不是这样的，就中止循环，然后输出错误的行。这样就会告诉我们第一行出错的位置(如果有的话)，并且我们可以看到出错的详细信息。

::

    inpath = 'data/ufo/ufo_awesome.tsv'

    inf = open(inpath, 'r')
    for i, line in enumerate(inf):
        splitline = line.split('\t')
        if len(splitline) != 6:
            first_bad_line = splitline
            print "First bad row:", i
            for j, col in enumerate(first_bad_line):
                print j, col
            break
    inf.close()

然后我们得到下面的输出：

::

    First bad row: 754

    0 19950704

    1 19950706

    2  Orlando, FL

    3

    4 4-5 min

    5 I would like to report three yellow oval lights which passed over Orlando,Florida on July 4, 1995 at aproximately 21:30 (9:30 pm). These were the sizeof Venus (which they passed close by). Two of them traveled one after the otherat exactly the same speed and path heading south-southeast. The third oneappeared about a minute later following the same path as the other two. Thewhole sighting lasted about 4-5 minutes. There were 4 other witnesses oldenough to report the sighting. My 4 year old and 5 year old children were theones who called my attention to the &quot;moving stars&quot;. These objects moved fasterthan an airplane and did not resemble an aircraft, and were moving much slowerthan a shooting star. As for them being fireworks, their path was too regularand coordinated. If anybody else saw this phenomenon, please contact me at:

    6 ler@gnv.ifas.ufl.edu

我们可以看到 754 行出了问题，我们碰到了一个七列的行(六个 tab)。第六列是一个对于 UFO 的很长的描述，并且看起来在长长的描述中有一个 tab 字符，引起了另外的一列。

但是为什么 R 没有这样的问题呢？我们可以在 MLFH 的第 15 页看到具体发生了什么。作者展示了第一列目击纪录日期的数据行 － 它不符合日期格式。比如说这个邮箱 ler@gnv.ifas.ufl.edu 实际上应该是第七列。显然， ``read.delim()`` 从刚开始的几行推断列数，然后把另外的列推到新的行中。

我认为在这点上 Pandas 比 R 强多了。虽然 R 实际上可以帮你加载数据而不显示过多的警告信息，但它还是想当然的把这件事搞砸了。如果你小心一点儿的话还是可以避免的。但是人总有疏忽的时候，就像在这里一样，R 甚至并不会给出警告。

记住，如果你使用了带 ``col.names`` 参数的 ``read.delim()`` ，然后 R 会抛出一个错误，如果它碰到了比指定参数更多的列的一行。

这是一个很无聊的问题，但是这个问题很重要。总结来说就是：

    Lesson 1: R 语言的 ``read.delim()`` 在没有 ``header = TRUE`` 或者 ``col.names`` 的时候是很危险的。如果你一定要加载数据来指出列名应该是什么，试试指定列名然后再次加载数据。

准备加载数据到数据框
----------------------

看到了上面的不足，我们来尝试修复这个问题。

我们有两个选择，都需要一行一行的处理文件。首先，我们可以把多余的数据附加在第六列后面。第六列是个很长的文本描述，附加的列很像是对描述的一个附加内容。但是，我们并不真的关心那么长长的描述，所以说还有一个选择就是直接删掉附加的列。

这个过程的实现在下面。它从原始文件 ``inpath`` 中一行一行的读入，然后把结果写到 ``outpath`` 里。注意这个函数并不返回什么东西。

::

    def ufotab_to_sixcols(inpath, outpath):
        '''
        Keep only the first 6 columns of data from messy UFO TSV file.

        The UFO data set is only supposed to have six columns. But...

        The sixth column is a long written description of the UFO sighting, and
        sometimes is broken by tab characters which create extra columns.

        For these records, we only keep the first six columns. This typically cuts off some of the long description.

        Sometimes a line has less than six columns. These are not written to the output file (i.e., they're dropped from the data). These records are usually so comprimised as to be uncleanable anyway.

        This function has (is) a side effect on the outpath file, to which it writes output.
        '''

        inf = open(inpath, 'r')
        outf = open(outpath, 'w')

        for line in inf:
            splitline = line.split('\t')
            # Skip short lines, which are dirty beyond repair, anyway.
            if len(splitline) < 6:
                continue

            newline = ('\t').join(splitline[ :6])
            # Records that have been truncated won't end in a newline character
            # so add one.
            if newline[-1: ] != '\n':
                newline += '\n'

            outf.write(newline)

        inf.close()
        outf.close()

这个函数做了以下步骤：

    1. 打开输入文件待读，打开输出文件待写。

    2. 从原始文件中读取一行。

    3. 按照 tab 把行分成列，使用 ``split()`` 方法。

    4. 如果这一行被分隔成了少于六列，忽略掉这一行，读下一行。

    5. 否则，将前六列使用 ``join()`` 方法重新粘合在一起。结果是 ``newline`` 。

    6. 如果在 ``newline`` 的结尾没有一个换行符，加上一个。

    7. 往输出文件写入 ``newline`` 。

    8. 对输入文件的每一行重复 2-7 步骤。

注意步骤 4 意味着少于六列(5 个 tab)的行不会被写入。我没有深究为什么有的行那么短并且是不是有办法来修复他们。但是修复并不一定方便和可信。

我运行了这个函数，创建了一个干净的使用 tab 隔开的文件，叫做 ``ufo_awesome_6col.tsv`` 。(输入文件 ``inpath`` 已经被定义过了。)

::

    outpath = 'data/ufo/ufo_awesome_6col.tsv'
    ufotab_to_sixcols(inpath, outpath)

再次尝试 ``read_table()``
-----------------------------

现在我会尝试使用 Pandas 并且再次调用 ``read_table()`` 来记载文件到一个文件框(因为我知道列名应该是什么，我只是把它们传给函数而不是稍后加载它们。)

::

    ufo = read_table('data/ufo/ufo_awesome_6col.tsv', sep = '\t', na_values = '',
                     header = None, names = ['date_occurred',
                                             'date_reported',
                                             'location',
                                             'short_desc',
                                             'duration',
                                             'long_desc'])

这次就没有故障了。我们使用 ``head()`` 和 ``to_string()`` 方法来比较前六行数据，看看 MLFH 14页的表会是什么结果。

::

    ufo.head(6).to_string(formatters =
                        {'long_desc' : lambda x : x[ :21]})

``formatters`` 参数里面的字典告诉 ``to_string()`` 只显示长描述的前 21 个字符。结果如下：

::

       date_occurred  date_reported               location short_desc duration              long_desc
    0       19951009       19951009          Iowa City, IA        NaN      NaN  Man repts. witnessing
    1       19951010       19951011          Milwaukee, WI        NaN   2 min.  Man  on Hwy 43 SW of
    2       19950101       19950103            Shelton, WA        NaN      NaN  Telephoned Report:CA
    3       19950510       19950510           Columbia, MO        NaN   2 min.  Man repts. son&apos;s
    4       19950611       19950614            Seattle, WA        NaN      NaN  Anonymous caller rept
    5       19951025       19951024   Brunswick County, ND        NaN  30 min.  Sheriff&apos;s office

这和作者在 14 页提供的表就一致了。我们开了一个好头。下一篇我们会从数据中挑出我们感兴趣的信息。

中英文对照表
--------------

数据框 － data frame
