部署一个 Turbogears 项目到 Dotcloud
===================================

大家好！

最近我了解到了一个云服务叫做 DotCloud . 它看起来不错，于是我想看看它对 Python 的支持情况如何. 我已经看了他们的文档，也看了他们的 Django 相关的文件，大体讲的是说他们支持 WSGI .而我想知道是在这它上面部署一项服务到底好不好用.于是我就开始了，注册了它然后部署了一个非常简单的 TurboGears 项目在它上面.结果看起来还不错.

来让我们继续一起部署一个简单快速的项目吧.

首先呢，你需要一个 DotCloud 账户。注册完后，你需要安装 dotcloud 的 CLI（http://docs.dotcloud.com/firststeps/install/）

现在我们既然已经没什么阻碍了，就开始创建 TurboGears 项目吧.

::
    
    $ virtualenv --no-site-packages tg2env
    $ cd tg2env/
    $ source bin/activate
    (tg2env)$ easy_install -i http://tg.gy/current tg.devtools
    (tg2env)$ paster quickstart -x -n -m example
    (tg2env)$ cd example
    (tg2env)$ python setup.py develop

上面的命令行是做什么的呢？

* 建立一个虚拟的 Python 环境
* 安装 TurboGears 和它的一些附件 
* 建立一个没有数据库的 TurboGears 项目和时期生效的支持，然后使用 mako 模板。

需要更多的信息，你只需要运行命令行 ``paster quickstart --help`` 或者访问  `Quickstarting A TurboGears2 Project <http://turbogears.org/2.1/docs/main/QuickStart.html>`_ 的文档。

我们需要 ``requirements.txt`` 这个文件，因为 DotCloud 使用它来安装我们项目的附件.接下来就创建一个 ``requirements.txt`` 然后添加下面的代码：

::
    
    # requirements.txt
    -i http://www.turbogears.org/2.1/downloads/current/index
    tg.devtools

我们在这朵云上建一个应用程序吧.使用 ``dotcloud create tgtest`` 命令行. 你也许很想知道”它是怎么工作的“. 但我们要知道 除非我们把一个在我们的项目中里做 ``wsgi.py`` 的文件中输入以下的代码，否则它是不可用的.

::

    # wsgi.py
    from paste.deploy import loadapp
    application = loadapp('config:/home/dotcloud/current/development.ini')


在我们的项目的地址中，我们需要一个 ``dotcloud.yml`` 文件，这样 DotCloud 就知道我们要要用什么了。

::
    
    # dotcloud.yml
    www:
      type: python

我们就剩一个东西没有做了。既然这个只是为了测试而建立的程序，我们就把配置文件叫做 ``development.ini`` 。 在这个文件中，有一个指令 ``debug = true`` ，把他改为 false，像这样 ``debug = false`` 。 现在我们就可以发布我们的项目了。

::
    
    dotcloud push tgtest

接下来就坐着静静的等待你的终端显示出以下的代码吧

::
    
    18:54:28 [www] Using /home/dotcloud/env/lib/python2.6/site-packages
    18:54:28 [www] Finished processing dependencies for example==0.1dev
    18:54:32 [www] Build completed successfully. Compiled image size is 17MB
    18:54:32 ---&gt; Initializing new services... (This may take a few minutes)
    18:54:32 [www.0] Initializing...
    18:54:42 [www.0] Service initialized
    18:54:44 ---&gt; All services have been initialized. Deploying code...
    18:54:44 [www.0] Deploying build revision rsync-1332269603326...
    18:54:50 [www.0] Running postinstall script...
    18:54:52 [www.0] Launching...
    18:54:53 [www.0] Waiting for the service to become responsive...
    18:54:54 [www.0] Re-routing traffic to the new build...
    18:54:55 [www.0] Successfully deployed build revision rsync-1332269603326
    18:54:55 ---&gt; Deploy finished

祝你好运！

部署完毕，你的应用可以通过下面的链接访问了～

www: http://tgtest-mengu.dotcloud.com/
