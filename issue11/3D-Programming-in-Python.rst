Python 3D编程 --- 第一部分
===========================

原文： `3D Programming in Python <http://greendalecs.wordpress.com/>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

在这一系列的帖子中，我将涉及Python 3D编程的基础知识。先来看看我们将使用(以及不使用)什么工具，以及为什么。

第一，我们将使用OpenGL。OpenGL(Open Graphics Library)是一个编写2D和3D应用程序的跨平台API。从本质上来说，它就是一组函数，你可以调用它们来告诉GPU在屏幕上绘制什么。因为我们想使用Python完成本教程的所有程序，因此将使用 `pyglet <http://www.pyglet.org/>`_ 库来调用所有要用到的OpenGL函数。虽然有其他可用的函数库，比如PyOpenGL，但我个人更喜欢pyglet，因为它本身的文档化(it's documentation)和全面的 `编程指南 <http://www.pyglet.org/doc/programming_guide/index.html>`_ 。

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

如果一切进展顺利，你应该能看到一个全黑的窗体。那么这里发生了什么呢？第1行导入pyglet包。第3行创建一个有效的OpenGL上下文的pyglet窗体。

5-7行，为刚创建的窗体覆盖on_draw()函数。这允许你定义每次窗体需要重绘时想要发生的事，这也是调用实际的OpenGL绘制函数的地方。目前仅是清空窗体，所以你看到是一块黑屏。

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

相比开始的那小段代码，这段代码改变了一些东西。首先，在第2行添加了：

::

    from pyglet.gl import *

这样就能访问各种OpenGL的东西而不需要每次都写pyglet.gl.我们想要使用的东西。具体到这个例子中，我们只需要写GL_COLOR_BUFFER_BIT而不是pyglet.gl.GL_COLOR_BUFFER_BIT，GL_POINTS而不是pyglet.gl.GL_POINTS。这样我们也能简便地调用OpenGL函数。pyglet的所有函数名和常量都是与C中对应的相一致的，所以任何在 `OpenGL文档 <http://www.opengl.org/sdk/docs/>`_ 定义的函数都可以通过相同的语法进行调用。

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

这里做了两件事。第一，在第10行以颜色缓冲为参数调用glClear函数。这是告诉OpenGL上下文清空颜色缓冲区中所有的信息。颜色缓冲区是什么？就是内存中用于记录屏幕每个像素该显示什么颜色的一块区域。因此清空它就能得到初始空白的调色板。当我们进而进行更高级的渲染之时，会有更多的缓冲区需要清空，但是目前，只需要清空颜色缓冲区就够了。

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

你应该注意到这段代码并没有改变多少东西。glBegin函数的参数现在是GL_LINES，以告诉OpenGL我们将绘制线而不是点。我们也另外添加了一个顶点。这样我们将得到两条线。当告诉OpenGL将要绘制GL_LINES，OpenGL就会等你定义两个顶点，并且一旦你定义好了，它就会在这两点之间画一线段。因此，本例中，我们将得到一根以点(50,50)和(75,100)为端点的线段，以及一根以(100,150)和(200,200)为端点的线段。

修改后的代码运行结果如下：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot1.png

另一种绘制线段的方式是使用GL_LINE_STRIP。如果使用这种方式，OpenGL会等你定义开始的两个顶点，然后在这两点之间画线。之后，OpenGL会在任意随后定义的顶点与之前一个顶点之间画线。因此，本例中，第一条线是从(50,50)画到(75,100)，第二条线从(75,100)到(100,150)，第三条线从(100,150)到(200,200)，看起来是这样的：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot2.png

最后，如果你想画一个闭合的环，可以使用GL_LINE_LOOP。其实它做的事情和GL_LINE_STRIP是一样的，除了会在最后定义的顶点与最开始的顶点之间画一条线，从而闭合这个环。在我们的例子中，结果如下所示：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot3.png

OK，既然我们已经掌握了画线，接下来就来画三角形。

