将Travis-CI用于Python和Django
===============================

原文：`Using Travis-CI With Python and Django <http://justcramer.com/2012/05/03/using-travis-ci/>`_ 

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

我使用 `Travis-CI <http://travis-ci.org/>`_ 已有一段时间了。我的个人项目，甚至在DISQUS我们维护的几个代码库都依赖于它进行持续集成。我想是时候表达我对Travis无法否认的喜爱之情，以及谈一些我们项目中的默认做法。

Travis-CI入门非常容易，包括在项目的根目录下放置一个 ``.travis.yml`` 文件，以及配置GitHub和Travis之间的钩子(hook)。如果你用的是组织类型的账户(译注：using organizations，应该是指github上的organization账户类型)，那么要将钩子配置好就不会总是这么简单了，所以对这一方面我不会讲太多。我想要分享的是我们是如何为Django和Python项目组织配置文件的。

一个基本的 ``.travis.yml`` 看起来大致是这样的：

::

    language: python
    python:
        - "2.6"
        - "2.7"

    install:
        - pip install -q -e . --use-mirrors
    script:
        - python setup.py test

由于我们的大部分项目都使用了Django，这就意味着需要测试几个Django的版本。Travis通过它的矩阵构建使得这事变得非常简单。在我们的案例中，需要设置一个DJANGO矩阵，并且确保已经安装它：

::

    env:
        - DJANGO=1.2.7
        - DJANGO=1.3.1
        - DJANGO=1.4
    install:
        - pip install -q Django==$DJANGO --use-mirrors
        - pip install -q -e . --use-mirrors

此外，我们通常会遵从pep8，并且总是想对我们的构建执行pyflakes。我们也使用一个定制版本的pyflakes以便筛选警告信息，因为它们绝非严重错误。使用 ``before_script`` 钩子(会在 ``script`` 中的测试运行之前运行)很容易加入这一功能。

::

    install:
        - pip install -q Django==$DJANGO --use-mirrors
        - pip install pep8 --use-mirrors
        - pip install https://github.com/dcramer/pyflakes/tarball/master
        - pip install -q -e . --use-mirrors
    before_script:
        - "pep8 --exclude=migrations --ignore=E501, E225, src"
        - pyflakes -x W src

说完做完所有这些，我们得到一个如下所示内容的配置文件：

::

    language: python
    python:
        - "2.6"
        - "2.7"
    env:
        - DJANGO=1.2.7
        - DJANGO=1.3.1
        - DJANGO=1.4
    install:
        - pip install -q Django==$DJANGO --use-mirrors
        - pip install pep8 --use-mirrors
        - pip install https://github.com/dcramer/pyflakes/tarball/master
        - pip install -q -e . --use-mirrors
    before_script:
        - "pep8 --exclude=migrations --ignore=E501,E225 src"
        - pyflakes -x W src
    script:
        - python setup.py test

Travis会自动针对每个Python版本和环境变量组合并行地执行构建，因此你能一次性地测试两者的所有组合。相当简单，对吧？
