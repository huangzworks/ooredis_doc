Set
========

``Set`` 类将 Redis 的 set 结构映射到一个集合对象，
它的操作和 Python 内置的 ``set`` 类型的实例完全一样。

以下代码演示了如何使用 ``Set`` 类：

::

    >>> from ooredis import Set

    >>> alpha = Set('alpha')        

    >>> alpha.add('a')                          # 加入新元素到集合

    >>> alpha.add('b')

    >>> len(alpha)                              # 查看集合基数
    2

    >>> alpha.random()                          # 随机返回集合中的一个元素
    'b'

    >>> alpha.random()
    'b'

    >>> alpha.random()
    'a'

    >>> alpha.pop()                             # 随机弹出并返回集合中的一个元素
    'b'

    >>> alpha |= set(['b', 'c', 'd'])           # 集合或操作

    >>> alpha
    Set Key 'alpha': set(['a', 'c', 'b', 'd'])

    >>> for a in alpha:
    ...   print(a)
    ... 
    a
    c
    b
    d

    >>> dir(alpha)
    ['__and__', '__class__', '__contains__', '__delattr__', '__dict__', '__doc__', '__eq__', '__format__',
    '__ge__', '__getattribute__', '__gt__', '__hash__', '__iand__', '__init__', '__ior__', '__isub__', '__iter__',
    '__le__', '__len__', '__lt__', '__module__', '__new__', '__or__', '__rand__', '__reduce__', '__reduce_ex__',
    '__repr__', '__ror__', '__rsub__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__',
    '__weakref__', '__xor__', '_client', '_decode', '_encode', '_represent', 'add', 'delete', 'exists', 'expire',
    'expireat', 'isdisjoint', 'issubset', 'issuperset', 'move', 'name', 'persist', 'pop', 'random', 'remove', 'ttl']


``Set`` 类实现了所有 Python 内置 ``set`` 类型支持的各种集合操作，比如并集、交集、差集，以及这些操作的赋值版本，比如 ``&=`` 、 ``|=`` 和 ``^=`` ，诸如此类：

::

    >>> x = Set('x')
    >>> x.add(1)
    >>> x.add(2)
    >>> x.add(3)

    >>> y = Set('y')
    >>> y.add(2)
    >>> y.add(5)
    >>> y.add(8)

    >>> x ^ y
    set([8, 1, 3, 5])

    >>> x & y
    set([2])

    >>> x | y
    set([1, 2, 3, 5, 8])

    >>> x - y
    set([1, 3])

    >>> z = Set('z')

    >>> z |= x

    >>> z
    Set Key 'z': set([1, 2, 3])

    >>> z |= y

    >>> z
    Set Key 'z': set([8, 1, 2, 3, 5])

    >>> z &= y

    >>> z
    Set Key 'z': set([8, 2, 5])


更方便的是，和 ``Set`` 对象进行集合操作的不仅可以是另一个 ``Set`` 对象，还可以是 Python 的 ``set`` 类型实例：

::

    >>> x
    Set Key 'x': set([1, 2, 3])

    >>> x & set([1, 2])
    set([1, 2])

    >>> x | set([1, 2])
    set([1, 2, 3])

    >>> x ^ set([1, 2])
    set([3])

    >>> x - set([1, 2])
    set([3])

    >>> x |= set([1, 2, 3, 4])

    >>> x
    Set Key 'x': set([1, 2, 3, 4])

    >>> x ^= set([1, 3, 5])

    >>> x
    set([2, 4, 5])


实例：使用集合操作实现好友关系
----------------------------------

基于 ``Set`` 类提供的方便的集合操作，我们可以在它之上，构建一个非常常用的好友关系系统。

其中，添加好友用 ``add`` 方法实现：

::

    >>> my_friend = Set('my-friend')
    >>> my_friend.add('peter')
    >>> my_friend.add('mary')

好友数量可以用 ``len`` 函数查看：

::

    >>> len(my_friend)
    2

检查一个人是否我的好友，可以用 ``in`` 关键字：

::

    >>> 'peter' in my_friend
    True
    >>> 'john' in my_friend
    False

最后，可以在多个用户之间进行集合操作，比如说，交集操作可以计算出我和 John 的共同好友：

::

    >>> john = Set('john-friend')
    >>> john.add('peter')
    >>> john.add('bob')

    >>> john
    Set Key 'john-friend': set(['bob', 'peter'])

    >>> my_friend
    Set Key 'my-friend': set(['peter', 'mary'])

    >>> my_friend & john
    set(['peter'])

或操作则可以计算出我和 John 的所有好友：

::

    >>> john | my_friend
    set(['bob', 'peter', 'mary'])


更多信息
------------

以上就是 ``Set`` 类的基本介绍， 更详细的 API 信息可以参考《底层实现 API》章节的 `Set 部分 <api/ooredis.key.html#module-ooredis.key.set>`_ 。
