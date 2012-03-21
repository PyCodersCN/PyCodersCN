Unicdoe之痛
===========

原文地址： `<http://nedbatchelder.com/text/unipain.html>`_

译者： `yudun1989 <http://www.douban.com/people/yudun1989/>`_

实用Unicode编程指南
-------------------

这是我在 Pycon2012 所做的演讲。你可以阅读本页的幻灯片和文字，或者直接在浏览器中打开 `演示 <http://nedbatchelder.com/text/unipain/unipain.html>`_ ，或者来看现场视频。

同时，点击文章的图片将会进入所在幻灯片的的对应位置，图片中使用了 Symbola 字体，但是如果想要显示一些特殊符号的话，则需先将该字体下载下来。

.. image:: http://nedbatchelder.com/text/unipain_000.png

大家好，我是Ned Batchelder.我已经有十年的Python编程经验，这以为着，很多很多的时候，我与其他程序员一样，犯过很多Unicode的编码错误。

.. image:: http://nedbatchelder.com/text/unipain_001.png

如果你和其他 Python 程序员一样，那你肯定也碰到过如下情况:你编写了一段很漂亮的代码，事情看起来很顺。然后某一天一个很奇怪的"方言字符"不知道从哪冒了出来，你的程序中就开始大量涌现 UnicodeErrors 。

你好像知道这种问题应该怎样解决，于是呢，就去在错误出现的地方添加了 encode 和 decode ，但是UnicodeError又开始出现在其他的地方。于是你又在另外一个地方添加了 decode 抑或 encode 。在你玩过一段"编码打地鼠"游戏之后，问题似乎被解决。

之后某一天，另一种"方言字符"又在另外一个地方出现了。然后你不得不又去来玩这种"打地鼠"直到问题解决掉。

.. image:: http://nedbatchelder.com/text/unipain_002.png

现在你的程序终于可以运行。但是你又烦恼又不适，这个问题花费了太多时间，你知道这样解决"正确"，于是开始憎恨自己。你对 Unicode 的主要了解就是你很讨厌它。

你不想去了解怪异的字符集，你只想要写一个你认为不是很糟糕的程序。

.. image:: http://nedbatchelder.com/text/unipain_003.png

你不必去玩打地鼠游戏. Unicode 会有些麻烦，但是它并不难。了解了相关知识并且加以练习，你也可以方便的优雅的解决相关问题。

接下来我会教给你five Facts of lie，然后给你一些专业建议来解决Unicode问题。下面的内容将会包含Unicode基本知识，如何在Python 2和Python 3中来实现。他们有一定差异，但是你使用的基本策略都是一样的。

世界&Unicode
------------

.. image:: http://nedbatchelder.com/text/unipain_004.png

我们从Unicode基本知识开始。

.. image:: http://nedbatchelder.com/text/unipain_005.png

事实之一:计算机中的一切均为bytes(字节)。硬盘中的文件为一系列的byte组成，网络中传输的只有byte。所有的信息，在你写的程序中进进出出的，均由byte组成。

孤立的byte是毫无意义的，所以我们来赋予他们含义。

.. image:: http://nedbatchelder.com/text/unipain_006.png

为了表示各种文字，我们有大约50年的时间都在用ASCII码。每一个byte被赋予95种符号的一种，所以，当我给你发送byte值为65的时候，你知道我想表达一个小写的A。

.. image:: http://nedbatchelder.com/text/unipain_007.png

ISO Latin 1,或者 8859-1对ASCII的96多种字符进行了扩展。这也许是你用一个byte可以做的最多的事情了。因为byte中没有容量可以存储更多的符号了。

.. image:: http://nedbatchelder.com/text/unipain_008.png

在windows中，增加了另外27种字符。这种叫做CP1252编码。

.. image:: http://nedbatchelder.com/text/unipain_009.png

事实之二是，世界上的字符远远比256个要多。一个简单的byte不能够表达世界范围内的字符。在你玩"编码打地鼠"的时候，你多么的希望世界上所有的人都说英语，但是事实并不是这样,人们需要更多的符号来交流。

