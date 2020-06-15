# Redis 学习笔记

## Redis 是什么，为什么这么快

为什么这么快呢？

Redis 采用 ANSI C 语言编写，它和 SQLite 一样。采用 C 语言进行编写的好处是底层代码执行效率高，依赖性低，因为使用 C 语言开发的库没有太多运行时（Runtime）依赖，而且系统的兼容性好，稳定性高。

此外，Redis 是基于内存的数据库，我们之前讲到过，这样可以避免磁盘 I/O，因此 Redis 也被称为缓存工具。  

它采用单进程单线程模型，这样做的好处就是避免了上下文切换和不必要的线程之间引起的资源竞争。

Redis 全称是 Remote Dictionary Server，从名字中你也能看出来它用字典结构存储数据，也就是 key-value 类型的数据。

多路 I/O 复用技术。这里的多路指的是多个 socket 网络连接，复用指的是复用同一个线程。采用多路 I/O 复用技术的好处是可以在同一个线程中处理多个 I/O 请求，尽量减少网络 I/O 的消耗，提升使用效率。

## Redis 的数据类型

相比 Memcached，Redis 有一个非常大的优势，就是支持多种数据类型。Redis 支持的数据类型包括字符串(String)、哈希(Hash)、列表(List)、集合(Set)、有序集合(Sorted Set)等。

类型 | 简介 | 特性 | 场景
- | :-: | :-: | :-:
String(字符串) | 二进制安全 | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | ---
Hash(字典) | 键值对集合,即编程语言中的Map类型 | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) |  存储、读取、修改用户属性
List(列表) | 链表(双向链表) | 增删快,提供了操作某一段元素的API | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列
Set(集合) | 哈希表实现,元素不重复 | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐
Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列  | 数据插入集合时,已经进行天然有序 | 1、排行榜 2、带权重的消息队列

### Strings

- APPEND
- GET
- SET
- SETEX
    key 存在才 SET
- SETNX
    key 不存在才 SET

### Lists

只有 pop push 类操作有 Lxx Rxx，其他的操作只有 Lxx

- LPOP
- LPUSH
- RPOP
- RPUSH
- LINDEX
- LLEN
- LSET
- LINSERT
- LREM key count element
    删除一个值为element的元素，count = 0 时全删，count < 0 时从尾到头删count个element，count > 0 时从头到尾删count个element
- LRANGE key start stop
    查看一段
- LTRIM key start stop
    截取一段
- Bxx
    阻塞调用

### Hashes

可以放入哈希中的字段数没有实际限制（可用内存除外）

- HDEL
- HEXIST
- HGETALL
- HLEN
- HKEYS
- HGET
- HSET


### Sets

- SADD
- SREM
- SCARD
- SSCAN
- SDIFF
- SINTER
- SMEMBERS
- SISMEMBER

### Sorted sets

> If A and B are two elements with a different score, then A > B if A.score is > B.score.  
> If A and B have exactly the same score, then A > B if the A string is lexicographically greater than the B string. A and B strings can't be equal since sorted sets only have unique elements.

- ZADD
- ZRANGE
    按名次列出成员
- ZRANGEBYSCORE
    按分数列出成员
- ZREMRANGEBYSCORE
    按分数删除成员
- ZCARD
    成员数量
- ZRANK
    获取成员的名次
- ZCOUNT
    分数在某个区间的 member 数量
- ZSCORE
    获取成员分数
- ZRANGEBYLEX, ZREVRANGEBYLEX, ZREMRANGEBYLEX, ZLEXCOUNT
    score 一样时按字典排序
- ZRExxx
    倒序
