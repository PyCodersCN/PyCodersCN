关于日期你需要知道的
=============================

原文： `<http://taaviburns.ca/what_you_need_to_know_about_datetimes/what_you_need_to_know_about_datetimes.pdf>`_

译者： `TheLoverZ <http://zhuang13.de>`_ 

译注：这篇文章实在是翻译的相当吃力，因为文中的东西平时几乎不怎么接触（我都是直接用UTC），所以如果有错误请不吝指正，谢谢！

时间真是一个复杂的东西
------------------------
- 相比三个星期以前，我学到了更多关于时间的知识。
- 到 PyCon 结束的时候我可能学到的会更多！

大纲
------------------
- “瞬间”（instant）指什么？
- 时间标准
- 时区
- Python 模块： ``time`` , ``calendar`` , 显式和隐式的时间（Naive vs Aware datetimes）, ``pytz``
- 互用性

瞬间
------------------
- “瞬间是指一个极小时间内的时刻，这个时刻所经历的过程是瞬时的。”
- http://en.wikipedia.org/wiki/Instant
- 比如说这个时间：2012-03-10 10:30:00 PST
- 上面的例子和这个是同一个时间：2012-03-10 13:30:00 EST
- 很有趣吧？我们来看看⋯

时间标准
--------------------
- “时间标准是测量时间用的规范：或者是时间流逝的速度，或者是具体时间点，或者都有。”
- http://en.wikipedia.org/wiki/Time_standard
- 比如：UT\ :sub:`1`\，TAI，UTC，POSIX，或者是别的

UT\ :sub:`1`\
----------------
- 完全按照地球的自转来测算。
- 一秒的长度取决于潮汐，天气，彗星，等等。
- 地球自转角度 = 2π(0.7790572732640 + 1.00273781191135448T\ :sub:`u`) 弧度
- http://en.wikipedia.org/wiki/Ut1

TAI
-----------
- 国际原子时（International Atomic Time）。
- 取遍布全球的200台原子钟的平均时间。
- 原子时的基本单位是原子时秒，定义为：在零磁场下，铯-133原子基态两个超精细能级间跃迁辐射9,192,631,770周所持续的时间。
- http://en.wikipedia.org/wiki/Caesium_standard
- `原子时（中文版） <http://zh.wikipedia.org/zh/%E5%8E%9F%E5%AD%90%E6%97%B6>`_
- 在确定原子时起点之后，由于地球自转速度不均匀，世界时与原子时之间的时差便逐年积累。

UTC
-----------
- 协调世界时，又称世界标准时间或世界协调时间（Coordinated Universal Time）。
- 以原子时秒长为基础，通常比 UT1 的秒短那么一点儿。
- 在有需要的情况下会在协调世界时内加上正或负闰秒。
- 2012-06-30 23:59:60 UTC 是真实存在的！
- 一种称为协调世界时的折衷时标于1972年面世。

（第10页图）

POSIX（UNIX）时间戳
--------------------------
- 从协调世界时1970年1月1日0时0分0秒起至现在的总秒数。
- “不包括”闰秒（实际上是有的）。
- 每天精确86400秒。
- 有闰秒时会在下一秒叠加。
- 闰年指：

::

    def _days_before_year(year):
        y = year - 1
        return y*365 + y//4 - y//100 + y//400

- Will not agree with history books on dates before October 1582, and is muddy into the early 1900s（这个貌似和历史以及他们那里的常识有关了，貌似是说20世纪的时候日历做了调整来弥补之前的误差之类的⋯）
- http://en.wikipedia.org/wiki/Gregorian_calendar
- `公历（中文版） <http://zh.wikipedia.org/wiki/%E5%85%AC%E5%8E%86>`_

闰秒和闰日
----------------
- 闰秒以“日居正中”作为正午的标准。
- 闰日以春分日作为标准。

时区
----------
- 过去的时候每个城镇都有自己的时钟，按照当地中午时间校准。
- 所以当时火车晚点很正常，列车上的人也不知道具体是几点！
- 世界上第一次引入时区的概念是在1847年12月1号，由大不列颠的岛屿上的铁路使用格林尼治标准时间。
- http://en.wikipedia.org/wiki/Time_zone
- `时区（中文版） <http://zh.wikipedia.org/wiki/%E6%97%B6%E5%8C%BA>`_

（第15页图）

GMT != UTC
-------------------
- GMT是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。
- 如果你不关心次秒级的精度，那你完全不必担心！
- 但还是有点混乱，因为英国夏天使用夏令时（British Summer Time）。
- UTC 的中午 **永远** 是12：00！
- 多种习惯：中午可以表示为12：00也可以表示为00：00！
- http://en.wikipedia.org/wiki/Gmt
- `格林尼治标准时间（中文版） <http://zh.wikipedia.org/wiki/%E6%A0%BC%E6%9E%97%E5%B0%BC%E6%B2%BB%E6%A8%99%E6%BA%96%E6%99%82%E9%96%93>`_

