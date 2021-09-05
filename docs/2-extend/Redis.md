## 一、NoSQL概述

### 1、为什么要使用NoSQL

- 现在是大数据时代（一般的数据库已经解决不了的数据：大数据（海量存储和并行计算））

  **SQL => NoSQL**

  > 1、单机时代（如MySQL）的**瓶颈**

  - 数据量太大，一个机器存不下
  - 数据的索引<B+Tree>（MySQL单表300万条数据，一定要建立索引）,一个机器内存放不下
  - 访问量大（MySQL读写混合 — 性能降低），一个服务器承受不了

  > 2、Memcached缓存（减轻服务器的压力） + MySQL + 垂直拆分（读写分离，多个MySQL服务器，有的值负责读，有的值负责写）

  发展过程：①优化数据结构和索引 => ②文件缓存（涉及IO操作）=> ③Memcached

  > 3、分库分表 + 水平拆分 + MySQL集群

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130110.png)

- 用户的个人信息（社交网络、地理位置、用户日志等）爆发是增长，无法使用关系型数据库去存储，那么需要NoSQL数据库

### 2、什么是NoSQL

> #### Not Only SQL —> 非关系型数据库

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130153.png)

> #### NoSQL特点

1. 方便扩展（数据之间没有关系）
2. 大数据高性能（Redis一秒写8万次，读取11万次，NoSQL的缓存记录级，是一种细粒度的缓存，性能比较高）
3. 数据类型是多样性的（不需要事先设计数据库（因为数据量大），随取随用）
4. 传统RDBMS和NoSQL
   - 传统RDBMS
     - 结构化组织
     - SQL
     - 数据和关系都存在单独的表中
     - 操作，数据定义语言
     - 严格的一致性
     - 基础的事务
     - …
   - NoSQL
     - 不仅仅是SQL
     - 没有固定的查询语句
     - 键值对存储，列存储，文档存储，图形数据库（社交关系）
     - 最终一致性
     - CAP定理和BASE
     - 高性能，高可用，高可扩
     - …

#### 了解：3V + 3高

大数据时代的3V：主要是描述问题的

1. 海量Volume
2. 多样Variety
3. 实时Velocity

大数据时代的3高：主要是对程序的要求

1. 高并发
2. 高可拓（集群）
3. 高性能

```
电商网站：# 1、商品的基本信息    名称、价格、商家信息：        关系型数据库： MySQL / Oracle (王坚：阿里去IOE（IBM小型机，Oracle数据库、EMC存储器）)# 2、商品的描述、评论（文字多）    文档型数据库：MongoDB# 3、图片    分布式文件系统：FastDFS、TFS（淘宝）、GFS（Google）、HDFS（Hadoop）、OSS云存储（阿里云）# 4、商品的关键字（搜索）    搜索引擎：solr、ElasticSearch、ISearch（阿里：多隆）# 5、商品热门的波段信息    内存数据库：Redis、Tair、Memcached、...# 6、商品的交易、外部的接口    三方应用
```

### 3、NoSQL的四大分类

#### 1.KV键值对

- 新浪：Redis
- 美团：Redis + Tair
- 阿里、百度：Redis + Memcached

#### 2.文档型数据库（bson格式）

- MongoDB（必须掌握）
  - 基于分布式文件存储的数据库（C++编写）
  - 主要用于处理大量的文档
  - 介于关系型数据库和非关系型数据库中的中间产品
- ConthDB

#### 3.列式存储

- HBase
- 分布式文件系统

#### 4.图关系数据库

- 存关系的，不是存图片的（比如，朋友圈社交网络，广告推荐）
- Neo4j、InfoGrid

#### 对比

| **分类**              | **Examples举例**                                           | **典型应用场景**                                             | **数据模型**                                | **优点**                                                     | **缺点**                                                     |
| --------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **键值（key-value）** | Tokyo<br>Cabine/Tyrant<br>Redis<br>Voldemort<br>Oracle BDB | **内容缓存**，主要用于处理大量数据的高访问负载，也用于一些**日志系统**等等 | Key指向Value的键值对，通常用hashtable来实现 | 查找速度快                                                   | 数据无结构化，通常只被当作字符串或者二进制数据               |
| **列存储数据库**      | Cassandra<br>HBase<br>Riak                                 | 分布式的文件系统                                             | 以列簇式存储，将同一列数据存在一起          | 查找速度快，可扩展性强，更容易进行分布式扩展                 | 功能相对局限                                                 |
| **文档型数据库**      | CouchDB<br>MongoDB                                         | Web应用（与Key-Value类似，Value是结构化的，不同的是数据库能够了解Value的内容） | Key-Value对应的键值对，Value为结构化数据    | 数据结构要求不严谨，表结构可变，不需要像关系型数据库一样需要预先定义表结构 | 查询性能不高，而且缺乏统一的查询语法                         |
| **图形数据库**        | Neo4J<br>InfoGrid<br>Infinite Graph                        | 社交网络，推荐系统等。专注于构建关系图谱                     | 图结构                                      | 利用图结构相关算法。比如最短路径寻址，N度关系查找等          | 很多时候需要对整个图做计算才能得出需要的信息，而且这种结构不太好做分布式的集群方案 |

## 二、Redis入门

### 1.Redis是什么

> 官网：[redis.io](https://www.kuangstudy.com/bbs/redis.io)
>
> 中文网：http://www.redis.cn/
>
> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作<mark>数据库</mark>、<mark>缓存</mark>和<mark>消息中间件</mark>。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

#### Redis 与其他 key - value 缓存产品有以下三个特点：

1. Redis**支持数据的持久化**，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
2. Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储（**多样的数据结构**）。
3. Redis支持数据的备份，即**master-slave模式的数据备份**。

#### Redis优势

1. **性能极高** – Redis能读的速度是110000次/s,写的速度是81000次/s 。
2. **丰富的数据类型** – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
3. **原子** – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。（**但是Redis事务多操作不支持原子性—当我们执行后有语句有错，其他语句仍然可以正常执行**，<mark>详情看事务</mark>）
4. **丰富的特性** – Redis还支持 publish/subscribe, 通知, key 过期等等特性。
5. Redis运行在内存中但是可以**持久化**到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相**比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单**，这样Redis可以做很多内部复杂性很强的事情。同时，**在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问**。

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130236.png)

### 2.能干什么？

- 内存存储、**持久化（rdb，aof）**
- 效率高、可以用于高速缓存
- 发布订阅系统
- 地图信息分析
- 计时器、计数器（浏览量）
- …

### 3.特性

- 多样的数据结构
- 持久化
- 集群
- 事务
- …

### 4.安装（因为Redis更适合在linux下使用，所以只有linux的安装版本）

> linux 使用的系统是 Centos 7.3
>
> 你可使用 虚拟机，也可已使用云服务器（学生机不贵），因为我有服务器，这里就在服务器上安装和学习Redis

①下载安装包

http://www.redis.cn/

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130247.png)

②加压安装包（提前使用xshell和xftp上传压缩包到服务器上）

> 压缩包在 /app/目录下

在/usr/local/ 下创建redis文件夹，并进入到文件夹中

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130258.png)

解压压缩包到该文件夹下

```
 tar -zxvf /app/redis-6.0.6.tar.gz -C ./
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130303.png)

③编译并安装（保证安装了编译环境）

> 没有安装编译环境，需要 yum install gcc-c++
>
> 如果下面操作报错：需要提升GCC的版本（redis6需要5.3+）
>
> ```
> //升级gcc到9以上yum -y install centos-release-sclyum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils//临时将此时的gcc版本改为9scl enable devtoolset-9 bash//或永久改变echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
> ```

```
cd redis-6.0.6make && make install
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130308.png)

