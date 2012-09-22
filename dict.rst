Dict
=========

``Dict`` 将 Redis 的 Hash 结构映射到一个字典对象上，
这个对象和 Python 内置的 ``dict`` 类实例操作完全一样。

以下是 ``Dict`` 的操作示例：

::

    >>> from ooredis import Dict
    >>>
    >>> project = Dict('ooredis-project-info')
    >>>
    >>> project['name'] = 'OORedis'
    >>> project['language'] = 'Python'
    >>> project['description'] = 'A Python-to-Redis Mapper'
    >>>
    >>> project['name']
    'OORedis'
    >>>
    >>> project.items()
    [('name', 'OORedis'), ('language', 'Python'), ('description', 'A Python-to-Redis Mapper')]
    >>>
    >>> project.pop('language')
    'Python'
    >>>
    >>> for key in project:
    ...     print(key)
    ... 
    name
    description
    language
    >>>
    >>> dir(project)
    ['_MutableMapping__marker', '__abstractmethods__', '__class__', '__contains__', '__delattr__',
    '__delitem__', '__dict__', '__doc__', '__eq__', '__format__', '__getattribute__', '__getitem__',
    '__hash__', '__init__', '__iter__', '__len__', '__metaclass__', '__module__', '__ne__', '__new__',
    '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__',
    '__subclasshook__', '__weakref__', '_abc_cache', '_abc_negative_cache', '_abc_negative_cache_version',
    '_abc_registry', '_client', '_decode', '_encode', '_represent', 'clear', 'delete', 'exists', 'expire',
    'expireat', 'get', 'items', 'iteritems', 'iterkeys', 'itervalues', 'keys', 'name', 'persist', 
    'pop', 'popitem', 'setdefault', 'ttl', 'update', 'values']

``Dict`` 类的使用实例可以参考\ `快速入门 <quick_intro.html>`_\ 部分，那里有一个用 ``Dict`` 类实例表示论坛数据的例子。

更多信息
------------

以上就是 ``Dict`` 类的基本介绍， 更详细的 API 信息可以参考《底层实现 API》章节的 `Dict 部分 <api/ooredis.key.html#module-ooredis.key.dict>`_ 。
