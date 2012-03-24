用 ElementTree 在 Python 中解析 XML
===========================================

原文： `<http://eli.thegreenplace.net/2012/03/15/processing-xml-in-python-with-elementtree/>`_

译者： `TheLover_Z <http://zhuang13.de>`_ 

当你需要解析和处理 XML 的时候，Python 表现出了它 “batteries included” 的一面。 `标准库 <http://docs.python.org/library/markup.html>`_ 中大量可用的模块和工具足以应对 Python 或者是 XML 的新手。

几个月前在 Python 核心开发者之间发生了一场 `有趣的讨论 <http://mail.python.org/pipermail/python-dev/2011-December/114812.html>`_ ，他们讨论了 Python 下可用的 XML 处理工具的优点，还有如何将它们最好的展示给用户看。这篇文章是我本人的拙作，我打算讲讲哪些工具比较好用还有为什么它们好用，当然，这篇文章也可以当作一个如何使用的基础教程来看。

这篇文章所使用的代码基于 Python 2.7，你稍微改动一下就可以在 Python 3.x 上面使用了。

应该使用哪个 XML 库？
-------------------------
Python 有非常非常多的工具来处理 XML。在这个部分我想对 Python 所提供的包进行一个简单的浏览，并且解释为什么 `ElementTree` 是你最应该用的那一个。

`xml.dom.*` 模块 － 是 `W3C DOM API <http://www.w3.org/DOM/>`_ 的实现。如果你有处理 DOM API 的需要，那么这个模块适合你。注意：在 `xml.dom` 包里面有许多模块，注意它们之间的不同。

`xml.sax.*` 模块 － 是 SAX API 的实现。这个模块牺牲了便捷性来换取速度和内存占用。SAX 是一个基于事件的 API，这就意味着它可以“在空中”（on the fly）处理庞大数量的的文档，不用完全加载进内存（见注释1）。

`xml.parser.expat` － 是一个直接的，低级一点的基于 C 的 `expat` 的语法分析器（见注释2）。 `expat` 接口基于事件反馈，有点像 SAX 但又不太像，因为它的接口并不是完全规范于 `expat` 库的。

最后，我们来看看 `xml.etree.ElementTree` （以下简称 ET ）。它提供了轻量级的 Python 式的 API ，它由一个 C 实现来提供。相对于 DOM 来说，ET 快了很多（见注释3）而且有很多令人愉悦的 API 可以使用。相对于 SAX 来说，ET 也有 `ET.iterparse` 提供了 “在空中” 的处理方式，没有必要加载整个文档到内存。ET 的性能的平均值和 SAX 差不多，但是 API 的效率更高一点而且使用起来很方便。我一会儿会给你们看演示。

**我的建议** 是尽可能的使用 ET 来处理 XML ，除非你有什么非常特别的需要。

ElementTree － 一个 API ，两种实现
---------------------------------------
`ElementTree` 生来就是为了处理 XML ，它在 Python 标准库中有两种实现。一种是纯 Python 实现例如 `xml.etree.ElementTree` ，另外一种是速度快一点的 `xml.etree.cElementTree` 。你要记住： **尽量使用** C 语言实现的那种，因为它速度更快，而且消耗的内存更少。如果你的电脑上没有 `_elementtree` （见注释4） 那么你需要这样做：
::

    try:
        import xml.etree.cElementTree as ET
    except ImportError:
        import xml.etree.ElementTree as ET

这是一个让 Python 不同的库使用相同 API 的一个比较常用的办法。还是那句话，你的编译环境和别人的很可能不一样，所以这样做可以防止一些莫名其妙的小问题。注意：从 Python 3.3 开始，你没有必要这么做了，因为 `ElementTree` 模块会自动寻找可用的 C 库来加快速度。所以只需要 `import xml.etree.ElementTree` 就可以了。但是在 3.3 正式推出之前，你最好还是使用我上面提供的那段代码。

将 XML 解析为树的形式
------------------------
我们来讲点基础的。XML 是一种分级的数据形式，所以最自然的表示方法是将它表示为一棵树。ET 有两个对象来实现这个目的 － `ElementTree` 将整个 XML 解析为一棵树， `Element` 将单个结点解析为树。如果是整个文档级别的操作（比如说读，写，找到一些有趣的元素）通常用 `ElementTree` 。单个 XML 元素和它的子元素通常用 `Element` 。下面的例子能说明我刚才啰嗦的一大堆。（见注释5）

我们用这个 XML 文件来做例子：
::

    <?xml version="1.0"?>
    <doc>
        <branch name="testing" hash="1cdf045c">
            text,source
        </branch>
        <branch name="release01" hash="f200013e">
            <sub-branch name="subrelease01">
                xml,sgml
            </sub-branch>
        </branch>
        <branch name="invalid">
        </branch>
    </doc>

