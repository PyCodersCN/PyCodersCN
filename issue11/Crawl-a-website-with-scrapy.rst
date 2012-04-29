使用scrapy抓取网站
====================

原文： `Crawl a website with scrapy <http://isbullsh.it/2012/04/Web-crawling-with-scrapy/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

简介
------

本文中，我们将看到怎样从一个网站抓取信息，特别是，当所有页面具有共同的URL模式。我们将会看到怎样使用 `Scrapy <http://scrapy.org/>`_ 做到这一点，Scrapy是一个非常强大却简单的网络爬虫框架。

例如，你也许感兴趣于抓取某个博客的每篇文章的信息，并将这些信息存储在数据库中。为了完成这样的一件事，我们将看到怎么使用 `Scrapy <http://scrapy.org/>`_ 实现一个简单的 `网络爬虫 <https://en.wikipedia.org/wiki/Web_crawler>`_ 来抓取博客并将抽取的信息存入 `MongoDB <http://www.mongodb.org/>`_ 数据库。

本文假设你有一个 `工作的MongoDB服务器 <http://www.mongodb.org/display/DOCS/Quickstart>`_ ，并已安装 ``pymongo`` 和 ``scrapy`` ，这两个python包都可以使用 ``pip`` 进行安装。

如果你从来没玩过 `Scrapy <http://scrapy.org/>`_ ，那么应该先读一下这篇 `简短的教程 <http://doc.scrapy.org/en/latest/intro/tutorial.html>`_ 。

第一步，识别URL模式
---------------------

本例中，我们将看到怎样从 `isbullsh.it <http://isbullsh.it/>`_ 的每篇博文中抽取如下信息：

- 标题(title)

- 作者(author)

- 标签(tag)

- 发布日期(release date)

- 统一资源定位符(url)

幸运地，所有的博文都有相同的URL模式： ``http://isbullsh.it/YYYY/MM/title`` 。这些链接可以从站点主页的不同页面上找到。

我们所需要的是一个爬虫，沿着符合这种模式的链接从目标网页上抓取需要的信息，验证数据的完整性，并存入MongoDB中。

构建网络爬虫
--------------

依据 `教程 <http://doc.scrapy.org/en/latest/intro/tutorial.html>`_ 的说明，创建一个Scrapy项目，得到如下所示的项目结构：

::

    isbullshit_scraping/
    |---isbullshit
    |   |--- __init__.py
    |   |--- items.py
    |   |--- pipelines.py
    |   |--- settings.py
    |   |___ spiders
    |        |--- __init__.py
    |        |--- isbullshit_spiders.py
    |___scrapy.cfg

首先在 ``items.py`` 中定义包含抽取信息的项(item)的结构：

::

    from scrapy.item import Item, Field

    class IsBullshitItem(Item):
        title = Field()
        author = Field()
        tag = Field()
        date = Field()
        link = Field()

然后在 ``isbullshit_spiders.py`` 中实现爬虫：

::

    from scrapy.contrib.spiders import CrawlSpider, Rule
    from scrapy.contrib.linkextractor.sgml import SgmlLinkExtractor
    from scrapy.selector import HtmlXPathSelector
    from isbullshit.items import IsBullshitItem

    class IsBullshitSpider(CrawlSpider):
        name = 'isbullshit'
        start_urls = ['http://isbullsh.it'] # urls from which the spider will start crawling
        rules = [Rule(SgmlLinkExtractor(allow=[r'page/\d+']),follow=True),
            # r'page/\d+' : regular expression for http://isbullsh.it/page/X URLs
            Rule(SgmlLinkExtractor(allow=[r'\d{4}/\d{2}\w+']), callback='parse_blogpost')]
            # r'\d{4}/\d{2}/\w+' : regular expression for http://isbullsh.it/YYYY/MM/title URLs

        def parse_blogpost(self, response):
            ...

我们的爬虫继承自 ``CrawlSpider`` ，因为它"提供了一种便利的机制，通过定义一组规则来跟随链接"。更多的信息请看 `这里 <http://readthedocs.org/docs/scrapy/en/0.14/topics/spiders.html#crawlspider>`_ 。

然后定义了两个简单的规则：

- 跟随指向 ``http://isbullsh.it/page/X`` 的链接

- 使用回调方法 ``parse_blogpost`` 从URL模式 ``http://isbullsh.it/YYYY/MM/title`` 定义的页面中抽取信息

