内置 Type Case 类
=======================

OORedis 的 Type Case 类用于在取出和写入数据到 Redis 时，进行自动的类型转换。

OORedis 内置的 Type Case 类放在 ``ooredis.type_case`` 模块中，要设置某个 Key 类的 Type Case ，只要在创建 Key 类实例时，将所选 Type Case 类作为 ``type_case`` 参数传入 Key 类就可以了。

以下代码创建了一个只接受整数值的 ``Dict`` 实例：

::

    >>> from ooredis import *

    >>> from ooredis.type_case import IntTypeCase

    >>> number = Dict('phone-number', type_case=IntTypeCase)

    >>> number['china mobile'] = 10086

    >>> number['china telecom'] = 10000

    >>> number['china mobile']
    10086

    >>> number['china telecom']
    10000

试图将一个不符合类型要求的值传给 Key 对象，将会引发 ``TypeError`` 异常：

::

    >>> number['unknown'] = 'unknown'
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "ooredis/key/helper.py", line 24, in wrapper
        return func(*args, **kwargs)
      File "ooredis/key/dict.py", line 65, in __setitem__
        redis_value = self._encode(python_value)
      File "ooredis/type_case/int_type_case.py", line 17, in encode
        raise TypeError
    TypeError

除了 ``IntTypeCase`` 之外， OORedis 还内置了 ``GenericTypeCase`` 、 ``FloatTypeCase`` 、 ``StringTypeCase`` 、 ``JsonTypeCase`` 和 ``SerializeTypeCase`` 共六种 Type Case 类，以下部分将对它们进行一一说明。
    

GenericTypeCase
----------------------

``GenericTypeCase`` 接受 ``str`` 、 ``unicode`` 、 ``int`` 、 ``long`` 和 ``float`` 五种类型的值，并在从数据库取出数据时自动将它们转换回原来的类型。

::

    >>> from ooredis import String

    >>> from ooredis.type_case import GenericTypeCase

    >>> generic_str = String('value', type_case=GenericTypeCase)

    >>> generic_str.set("i'm a string")
    >>> generic_str.get()
    "i'm a string"

    >>> generic_str.set(10086)
    >>> generic_str.get()
    10086

    >>> generic_str.set(3.14)
    >>> generic_str.get()
    3.14

    >>> generic_str.set(u'你好')
    >>> generic_str.get()
    '\xe4\xbd\xa0\xe5\xa5\xbd'
    >>> print(generic_str.get())
    你好

因为 ``GenericTypeCase`` 是 OORedis 所有内置 Key 类的默认 Type Case 类，因此，上面的代码也可以简化成：

::

    >>> generic_str = String('value')


IntTypeCase
-------------

``IntTypeCase`` 接受任何可以转换为整数类型的值，包括 ``int`` 、 ``long`` 和 ``float`` ，并根据需要，返回 ``int`` 或者 ``long`` 类型的值。

::

    >>> from ooredis import String

    >>> from ooredis.type_case import IntTypeCase

    >>> int_value = String('int-value', type_case=IntTypeCase)

    >>> int_value.set(3.14)
    >>> int_value.get()
    3

    >>> int_value.set(10086)
    >>> int_value.get()
    10086

    >>> int_value.set(10000000000000000000000000000000000000000000)
    >>> int_value.get()
    10000000000000000000000000000000000000000000L


FloatTypeCase
----------------

``FloatTypeCase`` 接受任何可以转换为浮点类型的值，包括 ``int`` 、 ``long`` 和 ``float`` ，并返回 ``float`` 类型的值。

::

    >>> from ooredis import String

    >>> from ooredis.type_case import FloatTypeCase

    >>> float_value = String('float-value', type_case=FloatTypeCase)

    >>> float_value.set(3.14)
    >>> float_value.get()
    3.14

    >>> float_value.set(10086)
    >>> float_value.get()
    10086.0

    >>> float_value.set(10000000000000000000000000000000000000000000)
    >>> float_value.get()
    1e+43


StringTypeCase
-----------------

``StringTypeCase`` 接受 ``str`` 类型或者 ``unicode`` 类型的值，并返回 ``str`` 或者 ``unicode`` 类型的值。

::

    >>> from ooredis import String

    >>> from ooredis.type_case import StringTypeCase

    >>> s_value = String('s-value', type_case=StringTypeCase)

    >>> s_value.set('hello')
    >>> s_value.get()
    'hello'

    >>> s_value.set(u'你好')
    >>> s_value.get()
    '\xe4\xbd\xa0\xe5\xa5\xbd'
    >>> print(s_value.get())
    你好


JsonTypeCase
---------------

``JsonTypeCase`` 接受 JSON 对象，并返回 JSON 对象。

::

    >>> from ooredis import String
    >>> from ooredis.type_case import JsonTypeCase

    >>> json = String('json', type_case=JsonTypeCase)

    >>> json.set({'key': 'value'})
    >>> json.get()
    {u'key': u'value'}

    >>> json.set([1, 2, 3])
    >>> json.get()
    [1, 2, 3]


SerializeTypeCase
------------------

``SerializeTypeCase`` 接受任何可以被 ``pickle`` 模块序列化的对象，并返回一个经过 ``pickle`` 模块反序列化的对象：

::

    >>> from ooredis import String

    >>> from ooredis.type_case import SerializeTypeCase

    >>> obj = String('obj', type_case=SerializeTypeCase)

    >>> obj.set([1, 2, 3])
    >>> obj.get()
    [1, 2, 3]

    >>> obj.set({'key':'value'})
    >>> obj.get()
    {'key': 'value'}

    >>> class Foo:
    ...   def __init__(self, value):
    ...     self.value = value
    ... 

    >>> f = Foo(10086)
    >>> f.value
    10086

    >>> obj.set(f)

    >>> unserialize_f = obj.get()
    >>> unserialize_f.value
    10086

    >>> type(unserialize_f)
    <type 'instance'>

    >>> isinstance(unserialize_f, Foo)
    True


更多信息
-----------

以上就是 OORedis 的所有内置 Type Case 类了，要了解更多信息，可以参考 `Type Case 模块的 API <api/ooredis.type_case.html>`_ ，要编写新的 Type Case 类，请参考 `创建新 Type Case <create_your_own_type_case.html>`_ 小节。