编译后，默认安装路径在 `/usr/local/bin`

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130313.png)

④安装系统服务并后台启动

> 可能会报错：
>
> ![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130318.png)
>
> 解决方法：vim install_server.sh，注释如下图内容
>
> ![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130327.png)

**安装系统服务**（可以指定选项，下面默认）

```
cd utils./install_server.sh
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130336.png)

> 默认的配置文件：/etc/redis/6379.conf（安装成功后，默认后台自启）

⑤开启redis服务

> 可以使用自定义的配置文件开启，这里使用systemctl
>
> ```
> /redis安装目录/redis-server    /配置文件目录/redis.conf# 关闭服务/redis安装目录/redis-cli shutdown
> ```

```
systemctl start redis_6379.service
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130344.png)

⑥客户端连接测试

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130348.png)

⑦设置远程连接

```
vim /etc/redis/6379.conf
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130353.png)

![关闭保护模式](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130357.png)

> 然后重启服务即可：
>
> ```
> systemctl restart redis_6379.service
> ```

⑧配置访问密码

```
vim /etc/redis/6379.conf
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130401.png)

重启服务

```
systemctl restart redis_6379.service
```

客户端连接测试

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130407.png)

关闭连接

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130411.png)

### 5.性能测试

> 使用自带的`redis-benchmark`工具测试即可

```
redis-benchmark [option] [option value]
```

**性能测试工具可选参数：**

| 序号 | 选项     | 描述                                       | 默认值    |
| :--- | :------- | :----------------------------------------- | :-------- |
| 1    | **-h**   | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**   | 指定服务器端口                             | 6379      |
| 3    | **-s**   | 指定服务器 socket                          |           |
| 4    | **-c**   | 指定并发连接数                             | 50        |
| 5    | **-n**   | 指定请求数                                 | 10000     |
| 6    | **-d**   | 以字节的形式指定 SET/GET 值的数据大小      | 3         |
| 7    | **-k**   | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**   | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**   | 通过管道传输 \<numreq> 请求                | 1         |
| 10   | **-q**   | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **—csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**   | 生成循环，永久执行测试                     |           |
| 13   | **-t**   | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**   | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

#### 测试

