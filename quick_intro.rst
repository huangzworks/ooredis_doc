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


实例：用 Dict 类实现论坛帐号的登录和注册
-------------------------------------------

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

假设你正在构建一个论坛，
其中每个用户的登录信息可以使用
``name`` 和 ``password`` 两个属性来表示，
比如这样：

====== ====================
 name   password
====== ====================
peter    peters_password
jack     123456
mary     happy_girl
====== ====================

这些用户信息可以使用 ``Dict`` 类来保存，
``Dict`` 类的操作方式和 Python 内置的 ``dict`` 类对象几乎完全一样，
不同的是，
``Dict`` 类会将数据保存到 Redis 数据库中（而不仅仅是留在内存里）：

::

    >>> peter = Dict('peter')
    >>> peter['password'] = 'peters_password'

    >>> jack = Dict('jack')
    >>> jack['password'] = 123456

    >>> mary = Dict('mary')
    >>> mary['password'] = 'happy_girl'

执行以上语句之后，这三个用户的数据就被保存到了 Redis 里面了，
可以在 Redis 里面执行命令来确认这一点：

::

    redis> HGETALL  peter
    1) "password"
    2) "peters_password"

    redis> HGETALL jack
    1) "password"
    2) "123456"

    redis> HGETALL mary
    1) "password"
    2) "happy_girl"

现在，基于 ``Dict`` 类，可以给出相应的登录验证函数：

::

    >>> def login(name, password):
    ...     user = Dict(name)
    ...     if not user.exists:
    ...         print(u'用户不存在')
    ...     elif user['password'] == password:
    ...         print(u'登录成功')
    ...     else:
    ...         print(u'密码错误')
    ... 

``login`` 函数的逻辑非常简单，
唯一需要解释的应该就是 ``Dict.exists`` 方法了，
这个方法检查给定的 key 在 Redis 中是否存在（效果等同于 Redis 的 ``EXISTS`` 命令），
在这里，它用于检查是否有名字为 ``name`` 的用户，
如果在 Redis 里没有这个 key ，
那么说明这个帐号并不存在。

来试试这个 ``login`` 函数：

::

    >>> login('peter', 'peters_password')
    登录成功

    >>> login('peter', 'hacking_peter_account')
    密码错误

    >>> login('not-exists-user', 'password')
    用户不存在

另外，还可以将之间的注册功能抽象成 ``register`` 函数：

::

    >>> def register(name, password):
    ...     user = Dict(name)
    ...     if user.exists:
    ...         print(u'注册失败，用户名已经被占用')
    ...     else:
    ...         user['password'] = password
    ...         print(u'注册成功')
    ... 

然后删掉之前的几个帐号（ ``Dict.delete()`` 函数等于 Redis 的 ``DEL`` 命令）：

::

    >>> peter.delete()
    >>> jack.delete()
    >>> mary.delete()

再使用 ``register`` 重新进行注册：

::

    >>> register('peter', 'peters_password')
    注册成功

    >>> register('jack', 123456)
    注册成功

    >>> register('mary', 'happy_girl')
    注册成功

另外值得一提的是，如果还有别的人试图注册已经有的用户名，
那么 ``register`` 函数会提示错误：

::

    >>> register('peter', 'another_peters_password')
    注册失败，用户名已经被占用

以上就是使用 ``Dict`` 类实现的基本注册和登录功能了，
一个更实际的论坛可能会在 ``Dict`` 类中添加更多的域，
比如性别，邮件地址，年龄等等，但是它们的实现原理是一样的。


小结
-----

在这个快速入门小节中，我们看到了如何通过 ``connect`` 函数连接 Redis 服务器，
OORedis 各个 Key 类的大概作用，
以及怎样使用 ``Dict`` 实现论坛的登录和注册功能，
希望你已经能对 OORedis 是什么以及能做什么有了大概的感觉。

在后续的章节中，文档会继续对各个 Key 类进行详细的介绍。
