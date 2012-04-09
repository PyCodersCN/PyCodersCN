用Python和Phantomjs创建书签服务
===================================

原文:  `<http://charlesleifer.com/blog/building-bookmarking-service-python-and-phantomjs>`_

译者:  Ruici Luo

使用Python和Phantomjs——一种headless webkit浏览器，可以快速搭建一个保存每一个页面截屏的自托管(self-hosted)书签服务。将该服务与一段简单的JavaScript代码构成的书签脚本结合在一起可以使你非常方便地存储书签。这篇帖子的目的就是介绍如何一步一步搭建并运行起一个简单的书签服务。

.. figure:: http://media.charlesleifer.com/images/photos/import_playground-182916.png
   :align: center
   :alt: import playground

安装Phantomjs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

首先我们需要安装Phantomjs并且确保它能够正确地工作。根据操作系统的不同，你可能需要不同的指令来完成安装。如果在此过程中遇到问题，请查阅 `Phantomjs文档 <http://code.google.com/p/phantomjs/wiki/Installation>`_ 。

根据系统结构选择合适的二进制安装包：

-  x86: `<http://phantomjs.googlecode.com/files/phantomjs-1.5.0-linux-x86-dynamic.tar.gz>`_
-  x86_64: `<http://phantomjs.googlecode.com/files/phantomjs-1.5.0-linux-x86_64-dynamic.tar.gz>`_

获取安装包并解压：

::

    mkdir ~/bin/
    cd ~/bin/
    wget http://phantomjs.googlecode.com/files/phantomjs-1.5.0-linux-x86-dynamic.tar.gz
    tar xzf phantomjs-1.5.0-linux-x86-dynamic.tar.gz

将可执行文件放入系统路径：

::

    sudo ln -s ~/bin/phantomjs/bin/phantomjs /usr/local/bin/phantomjs

安装依赖——fontconfig和freetype

::

    sudo pacman -S fontconfig freetype2

在终端下测试Phantomjs。你应该会看到如下输出：

::

    [charles@foo] $ phantomjs
    phantomjs>

设置Python环境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
我比较喜欢的一种使用 `flask框架 <http://flask.pocoo.org/>`_ 的方式是——整个应用包含在单一的Python模块中。为简单起见我使用 `peewee <http://peewee.readthedocs.org/en/latest/index.html>`_ 和sqlite做为数据持久层。我已经为flask和peewee编写了一些 `扩展 <https://github.com/coleifer/flask-peewee>`_ 并且包含了必要的依赖。以上这些就是我们所有需要安装的。

设置一个新的 `virtualenv <http://www.virtualenv.org/en/latest/index.html>`_  

::

    virtualenv --no-site-packages bookmarks
    cd bookmarks
    source bin/activate

(译者注：--no-site-packages选项在最新版本的virtualenv里已经是默认行为，可以省略)

安装flask-peewee:

::

    pip install flask-peewee

建立以下目录和空文件来保存我们的应用、屏幕截图和模版文件：

::

    bookmarks/ ----- *this is the root of our virtualenv
    bookmarks/app/
    bookmarks/app/app.py
    bookmarks/app/templates/
    bookmarks/app/templates/index.html

获取 `bootstrap <http://twitter.github.com/bootstrap/>`_ 做为静态资源:

::

    cd app/
    wget http://twitter.github.com/bootstrap/assets/bootstrap.zip
    unzip bootstrap.zip
    mv bootstrap/ static/

当你完成上述步骤后，你就应该拥有一个这样的目录：

.. figure:: http://media.charlesleifer.com/images/photos/file-layout.jpg
   :align: center
   
编写一些代码
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我期待的Python应用是相当地简单。它包含两个视图：一个将一组书签传给模版进行渲染，另一个负责添加新的书签。

以下是所有的代码：

::

    import datetime
    import hashlib
    import os
    import subprocess
    
    from flask import Flask, abort, redirect, render_template, request
    from flask_peewee.db import Database
    from flask_peewee.utils import object_list
    from peewee import *
    
    # app configuration
    APP_ROOT = os.path.dirname(os.path.realpath(__file__))
    MEDIA_ROOT = os.path.join(APP_ROOT, 'static')
    MEDIA_URL = '/static/'
    DATABASE = {
        'name': os.path.join(APP_ROOT, 'bookmarks.db'),
        'engine': 'peewee.SqliteDatabase',
    }
    PASSWORD = 'shh'
    PHANTOM = '/usr/local/bin/phantomjs'
    SCRIPT = os.path.join(APP_ROOT, 'screenshot.js')
    
    # create our flask app and a database wrapper
    app = Flask(__name__)
    app.config.from_object(__name__)
    db = Database(app)
    
    class Bookmark(db.Model):
        url = CharField()
        created_date = DateTimeField(default=datetime.datetime.now)
        image = CharField(default='')
    
        class Meta:
            ordering = (('created_date', 'desc'),)
    
        def fetch_image(self):
            url_hash = hashlib.md5(self.url).hexdigest()
            filename = 'bookmark-%s.png' % url_hash
    
            outfile = os.path.join(MEDIA_ROOT, filename)
            params = [PHANTOM, SCRIPT, self.url, outfile]
    
            exitcode = subprocess.call(params)
            if exitcode == 0:
                self.image = os.path.join(MEDIA_URL, filename)
    
    @app.route('/')
    def index():
        return object_list('index.html', Bookmark.select())
    
    @app.route('/add/')
    def add():
        password = request.args.get('password')
        if password != PASSWORD:
            abort(404)
    
        url = request.args.get('url')
        if url:
            bookmark = Bookmark(url=url)
            bookmark.fetch_image()
            bookmark.save()
            return redirect(url)
        abort(404)
    
    if __name__ == '__main__':
        # create the bookmark table if it does not exist
        Bookmark.create_table(True)
    
        # run the application
        app.run()

