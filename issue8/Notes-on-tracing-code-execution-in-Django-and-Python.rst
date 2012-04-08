笔记：追踪Django/Python代码执行
================================

`trace <http://docs.python.org/library/trace.html>`_ 能引发Python解释器打印正在执行的代码行。我是通过@brandon_rhodes的 `推文 <http://twitter.com/#!/brandon_rhodes/status/81332578283552768>`_ 了解到trace的。

**追踪一个Python程序** 
::

    python -m trace -t myprogram.py myargs

**伴随Django开发服务器进行追踪**

从我的经验来看，trace无法和Django的auto-reloader一起工作，所以需要使用--noreload选项。
::

    python -m trace -t manage.py runserver --noreload

**更多控制的追踪**

`这篇文章 <http://www.dalkescientific.com/writings/diary/archive/2005/04/20/tracing_python_code.html>`_ 展示了怎么编写传递给 `sys.settrace <http://docs.python.org/library/sys.html#sys.settrace>`_ 的定制性函数。

**Django追踪工具** , ``django-trace``

基于 ``sys.settrace`` 以及Django的管理命令，我编写了一个Django命令行管理工具。 https://github.com/saltycrane/django-trace .

安装
::

    $ pip install -e git://github.com/saltycrane/django-trace.git#egg=django_trace


+ 添加 ``'django_trace'`` 到 ``setting.py`` 文件的 ``INSTALLED_APPS``

用法
::

    $ python manage.py trace --help 
    Usage: manage.py trace [options] [command]

    Use sys.settrace to trace code

    Options:
        -v VERBOSITY, --verbosity=VERBOSITY
                            Verbosity level; 0=minimal output, 1=normal output,
                            2=verbose output, 3=very verbose output
        --settings=SETTINGS   The Python path to a settings module, e.g.
                            "myproject.settings.main". If this isn't provided, the
                            DJANGO_SETTINGS_MODULE environment variable will be
                            used.
        --pythonpath=PYTHONPATH
                            A directory to add to the Python path, e.g.
                            "/home/djangoprojects/myproject".
        --traceback               Print traceback on exception
        --include-builtins    Include builtin functions (default=False)
        --include-stdlib      Include standard library modules (default=False)
        --module-only         Display module names only (not lines of code)
        --calls-only          Display function calls only (not lines of code)
        --good=GOOD           Comma separated list of exact module names to match
        --bad=BAD             Comma separated list of exact module names to exclude
                            (takes precedence over --good and --good-regex)
        --good-regex=GOOD_REGEX
                            Regular expression of module to match
        --bad-regex=BAD_REGEX
                            Regular expression of module to exclude (takes
                            precedence over --good and --good-regex)
        --good-preset=GOOD_PRESET
                            A key in the GOOD_PRESETS setting
        --bad-preset=BAD_PRESET
                            A key in the BAD_PRESETS setting
        --version             show program's version number and exit
        -h, --help            show this help message and exit

或者
::

    $ python manage.py trace runserver 
    01->django.core.management:128:     try:
    01->django.core.management:129:         app_name = get_commands()[name]
    02-->django.core.management:95:     if _commands is None:
    02-->django.core.management:114:     return _commands
    01->django.core.management:130:         if isinstance(app_name, BaseCommand):
    01->django.core.management:134:             klass = load_command_class(app_name, name)
    02-->django.core.management:69:     module = import_module('%s.management.commands.%s' % (app_name, name))
    03--->django.utils.importlib:26:     if name.startswith('.'):
    03--->django.utils.importlib:35:     __import__(name)
    04---->django.contrib.staticfiles.management.commands.runserver:1: from optparse import make_option
    04---->django.contrib.staticfiles.management.commands.runserver:3: from django.conf import settings
    04---->django.contrib.staticfiles.management.commands.runserver:4: from django.core.management.commands.runserver import BaseRunserverCommand
    05----->django.core.management.commands.runserver:1: from optparse import make_option
    05----->django.core.management.commands.runserver:2: import os
    05----->django.core.management.commands.runserver:3: import re
    05----->django.core.management.commands.runserver:4: import sys
    05----->django.core.management.commands.runserver:5: import socket
    05----->django.core.management.commands.runserver:7: from django.core.management.base import BaseCommand, CommandError
    05----->django.core.management.commands.runserver:8: from django.core.servers.basehttp import AdminMediaHandler, run, WSGIServerException, get_internal_wsgi_application
    06------>django.core.servers.basehttp:8: """
    06------>django.core.servers.basehttp:10: import os
    06------>django.core.servers.basehttp:11: import socket
    06------>django.core.servers.basehttp:12: import sys
    06------>django.core.servers.basehttp:13: import traceback
    ...

或者
::

    $ python manage.py trace --bad=django,SocketServer --calls-only runserver 
    01->wsgiref:23: """
    01->wsgiref.simple_server:11: """
    02-->BaseHTTPServer:18: """
    03--->BaseHTTPServer:102: class HTTPServer(SocketServer.TCPServer):
    03--->BaseHTTPServer:114: class BaseHTTPRequestHandler(SocketServer.StreamRequestHandler):
    02-->wsgiref.handlers:1: """Base classes for server/gateway implementations"""
    03--->wsgiref.util:1: """Miscellaneous WSGI-related Utilities"""
    04---->wsgiref.util:11: class FileWrapper:
    03--->wsgiref.headers:6: """
    04---->wsgiref.headers:42: class Headers:
    03--->wsgiref.handlers:43: class BaseHandler:
    03--->wsgiref.handlers:371: class SimpleHandler(BaseHandler):
    03--->wsgiref.handlers:412: class BaseCGIHandler(SimpleHandler):
    03--->wsgiref.handlers:453: class CGIHandler(BaseCGIHandler):
    02-->wsgiref.simple_server:26: class ServerHandler(SimpleHandler):
    02-->wsgiref.simple_server:42: class WSGIServer(HTTPServer):
    02-->wsgiref.simple_server:83: class WSGIRequestHandler(BaseHTTPRequestHandler):
    01->contextlib:53: def contextmanager(func):
    01->contextlib:53: def contextmanager(func):
    Validating models...
    
    0 errors found
    Django version 1.4, using settings 'myproj.settings'
    Development server is running at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.
    01->myproj.wsgi:15: """
    01->wsgiref.simple_server:48:     def server_bind(self):
    02-->BaseHTTPServer:106:     def server_bind(self):
    02-->wsgiref.simple_server:53:     def setup_environ(self):
    01->wsgiref.simple_server:53:     def setup_environ(self):
    01->wsgiref.simple_server:66:     def set_app(self,application)