```
# 测试：100个并发连接  100000请求redis-benchmark -c 100 -n 100000
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130417.png)

### 6.基本知识说明（基本命令）

> 如果觉得这里命令不太好看，去这个博客https://www.cnblogs.com/wlandwl/p/redis.html，或者 官网：http://www.redis.cn/commands.html

**注意：以下的所有key都表示对应数据类型的 `键的名称` ； value表示`存储的值`(除非注释中有特殊说明)**

#### 1.Redis 有16个数据库（0~15），默认使用第0个

> 可以使用 `select`进行切换

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130423.png)

#### 2.查看数据库大小

> 使用 `dbsize`查看数据库大小

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130427.png)

#### 3.查看所有的key（当前库）

> 使用 `keys *`

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130431.png)

#### 4.清除当前数据库

> ```
> flushdb
> ```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130435.png)

#### 5.清空所有数据库

> ```
> flushall
> ```

略

#### 6.Redis是单线程的？

- Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis的性能瓶颈，**Redis的性能瓶颈是根据机器的<mark>内存</mark>和<mark>网络的带宽</mark>**，既然可以使用单线程，就不用使用多线程。(<mark>6.0后支持多线程</mark>)

![6.0另当别论](https://www.kuangstudy.com/bbs/2020-11-2-Redis%E8%AF%A6%E7%BB%86%E7%AC%94%E8%AE%B0.assets/image-20201112020411584.png)

#### 7.判断key是否存在

> `exists` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130441.png)

#### 8.移除key

> `move` key 1 // 1表示当前数据库
>
> `del` key // 删除当前数据库的key（可以多个）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130447.png)

#### 9.设置key的过期时间

> `expire` key 时间 // 单位时间为s
>
> `ttl` key // 查看剩余存活时间

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130453.png)

#### 10.查看当前key的类型

> `type` key

#### 11.字符串追加（String）

> `append` key appendValue

- <mark>如果当前 key 不存在，作用相当于 set key</mark>

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130458.png)

#### 12.获取字符串长度（String）

> `strlen` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130502.png)

#### 13.字符串i++操作（可用于阅读量实现）（String）

> `incr` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130506.png)

##### 同理，i—

> `decr` key

##### 步长设置

> `incrby` key 步长
>
> `decrby` key 步长

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130509.png)

#### 14.字符串片段 Range （String）

> `getrange` key startIndex endIndex

- **对应java里的substring**（<mark>但是这里会endIndex是一个闭区间</mark>）
- 特例，endIndex = -1 时，表示从startIndex 到最后

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130513.png)

#### 15.字符串替换 （String）

> `setrange` key index replaceString

- **对应java里的replace**
- 注意，如果replaceString是一个字符串，那么会替换源字符串中index后replaceString长度的片段，结果如下

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130517.png)

#### 16.特殊set设置 （String）

##### setex（set with expire）

> `setex` key 时间 value // 设置值，带过期时间

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130520.png)

##### setnx（set if not exist）

> `setnx` key value // 如果不存在，则设置

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130524.png)

#### 17.批量设置、批量获取（原子性操作） （String）

> `mset` k1 v1 k2 v2 …
>
> `mget` k1 k2 …

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130527.png)

- 特殊
  - `msetnx` 批量设置

#### 18.设置高阶 （String）

##### 1）getset

> `getset` key value // 先get再set（如果不存在，先返回nil，在设置值；如果存在，就先返回原值，再设置新值）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130531.png)

#### 19.重命名key

> `rename` key newName

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130541.png)

#### 20.返回一个随机key

> ```
> randomKey
> ```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130548.png)

#### 21.手动持久化操作

> `save` 阻塞

- SAVE 直接调用 rdbSave ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

> `bgsave` 非阻塞

- BGSAVE 则 fork 出一个子进程，子进程负责调用 rdbSave ，并在保存完成之后向主进程发送信号，通知保存已完成。 Redis 服务器在BGSAVE 执行期间仍然可以继续处理客户端的请求。

| 命令   | save               | bgsave                             |
| ------ | ------------------ | ---------------------------------- |
| IO类型 | 同步               | 异步                               |
| 阻塞？ | 是                 | 是（阻塞发生在fock()，通常非常快） |
| 复杂度 | O(n)               | O(n)                               |
| 优点   | 不会消耗额外的内存 | 不阻塞客户端命令                   |
| 缺点   | 阻塞客户端命令     | 需要fock子进程，消耗内存           |

#### 22.获取配置文件中用户密码

> ```
> config get requirepass
> ```

#### 23.设置配置文件中用户密码（临时，重启服务失效）

> ```
> config set requirepass password
> ```

#### 24.密码认证

> `auth password` 设置密码后必须认证才能使用客户端功能

#### 25.关闭redis服务

> ```
> shutdown
> ```

#### 26.**查看rdb文件存放的目录**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130556.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130600.png)

#### 27.查看服务器的信息

> `info` 参数

参数列表：

- `server`: Redis服务器的一般信息
- `clients`: 客户端的连接部分
- `memory`: 内存消耗相关信息
- `persistence`: RDB和AOF相关信息
- `stats`: 一般统计
- `replication`: 主/从复制信息
- `cpu`: 统计CPU的消耗
- `commandstats`: Redis命令统计
- `cluster`: Redis集群信息
- `keyspace`: 数据库的相关统计

它也可以采取以下值:

- `all`: 返回所有信息
- `default`: 值返回默认设置的信息

如果没有使用任何参数时，默认为`default`。

## 三、五大数据类型

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作<mark>数据库</mark>、<mark>缓存</mark>和<mark>消息中间件</mark>。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

<mark>单点登录、</mark>

### String（字符串）

略，具体看前面

#### 使用场景：

1. 计数器
2. 统计多单位的数量（uid:122:follow 10）
3. 粉丝数
4. 对象缓存存储

------

### List（列表）

#### 1.从头部/尾部<mark>插入</mark>数据，以及数据<mark>显示</mark>

> `lpush` key value
>
> `rpush` key value
>
> `lrange` key 0 -1

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130607.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130610.png)

#### 2.从头部/尾部<mark>移除</mark>数据

> `lpop` key
>
> `rpop` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130614.png)

#### 3.<mark>获取</mark>指定索引的值

> `lindex` key index

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130617.png)

#### 4.获取列表长度

> `llen` key

![image-20201112171308314](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130623.png)

#### 5.移除指定的值

> `lrem` key count(移除的个数) element

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130627.png)

#### 6.列表修剪 trim

> `ltrim` key startIndex endIndex

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130632.png)

#### 7.也可以使用set

> `lset` key index value // 将列表中 指定index的值替换为对应的value

- 需要保证key和index都存在，否则报错

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130638.png)

#### 8.插入指定的值

> `linsert` key before|after pivot(那个单词后) value

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130644.png)

#### 9.复杂操作

##### 1）rpoplpush

> `rpoplpush` source destination(newList) // 先移除source最后的元素，再将该元素添加到newList中

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130703.png)

#### 使用场景：

1. 栈（lpush、lpop）
2. 队列（lpush、rpop）
   1. 消息队列
3. 阻塞队列

### Set（集合）

> 集合中的值不能重复（无序）

#### 1.添加成员到集合中，并查看所有成员

> `sadd` key member
>
> `smembers` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130707.png)

#### 2.判定成员是否存在

> `sismember` key member

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130711.png)

#### 3.查看集合长度（特别）

> `scard` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130716.png)

#### 4.移除指定的成员

> `srem` key member

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130719.png)

#### 5.获取集合中的随机成员

> `srandmember` key [count]

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130722.png)

#### 6.随机移除成员

> `spop` key [count]

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130726.png)

#### 7.移动集合成员到其他集合

> `smove` source destination member(需要移动的成员)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130730.png)

#### 8.数字集合类：

- 差集 `sdiff` key1 ，key2 …
- 交集（共同好友）`sinter` key1，key2 …
- 并集 `sunion` key1 ，key2 …

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130734.png)

### Hash（哈希）

> key-Map or key-\<k,v>，value是一个Map
>
> Hash本质和hash没有区别，只是value变成了Map
>
> 用户信息保存，经常变动的信息，适合对象的存储

#### 1.简单存储Map和获取Map

> `hset` key field value [k1 v1 [k2 v2 …]]
>
> `hget` key field

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130741.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130745.png)

#### 2.获取所有Map字段及值

> `hgetall` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130748.png)

#### 3.删除Map中的字段

> `hdel` key field

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130753.png)

#### 4.查看Map中某字段是否存在

> `hexists` key field

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130758.png)

#### 5.获取所有字段或者所有字段对应的值

> `hkeys` key // 此处的key是Map名
>
> `hvals` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130801.png)

#### 6.增量i++

> `hincrby` key field value

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130805.png)

#### 7.不存在，就添加成功

> `hsetnx` key field value // field 在map中不存在就添加这个值，否则不做改变

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130808.png)

#### 8.适合存储对象

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130813.png)

### Zset（有序集合）

> 在Set基础上增加了一个值（用于排序的值）
>
> 存储班级成绩表，工资表排序，
>
> 普通消息 = 1，重要消息 = 2，带权重进行判断
>
> 排行榜应用实现

#### 1.添加 和 获取

> `zadd` key n value
>
> `zrange` key startIndex endIndex

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130816.png)

#### 2.排序实现（升序和降序）

> `zrangebyscore` key -inf +inf [withscores] // 升序
>
> `zrange` key 0 -1
>
> `zrevrangebyscore` key +inf -inf [withscores] // 降序
>
> `zrevrange` key 0 -1
>
> `zrangebyscore` key -inf 任意值n // 升序 + 显示区间 [-inf，n]

#### 3.移除指定的值

> `zrem` key value

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130821.png)

#### 4.集合的长度

> `zcard` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130826.png)

#### 5.指定区间的集合长度

> `zcount` key startIndex endIndex

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130829.png)

## 四、三种特殊数据类型

### Geospatial

> 朋友的定位，附近的人，打车距离计算
>
> 底层是 `Zset`，**即可以使用Zset的命令操作Geospatial**

```
Redis3.2就支持了
```

#### 1.添加地理位置

- 两极无法添加
- 经度：-180 ~ 180（度）
- 纬度：-85.05112878 ~ 85.05112878（度）

> `geoadd` key 经度 纬度 名称

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130835.png)

#### 2.获取指定位置的地理位置

> `geopos` key 名称

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130839.png)

#### 3.返回两个给定位置之间的距离（直线距离）

- 单位：
  - m ：米
  - km ：千米
  - mi ： 英里
  - ft ：英尺

> `geodist` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130844.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130848.png)

#### 4.以给定值为半径，以经度和维度为中心，查找

- 附近的人（获得所有附近的人的地址（开启定位））通过半径查询

> `georadius` key 经度 纬度 半径 单位

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130852.png)

#### 5.以给定值为半径，以成员(城市名)为中心，查找

> `georadiusbymember` key 成员名 半径 单位

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130856.png)

#### 6.返回一个或多个位置元素的geohash表示

- 如果两个字符串越相似，表示两个地方越近

> `geohash` key 成员1 成员2

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130900.png)

### Hyperloglog

> 基数统计的算法

- 优点
  - 占用内存是**固定的**，2<sup>64</sup>不同的元素的基数，只需要12KB的内存。（大数据情况下，有0.81%错误率）

> 基数：集合中元素的个数（先去重），如{1,2,2,3} 其基数为3（集合去重后为1,2,3 有3个元素）
>
> 网页的UV（一个人访问访问一个网站多次，但是还是算作一个人）

传统实现UV：Set保存用户的Id，然后统计set中的元素的数量作为标准判断（这种需要保存大量用户的ID）

```
Redis2.8.9
```

##### 1.测试

> 创建一组元素 ： `pfadd` key ele1 ele2 ele3 …
>
> 统计对应key的基数：`pfcount` key1 [key2 …] // 多个key 就是统计这些key并集的基数
>
> 合并：`pfmerge` destkey sourceKey1 sourceKey2 [sourceKey3 …]

![image-20201115213230322](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130905.png)

### Bitmaps

> 位存储，位图（操作二进制）

统计用户信息，活跃，不活跃！登录、未登录！打卡，365打卡！(两个状态都可以使用)

#### 1.案例：一周打卡记录

**一周过去**

> `setbit` key offset bit

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130909.png)

**查看单天打卡情况**

> `getbit` key offset

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130914.png)

**统计所有打卡的天数**

> `bitcount` key

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130919.png)

## 五、事务

> 注意：Redis单条命令是保证原子性的；<mark>**但是事务不保证原子性！**</mark>
>
> <mark>**Redis事务没有隔离级别的概念**，所有的命令在事务中，并没有直接被执行，只有发起执行命令时才执行</mark>

**Redis事务本质：**一组命令的集合，一个事务中的所有命令都会别序列化，在事务执行过程中，会按照顺序执行！

```
--- 队列 set set set 执行 ---
```

一次性、顺序性、排他性！执行一些列的命令

**Redis的事务：**

- 开启事务（`multi`）
- 命令入队（`其他命令`）
- 执行事务（`exec`）

> ### 正常事务的执行

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130924.png)

> ### 取消事务

```
discard
```

![、](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130927.png)

> ### 异常执行

#### 1.编译时（命令写错）

> **整个命令队列都不会执行**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130931.png)

#### 2.运行时

> **报错语句，会抛出异常；其他语句照样运行**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130935.png)

### 监控 Watch（面试常问）

#### 乐观锁：实现秒杀

- 顾名思义，很乐观，认为什么时候都不会出现问题，所以不会加锁！（更新数据的时候去判断一下，在此期间是否有人修改过这个数据）
  - 获取version
  - 更新的时候比较version

#### 悲观锁

- 顾名思义，很悲观，认为什么时候都会出现问题，无论做什么都会加锁！

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130942.png)

#### Redis实现乐观锁

##### 执行成功（单线程没有干扰情况）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130946.png)

##### 测试多线程修改值，使用watch可以当作redis的<mark>乐观锁操作</mark>

**演示**‘

①开启俩个，客户端，模拟多线程情况

![image-20201116083639168](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119130959.png)

②左边支出20元（但是不执行事务），然后右边修改money的数值

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131006.png)

③左边执行事务，发现执行操作返回nil，查看money和out，发现事务并没有被执行（确实有乐观锁的效果）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131010.png)

**如果修改失败获取最新的值就好（<mark>`exec`、`unwatch`、``discard`都可以清除连接时所有的监视）</mark>**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131014.png)