事实一和二共同造成了计算机设备结构与世界人类需求的一个冲突。

.. image:: http://nedbatchelder.com/text/unipain_010.png

当时为了解决冲突尝试了多种途径。通过一个byte来与符号或者字符进行对应的编码，每一种解决途径都没有解决事实二中的实质问题。

当时有很多一个byte的编码，都没有能够解决问题。每一个都只能解决人类语言的一部分。但是他们不能解决所有的文字问题。

.. image:: http://nedbatchelder.com/text/unipain_011.png

人们开始创造两个byte的字符集，但是仍然像碎片一样，只能够服务于不同地域的一部分人。

当时产生了不同的标准，讽刺的是，他们都不足以满足所有的符号的需求。

.. image:: http://nedbatchelder.com/text/unipain_012.png

Unicode 就是为了解决之前的老的字符集问题。Unicode 分配整形,被成为代码点( UNICODE 的字符被成为代码点（ CODE POINTS ）用 U 后面加上 XXXX 来表现，其中， X 为16进制的字符)来表示字符。他有 110 万的代码点，其中有十一万被占用，所以她可以有很多很多的空间可供未来的增长使用。

Unicode 的目的是包含一切，它从 ASCII 开始，包含了数以千计的代码，包含这著名的----雪人??,包含了世界上所有的书写系统，而且一直在被扩充。比如，最新的更新中，就有一大堆没用的词汇。

.. image:: http://nedbatchelder.com/text/unipain_013.png

这里有六个的异国 Unicode 字符。 Unicode 代码点写成 4- , 5- ,或者 6 位的十六进制编码，同时有一个 U 的前缀。每一个字符都有一个用 ASCII 字符规定的名称。

.. image:: http://nedbatchelder.com/text/unipain_014.png

所以说 Unicode 提供了所有我们需要的字符的空间。但是我们仍然需要处理事实一中所碰到的问题：计算机只能看懂 bytes 。我们需要一种用 bytes 来表示 Unicode 的方法这样才可以存储和传播他们。

Unicode 标准定义了多种方法来用 bytes 来表示成代码点，被成为 encoding 。

.. image:: http://nedbatchelder.com/text/unipain_015.png

UTF-8 是最流行的一种对 Unicode 进行传播和存储的编码方式。它用不同的 bytes 来表示每一个代码点。ASCII 字符每个只需要用一个 byte ，与 ASCII 的编码是一样的。所以说 ASCII 是 UTF-8 的一个子集。

这里我们展现了几个怪异字符的 UTF8 的表示方法。 ASCII 字符 H 和 I 只用一个 byte 就可以表示。其他的根据代码点的不同使用了两个或者三个 bytes 。尽管有些并不常用，但是一些代码点使用到四个 bytes。

Python 2
--------

.. image:: http://nedbatchelder.com/text/unipain_016.png

好，说完了这么多理论知识，我们来讲一讲 Python 2

.. image:: http://nedbatchelder.com/text/unipain_017.png

在 Python2 中，有两种字符串数据类型。一种纯旧式的文字: "str" 对象,存储 bytes 。如果你使用一个 "u" 前缀，那么你会有一个 "unicode" 对象，存储的是 code points 。在一个 unicode 字符串中，你可以使用反斜杠 u(\u) 来插入任何的 unicode 代码点。

你可以注意到 "string" 这个词是有问题的。不管是 "str" 还是 "unicode" 都是一种 "string" ，这会吸引叫他们都是 string ，但是为了直接还是将他们明确区分来。

.. image:: http://nedbatchelder.com/text/unipain_018.png

如果想要在 unicode 和 bytes 间转换的话，每个都会有个方法。 Unicode 字符串会有一个 ``.encode`` 方法来产生 bytes , bytes 串会有一个 ``.decode`` 方法来产生 unicode 。每个方法中都会有一个参数，来表明你要操作的编码类型。

