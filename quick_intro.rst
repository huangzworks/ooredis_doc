快速入门
=========


这个小节会告诉你如何使用 OORedis 的基本功能，
以及这些功能是如何让 Python 操作 Redis 变得简单的。


连接 Redis
---------------

在使用 OORedis 之前，需要先使用 ``connect`` 函数连接到 Redis 服务器：

::

    >>> from ooredis import *
    >>> connect()

因为 OORedis 的 ``connect`` 函数实际上的工作就是创建一个 redis-py 库
的 ``redis.Redis`` 对象的实例，
因此，任何用于 ``redis.Redis`` 的参数都可以作为 ``connect`` 函数的参数。

比如说，通过 ``db`` 参数，可以指定所使用的数据库编号：

::

    >>> connect(db=3)   # 使用 3 号数据库

通过 ``host`` 函数，可以指定连接的 Redis 服务器地址：

::

    >>> connect(host='127.0.0.1')   # 连接到地址为 127.0.0.1 的 Redis 服务器

诸如此类。


你好， Key 对象！
-------------------

在 Redis 中，所有的操作都是针对命令，以及 key 名来进行的：

::

    redis> SET 'a-string-key' 'value-of-the-string-key'
    OK

    redis> GET 'a-string-key'
    "value-of-the-string-key"

上面的这段代码通过 ``SET`` 命令完成了一次
对 key 名为 ``a-string-key`` 的字符串设置操作。

在 redis-py 库中，同样的操作可以通过调用 ``Redis`` 对象的 ``set`` 和 ``get`` 方法来进行：

::

    >>> from redis import Redis
    >>> r = Redis()
    >>> r.set('a-string-key', 'value-of-the-string-key')
    True
    >>> r.get('a-string-key')
    'value-of-the-string-key'

OORedis 将 Redis 的各个相关的命令组合成一个个 Key 类，
每个 Key 类都接受一个 key 名作为参数，
通过调用 Key 类的不同方法来对 key 进行各种操作。

以下是使用 String 类在 OORedis 中对
字符串 key ``a-string-key`` 进行 ``get`` 和 ``set`` 操作的方法：

::

    >>> s = String('a-string-key')          # 传入 a-string-key 作为 key
    >>> s.set('value-of-the-string-key')
    >>> s.get()
    'value-of-the-string-key'

除了 ``String`` 类之外， OORedis 还有其他几个 Key 类，
分别表示不同用途的几个数据结构：

- ``String`` 类： 保存字符串值，常用于缓存操作

- ``Counter`` 类： 计数器，可以进行自增、自减等操作

- ``Dict`` 类： 哈希字典，保存键-值对数据

- ``Deque`` 类： 双端队列，常用于消息队列

- ``Set`` 类： 集合，常用于关系操作

- ``SortedSet`` 类： 有序集合，常用于排名或频率统计

为了保证这个『快速入门』真的足够『快速』，
这里只给出一个简单的例子来展示 ``Dict`` 类的功能，
至于其他类，在文档的后续部分会给出详细的介绍。


实例：用 Dict 类实现论坛发贴
-------------------------------------------

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

很明显，你需要使用 OORedis 里的某个 Key 类来完成将数据保存到数据库的工作，
作为例子，上面的帖子数据可以通过执行以下表达式来保存：

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

``Dict`` 类的操作和 Python 内置的 ``set`` 对象的操作几乎一模一样，
其中的一个不同点是， ``Dict`` 不仅仅将数据保留在内存里，
它还会将赋值给 ``Dict`` 实例的数据保存到 ``Redis`` 数据库里，
可以通过命令在 Redis 里确认这一点：

::

    redis> HGETALL 10086
    1) "title"
    2) "say hello to OORedis"
    3) "author"
    4) "huangz"
    5) "content"
    6) "xyz..."

    redis> HGETALL 10087
    1) "title"
    2) "how to install Redis?"
    3) "author"
    4) "newbie"
    5) "content"
    6) "how to install ..."

    redis> HGETALL 10088
    1) "title"
    2) "PyConf 2012"
    3) "author"
    4) "john"
    5) "content"
    6) "PyConf 2012 video download ..."

这个创建帖子的动作可以抽象为一个函数 ``create_topic`` ：

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

这时可以通过使用 ``id`` 来实例化一个 ``Dict`` 对象，
用于查看帖子的各个属性：

::

    >>> t = Dict(10089)

    >>> t['title']
    'welcome to OORedis document!'

    >>> t['author']
    'huangz'

    >>> t['content']
    'OORedis is a ...'

以上查看帖子的动作同样可以抽象成一个 ``read_topic`` 函数：

::

    def read_topic(id):
        topic = Dict(id)
        if topic.exists:
            return dict(topic)
        else:
            raise Exception("topic not found")

``read_topic`` 中的 ``topic.exists`` 用于检查帖子是否存在，
效果等同于执行 Redis 命令 ``EXISTS`` ，
如果指定的帖子存在，那么将帖子的数据转换成一个字典并返回，
否则的话，就抛出一个异常。

试试使用 ``read_topic`` 查看刚刚创建的帖子：

::

    >>> read_topic(10089)
    {'content': 'OORedis is a ...', 'author': 'huangz', 'title': 'welcome to
    OORedis document!'}

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
但比起使用 redis-py ， ``Dict`` 处理数据的方式更有 Pythonic 味 ，
也更简单快捷。


小结
-----

在这个快速入门小节中，我们看到了如何通过 ``connect`` 函数连接 Redis 服务器，
知道了 OORedis 各个 Key 类的大概作用，
并且练习了怎样使用 ``Dict`` 实现论坛的发贴和读贴功能，
希望你已经对 OORedis 是什么以及能做什么有了大概的感觉，
在接下来的部分，
文档会陆续介绍 OORedis 的其他 Key 类，
你将看到 OORedis 是怎样用简单快捷的方式来解决各种实际问题的。
