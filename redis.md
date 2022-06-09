#### 命令

- 字符串

  | 命令                           | 说明                             | 复杂度                     |
  | ------------------------------ | -------------------------------- | -------------------------- |
  | set key value                  | 设置键值                         | O(1)                       |
  | get key                        | 获取指定键的值                   | O(1)                       |
  | mset key value [key value ...] | 批量设置键值对                   | O(n), n是键的个数          |
  | mget key [key ...]             | 批量获取给定键的值               | O(n), n是键的个数          |
  | incr key                       | 对指定键的值增1                  | O(1)                       |
  | decr key                       | 对指定键的值减1                  | O(1)                       |
  | incrby key increment           | 对指定键的值自增指定数字         | O(1)                       |
  | decrby key decrement           | 对指定键的值自减指定数字         | O(1)                       |
  | incrbyfloat key increment      | 对指定键的值自增指定浮点数       | O(1)                       |
  | append key value               | 对指定键的值追加值               | O(1)                       |
  | strlen key                     | 获取值的长度                     | O(1)                       |
  | setrange key offset value      | 从指定偏移量开始，覆盖部分字符串 | O(1)，当字符串较大时，O(n) |
  | getrange key start end         | 获取指定区间的字符串             | O(1)，当字符串较大时，O(n) |

- 哈希

  | 命令                                | 说明                    | 复杂度 |
  | ----------------------------------- | ----------------------- | ------ |
  | hset key field value                | 设置字段值              | O(1)   |
  | hget key field                      | 获取字段值              | O(1)   |
  | hdel key field [field ...]          | 批量删除指定字段        | O(n)   |
  | hlen key                            | 获取字段个数            | O(1)   |
  | hgetall key                         | 获取所有field-value     | O(1)   |
  | hmget field [field ...]             | 批量获取给定字段的值    | O(n)   |
  | hmset field value [field value ...] | 批量设置field-value     | O(n)   |
  | hexists key field                   | 判断指定字段是否存在    | O(1)   |
  | hkeys key                           | 获取所有字段            | O(n)   |
  | hvals key                           | 获取所有value           | O(n)   |
  | hsetnx key field value              | 设置field-value         | O(1)   |
  | hincrby key field increment         | field自增               | O(1)   |
  | hincrbyfloat key field increment    | field自增浮点数         | O(1)   |
  | hstrlen key field                   | 获取field值的字符串长度 | O(1)   |

- 列表

  | 命令                                  | 说明                                         | 复杂度 |
  | ------------------------------------- | -------------------------------------------- | ------ |
  | rpush key value [value ...]           | 从列表右端添加元素                           | O(n)   |
  | lpush key value [value ...]           | 从列表左端添加元素                           | O(n)   |
  | linsert key before\|after pivot value | 在给定元素的前面\|后面插入新的元素           | O(n)   |
  | lrange key start end                  | 获取指定区间的元素                           | O(n)   |
  | lindex key index                      | 获取指定索引处的元素                         | O(n)   |
  | llen key                              | 获取列表长度                                 | O(1)   |
  | lpop key                              | 删除并返回列表左端元素                       | O(1)   |
  | rpop key                              | 删除并返回列表右端的元素                     | O(1)   |
  | lrem key count value                  | 删除指定数量的value值，方向取决于count的正负 | O(n)   |
  | ltrim key start end                   | 删除指定区间的元素                           | O(n)   |
  | lset key index value                  | 修改指定索引处元素的值                       | O(n)   |
  | blpop brpop                           | 阻塞地删除元素                               | O(1)   |

- 集合

  | 命令                           | 说明                   | 复杂度   |
  | ------------------------------ | ---------------------- | -------- |
  | sadd key element [element ...] | 添加                   | O(e)     |
  | srem key element [element ...] | 删除                   | O(e)     |
  | scard key                      | 集合元素的个数         | O(1)     |
  | sismember key element          | 是否存在给定元素       | O(1)     |
  | srandmember key [count]        | 随机返回指定数量的元素 | O(count) |
  | spop key                       | 随机获取一个元素       | O(1)     |
  | smembers key                   | 获取所有元素           | O(e)     |
  | sinter key [key ...]           | 交集                   |          |
  | sunion key [key ...]           | 并集                   |          |
  | sdiff key [key ...]            | 差集                   |          |

- 有序集合

  | 命令                                                     | 说明                                 | 复杂度 |
  | -------------------------------------------------------- | ------------------------------------ | ------ |
  | zadd key score member [score member ...]                 | 添加给定分数的元素                   |        |
  | zcard key                                                | 元素个数                             | O(1)   |
  | zscore key member                                        | 获取指定元素的分数                   | O(1)   |
  | zrank\|zrevrank key member                               | 获取指定元素按照分数升序\|降序的排名 |        |
  | zrem key member [member ...]                             | 删除元素                             |        |
  | zincrby key increment member                             | 增加指定元素的分数                   |        |
  | zrange\|zrevrange key start end [withscores]             | 获取指定排名范围内的成员             |        |
  | zrangebyscore\|zrevrangebyscore key max min [withsocres] | 获取指定分数范围的成员               |        |
  | zcount key min max                                       | 获取指定分数范围内成员的个数         |        |
  | zremrangebyrank key start end                            | 删除指定排名范围内的成员             |        |
  | zremrangebyscore key min max                             | 删除指定分数范围内的成员             |        |
  | zinter key [key ...]                                     | 交集                                 |        |
  | zunion key [key ...]                                     | 并集                                 |        |

- bitmap

  | 命令                                  | 说明                                                         | 复杂度 |
  | ------------------------------------- | ------------------------------------------------------------ | ------ |
  | setbit key offset value               | 设置键的第offset（从0开始）个位的值                          |        |
  | getbit key offset                     | 获取键的第offset位的值                                       |        |
  | bitcount key [start end]              | 获取键的指定范围内1的个数，start end为字节                   |        |
  | bitop operation destkey key [key ...] | 对多个键进行交集and，并集or，非not，异或xor操作，<br/>并把结果保存到destkey中 |        |
  | bitpos key bit [start end]            | 返回指定字节范围内第一个为指定bit位的偏移量                  |        |
  
  
  
- 其他

  | 命令                                | 说明                                                         | 复杂度                    |
  | ----------------------------------- | ------------------------------------------------------------ | ------------------------- |
  | keys pattern                        | 查看所有键                                                   | O(n)                      |
  | dbsize                              | 查询键总数                                                   | O(1)                      |
  | exists key [key ...]                | 检查键是否存在                                               | O(n) n是要检查的key的数量 |
  | del key [key ...]                   | 删除键                                                       | O(n) 是要删除的key的数量  |
  | expire key seconds [NX\|XX\|GT\|LT] | 对键设置过期<br />NX-- 只有在键不存在才设置过期<br />XX-- 只有在键存在时才设置过期<br />GT-- 仅当新的过期时间大于当前过期时间才设置过期时间<br />LT- 仅当新的过期时间小于当前过期时间才设置过期时间 | O(1)                      |
  | ttl key                             | 返回键剩余的过期时间<br />返回结果：<br />>=0：键剩余的过期时间<br />-1：没有设置过期时间<br />-2：键不存在 | O(1)                      |
  | type key                            | 返回键的类型                                                 | O(1)                      |
  | object encoding key                 | 返回内部编码                                                 | O(1)                      |

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

redisson:

# [redisson-3.12.5](https://github.com/redisson/redisson/releases/tag/redisson-3.12.5)

改进 - 提高`RLock`故障转移期间的可靠性。`RedLock`已弃用

 