最好使用UTC
-----------------
- Armin Ronacher 说
- “永远使用 UTC 或者 UNIX 时间戳。”
- “不要使用偏移量感知日期时间。”

关于用户的输入输出
---------------------------
- 用 Armin Ronacher 的话来说就是：
- “如果你从用户那里得到了本地时间，马上把它转化为 UTC 时间。如果这个转换有歧义的话需要通知用户。”
- “转换以后整个世界都清静了（然后什么偏移量的都去死吧！）”
- From http://lucumr.pocoo.org/2011/7/15/eppur-si-muove/

Python 的一些时间模块
---------------------------
- time
- calendar
- datetime
- pytz (from pypi)

time
-----------
- ``libc`` 接口
- 考虑一下 ``thread`` 和 ``os.fork``
- 处理 POSIX 时间戳和 ``struct_time``
- 设置 ``os.environ["TZ"]`` 以后才有时区支持
- ``struct_time`` 是隐式的，但有一个 ``is_dst`` 的标志变量（flag）。
- 给出一个显式的DST（DST-aware）时区，它指明了 DST 有没有生效
- 有助于消除歧义，比如说 01:30
- 用 ``time.time()`` 来得到当前的 POSIX 时间戳。
- 用 ``time.gmtime(t)`` 来得到一个 ``struct_time`` 
- 如果 ``(t == None)`` 则是当前的时间，或者提供一个 POSIX 时间戳。

calendar
---------------
- 和 ``datetimes`` 没什么关联，除了⋯
- 用 ``calendar.timegm(tuple)`` 来把一个 UTC 的 ``struct_time`` 转化为 POSIX 时间戳。
- http://bugs.python.org/issue6280 提议把它移到 ``time`` 模块中但是被拒绝了。

datetime
-------------
- Python 对象，有 ``dates`` , ``times`` , ``intervals`` 和 ``timezones`` 接口。
- 考虑一下 ``threading`` 和 ``subprocess`` 。
- 两种形式：
- 隐式的，没有时区信息。
- 显式的，有时区信息。（注意这两者的区别！）
- 不要把它们搞混！
- 不幸的是，还是有很多麻烦的地方。

datetime - 要做的
---------------------
- 使用 ``pytz``
- 做一个 `时区信息数据库 <http://zh.wikipedia.org/wiki/%E6%97%B6%E5%8C%BA%E4%BF%A1%E6%81%AF%E6%95%B0%E6%8D%AE%E5%BA%93>`_
- **很有必要** 使用帮助函数（helper functions）来创建本地的显式的时间。
- 定期更新 ``pytz`` 来对应时区的改变（包括 DST 的改变）
- 显式的使用 UTC 来表示时间：

::

    >>> datetime(2011, 11, 6, 5, 30, tzinfo=pytz.UTC)
    datetime.datetime(2011, 11, 6, 5, 30, tzinfo=<UTC>)

- 使用 ``pytz.timezone().localize()`` 来得到给定时区的一个显式的时间：

::

    >>> helsinki = pytz.timezone('Europe/Helsinki')
    >>> helsinki.localize(datetime(2011, 11, 6, 5, 30))
    datetime.datetime(2011, 11, 6, 5, 30, tzinfo=<DstTzInfo 'Europe/Helsinki' EET+2:00:00 STD>)

- 如果你关心 DST 的话就在 ``.localize()`` 设置一下：

::

    >>> toronto = pytz.timezone('America/Toronto')
    >>> toronto.localize(
    ...   # Is this EDT or EST?
    ...   datetime(2011, 11, 6, 1, 30),
    ...   is_dst=None)
    pytz.tzinfo.AmbiguousTimeError: 2011-11-06 01:30:00

- 为当前时刻得到一个给定时区的显式时间，这儿有个不错的办法：

::

    >>> toronto = pytz.timezone('America/Toronto')
    >>> datetime.now(toronto)
    datetime.datetime(2012, 3, 5, 16, 40, 12, 967922, tzinfo=<DstTzInfo 'America/Toronto' EST-1 day, 19:00:00 STD>)
    >>> _.date()
    datetime.date(2012, 3, 5)

datetime - 不要做的
-------------------------------
- 使用 **看似** 简单实际上是 **错误** 的办法创建 非UTC 的显式时间：

::

    >>> toronto = pytz.timezone('America/Toronto')
    >>> datetime(2011, 6, 1, 0, 0, # summer = DST!
    ... tzinfo=toronto)
    datetime.datetime(2011, 6, 1, 0, 0, tzinfo=<DstTzInfo 'America/Toronto' EST-1 day, 19:00:00 STD>)
    >>> _.isoformat()
    '2011-06-01T00:00:00-05:00'

