Python Web 框架一览
===========================================

原文： `<http://www.konstruktor.ee/blog/python-web-framework-roundup/>`_

译者： `TheLover_Z <http://zhuang13.de>`_ 

Python 有许多 web 框架可以供你选择。网上甚至还有教你怎么制作自己专属的框架的教程，因为这实在是太容易了。然后就导致了现在框架的质量参差不齐。我们来对这些框架做一个概述然后你可以挑出自己喜欢的。

概述
----------
我们经常谈到的 python web 框架有这些：

名字  版本  最后更新  诞生时间  代码行数  

Django  1.3.1  2011-09-09  2005  115759  

Flask  0.8  2011-09-29  2010  4681  

Bottle  0.10.9  2012-02-11  2009  4634  

Tornado  2.2  2012-01-30  2009  11701  

Cherry.py  3.2.2  2011-10-19  2002  18828  

web.py  0.36  2011-07-04  2006  7398  

Brubeck  0.3.7  2011-12-20  011  1525  

注释1：加上 Brubeck 是因为我认为我们可以从这个框架身上学到很多，虽然这个框架已经不是 Python 框架了。

注释2：还有许多其他的框架比如说 Zope， Pylons， Pyramid － 我之所以没有写它们是因为我对它们没有经验。

注释3：如果 Flask 的代码加上 Werkzeug 和 Jinja2 的话一共约 35000 行。

注释4：代码行数意思是实际的 Python 代码。使用 `CLOC <http://cloc.sourceforge.net/>`_ 计数。

Django
-----------

这可能是最广为人知和使用最广泛的 Python web 框架了。我承认它确实非常强大。Django 有世界上最大的社区，最多的包，可以说只有你想不到的，没有它做不到的。它的文档非常完善，但是有的比较冷门的知识你还是需要去 StackOverflow 咨询一下。

Django 在配置上面遵循惯例，这样对于初学者来说比较容易，而且在比较复杂的应用上也有一定的灵活性。Django 致力于快速开发以及简洁实用的设计。

Django 只需要这么 `几行代码 <http://olifante.blogs.com/covil/2010/04/minimal-django.html>`_ 就可以实现一个“Hello World！”程序：

::

    import os
    from django.conf.urls.defaults import patterns
    from django.http import HttpResponse
    filepath, extension = os.path.splitext(__file__)
    ROOT_URLCONF = os.path.basename(filepath)

    def hello(request):
        return HttpResponse('Hello World!')

    urlpatterns = patterns('', (r'^/$', hello))

关于 Django 我不喜欢的一点就是它有点儿被焊死的感觉。不要尝试着去改变它，否则你会碰壁的。我来解释一下这个说法：比如说我想要使用 SQLAlchemy 作为 ORM 然后 SQLAlchemy 就会把 Admin， Auth， Form 等等几乎所有的部分都给搞砸。所以你最好使用它附带的工具包。

快速教程： `Django at a glance <https://docs.djangoproject.com/en/1.3/intro/overview/>`_

它们都基于 Django： `Disqus <http://disqus.com/>`_ ， `EveryBlock <http://www.everyblock.com/>`_ ， `Guardian (newspaper) <http://www.guardian.co.uk/>`_ ， `Firefox add-ons (Mozilla) <https://addons.mozilla.org/en-US/firefox/>`_ 

Flask
---------------

这个灵巧的框架是由 Armin Ronacher 创造的。它的名字暗示了它的含义，它基本上就是一个微型的胶水框架。它把 Werkzeug 和 Jinja 粘合在了一起。所以它很容易被扩展。

Flask 也有许多的 `扩展 <http://flask.pocoo.org/extensions/>`_ 可以供你使用，Flask 也有一群忠诚的粉丝和不断增加的用户群。它有一份很完善的文档，甚至还有一份唾手可得的常见范例。Flask 很容易使用，你只需要 3(7) 行代码就可以写出来一个 Hello World。

::

    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def hello():
        return "Hello World!"

    if __name__ == "__main__":
        app.run()

我觉得这是它最大的优点也是缺点 － Flask 并不强制一个特定的 ORM。这会使得写扩展有点儿困难，你可能会想用某种形式的数据库层，但是用哪一个呢？Python 有非常多可以选择的。这个 Blog 是用 Flask 写的，部署在 Google App Engine 上。这很容易因为 Google Datastore 和其他的不太一样。所以有时候不强制一个 ORM 也是好事儿。

如果你想建一个新站的话 Flask 是个非常不错的选择。但我并不会向所有的初学者都推荐 Flask，我指的是那些不关心“为什么可以”，只关心它们“可不可以”的初学者。

快速教程： `Flask quickstart <http://flask.pocoo.org/docs/quickstart/>`_ 

它们都基于 Flask： `Dev news aggregator for Battlefield3 <http://bf3.immersedcode.org/>`_ ， `Media Queries <http://mediaqueri.es/>`_ ， `Learn buffet <http://www.learnbuffett.com/>`_ ， `Konstruktor (appengine) <http://www.konstruktor.ee/blog/python-web-framework-roundup/www.konstruktor.ee,>`_ 

Bottle
------------

这个框架相对来说比较新。它受到了 Sinatra 的影响。Bottle 才是名副其实的微框架 － 它只有大约 4500 行代码。并且我认为这是最真实的基于 Python 的微框架，它除了 Python 标准库以外没有任何其它的依赖，甚至它还有自己独特的一点儿模版语言。Bottle 还是为数不多的支持 Python 3 的框架之一。

