快速入门
=========


在使用 OORedis 之前，需要先使用 ``connect`` 函数连接到 Redis 服务器：

::

    >>> from ooredis import *
    >>> connect()

因为 OORedis 的 ``connect`` 函数实际上的工作就是创建一个 redis-py 
库的 ``redis.Redis`` 对象的实例，
因此，任何用于 ``redis.Redis`` 的参数都可以作为 ``connect`` 函数的参数。

比如说，通过 ``db`` 参数，可以指定所使用的数据库编号：

::

    >>> connect(db=3)   # 使用 3 号数据库

通过 ``host`` 函数，可以指定连接的 Redis 服务器地址：

::

    >>> connect(host='127.0.0.1')   # 连接到地址为 127.0.0.1 的 Redis 服务器

诸如此类。


你好， Key 类！
-------------------

在 Redis 中，所有的操作都是针对命令，以及 key 名来进行的：

::

    redis> SET 'a-string-key' 'value-of-the-string-key'
    OK

    redis> GET 'a-string-key'
    "value-of-the-string-key"

上面的这段代码通过 ``SET`` 命令完成了一次对
key 名为 ``a-string-key`` 的字符串设置操作。

在 redis-py 库中，
同样的操作可以通过调用 ``Redis`` 对象的 ``set`` 和 ``get`` 方法来进行：

::

    >>> from redis import Redis
    >>> r = Redis()
    >>> r.set('a-string-key', 'value-of-the-string-key')
    True
    >>> r.get('a-string-key')
    'value-of-the-string-key'

OORedis 的做法是，
将 Redis 的各个相关的命令组合成一个个 Key 类，
每个 Key 类都接受一个 key 名作为参数，
通过调用 Key 类的不同方法来对 key 进行各种操作。

举例来说，以下是使用 String 类在 OORedis 
中对字符串 key ``a-string-key`` 进行 ``get`` 和 ``set`` 操作的方法：

::

    >>> s = String('a-string-key')          # 传入 a-string-key 作为 key
    >>> s.set('value-of-the-string-key')
    >>> s.get()
    'value-of-the-string-key'

除了 ``String`` 类之外， OORedis 还有其他几个 Key 类，
分别表示不同用途的几个数据结构，
以下是 OORedis 的所有 Key 类：

- ``String`` 类： 保存字符串值，常用于缓存操作

- ``Counter`` 类： 计数器，可以进行自增、自减等操作

- ``Dict`` 类： 哈希字典，保存键-值对数据

- ``Deque`` 类： 双端队列，常用于消息队列

- ``Set`` 类： 集合，常用于关系操作

- ``SortedSet`` 类： 有序集合，常用于排名或频率统计


实例：用 Dict 类实现论坛发贴
-------------------------------------------

上一节列出了 OORedis 中的所有 Key 类，
为了让这个快速入门章节保持简单，
在这一节，
我们通过一个简单的例子来学习如何使用 OORedis 提供的 Key
类来解决给定的问题，
在文档的后续部分，
我们再接着详细地深入探讨各个 Key 类。

假设你正在构建一个论坛，
其中每个帖子的数据可以用
``id`` 、 ``title`` 、 ``author`` 和 ``content`` 四个属性来表示，
比如这样：

====== ======================== ========== ====================================
 id      title                    author       content
====== ======================== ========== ====================================
10086   "say hello to OORedis"   "huangz"    "xyz..."
10087   "how to install Redis?"  "newbie"    "how to install ..."
10088   "PyConf 2012"            "john"      "PyConf 2012 video download ..."
====== ======================== ========== ====================================

观察表里面的数据可以会发现，
这些数据都是可以由一个唯一的 ``id`` 标识，
至于其他三个属性 ``title`` 、 ``author`` 和 ``content`` ，
则分别对应一个值。

在 Redis 中保存这种『键-值对』（key-value pair）形式的数据，
最适合的方法就是使用 Hash 结构，
在 OORedis 中，
操作 Hash 结构的函数都被很好地映射到 ``Dict`` 类的方法当中。

比如说，以下是 Redis 中一个使用 ``hset`` 和 ``hget`` 的例子：

::

    redis 127.0.0.1:6379> HSET redis-key hash-key hash-value
    (integer) 1

    redis 127.0.0.1:6379> HGET redis-key hash-key
    "hash-value"

而这两个操作同样可以使用 OORedis 来完成：

::

    >>> d = Dict('redis-key')

    >>> d['hash-key'] = 'hash-value'

    >>> d['hash-key']
    'hash-value'

以上的 OORedis 操作和之前的 Redis 操作完成的是一样的工作，
不同的是，
``Dict`` 类将使用 Hash 结构的方式变得像使用 Python 
内置的 ``dict`` 对象一样简单。

当然，
虽然 ``Dict`` 类的和内置的 ``dict`` 对象的操作非常相似，
但是它们之间有一个非常显著的区别，那就是：
``dict`` 只将数据保存在内存中，
而 ``Dict`` 则是通过写入和读取 Redis 数据库来保存和读取数据。

