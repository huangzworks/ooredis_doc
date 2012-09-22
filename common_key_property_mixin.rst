CommonKeyPropertyMixin
=========================

对于所有 Redis 的 key 来说，都有一些通用的操作和属性，比如设置过期时间的 ``EXPIRE`` 和 ``PERSIST`` 命令，删除命令 ``DELETE`` ，检查 key 是否存在的 ``EXISTS`` 命令，等等。

``CommonKeyPropertyMixin`` 并不是一个具体的 Key 类，但是它对于 OORedis 中的所有 Key 类来说，都非常重要，因为它实现了上述的这些 Redis key 的通用操作和属性。

除了 ``Key`` 类以外， OORedis 内置的其他 Key 类都混入了这个 mixin 类（通过继承来模拟混入）。

关于 ``CommonKeyPropertyMixin`` 提供的操作和方法，请参考相应的 `API 文档 <api/ooredis.key.html#module-ooredis.key.common_key_property_mixin>`_ 。