让我们加载并且解析这个 XML ：
::

    >>> import xml.etree.cElementTree as ET
    >>> tree = ET.ElementTree(file='doc1.xml')

然后抓根结点元素：
::

    >>> tree.getroot()
    <Element 'doc' at 0x11eb780>

和预期一样，root 是一个 `Element` 元素。我们可以来看看：
::

    >>> root = tree.getroot()
    >>> root.tag, root.attrib
    ('doc', {})

看吧，根元素没有任何状态（见注释6）。就像任何 `Element` 一样，它可以找到自己的子结点：
::

    >>> for child_of_root in root:
    ...   print child_of_root.tag, child_of_root.attrib
    ...
    branch {'hash': '1cdf045c', 'name': 'testing'}
    branch {'hash': 'f200013e', 'name': 'release01'}
    branch {'name': 'invalid'}

我们也可以进入一个指定的子结点：
::

    >>> root[0].tag, root[0].text
    ('branch', '\n        text,source\n    ')

找到我们感兴趣的元素
------------------------
从上面的例子我们可以轻而易举的看到，我们可以用一个简单的递归获取 XML 中的任何元素。然而，因为这个操作比较普遍，ET 提供了一些有用的工具来简化操作.

`Element` 对象有一个 `iter` 方法可以对子结点进行深度优先遍历。 `ElementTree` 对象也有 `iter` 方法来提供便利。
::

    >>> for elem in tree.iter():
    ...   print elem.tag, elem.attrib
    ...
    doc {}
    branch {'hash': '1cdf045c', 'name': 'testing'}
    branch {'hash': 'f200013e', 'name': 'release01'}
    sub-branch {'name': 'subrelease01'}
    branch {'name': 'invalid'}

遍历所有的元素，然后检验有没有你想要的。ET 可以让这个过程更便捷。 `iter` 方法接受一个标签名字，然后只遍历那些有指定标签的元素：
::

    >>> for elem in tree.iter(tag='branch'):
    ...   print elem.tag, elem.attrib
    ...
    branch {'hash': '1cdf045c', 'name': 'testing'}
    branch {'hash': 'f200013e', 'name': 'release01'}
    branch {'name': 'invalid'}

来自 XPath 的帮助
---------------------
为了寻找我们感兴趣的元素，一个更加有效的办法是使用 `XPath <http://en.wikipedia.org/wiki/XPath>`_ 支持。 `Element` 有一些关于寻找的方法可以接受 XPath 作为参数。 `find` 返回第一个匹配的子元素， `findall` 以列表的形式返回所有匹配的子元素， `iterfind` 为所有匹配项提供迭代器。这些方法在 `ElementTree` 里面也有。

给出一个例子：
::

    >>> for elem in tree.iterfind('branch/sub-branch'):
    ...   print elem.tag, elem.attrib
    ...
    sub-branch {'name': 'subrelease01'}

这个例子在 `branch` 下面找到所有标签为 `sub-branch` 的元素。然后给出如何找到所有的 `branch` 元素，用一个指定 `name` 的状态即可：
::

    >>> for elem in tree.iterfind('branch[@name="release01"]'):
    ...   print elem.tag, elem.attrib
    ...
    branch {'hash': 'f200013e', 'name': 'release01'}

想要深入学习 XPath 的话，请看 `这里 <http://effbot.org/zone/element-xpath.htm>`_ 。

建立 XML 文档
------------------
ET 提供了建立 XML 文档和写入文件的便捷方式。 `ElementTree` 对象提供了 `write` 方法。

现在，这儿有两个常用的写 XML 文档的脚本。

修改文档可以使用 `Element` 对象的方法：
::

    >>> root = tree.getroot()
    >>> del root[2]
    >>> root[0].set('foo', 'bar')
    >>> for subelem in root:
    ...   print subelem.tag, subelem.attrib
    ...
    branch {'foo': 'bar', 'hash': '1cdf045c', 'name': 'testing'}
    branch {'hash': 'f200013e', 'name': 'release01'}

我们在这里删除了根元素的第三个子结点，然后为第一个子结点增加新状态。然后这个树可以写回到文件中。
::

    >>> import sys
    >>> tree.write(sys.stdout)   # ET.dump can also serve this purpose
    <doc>
        <branch foo="bar" hash="1cdf045c" name="testing">
            text,source
        </branch>
    <branch hash="f200013e" name="release01">
        <sub-branch name="subrelease01">
            xml,sgml
        </sub-branch>
    </branch>
    </doc>

