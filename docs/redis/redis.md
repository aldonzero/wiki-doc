# 

## NoSQL

### 概述

NoSQL（Not-Only SQL）：泛指非关系型的数据库，作为关系型数据库的补充

MySQL 支持 ACID 特性，保证可靠性和持久性，读取性能不高，因此需要缓存的来减缓数据库的访问压力

作用：应对基于海量用户和海量数据前提下的数据处理问题

特征：

* 可扩容，可伸缩，SQL 数据关系过于复杂，Nosql 不存关系，只存数据
* 大数据量下高性能，数据不存取在磁盘 IO，存取在内存
* 灵活的数据模型，设计了一些数据存储格式，能保证效率上的提高
* 高可用，集群

常见的 NoSQL：Redis、memcache、HBase、MongoDB

***



### Redis

Redis (REmote DIctionary Server) ：用 C 语言开发的一个开源的高性能键值对（key-value）数据库

特征：

* 数据间没有必然的关联关系，**不存关系，只存数据**
* 数据**存储在内存**，存取速度快，解决了磁盘 IO 速度慢的问题
* 内部采用**单线程**机制进行工作
* 高性能，官方测试数据，50 个并发执行 100000 个请求，读的速度是 110000 次/s，写的速度是 81000 次/s
* 多数据类型支持
  * 字符串类型：string（String）：数据库分表，建立唯一主键ID
  * 列表类型：list（LinkedList）
  * 散列类型：hash（HashMap）：value只能放字符串，最多$2^{32} - 1$对
  * 集合类型：set（HashSet）
  * 有序集合类型：zset/sorted_set（TreeSet）
* 支持持久化，可以进行数据灾难恢复

#### 业务类型
 **原始业务功能** 
- 作为缓存使用
    - 秒杀业务的商品（618、双11）
    - 排队抢票
- 运营平台检测到突发高频访问数据
    - 突发时政要闻，微博热榜
- 高频复杂的统计数据
    - 在线人数，投票计数

 **附加功能** 
- 系统功能升级或优化
     - 但服务升级集群
    - session管理
    - token管理

***

## 持久化

### RDB

在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot），生成备份文件（dump.rdb）。有自动触发和手动触发两种。

**持久化命令**

- save：只会执行`save`命令，会阻塞主进程，导致redis在`fork`执行期间其他命令不能执行。
- bgsave（默认）：Redis后台异步快照操作，不会影响客户端的请求。

**关于fork**

> Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。子进程和父进程会共用一段物理内存。

持久化流程：

- redis执行`bgsave`持久化命令后，会创建一个`fork`子进程（一个后台进程，可以和主进程同时运行）（ps：fork到底是进程还是线程？）
- `fork`和主进程共享数据，直接获取数据页将数据写入rdb临时文件
- 当主进程要对数据进行写操作，会将待数据页复制一份供`fork`进程读取数据，主进程仍然进行写操纵。
- 当主进程是读取数据，`fork`进程会直接读取数据。

![bgSave流程](https://foruda.gitee.com/images/1678248378066521978/ecd4d8df_8616658.png)

#### 持久化配置

```bash
save 秒 至少触发多少次修改
```

**化默认配置**

**同步频率机制**



```
//6.0.12持久                              
15min   1次
10min	10次
1min	10000次
```

**7之后持久化默认配置**

```bash
1h 	1次
5min 100次
1min 10000次
```

#### RDB优点：

- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高更适合使用
- 节省磁盘空间
- 恢复速度快

#### RDB缺点：

- RDB在间隔期间备份数据，如果在达到间隔期间条件Redis意外dowm掉，最后一次快照备份数据可能会丢失一部分。
- 如果数据集很大，fork（） 可能会很耗时，如果数据集非常大且 CPU 性能不是很好，则可能会导致 Redis 停止为客户端提供服务几毫秒甚至一秒钟。

### AOF

以日志的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件。并在服务器启动时，通过重新执行这些命令来还原数据集。

**AOF同步机制**

```bas
appendfsync always //每次写入都会立刻记入日志；性能比较差但是数据完整性比较好
appendfsync everysec  //每秒同步一次，如果宕机，一秒间隔内写入的数据可能会丢失
appendfsync no  //将同步时机交给操作系统
```

**Rewite压缩机制**

AOF采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令`bgrewriteaof`

**重写机制**

为解决AOF文件越来越大问题，会重建一个当前AOF文件的体积优化版本。即使`bgrewriteaod`执行失败，也不会有任何数据丢失，因为旧的AOF文件在`bgrewriteaof`成功之前不会被修改。

AOF重写会由Rdis自动触发（当前文件是上次文件大小的2倍，且大于64M），`bgrewriteaof`仅用于手动触发重写操作。

**重写流程**

- 触发重写，判断是否当前有bgsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行
- 主进程fork出子进程执行重写操作，保证主进程不会阻塞
- 子进程遍历redis内存中数据到临时文件，客户端的写请求同时写入aof_buf缓冲区和aof_rewrite_buf重写缓冲区保证原AOF文件完整以及新AOF文件生成期间的新的数据修改动作不会丢失。
- 1).子进程写完新的AOF文件后，向主进程发信号，父进程更新统计信息。2).主进程把aof_rewrite_buf中的数据写入到新的AOF文件。
- 使用新的AOF文件覆盖旧的AOF文件，完成AOF重写

