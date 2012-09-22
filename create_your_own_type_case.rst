创建新 Type Case 类
=========================

内置的 Type Case 类并不能满足所有需求，如果有需要，可以自己写一个新的 Type Case 类。

每个 Type Case 类都必须实现两个静态方法，一个是 ``encode`` 方法，它用于处理从 Python 到 Redis 的输入值。另一个是 ``decode`` ，用于处理从 Redis 到 Python 的输出值。

作为例子，以下是内置的 ``IntTypeCase`` 类的定义：

::

    # coding: utf-8

    class IntTypeCase:

        """ 
        处理 int(long) 类型值的转换。
        """

        @staticmethod
        def encode(value):
            """ 
            接受 int 类型值，否则抛出 TypeError 。 
            """
            try:
                return int(value)
            except ValueError:
                raise TypeError

        @staticmethod
        def decode(value):
            """ 
            尝试将值转回 int 类型，
            如果转换失败，抛出 TypeError。
            """
            if value is None:
                return

            try:
                return int(value)
            except ValueError:
                raise TypeError

定义 Type Case 有三点需要注意：

首先，为了保持抛出异常的一致性，在输入值的类型错误时， Type Case 抛出 ``TypeError`` 而不是 ``ValueError`` 。

其次，在 ``decode`` 方法里，必须处理 Redis 返回值为 ``None`` 的情况。

最后， ``encode`` 和 ``decode`` 必须是静态方法。