::

    import pyglet
    from pyglet.gl import *

    win = pyglet.window.Window()

    @win.event
    def on_draw():

        # Clear buffers
        glClear(GL_COLOR_BUFFER_BIT)

        # Draw outlines only
        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)

        # Draw some stuff
        glBegin(GL_TRIANGLES)
        glVertex2i(50, 50)
        glVertex2i(75, 100)
        glVertex2i(200, 200)
        glEnd()

    pyglet.app.run()

再一次地，这段代码和我们前面使用的非常一致。然而你应该注意到我们添加了一行代码来调用glPolygonMode。我不会深入地解说传递给这个函数的第一个参数，GL_FRONT_AND_BACK，因为我可能会写一篇完整的单独的帖子来解释环绕的顺序(winding order)以及向前向后的方向(front and back faces)问题。然而，第二个参数，GL_LINE，却是非常直观的。它就是告诉OpenGL我们要画的所有东西都是轮廓。默认设置为GL_FILL，但这里不能改成这样，因为你不可能填充一条线。

调用参数为元件GL_TRIANGLES的glBegin，告诉OpenGL每3个顶点定义一个三角形。由于glPolygonMode的第二个参数设置为GL_LINE，这样我们每定义三个顶点，就会在屏幕上绘制一个三角形的轮廓。这段代码的结果如下图所示：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot4.png

接下来，我们看看三角形带(triangle strip)。

::

    import pyglet
    from pyglet.gl import *

    win = pyglet.window.Window()

    @win.event
    def on_draw():

        # Clear buffers
        glClear(GL_COLOR_BUFFER_BIT)

        # Draw outlines only
        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)

        # Draw some stuff
        glBegin(GL_TRIANGLE_STRIP)
        glVertex2i(50, 50)
        glVertex2i(75, 100)
        glVertex2i(200, 200)
        glVertex2i(50, 250)
        glEnd()

    pyglet.app.run()

三角形带的行为类似于线条。当我们调用glBegin(GL_TRIANGLE_STRIP)，OpenGL会使用最开始三个顶点画一个三角形。之后，每个接着定义的顶点都与之前两个顶点构成一个三角形。因此，本例中，第一个三角形是基于线条17，18，19的顶点画成的。第二个三角形基于线条18，19，20的顶点，得到如下图形：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot5.png

画三角形的最后一种方法是使用GL_TRIANGLE_FAN。这允许你围绕一个中心点绘制很多三角形。

::

    import pyglet
    from pyglet.gl import *

    win = pyglet.window.Window()

    @win.event
    def on_draw():

        # Clear buffers
        glClear(GL_COLOR_BUFFER_BIT)

        # Draw outlines only
        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)

        # Draw some stuff
        glBegin(GL_TRIANGLE_FAN)
        glVertex2i(200, 200)
        glVertex2i(200, 300)
        glVertex2i(250, 250)
        glVertex2i(300, 200)
        glVertex2i(250, 150)
        glVertex2i(200, 100)
        glEnd()

    pyglet.app.run()

我们定义的第一个顶点，(200, 200)，定义了三角扇形的原点。在使用接下来的两个顶点定义了第一个三角形之后，我们依次创建了一列顶点，并使用当前顶点，前一个顶点以及原始顶点定义三角形。因此本例中，第一个三角形基于线17, 18, 19的顶点，下一个三角形基于线17，19，20的顶点，再接下来的一个基于线17，20，21的顶点，最后一个则基于线17，21，22的顶点。结果如下图所示：

.. image:: http://greendalecs.files.wordpress.com/2012/04/shot6.png

还有另外三种元件类型我没涉及，主要是因为它们相对比较直观，如果你已掌握目前为止讲述的东西，那也应该能够理解如何使用它们，非常简单的。以下是我们已涉及的元件列表，包括三个我们还没涉及的：

- GL_POINTS

- GL_LINES

- GL_LINE_STRIP

- GL_LINE_LOOP

- GL_TRIANGLES

- GL_TRIANGLE_STRIP

- GL_TRIANGLE_FAN

- GL_QUADS

- GL_QUAD_STRIP

- GL_POLYGON

下一篇教程将涉及环绕顺序(winding order)以及怎样绘制大量的元件。
