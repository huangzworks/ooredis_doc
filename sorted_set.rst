SortedSet
=============

``SortedSet`` 类将 Redis 的 Sorted Set 结构映射到一个有序集合对象上，它的操作有一点复杂，这主要是因为 Sorted Set 结构不像 Hash 结构那样，在 Python 里面有一对一的对象可供参照和映射。

在加入新元素时，我们将 ``SortedSet`` 看作是一个字典，其中 ``member`` 作为字典值，而将 ``score`` 赋值给这个字典属性。

以下是一个关于水果价格的有序集例子，添加新元素时，水果被当作字典的键，而价钱则被赋值到字典里：

::

    >>> p = SortedSet('fruit-price')

    >>> p['apple'] = 3

    >>> p['banana'] = 2

    >>> p['cherry'] = 1.5

当从 ``SortedSet`` 里提取出数据时，需要将它看作是一个有序的列表，这个列表里的每个元素都是一个字典，字典里包含 ``member`` 和 ``score`` 两个属性：

::

    >>> list(p)
    [{'member': 'cherry', 'score': 1.5}, {'member': 'banana', 'score': 2.0}, {'member': 'apple', 'score': 3.0}]

    >>> p[0]
    {'member': 'cherry', 'score': 1.5}

    >>> p[1]
    {'member': 'banana', 'score': 2.0}

    >>> p[2]
    {'member': 'apple', 'score': 3.0}

    >>> p[0]['member']
    'cherry'

    >>> p[0]['score']
    1.5

再次提醒， ``SortedSet`` 不是字典，不可以用 ``member`` 作为键去向 ``SortedSet`` 取值，这是错误的：

::

    >>> p['apple']
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "ooredis/key/helper.py", line 26, in wrapper
        raise TypeError
    TypeError

``SortedSet`` 在添加元素时像是一个字典，取出元素时却像一个列表，这初看上去有点奇怪，但是仔细思考的话，就会发现这种用法的实用性：

首先考虑取值，我们使用有序集是为了获得一个有序的列表，一般来说，我们想取出有序集中的第 N 项，比如 ``s[0]`` 取出有序集中排名第一的项，而 ``s[:10]`` 取出有序集的前 10 名，这种用法要比 ``s[member]`` 来取出某个成员的 ``score`` 值要常见得多。因此在取值时，一个列表形式的有序集可以极大地方便我们的取值操作。

如果的确要取出某个 ``member`` 的 ``score`` 值， ``SortedSet`` 类也提供了 ``score`` 方法：

::

    >>> p.score('apple')
    3.0

另一方面，像字典一样的赋值模式减轻了设置元素时的负担：我们不需要考虑某个元素是否存在于有序集，不需要考虑如何将新元素放到合适的位置，只要简单地通过 ``s[member] = score`` 赋值就可以了。

如果 ``member`` 不存在于有序集，那么元素会自动被放在合适的地方。如果 ``member`` 已经存在于有序集，它的 ``score`` 值会被更新，并自动调整自己在有序集中的位置。

以下是 ``SortedSet`` 类的一些其他方法的演示：

::

    >>> list(reversed(p))           # 逆序的有序集
    [{'member': 'apple', 'score': 3.0}, {'member': 'banana', 'score': 2.0}, {'member': 'cherry', 'score': 1.5}]

    >>> p.rank('apple')             # 查看给定 member 在有序集中的排名
    2

    >>> p.reverse_rank('apple')     # 查看给定 member 在有序集中的逆序排名
    0

    >>> dir(p)
    ['__class__', '__contains__', '__delattr__', '__delitem__', '__dict__', '__doc__', '__eq__', '__format__',
    '__getattribute__', '__getitem__', '__hash__', '__init__', '__len__', '__module__', '__new__', '__reduce__',
    '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__',
    '__weakref__', '_client', '_decode', '_encode', '_represent', 'decr', 'delete', 'exists', 'expire',
    'expireat', 'incr', 'name', 'persist', 'rank', 'remove', 'reverse_rank', 'score', 'ttl']


实例：排行榜
----------------

基于 ``SortedSet`` 提供的排序列表功能，我们可以方便地实现排行榜系统。

这里以手机价格为例子。先将一系列手机价格输入到 ``SortedSet`` 里面：

::

    >>> from ooredis import SortedSet

    >>> p = SortedSet('phone-price')

    >>> p['V N820'] = 999
    >>> p['Galaxy S3'] = 4399
    >>> p['iPhone 4S'] = 4299
    >>> p['S5830I'] = 1239
    >>> p['MEIZU MX'] = 2199
    >>> p['Honor U8860'] = 1499

``SortedSet`` 默认以 ``score`` 值从低到高的方式排序数据，因此，要计算价格最低的三款手机，只要取出 ``SortedSet`` 的前三项就可以：

::

    >>> p[:3]
    [{'member': 'V N820', 'score': 999.0}, {'member': 'S5830I', 'score': 1239.0}, {'member': 'Honor U8860', 'score': 1499.0}]

至于价格最贵的三款手机，可以用通过组合使用 ``list`` 和 ``reversed`` 函数计算出来：

::

    >>> list(reversed(p))[:3]
    [{'member': 'Galaxy S3', 'score': 4399.0}, {'member': 'iPhone 4S', 'score': 4299.0}, {'member': 'MEIZU MX', 'score': 2199.0}]


更多信息
----------

以上就是 ``SortedSet`` 类的基本介绍， 更详细的 API 信息可以参考《底层实现 API》章节的 `SortedSet 部分 <api/ooredis.key.html#module-ooredis.key.sorted_set>`_  。
