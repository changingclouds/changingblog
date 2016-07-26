
###redis 面试题

#### 最适用什么场景

* 取最新N个数据的操作
* 排行榜应用，取TOP N操作
* 需要精准设定过期时间的应用
* 计数器应用
* Uniq操作，获取某段时间所有数据排重值
* 实时系统，反垃圾系统
* Pub/Sub构建实时消息系统
* 构建队列系统
* 缓存
* 用户投票
* 统计

#### 常用命令
keys *
save
CONFIG GET *

#### 常用的数据类型
* String
* Hash
* List
* Set
* Sorted set

#### flushdb 和flushall区别

* FLUSHALL删除所有现有的数据库，而不仅仅是当前选择的一个的键。此命令不会失败。
* flushdb  执行删除在某个db环境下执行的话，只删除当前db的数据

#### 使用redis有哪些好处？

* 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
* 支持丰富数据类型，支持string，list，set，sorted set，hash
* 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
* 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除


#### Memcache与Redis的区别都有哪些？
* 存储方式
Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。
Redis有部份存在硬盘上，这样能保证数据的持久性

* Memcache对数据类型支持相对简单。
    Redis有复杂的数据类型。

* 使用底层模型不同
它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。
Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。


#### redis常见性能问题和解决方案

* Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
* 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次
* 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
* 尽量避免在压力很大的主库上增加从库
* 主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1 <- Slave2 <- Slave3...这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变

#### mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。redis 提供 6种数据淘汰策略：
  
* volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
* volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
* volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
* allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
* allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
* no-enviction（驱逐）：禁止驱逐数据

#### 请用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次。具体登录函数或功能用空函数即可，不用详细写出。

* 列表中每个元素代表登陆时间,只要最后的第5次登陆时间和现在时间差不超过1小时就禁止登陆

#### Redis持久化的几种方式
* filesnapshotting
    默认redis是会以快照的形式将数据持久化到磁盘的（一个二进制文件，dump.rdb，这个文件名字可以指定），在配置文件中的格式是：save N M表示在N秒之内，redis至少发生M次修改则redis抓快照到磁盘。当然我们也可以手动执行save或者bgsave（异步）做快照。
工作原理简单介绍一下：当redis需要做持久化时，redis会fork一个子进程；子进程将数据写到磁盘上一个临时RDB文件中；当子进程完成写临时文件后，将原来的RDB替换掉，这样的好处就是可以copy-on-write

* Append-only
    filesnapshotting方法在redis异常死掉时，最近的数据会丢失（丢失数据的多少视你save策略的配置），所以这是它最大的缺点，当业务量很大时，丢失的数据是很多的。Append-only方法可以做到全部数据不丢失，但redis的性能就要差些。AOF就可以做到全程持久化，只需要在配置文件中开启（默认是no），appendonly yes开启AOF之后，redis每执行一个修改数据的命令，都会把它添加到aof文件中，当redis重启时，将会读取AOF文件进行“重放”以恢复到redis关闭前的最后时刻。

* AOF重写


#### Redis的并发竞争问题如何解决

* Redis为单进程单线程模式，采用队列模式将并发访问变为串行访问。Redis本身没有锁的概念，Redis对于多个客户端连接并不存在竞争，但是在Jedis客户端对Redis进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是由于客户端连接混乱造成。
1. 客户端角度，为保证每个客户端间正常有序与Redis进行通信，对连接进行池化，同时对客户端读写Redis操作采用内部锁synchronized。
2. 通过setnx 和 expire命令实现
3. 通过watch和Redis的事务命令实现

#### 了解Redis事务的CAS操作吗

* MULTI 命令用于开启一个事务，它总是返回 OK 。
* EXEC 命令



