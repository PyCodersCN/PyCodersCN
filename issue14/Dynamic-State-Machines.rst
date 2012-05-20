动态状态机
============

原文： `Dynamic State Machines <http://harkablog.com/dynamic-state-machines.html>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

昨天，我和同事 `Jeff <http://jeffelmore.org/>`_ 以及 Randy一起讨论Randy正在做的工作---在Django中实现一个有限状态机，然后我想起之前利用Python的动态类型实现过状态机。

在静态语言中，对象与其类型绑定，不能改变，那么要实现基于状态的行为就需要辅助性的类，通过在类之间的切换来实现不同状态的不同行为。Python的动态特征很自然地使这事简单得多。

使用动态类型实现状态
---------------------

Python中，一个对象的类型简单地决定于它的 ``__class__`` 属性的值。方便，但又令人相当震惊的是，这个属性是可变的。这意味着你可以通过简单地将一个新的类类型赋给 ``__class__`` 属性，就能改变一个对象的基本类型。

这种模式非常强大，也很容易被滥用，但是它能清晰高效地表达正在发生的事情，并且直接匹配我们的问题域。概念上，当一个对象改变状态，就成了一个不同类型的对象。实现上，当对象要改变状态，我们就改变对象的类型。如果我们想让不同的状态具有一些相同的行为，那么使用的类型可以是一个共同基类的子类，但这个是完全可选的。Python并不在意我们的状态是否相关。

在底层，当你试图访问某个对象的属性，Python首先查看对象自己是否有这个属性。如果有，就直接使用这个属性，如果没有，就检查对象所属的类，然后是超类，一直往上解析直到基类， ``object`` 。这个解释有些简单化了，忽略了Python赋予程序员的能力---使用 ``__getattr__`` 和 ``__getattribute__`` 来重载属性访问，但它对我们的目标来说够用了。

当对象改变状态时，它仍是那个对象，没有改变，具备以前所具备的属性，但是现在任何查找对象所属类的操作都会去查找新赋给对象的类类型，而不是在第一次创建对象时所使用的类类型。

.. image:: http://harkablog.com/static/images/morpho_menelaus.jpg

蝴蝶的生命周期
----------------

举例来说，我们来看看一个实现蝴蝶生命周期的状态机：

::

    class Egg(object):
        def __init__(self, species):
            self.species = species

        def hatch(self):
            self.__class__ = Caterpillar

    class Caterpillar(object):
        legs = 16
        def crawl(self): pass
        def eat(self): pass
        def pupate(self):
            self.__class__ = Pupa

    class Pupa(object):
        def emerge(self):
            self.__class__ = Butterfly

    class Butterfly(object):
        legs = 6
        def fly(self): pass
        def eat(self): pass
        def reproduce(self):
            return Egg(self.species)

对象生命开始时是一个 ``卵(Egg)`` ，孵化而成为一只 ``毛虫(Caterpillar)`` ，而后成为一只 ``蛹(Pupa)`` ，最后成为一只 ``蝴蝶(Butterfly)`` ，可以繁殖，创造新的 ``卵`` 。

实践过程看起来是这样的：

::

    >>> import buffterfly
    >>> critter = buffterfly.Egg('Morpho menelaus')
    >>> id(critter), type(critter), critter.species
    (10376583L, <class 'butterfly.Egg'>, 'Morpho menelaus')
    >>> hasattr(critter, 'legs')
    False

我们创建了一个卵对象---从各个方面来看都是一个普通的Python对象。这个实例有一个用户空间属性：物种(species)，在对象的整个生命周期中会一直存在。作为一个卵，没有legs属性，因为我们没有指定。卵唯一能做的事情就是孵化，然后成为一只 ``毛虫(Caterpillar)`` 。

::
    
    >>> critter.hatch()
    >>> id(critter), type(critter), critter.speices
    (10376583L, (class 'butterfly.Caterpillar'), 'Morpho menelaus')
    >>> critter.legs
    16
    >>> critter.crawl()
    >>> critter.eat()
    >>> critter.hatch()

(译注：这里的 ``critter.hatch()`` 的结果是一些方法调用错误信息，不知道作者是忘了写这个错误信息，还是故意不写的)

注意，孵化之后的critter的id与之前是一样。critter仍具有物种属性，保存着对象是一个 ``卵`` 时我们赋予它的值。现在critter有一个名为 ``legs`` 的类属性，表明这只 ``毛虫(Caterpillar)`` 有16条腿，可以爬和吃---在 ``Caterpillar`` 类中定义的那些方法，但是不再能孵化了。

::

    >>> critter.pupate()
    >>> id(critter), type(critter), critter.speices
    (10376583L, <class 'butterfly.Pupa'>, 'Morpho menelaus')
    >>> dir(critter)
    ['__class__', '__delattr__', '__dict__', '__doc__', '__format__',
    '__getattribute__', '__hash__', '__init__', '__module__', '__new__',
    '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
    '__str__', '__subclasshook__', '__weakref__', 'emerge', 'species']

这只 ``蛹(Pupa)`` 与之前 ``卵(Egg)`` 以及 ``毛虫(Caterpillar)`` 的id相同。它内部有一个实例属性 ``species`` (当然还是'Morpho menelaus')，以及一个方法, ``emerge()`` --- 能让我们的昆虫(critter)变成一只完全的 ``Butterfly`` 。

::

    >>> critter.emerge()
    >>> id(critter), type(critter), critter.species
    (10376583L, <class 'butterfly.Butterfly'>, 'Morpho menelaus')
    >>> critter.legs
    6
    >>> critter.eat()
    >>> critter.fly()
    >>> 'reproduce' in dir(critter)
    True
    >>> critterling = critter.reproduce()
    >>> id(critterling), type(critterling), critter.species
    (10377274L, <class 'butterfly.Egg'>, 'Morpho menelaus')

现在，这只完全成长的昆虫有六条腿。可以吃，飞，但是不再能爬(现实世界中的蝴蝶可以爬，但一个 ``Butterfly`` 对象不可以)，可以繁殖，创造会经历相同生命周期的critterling。

实现细节
----------

这是一种实现状态的强大模式，简单地通过改变模型的单个属性就能在行为上产生突变。它有一些特殊之处值得谨记：

- ``__init__`` 方法在状态改变时不会被调用。所以，你需要注意那些即使对象状态改变了仍需可用的实例属性。如果在状态改变之时需要做一些额外的事情，那么应该放在实现状态改变的方法中完成。

- 实例属性在对象的生命周期中一直存在，类属性只有在对象处于类所代表的状态中才可用。如果你需要一个特定对象上的属性，但这个属性只存在于特定状态，那么你需要自己来管理---在恰当的时间创建或销毁属性，或者以某种方式限制对属性的访问(这可能可以通过在不同的状态中创造性地使用 ``@property`` 装饰器来实现)。

- 一旦你理解其原理，重载对象的 ``__class__`` 属性也能是一种测试，调试，或者可以向程序员同事炫耀的强大技术。另一方面，为了代码更清晰明了，避免使用自作聪明的窍门也是更加聪明的做法，因此在良好控制的情况下，保守地使用这种技术。

快乐编码！

