Python 3D编程 --- 第一部分
===========================

原文： `3D Programming in Python <http://greendalecs.wordpress.com/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

在这一系列的帖子中，我将涉及Python 3D编程的基础知识。先来看看我们将使用(以及不使用)什么工具，以及为什么。

第一，我们将使用OpenGL。OpenGL(Open Graphics Library)是一个编写2D和3D应用程序的跨平台API。从本质上来说，它就是一组函数，你可以调用它们来告诉GPU在屏幕上绘制什么。因为我们想使用Python完成本教程的所有程序，因此我们将使用 `pyglet <http://www.pyglet.org/>`_ 库来调用所有要用到的OpenGL函数。虽然有其他可用的函数库，比如PyOpenGL，但我个人更喜欢pyglet，因为它本身的文档化和全面的 `编程指南 <http://www.pyglet.org/doc/programming_guide/index.html>`_ 。

如果你看过关于OpenGL编程的任意文档，其他教程或者书籍，也许遇到过比如GLU，GLUT这样的库。但我们在这里不会用到任何这些支持的库，这样你能更好地理解OpenGL的核心是如何工作的。

需要提示一下的是，阅读这一系列教程之前，你应该在自己的机器上安装好pyglet，以及知道怎样执行python脚本。pyglet网站提供的安装指南在 `这里 <http://www.pyglet.org/doc/programming_guide/installation.html>`_

让我们开始吧。本篇教程中，我将带你学习绘制OpenGL元件的基础知识。但是首先得创建一个OpenGL上下文的pyglet窗体。

::
    
    import pyglet

    win = pyglet.window.Window()

    @win.event
    def on_draw():
        win.clear()

    pyglet.app.run()


