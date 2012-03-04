.. _toctree-directive:

微框架 Django
============= 

原文地址： `<http://softwaremaniacs.org/blog/2011/01/07/django-micro-framework/en/>`_

翻译： `yudun1989 <http://www.douban.com/people/yudun1989/>`_

大家都知道Django不适合仅仅想要显示一些HTML而不需要其他特殊组件的小型项目。如果你的项目仅仅是几个函数能搞定的话，去启动一个Django Project实际上是不值得的。

对于我来说这个其实很显然，但是最近鉴于从同事那里听来的一些说法,发现不是很多人能够意识到这一点。因为每个人并不总是能想起这个问题,所以结果你懂的。。我将会来演示如何用最少代价的来创建一个这样的Django项目好让大家更能理解这一点。

精简(Simplification)
--------------------

1. 不要使用 `startproject <https://docs.djangoproject.com/en/dev/ref/django-admin/#startproject-projectname>`_ 命令。它会创建一个带着注释的配置文件的漂亮工程，但是如果这不是你第一次用Django的话，那你可以完全不用它。
2. manage.py只是一个对于django-admin.py的一个"helper"，所以也并不是必须的。
3. 不要期待项目成为一个app的容器，不要让你的的代码支持Django插件。app是Django最强大的一个结构特性之一，但是在小型任务中你需要舍弃这些来保证灵活性。

之后生成的代码会是这样

* settings.py

::
    
    DEBUG = True
    ROOT_URLCONF = 'urls'
    TEMPLATE_DIRS = ['.']

* urls.py

::
    
    from django.conf.urls.defaults import *
    import views
    urlpatterns = patterns('',
        (r'^$', views.index),
        (r'^test/(\d+)/$', views.test),
    )

* views.py    

::
    
    from django.shortcuts import render
    def index(request):
        return render(request, 'index.html', {})
    def test(request, id):
        return render(request, 'test.html', {'id': id})

这是一个已经带有全部功能的项目，你可以通过以下指令来运行它

::
    
    django-admin.py --settings=settings runserver

你得到的是一个 带有请求-应答通道和模板的Django系统。尽管我没有检查，但是其他一些库应该也可以工作。你丢失的只是ORM和其他所有依赖他的APP系统。但这毕竟可以用来应对一些小型工作了。

极端(Extreme)
-------------

我想要深入一下应用的想法直到看起来有点荒谬，我决定将所有的代码变成一个可以运行的模块，并且成功了。

::
    
    from django.conf import settings
    if not settings.configured:
        settings.configure(
            DEBUG = True,
            ROOT_URLCONF = 'web',
            TEMPLATE_DIRS = ['.'],
        )

    from django.conf.urls.defaults import patterns
    urlpatterns = patterns('',
        (r'^$', 'web.index'),
        (r'^test/(\d+)/$', 'web.test'),
    )

::
    
    #### Handlers
    
    from django.shortcuts import render
    def index(request):
        return render(request, 'index.html', {})
    def test(request, id):
        return render(request, 'test.html', {'id': id})


::
    
    #### Running
    
    if __name__ == '__main__':
        from django.core.management import execute_from_command_line
        execute_from_command_line()

你可以这样来运行它

::
    
    python web.py runserver

然而，将这些代码来放在一起并是很好。因为文件中的可读部分已经被分开到不同的段中。这种暗示意味着它们最终应该分开到不同的文件中去。另外，我不敢凭记忆将这些代码写出来因为有众多的琐碎: 要避免初始化不同的配置文件，要记住很多import等等。

但是它仍然非常有趣。


后记(Afterthought)
------------------

当我在工程初始化中挣扎时，我发现ROOT_URLCONF的设置是在请求处理中被调用的。这会将Django管道与一个定义了 urlpattens变量的模块绑定，我认为如果Django能够通过一个简单的list来动态的进行初始化配置的话将会更加灵活:

::
    
    settings.configure(
        DEBUG = True,
        urlconf = [
        (r'^$', index),
        ],
    )

也许你在匆忙中搭建一个环境时候也许会用到。比如在测试的时候。

