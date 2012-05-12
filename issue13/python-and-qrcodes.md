# Python 和 QR Code

几天前，我决定尝试来生成一些 QR Code。这篇文章展示了如何用 Python 来完成这个工作。

## 工具

用 Google 搜索 _QRCodes generation_ (_QR Code 生成_，译注)，我发现一些网站可以为你生成 QR Code。

其中 [ZXing 生成器](http://zxing.appspot.com/generator/) 和 [Kaywa 生成器](http://qrcode.kaywa.com/) 看起来十分强大和完善，不过我是在寻找一种可以在软件中直接使用，而不是需要发起网络请求的 QR Code 生成方式。

一个人 (MarkTraceur) 在我在 reddit 的帖子上做了评论，告诉我一个他创建的工具：[QRustom](https://qrustom.com/)！非常感谢他！

在 Python 中，你可以使用 [pyqrcode](http://pyqrcode.sourceforge.net/)，不过它使用一个 C/C++ 编码器和一个 Java 解码器……

我也发现了 [PyQRNative](http://code.google.com/p/pyqrnative/) 库，看起来是一个 [JavaScript 版生成器](http://d-project.googlecode.com/svn/trunk/misc/qrcode/js/qrcode.js) 的重写版本 (非常确信可以用这个 JS 库和 [Node.js](http://nodejs.org/) 做出非常棒的东西)。

它的代码 (你可以从[这里](http://pyqrnative.googlecode.com/svn/trunk/pyqrnative/src/PyQRNative.py) `wget` 它) 应该需要一次认真的重写以使之文档化并符合 [PEP8](http://www.python.org/dev/peps/pep-0008/)，不过它确实可以用 (这里有一个由 PyQRNative 生成的 QR Code 包含了这个帖子的 URL)。

**更新：**我在 reddit 发帖之后，Chris Beaven 告诉我他已经重写了它。他的代码已经在 [Pypi](http://pypi.python.org/pypi/qrcode) 上了。我也已经重写了这篇文章使用他的库。

请注意你同样需要安装 [Python Imaging Library](http://www.pythonware.com/products/pil/) (PIL) 来自己生成图像。

只需要运行：

	$ sudo pip install pil qrcode

## 使用

	from qrcode import *
	
	qr = QRCode(version=20, error_correction=ERROR_CORRECT_L)
	qr.add_data("http://blog.matael.org/")
	qr.make() # 自己生成 QR Code
	
	# im 包含了一个 PIL.Image.Image 对象
	im = qr.make_image()
	
	# 保存它
	im.save("filename.png")

在第三行我们使用两个参数实例化了一个新的 `qrcode.QRCode` 对象。这个类还有其他我们没有在这里用到的参数 (`box_size`、`border` 等等)。

第一个是 QR 的版本，一个在0到40之间的整数定义了条码的大小和我们能够储存的数据的量。

第二个是修正级别 (冗余度)。根据[维基百科的资料](http://en.wikipedia.org/wiki/QR_code)，我们可以从它们之中选择一个：

* `ERROR_CORRECT_L` 7%的内容可以被还原
* `ERROR_CORRECT_M` 15%的内容可以被还原 (默认)
* `ERROR_CORRECT_Q` 25% 的内容可以被还原
* `ERROR_CORRECT_H` 30% 的内容可以被还原

正是它的冗余特性使得即使代码被破坏也依然可以解码出来。

### 让库来猜测你需要什么

`qrcode` 模块给 `QRCode.make()` 方法添加了 `fit` 参数，如果使用了 `fit` 参数，`QRCode.version` 就会被设为 `None`，`qrcode` 将猜测正确的版本。

### 更快！

这是一个用来快速生成的很短的代码：

	import qrcode
	img = qrcode.make("your awesome data")

**Chris，感谢你伟大的作品！**

## 结论

QR Code 是一个非常优美的在设备之间分享数据的方式。它可以被用在很多应用上，从工厂内的产品跟踪到博客地址 URL。

它的冗余特性也让艺术化的使用和 QR Code 变形成为可能，你可以做出像这样的东西：

![](http://blog.matael.org/static/images/qr/qr_matael.png)

我真的觉得这个编码非常强大。

此外还有，使用 [processing](http://processing.org/) 和 QR Code，你可以实现[增强现实](http://processing.org/) ;)