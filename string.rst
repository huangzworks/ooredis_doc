String
===========

``String`` 类，类如其名，
它的工作就是负责从 Redis 数据库中写入或读取字符串值。

除了从 ``BaseKey`` 里继承来的那些方法之外，
``String`` 类还实现了 ``set`` 、 ``setex`` 、
``setnx`` 、 ``get`` 和 ``getset`` 五个方法。

以下代码演示了如何使用 ``String`` 类：

::

    >>> s = String('str')

    >>> dir(s)
    ['__class__', '__delattr__', '__dict__', '__doc__', '__eq__', '__format__',
    '__getattribute__', '__hash__', '__init__', '__module__', '__new__',
    '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
    '__str__', '__subclasshook__', '__weakref__', '_client', '_represent',
    '_type_case', 'delete', 'exists', 'expire', 'expireat', 'get', 'getset',
    'name', 'persist', 'set', 'setex', 'setnx', 'ttl']

    >>> s.get()                                     # 不存在的 key 返回 None

    >>> s.set("i'm a string!")                      # 给 key 设置值

    >>> s.get()                                     # 返回 key 值
    "i'm a string!"

    >>> s.setnx("this message will not be set")     # setnx 只会在 key 不存在时设置值
    False

    >>> s.setex("i'm a string with ttl", 10086)     # setex 设置值和生存时间
    True

    >>> s.ttl                                       # 生存时间
    10084L

    >>> s.getset("new string here!")                # getset 设置新值并返回旧值
    "i'm a string with ttl"


实例：使用 String 类配合 TypeCase 类保存不同类型的值
--------------------------------------------------------

有些时候，除了保存单个的字符串值之外，
我们还想保存其他别的类型的值，
比如整数、浮点数、JSON 对象甚至是一个 Python 对象。

通过使用 ``String`` 类配合相应的 TypeCase 类，
可以达到保存不同类型的值的目的。

比如说，以下就是用于保存单个整数值的 ``Integer`` 类：

::

    >>> from ooredis.type_case import IntTypeCase

    >>> class Integer(String):
    ...     def __init__(self, *args, **kwargs):
    ...         kwargs.update({'type_case': IntTypeCase})           # 将 TypeCase 设置为 IntTypeCase
    ...         super(Integer, self).__init__(*args, **kwargs)
    ... 

    >>> i = Integer('int-key')

    >>> i.get()                     # 不存在 key 的默认值为 None

    >>> i.set(10)

    >>> i.get()
    10

    >>> type(i.get())
    <type 'int'>

    >>> i.set("a string value")     # 尝试设置错误的类型的值
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/lib/python2.7/site-packages/ooredis/key/string.py", line 24, in wrapper
        return func(*args, **kwargs)
    File "/usr/lib/python2.7/site-packages/ooredis/key/string.py", line 55, in set
        redis_value = self._type_case.to_redis(python_value)
    File "/usr/lib/python2.7/site-packages/ooredis/type_case/int_type_case.py", line 19, in to_redis
        raise TypeError
    TypeError

``Integer`` 类只接受和返回整数类型的值（或者能被强制转换为整数的字符串值），
如果向它 ``set`` 一个非整数类型的值，
那么它会抛出 ``TypeError`` 。

除了单个值之外， ``String`` 还可以配合 ``JsonTypeCase`` 
或 ``SerializeTypeCase`` 来保存多个值或对象。

作为例子， ``Json`` 类的定义如下：

::

    >>> from ooredis.type_case import JsonTypeCase

    >>> class Json(String):
    ...     def __init__(self, *args, **kwargs):
    ...         kwargs.update({'type_case': JsonTypeCase})      # TypeCase 设置为 JsonTypeCase
    ...         super(Json, self).__init__(*args, **kwargs)
    ... 

    >>> j = Json('json-key')

    >>> j.set({'name': 'ooredis',
    ...        'lang': 'python',
    ...        'version': 1.9,})

    >>> j.get()
    {u'lang': u'python', u'version': 1.9, u'name': u'ooredis'}

    >>> type(j.get())
    <type 'dict'>

``Json`` 类接受一个或多个值（只要它们是 JSON 能接受的值就可以了），
然后将这些只转换成 JSON 对象，并保存到 Redis 中，
当从数据库中取出数据时，
``Json`` 类将所得的 JSON 对象转换回原来的 Python 类型。


实例：使用 String 实现缓存（caching）
-----------------------------------------

缓存是 Redis 的 string 结构的常见用法之一，
在 OORedis 中，
我们可以通过 ``JsonTypeCase`` 或者 ``SerializeTypeCase`` ，
配合 ``setex`` 、 ``setnx`` 等方法，
实现缓存功能。

假设现在有一些论坛的帖子数据要缓存，
其中每个帖子都可以被转换成 JSON 对象：

::

    >>> topic = {
    ...           'id': 10086,
    ...           'title': 'welcome to OORedis bbs',
    ...           'author': 'huangz',
    ...           'date': 2012-6-19,
    ...           'content': '...',
    ...         }

要缓存这种帖子数据，可以使用以下 ``set_cache`` 函数
（会用到前一节定义的 ``Json`` 对象）：

::

    >>> DEFAULT_CACHING_TIME = 3600
    >>> def set_cache(key, value, ttl=DEFAULT_CACHING_TIME):
    ...     cache = Json(key)
    ...     cache.setex(value, ttl)
    ... 

读取缓存可以使用 ``get_cache`` 函数：

::

    >>> def get_cache(key):
    ...     return Json(key).get()

用 ``set_cache`` 设置的缓存会在 ``ttl`` 秒之后过期，
而 ``get_cache`` 负责返回缓存，
如果缓存不存在，那么返回 ``None`` 。

演示：

::

    >>> topic = {
    ...           'id': 10086,
    ...           'title': 'welcome to OORedis bbs',
    ...           'author': 'huangz',
    ...           'date': 2012-6-19,
    ...           'content': '...',
    ...         }

    >>> set_cache(topic['id'], topic)   # 将帖子的 id 用作缓存的 key

    >>> get_cache(10086)                # 注意：经过 JSON 编码的字符串值都会变成 unicode 类型
    {u'date': 1987,
     u'content': u'...',
     u'title': u'welcome to OORedis bbs',
     u'id': 10086, u'author': u'huangz'}


更多信息
------------

以上就是 ``String`` 类的基本介绍，
更详细的 API 信息可以参考《底层实现 API》章节的
`string 部分 <api/ooredis.key.html#module-ooredis.key.string>`_ 。