我们可以定义一个 Unicode 字符串叫做 my_unicode ，然后看这九个字符，我们使用 encode 方法来创建 my_unicode 的 bytes 串。会有 19 个 bytes ，想你所期待的那样。将 bytes 串来 decode 将会得到 utf-8 串。

.. image:: http://nedbatchelder.com/text/unipain_019.png

不幸的是，如果指明的编码名称错误的话，那么 encode 和 decode 会产生错误。现在我们去尝试 encode 我们的几个诡异的字符到 ascii ，会失败。因为 ascii 只能表示 0-127个 字符中的一个。然而我们的 Unicode 字符串早已经超出了范围。

抛出的异常为 UnicodeEncodeError ，它展现了你使用的编码方式， "codec" 即编码，解码器，展现了导致问题的字符的位置。

.. image:: http://nedbatchelder.com/text/unipain_020.png

解码同样会知道出一些问题。现在我们去把一个 UTF-8 字符串解码成 ASCII ,会得到一个 UnicodeDecodeError ，原因一样， ASCII 只接受 127 内的值，我们的 UTF-8字 符串超出了范围。

尽管 UTF-8 不能解码成任何的 bytes 串，我们尝试来 decode 一些垃圾信息。同样也产生了 UnicodeDecodeError 错误。最终， UTF-8 的优势是，有效的 bytes 串，将会帮助我们来创建高鲁棒性的系统:如果数据无效的话，数据不会被接受。

.. image:: http://nedbatchelder.com/text/unipain_021.png

当编码或者解码的时候，你可以指明如果 codec 不能够处理数据的时候，会发生什么情况。 encode 或者 decode 时候的第二个参数指明了规则。默认的值是 "strict" ，意味着像刚刚一样，将会抛出一个异常。

"replace" 值意味着，将会返回一个标准的替代字符。当编码的时候，替代值是一个问好，所以任何不能被编码的值将会产生一个 "?"。

一些其他的 handler 非常有用。"xmlcharrefreplace" 将会产生一个完全替代的 HTML/XML 字符，所以 \u01B4 将会变成 "&#436" (因为十六进制的 01B4 是十进制的 436 )。如果你需要将返回的值来输出到 html 文件中的话，将会非常有用。

注意要根据不同的错误原因使用不同的错误处理方式。"Replace" 是一个处理不能被解析的数据的自卫型方式，会丢失数据。"Xmlcharrefreplace" 会保护所有的原始数据，在XML转义符可以使用的时候来输出数据。

.. image:: http://nedbatchelder.com/text/unipain_022.png

你也可以指定在解码的时候的错误处理方式。"Ignore" 将会直接将不能解码的 bytes 丢掉。"Replace" 将会直接添加 Unicode U+FFFD ,给有问题的 bytes 来直接替换成"替换字符"。注意因为解码器不能解码这些数据。它并不知道到底有多少 Unicode 字符。解码我们的 UTF-8 字符串成为 ASCII 制造出了 16 个"替换字符"。每个 byte 不能被解析都被替换掉了。然而这些 bytes 只想要表示 6 个 Unicode 字符。

.. image:: http://nedbatchelder.com/text/unipain_023.png

Python 2 已经试图在处理 unicode 和 byte 串的时候变得有用些。如果你系那个要把 Unicode 字符串串和 byte 字符串来组合起来的话, Python 2 将会自动的将 byte 串来解码成 unicode 字符串。从而产生一个新的 Unicode 字符串。

比如，我们想要连接 Unicode 串 "hello" 和一个 byte 字符串 "world"。结果是一个 Unicode 的 "hello world"。在我们看来。Python 2 将 "world" 使用 ASCII codec 进行了解码。这次在解码中使用的字符集的值与 sys.getdefaultencoding() 的值相等。

这里这个系统中的字符集为 ASCII, 因为这是唯一的一种猜测: ASCII 如此被广泛接受。它是这么多编码的子集。不像是错误的。

.. image:: http://nedbatchelder.com/text/unipain_023.png

当然，这些隐藏的编码转换不能免疫于去解码错误。如果你想要连接 一个 byte 字符串和一个 unicode 字符串，并且 byte 字符串不能被解码成 ASCII 的话，将会抛出一个 UnicodeDecodeError。

