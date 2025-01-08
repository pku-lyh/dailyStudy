## Redis常见面试题
### Redis 和 Memcached 有什么区别？
**共同点：**

1. <font style="color:rgb(44, 62, 80);">都是基于内存的数据库，一般都用来当做缓存使用。</font>
2. <font style="color:rgb(44, 62, 80);">都有过期策略。</font>
3. <font style="color:rgb(44, 62, 80);">两者的性能都非常高。</font>

**<font style="color:rgb(44, 62, 80);">区别：</font>**

1. <font style="color:rgb(44, 62, 80);">Redis 支持的数据类型更丰富（String、Hash、List、Set、ZSet），而 Memcached 只支持最简单的 key-value 数据类型；</font>
2. <font style="color:rgb(44, 62, 80);">Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而 Memcached 没有持久化功能，数据全部存在内存之中，Memcached 重启或者挂掉后，数据就没了；</font>
3. <font style="color:rgb(44, 62, 80);">Redis 原生支持集群模式，Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；</font>
4. <font style="color:rgb(44, 62, 80);">Redis 支持发布订阅模型、Lua 脚本、事务等功能，而 Memcached 不支持；</font>

### 为什么用 Redis 作为 MySQL 的缓存？
1. Redis是内存数据库，访问速度非常快（高性能）
2. Redis的单机QPS是MySql的十倍，可以轻松突破10w（高并发）

### Redis 常用的数据类型有哪些？
Redis 中比较常见的数据类型有下面这些：

+ **5 种基础数据类型**：String（字符串）、List（列表）、Set（集合）、Hash（散列）、Zset（有序集合）。
+ **4 种特殊数据类型**：HyperLogLog（基数统计）、Bitmap（位图）、Geospatial (地理位置)、Stream（流）。

**常用场景：**

1. <font style="color:rgb(44, 62, 80);">String 类型的应用场景：缓存对象、常规计数、分布式锁、共享 session 信息等。</font>
2. <font style="color:rgb(44, 62, 80);">List 类型的应用场景：消息队列（但是有两个问题：1. 生产者需要自行实现全局唯一 ID；2. 不能以消费组形式消费数据）等。</font>
3. <font style="color:rgb(44, 62, 80);">Hash 类型：缓存对象、购物车等。</font>
4. <font style="color:rgb(44, 62, 80);">Set 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动等。</font>
5. <font style="color:rgb(44, 62, 80);">Zset 类型：排序场景，比如排行榜、电话和姓名排序等。</font>
6. <font style="color:rgb(44, 62, 80);">BitMap（2.2 版新增）：二值状态统计的场景，比如签到、判断用户登陆状态、连续签到用户总数等；</font>
7. <font style="color:rgb(44, 62, 80);">HyperLogLog（2.8 版新增）：海量数据基数统计的场景，比如百万级网页 UV 计数等；</font>
8. <font style="color:rgb(44, 62, 80);">GEO（3.2 版新增）：存储地理位置信息的场景，比如滴滴叫车；</font>
9. <font style="color:rgb(44, 62, 80);">Stream（5.0 版新增）：消息队列，相比于基于 List 类型实现的消息队列，有这两个特有的特性：自动生成全局唯一消息ID，支持以消费组形式消费数据。</font><font style="color:rgb(44, 62, 80);">  
</font><font style="color:rgb(44, 62, 80);">五种常见的 Redis 数据类型是怎么实现？</font>

### 五种常见的Redis数据结构是怎么实现
#### String 类型内部实现（SDS）
+ `<font style="color:rgb(60, 60, 67);">len</font>`<font style="color:rgb(60, 60, 67);">：字符串的长度也就是已经使用的字节数</font>
+ `<font style="color:rgb(60, 60, 67);">alloc</font>`<font style="color:rgb(60, 60, 67);">：总共可用的字符空间大小，alloc-len 就是 SDS 剩余的空间大小</font>
+ `<font style="color:rgb(60, 60, 67);">buf[]</font>`<font style="color:rgb(60, 60, 67);">：实际存储字符串的数组</font>
+ `<font style="color:rgb(60, 60, 67);">flags</font>`<font style="color:rgb(60, 60, 67);">：低三位保存类型标志</font>

**SDS 相比于 C 语言中的字符串有如下提升：**

1. **可以避免缓冲区溢出**<font style="color:rgb(60, 60, 67);">：C 语言中的字符串被修改（比如拼接）时，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。SDS 被修改时，会先根据 len 属性检查空间大小是否满足要求，如果不满足，则先扩展至所需大小再进行修改操作。</font>
2. **获取字符串长度的复杂度较低**<font style="color:rgb(60, 60, 67);">：C 语言中的字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。SDS 的长度获取直接读取 len 属性即可，时间复杂度为 O(1)。</font>
3. **减少内存分配次数**<font style="color:rgb(60, 60, 67);">：为了避免修改（增加/减少）字符串时，每次都需要重新分配内存（C 语言的字符串是这样的），SDS 实现了空间预分配和惰性空间释放两种优化策略。当 SDS 需要增加字符串时，Redis 会为 SDS 分配好内存，并且根据特定的算法分配多余的内存，这样可以减少连续执行字符串增长操作所需的内存重分配次数。当 SDS 需要减少字符串时，这部分内存不会立即被回收，会被记录下来，等待后续使用（支持手动释放，有对应的 API）。</font>
4. **二进制安全**<font style="color:rgb(60, 60, 67);">：C 语言中的字符串以空字符 </font>`<font style="color:rgb(60, 60, 67);">\0</font>`<font style="color:rgb(60, 60, 67);"> 作为字符串结束的标识，这存在一些问题，像一些二进制文件（比如图片、视频、音频）就可能包括空字符，C 字符串无法正确保存。SDS 使用 len 属性判断字符串是否结束，不存在这个问题。</font>

#### <font style="color:rgb(60, 60, 67);">List类型内部实现</font>
List 类型的底层数据结构是由**双向链表或压缩列表**实现的：

+ 如果列表的元素个数小于 512 个（默认值，可由 list-max-ziplist-entries 配置），列表每个元素的值都小于 64 字节（默认值，可由 list-max-ziplist-value 配置），Redis 会使用**压缩列表**作为 List 类型的底层数据结构；
+ 如果列表的元素不满足上面的条件，Redis 会使用**双向链表**作为 List 类型的底层数据结构；

<font style="color:rgb(44, 62, 80);">但</font>是在 Redis 3.2 版本之后，List 数据类型底层数据结构就只由**quicklist** 实现了，替代了双向链表和压缩列表。

#### Hash类型内部实现
Hash 类型的底层数据结构是由**压缩列表或哈希表**实现的：

+ 如果哈希类型的元素个数小于 512 个（默认值，可由 hash-max-ziplist-entries 配置），列表每个元素的值都小于 64 字节（默认值，可由 hash-max-ziplist-value 配置），Redis 会使用**压缩列表**作为 Hash类型的底层数据结构；
+ 如果哈希类型元素不满足上面条件，Redis 会使用**哈希表**作为 Hash 类型的底层数据结构。

在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 **listpack** 数据结构来实现了。

#### Set类型内部实现
Set 类型的底层数据结构是由**哈希表或整数集合**实现的：

+ 如果集合中的元素都是整数且元素个数小于 512 （默认值，set-maxintset-entries配置）个，Redis 会使用**整数集合**作为 Set 类型的底层数据结构；
+ 如果集合中的元素不满足上面条件，则 Redis 使用**哈希表**作为 Set 类型的底层数据结构。

ZSet类型内部实现



