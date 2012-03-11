在 Python 中使用模糊匹配根据发音搜索
====================================

原文地址： http://www.informit.com/articles/article.aspx?p=1848528

译者： `Upsuper <http://upsuper.org/>`_

**当你编写代码搜索数据库时，你不能总是依赖于相信所有的数据项都有正确的拼写。DreamHost 的开发者以及《Python 标准库编程范例》(** `The Python Standard Library by Example <http://www.informit.com/store/product.aspx?isbn=0321767349>`_ **) 的作者 Doug Hellmann 在这篇文章中回顾了一些根据目标的发音，而不是准确的拼写，进行数据库搜索的方法。**

在数据库中搜索人名是一项独特的挑战。对于不同来源和不同年代的数据，你不但不能指望其中名字的拼写是正确的，甚至相同的名字如果多次出现时，它们的拼写都不一定一样。而储存的数据和搜索项之间也有可能因为个人喜好、文化差异、 `同音词 <http://zh.wikipedia.org/wiki/%E5%90%8C%E9%9F%B3%E7%95%B0%E7%BE%A9%E8%AA%9E>`_ 、拼写错误、文盲或仅仅因为在某些时期根本没有标准拼法而出现差异。这些问题在历史学家、谱系学家和其他研究者的手写的文本记录中尤为常见。

一个常用的解决这样的字符串搜索问题的方法是寻找与搜索目标相近的值。但是，使用传统的 `模糊匹配算法 <http://en.wikipedia.org/wiki/Approximate_string_matching>`_ 计算两个任意字符串之间的相似度，代价是很大的，同时它也不适合用于搜索大规模数据集。一个更好的解决方案是为数据库里的每一项预先计算一个哈希值，有一些专门的哈希算法正是为此设计的。这些语音算法
(Phonetic algorithm) 让你可以基于发音，而不是精确的拼写，来比较两个单词或者名字。

早期成果：Soundex
-----------------

一个这样的算法叫做 `Soundex <http://zh.wikipedia.org/wiki/Soundex>`_ ，它由 Margaret K. Odell、Robert C. Russell 于1900年代早期开发出来。由于 Soundex 算法被美国人口普查所使用，并且它正是专门为编码姓名所设计的，因此在谱系相关的环境中十分常见。计算 Soundex 的哈希值，首先保留名字的首字母，之后将后面的辅音字母根据一张简单的对照表转换为数字。舍去元音和连续的数字，并将结果保留到4个字符。

`Fuzzy <http://pypi.python.org/pypi/Fuzzy>`_ 库包含了一个 Python 程序可以使用的 Soundex 实现：

::
    
    #!/usr/bin/env python
    
    import fuzzy
    
    names = ['Catherine', 'Katherine', 'Katarina',
                    'Johnathan', 'Jonathan', 'John',
                    'Teresa', 'Theresa',
                    'Smith', 'Smyth',
                    'Jessica',
                    'Joshua',
                    ]
    
    soundex = fuzzy.Soundex(4)
    
    for n in names:
            print '%-10s' % n, soundex(n)

这个程序 ``show_soundex.py`` 的输出显示出有一部分有相近读音的名字被编码为了相同的哈希值，但结果并不理想：

::
    
    $ python show_soundex.py
    Catherine  C365
    Katherine  K365
    Katarina   K365
    Johnathan  J535
    Jonathan   J535
    John       J500
    Teresa     T620
    Theresa    T620
    Smith      S530
    Smyth      S530
    Jessica    J200
    Joshua     J200

在这个例子中， *Theresa* 和 *Teresa* 产生了相同的 Soundex 值，但 *Catherine* 和 *Katherine* 虽然听起来一样，却因为有不同的首字母而输出了不同的结果。而最后两个名字 *Jessica* 和 *Joshua* 他们一点关系都没有，但却得到了相同的结果，仅仅因为字母 J、S 和 C 都映射到了数字2上，并且算法删除了重复项。这类错误正体现出了 Soundex 的主要缺陷。

超越英语：NYSIIS
----------------

在 Soundex 之后又发展出一些使用不同编码方案的算法，有的基于 Soundex 并改进了对照表，有的从头开始构建了自己的规则。所有这些算法都用不同的方法来处理 `音位 <http://zh.wikipedia.org/wiki/%E9%9F%B3%E4%BD%8D>`_ 以提高精确度。例如70年代，由Robert L. Taft 公布的 `纽约州模式识别与智能系统 <http://en.wikipedia.org/wiki/NYSIIS>`_ (New York State Identification and Intelligence System, NYSIIS) 算法。NYSIIS 最初被用于现在被称作纽约州刑事司法事物司 (New York State Division of Criminal Justice Services) 的部门，用以帮助他们识别他们数据库中的人。由于特别关注了对欧洲和西班牙姓氏中出现的音元的处理，它产生的结果好于 Soundex。

