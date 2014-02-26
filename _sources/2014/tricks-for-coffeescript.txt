=======================
Tricks for CoffeeScript
=======================

.. sectionauthor:: momo <mobeiheart@gmail.com>

.. feed-entry::
   :date: 2014-02-26

1.字符串反转::

    str = "hello"
    (s for s in str by -1).join('')

2.遍历Object，注意是 `of` ，习惯容易写错成 `in` ::

    "#{key} = #{value}" for key, value of {'a': 'b', 'c': 'd'}


3.除后取整，注意 `//` 的写法在 1.7.0 版本才添加::

    (new Date(2014, 1, 2).getTime() - new Date(2014, 1, 1).getTime()) // 1000
    Math.floor((new Date(2014, 1, 2).getTime() - new Date(2014, 1, 1).getTime() / 1000)