##### 小结

- 使用Redis实现乐观锁（watch监听某一个key，获取其最新的value）
  - 在提交事务时，如果key的value没有发生变化，则成功执行
  - 在提交事务时，如果key的value发生了变化，则无法成功执行

## 六、Jedis

### 1、什么是Jedis

> Jedis是一个用java写的Redis数据库操作的客户端，通过Jedis，可以很方便的对redis数据库进行操作

### 2、使用

①新建空maven项目

②导入依赖

```
<!--导入jedis包 Redis客户端--><dependency>    <groupId>redis.clients</groupId>    <artifactId>jedis</artifactId>    <version>3.3.0</version></dependency><!--导入fastjson--><dependency>    <groupId>com.alibaba</groupId>    <artifactId>fastjson</artifactId>    <version>1.2.70</version></dependency>
```

③编码

> 下面使用远程连接需要的操作：https://blog.csdn.net/weixin_43423864/article/details/109087670

1. **连接数据库**

```
public class PingTest {    public static void main(String[] args) {        // 1. new Jedis 对象        Jedis jedis = new Jedis("ip地址",6379);        // 如果设置密码 需要认证，没有设置忽略下面这条语句        jedis.auth("密码");        /// jedis 所有的命令(方法)都是之前学的命令        System.out.println(jedis.ping());// 测试连接    }}
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131021.png)

1. **操作**

   ```
   // 连通Jedis jedis = new Jedis("ip地址", 6379);jedis.auth("密码");//基本操作System.out.println("清空数据：" + jedis.flushAll());System.out.println("判断key（name）是否存在：" + jedis.exists("name"));System.out.println("设置name的value：" + jedis.setnx("name", "liuyou"));System.out.println("设置pwd的value：" + jedis.setnx("pwd", "密码"));System.out.println("打印所有的key：" + jedis.keys("*"));System.out.println("获取该name的value：" + jedis.get("name"));System.out.println("删除pwd：" + jedis.del("pwd"));System.out.println("重命名name为username：" + jedis.rename("name", "username"));System.out.println("打印所有的key：" + jedis.keys("*"));System.out.println("返回当前数据库中key的数目：" + jedis.dbSize());
   ```

   ```
   清空数据：OK判断key（name）是否存在：false设置name的value：1设置pwd的value：1打印所有的key：[pwd, name]获取该name的value：liuyou删除pwd：1重命名name为username：OK打印所有的key：[username]返回当前数据库中key的数目：1
   ```

2. **关闭连接**

```
jedis.close()
```

1. **事务**

```
// 连通Jedis jedis = new Jedis("IP地址", 6379);jedis.auth("密码");JSONObject jsonObject = new JSONObject();jsonObject.put("hello","world");jsonObject.put("name","liuyou");jsonObject.put("pwd","密码");String s = jsonObject.toJSONString();jedis.flushAll();/// 加监听 watch// jedis.watch("user");// 开启事务Transaction multi = jedis.multi();try {    multi.set("user",s);    // 其他语句    // 执行事务    multi.exec();} catch (Exception e) {    // 取消事务    multi.discard();    e.printStackTrace();} finally {    System.out.println(jedis.get("user"));    // 关闭连接    jedis.close();}
```

## 七、SpringBoot整合

> 在SpringBoot2.x之后，原来的jedis被替换为lettuce

- jedis
  - 采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全，使用jedis pool连接池！BIO
- lettuce
  - 采用netty，实例可以在多个线程**享，不存在线程不安全问题，可以减少线程数据了，性能高，NIO

#### ①新建springboot项目

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131030.png)

#### ② `RedisAutoConfiguration`源码分析

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119205923.png)

#### ③整合测试

1、导入依赖

```
<dependency>    <groupId>org.springframework.boot</groupId>    <artifactId>spring-boot-starter-data-redis</artifactId></dependency>
```

2、配置Redis（application.xml）

```
# SpringBoot 整合Redisspring.redis.host=xxxspring.redis.port=6379spring.redis.password=liuyou
```

3、编写测试类

```
@SpringBootTestclass Redis02SpringbootApplicationTests {    @Autowired    RedisTemplate redisTemplate;    @Test    void contextLoads() {        //redisTemplate        // 1.使用redisTemplate.opsForxxx 操作对应的数据结构        // 2.可使用redisTemplate 进行简单的key操作，如multi、move、watch、keys 等操作        /// 3.可使用获取连接，通过连接进行更多操作           // RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();           // RedisZSetCommands redisZSetCommands = connection.zSetCommands();        // 这里只使用1.演示        ValueOperations str = redisTemplate.opsForValue();        str.set("name","liuminkai刘民锴");        System.out.println(str.get("name"));    }}
```

![结果展示](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131040.png)

> 但是存在问题，没有序列化，存储在Redis中的中文会被转义，如下图

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131044.png)

> 为啥会出现这个情况？ RedisTemplate默认序列化使用JDK的，我们需要使用JSON格式

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131048.png)

> 如何解决？我们需要编写自定义Redis配置类，自定义RedisTemplate

#### ④自定义RedisTemplate

```
@Configurationpublic class MyRedisConfig {    // 更改Key ：Object ==> String 符合日常使用    // 自己定义了一个 RedisTemplate    @Bean    @SuppressWarnings("all")    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {        // 我们为了自己开发方便，一般直接使用 <String, Object>        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();        template.setConnectionFactory(factory);        // Json序列化配置        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);        ObjectMapper om = new ObjectMapper();        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);        jackson2JsonRedisSerializer.setObjectMapper(om);        // String 的序列化        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();        // key采用String的序列化方式        template.setKeySerializer(stringRedisSerializer);        // hash的key也采用String的序列化方式        template.setHashKeySerializer(stringRedisSerializer);        // value序列化方式采用jackson        template.setValueSerializer(jackson2JsonRedisSerializer);        // hash的value序列化方式采用jackson        template.setHashValueSerializer(jackson2JsonRedisSerializer);        template.afterPropertiesSet();        return template;    }}
```

> 自定义RedisTemplate后，再启动③的测试类，结果Redis中正常显示

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131054.png)

#### ⑤<mark>RedisUtils工具类</mark>

> 日常开发我们都不会使用原始的`RedisTemplate`，都会封装一个RedisUtils工具类便于使用

```
package com.liuyou.utils;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.data.redis.core.RedisTemplate;import org.springframework.stereotype.Component;import org.springframework.util.CollectionUtils;import java.util.Map;import java.util.Set;import java.util.List;import java.util.concurrent.TimeUnit;@Componentpublic final class RedisUtils {    @Autowired    private RedisTemplate<String, Object> redisTemplate;    // =============================common============================    /**     * 指定缓存失效时间     * @param key  键     * @param time 时间(秒)     */    public boolean expire(String key, long time) {        try {            if (time > 0) {                redisTemplate.expire(key, time, TimeUnit.SECONDS);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 根据key 获取过期时间     * @param key 键 不能为null     * @return 时间(秒) 返回0代表为永久有效     */    public long getExpire(String key) {        return redisTemplate.getExpire(key, TimeUnit.SECONDS);    }    /**     * 判断key是否存在     * @param key 键     * @return true 存在 false不存在     */    public boolean hasKey(String key) {        try {            return redisTemplate.hasKey(key);        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 删除缓存     * @param key 可以传一个值 或多个     */    @SuppressWarnings("unchecked")    public void del(String... key) {        if (key != null && key.length > 0) {            if (key.length == 1) {                redisTemplate.delete(key[0]);            } else {                redisTemplate.delete(CollectionUtils.arrayToList(key));            }        }    }    // ============================String=============================    /**     * 普通缓存获取     * @param key 键     * @return 值     */    public Object get(String key) {        return key == null ? null : redisTemplate.opsForValue().get(key);    }    /**     * 普通缓存放入     * @param key   键     * @param value 值     * @return true成功 false失败     */    public boolean set(String key, Object value) {        try {            redisTemplate.opsForValue().set(key, value);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 普通缓存放入并设置时间     * @param key   键     * @param value 值     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期     * @return true成功 false 失败     */    public boolean set(String key, Object value, long time) {        try {            if (time > 0) {                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);            } else {                set(key, value);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 递增     * @param key   键     * @param delta 要增加几(大于0)     */    public long incr(String key, long delta) {        if (delta < 0) {            throw new RuntimeException("递增因子必须大于0");        }        return redisTemplate.opsForValue().increment(key, delta);    }    /**     * 递减     * @param key   键     * @param delta 要减少几(小于0)     */    public long decr(String key, long delta) {        if (delta < 0) {            throw new RuntimeException("递减因子必须大于0");        }        return redisTemplate.opsForValue().increment(key, -delta);    }    // ================================Map=================================    /**     * HashGet     * @param key  键 不能为null     * @param item 项 不能为null     */    public Object hget(String key, String item) {        return redisTemplate.opsForHash().get(key, item);    }    /**     * 获取hashKey对应的所有键值     * @param key 键     * @return 对应的多个键值     */    public Map<Object, Object> hmget(String key) {        return redisTemplate.opsForHash().entries(key);    }    /**     * HashSet     * @param key 键     * @param map 对应多个键值     */    public boolean hmset(String key, Map<String, Object> map) {        try {            redisTemplate.opsForHash().putAll(key, map);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * HashSet 并设置时间     * @param key  键     * @param map  对应多个键值     * @param time 时间(秒)     * @return true成功 false失败     */    public boolean hmset(String key, Map<String, Object> map, long time) {        try {            redisTemplate.opsForHash().putAll(key, map);            if (time > 0) {                expire(key, time);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 向一张hash表中放入数据,如果不存在将创建     *     * @param key   键     * @param item  项     * @param value 值     * @return true 成功 false失败     */    public boolean hset(String key, String item, Object value) {        try {            redisTemplate.opsForHash().put(key, item, value);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 向一张hash表中放入数据,如果不存在将创建     *     * @param key   键     * @param item  项     * @param value 值     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间     * @return true 成功 false失败     */    public boolean hset(String key, String item, Object value, long time) {        try {            redisTemplate.opsForHash().put(key, item, value);            if (time > 0) {                expire(key, time);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 删除hash表中的值     *     * @param key  键 不能为null     * @param item 项 可以使多个 不能为null     */    public void hdel(String key, Object... item) {        redisTemplate.opsForHash().delete(key, item);    }    /**     * 判断hash表中是否有该项的值     *     * @param key  键 不能为null     * @param item 项 不能为null     * @return true 存在 false不存在     */    public boolean hHasKey(String key, String item) {        return redisTemplate.opsForHash().hasKey(key, item);    }    /**     * hash递增 如果不存在,就会创建一个 并把新增后的值返回     *     * @param key  键     * @param item 项     * @param by   要增加几(大于0)     */    public double hincr(String key, String item, double by) {        return redisTemplate.opsForHash().increment(key, item, by);    }    /**     * hash递减     *     * @param key  键     * @param item 项     * @param by   要减少记(小于0)     */    public double hdecr(String key, String item, double by) {        return redisTemplate.opsForHash().increment(key, item, -by);    }    // ============================set=============================    /**     * 根据key获取Set中的所有值     * @param key 键     */    public Set<Object> sGet(String key) {        try {            return redisTemplate.opsForSet().members(key);        } catch (Exception e) {            e.printStackTrace();            return null;        }    }    /**     * 根据value从一个set中查询,是否存在     *     * @param key   键     * @param value 值     * @return true 存在 false不存在     */    public boolean sHasKey(String key, Object value) {        try {            return redisTemplate.opsForSet().isMember(key, value);        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 将数据放入set缓存     *     * @param key    键     * @param values 值 可以是多个     * @return 成功个数     */    public long sSet(String key, Object... values) {        try {            return redisTemplate.opsForSet().add(key, values);        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }    /**     * 将set数据放入缓存     *     * @param key    键     * @param time   时间(秒)     * @param values 值 可以是多个     * @return 成功个数     */    public long sSetAndTime(String key, long time, Object... values) {        try {            Long count = redisTemplate.opsForSet().add(key, values);            if (time > 0){                expire(key, time);            }            return count;        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }    /**     * 获取set缓存的长度     *     * @param key 键     */    public long sGetSetSize(String key) {        try {            return redisTemplate.opsForSet().size(key);        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }    /**     * 移除值为value的     *     * @param key    键     * @param values 值 可以是多个     * @return 移除的个数     */    public long setRemove(String key, Object... values) {        try {            Long count = redisTemplate.opsForSet().remove(key, values);            return count;        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }    // ===============================list=================================    /**     * 获取list缓存的内容     *     * @param key   键     * @param start 开始     * @param end   结束 0 到 -1代表所有值     */    public List<Object> lGet(String key, long start, long end) {        try {            return redisTemplate.opsForList().range(key, start, end);        } catch (Exception e) {            e.printStackTrace();            return null;        }    }    /**     * 获取list缓存的长度     *     * @param key 键     */    public long lGetListSize(String key) {        try {            return redisTemplate.opsForList().size(key);        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }    /**     * 通过索引 获取list中的值     *     * @param key   键     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推     */    public Object lGetIndex(String key, long index) {        try {            return redisTemplate.opsForList().index(key, index);        } catch (Exception e) {            e.printStackTrace();            return null;        }    }    /**     * 将list放入缓存     *     * @param key   键     * @param value 值     */    public boolean lSet(String key, Object value) {        try {            redisTemplate.opsForList().rightPush(key, value);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 将list放入缓存     * @param key   键     * @param value 值     * @param time  时间(秒)     */    public boolean lSet(String key, Object value, long time) {        try {            redisTemplate.opsForList().rightPush(key, value);            if (time > 0){                expire(key, time);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 将list放入缓存     *     * @param key   键     * @param value 值     * @return     */    public boolean lSet(String key, List<Object> value) {        try {            redisTemplate.opsForList().rightPushAll(key, value);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 将list放入缓存     *     * @param key   键     * @param value 值     * @param time  时间(秒)     * @return     */    public boolean lSet(String key, List<Object> value, long time) {        try {            redisTemplate.opsForList().rightPushAll(key, value);            if (time > 0){                expire(key, time);            }            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 根据索引修改list中的某条数据     *     * @param key   键     * @param index 索引     * @param value 值     * @return     */    public boolean lUpdateIndex(String key, long index, Object value) {        try {            redisTemplate.opsForList().set(key, index, value);            return true;        } catch (Exception e) {            e.printStackTrace();            return false;        }    }    /**     * 移除N个值为value     *     * @param key   键     * @param count 移除多少个     * @param value 值     * @return 移除的个数     */    public long lRemove(String key, long count, Object value) {        try {            Long remove = redisTemplate.opsForList().remove(key, count, value);            return remove;        } catch (Exception e) {            e.printStackTrace();            return 0;        }    }}
```

> 使用该工具类直接`@Autowired`注入即可

## 进阶

> 学会前面的所有内容 ==> Redis基础
>
> 高级操作都在后面

## 八、Redis.conf详解

> 我们启动Redis，一般都是通过Redis.conf启动（我前面是使用安装服务的6379.conf启动）
>
> 因此，我们必须了解Redis.conf的配置，才能更好理解和使用Redis
>
> ### 单位

- **大小写不敏感**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131119.png)

> ### 包含

- 可以包含多个配置文件（即，这些文件导入到主配置文件 `Redis.conf` 中）

  ![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131123.png)

> ### 网络

```
bind 0.0.0.0 # IP（默认127.0.0.1）protected-mode no # 保护模式（默认yes）port 6379 # 端口设置（默认6379）
```

> ### 通用

```
daemonize yes # 以守护进程方式运行，即后台运行（默认no）pidfile /var/run/redis_6379.pid # 如果以后台运行，必须指定一个pid文件# 日志# Specify the server verbosity level.# This can be one of:# debug (大量信息, 使用于测试或开发阶段)# verbose (许多很少有用的信息，但不像调试级别那样混乱)# notice (比较冗长，你可能想在生产环境中使用)# warning (只有非常重要/关键的消息被记录下来)loglevel notice  # 默认noticelogfile "" # 日志的文件位置名databases 16 # 数据库的数量（默认16）always-show-logo yes # 是否开启 logo （默认yes）
```

> ### 快照

持久化，在规定时间内，执行了多少次操作，会被持久化到文件(.rdb，.aof)

```
save 900 1 # 900秒内（15分钟），如果至少有1个Key进行修改，我们就进行持久化操作save 300 10 # 300秒内（5分钟），如果至少有10个Key进行修改，我们就进行持久化操作save 60 10000 # 60秒内（1分钟），如果至少有10000个Key进行修改，我们就进行持久化操作stop-writes-on-bgsave-error yes # 持久化如果出错，是否还需要继续工作（默认yes）rdbcompression yes # 是否压缩rdb文件（默认yes），会消耗一些CPU资源rdbchecksum yes # 保存rdb文件时，进行错误检查检验dir ./ # rdb文件保存的目录
```

> #### 主从复制 REPLICATION
>
> #### 安全 SECURITY

```
requirepass 你的密码 # 设置密码（默认被注释着需要自己解开注释）
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131144.png)

**当然可以通过命令行配置(临时，服务重启失效)**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131149.png)

> #### 客户端限制

```
# maxclients 10000 # 限制最多10000个客户端访问（默认注释）
```

> #### 内存管理

```
# maxmemory <bytes> # 最大内存设置（默认注释）# maxmemory-policy noeviction # 内存达到上限之后的处理策略（默认noeviction）    # 1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）     # 2、allkeys-lru ： 删除lru算法的key       # 3、volatile-random：随机删除即将过期key       # 4、allkeys-random：随机删除       # 5、volatile-ttl ： 删除即将过期的       # 6、noeviction ： 永不过期，返回错误
```

> ### APPEND ONLY MODE (AOF配置)

```
appendonly no # 默认是不开启aof的，默认使用rdb方式持久化appendfilename "appendonly.aof" # 持久化的文件名# appendfsync always # 每次修改都会同步，销耗性能appendfsync everysec # 每秒执行一次同步，可能会丢失这1秒的数据# appendfsync no # 不同步，操作系统自己同步数据，速度最快
```

## 九、Redis持久化（重点）

> Redis是内存数据库，数据库状态断电及失，因此Redis提供了持久化功能（内存数据写入磁盘）

### 1、RDB（Redis DataBase）

> RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。 保存文件格式 `dump.rdb`

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131158.png)

![img](https://www.kuangstudy.com/bbs/2020-11-2-Redis%E8%AF%A6%E7%BB%86%E7%AC%94%E8%AE%B0.assets/image-20201117102650803.png)

#### 设置RDB文件保存条件

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131204.png)

**重启服务**

```
systemctl restart redis_6379
```

##### 结果测试

**1.查看rdb文件存放的目录**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131209.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131213.png)

