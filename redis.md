#### 数据结构

##### 字符串（string）

​		C语言中，没有string类型，故使用数组或指针来实现。

##### 列表（list）

​		3.2版本之前，使用ziplist和linkedlist实现。后续版本，采用quicklist实现。

###### ziplist

​		当同时满足以下条件时，采用ziplist：

 - 字符串元素大小都小于64byte.

 - 列表元素个数小于512.

    当不满足任一条件时，改为linkedlist实现。

ziplist数据结构：由表头和N个entry节点和压缩列表尾部标识符zlend组成的一个连续的内存块。然后通过一系列的编码规则，提高内存的利用率，主要用于存储整数和比较短的字符串。

linkedlist数据结构：和普通链表相同。

quicklist数据结构：整体上是一个双向链表，但组成链表的entry是一个ziplist。

##### 哈希（hash）

实现hash的方式有两种：一种是ziplist，一种是hashtable。

当同时满足以下条件时才使用ziplist：

- 键个数小于512
- 值的大小都小于64byte

##### 集合（set）

采用hashtable来存储。

##### 有序集合（zset）

有两种方式：一种是ziplist。一种是skiplist与dict的结合。