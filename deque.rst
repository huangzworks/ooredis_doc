Deque
=========

``Deque`` 类将 Redis 的 List 数据结构映射到双端列表对象上，
它的操作和 ``collections.deque`` 类完全一样，
另外加上了几个额外的 Redis 操作原语。

以下是 ``Deque`` 类的使用示例：

::

    >>> from ooredis import Deque

    >>> q = Deque('my-book-list')

    >>> q.append('SICP')                # 添加元素到右边

    >>> q.appendleft('CTMCP')           # 添加元素到列表左边

    >>> list(q)                         # 列出所有元素
    ['CTMCP', 'SICP']

    >>> len(q)                          # 列表长度
    2

    >>> q.pop()                         # 弹出最右边的元素
    'SICP'

    >>> q.popleft()                     # 弹出最左边的元素
    'CTMCP'

    >>> q.extend(['Algorithms in C', 'Real World Haskell', 'Programming Ruby'])     # 往左边加入多个元素

    >>> for book in q:
    ...   print(book)
    ... 
    Algorithms in C
    Real World Haskell
    Programming Ruby

    >>> q.extendleft(['APUE', 'CSAPP'])     # 往右边加入多个元素

    >>> for book in q:
    ...     print(book)
    ... 
    CSAPP
    APUE
    Algorithms in C
    Real World Haskell
    Programming Ruby

    >>> dir(q)
    ['__class__', '__delattr__', '__delitem__', '__dict__', '__doc__', '__eq__', '__format__',
    '__getattribute__', '__getitem__', '__hash__', '__init__', '__iter__', '__len__', '__module__',
    '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__',
    '__str__', '__subclasshook__', '__weakref__', '_client', '_decode', '_encode', '_represent', 'append',
    'appendleft', 'block_pop', 'block_popleft', 'clear', 'count', 'delete', 'exists', 'expire', 'expireat',
    'extend', 'extendleft', 'name', 'persist', 'pop', 'popleft', 'ttl']


实例：使用弹出原语实现消息队列
-----------------------------------

除了对双端列表 API 的支持以外， ``Deque`` 还带有 ``block_pop`` 和 ``block_popleft`` 两个方法，用于支持带阻塞的弹出操作：

::

    >>> q.block_pop(timeout=10)
    'CTMCP'

    >>> q.block_popleft(timeout=5)
    'TAoCP'

阻塞时间由 ``timeout`` 参数控制，它的默认为 ``0`` ，表示可以无限阻塞直到有元素弹出为止。

使用这些阻塞操作，可以实现一个简单的 FIFO 消息队列，其中，队列的一个方向用于接受元素，而队列的另一个方向用于弹出元素。

以下队列从列表左边弹出元素，从列表右边接收新元素：

::

    from ooredis import Deque

    def enque(item):
        q = Deque('message-queue')
        return q.append(item)

    def deque(item, timeout=0):
        q= Deque('message-queue') 
        return q.block_popleft(timeout=timeout)


更多信息
-----------

以上就是 ``Deque`` 类的基本介绍， 更详细的 API 信息可以参考《底层实现 API》章节的 `Deque 部分 <api/ooredis.key.html#module-ooredis.key.deque>`_ 。