**2.删除已有的dump.rdb文件**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131218.png)

**3.添加5个key**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131223.png)

![发现重新生成rdb文件](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131228.png)

#### **RDB文件生成触发机制**

1.满足redis.conf中，快照save生成条件

2.使用flushall命令，自动生成一个RDB文件

3.退出redis

#### RDB文件恢复

> 只需要将RDB文件放入Redis启动目录就可以了，Redis自动加载

#### 优缺点

- 优点：
  - 适合大规模的数据恢复！
  - 对数据完整性要求不高
- 缺点：
  - 需要一定的时间间隔进程操作；如果redis意外宕机，最后一次修改数据就没有了
  - fork进程的时候，会占用一定的空间

### 2、AOF（Append Only File）

> AOF持久化以**日志**的形式**记录服务器所处理的每一个写、删除操作**，**查询操作不会记录**，<mark>只许追加文件，不许改写文件</mark>。以文本的方式记录，可以打开文件看到详细的操作记录。
>
> 保存文件格式 `appendonly.aof`

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131235.png)

**aof默认不开启，需要到配置文件中开启**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131240.png)

##### 重启redis后，appendonly.aof文件自动生成

**客户端进行一些操作**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131247.png)

**appendonly.aof文件内容，日志形式记录**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131252.png)