比如说，我们可以创建一个新的 ``Dict`` 对象，
并往里面写入一些数据，
这些数据就会被写入到 Redis 数据库中：

::

    >>> project = Dict("ooredis")

    >>> project['name'] = "OORedis"
    >>> project['language'] = "Python"
    >>> project['type'] = "Database Mapper"

可以在 Redis 中使用命令来确认这一点：

::

    redis 127.0.0.1:6379> HGETALL ooredis
    1) "name"
    2) "OORedis"
    3) "language"
    4) "Python"
    5) "type"
    6) "Database Mapper"

好的，对 ``Dict`` 类的介绍暂时就到此为止，
既然已经知道 ``Dict`` 类的使用方式，
那么现在可以将之前的帖子数据都通过 ``Dict`` 类保存起来了：

::

    >>> t_10086 = Dict(10086)
    >>> t_10086['title'] = "say hello to OORedis"
    >>> t_10086['author'] = "huangz"
    >>> t_10086['content'] = "xyz..."

    >>> t_10087 = Dict(10087)
    >>> t_10087['title'] = "how to install Redis?"
    >>> t_10087['author'] = "newbie"
    >>> t_10087['content'] = "how to install ..."

    >>> t_10088 = Dict(10088)
    >>> t_10088['title'] = "PyConf 2012"
    >>> t_10088['author'] = "john"
    >>> t_10088['content'] = "PyConf 2012 video download ..."

以上表达式在 redis-py 中实际执行以下命令：

::

    >>> r.hset(10086, 'title', 'say hello to OORedis')    # r 是 redis.Redis 对象的实例
    1L
    >>> r.hset(10086, 'author', 'huangz')
    1L
    >>> r.hset(10086, 'content', 'xyz...')
    1L

    >>> r.hset(10087, 'title', 'how to install Redis?')
    1L
    >>> # ...

可以将这个创建帖子的动作抽象为一个函数 ``create_topic`` ：

::

    def create_topic(id, title, author, content):
        new_topic = Dict(id)
        new_topic['title'] = title
        new_topic['author'] = author
        new_topic['content'] = content

用这个 ``create_topic`` 函数创建一个新帖子试试：

::

    >>> create_topic(
            10089,
            "welcome to OORedis document!",
            "huangz",
            "OORedis
        )
    >>>

通过将一个已存在的 ``id`` 作为 ``key`` 传入 ``Dict`` 对象，
可以查看这个帖子的各个属性：

::

    >>> t = Dict(10089)

    >>> t['title']
    'welcome to OORedis document!'

    >>> t['author']
    'huangz'

    >>> t['content']
    'OORedis is a ...'

以上表达式在 redis-py 中实际执行以下命令：

::
    
    >>> r.hget(10089, 'title')    # r 是 redis.Redis 对象的实例
    'welcome to OORedis document'

    >>> r.hget(10089, 'author')
    'huangz'

    >>> r.hget(10089, 'content')
    'OORedis'

查看帖子的动作同样可以抽象成一个 ``read_topic`` 函数：

::

    def read_topic(id):
        topic = Dict(id)
        if topic.exists:
            return dict(topic)
        else:
            raise Exception("topic not found")

``read_topic`` 中的 ``topic.exists`` 用于检查帖子 ``id`` 是否存在，
效果等同于执行 Redis 的 ``EXISTS`` 命令，
如果指定的 ``id`` 存在，那么将帖子的数据转换成一个字典并返回，
否则的话，就抛出一个异常。

试试使用 ``read_topic`` 查看刚刚创建的帖子：

::

    >>> read_topic(10089)
    {'content': 'OORedis is a ...',
     'author': 'huangz',
     'title': 'welcome to OORedis document!'}

试试使用 ``read_topic`` 查看一个不存在的帖子：

::

    >>> read_topic(123456789)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 6, in read_topic
    Exception: topic not found

以上就是一个简单的使用 ``Dict`` 类来创建和阅读论坛帖子的例子，
可以看到，
``Dict`` 类实际上执行的工作和调用 redis-py 执行 ``HSET`` 或者 
``HGET`` 命令没有什么两样，
但比起使用 redis-py ， ``Dict`` 处理数据的方式更 Pythonic  ，
也更简单快捷。


自动类型转换
----------------

在 Redis 中，所有输入都会被转换成字符类型，
因此，每次使用 redis-py 在数据库进行读取操作时，
都要对取出的数据进行类型转换：

::

    >>> r = Redis()

    >>> r.hset('type', 'int', 10086)
    1L
    >>> r.hset('type', 'float', 3.14)
    1L
    >>> r.hset('type', 'str', 'hello, world!')
    1L

    >>> r.hmget('type', 'int', 'float', 'str')      # 所有数据都被转换成了字符串
    ['10086', '3.14', 'hello, world!']

    >>> int(r.hget('type', 'int'))
    10086
    >>> float(r.hget('type', 'float'))
    3.14
    >>> r.hget('type', 'str')
    'hello, world!'

