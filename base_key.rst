BaseKey
========

在前面的《快速入门》章节里介绍过，
OORedis 将一些相互有关的 Redis 命令
（通常是同一个结构类型的命令）
组合成一个个 Key 类，
然后通过使用这些 Key 类来达到操作 Redis 的目的。

OORedis 的所有 Key 类有一个共同的基类（base class），
那就是 ``BaseKey`` 类：


::

              /  String 
             /   Counter
            /    Dict
    BaseKey      
            \    Deque
             \   Set
              \  SortedSet


使用 BaseKey
-------------------

``BaseKey`` 类只定义了两个方法， ``__init__`` 和 ``__eq__`` ：
前者用于保存输入的 key 名字、客户端和所使用的 TypeCase 类，
后者则用于对比两个 Key 对象之间是否相等。

因为 ``BaseKey`` 的主要作用是作为各个 Key 类的基类对象，单独使用它是没有意义的，所以它没有被直接暴露在命名空间里。

如果需要创建一个全新的 Key 类的话，可以从 ``ooredis.key.base_key`` 模块中载入它：

::

            
    >>> from ooredis.key.base_key import BaseKey
    >>> class NewKeyClass(BaseKey):
    ...   pass
    ... 


更多信息
----------

以上就是 ``BaseKey`` 类的基本介绍，
更详细的 API 信息可以参考《底层实现 API》章节的 `base_key
<./api/ooredis.key.html#module-ooredis.key.base_key>`_ 部分。