注意状态的顺序和原文档的顺序不太一样。这是因为 ET 讲状态保存在无序的字典中。语义上来说，XML 并不关心顺序。

建立一个全新的元素也很容易。ET 模块提供了 `SubElement` 函数来简化过程：
::

    >>> a = ET.Element('elem')
    >>> c = ET.SubElement(a, 'child1')
    >>> c.text = "some text"
    >>> d = ET.SubElement(a, 'child2')
    >>> b = ET.Element('elem_b')
    >>> root = ET.Element('root')
    >>> root.extend((a, b))
    >>> tree = ET.ElementTree(root)
    >>> tree.write(sys.stdout)
    <root><elem><child1>some text</child1><child2 /></elem><elem_b /></root>

使用 iterparse 来处理 XML 流
--------------------------------
就像我在文章一开头提到的那样，XML 文档通常比较大，所以将它们全部读入内存的库可能会有点儿小问题。这也是为什么我建议使用 SAX API 来替代 DOM 。

我们刚讲过如何使用 ET 来将 XML 读入内存并且处理。但它就不会碰到和 DOM 一样的内存问题么？当然会。这也是为什么这个包提供一个特殊的工具，用来处理大型文档，并且解决了内存问题，这个工具叫 `iterparse` 。

我给大家演示一个 `iterparse` 如何使用的例子。我用 `自动生成 <http://www.xml-benchmark.org/generator.html>`_ 拿到了一个 XML 文档来进行说明。这只是开头的一小部分：
::

    <?xml version="1.0" standalone="yes"?>
    <site>
        <regions>
            <africa>
                <item id="item0">
                    <location>United States</location>    <!-- Counting locations -->
                    <quantity>1</quantity>
                    <name>duteous nine eighteen </name>
                    <payment>Creditcard</payment>
                    <description>
                        <parlist>
    [...]

我已经用注释标出了我要处理的元素，我们用一个简单的脚本来计数有多少 `location` 元素并且文本内容为“Zimbabwe”。这是用 `ET.parse` 的一个标准的写法：
::

    tree = ET.parse(sys.argv[2])

    count = 0
    for elem in tree.iter(tag='location'):
        if elem.text == 'Zimbabwe':
            count += 1
    print count

所有 XML 树中的元素都会被检验。当处理一个大约 100MB 的 XML 文件时，占用的内存大约是 560MB ，耗时 2.9 秒。

注意：我们并不需要在内存中加载整颗树。它检测我们需要的带特定值的 `location` 元素。其他元素被丢弃。这是 `iterparse` 的来源：
::

    count = 0
    for event, elem in ET.iterparse(sys.argv[2]):
        if event == 'end':
            if elem.tag == 'location' and elem.text == 'Zimbabwe':
                count += 1
        elem.clear() # discard the element

    print count

这个循环遍历 `iterparse` 事件，检测“闭合的”（end）事件并且寻找 `location` 标签和指定的值。在这里 `elem.clear()` 是关键 —— `iterparse` 仍然建立一棵树，只不过不需要全部加载进内存，这样做可以有效的利用内存空间（见注释7）。

处理同样的文件，这个脚本占用内存只需要仅仅的 7MB ，耗时 2.5 秒。速度的提升归功于生成树的时候只遍历一次。相比较来说， `parse` 方法首先建立了整个树，然后再次遍历来寻找我们需要的元素（所以慢了一点）。

结论
------------
在 Python 众多处理 XML 的模块中， `ElementTree` 真是屌爆了。它将轻量，符合 Python 哲学的 API ，出色的性能完美的结合在了一起。所以说如果要处理 XML ，果断地使用它吧！

这篇文章简略地谈了谈 ET 。我希望这篇拙作可以抛砖引玉。

注释
--------------
注释1：和 DOM 不一样，DOM 将整个 XML 加载进内存并且允许随机访问任何深度地元素。

注释2： `expat <http://expat.sourceforge.net/>`_ 是一个开源的用于处理 XML 的 C 语言库。Python 将它融合进自身。

注释3：Fredrik Lundh，是 `ElementTree` 的原作者，他提到了一些 `基准 <http://effbot.org/zone/celementtree.htm>`_ 。

注释4：当我提到 `_elementtree` 的时候，我意思是 C 语言的 `cElementTree._elementtree` 扩展模块。

注释5：确定你手边有 `模块手册 <http://docs.python.org/library/xml.etree.elementtree.html>`_ 然后可以随时查阅我提到的方法和函数。

注释6： *状态* 是一个意义太多的术语。Python 对象有状态，XML 元素也有状态。希望我能将它们表达的更清楚一点。

注释7：准确来说，树的根元素仍然存活。在某些情况下根结点非常大，你也可以丢弃它，但那需要多一点点代码。

































