- 或者是：

::

    >>> datetime(2011, 11, 6, 5, 30,
    ...   tzinfo=helsinki)
    datetime.datetime(2011, 11, 6, 5, 30, tzinfo=<DstTzInfo 'Europe/Helsinki' HMT+1:40:00 STD>)

- 使用 ``.replace()`` 向一个隐式的时间添加时区：

::

    >>> datetime(2011, 11, 6, 5, 30).replace(tzinfo=helsinki)
    datetime.datetime(2011, 11, 6, 5, 30, tzinfo=<DstTzInfo 'Europe/Helsinki' HMT+1:40:00 STD>)

（32页图）

互用性（Interoperability）
------------------------------------
- 这个世界并不是由 Python 所统治的。
- MySQL
- PostgreSQL
- SQLite
- JavaScript

MySQL
---------------
- 日期：纯日期，没有时区。
- 时间：类似 Python 的显式的时间。
- 时间戳：在内部使用 POSIX 时间戳存储。有读取需求的时候按照需求给定。
- 如果需要改变时区的话使用这个：``CONVERT_TZ(dt,from_tz,to_tz)`` ，参见 `这里 <http://dev.mysql.com/doc/refman/5.1/en/date-and-time-functions.html#function_convert-tz>`_ 。
- 但是如果你不小心的话 DST 还是会让你碰钉子的。

PostgreSQL
--------------------
- 日期：纯日期，没有时区。
- 时间：“我们不建议使用带时区的时间。”
- 时间戳：在内部按照从 2000-01-01 00:00:00 UTC 到现在的秒数存储。有读取需求的时候按照需求给定。
- 如果需要改变时区的话使用这个： ``AT TIME ZONE`` 。
- `PostgreSQL 官方文档 <http://www.postgresql.org/docs/8.1/static/functions- datetime.html#FUNCTIONS-DATETIME- ZONECONVERT>`_

SQLite
----------------
- 文本型：“按照 ISO8601 标准。即YYYY-MM-DD HH:MM:SS.SSS”
- 实型：“按照从格林尼治标准时间的公元前4714年12月24号正午到现在的天数。”（这个好奇怪，我没有翻译错吧。）
- 整型：“和 UNIX 时间一样，取从 1970-01-01 00:00:00 UTC 到现在的秒数。”
- 内置的时间和日期函数很有限，但可以使用 ``unixepoch`` ， ``localtime`` ， ``utc`` 。
- `SQLite 官方文档 <http://www.sqlite.org/lang_datefunc.html>`_

JavaScript
-------------------
- 日期对象默认设置为本地时区和当前的 DST 。
- 如果你需要的话可以用这个 ``getUTC*``
- “本地时间是指 JS 被执行的这台电脑的时间。”
- 你也可以用 POSIX 时间戳：

::

    new Date(posixTimestamp * 1000);
    var posixTimestamp = Date.now()/1000;
    (new Date(posixTimestamp * 1000)).getTime() / 1000 == posixTimestamp

- 但是永远不要在不同的时区做任何事，或者跨越 DST 的界限。

我读的东西
-------------
- http://en.wikipedia.org/wiki/International_Atomic_Time
- https://www.eff.org/press/releases/eff-wins-protection-time-zone-database
- http://www.ucolick.org/~sla/leapsecs/amsci.html
- http://www.cl.cam.ac.uk/~mgk25/time/leap/
- http://www.bipm.org/en/si/si_brochure/chapter2/2-1/second.html
- http://www.iana.org/time-zones
- http://lucumr.pocoo.org/2011/7/15/eppur-si-muove/
- http://unix4lyfe.org/time/
- http://opensourcehacker.com/2008/06/30/relativity-of-time-shortcomings-in-python-datetime-and-workaround/
- http://www.mail-archive.com/leapsecs@rom.usno.navy.mil/msg00109.html
- http://pypi.python.org/pypi/pytz/
- http://labix.org/python-dateutil
- 等等。

附加主题
-------------
- 闰秒
- ``timegm()`` 的实现

闰秒
------------
::

    os.environ['TZ'] = 'right/UTC'
    time.tzset()

- ``mktime`` 这时候和 ``gmtime`` 正好相反。
- 当心：你的时间戳现在不是 POSIX 时间戳了，所以你得到的可能比正常早24秒（6月以后是25秒）

timegm 的实现
------------------------
- 如果你的 Python 版本在 2.7 以上，那么你只需要：

::

    _EPOCH_DATETIME = datetime(1970, 1, 1)
    _SECOND = timedelta(seconds=1)
    def timegm(tuple):
        return (datetime(*tuple[:6]) - _EPOCH_DATETIME) // _SECOND