##### 重写规制

> 如果aof文件大于64mb，就会fork一个新进程来将我们的文件进行重写（清除之前的64mb）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131257.png)

##### 错误修复

如果aof文件有错误，redis是启动不了的，可以使用官方自带 `redis-check-aof --fix aof文件`进行修复

##### 优缺点

- 优点
  - 每一次修改都同步，文件完整性会更好
  - 每秒同步一次，可能会丢失一秒数据
- 缺点
  - 相对于数据文件来说，aof远远大于rdb，修复速度比rdb慢
  - aof运行速率比rdb慢（追加，频繁IO操作）

### 扩展

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131303.png)

## 十、Redis发布订阅

Redis 发布订阅(pub/sub)是一种**消息通信模式**：发送者(pub)发送消息，订阅者(sub)接收消息。微信、微博、关注系统！

Redis客户端可以订阅任意数量的频道

**订阅/发布消息图：**

![订阅/发布消息图](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131309.png)

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131317.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131322.png)

### 命令

这些命令被广泛用于构建即时通信应用，比如网络聊天室（chatroom）和实时广播、实时提醒等。

| 命令                                     | 描述                               |
| ---------------------------------------- | ---------------------------------- |
| `PSUBSCRIBE pattern [pattern..]`         | 订阅一个或多个符合给定模式的频道。 |
| `PUNSUBSCRIBE pattern [pattern..]`       | 退订一个或多个符合给定模式的频道。 |
| `PUBSUB subcommand [argument[argument]]` | 查看订阅与发布系统状态。           |
| `PUBLISH channel message`                | 向指定频道发布消息                 |
| `SUBSCRIBE channel [channel..]`          | 订阅给定的一个或多个频道。         |
| `SUBSCRIBE channel [channel..]`          | 退订一个或多个频道                 |