这就是那些可恶的 UnicodeError 的圆圈。你的代码中包含了 unicode 和 byte 字符串，只要数据全部是 ASCII 的话，所有的转换都是正确的，一旦一个非 ASCII 字符偷偷进入你的程序，那么默认的解码将会失效，从而造成 UnicodeDecodeError 的错误。

.. image:: http://nedbatchelder.com/text/unipain_024.png

Python 2 的哲学就是 Unicode 字符串和 byte 字符串都是混乱的，他试图去通过自动转换来减轻你的负担。就像在 int 和 float 之间的转换一样， int 到 float 的转换不会失败，byte 字符串到 unicode 字符串会失败。

Python 2 悄悄掩盖掉了 byte 到 unicode 的转换，让程序在处理 ASCII 的时候更加简单。你复出的代价就是在处理非 ASCII 的时候将会失败。

.. image:: http://nedbatchelder.com/text/unipain_024.png

有很多方法来合并两个字符串，所有的都会解析 byte 成为 unicode，所以他们处理的时候你必须多加小心。

首先我们使用 ASCII 格式字符串，和 unicode 来结合。那么最终的输出将会变成 unicode。返回一个 unicode 字符串。

之后我们将两个交换一下：一个 unicode 格式的字符串和一个 byte 串再一次合并，生成了一个 unicode 字符串，因为 byte 串可以被解码成 ASCII。

简单的去打印出一个 unicode 字符串将会调用隐式的编码:输出总会是 bytes, 所以在 unicode 被打印之前必须被编码成 byte 串。

接下来的事情非常不可理解:我们让一个 byte 串编码成 UTF-8，却得到一个错误说不能被解码成 ASCII！这里的问题是 byte 串不能被编码，要记住编码是你将 Unicode 变成了 byte 串。所以想要执行你的操作的话，Python2 需要的是一个 unicode 字符串，隐式的将你的字符串解码成 ASCII。

最后，我们将 ASCII 字符串编码成 UTF-8。现在我们进行相同的隐式编码操作，因为字符串为 ASCII，编码成功。并且将它编码成了 UTF-8 ,打印出了原始的 byte 字符串，因为 ASCII 是 UTF-8 的一个子集。

.. image:: http://nedbatchelder.com/text/unipain_025.png

最重要的事实之三: byte 和 unicode 都非常重要，你必须将两个都处理好。你不能假设所有的字符串都是 byte ,或者所有的字符串都是 unicode ,你必须适当的运用他们必要时转换它们。

Python 3
--------
.. image:: http://nedbatchelder.com/text/unipain_026.png

我们看到了 Python 2 版本中有关 Unicode 之痛。现在我们看一下 Python 3，在 Python 2 到 Python 3 中最重要的变化就是他们对 Unicode 的处理。

.. image:: http://nedbatchelder.com/text/unipain_026.png

跟 Python 2 类似，Python 3 也有两种类型，一个是 Unicode,一个是 byte 码。但是他们有不同的命名。

现在你从普通文本转换成 "str" 类型后存储的是一个 unicode, "bytes" 类型存储的是 byte 串。你也可以通过一个 b 前缀来制造 byte 串。

所以在 Python 2 中的 "str" 现在叫做 "bytes"，而 Python 2 中的 "unicode" 现在叫做 "str"。这比起Python 2中更容易理解，因为 Unicode 是你总想要存储的内容。而 bytes 字符串只有你在想要处理 byte 的时候得到。

.. image:: http://nedbatchelder.com/text/unipain_027.png

Python 3 中对 Unicode 支持的最大变化就是将会没有对 byte 字符串的自动解码。如果你想要用一个 byte 字符串和一个 unicode 相链接的话，你将会得到一个错误，不管你包含的内容是什么。

所有这些在 Python 2 中都将会有隐式的处理，而在 Python 3 中你将会得到一个错误。

