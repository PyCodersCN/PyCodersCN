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

如果一切进展顺利，你应该能看到一个全黑的窗体。那么这里发生了什么呢？第1行导入pyglet包。第3行创建了一个有效的OpenGL上下文的pyglet窗体。

5-7行，为刚创建的窗体覆盖on_draw()函数。这允许你定义每次窗体需要重绘时你想要发生的事，这也是调用实际的OpenGL绘制函数的地方。目前仅是清空窗体，所以你看到是一块黑屏。

最后，第9行调用函数pyglet.app.run()，从而开始事件循环(需要一直检测以查看用户是否执行了某些操作，比如按键或者点击鼠标)。当你关闭这个窗体，run方法将返回，应用程序结束运行。

好了，让我们来画些东西吧，从画些点开始。

::

    import pyglet
    from pyglet.gl import *

    win = pyglet.window.Window()

    @win.event
    def on_draw():

        # Clear buffers
        glClear(GL_COLOR_BUFFER_BIT)

        # Draw some stuff
        glBegin(GL_POINTS)
        glVertex2i(50, 50)
        glVertex2i(75, 100)
        glVertex2i(100, 150)
        glEnd()

    pyglet.app.run()

相比开始的那小段代码，这段代码改变了一些东西。首先，在第2行我们添加了：

::

    from pyglet.gl import *

这样我们就能访问各种OpenGL的东西而不需要每次都写pyglet.gl.我们想要使用的东西。具体到这个例子中，我们只需要写GL_COLOR_BUFFER_BIT而不是pyglet.gl.GL_COLOR_BUFFER_BIT，GL_POINTS而不是pyglet.gl.GL_POINTS。这样我们也能简便地调用OpenGL函数。pyglet的所有函数名和常量都是与C中对应的相一致的，所以任何在 `OpenGL文档 <http://www.opengl.org/sdk/docs/>`_ 定义的函数都可以通过相同的语法进行调用。

我们也将on_draw()函数改变为：

::
    
    # Clear buffers
    glClear(GL_COLOR_BUFFER_BIT)

    # Draw some stuff
    glBegin(GL_POINTS)
    glVertex2i(50, 50)
    glVertex2i(75, 100)
    glVertex2i(100, 150)
    glEnd()

这里做了两件事。第一，在第10行以颜色缓冲为参数调用glClear函数。这是告诉OpenGL上下文清空颜色缓冲区中所有的信息。颜色缓冲区是什么？就是内存中用于记录屏幕每个像素该显示什么颜色的一块区域。因此清空它就能得到初始空白的调色板。当我们进而进行更高级的渲染之时，会有更多的缓冲区需要清空，但是目前，只需要情况颜色缓冲区就够了。

接下来，我们调用了一堆OpenGL函数来画一些点。首先，glBegin函数告诉OpenGL需要画哪种元件。本例中，我们将画点。接下来的3行就是画指定位置的点。来快速地看一下这个函数的语法，glVertex2i表示用两个整数定义一个顶点。如果要创建一个3D顶点，就需要调用glVertex3i并传给它三个整数。如果要使用浮点数创建这些顶点，则需要使用glVertex2f或者glVertex3f。最后一个函数，glEnd，告诉OpenGL绘点结束。

上述代码运行结果如下所示：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot.png

相当的酷？恩，没什么酷的。不过不要担心，好戏就要开始了。

看看如下代码：

::

    import pyglet
    from pyglet.gl import *

    win = pyglet.window.Window()

    @win.event
    def on_draw():

        # Clear buffers
        glClear(GL_COLOR_BUFFER_BIT)

        # Draw some stuff
        glBegin(GL_LINES)
        glVertex2i(50, 50)
        glVertex2i(75, 100)
        glVertex2i(100, 150)
        glVertex2i(200, 200)
        gl.End()

    pyglet.app.run()


