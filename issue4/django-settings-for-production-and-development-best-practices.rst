Django 生产开发最佳设置
=============================


当你刚开始使用 Django 的时候，你会很诧异竟然没有一个标准的配置。然而，作为一个初学者，确实有一些简单好用的办法。

**问题在哪儿？** Django 把所有的项目设置都存在一个叫做 settings.py 的文件里。在你不需要为了适应某些特定环境而手动改变配置的时候（比如说生产开发环境），Django 的做法看起来还算不错。

有不同配置文件的必要性
----------------------------

开发使用的配置文件应该和生产的不一样。为什么？

- 你应该在一个独立的文件里保护敏感数据，比如说数据库的密码，api密钥（api keys），私钥。
- 生产时候的行为和开发的不一样：比如说在你开发一个新特性的时候你不会想着要发一封email。

这里我列出了在不同环境之间可能会有变化的东西：

- 数据库细节：数据库名字，用户名字和密码
- api密钥
- 加密数据所使用的私钥
- 调试开关变量（ ``DEBUG`` 或者 ``TEMPLATE_DEBUG`` ）
- 所需的不同的开发工具的路径
- 或者任何需要让生产环境与开发环境表现不同的时候

我们接下来要用三种方法来配置你的配置文件。

第一种：本地设置
----------------------------

这个方法有一个 ``settings.py`` 文件，一些公共设置和一个特殊环境使用的 ``local_settings`` 文件。

我们来看看：
::

    ### settings.py file
    ### settings that are not environment dependent
    try:
        from local_settings import *
    except ImportError:
        pass

::

    ### local_settings.py
    ### environment-specific settings
    ### example with a development environment
    DEBUG = True
    DATABASES = {
        'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django',
        'USER': 'django',
        'PASSWORD': '1234',
        'HOST': '',
        'PORT': '',
    }

- 优点：如果你只需要一个生产环境和开发环境的话这样很简单 —— ``local_settings.py`` 应该在源码管理的范围以外，你需要一个独立的文件来应对生产和开发环境。
- 缺点：这个方法限制了你对设置的权限，比如说修改 ``local_settings`` 的公共设置。但是这种方法在大多数简单的情况下都有用。

第二种：基于环境的设置
----------------------------------

在 `Ches Martin <http://chesmart.in/>`_ 这儿有个例子。这个方法依赖于一个指向正确的 Python 模块的环境变量。它有直接明了的优点，所以你可以直接命名你的特殊设置文件（在这个例子中 ``production.py`` 是生产环境的配置文件）：
::

    [myapp]$ls settings
    __init__.py
    defaults.py
    dev.py
    staging.py
    production.py

我们引用设置文件的时候会发生什么？设置文件在这个情况下是一个包（Python 的包是一个内含 ``__init__.py`` 的文件夹），所以当编译器加载包的时候会执行 ``__init__.py`` 文件。

默认情况下，我们来让它在开发环境下可以工作。
::

    ### __init.py__
    from dev import *

::

    ### default.py__
    ## sensible choices for default settings

::

    ### dev.py
    from defaults import *
    DEBUG = True
    ### other development-specific stuff

::

    ### production.py
    from defaults import *
    DEBUG = False
    ### other production-specific stuff

怎么在生产环境和部署环境（Staging Environment）使用这个配置呢？

技巧就在于设置模块可以被覆盖。所以在启动服务器之前我们需要覆盖环境变量。比如说如果我们用 Apache 作为 Django 服务器的话，把你的 Apache 配置修改成：
::

    SetEnv DJANGO_SETTINGS_MODULE myapp.settings.production

第三种：系统范围的设置
----------------------------------
::

    ### settings.py
    import os
    ENVIRONMENT_SETTING_FILE = '/etc/django.myproject.settings'
    ### this will load all environment file settings in here
    execfile(ENVIRONMENT_SETTING_FILE)
    ### all common settings
    ### ...

显而易见的，这种办法的一个问题就在于我们不能修改公共配置文件 ``settings.py`` 里面定义好的变量。

但是另一个方面，这个方法简化了配置文件的管理。分别创造一个开发所需的，一个部署所需的，以及生产所需的，做好保密措施，然后你就没事儿了。

结论
------------

还有很多办法来管理你的配置，所以你可以多折腾折腾。看看下面列出的资料会很有好处。如果你有高见，来留个言吧！

资料
-------------
- `Splitting up the settings file <https://code.djangoproject.com/wiki/SplitSettings>`_ ，非常专业的 Django 官方文档。
- `Django settings <https://docs.djangoproject.com/en/dev/topics/settings/>`_ ，又一篇来自于 Django 官方的文档。
- `Extending Django settings for the real world <http://tech.yipit.com/2011/11/02/django-settings-what-to-do-about-settings-py/>`_ ，来自于Yipit。
- `Django settings at Disqus <http://justcramer.com/2011/01/13/settings-in-django/>`_ ，一个更加复杂的配置。
- Stack Overflow:
    - `How to modularize Django settings.py? <http://stackoverflow.com/questions/2035733/how-to-modularize-django-settings-p>`_
    - `How do you configure Django for simple development and deployment? <http://stackoverflow.com/questions/88259/how-do-you-configure-django-for-simple-development-and-deployment>`_
- Django Snippets `Keep settings.py in version control safely <http://djangosnippets.org/snippets/94/>`_