#Unicdoe之痛
#实用Unicode编程指南

这是我在Pycon2012所做的演讲。你可以阅读本页的幻灯片和文字，或者直接在浏览器中打开[演示](http://nedbatchelder.com/text/unipain/unipain.html)，或者来看现场视频

同时，点击文章的图片将会进入所在幻灯片的的对应位置，图片中实用了Symbola字体，但是如果想要显示一些特殊符号的话，则需先将该字体下载下来。

![Pragmatic Unicode or how do i stop the pain](http://nedbatchelder.com/text/unipain_000.png)

大家好，我是Ned Batchelder.我已经有十年的Python编程经验，这以为着，很多很多的时候，我与其他程序员一样，犯过很多Unicode的编码错误。

![](http://nedbatchelder.com/text/unipain_001.png)

如果你和其他Python程序员一样，那你肯定也碰到过如下情况:你编写了一段很漂亮的代码，事情看起来很顺。然后某一天一个很奇怪的"方言字符"不知道从哪冒了出来，你的程序中就开始大量涌现UnicodeErrors。

你好像知道这种问题应该怎样解决，于是呢，就去在错误出现的地方添加了encode和decode，但是UnicodeError又开始出现在其他的地方。于是你又在另外一个地方添加了decode抑或encode。在你玩过一段"编码打地鼠"游戏之后，问题似乎被解决。

之后某一天，另一种"方言字符"又在另外一个地方出现了。然后你不得不又去来玩这种"打地鼠"直到问题解决掉。

![](http://nedbatchelder.com/text/unipain_002.png)

现在你的程序终于可以运行。但是你又烦恼又不适，这个问题花费了太多时间，你知道这样解决"正确"，于是开始憎恨自己。你对Unicode的主要了解就是你很讨厌它。

你不想去了解怪异的字符集，你只想要写一个你认为不是很糟糕的程序。

![](http://nedbatchelder.com/text/unipain_003.png)

你不必去玩打地鼠游戏.Unicode会有些麻烦，但是它并不难。了解了相关知识并且加以练习，你也可以方便的优雅的解决相关问题。

接下来我会教给你five Facts of lie，然后给你一些专业建议来解决Unicode问题。下面的内容将会包含Unicode基本知识，如何在Python 2和Python 3中来实现。他们有一定差异，但是你使用的基本策略都是一样的。

#####世界&Unicode
**********************************
![](http://nedbatchelder.com/text/unipain_004.png)

我们从Unicode基本知识开始。
![](http://nedbatchelder.com/text/unipain_005.png)
事实之一:计算机中的一切均为bytes(字节)。硬盘中的文件为一系列的byte组成，网络中传输的只有byte。所有的信息，在你写的程序中进进出出的，均由byte组成。

孤立的byte是毫无意义的，所以我们来赋予他们含义。
![](http://nedbatchelder.com/text/unipain_006.png)

为了表示各种文字，我们有大约50年的时间都在用ASCII码。每一个byte被赋予95种符号的一种，所以，当我给你发送byte值为65的时候，你知道我想表达一个小写的A。

![](http://nedbatchelder.com/text/unipain_007.png)

ISO Latin 1,或者 8859-1对ASCII的96多种字符进行了扩展。这也许是你用一个byte可以做的最多的事情了。因为byte中没有容量可以存储更多的符号了。

![](http://nedbatchelder.com/text/unipain_008.png)

在windows中，增加了另外27种字符。这种叫做CP1252编码。

![](http://nedbatchelder.com/text/unipain_009.png)

事实之二是，世界上的字符远远比256个要多。一个简单的byte不能够表达世界范围内的字符。在你玩"编码打地鼠"的时候，你多么的希望世界上所有的人都说英语，但是事实并不是这样,人们需要更多的符号来交流。

事实一和二共同造成了计算机设备结构与世界人类需求的一个冲突。

![](http://nedbatchelder.com/text/unipain_010.png)

当时为了解决冲突尝试了多种途径。通过一个byte来与符号或者字符进行对应的编码，每一种解决途径都没有解决事实二中的实质问题。???????

当时有很多一个byte的编码，都没有能够解决问题。每一个都只能解决人类语言的一部分。但是他们不能解决所有的文字问题。

![](http://nedbatchelder.com/text/unipain_011.png)

人们开始创造两个byte的字符集，但是仍然像碎片一样，只能够服务于不同地域的一部分人。

当时产生了不同的标准，讽刺的是，他们都不足以满足所有的符号的需求。

![](http://nedbatchelder.com/text/unipain_012.png)

Unicode就是为了解决之前的老的字符集问题。Unicode分配整形,被成为代码点(UNICODE的字符被成为代码点（CODE POINTS）用U后面加上XXXX来表现，其中，X为16进制的字符)来表示字符。他有110万的代码点，其中有十一万被占用，所以她可以有很多很多的空间可供未来的增长使用。

Unicode的目的是包含一切，它从ASCII开始，包含了数以千计的代码，包含这著名的----雪人??,包含了世界上所有的书写系统，而且一直在被扩充。比如，最新的更新中，就有一大堆没用的词汇。

![](http://nedbatchelder.com/text/unipain_013.png)

这里有六个的异国Unicode字符。Unicode代码点写成4-,5-,或者6位的十六进制编码，同时有一个U的前缀。每一个字符都有一个清楚的全程------总是在ASCII中是大写。
![](http://nedbatchelder.com/text/unipain_014.png)

所以说Unicode提供了所有我们需要的字符的空间。但是我们仍然需要处理事实一中所碰到的问题：计算机只能看懂bytes。我们需要一种用bytes来表示Unicode的方法这样才可以存储和传播他们。

Unicode标准定义了多种方法来用bytes来表示成代码点，被成为encoding。

![](http://nedbatchelder.com/text/unipain_015.png)

UTF-8是最流行的一种对Unicode进行传播和存储的编码方式。它用不同的bytes来表示每一个代码点。ASCII字符每个只需要用一个byte，与ASCII的编码是一样的。所以说ASCII是UTF-8的一个子集。

这里我们展现了几个怪异字符的UTF8的表示方法。ASCII字符H和I只用一个byte就可以表示。其他的根据代码点的不同使用了两个或者三个bytes。尽管有些并不常用，但是一些代码点使用到四个bytes。

#####Python 2
*****************************
![](http://nedbatchelder.com/text/unipain_016.png)

好，说完了这么多理论知识，我们来讲一讲Python 2
![](http://nedbatchelder.com/text/unipain_017.png)

在Python2中，有两种字符串数据类型
。一种纯旧式的文字:"str"对象,存储bytes。如果你使用一个"u"前缀，那么你会有一个"unicode"对象，存储的是code points。在一个unicode字符串中，你可以使用反斜杠u(\u)来插入任何的unicode代码点。

你可以注意到"string"这个词是有问题的。不管是"str"还是"unicode"都是一种"string"，这会吸引叫他们都是string，但是为了直接还是将他们明确区分来。
![](http://nedbatchelder.com/text/unipain_018.png)
如果想要在unicode和bytes间转换的话，每个都会有个方法。Unicode字符串会有一个 `.encode`方法来产生bytes,bytes串会有一个 `.decode` 方法来产生unicode。每个方法中都会有一个参数，来表明你要操作的编码类型。

我们可以定义一个Unicode字符串叫做my_unicode，然后看这九个字符，我们使用encode方法来创建my_unicode 的bytes串。会有19个bytes，想你所期待的那样。将bytes串来decode将会得到utf-8串。

![](http://nedbatchelder.com/text/unipain_019.png)

不幸的是，如果指明的编码名称错误的话，那么encode和decode会产生错误。现在我们去尝试encode我们的几个诡异的字符到ascii，会失败。因为ascii只能表示0-127个字符中的一个。然而我们的Unicode字符串早已经超出了范围。

抛出的异常为UnicodeEncodeError，它展现了你使用的编码方式，"codec"即编码，解码器，展现了导致问题的字符的位置。

![](http://nedbatchelder.com/text/unipain_020.png)

解码同样会知道出一些问题。现在我们去把一个UTF-8字符串解码成ASCII,会得到一个UnicodeDecodeError，原因一样，ASCII只接受127内的值，我们的UTF-8字符串超出了范围。

尽管UTF-8不能解码成任何的bytes串，我们尝试来decode一些垃圾信息。同样也产生了UnicodeDecodeError错误。最终，UTF-8的优势是，有效的bytes串，将会帮助我们来创建高鲁棒性的系统:如果数据无效的话，数据不会被接受。

![](http://nedbatchelder.com/text/unipain_021.png)

当编码或者解码的时候，你可以指明如果codec不能够处理数据的时候，会发生什么情况。encode或者decode时候的第二个参数指明了规则。默认的值是"strict"，意味着像刚刚一样，将会抛出一个异常。

"replace"值意味着，将会返回一个标准的替代字符。当编码的时候，替代值是一个问好，所以任何不能被编码的值将会产生一个"?"。

一些其他的handler非常有用。"xmlcharrefreplace"将会产生一个完全替代的HTML/XML字符，所以\u01B4将会变成"&#436"(因为十六进制的01B4是十进制的436)。如果你需要将返回的值来输出到html文件中的话，将会非常有用。

注意要根据不同的错误原因使用不同的错误处理方式。"Replace"是一个处理不能被解析的数据的自卫型方式，会丢失数据。"Xmlcharrefreplace"会保护所有的原始数据，在XML转义符可以使用的时候来输出数据。

![](http://nedbatchelder.com/text/unipain_022.png)

你也可以指定在解码的时候的错误处理方式。"Ignore"将会直接将不能解码的bytes丢掉。"Replace"将会直接添加Unicode U+FFFD,给有问题的bytes来直接替换成"替换字符"。注意因为解码器不能解码这些数据。它并不知道到底有多少Unicode字符。解码我们的UTF-8字符串成为ASCII制造出了16个"替换字符"。每个byte不能被解析都被替换掉了。然而这些bytes只想要表示6个Unicode字符。

![](http://nedbatchelder.com/text/unipain_023.png)

Python 2已经试图在处理unicode和byte串的时候变得有用些。如果你系那个要把Unicode字符串串和byte字符串来组合起来的话,Python2将会自动的将byte串来解码成unicode字符串。从而产生一个新的Unicode字符串。

比如，我们想要连接 Unicode串"hello"和一个byte字符串"world"。结果是一个Unicode的"hello world"。在我们看来。Python 2将"world" 使用 ASCII codec进行了解码。这次在解码中使用的字符集的值与 sys.getdefaultencoding()的值相等。

这里这个系统中的字符集为ASCII,因为这是唯一的一种猜测:ASCII如此被广泛接受。它是这么多编码的子集。不像是错误的。

![](http://nedbatchelder.com/text/unipain_023.png)

当然，这些隐藏的编码转换不能免疫于去解码错误。如果你想要连接 一个byte字符串和一个unicode字符串，并且byte字符串不能被解码成ASCII的话，将会抛出一个UnicodeDecodeError。

这就是那些可恶的UnicodeError的圆圈。你的代码中包含了unicode和byte字符串，只要数据全部是ASCII的话，所有的转换都是正确的，一旦一个非ASCII字符偷偷进入你的程序，那么默认的解码将会失效，从而造成UnicodeDecodeError的错误。

![](http://nedbatchelder.com/text/unipain_024.png)

Python 2的哲学就是Unicode字符串和byte字符串都是混乱的，他试图去通过自动转换来减轻你的负担。就像在int和float之间的转换一样，int到float的转换不会失败，byte字符串到unicode字符串会失败。

Python 2悄悄掩盖掉了byte到unicode的转换，让程序在处理ASCII的时候更加简单。你复出的代价就是在处理非ASCII的时候将会失败。

![](http://nedbatchelder.com/text/unipain_024.png)

有很多方法来合并两个字符串，所有的都会解析byte成为unicode，所以他们处理的时候你必须多加小心。

首先我们使用ASCII格式字符串，和unicode来结合。那么最终的输出将会变成unicode。返回一个unicode字符串。

之后我们将两个交换一下:一个unicode格式的字符串和一个byte串再一次合并，生成了一个unicode字符串，因为byte串可以被解码成ASCII。

简单的去打印出一个unicode字符串将会调用隐式的编码:输出总会是bytes,所以在unicode被打印之前必须被编码成byte串。

接下来的事情非常不可理解:我们让一个byte串编码成UTF-8，却得到一个错误说不能被解码成ASCII！这里的问题是byte串不能被编码，要记住编码是你将Unicode变成了byte串。所以想要执行你的操作的话，Python2需要的是一个unicode字符串，隐式的将你的字符串解码成ASCII。

最后，我们将ASCII字符串编码成UTF-8。现在我们进行相同的隐式编码操作，因为字符串为ASCII，编码成功。并且将它编码成了UTF-8,打印出了原始的byte字符串，因为ASCII是UTF-8的一个子集。

![](http://nedbatchelder.com/text/unipain_025.png)

最重要的事实之三:byte和unicode都非常重要，你必须将两个都处理好。你不能假设所有的字符串都是byte,或者所有的字符串都是unicode,你必须适当的运用他们必要时转换它们。

#####Python 3
****************************************
![](http://nedbatchelder.com/text/unipain_026.png)
我们看到了Python 2版本中有关Unicode之痛。现在我们看一下Python 3，在Python 2到Python 3中最重要的变化就是他们对Unicode的处理。

![](http://nedbatchelder.com/text/unipain_026.png)
跟Python 2类似，Python 3也有两种类型，一个是Unicode,一个是byte码。但是他们有不同的命名。

现在你从普通文本转换成"str"类型后存储的是一个unicode,"bytes"类型存储的是byte串。你也可以通过一个b前缀来制造byte串。

所以在Python 2中的"str"现在叫做"bytes"，而Python 2中的"unicode"现在叫做"str"。这比起Python 2中更容易理解，因为Unicode是你总想要存储的内容。而bytes字符串只有你在想要处理byte的时候得到。
![](http://nedbatchelder.com/text/unipain_027.png)

Python 3中对Unicode支持的最大变化就是将会没有对byte字符串的自动解码。如果你想要用一个byte字符串和一个unicode相链接的话，你将会得到一个错误，不管你包含的内容是什么。

所有这些在Python 2中都将会有隐式的处理，而在Python 3中你将会得到一个错误。

另外如果一个 Unicode字符串和byte字符串中包含的是相同的ASCII码，Python 2中将认为两个是相等的，而在Pthon 3中不会。这样做的结果是Unicode中的键不能找到byte字符串中的值，反之亦然，然而在Python 2中是可行的。
![](http://nedbatchelder.com/text/unipain_028.png)

这样彻底了改变了Python 3中的Unicode痛楚之源。在Python 2中，只要你使用ASCII数据，那么混合Unicode和byte将会成功，而在Python 3会直接忽略数据而失败。

这样的话在 Python 2中所遇到的，你认为你的程序是正确的，但是最后发现由于一些特殊字符而失败的错误就会避免。

Python 3中，你的程序马上就会产生错误，所以即使你处理的是ASCII码，那么你也必须处理bytes和Unicode之间的关系。

Python 3 中对于bytes和unicode的处理非常严格，你被迫去处理这些事情。这曾经引起争议。
![](http://nedbatchelder.com/text/unipain_029.png)

这样处理的原因之一是对读取文件的变化，Python 对于读取文件有两种方式，一种是二进制，一种是文本。在Python 2中，它只会影响到行尾符号。尽管是一个无作业的-------???

在Python 3中。这两种模式将会返回不同的结果。当你用文本模式打开一个文件时不管是你是用的"r"模式或者是它默认的模式，读取成的文件将会自动转码成unicode,你将会得到str对象。

如果你用二进制模式打开一个文，在参数中输入"rb"，那么从文件中读取的数据将会是bytes,对它们没有任何处理。

隐式的对bytes到unicode的处理使用的是 locale.getpreferedencoding()，然而它有可能输出你不想要的结果。比如，当你读取 hi_utf8.txt时，他被解码成语言偏好中所设置的编码方式，如果我们这些例子在windows中创建的话，那么就是"cp1252"。像ISO 8859-1,CP-1252这些可以得到任意的byte值，所以不会抛出UnicodeDecodeError,当然也意味着他们会直接将数据解码成CP-1252，制造出我们并不需要的垃圾信息。

为了文件读取正确的话，你应该指明想要的编码。open函数现在已经可以通过参数来指明编码。


#####减轻痛苦
![](http://nedbatchelder.com/text/unipain_030.png)
好，那么如何来减少这些痛苦？好消息是减轻痛苦的规则非常简单，在Python 2和 Python 3中都比较适用。
![](http://nedbatchelder.com/text/unipain_031.png)
正如我们在事实一中所看到的，在你的程序中进进出出的只有bytes,但是在你的程序中你不必处理所有的bytes。最好的策略是将输入的bytes马上解码成unicode。你在程序中均使用unicode,当在进行输出的时候，尽早将之编码成bytes。

制造一个Unicode三明治，bytes在外，Unicode在内。

要记者，有时候一些库将会帮助你完成类似的事情。一些库可能让你输入unicode,输出unicode .它会帮你完成转换的功能。比如Django在它的json模块中提供Unicode。

![](http://nedbatchelder.com/text/unipain_032.png)

第二条规则是。你需要知道你现在处理的是哪种类型的数据，在你的程序中任何一个位置，你需要知道你处理的是byte串还是一个unicode串。它不能是一种猜测，而应该被设计好。

另外，如果你有一个byte串的话，如果你想对他进行处理。那么你应该知道他是怎样的编码。

在对你的代码进行debug的时候，不能仅仅将之打印出来来看它的类型。你应该查看它的type,或者查看它repr之后的值来查看你的数据到底是什么类型。

![](http://nedbatchelder.com/text/unipain_033.png)

我曾经说过，你应该了解你的byte字符串的编码类型。好这里要我讲事实四:你不能通过检查它来判断这个字符串编码的类型。你应该通过其他途径来了解。比如很多协议中将会指明编码类型。这里我们给出HTTP,H™L,XML,Python源文件中的例子。你也可以通过预先的指定来了解编码。比如数据源码中可能会指明编码。

有一些方式可以来猜测一些 bytes的编码类型。但是仅仅是猜测。能够确定的唯一方式是通过其他方式。


![](http://nedbatchelder.com/text/unipain_034.png)

这里是给出一些怪异的字符的编码猜测。我们用UTF-8 便民店的一些字符，被不同的解码方式解码之后的输出。你可以看见。有时候用不正确的解码方式解码可能会输出正确，但是会输出错误的字符。你的程序不能告诉你这些解析错误了。只有当用户察觉到的时候你才会发现错误。

这是事实四的一个好例子:同样的bytes流通过不同的解码器是可以解码的。而bytes本身不能指明它自己用的哪种编码方式。

顺便说一下，这些垃圾信息的显示只遵循一个规则，那就是乱码。


![](http://nedbatchelder.com/text/unipain_035.png)

不幸的是，bytes流会根据自己的来源不同而进行不同的编码，有时候我们指明的编码方式可能是错误的。比如你有可能将一个HTML从网上抓取下来，HTTP头中指明编码方式是8859-1,然而实际上的编码确是UTF-8。
在一些情况下编码方式的不匹配可能会产生乱码，而有些时候，则会产生 UnicodeError。

![](http://nedbatchelder.com/text/unipain_036.png)

不用说。你应该测试你的Unicode支持。为了这样。你首先应该在你的代码中首先去先把Unicode来提取出。如果你只会说英语，这可能会有些困难。因为有些Unicode 数据会比较难以读。幸运的是，大部分时候一些复杂结构的Unicode字符串还是比较具有可读性的。

这里是一个栗子。ASCII文本中可以读的文本，和倒置的文本。这些文本的一些有时候是一些青年人会粘贴到社交网络中。


![](http://nedbatchelder.com/text/unipain_037.png)

根据你的程序，你有可能在Unicode的道路中越挖越深。还有很多很多的细节我这里没有解释清楚。可以被涉及到。我们称之为事实五。因为你不必去对此了解太详细。


![](http://nedbatchelder.com/text/unipain_038.png)

复习一下，我们有五个不可忽视的事实:

1.程序中所有的输入和输出均为byte
2.世界上的文本需要比256更多的符号来表现
3.你的程序必须处理byte和unicode
4.byte流中不会包含编码信息
5.指明的编码有可能是错误的



![](http://nedbatchelder.com/text/unipain_039.png)

这是你在编程中保持Unicode清洁的三个建议:

1.Unicode三明治:让所有你的程序中处理的信息均为Unicode,越详细越好。??

2.了解你的字符串。你应该知道你的程序中，哪些是unicode,哪些是byte,对于这些byte串，你应该知道，他们的编码是什么。

3.测试Unicode支持。使用一些奇怪的符号来测试你是否已经做到了以上几点。

如果你遵循以上建议的话，你将会写出对Unicode支持很好的代码。不管Unicode中有多么不规整的编码你的程序也不会挂掉。


![](http://nedbatchelder.com/text/unipain_040.png)

一些其他你可能需要的资源Joel Spolsky编写的[ The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](www.joelonsoftware.com/articles/Unicode.html)概括了Unicode的工作方式和原因。虽然没有Python的内容，但是比我解释的详细多了!

如果你需要处理一些语义上的Unicode字符问题。那么[unicodedata module](docs.python.org/library/unicodedata.html)也许会对你有些帮助。

测试Unicode，不同的[编码文本计算器](http://fsymbols.com/generators/encool)将会非常有用。


![](http://nedbatchelder.com/text/unipain_041.png)