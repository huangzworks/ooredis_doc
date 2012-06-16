安装 OORedis
===============


软件需求
-------------

因为 OORedis 构建在一些别的软件之上，
在安装 OORedis 之前， 请确保你已经安装了以下这些软件：

- Python 2.7 或以上版本 ：
  `http://python.org/ <http://python.org/>`_

- Redis 2.4 或以上版本 ：
  `http://redis.io/download <http://redis.io/download>`_

- redis-py 2.4.13 或以上版本 ：
  `https://github.com/andymccurdy/redis-py
  <https://github.com/andymccurdy/redis-py>`_

- nosetest 1.1.2 或以上版本(用于测试) ： 
  `http://nose.readthedocs.org <http://nose.readthedocs.org>`_


安装 OORedis
-------------------

可以通过 ``pip`` 来安装 OORedis ：

::

    $ sudo pip install ooredis
    Downloading/unpacking ooredis
    Downloading ooredis-1.9.0.tar.gz
    Running setup.py egg_info for package ooredis
            
    Installing collected packages: ooredis
    Running setup.py install for ooredis
                  
    Successfully installed ooredis
    Cleaning up...

    $ python2
    Python 2.7.3 (default, Apr 24 2012, 00:06:13) 
    [GCC 4.7.0 20120414 (prerelease)] on linux2
    Type "help", "copyright", "credits" or "license" for more
    information.
    >>> import ooredis
    >>>

注意， 如果你的电脑上安装有多个版本的 Python ，
那么 ``pip`` 有可能被改名为 ``pip2`` 或者 ``pip-2.7``  。

另外， ``pip`` 的打印信息表示正在安装的是 ``1.9.0`` 版本的 OORedis ，
因为 OORedis 总是在不断更新中，
所以你安装的版本可能与这里显示的稍有不同，这是正常的，不必担心。