Bottle 的文档很详细并且抓住了事物的实质。Hello World 例子很像 Flask，也使用了装饰器来定义路径。

::

    from bottle import route, run

    @route('/hello/:name')
    def hello(name):
        return '<h1>Hello %s!</h1>' % name.title()

    run(host='localhost', port=8080)

我知道 Bottle 内部有一座桥梁来沟通各个部分，因为它只有一个文件。但是很难找到你想要的东西，它的代码散布的到处都是，看起来一团糟。

对于非常小的项目或者是实验性的项目来说，Bottle 是一个不错的选择。但是对于一些大型的项目来说最好就不要使用它了，因为它的扩展并不多。

快速教程： `Bottle tutorial <http://bottlepy.org/docs/dev/tutorial.html>`_

它们都基于 Bottle： `Plush (monitoring) <http://pulsh.com/>`_ ， `Hobo (Blog enginee) <http://andrewnelder.github.com/hobo/>`_ 

web.py
----------

以前 web.py 还很流行的时候被用来写 reddit。它能很好的处理流量问题。如果你用 web.py 开发 web 应用的话，你会发现它并不会阻碍你。 `标准配置 <http://webpy.org/skeleton/0.3>`_ 很简单也很直观。web.py 在文件和文件夹的分类上面也做的非常棒。

::

    import web

    urls = ('/(.*)', 'hello')
    app = web.application(urls, globals())

    class hello:        
        def GET(self):
            return 'Hello, World!'
    if __name__ == "__main__":
        app.run()

很遗憾的是这个库最近已经成为了 rails 框架狂热者的受害者。它有可以帮你做几乎所有事情的自己的库 － 模版，表格，数据库。可能它们并不像其它库一样得到了良好的维护，但是还是有许多人在用它。

快速教程： `web.py tutorial <http://webpy.org/docs/0.3/tutorial>`_ 

它们基于 web.py： `Yandex (russian search engine) <http://www.yandex.ru/>`_ ， `Telephone directory (Switzerland) <http://www.local.ch/>`_ 

Tornado
------------

Tornado 不单单是个框架，还是个 web 服务器。它一开始是给 FriendFeed 开发的，后来在 2009 年的时候也给 Facebook 使用。它是为了解决实时服务而诞生的。为了做到这一点，Tornado 使用了异步非阻塞 IO。

Tornado 的文档非常技术性。它并不是为初学者准备的。这是一个 Hello World 程序：

::

    import tornado.ioloop
    import tornado.web

    class MainHandler(tornado.web.RequestHandler):
        def get(self):
            self.write("Hello, world")

    application = tornado.web.Application([
        (r"/", MainHandler),
    ])

    if __name__ == "__main__":
        application.listen(8888)
        tornado.ioloop.IOLoop.instance().start()

我从来没有实际使用过这个例子。貌似默认情况下 Tornado 会传递 WSGI 层，因为 WSGI 并不能处理异步请求。Tornado 确实性能非常强，但是当调用数据库的时候它会阻塞 IO。

快速教程： `Tornado overview <http://www.tornadoweb.org/documentation/overview.html>`_ 

它们基于 Tornado： `Too cool for me <http://toocoolfor.me/>`_ ， `FriendFeed <http://friendfeed.com/>`_ 

CherryPy
-------------

这是最古老的 Python 框架的一种。CherryPy 并没有得到广泛的应用，大家提到它第一反应是 web 服务器然后才是一个框架。在处理请求方面 CherryPy 也使用了队列来优化性能，但是它使用的是 `线程池 <http://en.wikipedia.org/wiki/Thread_pool_pattern>`_ 技术。

CherryPy 的文档实际上非常少，但是基本上都可以涵盖主要的方面。CherryPy 也可以支持 Python 3。我必须说，它的 Hello World 例子非常漂亮：

::

    import cherrypy
    class HelloWorld(object):
        def index(self):
        return "Hello World!"
        index.exposed = True

    cherrypy.quickstart(HelloWorld())

快速教程： `CherryPy introduction <http://docs.cherrypy.org/stable/intro/index.html>`_ 

它们基于 CherryPy： `YouGov <http://global.yougov.com/>`_ ， `Cuil search engine (ended 2010) <http://en.wikipedia.org/wiki/Cuil>`_ 

Brubeck
----------

这是一个新的 Python 框架。其并不使用 WSGI 而直接在语言级别用 Mongrel2 作为服务器使用，这个仅把请求处理交给 Python 程序，请求作为协同程序来处理。

模块方面 Brubeck 使用了 DictShield 库，which means that plugins for different database layers could be written on top of it. (求翻译)

Brubeck 的文档非常少，但是你看到源码以后，你会知道其实并没有多少东西。所以它还是一个非常年轻并且在不断发展的框架。Hello World 例子看起来也很漂亮。

::

    class DemoHandler(WebMessageHandler):
        def get(self):
            self.set_body('Hello world')
            return self.render()

    urls = [(r'^/', DemoHandler)]
    mongrel2_pair = ('ipc://127.0.0.1:9999', 'ipc://127.0.0.1:9998')

    app = Brubeck(mongrel2_pair=mongrel2_pair,
              handler_tuples=urls)
    app.run()

唯一需要注意的是当你使用 Brubeck 的时候你也需要看看 Mongrel2 服务器的相关知识。

快速教程： `Brubecks design <http://brubeck.io/design.html>`_ 

它们基于 Brubeck： `ListSurf <https://github.com/j2labs/listsurf>`_ 


中英文对照表
---------------

common patterns － 常见范例

micro glue framework － 微型的胶水框架

coroutine － 协同程序





