另外如果一个 Unicode 字符串和 byte 字符串中包含的是相同的 ASCII 码，Python 2 中将认为两个是相等的，而在 Python 3 中不会。这样做的结果是 Unicode 中的键不能找到 byte 字符串中的值，反之亦然，然而在 Python 2 中是可行的。

.. image:: http://nedbatchelder.com/text/unipain_028.png

这样彻底了改变了 Python 3 中的 Unicode 痛楚之源。在 Python 2 中，只要你使用 ASCII 数据，那么混合 Unicode 和 byte 将会成功，而在 Python 3 会直接忽略数据而失败。

这样的话在 Python 2 中所遇到的，你认为你的程序是正确的，但是最后发现由于一些特殊字符而失败的错误就会避免。

Python 3 中，你的程序马上就会产生错误，所以即使你处理的是 ASCII 码，那么你也必须处理 bytes 和 Unicode 之间的关系。

Python 3 中对于 bytes 和 unicode 的处理非常严格，你被迫去处理这些事情。这曾经引起争议。

.. image:: http://nedbatchelder.com/text/unipain_029.png

这样处理的原因之一是对读取文件的变化，Python 对于读取文件有两种方式，一种是二进制，一种是文本。在 Python 2 中，它只会影响到行尾符号，甚至在 Unix 系统上的时候，基本没有区别。

在 Python 3中。这两种模式将会返回不同的结果。当你用文本模式打开一个文件时不管是你是用的 "r" 模式或者是它默认的模式，读取成的文件将会自动转码成 unicode ,你将会得到 str 对象。

如果你用二进制模式打开一个文，在参数中输入 "rb" ，那么从文件中读取的数据将会是 bytes ,对它们没有任何处理。

隐式的对 bytes 到 unicode 的处理使用的是 locale.getpreferedencoding() ，然而它有可能输出你不想要的结果。比如，当你读取 hi_utf8.txt 时，他被解码成语言偏好中所设置的编码方式，如果我们这些例子在 windows 中创建的话，那么就是 "cp1252" 。像 ISO 8859-1, CP-1252 这些可以得到任意的 byte 值，所以不会抛出 UnicodeDecodeError ,当然也意味着他们会直接将数据解码成 CP-1252，制造出我们并不需要的垃圾信息。

为了文件读取正确的话，你应该指明想要的编码。open 函数现在已经可以通过参数来指明编码。


减轻痛苦
--------

.. image:: http://nedbatchelder.com/text/unipain_030.png

好，那么如何来减少这些痛苦？好消息是减轻痛苦的规则非常简单，在Python 2和 Python 3中都比较适用。

.. image:: http://nedbatchelder.com/text/unipain_031.png

正如我们在事实一中所看到的，在你的程序中进进出出的只有 bytes, 但是在你的程序中你不必处理所有的 bytes。最好的策略是将输入的 bytes 马上解码成 unicode。你在程序中均使用 unicode ,当在进行输出的时候，尽早将之编码成 bytes 。

制造一个 Unicode 三明治， bytes 在外， Unicode 在内。

要记者，有时候一些库将会帮助你完成类似的事情。一些库可能让你输入 unicode, 输出 unicode .它会帮你完成转换的功能。比如 Django 在它的 json 模块中提供 Unicode。

.. image:: http://nedbatchelder.com/text/unipain_032.png

第二条规则是。你需要知道你现在处理的是哪种类型的数据，在你的程序中任何一个位置，你需要知道你处理的是 byte 串还是一个 unicode 串。它不能是一种猜测，而应该被设计好。

另外，如果你有一个 byte 串的话，如果你想对他进行处理。那么你应该知道他是怎样的编码。

在对你的代码进行 debug 的时候，不能仅仅将之打印出来来看它的类型。你应该查看它的 type ,或者查看它 repr 之后的值来查看你的数据到底是什么类型。

.. image:: http://nedbatchelder.com/text/unipain_033.png