![aof重写流程](https://foruda.gitee.com/images/1678258446218040143/f58fe3a1_8616658.png "屏幕截图")



AOF优点：

- 备份机制更稳健，丢失数据概率更低

AOF缺点：

- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。

默认是不开启AOF，开启RDB

两种方式同时开启

## Redis主从

单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离。

![输入图片说明](https://foruda.gitee.com/images/1678261296829843713/b59f54f8_8616658.png "屏幕截图")

### 主从同步原理

#### 全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程：

![输入图片说明](https://foruda.gitee.com/images/1678261437831226624/dbe75a1d_8616658.png "屏幕截图")

master如何得知salve是第一次来连接呢？

判断依据：

- **Replication Id**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid
- **offset**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。

因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据。



因为slave原本也是一个master，有自己的replid和offset，当第一次变成slave，与master建立连接时，发送的replid和offset是自己的replid和offset。

master判断发现slave发送来的replid与自己的不一致，说明这是一个全新的slave，就知道要做全量同步了。

master会将自己的replid和offset都发送给这个slave，slave保存这些信息。以后slave的replid就与master一致了。

因此，**master判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

![输入图片说明](https://foruda.gitee.com/images/1678261775340226039/0ecd0546_8616658.png "屏幕截图")

完整流程描述：

- slave节点请求增量同步
- master节点判断replid，发现不一致，拒绝增量同步
- master将完整内存数据生成RDB，发送RDB到slave
- slave清空本地数据，加载master的RDB
- master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
- slave执行接收到的命令，保持与master之间的同步

#### 增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输个slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。

![输入图片说明](https://foruda.gitee.com/images/1678262185509338654/45d765bc_8616658.png "屏幕截图")



那么master怎么知道slave与自己的数据差异在哪里呢?

### repl_backlog原理

repl_baklog文件，这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。

repl_baklog中会记录Redis处理过的命令日志及offset，包括master当前的offset，和slave已经拷贝到的offset，slave与master的offset之间的差异，就是salve需要增量拷贝的数据了。

- 不断有数据写入，master的offset逐渐变大，slave也不断的拷贝，追赶master的offset，直到数组被正常填满

- 如果有新的数据写入，就会覆盖数组中的旧数据，绿色是slave已经同步的数据，覆盖也没有什么影响。

- 如果slave出现网络阻塞，导致master的offset远远超过了slave的offset

- 如果master继续写入新数据，其offset就会覆盖旧的数据，直到将slave现在的offset也覆盖

- 棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却发现自己的offset都没有了，无法完成增量同步了。只能做全量同步。

  ​

![输入图片说明](https://foruda.gitee.com/images/1678263165965238439/8d8319ba_8616658.png "屏幕截图")

> **注意：**repl_backlog大小有上限，写满后会覆盖最早的数据。如果slave断开时间过久，导致尚未备份的数据被覆盖，则无法基于log做增量同步，只能在此全量同步。

### 主从同步优化

主从同步可以保证主从数据的一致性，非常重要。

可以从以下几个方面来优化Redis主从就集群：

- 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO。
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用主-从-从链式结构，减少master压力

主从从架构图：

![输入图片说明](https://foruda.gitee.com/images/1678263841456229983/4ae13c51_8616658.png "屏幕截图")



### 小结

简述全量同步和增量同步区别？

- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave

什么时候执行全量同步？

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时

什么时候执行增量同步？

- slave节点断开又恢复，并且在repl_baklog中能找到offset时


##  Redis哨兵



### 概述

***

Sentinel（哨兵）是 Redis 的高可用性（high availability）解决方案，由一个或多个 Sentinel 实例 instance 组成的 Sentinel 系统可以监视任意多个主服务器，以及这些主服务器的所有从服务器，并在被监视的主服务器下线时进行故障转移。(通俗理解，哨兵就是一个监控系统)

### 哨兵的作用

***

监控 master 和 slave的状态，当被监控服务器出现问题，向其他哨兵发送通知，当master出现故障时，选取一个slave作为master。

- 监控：监控 master 和 slave，不断的检查 master 和 slave 是否正常运行，master 存活检测、master 与slave 运行情况检测
- 通知：当被监控的服务器出现问题时，向其他哨兵发送通知
- 自动故障转移：断开 master 与 slave 连接，选取一个 slave 作为 master，将其他 slave 连接新的 master，并告知客户端新的服务器地址

<img src="https://foruda.gitee.com/images/1678265058759869745/3af4bfc8_8616658.png" style="zoom:60%;" />

### 哨兵启用流程

***

1. 初始化服务器
2. 将普通 Redis 服务器使用的代码替换成 Sentinel 专用代码
3. 初始化 Sentinel 状态
4. 根据给定的配置文件，初始化 Sentinel 的监视主服务器列表
5. 创建连向主服务器的网络连接

#### 1、初始化

Sentinel 本质上只是一个运行在特殊模式下的 Redis 服务器，当一个 Sentinel 启动时，首先初始化 Redis 服务器，但是初始化过程和普通 Redis 服务器的初始化过程并不完全相同，哨兵不提供数据相关服务，所以不会载入RDB、AOF 文件

#### 2、代码替换

将一部分普通 Redis 服务器使用的代码替换成 Sentinel 专用代码

#### 3、初始化 Sentinel 状态

服务器会初始化一个 sentinelState 结构，又叫 Sentinel 状态，结构保存了服务器中所有和 Sentinel 功能有关的状态（服务器的一般状态仍然由 redisServer 结构保存）

#### 4、监控列表
Sentinel 状态的初始化将 masters 字典的初始化，根据被载入的 Sentinel 配置文件 conf 来进行属性赋值

Sentinel 状态中的 masters 字典记录了所有被 Sentinel 监视的主服务器的相关信息，字典的键是被监视主服务器的名字，值是主服务器对应的实例结构

实例结构是一个 sentinelRedisinstance 数据类型，代表被 Sentinel 监视的实例，这个实例可以是主、从服务器，或者其他 Sentinel

#### 5、网络连接

初始化 Sentinel 的最后一步是创建连向被监视主服务器的网络连接，Sentinel 将成为主服务器的客户端，可以向主服务器发送命令，并从命令回复中获取相关的信息

每个被 Sentinel 监视的主服务器，Sentinel 会创建两个连向主服务器的异步网络连接：
- 命令连接：用于向主服务器发送命令，并接收命令回复
- 订阅连接：用于订阅主服务器的 _sentinel_:hello 频道

建立两个连接的原因：
- 在 Redis 目前的发布与订阅功能中，被发送的信息都不会保存在 Redis 服务器里， 如果在信息发送时接收信息的客户端离线或断线，那么这个客户端就会丢失这条信息，为了不丢失 hello 频道的任何信息，Sentinel 必须用一个订阅连接来接收该频道的信息

  Sentinel 还必须向主服务器发送命令，以此来与主服务器进行通信，所以 Sentinel 还必须向主服务器创建命令连接

  说明：断线的意思就是网络连接断开



### 信息交互

***



![输入图片说明](https://foruda.gitee.com/images/1680252806934441821/9d126d94_8616658.png "屏幕截图")

#### 获取信息

##### 主服务器

Sentinel 默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送 INFO 命令，来获取主服务器的信息

- 一部分是主服务器本身的信息，包括 runid 域记录的服务器运行 ID，以及 role 域记录的服务器角色
- 另一部分是服务器属下所有从服务器的信息，每个从服务器都由一个 slave 字符串开头的行记录，根据这些IP 地址和端口号，Sentinel 无须用户提供从服务器的地址信息，就可以**自动发现从服务器**

```bash
# Server
run_id:76llc59dc3a29aa6fa0609f84lbb6al019008a9c
...
# Replication
role:master
...
slave0: ip=l27.0.0.1, port=11111, state=online, offset=22, lag=0
slave1: ip=l27.0.0.1, port=22222, state=online, offset=22, lag=0
...
```

根据 run_id 和 role 记录的信息 Sentinel 将对主服务器的实例结构进行更新，比如主服务器重启之后，运行 ID 就会和实例结构之前保存的运行 ID 不同，哨兵检测到这一情况之后就会对实例结构的运行 ID 进行更新

对于主服务器返回的从服务器信息，用实例结构的 slaves 字典记录了从服务器的信息：

* 如果从服务器对应的实例结构已经存在，那么 Sentinel 对从服务器的实例结构进行更新
* 如果不存在，为这个从服务器新创建一个实例结构加入字典，字典键为 `ip:port`



##### 从服务器

当 Sentinel 发现主服务器有新的从服务器出现时，会为这个新的从服务器创建相应的实例结构，还会**创建到从服务器的命令连接和订阅连接**，所以 Sentinel 对所有的从服务器之间都可以进行命令操作

Sentinel 默认会以每十秒一次的频率，向从服务器发送 INFO 命令：

```sh
# Server 
run_id:76llc59dc3a29aa6fa0609f84lbb6al019008a9c	#从服务器的运行 id
...
# Replication 
role:slave 				# 从服务器角色
...
master_host:127.0.0.1 	# 主服务器的 ip
master_port:6379 		# 主服务器的 port
master_link_status:up 	# 主从服务器的连接状态
slave_repl_offset:11111	# 从服务器的复制偏移蜇
slave_priority:100 		# 从服务器的优先级
...
```

* **优先级属性**在故障转移时会用到

根据这些信息，Sentinel 会对从服务器的实例结构进行更新



#### 发送信息

Sentinel 在默认情况下，会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令：

```sh
PUBLISH _sentinel_:hello "<s_ip>, <s_port>, <s_runid>, <s_epoch>, <m_name>, <m_ip>, <m_port>, <m_epoch>
```

这条命令向服务器的 `_sentinel_:hello` 频道发送了一条信息，信息的内容由多个参数组成：

* 以 s_ 开头的参数记录的是 Sentinel 本身的信息
* 以 m_ 开头的参数记录的则是主服务器的信息

说明：**通过命令连接发送的频道信息**



#### 发送信息

Sentinel 在默认情况下，会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令：

```sh
PUBLISH _sentinel_:hello "<s_ip>, <s_port>, <s_runid>, <s_epoch>, <m_name>, <m_ip>, <m_port>, <m_epoch>
```

这条命令向服务器的 `_sentinel_:hello` 频道发送了一条信息，信息的内容由多个参数组成：

* 以 s_ 开头的参数记录的是 Sentinel 本身的信息
* 以 m_ 开头的参数记录的则是主服务器的信息

说明：**通过命令连接发送的频道信息**



***



#### 接受信息

##### 订阅频道

Sentinel 与一个主或从服务器建立起订阅连接之后，就会通过订阅连接向服务器发送订阅命令，频道的订阅会一直持续到 Sentinel 与服务器的连接断开为止

```sh
SUBSCRIBE _sentinel_:hello
```

订阅成功后，Sentinel 就可以通过订阅连接从服务器的 `_sentinel_:hello` 频道接收信息，对消息分析：

* 如果信息中记录的 Sentinel 运行 ID 与自己的相同，不做进一步处理
* 如果不同，将根据信息中的各个参数，对相应主服务器的实例结构进行更新

Sentinel 为主服务器创建的实例结构的 sentinels 字典保存所有同样监视这个**主服务器的 Sentinel 信息**（包括 Sentinel 自己），字典的键是 Sentinel 的名字，格式为 `ip:port`，值是键所对应 Sentinel 的实例结构

监视同一个服务器的 Sentinel 订阅的频道相同，Sentinel 发送的信息会被其他 Sentinel 接收到（发送信息的为源 Sentinel，接收信息的为目标 Sentinel），目标 Sentinel 在自己的 sentinelState.masters 中查找源 Sentinel 服务器的实例结构进行添加或更新

因为 Sentinel 可以接收到的频道信息来感知其他 Sentinel 的存在，并通过发送频道信息来让其他 Sentinel 知道自己的存在，所以用户在使用 Sentinel 时并不需要提供各个 Sentinel 的地址信息，**监视同一个主服务器的多个 Sentinel 可以相互发现对方**

哨兵实例之间可以相互发现，要归功于 Redis 提供发布订阅机制



***



##### 命令连接

Sentinel 通过频道信息发现新的 Sentinel，除了创建实例结构，还会创建一个连向新 Sentinel 的命令连接，而新 Sentinel 也同样会创建连向这个 Sentinel 的命令连接，最终监视同一主服务器的多个 Sentinel 将形成相互连接的网络

作用：**通过命令连接相连的各个 Sentinel** 可以向其他 Sentinel 发送命令请求来进行信息交换

Sentinel 之间不会创建订阅连接：

* Sentinel 需要通过接收主服务器或者从服务器发来的频道信息来发现未知的新 Sentinel，所以才创建订阅连接
* 相互已知的 Sentinel 只要使用命令连接来进行通信就足够了




***



### 下线检测

#### 主观下线

Sentinel 在默认情况下会以每秒一次的频率向所有与它创建了命令连接的实例（包括主从服务器、其他 Sentinel）发送 PING 命令，通过实例返回的 PING 命令回复来判断实例是否在线

* 有效回复：实例返回 +PONG、-LOADING、-MASTERDOWN 三种回复的其中一种
* 无效回复：实例返回除上述三种以外的任何数据

Sentinel 配置文件中 down-after-milliseconds 选项指定了判断实例进入主观下线所需的时长，如果主服务器在该时间内一直向 Sentinel 返回无效回复，Sentinel 就会在该服务器对应实例结构的 flags 属性打开 SRI_S_DOWN 标识，表示该主服务器进入主观下线状态

配置的 down-after-milliseconds 值不仅适用于主服务器，还会被用于当前 Sentinel 判断主服务器属下的所有从服务器，以及所有同样监视这个主服务器的其他 Sentinel 的主观下线状态

注意：对于监视同一个主服务器的多个 Sentinel 来说，设置的 down-after-milliseconds 选项的值可能不同，所以当一个 Sentinel 将主服务器判断为主观下线时，其他 Sentinel 可能仍然会认为主服务器处于在线状态



***



#### 客观下线

当 Sentinel 将一个主服务器判断为主观下线之后，会向同样监视这一主服务器的其他 Sentinel 进行询问

Sentinel 使用命令询问其他 Sentinel 是否同意主服务器已下线：

```sh
SENTINEL is-master-down-by-addr <ip> <port> <current_epoch> <runid>
```

* ip：被 Sentinel 判断为主观下线的主服务器的 IP 地址
* port：被 Sentinel 判断为主观下线的主服务器的端口号
* current_epoch：Sentinel 当前的配置纪元，用于选举领头 Sentinel
* runid：取值为 * 符号代表命令仅仅用于检测主服务器的客观下线状态；取值为 Sentinel 的运行 ID 则用于选举领头 Sentinel

目标 Sentinel 接收到源 Sentinel 的命令时，会根据参数的 lP 和端口号，检查主服务器是否已下线，然后返回一条包含三个参数的 Multi Bulk 回复：

* down_state：返回目标 Sentinel 对服务器的检查结果，1 代表主服务器已下线，0 代表未下线
* leader_runid：取值为 * 符号代表命令仅用于检测服务器的下线状态；而局部领头 Sentinel 的运行 ID 则用于选举领头 Sentinel
* leader_epoch：目标 Sentinel 的局部领头 Sentinel 的配置纪元

源 Sentinel 将统计其他 Sentinel 同意主服务器已下线的数量，当这一数量达到配置指定的判断客观下线所需的数量（quorum）时，Sentinel 会将主服务器对应实例结构 flags 属性的 SRI_O_DOWN 标识打开，代表客观下线，并对主服务器执行故障转移操作

注意：**不同 Sentinel 判断客观下线的条件可能不同**，因为载入的配置文件中的属性 quorum 可能不同



***



### 领头选举

主服务器被判断为客观下线时，**监视该主服务器的各个 Sentinel 会进行协商**，选举出一个领头 Sentinel 对下线服务器执行故障转移

Redis 选举领头 Sentinel 的规则：

* 所有在线的 Sentinel 都有被选为领头 Sentinel 的资格
* 每个发现主服务器进入客观下线的 Sentinel 都会要求其他 Sentinel 将自己设置为局部领头 Sentinel

* 在一个配置纪元里，所有 Sentinel 都只有一次将某个 Sentinel 设置为局部领头 Sentinel 的机会，并且局部领头一旦设置，在这个配置纪元里就不能再更改
* Sentinel 设置局部领头 Sentinel 的规则是先到先得，最先向目标 Sentinel 发送设置要求的源 Sentinel 将成为目标 Sentinel 的局部领头 Sentinel，之后接收到的所有设置要求都会被目标 Sentinel 拒绝
* 领头 Sentinel 的产生**需要半数以上 Sentinel 的支持**，并且每个 Sentinel 只有一票，所以一个配置纪元只会出现一个领头 Sentinel，比如 10 个 Sentinel 的系统中，至少需要 `10/2 + 1 = 6` 票

选举过程：

* 一个 Sentinel 向目标 Sentinel 发送 `SENTINEL is-master-down-by-addr` 命令，命令中的 runid 参数不是＊符号而是源 Sentinel 的运行 ID，表示源 Sentinel 要求目标 Sentinel 将自己设置为它的局部领头 Sentinel
* 目标 Sentinel 接受命令处理完成后，将返回一条命令回复，回复中的 leader_runid 和 leader_epoch 参数分别记录了目标 Sentinel 的局部领头 Sentinel 的运行 ID 和配置纪元
* 源 Sentinel 接收目标 Sentinel 命令回复之后，会判断 leader_epoch 是否和自己的相同，相同就继续判断 leader_runid 是否和自己的运行 ID 一致，成立表示目标 Sentinel 将源 Sentinel 设置成了局部领头 Sentinel，即获得一票
* 如果某个 Sentinel 被半数以上的 Sentinel 设置成了局部领头 Sentinel，那么这个 Sentinel 成为领头 Sentinel
* 如果在给定时限内，没有一个 Sentinel 被选举为领头 Sentinel，那么各个 Sentinel 将在一段时间后**再次选举**，直到选出领头
* 每次进行领头 Sentinel 选举之后，不论选举是否成功，所有 Sentinel 的配置纪元（configuration epoch）都要自增一次

Sentinel 集群至少 3 个节点的原因：

* 如果 Sentinel 集群只有 2 个 Sentinel 节点，则领头选举需要 `2/2 + 1 = 2` 票，如果一个节点挂了，那就永远选不出领头
* Sentinel 集群允许 1 个 Sentinel 节点故障则需要 3 个节点的集群，允许 2 个节点故障则需要 5 个节点集群

**如何获取哨兵节点的半数数量**？

* 客观下线是通过配置文件获取的数量，达到  quorum 就客观下线
* 哨兵数量是通过主节点是实例结构中，保存着监视该主节点的所有哨兵信息，从而获取得到


***



### 故障转移

#### 执行流程

领头 Sentinel 将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤

* 从下线主服务器属下的所有从服务器里面，挑选出一个从服务器，执行 `SLAVEOF no one`，将从服务器升级为主服务器

  在发送 SLAVEOF no one 命令后，领头 Sentinel 会以**每秒一次的频率**（一般是 10s/次）向被升级的从服务器发送 INFO 命令，观察命令回复中的角色信息，当被升级服务器的 role 从 slave 变为 master 时，说明从服务器已经顺利升级为主服务器

* 将已下线的主服务器的所有从服务器改为复制新的主服务器，通过向从服务器发送 SLAVEOF 命令实现

* 将已经下线的主服务器设置为新的主服务器的从服务器，设置是保存在服务器对应的实例结构中，当旧的主服务器重新上线时，Sentinel 就会向它发送 SLAVEOF 命令，成为新的主服务器的从服务器

示例：sever1 是主，sever2、sever3、sever4 是从服务器，sever1 故障后选中 sever2 升级

![输入图片说明](https://foruda.gitee.com/images/1680251637655239274/608a310b_8616658.png "屏幕截图")



#### 选择算法

领头 Sentinel 会将已下线主服务器的所有从服务器保存到一个列表里，然后按照以下规则对列表进行过滤，最后挑选出一个**状态良好、数据完整**的从服务器

* 删除列表中所有处于下线或者断线状态的从服务器，保证列表中的从服务器都是正常在线的

* 删除列表中所有最近五秒内没有回复过领头 Sentinel 的 INFO 命令的从服务器，保证列表中的从服务器最近成功进行过通信

* 删除所有与已下线主服务器连接断开超过 `down-after-milliseconds * 10` 毫秒的从服务器，保证列表中剩余的从服务器都没有过早地与主服务器断开连接，保存的数据都是比较新的

  down-after-milliseconds 时间用来判断是否主观下线，其余的时间完全可以完成客观下线和领头选举

* 根据从服务器的优先级，对列表中剩余的从服务器进行排序，并选出其中**优先级最高**的从服务器

* 如果有多个具有相同最高优先级的从服务器，领头 Sentinel 将对这些相同优先级的服务器按照复制偏移量进行排序，选出其中偏移量最大的从服务器，也就是保存着最新数据的从服务器

* 如果还没选出来，就按照运行 ID 对这些从服务器进行排序，并选出其中运行 ID 最小的从服务器



## 集群模式

### 集群节点

#### 节点概述

Redis 集群是 Redis 提供的分布式数据库方案，集群通过分片（sharding）来进行数据共享， 并提供复制和故障转移功能，一个 Redis 集群通常由多个节点（node）组成，将各个独立的节点连接起来，构成一个包含多节点的集群

一个节点就是一个**运行在集群模式下的 Redis 服务器**，Redis 在启动时会根据配置文件中的 `cluster-enabled` 配置选项是否为 yes 来决定是否开启服务器的集群模式

节点会继续使用所有在单机模式中使用的服务器组件，使用 redisServer 结构来保存服务器的状态，使用 redisClient 结构来保存客户端的状态，也有集群特有的数据结构

![输入图片说明](https://foruda.gitee.com/images/1680253055667759249/2e809585_8616658.png "屏幕截图")



#### 数据结构

每个节点都保存着一个集群状态 clusterState 结构，这个结构记录了在当前节点的视角下，集群目前所处的状态

```c
typedef struct clusterState {
    // 指向当前节点的指针
	clusterNode *myself;
    
	// 集群当前的配置纪元，用于实现故障转移
	uint64_t currentEpoch;
    
	// 集群当前的状态，是在线还是下线
	int state;
    
	// 集群中至少处理着一个槽的（主）节点的数量，为0表示集群目前没有任何节点在处理槽
    // 【选举时投票数量超过半数，从这里获取的】
	int size;

    // 集群节点名单（包括 myself 节点），字典的键为节点的名字，字典的值为节点对应的clusterNode结构 
    dict *nodes;
}
```

每个节点都会使用 clusterNode 结构记录当前状态，并为集群中的所有其他节点（包括主节点和从节点）都创建一个相应的 clusterNode 结构，以此来记录其他节点的状态

```c
struct clusterNode {
    // 创建节点的时间
    mstime_t ctime;
    
    // 节点的名字，由 40 个十六进制字符组成
    char name[REDIS_CLUSTER_NAMELEN];
    
    // 节点标识，使用各种不同的标识值记录节点的角色（比如主节点或者从节点）以及节点目前所处的状态（比如在线或者下线）
    int flags;
    
    // 节点当前的配置纪元，用于实现故障转移
    uint64_t configEpoch;
    
    // 节点的IP地址
    char ip[REDIS_IP_STR_LEN];
    
    // 节点的端口号
    int port;
    
    // 保存连接节点所需的有关信息
    clusterLink *link;
}
```

clusterNode 结构的 link 属性是一个 clusterLink 结构，该结构保存了连接节点所需的有关信息

```c
typedef struct clusterLink {
    // 连接的创建时间 
    mstime_t ctime;
    
	// TCP套接字描述符
	int fd;
    
	// 输出缓冲区，保存着等待发送给其他节点的消息(message)。 
    sds sndbuf;
    
	// 输入缓冲区，保存着从其他节点接收到的消息。
	sds rcvbuf;
    
	// 与这个连接相关联的节点，如果没有的话就为NULL
	struct clusterNode *node; 
}
```

* redisClient 结构中的套接宇和缓冲区是用于连接客户端的
* clusterLink 结构中的套接宇和缓冲区则是用于连接节点的

#### MEET

CLUSTER MEET 命令用来将 ip 和 port 所指定的节点添加到接受命令的节点所在的集群中

```sh
CLUSTER MEET <ip> <port> 
```

假设向节点 A 发送 CLUSTER MEET 命令，让节点 A 将另一个节点 B 添加到节点 A 当前所在的集群里，收到命令的节点 A 将与根据 ip 和 port 向节点 B 进行握手（handshake）：

* 节点 A 会为节点 B 创建一个 clusterNode 结构，并将该结构添加到自己的 clusterState.nodes 字典里，然后节点 A 向节点 B **发送 MEET 消息**（message）
* 节点 B 收到 MEET 消息后，节点 B 会为节点 A 创建一个 clusterNode 结构，并将该结构添加到自己的 clusterState.nodes 字典里，之后节点 B 将向节点 A **返回一条 PONG 消息**
* 节点 A 收到 PONG 消息后，代表节点 A 可以知道节点 B 已经成功地接收到了自已发送的 MEET 消息，此时节点 A 将向节点 B **返回一条 PING 消息**
* 节点 B 收到 PING 消息后， 代表节点 B 可以知道节点 A 已经成功地接收到了自己返回的 PONG 消息，握手完成

![输入图片说明](https://foruda.gitee.com/images/1680254124558376003/6036cf7c_8616658.png "屏幕截图")

节点 A 会将节点 B 的信息通过 Gossip 协议传播给集群中的其他节点，让其他节点也与节点 B 进行握手，最终经过一段时间之后，节点 B 会被集群中的所有节点认识



### 槽指派

#### 基本操作

Redis 集群通过分片的方式来保存数据库中的键值对，集群的整个数据库被分为 16384 个槽（slot），数据库中的每个键都属于 16384 个槽中的一个，集群中的每个节点可以处理 0 个或最多 16384 个槽（**每个主节点存储的数据并不一样**）

* 当数据库中的 16384 个槽都有节点在处理时，集群处于上线状态（ok）
* 如果数据库中有任何一个槽得没有到处理，那么集群处于下线状态（fail）

通过向节点发送 CLUSTER ADDSLOTS 命令，可以将一个或多个槽指派（assign）给节点负责

```sh
CLUSTER ADDSLOTS <slot> [slot ... ] 
```

```sh
127.0.0.1:7000> CLUSTER ADDSLOTS 0 1 2 3 4 ... 5000 # 将槽0至槽5000指派给节点7000负责
OK 
```

命令执行细节：

* 如果命令参数中有一个槽已经被指派给了某个节点，那么会向客户端返回错误，并终止命令执行
* 将 slots 数组中的索引 i 上的二进制位设置为 1，就代表指派成功




***



#### 节点指派

clusterNode 结构的 slots 属性和 numslot 属性记录了节点负责处理哪些槽：

```c
struct clusterNode {
    // 处理信息，一字节等于 8 位
    unsigned char slots[l6384/8];
    // 记录节点负责处理的槽的数量，就是 slots 数组中值为 1 的二进制位数量
    int numslots;
}
```

slots 是一个二进制位数组（bit array），长度为 `16384/8 = 2048` 个字节，包含 16384 个二进制位，Redis 以 0 为起始索引，16383 为终止索引，对 slots 数组的 16384 个二进制位进行编号，并根据索引 i 上的二进制位的值来判断节点是否负责处理槽 i：

* 在索引 i 上的二进制位的值为 1，那么表示节点负责处理槽 i
* 在索引 i 上的二进制位的值为 0，那么表示节点不负责处理槽 i

![输入图片说明](https://foruda.gitee.com/images/1680253724013247656/72fbcfef_8616658.png "屏幕截图")

取出和设置 slots 数组中的任意一个二进制位的值的**复杂度仅为 O(1)**，所以对于一个给定节点的 slots 数组来说，检查节点是否负责处理某个槽或者将某个槽指派给节点负责，这两个动作的复杂度都是 O(1)

**传播节点的槽指派信息**：一个节点除了会将自己负责处理的槽记录在 clusterNode 中，还会将自己的 slots 数组通过消息发送给集群中的其他节点，每个接收到 slots 数组的节点都会将数组保存到相应节点的 clusterNode 结构里面，因此集群中的**每个节点**都会知道数据库中的 16384 个槽分别被指派给了集群中的哪些节点





***



#### 集群指派

集群状态 clusterState 结构中的 slots 数组记录了集群中所有 16384 个槽的指派信息，数组每一项都是一个指向 clusterNode 的指针

```c
typedef struct clusterState {
    // ...
    clusterNode *slots[16384];
}
```

* 如果 slots[i] 指针指向 NULL，那么表示槽 i 尚未指派给任何节点
* 如果 slots[i] 指针指向一个 clusterNode 结构，那么表示槽 i 已经指派给该节点所代表的节点

通过该节点，程序检查槽 i 是否已经被指派或者取得负责处理槽 i 的节点，只需要访问 clusterState. slots[i] 即可，时间复杂度仅为 O(1)



***



#### 集群数据

集群节点保存键值对以及键值对过期时间的方式，与单机 Redis 服务器保存键值对以及键值对过期时间的方式完全相同，但是**集群节点只能使用 0 号数据库**，单机服务器可以任意使用

除了将键值对保存在数据库里面之外，节点还会用 clusterState 结构中的 slots_to_keys 跳跃表来**保存槽和键之间的关系**

```c
typedef struct clusterState {
    // ...
    zskiplist *slots_to_keys;
}
```

slots_to_keys 跳跃表每个节点的分值（score）都是一个槽号，而每个节点的成员（member）都是一个数据库键（按槽号升序）

* 当节点往数据库中添加一个新的键值对时，节点就会将这个键以及键的槽号关联到 slots_to_keys 跳跃表
* 当节点删除数据库中的某个键值对时，节点就会在 slots_to_keys 跳跃表解除被删除键与槽号的关联

![](https://seazean.oss-cn-beijing.aliyuncs.com/img/DB/Redis-槽和键跳跃表.png)

通过在 slots_to_keys 跳跃表中记录各个数据库键所属的槽，可以很方便地对属于某个或某些槽的所有数据库键进行批量操作，比如 `CLUSTER GETKEYSINSLOT <slot> <count>` 命令返回最多 count 个属于槽 slot 的数据库键，就是通过该跳表实现



### 集群监控原理

Sentinel基于心跳机制监测服务状态，每隔1秒向集群的每个实例发送ping命令：

•主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**。

•客观下线：若超过指定数量（quorum）的sentinel都认为该实例主观下线，则该实例**客观下线**。quorum值最好超过Sentinel实例数量的一半。

![输入图片说明](https://foruda.gitee.com/images/1678265187954575712/4b4ec4a1_8616658.png "屏幕截图")



### 集群故障恢复原理

一旦发现master故障，sentinel需要在salve中选择一个作为新的master，选择依据是这样的：

- 首先会判断slave节点与master节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该slave节点
- 然后判断slave节点的slave-priority值，越小优先级越高，如果是0则永不参与选举
- 如果slave-prority一样，则判断slave节点的offset值，越大说明数据越新，优先级越高
- 最后是判断slave节点的运行id大小，越小优先级越高。

当选出一个新的master后，该如何实现切换呢？

流程如下：

- sentinel给备选的slave1节点发送slaveof no one命令，让该节点成为master
- sentinel给所有其它slave发送slaveof 192.168.150.101 7002 命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave，当故障节点恢复后会自动成为新的master的slave节点

![输入图片说明](https://foruda.gitee.com/images/1678265642741275034/2788acf3_8616658.png "屏幕截图")



### 小结

Sentinel的三个作用是什么？

- 监控
- 故障转移
- 通知

Sentinel如何判断一个redis实例是否健康？

- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线
- 如果大多数sentinel都认为实例主观下线，则判定服务下线

故障转移步骤有哪些？

- 首先选定一个slave作为新的master，执行slaveof no one
- 然后让所有节点都执行slaveof 新master
- 修改故障节点配置，添加slaveof 新master




## 集群

***



## Redis分片集群



### 搭建分片集群

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决：

- 海量数据存储问题
- 高并发写的问题

使用分片集群可以解决上述问题，如图:

![分片集群](https://foruda.gitee.com/images/1678326981303066349/3407b63e_8616658.png "屏幕截图")



分片集群特征：

- 集群中有多个master，每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点

### 散列插槽

#### 插槽原理

Redis会把每一个master节点映射到0~16383共16384个插槽（hash slot）上，查看集群信息时就能看到

![散列卡槽](https://foruda.gitee.com/images/1678328286770708867/78bf099e_8616658.png "屏幕截图")



数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：

- key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分

例如：key是num，那么就根据num计算，如果是{itcast}num，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

如图，在7001这个节点执行set a 1时，对a做hash运算，对16384取余，得到的结果是15495，因此要存储到103节点。

到了7003后，执行`get num`时，对num做hash运算，对16384取余，得到的结果是2765，因此需要切换到7001节点

### 小结

Redis如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key的有效部分计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

如何将同一类数据固定的保存在同一个Redis实例？

- 这一类数据使用相同的有效部分，例如key都以{typeId}为前缀



## 多级缓存

传统的缓存策略一般是请求到达Tomcat后，先查询Redis，如果未命中则查询数据库，如图：

![输入图片说明](https://foruda.gitee.com/images/1678329581500169061/43c5dc43_8616658.png "屏幕截图")

存在下面的问题：

• 请求要经过Tomcat处理，Tomcat的性能成为整个系统的瓶颈

• Redis缓存失效时，会对数据库产生冲击

多级缓存就是充分利用请求处理的每个环节，分别添加缓存，减轻Tomcat压力，提升服务性能：

- 浏览器访问静态资源时，优先读取浏览器本地缓存
- 访问非静态资源（ajax查询数据）时，访问服务端
- 请求到达Nginx后，优先读取Nginx本地缓存
- 如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）
- 如果Redis查询未命中，则查询Tomcat
- 请求进入Tomcat后，优先查询JVM进程缓存
- 如果JVM进程缓存未命中，则查询数据库

![输入图片说明](https://foruda.gitee.com/images/1678329907171655306/c6108864_8616658.png "屏幕截图")

多级缓存的关键有两个：

- 一个是在nginx中编写业务，实现nginx本地缓存、Redis、Tomcat的查询
- 另一个就是在Tomcat中实现JVM进程缓存

其中Nginx编程则会用到OpenResty框架结合Lua这样的语言。



### JVM进程缓存

#### Caffeine

缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。我们把缓存分为两类：

- 分布式缓存，例如Redis：
  - 优点：存储容量更大、可靠性更好、可以在集群间共享
  - 缺点：访问缓存有网络开销
  - 场景：缓存数据量较大、可靠性要求较高、需要在集群间共享
- 进程本地缓存，例如HashMap、GuavaCache：
  - 优点：读取本地内存，没有网络开销，速度更快
  - 缺点：存储容量有限、可靠性较低、无法共享
  - 场景：性能要求较高，缓存数据量较小

**Caffeine**是一个基于Java8开发的，提供了近乎最佳命中率的高性能的本地缓存库。目前Spring内部的缓存使用的就是Caffeine。GitHub地址：https://github.com/ben-manes/caffeine

Caffeine既然是缓存的一种，肯定需要有缓存的清除策略，不然的话内存总会有耗尽的时候。

Caffeine提供了三种缓存驱逐策略：

- **基于容量**：设置缓存的数量上限

  ```java
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      .maximumSize(1) // 设置缓存大小上限为 1
      .build();
  ```

- **基于时间**：设置缓存的有效时间

  ```java
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      // 设置缓存有效期为 10 秒，从最后一次写入开始计时 
      .expireAfterWrite(Duration.ofSeconds(10)) 
      .build();

  ```

- **基于引用**：设置缓存为软引用或弱引用，利用GC来回收缓存数据。性能较差，不建议使用。

> **注意**：在默认情况下，当一个缓存元素过期的时候，Caffeine不会自动立即将其清理和驱逐。而是在一次读或写操作后，或者在空闲时间完成对失效数据的驱逐。



## Redis原理

### Redis数据结构

#### Redis数据结构-动态字符串

Redis构建了一种新的字符串结构，称为简单动态字符串（Simple Dynamic String），简称SDS。

C语言字符串存在很多问题：

- 获取字符串长度的需要通过运算
- 非二进制安全
- 不可修改

![输入图片说明](https://foruda.gitee.com/images/1678435436062038115/eeb9a5e0_8616658.png "屏幕截图")

![输入图片说明](https://foruda.gitee.com/images/1678331103330434537/88fe3858_8616658.png "屏幕截图")



扩容：

- 如果新字符串小于1M，则新空间为扩展后字符串长度的两倍+1；
- 如果新字符串大于1M，则新空间为扩展后字符串长度+1M+1。称为内存预分配。

优点：

- 获取字符串长度的时间复杂度为$O(1)$
- 支持动态扩容
- 减少内存分配次数
- 二进制安全

#### Redis数据结构-intset

IntSet是Redis中set集合的一种实现方式，基于整数数组来实现，并且具备长度可变、有序等特征。

```c
typedef struct intset {
  uint32_t encoding; /*编码方式，支持16位、32位、64位整数*/
  uint32_t length; /*元素个数*/
  int8_t contents[]; /*整数数组，保存集合元素*/
}
```



![输入图片说明](https://foruda.gitee.com/images/1678331419171305735/2ec42e06_8616658.png "屏幕截图")

encoding包含三种模式，表示存储的整数大小不同：

- int_16类似Java中的short
- int_32类似Java中的int
- int_64类似Java中long

Inset的特点：

- Redis会确保Inset中的元素唯一、有序
- 具备类型升级机制，可以节省内存空间
- 底层采用二分查找方式来查询



#### Redis数据结构-Dict

Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）

向Dict添加键值对时，Redis首先根据key计算出hash值（h），然后利用 h & sizemask来计算元素应该存储到数组中的哪个索引位置。我们存储k1=v1，假设k1的哈希值h =1，则1&3 =1，因此k1=v1要存储到数组角标1位置。

![输入图片说明](https://foruda.gitee.com/images/1678439549994842990/5438a42b_8616658.png "屏幕截图")

**Dict的扩容**

Dict中的HashTable就是数组结合单向链表的实现，当集合中元素较多时，必然导致哈希冲突增多，链表过长，则查询效率会大大降低。
Dict在每次新增键值对时都会检查负载因子（LoadFactor = used/size） ，满足以下两种情况时会触发哈希表扩容：

- 哈希表的 LoadFactor >= 1，并且服务器没有执行 BGSAVE 或者 BGREWRITEAOF 等后台进程；
- 哈希表的 LoadFactor > 5 ；

**Dict的rehash**

不管是扩容还是收缩，必定会创建新的哈希表，导致哈希表的size和sizemask变化，而key的查询与sizemask有关。因此必须对哈希表中的每一个key重新计算索引，插入新的哈希表，这个过程称为rehash。过程是这样的：

- 计算新hash表的realeSize，值取决于当前要做的是扩容还是收缩：
  - 如果是扩容，则新size为第一个大于等于dict.ht[0].used + 1的2^n
  - 如果是收缩，则新size为第一个大于等于dict.ht[0].used的2^n （不得小于4）
- 按照新的realeSize申请内存空间，创建dictht，并赋值给dict.ht[1]
- 设置dict.rehashidx = 0，标示开始rehash
- 将dict.ht[0]中的每一个dictEntry都rehash到dict.ht[1]
- 将dict.ht[1]赋值给dict.ht[0]，给dict.ht[1]初始化为空哈希表，释放原来的dict.ht[0]的内存
- 将rehashidx赋值为-1，代表rehash结束
- 在rehash过程中，新增操作，则直接写入ht[1]，查询、修改和删除则会在dict.ht[0]和dict.ht[1]依次查找并执行。这样可以确保ht[0]的数据只减不增，随着rehash最终为空

整个过程可以描述成：

![输入图片说明](https://foruda.gitee.com/images/1678671113227856232/23928845_8616658.png "屏幕截图")

总结：

Dict的结构：

- 类似java的HashTable，底层是数组加链表来解决哈希冲突
- Dict包含两个哈希表，ht[0]平常用，ht[1]用来rehash

Dict的伸缩：

- 当LoadFactor大于5或者LoadFactor大于1并且没有子进程任务时，Dict扩容
- 当LoadFactor小于0.1时，Dict收缩
- 扩容大小为第一个大于等于used + 1的2^n
- 收缩大小为第一个大于等于used 的2^n
- Dict采用渐进式rehash，每次访问Dict时执行一次rehash
- rehash时ht[0]只减不增，新增操作只在ht[1]执行，其它操作在两个哈希表

### Redis数据结构-ZipList

ZipList 是一种特殊的“双端链表” ，由一系列特殊编码的连续内存块组成。可以在任意一端进行压入/弹出操作, 并且该操作的时间复杂度为 O(1)。

![输入图片说明](https://foruda.gitee.com/images/1678671228635533987/00930a2d_8616658.png "屏幕截图")





| **属性**  | **类型**   | **长度** | **用途**                                   |
| ------- | -------- | ------ | ---------------------------------------- |
| zlbytes | uint32_t | 4 字节   | 记录整个压缩列表占用的内存字节数                         |
| zltail  | uint32_t | 4 字节   | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节，通过这个偏移量，可以确定表尾节点的地址。 |
| zllen   | uint16_t | 2 字节   | 记录了压缩列表包含的节点数量。 最大值为UINT16_MAX （65534），如果超过这个值，此处会记录为65535，但节点的真实数量需要遍历整个压缩列表才能计算得出。 |
| entry   | 列表节点     | 不定     | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定。             |
| zlend   | uint8_t  | 1 字节   | 特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。         |



**ZipListEntry**

ZipList 中的Entry并不像普通链表那样记录前后节点的指针，因为记录两个指针要占用16个字节，浪费内存。而是采用了下面的结构：

![输入图片说明](https://foruda.gitee.com/images/1678672452202874427/7a112a0f_8616658.png "屏幕截图")



- previous_entry_length：前一节点的长度，占1个或5个字节。
  - 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
  - 如果前一节点的长度大于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据
- encoding：编码属性，记录content的数据类型（字符串还是整数）以及长度，占用1个、2个或5个字节
- contents：负责保存节点的数据，可以是字符串或整数

ZipList中所有存储长度的数值均采用小端字节序，即低位字节在前，高位字节在后。例如：数值0x1234，采用小端字节序后实际存储值为：0x3412

### Redis数据结构-ZipList的连锁更新问题

ZipList的每个Entry都包含previous_entry_length来记录上一个节点的大小，长度是1个或5个字节：
如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
如果前一节点的长度大于等于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据
现在，假设我们有N个连续的、长度为250~253字节之间的entry，因此entry的previous_entry_length属性用1个字节即可表示，如图所示：

![输入图片说明](https://foruda.gitee.com/images/1678673355987556423/ca2bff32_8616658.png "屏幕截图")

ZipList这种特殊情况下产生的连续多次空间扩展操作称之为连锁更新（Cascade Update）。新增、删除都可能导致连锁更新的发生。

**小总结：**

**ZipList特性：**

- 压缩列表的可以看做一种连续内存空间的"双向链表"
- 列表的节点之间不是通过指针连接，而是记录上一节点和本节点长度来寻址，内存占用较低
- 如果列表数据过多，导致链表过长，可能影响查询性能
- 增或删较大数据时有可能发生连续更新问题



### Redis数据结构-QuickList

问题1：ZipList虽然节省内存，但申请内存必须是连续空间，如果内存占用较多，申请内存效率很低。怎么办？

​	答：为了缓解这个问题，我们必须限制ZipList的长度和entry大小。

问题2：但是我们要存储大量数据，超出了ZipList最佳的上限该怎么办？

​	答：我们可以创建多个ZipList来分片存储数据。

问题3：数据拆分后比较分散，不方便管理和查找，这多个ZipList如何建立联系？

​	答：Redis在3.2版本引入了新的数据结构QuickList，它是一个双端链表，只不过链表中的每个节点都是一个ZipList。

![输入图片说明](https://foruda.gitee.com/images/1678673528759834449/e4b2df2a_8616658.png "屏幕截图")



总结：

QuickList的特点：

- 是一个节点为ZipList的双端链表
- 节点采用ZipList，解决了传统链表的内存占用问题
- 控制了ZipList大小，解决连续内存空间申请效率问题
- 中间节点可以压缩，进一步节省了内存

#### Redis数据结构-SkipList

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：
元素按照升序排列存储
节点可能包含多个指针，指针跨度不同。

![输入图片说明](https://foruda.gitee.com/images/1678673594375370078/d512dee7_8616658.png "屏幕截图")

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：
元素按照升序排列存储
节点可能包含多个指针，指针跨度不同。

### Redis数据结构-RedisObject

Redis中的任意数据类型的键和值都会被封装为一个RedisObject，也叫做Redis对象，源码如下：

1、什么是redisObject：
从Redis的使用者的角度来看，⼀个Redis节点包含多个database（非cluster模式下默认是16个，cluster模式下只能是1个），而一个database维护了从key space到object space的映射关系。这个映射关系的key是string类型，⽽value可以是多种数据类型，比如：
string, list, hash、set、sorted set等。我们可以看到，key的类型固定是string，而value可能的类型是多个。
⽽从Redis内部实现的⾓度来看，database内的这个映射关系是用⼀个dict来维护的。dict的key固定用⼀种数据结构来表达就够了，这就是动态字符串sds。而value则比较复杂，为了在同⼀个dict内能够存储不同类型的value，这就需要⼀个通⽤的数据结构，这个通用的数据结构就是robj，全名是redisObject。

![输入图片说明](https://foruda.gitee.com/images/1678674941202350797/811db118_8616658.png "屏幕截图")



#### Redis数据结构-String

底层实现⽅式：动态字符串sds 或者 long

String的内部存储结构⼀般是sds（Simple Dynamic String，可以动态扩展内存），但是如果⼀个String类型的value的值是数字，并且在long的取值范围，那么Redis内部会把它转成long类型来存储，从⽽减少内存的使用。

### Redis数据结构-List

Redis的List类型可以从首、尾操作列表中的元素：

Redis的List结构类似一个双端链表，可以从首、尾操作列表中的元素：

在3.2版本之前，Redis采用ZipList和LinkedList来实现List，当元素数量小于512并且元素大小小于64字节时采用ZipList编码，超过则采用LinkedList编码。

在3.2版本之后，Redis统一采用QuickList来实现List

### Redis数据结构-Set结构

Set是Redis中的单列集合，满足下列特点：

- 不保证有序性
- 保证元素唯一
- 求交集、并集、差集

Set是Redis中的集合，不一定确保元素有序，可以满足元素唯一、查询效率要求极高。
为了查询效率和唯一性，set采用HT编码（Dict）。Dict中的key用来存储元素，value统一为null。
当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用IntSet编码，以节省内存

ZSet也就是SortedSet，其中每一个元素都需要指定一个score值和member值：

- 可以根据score值排序后
- member必须唯一
- 可以根据member查询分数

当元素数量不多时，HT和SkipList的优势不明显，而且更耗内存。因此zset还会采用ZipList结构来节省内存，不过需要同时满足两个条件：

- 元素数量小于zset_max_ziplist_entries，默认值128
- 每个元素都小于zset_max_ziplist_value字节，默认值64

ziplist本身没有排序功能，而且没有键值对的概念，因此需要有zset通过编码实现：

- ZipList是连续内存，因此score和element是紧挨在一起的两个entry， element在前，score在后
- score越小越接近队首，score越大越接近队尾，按照score值升序排列

### Redis数据结构-Hash

Hash结构与Redis中的Zset非常类似：

- 都是键值存储
- 都需求根据键获取值
- 键必须唯一

区别如下：

- zset的键是member，值是score；hash的键和值都是任意值
- zset要根据score排序；hash则无需排序

底层实现方式：压缩列表ziplist 或者 字典dict
当Hash中数据项比较少的情况下，Hash底层才⽤压缩列表ziplist进⾏存储数据，随着数据的增加，底层的ziplist就可能会转成dict

Redis的hash之所以这样设计，是因为当ziplist变得很⼤的时候，它有如下几个缺点：

- 每次插⼊或修改引发的realloc操作会有更⼤的概率造成内存拷贝，从而降低性能。
- ⼀旦发生内存拷贝，内存拷贝的成本也相应增加，因为要拷贝更⼤的⼀块数据。
- 当ziplist数据项过多的时候，在它上⾯查找指定的数据项就会性能变得很低，因为ziplist上的查找需要进行遍历。

总之，ziplist本来就设计为各个数据项挨在⼀起组成连续的内存空间，这种结构并不擅长做修改操作。⼀旦数据发⽣改动，就会引发内存realloc，可能导致内存拷贝。

Hash底层采用的编码与Zset也基本一致，只需要把排序有关的SkipList去掉即可：

Hash结构默认采用ZipList编码，用以节省内存。 ZipList中相邻的两个entry 分别保存field和value

当数据量较大时，Hash结构会转为HT编码，也就是Dict，触发条件有两个：

- ZipList中的元素数量超过了hash-max-ziplist-entries（默认512）
- ZipList中的任意entry大小超过了hash-max-ziplist-value（默认64字节）




## Redis网络模型

用户应用通常是无法执行访问操作系统硬件的，通常通过内核访问计算机硬件。

计算机硬件包括，如cpu，内存，网卡等等，内核（通过寻址空间）可以操作硬件的，但是内核需要不同设备的驱动，有了这些驱动之后，内核就可以去对计算机硬件去进行 内存管理，文件系统的管理，进程的管理等等

![输入图片说明](https://foruda.gitee.com/images/1678756353458477204/3ef3f807_8616658.png "屏幕截图")

一般情况下，用户的操作是运行在用户空间，而内核运行的数据是在内核空间的，而有的情况下，一个应用程序需要去调用一些特权资源，去调用一些内核空间的操作，所以此时他俩需要在用户态和内核态之间进行切换。

Linux系统为了提高IO效率，会在用户空间和内核空间都加入缓冲区：

- 写数据时，要把用户缓冲数据拷贝到内核缓冲区，然后写入设备
- 读数据时，要从设备读取数据到内核缓冲区，然后拷贝到用户缓冲区

用户在读写数据时，会去向内核态申请，想要读取内核的数据，而内核数据要去等待驱动程序从硬件上读取数据，当从磁盘上加载到数据之后，内核会将数据写入到内核的缓冲区中，然后再将数据拷贝到用户态的buffer中，然后再返回给应用程序，整体而言，速度慢。

![输入图片说明](https://foruda.gitee.com/images/1678756983655323440/74aecddb_8616658.png "屏幕截图")



### 网络模型-阻塞IO

在《UNIX网络编程》一书中，总结归纳了5种IO模型：

- 阻塞IO（Blocking IO）
- 非阻塞IO（Nonblocking IO）
- IO多路复用（IO Multiplexing）
- 信号驱动IO（Signal Driven IO）
- 异步IO（Asynchronous IO）

应用程序想要去读取数据，他是无法直接去读取磁盘数据的，他需要先到内核里边去等待内核操作硬件拿到数据，这个过程就是1，是需要等待的，等到内核从磁盘上把数据加载出来之后，再把这个数据写给用户的缓存区，这个过程是2，如果是阻塞IO，那么整个过程中，用户从发起读请求开始，一直到读取到数据，都是一个阻塞状态。

![输入图片说明](https://foruda.gitee.com/images/1678757564451956817/4f529abe_8616658.png "屏幕截图")

阻塞IO就是两个阶段都必须阻塞等待，用户去读取数据时，会去先发起recvform一个命令，去尝试从内核上加载数据，如果内核没有数据，那么用户就会等待，此时内核会去从硬件上读取数据，内核读取数据之后，会把数据拷贝到用户态，并且返回ok，整个过程，都是阻塞等待的，这就是阻塞IO。

![输入图片说明](https://foruda.gitee.com/images/1678757658985043712/96db5034_8616658.png "屏幕截图")



### 网络模型-非阻塞IO

非阻塞IO的recvfrom操作会立即返回结果而不是阻塞用户进程。

阶段一：

- 用户进程尝试读取数据（比如网卡数据）
- 此时数据尚未到达，内核需要等待数据
- 返回异常给用户进程
- 用户进程拿到error后，再次尝试读取
- 循环往复，直到数据就绪

阶段二：

- 将内核数据拷贝到用户缓冲区
- 拷贝过程中，用户进程依然阻塞等待
- 拷贝完成，用户进程解除阻塞，处理数据
- 可以看到，非阻塞IO模型中，用户进程在第一个阶段是非阻塞，第二个阶段是阻塞状态。虽然是非阻塞，但性能并没有得到提高。而且忙等机制会导致CPU空转，CPU使用率暴增。

![输入图片说明](https://foruda.gitee.com/images/1678757784116346023/8168e0de_8616658.png "屏幕截图")



阻塞IO和非阻塞IO的差别？

差别在于用户应用读取数据的第一阶段调用recvform阶段的处理：

- 阻塞IO调用recvform，如果内核状态恰好没有数据，则使进程阻塞（进程进入等待状态），等待数据就绪
- 非阻塞IO调用recvform，如果内核状态恰好没有数据，内核态返回一个异常结果，用户根据异常结果再次请求，使得CPU空转（用户进程一直在CPU进程执行队列中 ）。



### 网络模型-IO多路复用

文件描述符（File Descriptor）：简称FD，是一个从0 开始的无符号整数，用来关联Linux中的一个文件。在Linux中，一切皆文件，例如常规文件、视频、硬件设备等，当然也包括网络套接字（Socket）。

通过FD，我们的网络模型可以利用一个线程监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。

#### 网络模型-IO多路复用-select方式

阶段一：

- 用户进程调用select，指定要监听的FD集合
- 核监听FD对应的多个socket
- 任意一个或多个socket数据就绪则返回readable
- 此过程中用户进程阻塞

阶段二：

- 用户进程找到就绪的socket
- 依次调用recvfrom读取数据
- 内核将数据拷贝到用户空间
- 用户进程处理数据

当用户去读取数据的时候，不再去直接调用recvfrom了，而是调用select的函数，select函数会将需要监听的数据交给内核，由内核去检查这些数据是否就绪了，如果说这个数据就绪了，就会通知应用程序数据就绪，然后来读取数据，再从内核中把数据拷贝给用户态，完成数据处理，如果N多个FD一个都没处理完，此时就进行等待。

![输入图片说明](https://foruda.gitee.com/images/1678761671034642237/5c8b7626_8616658.png "屏幕截图")



#### 网络模型-IO多路复用模型-poll模式

IO流程：

- 创建pollfd数组，向其中添加关注的fd信息，数组大小自定义
- 调用poll函数，将pollfd数组拷贝到内核空间，转链表存储，无上限
- 内核遍历fd，判断是否就绪
- 数据就绪或超时后，拷贝pollfd数组到用户空间，返回就绪fd数量n
- 用户进程判断n是否大于0,大于0则遍历pollfd数组，找到就绪的fd

#### 网络模型-IO多路复用模型-epoll函数

内部包含连个结构：

- 红黑树-> 记录的事要监听的FD
- 一个是链表->一个链表，记录的是就绪的FD

##### 网络模型-epoll中的ET和LT

当FD有数据可读时，我们调用epoll_wait（或者select、poll）可以得到通知。但是事件通知的模式有两种：

- LevelTriggered：简称LT，也叫做水平触发。只要某个FD中有数据可读，每次调用epoll_wait都会得到通知。
- EdgeTriggered：简称ET，也叫做边沿触发。只有在某个FD有状态变化时，调用epoll_wait才会被通知。

### 网络模型-信号驱动

信号驱动IO是与内核建立SIGIO的信号关联并设置回调，当内核有FD就绪时，会发出SIGIO信号通知用户，期间用户应用可以执行其它业务，无需阻塞等待。

阶段一：

- 用户进程调用sigaction，注册信号处理函数
- 内核返回成功，开始监听FD
- 用户进程不阻塞等待，可以执行其它业务
- 当内核数据就绪后，回调用户进程的SIGIO处理函数

阶段二：

- 收到SIGIO回调信号
- 调用recvfrom，读取
- 内核将数据拷贝到用户空间
- 用户进程处理数据

![输入图片说明](https://foruda.gitee.com/images/1678762897804968824/081d3f44_8616658.png "屏幕截图")

当有大量IO操作时，信号较多，SIGIO处理函数不能及时处理可能导致信号队列溢出，而且内核空间与用户空间的频繁信号交互性能也较低。

### 异步IO

由内核将所有数据处理完成后，由内核将数据写入到用户态中，然后才算完成，所以性能极高，不会有任何阻塞，全部都由内核完成，可以看到，异步IO模型中，用户进程在两个阶段都是非阻塞状态。

![输入图片说明](https://foruda.gitee.com/images/1678763732855927303/2939d96b_8616658.png "屏幕截图")

### Redis单线程（核心命令处理）和多线程网络模型

如果仅仅聊Redis的核心业务部分（命令处理），答案是单线程；如果是聊整个Redis，那么答案就是多线程；

在Redis版本迭代过程中，在两个重要的时间节点上引入了多线程的支持：

- Redis v4.0：引入多线程异步处理一些耗时较旧的任务，例如异步删除命令unlink
- Redis v6.0：在核心网络模型中引入 多线程，进一步提高对于多核CPU的利用率

因此，对于Redis的核心网络模型，在Redis 6.0之前确实都是单线程。是利用epoll（Linux系统）这样的IO多路复用技术在事件循环中不断处理客户端情况。

**为什么Redis要选择单线程？**

- 抛开持久化不谈，Redis是纯  内存操作，执行速度非常快，它的性能瓶颈是网络延迟而不是执行速度，因此多线程并不会带来巨大的性能提升。
- 多线程会导致过多的上下文切换，带来不必要的开销
- 引入多线程会面临线程安全问题，必然要引入线程锁这样的安全手段，实现复杂度增高，而且性能也会大打折扣

![输入图片说明](https://foruda.gitee.com/images/1678764140650862492/ee95bf76_8616658.png "屏幕截图")



### Redis通信协议-RESP协议

### Redis内存回收-过期key处理

**惰性删除**

惰性删除：顾明思议并不是在TTL到期后就立刻删除，而是在访问一个key的时候，检查该key的存活时间，如果已经过期才执行删除。

**周期删除**

周期删除：顾明思议是通过一个定时任务，周期性的抽样部分过期的key，然后执行删除。

执行周期有两种：

- Redis服务初始化函数initServer()中设置定时任务，按照server.hz的频率来执行过期key清理，模式为SLOW
- Redis的每个事件循环前会调用beforeSleep()函数，执行过期key清理，模式为FAST

### Redis内存回收-内存淘汰策略

内存淘汰：就是当Redis内存使用达到设置的上限时，主动挑选部分key删除以释放更多内存的流程。Redis会在处理客户端命令的方法processCommand()中尝试做内存淘汰：

Redis支持8种不同策略来选择要删除的key：

- noeviction： 不淘汰任何key，但是内存满时不允许写入新数据，默认就是这种策略。
- volatile-ttl： 对设置了TTL的key，比较key的剩余TTL值，TTL越小越先被淘汰
- allkeys-random：对全体key ，随机进行淘汰。也就是直接从db->dict中随机挑选
- volatile-random：对设置了TTL的key ，随机进行淘汰。也就是从db->expires中随机挑选。
- allkeys-lru： 对全体key，基于LRU算法进行淘汰
- volatile-lru： 对设置了TTL的key，基于LRU算法进行淘汰
- allkeys-lfu： 对全体key，基于LFU算法进行淘汰
- volatile-lfu： 对设置了TTL的key，基于LFI算法进行淘汰
  比较容易混淆的有两个：
  - LRU（Least Recently Used），最少最近使用。用当前时间减去最后一次访问时间，这个值越大则淘汰优先级越高。
  - LFU（Least Frequently Used），最少频率使用。会统计每个key的访问频率，值越小淘汰优先级越高。