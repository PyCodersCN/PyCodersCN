使用scrapy抓取网站
====================

原文： `Crawl a website with scrapy <http://isbullsh.it/2012/04/Web-crawling-with-scrapy/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

简介
------

本文中，我们将看到怎样从一个网站抓取信息，特别是，当所有页面具有共同的URL模式。我们将会看到怎样使用 `Scrapy <http://scrapy.org/>`_ 做到这一点，Scrapy是一个非常强大却简单的网络爬虫框架。

例如，你也许感兴趣于抓取某个博客的每篇文章的信息，并将这些信息存储在数据库中。为了完成这样的一件事，我们将看到怎么使用 `Scrapy <http://scrapy.org/>`_ 实现一个简单的 `网络爬虫 <https://en.wikipedia.org/wiki/Web_crawler>`_ 来抓取博客并将抽取的信息存入 `MongoDB <http://www.mongodb.org/>`_ 数据库。

本文假设你有一个 `工作的MongoDB服务器 <http://www.mongodb.org/display/DOCS/Quickstart>`_ ，并已安装 ``pymongo`` 和 ``scrapy`` ，这两个python包都可以使用 ``pip`` 进行安装。

如果你从来没玩过 `Scrapy <http://scrapy.org/>`_ ，那么你应该先读一下这篇 `简短的教程 <http://doc.scrapy.org/en/latest/intro/tutorial.html>`_ 。

第一步，识别URL模式
---------------------

本例中，我们将看到怎样从 `isbullsh.it <http://isbullsh.it/>`_ 的每篇博文中抽取如下信息：

- 标题(title)

- 作者(author)

- 标签(tag)

- 发布日期(release date)

- url

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


