关于日期你需要知道的
=============================

时间真是一个复杂的东西
------------------------
- 相比三个星期以前，我学到了更多关于时间的知识。
- 到 PyCon 结束的时候我可能学到的会更多！

大纲
------------------
- “瞬间”（instant）指什么？
- 时间标准
- 时区
- Python 模块：time, calendar, Naive vs Aware datetimes, pytz
- 互用性

瞬间
------------------
- “瞬间是指一个极小时间内的时刻，这个时刻所经历的过程是瞬时的。”
- `http://en.wikipedia.org/wiki/Instant <http://en.wikipedia.org/wiki/Instant>`_ 

瞬间
-------------
- 比如说这个时间：2012-03-10 10:30:00 PST
- 上面的例子和这个例子是同一个时间：2012-03-10 13:30:00 EST
- 很有趣吧？我们来看看

时间标准
--------------------
- 时间标准是测量时间用的规范：或者是时间流逝的速度，或者是具体时间点，或者都有。
- `http://en.wikipedia.org/wiki/Time_standard <http://en.wikipedia.org/wiki/Time_standard>`_ 
- 比如：UT\ :sub:`1`\，TAI，UTC，POSIX，或者是别的

UT\ :sub:`1`\
----------------
- 完全按照地球的自转来定义。
- 一秒的长度取决于潮汐，天气，彗星，等等。
- 地球自转角度 = 2π(0.7790572732640 + 1.00273781191135448T\ :sub:`u`) 弧度
- `http://en.wikipedia.org/wiki/Ut1 <http://en.wikipedia.org/wiki/Ut1>`_ 

TAI
-----------
- 国际原子时（International Atomic Time）。
- 取遍布全球的200台原子钟的平均时间。
- 原子时的基本单位是原子时秒，定义为：在零磁场下，铯-133原子基态两个超精细能级间跃迁辐射9,192,631,770周所持续的时间。
- `http://en.wikipedia.org/wiki/Caesium_standard <http://en.wikipedia.org/wiki/Caesium_standard>`_
- `原子时（中文版） <http://zh.wikipedia.org/zh/%E5%8E%9F%E5%AD%90%E6%97%B6>`_
- 在确定原子时起点之后，由于地球自转速度不均匀，世界时与原子时之间的时差便逐年积累。

UTC
-----------
- 协调世界时，又称世界标准时间或世界协调时间（Coordinated Universal Time）。
- 以原子时秒长为基础，通常比 UT1 的秒短那么一点儿。
- 在有需要的情况下会在协调世界时内加上正或负闰秒
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
- ``def _days_before_year(year):``

		``y = year - 1``

		``return y*365 + y//4 - y//100 + y//400``

- Will not agree with history books on dates before October 1582, and is muddy into the early 1900s（这个貌似和历史有关了）
- `http://en.wikipedia.org/wiki/Gregorian_calendar <http://en.wikipedia.org/wiki/Gregorian_calendar>`_
- `公历（中文版） <http://zh.wikipedia.org/wiki/%E5%85%AC%E5%8E%86>`_

闰秒和闰日
----------------
- 闰秒以“日居正中”作为中午的标准。
- 闰日以春分日作为标准。

时区
----------
- 过去的时候每个城镇都有自己的时钟，按照当地中午时间校准。
- 所以当时火车晚点很正常，列车上的人也不知道具体是几点！
- 世界上第一次引入时区的概念是在1847年12月1号，由大不列颠的岛屿上的铁路使用格林尼治标准时间。
- `http://en.wikipedia.org/wiki/Time_zone <http://en.wikipedia.org/wiki/Time_zone>`_
- `时区（中文版） <http://zh.wikipedia.org/wiki/%E6%97%B6%E5%8C%BA>`_

（第15页图）

GMT != UTC
-------------------
- GMT是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。
- 如果你不关心次秒级的精度，那你完全不必担心！
- 但还是有点混乱，因为英国夏天使用夏令时（British Summer Time）。
- UTC 的中午 **永远** 是12：00！
- 多种习惯：中午可以表示为12：00也可以表示为00：00！
- `http://en.wikipedia.org/wiki/Gmt <http://en.wikipedia.org/wiki/Gmt>`_
- `格林尼治标准时间（中文版） <http://zh.wikipedia.org/wiki/%E6%A0%BC%E6%9E%97%E5%B0%BC%E6%B2%BB%E6%A8%99%E6%BA%96%E6%99%82%E9%96%93>`_

最好使用UTC
-----------------
- Armin Ronacher 说
- “永远使用 UTC 或者 UNIX 时间戳。”
- “不要使用偏移量感知日期时间。”

用户的输入输出
---------------------------
- 用 Armin Ronacher 的话来说就是：
- “如果你从用户那里得到了本地时间，马上把它转化为 UTC 时间。如果这个转换有歧义的话需要通知用户。”
- “Rebase for Formatting (then throw away that filthy offset aware datetime object)”（待翻译）
- From `http://lucumr.pocoo.org/2011/7/15/eppur-si- muove/ <http://lucumr.pocoo.org/2011/7/15/eppur-si- muove/>`_

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
- ``struct_time`` 很简单，但有一个 ``is_dst`` 的标志变量（flag）。
- 给出一个 DST-aware 时区，它指明了 DST 有没有生效
- 有助于消除歧义，比如说 01:30
- 用 ``time.time()`` 来得到当前的 POSIX 时间戳。
- 用 ``time.gmtime(t)`` 来得到一个 ``struct_time`` 
- (if t == None) the current instant, or（待翻译）
- the provided POSIX timestamp（待翻译）

calendar
---------------
- 和 ``datetimes`` 没什么关联，除了⋯
- 用 ``calendar.timegm(tuple)`` 来把一个 UTC 的 ``struct_time`` 转化为 POSIX 时间戳。
- `http://bugs.python.org/issue6280 <http://bugs.python.org/issue6280>`_ 提议把它移到 ``time`` 模块中但是被拒绝了。

datetime
-------------
- Python 对象，有 ``dates`` , ``times`` , ``intervals`` 和 ``timezones`` 接口。
- 考虑一下 ``threading`` 和 ``subprocess`` 。
- 两种形式：
- 简单一些的，没有时区信息。
- 稍微复杂一些的，有时区信息。
- 不要把它们搞混！
- 不幸的是，还是有很多麻烦的地方。

datetime - 要做的
---------------------
（待续）