::
    
    #!/usr/bin/env python
    
    import fuzzy
    
    names = ['Catherine', 'Katherine', 'Katarina',
                    'Johnathan', 'Jonathan', 'John',
                    'Teresa', 'Theresa',
                    'Smith', 'Smyth',
                    'Jessica',
                    'Joshua',
                    ]
    
    for n in names:
            print '%-10s' % n, fuzzy.nysiis(n)

在我们的样例数据中， ``show_nysiis.py`` 的输出结果要好于 Soundex：

::
    
    $ python show_nysiis.py
    Catherine  CATARAN
    Katherine  CATARAN
    Katarina   CATARAN
    Johnathan  JANATAN
    Jonathan   JANATAN
    John       JAN
    Teresa     TARAS
    Theresa    TARAS
    Smith      SNATH
    Smyth      SNATH
    Jessica    JASAC
    Joshua     JAS

在这里， *Catherine* 、 *Katherine* 和 *Katariha* 被映射到了相同的哈希值上。而由于 NYSIIS 使用了更多字母， *Jessica* 和 *Joshua* 的错误匹配也被消除了。

新方法：Metaphone
-----------------

由 Lawrence Philips 在1990年发布的 Metaphone 算法是对早期系统如 Soundex 和 NYSIIS 的另一个改进。这种算法比其他的算法要远远复杂得多，因为它包含了许多特殊的规则用于处理拼写不一致和检查辅音与一些元音的组合。一个叫做 Double Metaphone 的升级版算法走得更远，它进一步添加了一些用于处理其他语言的拼写和发音的规则。

::
    
    #!/usr/bin/env python
    
    import fuzzy
    
    names = ['Catherine', 'Katherine', 'Katarina',
                    'Johnathan', 'Jonathan', 'John',
                    'Teresa', 'Theresa',
                    'Smith', 'Smyth',
                    'Jessica',
                    'Joshua',
                    ]
    
    dmetaphone = fuzzy.DMetaphone(4)
    
    for n in names:
            print '%-10s' % n, dmetaphone(n)

除了有更大的编码规则集，Double Metaphone 还为每个输入的单词产生两个可选的哈希值，这让调用者可以实现两级精度的搜索。在我们样例程序的结果中， *Catherine* 和 *Katherine* 的主哈希值是相同的，它们的次哈希值和 *Katarina* 的主哈希值是相同的。这样就发现了 Soundex 无法发现的匹配，同时又降低了结果的权重，不像 NYSIIS 那样完全没有差别。

::
    
    $ python show_dmetaphone.py
    Catherine  ['K0RN', 'KTRN']
    Katherine  ['K0RN', 'KTRN']
    Katarina   ['KTRN', None]
    Johnathan  ['JN0N', 'ANTN']
    Jonathan   ['JN0N', 'ANTN']
    John       ['JN', 'AN']
    Teresa     ['TRS', None]
    Theresa    ['0RS', 'TRS']
    Smith      ['SM0', 'XMT']
    Smyth      ['SM0', 'XMT']
    Jessica    ['JSK', 'ASK']
    Joshua     ['JX', 'AX']

应用语音检索
------------

在你的程序中使用语音检索是非常简单的，但是你也许需要给数据库服务器添加扩展或者给你的程序捆绑第三方库。MySQL、PostgreSQL、SQLite 和 Microsoft SQL Server 都支持使用一个可以直接在查询中调用的字符串函数来计算 Soundex。PostgreSQL 同时也包含了用于计算原始的 Metaphone 和 Double Metaphone 的函数。

对于主流的语言，如 Python、PHP、Ruby、Perl、C/C++ 和 Java，每种算法也都有独立的实现。这些库可以被用于那些没有提供内建的语音算法支持的数据库，如 MongoDB。举例来说，下面的脚本加载一系列的名字到数据库，同时为每个名字预计算他们的哈希值使得将来的搜索更容易：