抽取数据
----------

为了从HTML代码中抽取标题，作者，等等，我们将使用 ``scrapy.selector.HtmlXPathSelector`` 对象，这个对象使用了 ``libxml2`` HTML语法分析器。如果你对这个对象不熟悉，那么应该读一读 ``XPathSelector`` `文档 <http://readthedocs.org/docs/scrapy/en/0.14/topics/selectors.html#using-selectors-with-xpaths>`_ 。

现在我们在 ``parse_blogpost`` 方法中定义抽取的逻辑(这里仅定义标题和标签的抽取逻辑，因为其实逻辑基本上都是一样的)：

::

    def parse_blogpost(self, response):
        hxs = HtmlXPathSelector(response)
        item = IsBullshitItem()
        # Extract title
        item['title'] = hxs.select('//header/h1/text()').extract()  # XPath selector for title
        item['tag'] = hxs.select("//header/div[@class='post-data']/p/a/text()").extract()   # Xpath selector for tag(s)
        ...
        return item

**注解** : 为了确认你定义的XPath选择器(selectors)，我建议使用Firebug，Firefox Inspect或者其他类似的工具来检查页面的HTML代码，然后在 `Scrapy shell <http://doc.scrapy.org/en/latest/intro/tutorial.html#trying-selectors-in-the-shell>`_ 中测试选择器，只有当数据位置与你要抓取的所有网页相一致时才有效。

结果存入MongoDB
-----------------

我们想要每次 `parse_blogpost` 方法返回的项都被发送到一个管道中进行数据验证并存入MongoDB中。

首先，我们需要在 ``settings.py`` 中加入一些东西：

::

    ITEM_PIPELINES = ['isbullshit.pipelines.MongoDBPipeline',]

    MONGODB_SERVER = "localhost"
    MONGODB_PORT = 27017
    MONGODB_DB = "isbullshit"
    MONGODB_COLLECTION = "blogposts"

既然已经定义了管道，MongoDB数据库以及数据集合(collection)，下面来看看管道的实现。我们希望确保没有任何缺失的数据(例如：没有标题，或者作者，或者其他信息的博文)。

如下就是 ``pipelines.py`` 文件的内容：

::

    import pymongo

    from scrapy.exception import DropItem
    from scrapy.conf import settings
    from scrapy import log
    class MongoDBPipeline(object):
        def __int__(self):
            connection = pymongo.Connection(settings['MONGODB_SERVER'], settings['MONGODB_PORT'])
            db = connection[settings['MONGODB_DB']]
            self.collection = db[settings['MONGODB_COLLECTION']]

        def process_item(self, item, spider):
            valid = True
            for data in item:
                # here we only check if the data is not null
                # but we could do any crazy validation we want
                if not data:
                    valid = False
                    raise DropItem("Missing %s of blogpost from %s" % (data, item['url']))
            if valid:
                self.collection.insert(dict(item))
                log.msg("Item wrote to MongoDB database %s/%s" %
                        (settings['MONGODB_DB'], settings['MONGODB_COLLECTION']),
                        level=log.DEBUG, spider=spider)
            return item

发布爬虫
-------------

现在，我们需要做的就是切换到项目的根目录，执行：

::

    $ scrapy crawl isbullshit

然后爬虫就会沿着所有指向博文的链接，检索博文的标题，作者名字，日期，等等，验证抽取的数据，如果顺利通过验证则把所有东西存入MongoDB的数据集合中。

相当简洁，是不是？

结论
-----

这个案例有点过分简单化了：所有的URL都有相似的模式，所有链接都是在HTML代码中硬编码的，没有涉及JS。在链接由JS代码生成的情况下，你可能会想试试 `Selenium <http://pypi.python.org/pypi/selenium>`_ 库。你可以给爬虫添加新的规则或者更加复杂的正则表达式，但我只是想演示一下Scrapy是如何工作的，而不是陷入疯狂的正则表达式解说。

而且，请注意，有时，在玩玩网页抓取与 `惹上麻烦 <https://en.wikipedia.org/wiki/Web_scraping#Legal_issues>`_ 之间只是一纸之隔。

最后，在玩网页抓取时，请谨记也许你是在使用大量的请求冲击服务器，有时这会造成你的IP被屏蔽。

请别成了一只 `鸭子(d*ick) <http://cdn.memegenerator.net/instances/400x/19196799.jpg>`_