### 测试

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131328.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131335.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131340.png)

```
127.0.0.1:6379> subscribe blog # 订阅频道Reading messages... (press Ctrl-C to quit) # 等待推送信息1) "subscribe"2) "blog"3) (integer) 11) "message" # 消息2) "blog" # 消息来自频道3) "hello world!" # 消息内容127.0.0.1:6379> publish blog "hello world!" # 发送消息到频道(integer) 1127.0.0.1:6379>
```

### 原理

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131347.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131412.png)

每个 Redis 服务器进程都维持着一个表示服务器状态的 redis.h/redisServer 结构， 结构的 pubsub_channels 属性是一个字典， 这个字典就用于保存订阅频道的信息，其中，**字典的键为正在被订阅的频道**， 而**字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端**。

客户端订阅，就被链接到对应频道的链表的尾部，退订则就是将客户端节点从链表中移除。

### 使用场景：

- 实时消息系统！
- 实时聊天！(频道当作聊天室，将信息回显给所有人)
- 订阅，关注系统都是可以

> 复杂的情况，使用专业的消息中间件

### 做订阅的缺点

1. 如果一个客户端订阅了频道，但自己读取消息的速度却不够快的话，那么不断积压的消息会使redis输出缓冲区的体积变得越来越大，这可能使得redis本身的速度变慢，甚至直接崩溃。
2. 这和数据传输可靠性有关，如果在订阅方断线，那么他将会丢失所有在短线期间发布者发布的消息。

## 十一、主从复制

### 概念

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点（Master/Leader）,后者称为从节点（Slave/Follower）， 数据的复制是单向的！**只能由主节点复制到从节点**（主节点以写为主、从节点以读为主）。

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131424.png)

### 主从复制的主要作用

1. **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余的方式。
2. **故障恢复**：当主节点故障时，从节点可以暂时替代主节点提供服务，是一种服务冗余的方式
3. **负载均衡**：在主从复制的基础上，配合读写分离，由主节点进行写操作，从节点进行读操作，分担服务器的负载；尤其是在多读少写的场景下，通过多个从节点分担负载，提高并发量。
4. **高可用基石**：主从复制还是哨兵和集群能够实施的基础。

一般来说，要将Redis运用与工程项目中，只使用一台Redis是万万不能的（避免宕机，一主二从），原因如下：

1. 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大
2. 从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，**单台服务器最大使用内存不超过20G。**

> 只要在公司中，不可能使用单机（有瓶颈），必须配置集群、使用主从复制

### 环境配置

<mark>只需配置从库，无需配置主库</mark>

**查看主从复制信息：**

> 默认情况下，每一个Redis服务器都是主节点

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131432.png)

```
127.0.0.1:6379> info replication# Replicationrole:master  # 主节点connected_slaves:0  # 没有从节点master_replid:7776dea8df483b02d12cd482d2034ba55ec7dab0master_replid2:0000000000000000000000000000000000000000master_repl_offset:0second_repl_offset:-1repl_backlog_active:0repl_backlog_size:1048576repl_backlog_first_byte_offset:0repl_backlog_histlen:0
```

#### 1、复制配置文件redis.conf到Redis安装目录下

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131438.png)

#### 2、再从该文件拷贝出3个文件

**主：**`redis_6379.conf`

**从：**`redis_6380.conf、redis_6381.conf`

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131447.png)

#### 3、修改主配置文件

> 由于端口和后台默认运行pid都是6379的，就不改了

![image-20201117125611885](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131452.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131455.png)

#### 4、修改从配置文件

> 下面以 `redis_6380.conf` 为例， `redis_6381.conf` 同理

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131500.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131504.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131508.png)

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131512.png)

#### 配置文件修改的信息

1. 端口（92行）
2. pid进程名（244行）
3. 日志文件名（257行）
4. rdb文件名（339行）

#### 5、启动服务（单机多服务）

```
cd /usr/local/bin # 进入配置文件所在目录[root@liuyou bin]# redis-server redis_6379.conf[root@liuyou bin]# redis-server redis_6380.conf[root@liuyou bin]# redis-server redis_6381.conf[root@liuyou bin]# ps -ef | grep redis  # 查看启动状态root      2862     1  0 13:13 ?        00:00:00 redis-server 0.0.0.0:6379root      2868     1  0 13:13 ?        00:00:00 redis-server 0.0.0.0:6380root      2874     1  0 13:13 ?        00:00:00 redis-server 0.0.0.0:6381root      2880  2393  0 13:13 pts/0    00:00:00 grep --color=auto redis
```

#### 6、登录客户端

①开启4个窗口，前三用于主从复制，最后一个用于测试

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131520.png)

②登录（注意端口）

![窗口1](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131528.png)

![窗口2](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131535.png)

![窗口3](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131542.png)

#### 7、一主二从

<mark>默认情况下，每一个Redis服务器都是主节点</mark>

**一般情况下，只配置从机**

> 如果有密码，需要在从配置文件中，进行配置，后面会解释
>
> 下面以 窗口3（端口为6381） 为例，窗口2同理

##### **使用slaveof 指定 主节点 ip 和 端口（临时配置）**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131549.png)

**查看主节点，主从复制信息**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131554.png)

##### 有密码配置（+永久配置）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131609.png)

> 配置完重启服务

##### 测试

> Redis只允许，主机写，从机读

**主机写**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131615.png)

**从机读**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131621.png)

**从机不能写**

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131626.png)

> **主机断开连接**，从机依旧连接主机，但是没有写操作，如果从机中途恢复正常，从机依旧可以获取主机写的内容
>
> 如果使用命令行配置，从机重启，就会变为主机（只有变为从机，才能获取之前主机内容）
>
> **从机断开连接**，期间，主机继续写新内容，只要从机恢复，就可获得主机写的新内容

##### 复制原理

Slave 启动成功连接到Master后会发送一个sync命令

Master接收到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，Master将传送整个数据文件到Slave，并完成一次完全同步

**全量复制**：而Slave服务在接收到数据库文件数据后，将其存盘并加载到内存中

**增量复制**：Master继续将新的所有收集到修改命令依次传给Slave，完成同步

> 但是只要重新连接Master，一次完全同步（全量复制）将被自动执行。

#### 8、毛毛虫配置（主从）

> 上一个M连接下一个S，（可以看做主从复制的一种实现）
>
> 配置只需：将上面配好的，窗口3的主机改为窗口2即可

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131637.png)

