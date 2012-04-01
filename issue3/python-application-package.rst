Python应用程序包
==========================

原文地址： `<http://blog.ianbicking.org/2012/02/29/python-application-package/>`_

翻译： `CMGS <http://cmgs.me/>`_

我经常琢磨着怎样统一的部署Python的Web应用（也是准备Python `夏日峰会 <https://us.pycon.org/2012/community/WebDevSummit/>`_ 的一部分）。现在我有了想法。

我在一年前写了 `这篇 <http://blog.ianbicking.org/2011/03/31/python-webapp-package/>`_ 文章，并且在最近修订了一些 `建议 <https://github.com/ianb/pywebapp/blob/master/docs/spec.txt>`_ ，同时我也考虑了一些更加基础的东西：一种简单的方法部署服务端应用和打包代码。Web应用仅仅只是这种部署方案的一种用况而已。

现在，让我们称这种部署方案为“Python application package”。它拥有以下几点特征：

1. 必须有应用描述：告诉环境这是个嘛应用。（有些时候我们称这个为“配置”但是这种说法是广义的具有迷惑性的；我认为“描述”会更加清晰。）
2. 通过提供描述，你能建立起一个运行时环境跑应用代码并且从应用获得那一堆对象们。因此需要一种明确的途径去设置sys.path，并且指定那些被应用需求又没有直接绑定到应用中的库。
3. 环境信息能被应用得到（当然这种也可以称为“配置”，但还是不要这样做）。比如怎样指定环境，应用怎样连接数据库（host, username，等）。（编者按：丫的坑爹这句直接英文不就好了么！）
4. 同时需要一种途径从应用执行命令和获得其对象。运行时环境在应用描述中应该得到这些命令或者对象的名称，并且可以依据应用目的以明确方式使用。比如吧，WSGI的Web应用会吧环境映射为应用对象。一个Tornado应用有着简单的命令去运行它（通过运行时环境指定跑在哪个端口上）。

还有很多东西你可以从这4点中引申出来，并且在复杂的应用中你也能用到他们。你可能有些WSGI应用，也可能不是WSGI的应用去监听Web Sockets，或者来自Celery队列的一些东西，或者接受传入邮件等。几乎所有的情况下，我认为最基本的应用生命周期应该是这样：当应用部署完毕通过命令启动他们，有时候还会检测环境是不是对的，有时候也可以备份它的数据，或者干脆卸载它。

还有一些的是，所有的环境必须设置或者内嵌到应用之中。比如$TMPDIR应该指向一个地方放置应用的临时文件。或者，所有的应用有着自己的目录放置自己的日志文件（也许通过另外一个环境变量指定）。

细节？
------

为了更加具体，这里有我想象中的一个小型应用的描述；大概YAML应该是更好的文件格式：

::
    
    platform: python, wsgi
    require:
      os: posix
      python: <3
      rpm: m2crypto
      deb: python-m2crypto
      pip: requirements.txt
    python:
      paths: vendor/
    wsgi:
      app: myapp.wsgiapp:application

我想平台意味着一系列的技术。系统并不是真正需要特定的python；当需要创建类似于 `Silver Lining <http://cloudsilverlining.org/>`_ 的时候，我发现比较容易添加PHP支持（编译型语言可能不能这样容易移植，比如Go，这样会更容易扩展）。因此Python也是应用使用的特性中的一种。你能想象到很多其他模块化的功能，但是也会很容易导致非生产环境下的混乱。

应用对环境有一定的要求，比如说Python的版本和OS类型。应用可能还需要其他的库，可能这货还不好移植（M2Crypto就是一个例子）。现代化的包管理器能工作得很靠谱，所以依赖系统包管理器是我首先会尝试并且相信是最好的解决方法（我会把requirements.txt作为后备计划，但不会把它当做主要解决依赖的方案）。

我同时也认为应用程序主要依靠直接绑定在他们内部的依赖会是更可靠的选择（比如用vendor目录）。这种方案就是工具可能会参差不齐（译者表示这丫的是想表达版本混乱吧），但是我相信通过统一包格式会解决这个问题。这里有个 `例子 <https://gist.github.com/1368649>`_ 告诉你肿么设置一个虚拟环境来管理依赖库（你这个时候不需要virtualenv才能去用这些库），这样做的话你能从你的版本控制中检查结果。这样做会有些复杂，不过还算能工作（好吧，几乎能工作-bin/files都需要修正，玩死你丫的）。但至少，这算一个好的开始吧。

依赖
----

在环境这一方面来说我们需要一个很好的库。 `pywebapp <https://github.com/ianb/pywebapp/>`_ 有一些基础的特性，虽然这货还没完成。我想要一个库像这样的：

::
    
    from apppackage import AppPackage
    app = AppPackage('/var/apps/app1.2012.02.11')
    # Maybe a little Debian support directly:
    subprocess.call(['apt-get', 'install'] +
                    app.config['require']['deb'])
    # Or fall back of virtualenv/pip
    app.create_virtualenv('/var/app/venvs/app1.2012.02.11')
    app.install_pip_requirements()
    wsgi_app = app.load_object(app.config['wsgi']['app'])

你想象下通过这个建立起一个服务，或者设置一个持续集成服务器（app.run_command(app.config['unit_test'])），等等。

本地开发
--------

如果设计得当，我认为这种应用在部署上格式也可以用在本地开发上。当checkout代码之后能直接在“开发环境”中运行，就像在其他环境中运行一样。

使用zip文件或者tar文件作为包格式已经过时了，或者说至少不能带来更多有利的一面。我看到用这些存档格式唯一的理由是这货容易迁移；但是我们得着眼于未来，我们有很多方法来移动目录，不需要迎合旧的习惯。如果用一个脚本创建一个tar，FTP到另一台机器，然后解压搞定 - 这种格式不应该知道实际上你是怎么传递文件的。But let’s not worry about copying `WARs <http://en.wikipedia.org/wiki/WAR_file_format_%28Sun%29>`_ 。（译者按，最后一句，自己看wiki吧，这货就一坑啊，明显嘲讽Java的War格式）
