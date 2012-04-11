Deferreds Are A Dataflow Abstraction
====================================

原文: `Deferreds Are A Dataflow Abstraction <http://dreid.org/2012/03/30/deferreds-are-a-dataflow-abstraction/>`_

翻译: `lepture <http://lepture.com>`_

本文的初衷是回应 Duncan McGreggor 博客所载的 《 `和 Guido 谈回调 <http://oubiwann.blogspot.com/2012/03/conversation-with-guido-about-callbacks.html>`_ 》 一文。

首先，我先给出一个jQuery写法的回调的例子::

    def handleResponse(resp):
        def handleParsed(xmlObj):
            firstStyle = xmlObj.xpath('//link[rel="stylesheet"]')[0]

            def handleAnotherResponse(anotherResp):
                  print anotherResp

            getPage(firstStyle['href'], handleAnotherResponse)

        asyncParser(resp.body, onComplete)

    getPage("http://example.com", handleResponse)


这个例子里的回调处理太过荒谬，就我的了解看来，如此多层级回调写法实在是诡异。
我看过很多 JavaScript 的写法，在实际使用中，比 Python 的这种写法更简洁，更易懂。
因为 JavaScript 匿名函数可以包含多条语句，并且可以作为回调用在函数调用中。
（尽管这一特性还有其它可读性方面的副作用 [1]_ ，例如设计只有一个参数的匿名函数，
用来调用另一个只有一个参数的函数 [2]_ ）

如果用这种写法来写回调，自然是没什么人喜欢的。尤其是和非回调写法的代码作比较::

    resp = getPage('http://example.com')
    xmlObj = parser(resp.body)
    firstStyle = xmlObj.xpath('…')[0]
    print getPage(firstStyle['href'])


非回调的写法只用4行，回调的要用8行（如果算上空行的话，有11行）。你觉得 Guido [3]_
认同这种方法？当然不会。第一个例子的代码虽然很烂，但是很好的描述了回调的写法。
我们简单列举一下它的坏处：

1. 一切都变得无序了。
2. 太多缩进了。扁平优于嵌套，但是近比远好 [4]_ ，所以还是嵌套更好
3. The desire for locality causes function definitions to be scattered among statements.
4. 定义了很多无法重用无法测试的函数。

当然你肯定还有其它方面的不满（比如不遵循 PEP8_ 的驼峰命名 [5]_ ），但是这4点才是我想讨论的重点。

顺序很重要
-------------

大部分人都是从上到下从左到右来阅读的，即使你的母语并不是按这个顺序，
但是我确信你读代码还是按这个顺序的。当我们列一个TODO列表时，我们通常会有一个排序，
比如最重要的事情会放在最上面。购物清单一般会把蔬菜列在最开头，然后才会关心其它琐碎的事物，
比如冰淇淋什么的，因为商店也正是这样布局的。电脑程序也是这样，一行一行按我们所设想的来执行，
这也是电脑本身的工作机制。

顺序很重要。尤其是第一次接触代码，顺序就更重要了。

相关语句应该写在一起
--------------------

上面说的第2条和第3条的就是这个意思，就是说，相关性的东西应该放在一起。这也是最基本组织方式，
无论是在厨房还是在代码组织上。

一般来说，当你阅读时，你遇到读不懂的或者不明白的语句和单词时，
你应该会回过头去看看前面几行到底是说什么。也就是说，如果相关性的事物隔的很远的话，
你回头去看所要花费的时间也就更多。

简洁，重用，调试
----------------

我们写函数是为了改善代码的可读性和可测性，我们也会把大块的函数拆成很多个独立功能的小块的函数。
同时这些小块的函数又方便了我们构造其它的函数，因为它小，它只关注某一特定功能，
所以也更方便我们调试。

比如说要构造一个 http 请求，你不会把所有的 socket 调用都写到一个函数吧。
一个可以很好维护的软件，首先要抽象出它里面可重用可调试的模块，
所以为何回调机制不能也做到可重用可调试呢？

更好的方法
----------

我必须承认，对于回调我是爱恨交加的，在事件驱动编程里我不得不去用它。
当然，你或许已经想到了一个更好的处理回调的方法。就是延迟机制与数据流编程。
很多人第一次尝试延迟机制时，和他们写回调很相似::

    d = getPage(…)

    def handleResponse(resp):
        def handleParsed(xmlObj):
            firstStyle = xmlObj.xpath(…)

            def handleOtherResponse(otherResponse):
                print otherResponse

            d3 = getPage(firstStyle['href'])
            d3.addCallback(handleOtherResponse)

        d2 = asyncParser(resp.body)
        d2.addCallback(handleParsed)

    d.addCallback(handleResponse)


尽管这个例子有点不切实际，但它确实已经完美了实现了功能，而且也确实在 Twisted 中有用到。
当然它还是一如继往的烂，和上面的回调写法一样烂。

所以这也不是一个更好的方法，那么更好的方法是什么呢？

**Deferreds as a dataflow abstraction.**

.. image:: deferred-process.png
    :alt: deferred process


这是一张非环状的回调数据链。单个的操作运算相较于整体的信息流结构是微不足道的。
事件的成功结果将会被放到 deferred 回调链中继续执行下一步，如果发生了异常，
就放到异常处理回调链中。同时异常处理回调链中，当执行正常后，又可将处理交给正常回调链。

一个简单的操作链的例子：

::

    getPage(getFirstStyle(parse(getPage("http://example.com")))


上面的例子就是数据流编程。我们前面的例子则不是，因为它的操作集是有副作用的，


::

    d = getPage("http://example.com")
    d.addCallback(parse)
    d.addCallback(getFirstStyle)
    d.addCallback(getPage)
    d.addCallback(printResult)


现在这个例子里，一切都是有序的，相关的操作块在一起，同时操作块也做了很好的切割，
每个小函数都是独立的可测试的单元。

我希望到这里已经阐述清楚了为什么延迟机制要比回调机制好。

::

    urls.flowTo(getPage)
        .flowTo(parse)
        .flowTo(getFirstStyle)
        .flowTo(getPage)
        .flowTo(stdout)

        urls.put("http://example.com")
        urls.put("http://google.com")


现在我们已经封装好了这样一套机制，不仅仅只是提供了处理一个返回的机制，
它甚至可以处理无数个事件。使用这套机制，我们就可以构建实时分布式系统了。

好了，就这样吧。你应该看一下 Storm_ 的 Orc_ 。


.. _PEP8: http://www.python.org/dev/peps/pep-0008/

.. _Storm: https://github.com/nathanmarz/storm/wiki/Tutorial

.. _Orc: http://orc.csres.utexas.edu/

.. [1] 在程式设计语言中，执行一个除产生结果值的函数程式外的函数程式所引起的任何外部作用，而不是指函数所产生的结果值。

.. [2] 译者无法理解，原文为 such as the pattern of defining a single argument anonymous function to invoke a single, single argument function.

.. [3] Python 之父

.. [4] 原文为 Close is better than far

.. [5] 应该是因为作者要写jQuery类的代码，所以用的驼峰命名
