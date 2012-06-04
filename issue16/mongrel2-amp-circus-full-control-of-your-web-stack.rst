Mongrel2&Circul = web栈的完全控制
==================================
原文: `<http://blog.ziade.org/2012/05/31/mongrel2-amp-circus-full-control-of-your-web-stack/>`_

译者: `yudun1989 <http://www.douban.com/people/yudun1989/>`_

Gunicorn 的挫折
---------------
我很喜欢Gunicorn,但是当 `Circus <http://circus.readthedocs.org/en/latest/index.html>`_ 开始发展之后我开始逐渐对Gunicorn对于web workers控制方面的缺陷开始感到困惑。

基本上,Gunicorn与Circus在进程上的处理做的是相同的工作:部署进程并且管理它们的生命周期。

但是Circus对于进程的管理给我们更多的控制权，我们可以:

-对于每个进程CPU/内存的使用给与持续的反馈结果
-添加或者删除某个进程
-在一个活动的栈做一些基本的维护操作

而且，对于flapping control,或者0.4版本中将会出现的 `Web Console <http://circus.readthedocs.org/en/latest/circushttpd/>`_ ,让Circus的使用更加有吸引力。

我想拥有的是一个简单的Python进程控制，它接收HTTP请求并且返回响应。这样我可以将我的进程由Circus控制，和对其他的控制是一样的。

Gunicorn则不然，因为它使用prefork模型，它用这些进程来处理进程本身。

如果我启动一个单实例的Gunicorn服务,在它运行的时候，我不能添加更多的进程进去。

Gunicorn绑定的套接字隶属于它的主进程，你不能在它上面运行其他进程。

以上内容被编辑过:如Philips在评论中所说，Gunicorn可以让你用TTIN和TTOU信号来添加和删除进程,但是Circus可以有更多的控制。看: `<http://circus.readthedocs.org/en/latest/commands/#circus-ctl-commands>`_

ZeroMQ 套接字会有一些不同，因为你会在一个套接字上连接尽可能多的进程，然后进行自由的负载均衡。

我猜想有一个解决方案，那就是重写WSGI服务器，将HTTP请求阻断并且将它推到ZMQ套接字中，并由另外一个进程来接收它。

等等...有一样东西

Mongrel2
------------
`Mongrel2 <http://mongrel2.org/>`_  是我一直在等待的一个东西，因为它就是将HTTP请求来转发到ZMQ套接字中，之后我就可以连接任意数量的进程。

`m2wsgi <https://github.com/rfk/m2wsgi>`_  是一个WSGI处理器，使用它就可以得到ZMQ从Mongre2得到的请求，然后返回响应。

创建一个脚本来运行使用m2wsgi来运行的Python WSGI程序很简单:


::

    from m2wsgi.io.standard import WSGIHandler
    from mysuperapp import wsgiapp

    zmq_port =  "tcp://127.0.0.1:9999"
    handler = WSGIHandler(wsgiapp, zmq_port)
    handler.serve()

然后用Mongrel2向那个端口来发送一些东西，就像这样:

::

    wsgi_handler = Handler(send_spec='tcp://127.0.0.1:9999',
                        send_ident='be4ee7d-6a47-42dd-9acd-1707add81835',
                        recv_spec='tcp://127.0.0.1:9998',.
                        recv_ident='')
    routes={ '/': wsgi_handler }
    localhost = Host(name='localhost', routes=routes)
    localip = Host(name='127.0.0.1', routes=routes)
    main = Server(
        uuid='31bf6b07-a147-466c-87b5-961481b99201',
        access_log='/logs/access.log',
        error_log='/logs/error.log',
        chroot='/var/www/mongrel2/',
        pid_file='/run/mongrel2.pid',
        default_host='localhost',
        name='main',
        port=6767,
        hosts=[localhost, localip]
    )
    settings = {'zeromq.threads': 1}
    servers = [main]

(灵感来源于 `<http://server.dzone.com/articles/deploying-graphite-mongrel2>`_ )

最后，通过Circus来管理Mongrel和m2wsgi进程:

::

    [circus]
    check_delay = 5
    endpoint = tcp://127.0.0.1:5555
    pubsub_endpoint = tcp://127.0.0.1:5556
    stats_endpoint = tcp://127.0.0.1:5557
    [watcher:mongrel2]
    cmd = mongrel2 tests/config.sqlite 31bf6b07-a147-466c-87b5-961481b99201
    warmup_delay = 0
    numprocesses = 1
    working_dir = /Users/tarek/Dev/github.com/circus-wsgi
    stdout_stream.class = StdoutStream
    stderr_stream.class = StdoutStream

    [watcher:m2wsgi]
    cmd = bin/python server.py
    numprocesses = 2

然后,circusctl,circus-top和circushttpd会给我一个完全的管理，就像这样:


::

    $ circusctl list
    circusd-stats,m2wsgi,mongrel2

    $ circusctl list m2wsgi
    1

    $ circusctl incr m2wsgi
    2

    $ circusctl stats m2wsgi
    m2wsgi:
    1: 10936  python tarek 0 N/A N/A N/A N/A N/A
    2: 10946  python tarek 0 N/A N/A N/A N/A N/A

接下来呢
----------
我会将Mongrel2-Circus来管理的服务器栈与Gunicorn栈来进行性能的比较。

如果结果不错，我会写一个小的circus-wsgi集成包来让安装和配置这一切变的更加简单。

