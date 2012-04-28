# 一行 Python 实现树

使用 Python 内置的 [`defaultdict`](http://docs.python.org/library/collections.html#collections.defaultdict)，我们可以很容易的定义一个树形数据结构：

```python
def tree(): return defaultdict(tree)
```

就是这样！

简单地来说，一颗树就是一个默认值是其子树的字典。

(使用之前需要确认已经 `from collections import defaultdict` 了)

(另: Hacker News 读者 @zbuc 指出这种结构被称为 [autovivification](https://en.wikipedia.org/wiki/Autovivification)。 太酷了！)

## 例子
### JSON 风格
现在我们可以创建一个 JSON 风格的嵌套字典，我们不需要显式地创建子字典——当我们需要时，它们神奇地自动出现了：

```python
users = tree()
users['harold']['username'] = 'hrldcpr'
users['handler']['username'] = 'matthandlersux'
```

我们可以将这些用 `print(json.dumps(users))` 以 JSON 的形式输出，于是我们得到：

```javascript
{"harold": {"username": "hrldcpr"}, "handler": {"username": "matthandlersux"}}
```

### 不需要赋值
我们甚至可以不需要任何赋值就可以创建整个树结构：

```python
taxonomy = tree()
taxonomy['Animalia']['Chordata']['Mammalia']['Carnivora']['Felidae']['Felis']['cat']
taxonomy['Animalia']['Chordata']['Mammalia']['Carnivora']['Felidae']['Panthera']['lion']
taxonomy['Animalia']['Chordata']['Mammalia']['Carnivora']['Canidae']['Canis']['dog']
taxonomy['Animalia']['Chordata']['Mammalia']['Carnivora']['Canidae']['Canis']['coyote']
taxonomy['Plantae']['Solanales']['Solanaceae']['Solanum']['tomato']
taxonomy['Plantae']['Solanales']['Solanaceae']['Solanum']['potato']
taxonomy['Plantae']['Solanales']['Convolvulaceae']['Ipomoea']['sweet potato']
```

我们接下来将漂亮地输出他们，不过需要先将他们转换为标准的字典：

```python
def dicts(t): return {k: dicts(t[k]) for k in t}
```

现在我们用 `pprint(dicts(taxonomy))` 来漂亮地输出结构：

```python
{'Animalia': {'Chordata': {'Mammalia': {'Carnivora': {'Canidae': {'Canis': {'coyote': {},
                                                                            'dog': {}}},
                                                      'Felidae': {'Felis': {'cat': {}},
                                                                  'Panthera': {'lion': {}}}}}}},
 'Plantae': {'Solanales': {'Convolvulaceae': {'Ipomoea': {'sweet potato': {}}},
                           'Solanaceae': {'Solanum': {'potato': {},
                                                      'tomato': {}}}}}}
```

于是我们引用到的子结构以字典的形式存在，空字典即代表了叶子。

### 迭代
这棵树可以很欢乐地被迭代处理，同样因为只要简单地引用一个结构它就会出现。

举例来说，假设我们想要解析一个新动物的列表，将它们加入我们上面的 taxonomy，我们只要这样调用一个函数：
```python
add(taxonomy,
    'Animalia,Chordata,Mammalia,Cetacea,Balaenopteridae,Balaenoptera,blue whale'.split(','))
```

我们可以简单地这样实现它：

```python
def add(t, keys):
  for key in keys:
    t = t[key]
```

再一次，我们完全没有对字典使用任何赋值，仅仅是引用了这些键，我们便创建了我们新的结构：

```python
{'Animalia': {'Chordata': {'Mammalia': {'Carnivora': {'Canidae': {'Canis': {'coyote': {},
                                                                            'dog': {}}},
                                                      'Felidae': {'Felis': {'cat': {}},
                                                                  'Panthera': {'lion': {}}}},
                                        'Cetacea': {'Balaenopteridae': {'Balaenoptera': {'blue whale': {}}}}}}},
 'Plantae': {'Solanales': {'Convolvulaceae': {'Ipomoea': {'sweet potato': {}}},
                           'Solanaceae': {'Solanum': {'potato': {},
                                                      'tomato': {}}}}}}
```

## 结论
这也许并不特别实用，而且也出现了一些令人困惑的代码 (见上面的 `add()`)。

不过如果你喜欢 Python，我希望思考这些会让你觉得有趣，或者值得去理解。