::
    
    #!/usr/bin/env python
    
    import argparse
    
    import fuzzy
    from pymongo import Connection
    
    parser = argparse.ArgumentParser(description='Load names into the database')
    parser.add_argument('name', nargs='+')
    args = parser.parse_args()
    
    c = Connection()
    db = c.phonetic_search
    dmetaphone = fuzzy.DMetaphone()
    soundex = fuzzy.Soundex(4)
    
    for n in args.name:
            # Compute the hashes. Save soundex
            # and nysiis as lists to be consistent
            # with dmetaphone return type.
            values = {'_id':n,
                            'name':n,
                            'soundex':[soundex(n)],
                            'nysiis':[fuzzy.nysiis(n)],
                            'dmetaphone':dmetaphone(n),
                            }
            print 'Loading %s: %s, %s, %s' % \
                    (n, values['soundex'][0], values['nysiis'][0],
                    values['dmetaphone'])
            db.people.update({'_id':n}, values,
                            True, # insert if not found
                            False,
                            )

在命令行执行 ``mongodb_load.py`` 来保存名字，并且稍后将他们取出来：

::
    
    $ python mongodb_load.py Jonathan Johnathan Joshua Jessica
    Loading Jonathan: J535, JANATAN, ['JN0N', 'ANTN']
    Loading Johnathan: J535, JANATAN, ['JN0N', 'ANTN']
    Loading Joshua: J200, JAS, ['JX', 'AX']
    Loading Jessica: J200, JASAC, ['JSK', 'ASK']
    
    $ python mongodb_load.py Catherine Katherine Katarina
    Loading Catherine: C365, CATARAN, ['K0RN', 'KTRN']
    Loading Katherine: K365, CATARAN, ['K0RN', 'KTRN']
    Loading Katarina: K365, CATARAN, ['KTRN', None]

搜索程序 ``mongodb_search.py`` 让用户可以选择一种哈希函数，然后构建一个 MongoDB 查询来找到所有哈希值与输入的名字匹配的项。

::
    
    #!/usr/bin/env python
    
    import argparse
    
    import fuzzy
    from pymongo import Connection
    
    ENCODERS = {
            'soundex':fuzzy.Soundex(4),
            'nysiis':fuzzy.nysiis,
            'dmetaphone':fuzzy.DMetaphone(),
            }
    
    parser = argparse.ArgumentParser(description='Search for a name in the database')
    parser.add_argument('algorithm', choices=('soundex', 'nysiis', 'dmetaphone'))
    parser.add_argument('name')
    args = parser.parse_args()
    
    c = Connection()
    db = c.phonetic_search
    
    encoded_name = ENCODERS[args.algorithm](args.name)
    query = {args.algorithm:encoded_name}
    
    for person in db.people.find(query):
            print person['name']

在这样例中，结果集里额外返回的值正是我们所需要的，因为它们是正确的匹配项。另一方面我们也看到，用 Soundex 搜索 *Joshua* 再次返回了不相关的值 *Jessica* ：

::
    
    $ python mongodb_search.py soundex Katherine
    Katherine
    Katarina
    
    $ python mongodb_search.py nysiis Katherine
    Catherine
    Katherine
    Katarina
    
    $ python mongodb_search.py soundex Joshua
    Joshua
    Jessica
    
    $ python mongodb_search.py nysiis Joshua
    Joshua

虽然 Soundex 产生的结果比其他的算法差很多，但由于它内建于许多数据库服务器，它仍然被广泛地应用。同时它的简单也让它比 NYSIIS 或 Double Metaphone 更快。在它的结果可以被接受的情况下，它的速度就成为了选择它的决定性因素。

最后的思考
----------

我希望这篇文章给你展示了语音算法可以给你程序增添的搜索特性的力量，以及如何简单地实现他们。你的数据和你想要进行的搜索决定了哪一种算法才是你的正确选择。如果从数据上看，很难决定使用哪一个，或许你可以给用户提供一个选项让他们来选择一个恰当的算法。虽然给用户提供选择会让你需要做更多的工作来建立索引，但这为实验和改善搜索带来了极大的灵活性。很多研究者、历史学家和谱系学家对于这些算法的名字都很熟悉，即使不清楚他们的实现。所以给他们相应的选项应该不会吓跑这些用户。

引用
----

* `The Soundex Indexing System <http://www.archives.gov/research/census/soundex.html>`_ , U.S. National Archives
* \R. \L. Taft, Name Search Techniques (Albany, New York: New York State Identification and Intelligence System, 1970)
* Lawrence Philips, "`The Double Metaphone Search Algorithm <http://drdobbs.com/184401251?pgno=2>`_," Dr. Dobb's (June 1, 2000)
* `Fuzzy <http://pypi.python.org/pypi/Fuzzy>`_

