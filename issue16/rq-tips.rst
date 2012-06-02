RQ 小提示
==========

**给那些准备在 Django 中使用 RQ 的人一些提示。**

译者: `Zeray Rice <http://www.fanhe.org>`_

我最近在我的项目中把 RQ 作为异步任务管理器添加了进来，如果你还没有听说过 `RQ <http://python-rq.org/>`_ ，它是一个简单的 `Celery <http://celeryproject.org/>`_ 的替代品，用于处理异步任务。

热情模式 [1]_
--------------

如果你很了解 Celery 的话，你可能知道他有一个热情模式，能够让任务变成同步式的，这样会使测试与本地开发的过程变的更方便一些， RQ 并不支持这个特性，但是他能够很轻松的实现这样的功能。

::
    
    import redis
    import rq
    
    from django.conf import settings
    
    
    def enqueue(function, *args, **kwargs):
        opts = getattr(settings, 'RQ', {})
        eager = opts.get('eager', False)
            if eager:
            return function(*args, **kwargs)
    
        else:
            conn = redis.Redis(**opts)
            queue = rq.Queue('default', connection=conn)
            return queue.enqueue(function, *args, **kwargs)

在这段代码中，我们使用 Django 设置来选择是否启用“热情模式”，同时在这个设置中也包含了对 Redis 服务器的设置（域名，端口，数据库名等等），最简单的运行“热情模式”的 settings.py 是：

::
    
    RQ = {'eager': True}

现在当你需要使用 RQ 的 ``Queue.enqueue`` 方法时，请改用上面我们实现的 ``enqueue`` 函数，这个例子只是简单的使用了默认队列，你可以根据你的需要去改动他。

集成 Sentry
-----------

当 RQ 任务失败的时候，回溯栈将被记录，使用 `rq-dashboard <https://github.com/nvie/rq-dashboard>`_ 等工具我们可以方便的找到哪里出了问题，但是如果你使用 `Sentry <https://github.com/dcramer/sentry>`_ ，你就无法看到正确的错误信息。

通过一个简单的修饰器，你可以让 RQ 把错误报告提交给 Sentry:

::
    
    from functools import wraps
    
    from django.conf import settings
    from raven import Client
    
    
    def raven(function):
        @wraps(function)
        def ravenify(*args, **kwargs):
            try:
                function(*args, **kwargs)
            except Exception:
                if not settings.DEBUG:
                    client = Client(dsn=settings.SENTRY_DSN)
                    client.captureException()
                raise
        return ravenify

嗯，现在只要保证你的设置中有 ``SENTRY_DSN`` 字段，用 ``@raven`` 修饰你的任务，就能够使 Sentry 正常工作了。

.. [1] 译注：即惰性求值的反义词，详见维基百科 `及早求值 <http://zh.wikipedia.org/wiki/%E5%8F%8A%E6%97%A9%E6%B1%82%E5%80%BC>`_ 词条