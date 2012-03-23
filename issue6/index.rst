issue #6: 春分
==============

Hi Pythonistas,

正如我们承诺过的，我们已经建立了往期 PyCoder's Weekly 的存档页，你可以在我们的 `主页 <http://pycoders.com/>`_ 的 ``archives`` 中找到它。我们正在策划让邮件列表变的更方便一些，如果你有什么建议，请直接回复这封邮件告诉我们。同时不要忘记把 `我们 <https://twitter.com/#!/pycoders>`_ 分享给你的朋友～

Enjoy.

--
Mahdi and Mike 

新闻与开发动态
--------------

在计算机科学或编程教学中使用 Python
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MIT has an Introduction to Computer Science course where the fundamentals of programming and Computer Science are taught with python. 

Pyramid 网页框架 1.3 版本发布
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Version 1.3 of the Pyramid Web Framework dropped this week. Among other features, Pyramid 1.3 has support for Python 3 compatibility. Check out the site for the full list of feature additions and updates.

Django 之旅
^^^^^^^^^^^

I am sure we all remember when we were starting out learning whatever framework we use to build tools; scouring the internet for good reliable content. Look no further, Cory has been touching on a lot of beginner Django topics and he is just getting started. 

项目
----

discoball
^^^^^^^^^

Discoball is a tool to filter streams and colorize patterns. This was originally written in Ruby; and it was definitely improved by two commits. One removing all the ruby. Two rewriting it in python. This is pretty cool for adding color to commands you use everyday, like highlighting line numbers in stack traces. 

Airy: 网页应用框架
^^^^^^^^^^^^^^^^^^

This is a pretty cool web framework In Contrast to most currently available frameworks, Airy doesn't use the standard notion of HTTP requests and pages. Instead, it makes use of WebSockets (via Socket.io). Unfortunately no SQL support is planned to added, hopefully this will change. 

Django Twilio
^^^^^^^^^^^^^

Ever wanted to quickly add twilio support to your django app? Well look no further, Randall Degges has got you covered. 

Email Pie
^^^^^^^^^

This can be really useful, especially when creating email forms on the web(maybe even like a newsletter signup?). Email Pie is a JSON Api that allows you to easily validate email address. It checks things like email format, mx records and obvious misspellings. Nice work Bryan!

Django 1.4 最好的模板系统
^^^^^^^^^^^^^^^^^^^^^^^^^

This is very useful if you are starting a brand new Django 1.4 Project. Its a base template for Django 1.4, so basically you run django-admin startproject and specify this as its template and it sets up your django project based on the template. This is based off the work of Mozilla’s Playdoh and even if it doesn’t suit your needs entirely its a good place to start to fork and create your own project template.


pyprocessing - Python 图形开发的类进程环境
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

译注：类进程英文原文为 ``Processing-like`` ，没找到规范的译法，暂时这么翻译了。

This project provides a python package that creates an environment for graphics applications that closely resembles that of the Processing system. The pyprocessing backend is built upon OpenGL and Pyglet, which provide the actual graphics rendering.

讨论
----

Python 3 转移之痛，要考虑 Python 2/3 都兼容的感觉实在是太糟糕了
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Extremely cool and very informative article on the inner workings for python. A must read for the curious python developer. 

博文
----

Python 魔术方法指南
^^^^^^^^^^^^^^^^^^^

Extremely cool and very informative article on the inner workings for python. A must read for the curious python developer. 

Reddit 评级算法的工作原理
^^^^^^^^^^^^^^^^^^^^^^^^^

Very informative article on how the ranking system for Reddit works with sample implementations in our favourite development language. 

Ajax 与 Django
^^^^^^^^^^^^^^

Seeing that quite a few people have trouble with Ajax and Django, the guys from Bracket have decided to do a write up on how to get things done. Quite long, but good overview on how to get on your way. 

Python 内部研究：调用的工作原理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Eli Bendersky wrote this great article about how callables work in Python 3.3. If you are looking for a deep dive into some of Python’s internals this is definitely worth checking out.

在 DotCloud 部署 TurboGears 应用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here is a quick tutorial of how to get a Turbogears project working on the cloud hosting service DotCloud. If you are a Turbogears user this might be worth checking out.

用 ElementTree 在 Python 中解析 XML
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is another one by Eli Bendersky, this article gives you a really great breakdown of your options for parsing/processing XML with the modules included in Python 2.7 and then goes on to show in depth examples for accomplishing many common XML parsing/processing tasks with Python. 

自定义 Bash 中 Fabric 任务的自动补全功能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

One of the most useful tools available to a Python hacker doing things for the web is Fabric. This is a simple bash completion script that allows you to complete commands for tasks in your fabfiles.