我曾经说过，你应该了解你的 byte 字符串的编码类型。好这里要我讲事实四:你不能通过检查它来判断这个字符串编码的类型。你应该通过其他途径来了解。比如很多协议中将会指明编码类型。这里我们给出 HTTP, HTML, XML, Python 源文件中的例子。你也可以通过预先的指定来了解编码。比如数据源码中可能会指明编码。

有一些方式可以来猜测一些 bytes 的编码类型。但是仅仅是猜测。能够确定的唯一方式是通过其他方式。

.. image:: http://nedbatchelder.com/text/unipain_034.png

这里是给出一些怪异的字符的编码猜测。我们用UTF-8 便民店的一些字符，被不同的解码方式解码之后的输出。你可以看见。有时候用不正确的解码方式解码可能会输出正确，但是会输出错误的字符。你的程序不能告诉你这些解析错误了。只有当用户察觉到的时候你才会发现错误。

这是事实四的一个好例子:同样的 bytes 流通过不同的解码器是可以解码的。而 bytes 本身不能指明它自己用的哪种编码方式。

顺便说一下，这些垃圾信息的显示只遵循一个规则，那就是乱码。

.. image:: http://nedbatchelder.com/text/unipain_035.png

不幸的是，bytes 流会根据自己的来源不同而进行不同的编码，有时候我们指明的编码方式可能是错误的。比如你有可能将一个 HTML 从网上抓取下来，HTTP 头中指明编码方式是 8859-1, 然而实际上的编码确是 UTF-8。

在一些情况下编码方式的不匹配可能会产生乱码，而有些时候，则会产生 UnicodeError。

.. image:: http://nedbatchelder.com/text/unipain_036.png

不用说。你应该测试你的 Unicode 支持。为了这样。你首先应该在你的代码中首先去先把 Unicode 来提取出。如果你只会说英语，这可能会有些困难。因为有些 Unicode 数据会比较难以读。幸运的是，大部分时候一些复杂结构的 Unicode 字符串还是比较具有可读性的。

这里是一个例子。ASCII 文本中可以读的文本，和倒置的文本。这些文本的一些有时候是一些青年人会粘贴到社交网络中。

.. image:: http://nedbatchelder.com/text/unipain_037.png

根据你的程序，你有可能在 Unicode 的道路中越挖越深。还有很多很多的细节我这里没有解释清楚。可以被涉及到。我们称之为事实五。因为你不必去对此了解太详细。


.. image:: http://nedbatchelder.com/text/unipain_038.png

复习一下，我们有五个不可忽视的事实:

1. 程序中所有的输入和输出均为 byte
2. 世界上的文本需要比 256 更多的符号来表现
3. 你的程序必须处理 byte 和 unicode
4. byte 流中不会包含编码信息
5. 指明的编码有可能是错误的

.. image:: http://nedbatchelder.com/text/unipain_039.png

这是你在编程中保持 Unicode 清洁的三个建议:

1. Unicode 三明治：尽可能的让你程序处理的文本都为 Unicode 。
2. 了解你的字符串。你应该知道你的程序中，哪些是 unicode, 哪些是 byte, 对于这些 byte 串，你应该知道，他们的编码是什么。
3. 测试 Unicode 支持。使用一些奇怪的符号来测试你是否已经做到了以上几点。

如果你遵循以上建议的话，你将会写出对 Unicode 支持很好的代码。不管 Unicode 中有多么不规整的编码你的程序也不会挂掉。


.. image:: http://nedbatchelder.com/text/unipain_040.png

一些其他你可能需要的资源

Joel Spolsky 编写的 `The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!) <http://www.joelonsoftware.com/articles/Unicode.html>`_ 概括了 Unicode 的工作方式和原因。虽然没有 Python 的内容，但是比我解释的详细多了!

如果你需要处理一些语义上的 Unicode 字符问题。那么 `unicodedata module <http://docs.python.org/library/unicodedata.html>`_ 也许会对你有些帮助。

如果你希望找一些 Unicode 来测试的话，网上各种的 `编码文本计算器 <http://fsymbols.com/generators/encool>`_ 会对你很有帮助。

.. image:: http://nedbatchelder.com/text/unipain_041.png
