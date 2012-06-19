BaseKey
========

在前面的《快速入门》章节里介绍过，
OORedis 将一些相互有关的 Redis 命令
（通常是同一个结构类型的命令）
组合成一个个 Key 类，
然后通过使用这些 Key 类来达到操作 Redis 的目的。

OORedis 的所有 Key 类有一个共同的基类（base class），
那就是 ``BaseKey`` 类。

``BaseKey`` 的主要作用是为其他每个 Key 类提供一些必要的属性和方法：

比如 ``BaseKey.name`` 用于记录 Redis key 的名字（key name），
``BaseKey._type_case`` 用于记录 Key 类所使用的 TypeCase 类，
还有 ``BaseKey.expire`` 方法对应 Redis 的 ``EXPIRE`` 命令、
``BaseKey.ttl`` 属性对应 Redis 的 ``TTL`` 命令、
``BaseKey.delete`` 方法对应 Redis 的 ``DEL`` 命令，
等等。

``BaseKey`` 包裹起所有的这些属性和方法，
使得它可以被用作构建其他 Key 类的起点，
以下是 OORedis 中类型的层次结构：

::

              /  String  -- Counter
             /   Dict
    BaseKey      Deque
             \   Set
              \  SortedSet


使用 BaseKey 类
--------------------

因为 ``BaseKey`` 本身只是为了提供公用方法而存在的，
因此它没有直接暴露在 ``ooredis`` 命名空间之下。


当你需要创建一个新的 OORedis 类的时候，
在 ``ooredis.key.base_key`` 可以找它：

::

    >>> from ooredis.key.base_key import BaseKey
    >>> BaseKey('say-hello-to-base-key')
    <ooredis.key.base_key.BaseKey object at 0xb731952c>

虽然一般不直接使用 ``BaseKey`` 类，但是我们在这里测试一下它也无妨：

::

    >>> k = BaseKey('a-base-key')

    >>> dir(k)
    ['__class__', '__delattr__', '__dict__', '__doc__', '__eq__', '__format__',
    '__getattribute__', '__hash__', '__init__', '__module__', '__new__',
    '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
    '__str__', '__subclasshook__', '__weakref__', '_client', '_represent',
    '_type_case', 'delete', 'exists', 'expire', 'expireat', 'name', 'persist',
    'ttl']

    >>> k.name          # 查看 key 名
    'a-base-key'

    >>> k._type_case    # 查看所使用的 TypeCase 类
    <class ooredis.type_case.generic_type_case.GenericTypeCase at 0xb73294dc>

    >>> k.exists        # 检查 key 是否存在
    False

    >>> k.ttl           # 查看 key 的生存时间，
    >>>                 # 因为 key 不存在，所以返回 None


更多信息
----------

以上就是 ``BaseKey`` 类的基本介绍，
更详细的 API 信息可以参考《底层实现 API》章节的 `base_key
<./api/ooredis.key.html#module-ooredis.key.base_key>`_ 部分。
