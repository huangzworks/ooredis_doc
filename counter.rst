Counter
==========

``Counter`` 是一个计数器类，
它将自己映射到一个保存数字值（整数或浮点）的 Redis 键，
并实现了几个常用的计数操作，比如 ``incr`` 、 ``decr`` ，以及 Python 的 ``+=`` 和 ``-=`` 等等。

以下代码演示了如何使用 ``Counter`` 类：

::

    >>> from ooredis import Counter

    >>> c = Counter('my-counter')

    >>> dir(c)
    ['__class__', '__delattr__', '__dict__', '__doc__', '__eq__', '__format__',
    '__getattribute__', '__hash__', '__iadd__', '__init__', '__isub__', '__module__',
    '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
    '__str__', '__subclasshook__', '__weakref__', '_client', '_decode', '_encode', '_represent',
    'decr', 'delete', 'exists', 'expire', 'expireat', 'get', 'getset', 'incr',
    'name', 'persist', 'set', 'setex', 'setnx', 'ttl']

    >>> c.get()             # 没有值，返回 None

    >>> c.incr()            # 自增，默认增量为 1
    1

    >>> c.decr()            # 自减，默认减量为 1
    0

    >>> c.incr(10)          # 可以设置增量
    10

    >>> c.decr(10)          # 可以设置减量
    0

    >>> c += 10086          # 自增操作的语法糖，但是不返回值

    >>> c.get()
    10086

    >>> c -= 10086          # 自减操作的语法糖，不返回值

    >>> c.get()
    0

    >>> c.set(123)          # 和 String 类一样， Counter 也是先了 set 、 get 、 getset 等操作。

    >>> c.get()
    123

    >>> c.getset(10086)
    123

    >>> c.get()
    10086


实例：页面阅览计数
--------------------

``Counter`` 的一个典型应用案例是，统计网页的页面点击数量。

以下就是一个 flask 框架的例子，演示了如何统计特定用户的页面访问量：

::

    @app.route('/user/<int:uid>')
    def render_user_page(uid):
        # 每次访问时计数器加一
        pv_counter = Counter(str(uid) + 'pv')
        pv_counter += 1
        # ... 


实例：ID 生成器
------------------------

生成 ID 也是计数器的一个常见应用。

实现非常简单，只要用一个函数包裹 ``Counter`` 类，再进行相应的调用就可以了：

::

    from ooredis import Counter

    def generate_id(tag):
        counter = Counter(tag + 'id')
        return counter.incr()

接下来，只要用相应的 ``tag`` 来调用 ``generate_id`` 函数，就可以生成不同的 id 了：

::

    >>> generate_id('user')     # 生成用户 id
    >>> 1

    >>> generate_id('user')
    >>> 2

    >>> generate_id('book')     # 生成书籍 id
    >>> 1

    >>> generate_id('note')     # 生成笔记 id
    >>> 1


更多信息
----------

以上就是 ``Counter`` 类的基本介绍， 更详细的 API 信息可以参考《底层实现 API》章节的 `Counter 部分 <api/ooredis.key.html#module-ooredis.key.counter>`_ 。