频繁的类型转换工作不仅让人厌烦，
而且非常容易出错。

为了解决这个问题， OORedis 增加了一种称为 TypeCase 的类型转换机制，
TypeCase 可以在创建 Key 对象时通过 ``type_case`` 参数传入，
通过指定各种不同的 TypeCase 类，
OORedis 可以在写入和读取的时候自动对数据进行类型转换：

::

    >>> from ooredis.type_case import GenericTypeCase

    >>> t = Dict('ooredis-type', type_case=GenericTypeCase)

    >>> t['int'] = 10086
    >>> t['float'] = 3.14
    >>> t['str'] = 'hello, world'

    >>> dict(t)                 # 所有值的类型不变
    {'int': 10086,
     'float': 3.14,
     'str': 'hello, world'}

    >>> type(t['int'])
    <type 'int'>
    >>> type(t['float'])
    <type 'float'>
    >>> type(t['str'])
    <type 'str'>

在上面的代码例子中，
``Dict`` 实例接受了 ``GenericTypeCase`` 作为 ``type_case`` 参数，
``GenericTypeCase`` 接受 ``int`` 、 ``float`` 、 ``str`` 、 ``unicode`` 
四种类型的值，并且在取出数据时将数据转换回原来的类型。

顺带一提，因为 ``GenericTypeCase`` 是所有 Key 对象的默认 ``type_case`` 值，
因此，前面的代码例子也可以简化为：

::

    >>> t = Dict('ooredis-type')

    >>> ...

除了 ``GenericTypeCase`` 之外，
OORedis 还内置了其他一些 TypeCase 类，
分别可以用于不同类型的值的转换：

- ``IntTypeCase`` ：接受整数值、或者表示为整数值的字符串值（比如 ``"10086"`` ），
  并返回整数值。

- ``FloatTypeCase`` ：接受浮点数值、
  或者表示为整数值的字符串值（比如 ``"3.14"`` ），
  并返回浮点数值。

- ``JsonTypeCase`` ：接受 JSON 类型的值，并返回 JSON 类型的值。

- ``SerializeTypeCase`` ：使用 Python 的 ``Pickle`` 模块，
  可以将 Python 对象保存在 Redis 数据库中，
  并在取出的时候自动还原成 Python 对象。

- ``StringTypeCase`` ：接受 ``str`` 或者 ``uncide`` 类型的值，
  并在取出的时候将输入值转换成原来的类型。

如果这些 TypeCase 类都不符合你的要求，
你还可以编写自己的 TypeCase 类，
在稍后的文档里会介绍编写 TypeCase 类的方法。


内置文档
---------

经过前面几个小节的介绍，
我想你已经准备好载入 OORedis 库，
并开始进行荒野求生式的探险了。

但是，先等等，别着急，还有一样很重要的工具还没介绍给你，
那就是 OORedis 的内置文档。

在 OORedis 里，
所有的函数、类和方法，都带有详细的内置文档，
这些文档很好地记录了函数/方法的参数、返回值、时间复杂度和可能抛出的异常，
如果在探险的过程中遇上什么问题，可以随时查阅这些文档：

::

    Help on class Dict in module ooredis.mix.dict:

    class Dict(ooredis.mix.key.Key, _abcoll.MutableMapping)
    |  一个字典对象，底层是 Redis 的 Hash 结构。
    |  
    |  Method resolution order:
    |      Dict
    |      ooredis.mix.key.Key
    |      _abcoll.MutableMapping
    |      _abcoll.Mapping
    |      _abcoll.Sized
    |      _abcoll.Iterable
    |      _abcoll.Container
    |      __builtin__.object
    |  
    |  Methods defined here:
    |  
    |  __delitem__(*args, **kwargs)
    |      删除字典键 key 的值。
    |      如果键 key 的值不存在，那么抛出 KeyError 。
    |      
    |      Args:
    |          key
    |      
    |      Time:
    |          O(1)
    |      
    |      Returns:
    |          None
    |      
    |      Raises:
    |          KeyError: key 不存在时抛出。
    |          TypeError: Key 对象不是 Dict 类型时抛出。
    |  
    |  __getitem__(*args, **kwargs)
    |      返回字典中键 key 的值。
    |      如果键 key 在字典中不存在，那么抛出 KeyError 。
    |      
    |      Args:

    ...
    

小结
-----

在这个快速入门章节中，我们看到了如何通过 ``connect`` 函数连接 Redis 服务器，
知道了 OORedis 各个 Key 类的大概作用，
并且练习了怎样使用 ``Dict`` 实现论坛的发贴和读贴功能，
看到了如何使用 TypeCase 进行自动类型转换，等等。

希望你通过这些简短的介绍，
能对 OORedis 是什么以及能做什么有了大概的感觉。

在接下来的部分，
文档会陆续介绍 OORedis 的其他 Key 类，
你将看到 OORedis 是怎样用简单快捷的方式来解决各种实际问题的。