#### 9、宕机手动配置主机

```
slaveof no one # 如果主机断开连接，从机可以使用该命令，让自己变为主机，其他节点连到该节点
```

> 如果主机如果会来了，也没有从机连接上它。

## 十二、哨兵模式

> 自动选举Redis主服务器（如果主服务器宕机）

在 `Redis2.8`之前，采用手动配置主机的形式（会导致一段时间服务不可用）

`Redis2.8`之后，Redis正是提供了Sentinel（哨兵）来解决这个问题（**主机宕机，根据投票自动在从机中选出新主机**）

**哨兵模式**是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，他会独立运行**。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**

### 单哨兵模式

![单机哨兵](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131646.png)

### 哨兵的作用：

1. 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器
2. 当哨兵监测到master宕机，会自动将slave切换到master，然后通过 **发布订阅模式** 通知其他的从服务器，修改配置文件，让他们切换主机

### 多哨兵模式

![多机哨兵](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131654.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象称为**主观下线**。

当其他哨兵也检测到主服务器不可用，并且达到一定数量时，那么哨兵之间就会进行一次投票，投票的结果有一个哨兵发起，进行failover故障转移操作。

切换成功后，就会通过**发布订阅模式**，让各个哨兵把自己监控的从服务器实现切换主机，这个时候称为**客观下线**

### 测试（一主二从，单哨兵）

#### 1、配置哨兵配置文件（文件名 `sentinel.conf`）

> **文件名不能写错**，下面是文件内容（当然这个文件需要自己创建）

```
# sentinel monitor 被监控的名称 host port 1       # 1 表示 主机宕机，从机投票选举sentinel monitor myredis  127.0.0.1 6379 1
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131701.png)

#### 2、启动哨兵模式

> 如果被监控有密码 需在 `sentinel.conf` 配置文件中，追加
>
> ```
> # sentinel auth-pass 被监控主机名称  <password> sentinel auth-pass myredis  <password>
> ```

```
redis-sentinel sentinel.conf
```

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131708.png)

#### 3、关闭主机，测试选举情况

①主机关闭

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131716.png)

②哨兵详情

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131721.png)

③检查窗口3（端口为6381）

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131732.png)

④我们恢复窗口1（即之前的主机），看看情况

> 变成从机

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131738.png)

### 优缺点

- 优点：
  - 哨兵集群，基于主从复制模式，所有的主从配置优点，它全有
  - 主从可以切换，故障可以转移，系统的可用性就会更好
  - 哨兵模式就是主从模式的升级，手动到自动，更加健壮
- 缺点
  - Redis不好在线扩容的，集群容量一旦达到上线，在线扩容十分麻烦
  - 实现哨兵模式的配置其实是很麻烦的，里面有很多选择

### 哨兵模式的全部配置

```
# Example sentinel.conf# 哨兵sentinel实例运行的端口 默认26379  # 如果有哨兵集群 需要配置多个端口port 26379  # 哨兵sentinel的工作目录dir /tmp# 哨兵sentinel监控的redis主节点的 ip port # master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。# quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了# sentinel monitor <master-name> <ip> <redis-port> <quorum>sentinel monitor mymaster 127.0.0.1 6379 1# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码# sentinel auth-pass <master-name> <password>sentinel auth-pass mymaster MySUPER--secret-0123passw0rd# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒# sentinel down-after-milliseconds <master-name> <milliseconds>sentinel down-after-milliseconds mymaster 30000# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，# 这个数字越小，完成failover所需的时间就越长，# 但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。# 可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。# sentinel parallel-syncs <master-name> <numslaves>sentinel parallel-syncs mymaster 1# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： #1. 同一个sentinel对同一个master两次failover之间的间隔时间。#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。#3.当想要取消一个正在进行的failover所需要的时间。  #4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了# 默认三分钟# sentinel failover-timeout <master-name> <milliseconds>sentinel failover-timeout mymaster 180000# SCRIPTS EXECUTION#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。#对于脚本的运行结果有以下规则：#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，#这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，#一个是事件的类型，#一个是事件的描述。#如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。#通知脚本# sentinel notification-script <master-name> <script-path>  sentinel notification-script mymaster /var/redis/notify.sh# 客户端重新配置主节点参数脚本# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。# 以下参数将会在调用脚本时传给脚本:# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port># 目前<state>总是“failover”,# <role>是“leader”或者“observer”中的一个。 # 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的# 这个脚本应该是通用的，能被多次调用，不是针对性的。# sentinel client-reconfig-script <master-name> <script-path>sentinel client-reconfig-script mymaster /var/redis/reconfig.sh # 运维配置
```

## 十三、缓存穿透和雪崩（面试高频，工作常用）

> 这里仅仅是了解，不涉及解决方案底层

Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它带来了一些问题。其中最要害的问题，就是数据一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。另外的一些典型问题就是，**缓存穿透**、**缓存雪崩**和**缓存击穿**。目前，业界也都有比较流行的解决方案。

### 1、缓存穿透（查不到）

缓存穿透：用户想要查一个数据，发现Redis内存数据库中没有，也就是**缓存没命中，于是向持久层数据库查询。发现也没有，于是本次查询失败**。当用户很多时，缓存都没有命中（秒杀！），于是都去请求持久层数据库。这会导致持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

#### 解决方案

##### 布隆过滤器

布隆过滤器是一种数据结构，对所有可能查询的参数以Hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免存储系统的查询压力。

![img](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131751.png)

##### 缓存空对象

当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源

![image-20201118094132029](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131755.png)

**但是这个方法会存在两个问题**

- 如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键
- 即使空值设置了过期时间，还会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响

### 2、缓存雪崩（集中失效）

缓存雪崩，是指在某一时间段，缓存集中过期失效。Redis宕机！

**产生原因之一：**

比如，双十一零点抢购，会把同一批商品信息比较集中的放入缓存中，假设缓存设置一个小时的过期时间，那么到凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压抑波峰。于是所有的请求就会向存储层，存储层的调用量会暴增，**可能**造成存储层奔溃，服务器宕机。

![image-20201118094811319](https://liuyou-images.oss-cn-hangzhou.aliyuncs.com/markdown/20201119131802.png)

其实集中过期，倒不是非常致命，**比较致命的缓存雪崩，是缓存服务器某个结点宕机或断网**。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而**缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很可能瞬间就把数据库压垮。**

#### 解决方案

##### Redis高可用

这个思想含义是，既然Redis有可能挂到，那我们多增设几台Redis，这样挂掉之后其他的还可以继续工作，其实就是**搭建集群**

##### 限流降级

这个解决方案的思想是，在缓存失效后，通过加锁或队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

##### 数据预热

数据加热的含义就是在正式部署之前，我们先把数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中，在即将发生大并发访问**手动触发加载缓存不同的key**，设置不同的过期时间，让缓存失效的时间点尽量均匀。

### 3、缓存击穿（查询量太大，缓存过期瞬间）

这里需要注意和缓存穿透的区别，缓存击穿：是指一个key非常热点，在不停的扛着大并发，大并发集中对一个点进行访问，当这个key在失效的瞬间，执行的大并发就穿破缓存，直接请求持久层数据库，就像在一个屏幕上凿开一个洞。

当某个key在过期瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导致数据库瞬间压力过大。

#### 解决方案

##### 设置热点数据永不过期

从缓存层面，没有设置过期时间，所以不会出现热点key过期后产生的问题。

##### 加互斥锁

分布式锁：使用分布式锁，保证对于每个key同时只能有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此分布式锁的考验很大。