添加index模版
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

index模版经过index视图的渲染后会显示很优美的一组书签。我们利用了Bootstrap提供的一些非常不错的css样式来展现书签列表。flask-peewee的object_list helper提供了分页功能：

::

    <!doctype html>
    <html>
    <head>
      <title>Bookmarks</title>
      <link rel=stylesheet type=text/css href="{{ url_for('static', filename='css/bootstrap.min.css') }}" />
    </head>
    <body>
      <div class="container">
        <div class="row">
          <div class="page-header">
            <h1>Bookmarks</h1>
          </div>
          <ul class="thumbnails">
            {% for bookmark in object_list %}
              <li class="span6">
                <div class="thumbnail">
                  <a href="{{ bookmark.url }}" title="{{ bookmark.url }}">
                    <img style="width:450px;" src="{{ bookmark.image }}" />
                  </a>
                  <p><a href="{{ bookmark.url }}">{{ bookmark.url|urlize(25) }}</a></p>
                  <p>{{ bookmark.created_date.strftime("%m/%d/%Y %H:%M") }}</p>
                </div>
              </li>
            {% endfor %}
          </ul>
    
          <div class="pagination">
            {% if page > 1 %}<a href="./?page={{ page - 1 }}">Previous</a>{% endif %}
            {% if pagination.get_pages() > page %}<a href="./?page={{ page + 1 }}">Next</a>{% endif %}
          </div>
        </div>
      </div>
    </body>
    </html>

屏幕截图脚本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

最后一步是获取屏幕截图的脚本。这个脚本应该被命名为"screenshot.js"并且放在整个应用的根目录下(和app.py在同一目录下)。虽然宽度、高度等参数都是硬编码在代码中，但是它们可以很容易地通过命令行传递给脚本:

::

    var page = new WebPage(),
    address, outfile, width, height, clip_height;

    address = phantom.args[0];
    outfile = phantom.args[1];
    width = 1024;
    clip_height = height = 800;
    
    page.viewportSize = { width: width, height: height };
    page.clipRect = { width: width, height: clip_height };
    
    page.open(address, function (status) {
      if (status !== 'success') {
        phantom.exit(1);
      } else {
        page.render(outfile);
        phantom.exit();
      }
    });

测试
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

为了测试书签服务，我们可以启动应用：

::

    $ python app.py
    * Running on http://127.0.0.1:5000/

你可以访问这个URL，然后会看到一个没有任何书签的简单页面。让我们访问如下两个url来加入两个书签吧！

-   `<http://127.0.0.1:5000/add/?url=http://charlesleifer.com.com/&password=shh>`_
-   `<http://127.0.0.1:5000/add/?url=http://charlesleifer.com.com/&password=shh>`_

如果一切顺利，那么经过短暂的停顿（phantomjs截图），页面会被重定向至响应的请求url。这里的重定向是为了一会儿我们添加的”书签脚本“——当你收藏完某个页面后，浏览器会自动重定向到之前你访问的这个收藏页面。

回到应用，加入两个书签后的页面如下:

.. figure:: http://media.charlesleifer.com/images/photos/bookmarks.jpg
   :align: center

添加JavaScript书签脚本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

打开浏览器的书签管理器，建立一个名为"Bookmark Service"的新书签。该书签并不指向一个特定的URL，取而代之的是一段JavaScript脚本，它可以将浏览器当前访问的页面发送到我们的书签服务:

::

    javascript:location.href='http://127.0.0.1:5000/add/?password=shh&url='+location.href;

试试访问另外一个页面，然后点击上述书签脚本

书签服务的改进
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个简单的服务还有很多改进的余地！这里是一些想法：

-  支持多种clipping height，假如你需要获取完整的页面
-  在任务队列中生成图像
-  增加安全机制
-  增加书签删除功能
-  获取页面标题并将他们存入数据库(提示：使用书签脚本)
-  尝试使用 `ghost <https://github.com/jeanphix/Ghost.py>`_ 或者是一种基于pyqt的浏览器来替代Phantomjs
 
感谢您的阅读，我希望您对这篇帖子感到满意。如果有任何建议，请留下回复。
