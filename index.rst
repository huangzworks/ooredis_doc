.. OORedis 项目文档 documentation master file, created by
   sphinx-quickstart on Sat Jun 16 10:29:30 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

OORedis 文档
===================

OORedis 是一个用 Python 编写的 Redis 映射器，它将 Redis 的各种数据结构映射到 Python 的数据结构上，
使得对 Redis 的操作更简单、更 Pythonic 。

除此之外，内置的 Type Case 类型转换机制也为写入和读取数据到 Redis 提供了方便。

这个文档由浅入深地介绍了 OORedis 的使用方法，并提供了详细的 API 可供查询。

简介
---------------

.. toctree::
   :maxdepth: 2

   install
   quick_intro


Key 类详解
-------------------

.. toctree::
   :maxdepth: 2

   base_key
   string
   counter
   dict
   deque
   set
   sorted_set
   common_key_property_mixin


Type Case 类
------------------

.. toctree::
   :maxdepth: 2

   build_in_type_case
   create_your_own_type_case


底层实现 API
---------------

.. toctree::
   :maxdepth: 2
   
   api/modules
   api/ooredis.key.rst
   api/ooredis
   api/ooredis.type_case


相关链接
-----------

OORedis 项目地址： `https://github.com/huangz1990/ooredis <https://github.com/huangz1990/ooredis>`_

本文当项目地址： `https://github.com/huangz1990/ooredis_doc <https://github.com/huangz1990/ooredis_doc>`_ 


索引和目录
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
