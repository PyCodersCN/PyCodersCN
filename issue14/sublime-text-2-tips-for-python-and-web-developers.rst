给用 Python 的 web 开发者的 Sublime Text 2 小贴士
=================================================

原文： `<http://opensourcehacker.com/2012/05/11/sublime-text-2-tips-for-python-and-web-developers/>`_

译者： `TheLover_Z <http://zhuang13.de>`_ 

Sublime Text 2 是一个很强大的编辑器，最近开始获得了很高的人气 － 这不是没有理由的。它是商业软件。Sublime Text 2 有很多支持 Python 的插件。虽然核心部分保持封闭但围绕着这个编辑器还是形成了 `活跃的插件生态系统 <http://www.sublimetext.com/forum/viewforum.php?f=6>`_ 。

*提示：你可以免费使用 Sublime 。它只是会提示你“请购买”而已。*

这是我使用 Sublime 这么久以来发现的小技巧。我是站在在 OS X 视角来写的，但在 Linux 和 Windows 平台这些技巧应该也适用。

我以前是 Eclipse 的死党。虽然 Sublime 并没有像 Eclipse 那么强大，但是我发现 Sublime 最近的几个版本让我用起来是相当的舒服。最大的原因就是 Eclipse 要求文件夹和文件必须引入到所谓的 “Eclipse space” 下面。

Sublime 用起来更舒适。当你需要使用不同的工具和工程的时候，Sublime 比 Eclipse 更直观易懂。

插件管理
-----------

`Sublime 插件包管理 <http://opensourcehacker.com/2012/04/12/jslint-integration-for-sublime-text-2/>`_ 。你按照这样的步骤就可以安装任何插件了。它可以自动帮你使用云端(或者 Github)的资源安装。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.04.05-AM.png
    :alt: Sublime 插件包管理

在插件包管理安装以后你可以按 CMD + SHIFT + P 来添加新的包。

从命令行打开文件
------------------

我在我的 `.rc` 文件添加了下面的命令，然后我就可以在终端直接打开文件了

::

    alias subl="'/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl'"
    alias nano="subl"
    export EDITOR="subl"

从命令行打开文件夹
--------------------

你也可以在 Sublime Text 打开文件夹。

就像这样：

::

    subl src

然后整个 src/ 文件夹就会被打开。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.03.02-AM.png
    :alt: 在 Sublime 中打开文件夹

*提示：一个文件夹 = 一个项目 = 一个窗口？我不确定是不是还有办法让一个窗口显示多个项目*

使用更聪明的 tab 配置
----------------------

永远不要使用硬 tab 字符。

点击 *View > Indentation > Convert Indentation to Spaces* 然后确定 *Indent using spaces* 是被勾选的。新版本的 Sublime 应该会根据文件类型记住这个设置。

注意 `Sublime 会试图自动检测所有打开的文件的 tab 设置 <http://www.sublimetext.com/docs/indentation>`_ ，所以处理外部文件的时候要小心。使用 git 的 pre-commit 来预防出现这种情况。

你也可以配置通过 Preferences 菜单的配置文件来 `实现 <http://www.sublimetext.com/docs/2/indentation.html>`_ ，但是尝试了好多次以后还是失败了。

使用特定的语法高亮
-------------------

如果你有一个文件你想要使用特定的高亮，比如说你想对 ZCML 文件使用 XML 的高亮方式。

打开文件，然后 *View > Syntax >Open all with current extension as… ->[你想要的语法选择]*

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.01.03-AM.png
    :alt: 语法高亮的选择

查看更多关于语法高亮的 `资料 <http://stackoverflow.com/a/8014142/315168>`_ 。

语法匹配检查
------------

`SublimeLinter <https://github.com/Kronuz/SublimeLinter/>`_ 会在你键入的时候在后台扫描你的文件以保证正确性。请查看 README 的 Configuration 部分。不过你可能需要安装额外的软件(比如 Node.js)来保证功能的完整性。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-1.48.41-AM.png
    :alt: 语法匹配

使用 CodeIntel 自动补全支持
-------------------------------

从插件包管理器安装 CodeIntel。

如果你正在处理 Python 项目，这个代码就可以：

::

    [codeintel]
    recipe = corneti.recipes.codeintel
    eggs = ${instance:eggs}
    extra-paths = ${omelette:location}

然后就会自动生成 ``.codeintel`` 文件。

CodeIntel 假定在你的项目根目录下面有 ``.codeintel`` 文件，键入：

::

    subl .

来打开文件夹。

现在 Sublime 应该支持自动补全了。比如说你键入

::

    from zope.interface import <--- 这时候就应该蹦出来自动补全选项了

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-1.57.41-AM1.png
    :alt: 自动补全

而且，ALT + 鼠标左键可以把你带到 ``import`` 或者函数的声明部分。

CodeIntel 也支持 PHP, Ruby, JS, 这里只列出一小部分。

快速到达任何地方
----------------

按下 CMD + P，键入文件名的一小部分，就可以提示你全路径。非常强大的一个特性。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.08.51-AM.png
    :alt: 快速到达任何地方

使用 CTRL + G 来到达任意行。

文件内上下文敏感搜索
----------------------

针对 JS, CSS, Python 等语言设计。按下 CMD + R。输入你需要查找的内容，就会自动跳转到声明部分。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.12.41-AM.png
    :alt: 文件内搜索1

或者 Python：

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.29.11-AM.png
    :alt: 文件内搜索2

HTML/XML 标签工具
-------------------

针对 XML 的语法，Sublime Text 提供了一些必杀技。

用插件包管理器安装 ``Tag`` 。然后你就可以：

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.13.31-AM.png
    :alt: 标签工具

选择文本然后按下 CMD + SHIFT + P 搜索标签。然后就会一目了然，你也可以配置你自己风格的偏好。

git
---------

你可以使用 kemayo 的 `git 插件 <https://github.com/kemayo/sublime-text-2-git/wiki>`_ ，也是从插件包管理器安装。

.. image:: http://opensourcehacker.com/wp-content/uploads/2012/05/Screen-shot-2012-05-11-at-2.18.06-AM.png
    :alt: git支持

还未解决的问题
----------------

- `怎么编辑注释和增加新行同时保持注释闭合 <http://www.sublimetext.com/forum/viewtopic.php?f=2&t=7137>`_

- `更好的 restructed/markdown 语法高亮支持 <http://www.sublimetext.com/forum/viewtopic.php?f=2&t=7153>`_ 

- 在 Sublime 窗口运行 Plone 或者单元测试，用鼠标点击可以跟踪到相关的错误行