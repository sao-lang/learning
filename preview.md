# MySQL 面试知识体系

## 一、MySQL 基础架构

MySQL 主要分为两层：**Server层** 和 **存储引擎层**。

### Server层

#### 1. 连接管理
客户端通过 `mysql -u root -p` 连接 MySQL，MySQL 会创建连接、验证账号密码、分配线程。连接过多会导致内存增加和上下文切换增加，生产环境通常使用连接池（如 HikariCP、mysql2 pool）并限制最大连接数。

#### 2. SQL解析器
把 SQL 字符串解析成执行结构。例如 `select name from user where id = 1;` 解析为 SELECT / FROM user / WHERE id=1。同时检查 SQL 语法、表和字段是否存在。

#### 3. SQL优化器
选择最优执行方案。例如 `select * from user where id=100;`，可能方案 A 是全表扫描，方案 B 走主键索引，优化器选择方案 B。

#### 4. 执行器
负责真正执行 SQL：调用存储引擎 → 读取数据 → 返回结果。

## 二、MySQL 存储引擎

常见引擎：**InnoDB**（目前业务系统基本使用）和 **MyISAM**。

### 1. InnoDB特点
- **支持事务**：如订单支付（扣库存 + 增加订单状态 + 增加支付记录）必须全部成功或全部失败。
- **支持行级锁**：两个用户购买不同商品不会互相阻塞。
- **支持外键**：保证数据关系完整性，如 `order.user_id` 必须存在于 `user.id`。

### 2. MyISAM
不支持事务、不支持行锁，查询性能较高，适合读多写少的场景（如历史日志）。互联网业务基本使用 InnoDB。

## 三、数据库设计

核心：表设计、字段设计、关系设计、索引设计。

### 1. 表设计原则 — 单一职责
一个表负责一个业务。不要全部混在一个 `user_order_product` 表里，应该拆分为 `user`、`order`、`order_item`、`product`。

### 2. 三大范式

**第一范式（1NF）**：字段不可再拆。错误：user 表存 `phone1,phone2,phone3`；应该拆为 `user_phone(id, user_id, phone)`。

**第二范式（2NF）**：非主键字段必须完全依赖主键。错误：`order_item` 包含 `product_name`（依赖 product_id 而非 order_item）；应该拆为 `product(id, name, price)` 和 `order_item(id, order_id, product_id, count)`。

**第三范式（3NF）**：不能存在传递依赖。错误：user 表包含 `department_name`（依赖 department_id）；应该拆为 `user(id, name, department_id)` 和 `department(id, name)`。

### 3. 反范式设计
实际业务不会完全遵循范式，原因在于查询性能。例如帖子展示需要头像和昵称，如果每次都要 join user 表，高并发下可以冗余字段：`post(id, user_id, user_name, user_avatar, content)`。优点：查询快；缺点：数据同步复杂。根据业务权衡。

## 四、索引设计

### 1. 什么是索引
类似书的目录。没有索引需要逐页查找，有索引通过目录直接定位。数据库中索引是帮助快速定位数据的数据结构。

### 2. MySQL为什么使用B+Tree
- **高度低**：100万数据 B+Tree 高度通常 3~4 层，最多几次磁盘 IO。
- **支持范围查询**：叶子节点通过链表连接（如 `100 → 101 → 102 → 103`），范围查询非常快。
- **适合磁盘存储**：减少磁盘访问次数。

### 3. 聚簇索引
InnoDB 主键索引就是聚簇索引。叶子节点保存完整数据（如 `5 张三 18`），根据主键查询最快。

### 4. 非聚簇索引
如 `create index idx_name on user(name);`，叶子节点保存主键值。查询时先找到 name 对应的 id，再根据 id 查询完整数据，这个过程叫**回表**。

### 5. 覆盖索引
如果查询字段全部存在索引中，如索引 `(name,id)`，执行 `select id,name from user where name='张三';` 直接从索引返回，不需要回表，减少 IO。

### 6. 联合索引
如 `create index idx_user on user(name,age,sex);`，遵循**最左匹配原则**。有效查询：`where name='张三'`、`where name='张三' and age=20`、`where name='张三' and age=20 and sex=1`。无效查询：`where age=20`（没有 name）。

### 7. 索引设计原则
- **高频查询字段**：如 `where phone=xxx`，phone 适合索引。
- **区分度高**：身份证（唯一）适合，性别（男/女）不适合（过滤效果差）。
- **经常排序字段**：如 `order by created_time desc`，可以为 created_time 建立索引。

---

# MySQL 面试知识体系（二）

## 五、MySQL事务

### 1. 什么是事务
事务是一组数据库操作，要么全部成功，要么全部失败。例如银行转账（A扣100 + B加100），如果 A 成功但 B 失败则数据错误。

```sql
BEGIN;
update account set money=money-100 where id=1;
update account set money=money+100 where id=2;
COMMIT;   -- 失败则 ROLLBACK
```

### 2. 事务四大特性 ACID

- **A - Atomicity（原子性）**：事务中所有操作不可分割（如创建订单 + 扣库存 + 生成支付记录必须全部完成，否则全部回滚）。主要依靠 undo log。
- **C - Consistency（一致性）**：事务前后数据库状态满足业务规则（如库存不能为 -1）。一致性是最终目标，依赖原子性、隔离性、持久性。
- **I - Isolation（隔离性）**：多个事务同时执行互相不能影响，决定事务看到什么数据。
- **D - Durability（持久性）**：事务提交后数据永久保存，即使数据库重启。依靠 redo log。

### 3. MySQL事务日志

**undo log**：回滚日志。如 `update user set age=20 where id=1;` 如果事务失败，undo log 保存之前的 age=18 用于恢复。

**redo log**：保证持久性。修改数据直接刷磁盘很慢，所以先写 redo log：修改数据 → 写 redo log → 提交事务 → 异步刷磁盘。即使宕机也能通过 redo log 恢复。

## 六、事务隔离级别

MySQL 支持四种隔离级别：

### 1. READ UNCOMMITTED（读未提交）
最低级别，可能出现**脏读**：事务A更新 money=0 未提交，事务B读到 money=0，然后事务A回滚，实际 money 还是 100，B读到了不存在的数据。

### 2. READ COMMITTED（读已提交）
解决脏读，但可能出现**不可重复读**：同一个事务中两次查询结果不同（事务A查询 money=100，事务B修改为200并提交，事务A再次查询变成200）。

### 3. REPEATABLE READ（可重复读）
MySQL 默认隔离级别，保证同一个事务多次读取结果一致，通过 MVCC 实现。

### 4. SERIALIZABLE（串行化）
最高隔离，事务完全排队执行。优点：安全；缺点：性能差。

## 七、MVCC（Multi-Version Concurrency Control）

多版本并发控制，作用：提高读写并发。传统方式读数据必须加锁（锁住→修改→释放），并发低。MVCC 保存数据多个版本，读取时根据事务版本选择。

## 八、MySQL锁机制

### 1. 表锁
锁整个表，其他操作等待。优点：简单；缺点：并发低。

### 2. 行锁
锁具体记录，如 `select * from user where id=1 for update;` 只锁 id=1，其他用户可以操作其他数据。InnoDB 主要使用行锁。

### 3. 共享锁（读锁）
多个事务可以同时读，不能修改。SQL：`select * from user where id=1 lock in share mode;`

### 4. 排他锁（写锁）
一个事务持有后其他事务不能修改。SQL：`select * from user where id=1 for update;`

### 5. 乐观锁
认为冲突很少，不加数据库锁，通过版本号实现。表结构 `product(id, stock, version)`，查询得到 stock=10,version=1，更新时 `update product set stock=9, version=2 where id=1 and version=1;`。如果影响行数为 0，说明数据已被修改。

### 6. 悲观锁
认为一定会冲突，直接加锁。如秒杀库存：`select stock from product where id=1 for update;`

## 九、SQL优化

### 1. 为什么SQL慢？
常见原因：没有索引、索引失效、返回数据过多、join过多、数据量增长。

### 2. EXPLAIN
分析 SQL 执行计划，关注字段：
- **type**：访问类型，性能排序 system > const > eq_ref > ref > range > index > **ALL**（全表扫描，重点避免）
- **key**：实际使用索引（如 `idx_user_name`）
- **rows**：预计扫描行数，越少越好

### 3. SQL优化案例

**分页优化**：普通 `limit 100000,20` 需要扫描 100020 条再丢弃前面。优化为游标分页：
```sql
-- 第一次
select * from post order by id limit 20;
-- 记录最后 id=100，下一页：
select * from post where id>100 order by id limit 20;
```

**避免 select \***：查询无用字段、增加网络传输、影响覆盖索引。推荐 `select id,name from user;`

**JOIN 优化**：关联字段需要索引（如 `user.id` 和 `order.user_id`），否则大量扫描。

## 十、高并发数据库设计

### 1. 读写分离
写走主库，读走从库（主库同步到从库），降低数据库压力。

### 2. 分库分表
单表数据过大时拆分，如订单表 10 亿数据按用户维度拆为 `order_1`、`order_2`、`order_3`。

### 3. 数据归档
历史数据不要一直存在主表。如最近一年存 order，历史存 order_history，提高查询性能。

## 十一、结合简历项目回答

**问：为什么社区项目使用MySQL？**
> 社区业务包含用户、帖子、评论、积分等强结构化数据，这些数据之间存在明确关系且需要事务保证（如发帖同时增加积分），因此选择 MySQL 作为主要业务数据库。

**问：如何设计帖子表？**
> 帖子表保存帖子核心信息，通过 user_id 关联用户，评论单独设计评论表通过 post_id 关联。对于高频展示字段（如作者昵称）可适当冗余减少 join，同时针对 user_id、created_time 等查询条件建立索引。

**问：如何优化慢SQL？**
> 首先通过慢查询日志定位 SQL，然后使用 EXPLAIN 分析执行计划，检查是否走索引，根据查询条件设计联合索引，避免索引失效场景，必要时进行分页优化、数据归档等。

---

# Redis 面试知识体系

适用于：高级前端开发工程师 / 全栈开发工程师

目标：理解 Redis 为什么快、掌握缓存设计方案、能解决缓存一致性问题、能回答项目中的 Redis 使用场景。

## 一、Redis 基础

### 1. Redis是什么
Redis（Remote Dictionary Server）是一种**基于内存的高性能 Key-Value 数据库**。特点：数据存储在内存、支持丰富数据结构、单线程处理核心命令、支持持久化、支持高并发访问。常见用途：数据缓存、分布式锁、限流、排行榜、Session存储、消息队列。

### 2. 为什么Redis性能高？（QPS可达几十万级）

**基于内存**：传统数据库访问路径 CPU→内存→磁盘（磁盘IO慢），Redis 直接 CPU→内存，访问速度非常快。

**单线程模型**：Redis 核心命令执行主要是单线程，避免了线程切换和锁竞争。

**IO多路复用**：使用事件驱动模型（epoll/select），一个线程处理多个客户端连接。

## 二、Redis数据结构

### 1. String（最基础结构）
命令：`SET user:1001:name 张三` / `GET user:1001:name`

使用场景：
- **缓存**：如 `product:10001` 缓存商品详情 `{name:"手机", price:3999}`
- **计数器**：`INCR article:view:1001` → 100, 101, 102...
- **分布式锁**：`SET lock:user:1001 1 NX EX 30`

### 2. Hash（类似Java对象）
```redis
HSET user:1001 name 张三
HSET user:1001 age 20
HGETALL user:1001
```
适合对象缓存，如 `user:{id}`。

### 3. List（双向链表，先进先出）
```redis
LPUSH queue task1
RPOP queue
```
应用：简单消息队列（用户注册→发送邮件任务→Redis List→消费者处理）。

### 4. Set（集合，元素唯一）
```redis
SADD post:1001:like user1
SCARD post:1001:like   -- 查询点赞人数
```
应用：去重、共同好友、标签。

### 5. Sorted Set / ZSet（带权重排序）
结构：`member + score`。
```redis
ZADD rank 100 userA
ZADD rank 200 userB
ZREVRANGE rank 0 10   -- 查询前10名
```
应用：排行榜、热度排行、延迟任务。

## 三、Redis缓存设计

### 1. 为什么使用缓存？
数据库数据可靠且支持事务，但查询压力大。Redis 查询快，所以热点数据放 Redis。

典型架构：用户请求 → Redis缓存（存在直接返回）→ 不存在 → MySQL → 写回Redis → 返回。

### 2. Cache Aside模式（旁路缓存，企业最常用）

**查询流程**：先查 Redis，存在直接返回；不存在则查 MySQL，写回 Redis，再返回。

**更新流程**：先更新 MySQL，再删除 Redis（不更新 Redis）。因为数据库才是真实数据源，删除缓存比更新缓存更安全（避免并发问题）。

## 四、缓存穿透、击穿、雪崩（面试高频）

### 1. 缓存穿透
**问题**：查询不存在的数据（如攻击 `/user?id=-999999`），Redis 没有 → MySQL 也没有，大量请求直接打数据库。

**解决方案**：
- **缓存空值**：设置短 TTL（如5分钟）。优点：简单；缺点：可能缓存大量无效 key。
- **布隆过滤器**：提前判断数据是否存在，不存在直接返回。优点：空间小；缺点：存在误判。

### 2. 缓存击穿
**问题**：一个热点 Key 过期（如热门帖子 `post:10001`），10000人同时访问全部打到 MySQL。

**解决方案**：
- **互斥锁**：第一个请求获取锁、查数据库、写缓存、释放锁，其他请求等待。
- **热点数据永不过期**：后台主动刷新。

### 3. 缓存雪崩
**问题**：大量 Key 同时过期（如全部缓存设24小时，晚上12点同时失效）。

**解决方案**：
- **TTL随机化**：24小时 + 随机0~1小时，避免同时过期。
- **分批加载**：不同时间生成缓存。

## 五、Redis和MySQL数据一致性（高级面试重点）

### 1. 为什么会不一致？
更新 MySQL 后再更新 Redis，如果 Redis 更新失败就会导致 MySQL 是"李四"而 Redis 还是"张三"。

### 2. 常见方案
**更新数据库，删除缓存（推荐）**：修改 MySQL → 删除 Redis。缓存只是副本，数据库才是真实数据。

### 3. 为什么删除缓存而不是更新缓存？
更新缓存可能产生并发问题：线程A更新数据库 name=李四，线程B查询到旧数据 name=张三 并更新 Redis，最终 Redis 错误。

### 4. 延迟双删（高并发场景）
第一次删除缓存 → 更新数据库 → 等待几十毫秒 → 再次删除缓存。目的：避免并发读取旧数据重新写缓存。

## 六、Redis分布式锁

### 1. 为什么需要分布式锁？
单机可以用 `synchronized`，多机器（服务器A/B/C）本地锁无效，需要 Redis 锁。

### 2. 基础实现
加锁：`SET lock:order 123 NX EX 30`（NX：不存在才创建，EX：30秒自动过期）。

释放锁不能直接 `DEL lock:order`（可能删除别人的锁）。正确做法：value 保存唯一 ID（如 `uuid-123`），释放时判断是不是自己的锁。

## 七、Redis持久化

### 1. RDB（Redis Database）
定时生成快照（如10分钟/1000次修改），保存到 `dump.rdb`。优点：文件小、恢复快；缺点：可能丢失最近数据。

### 2. AOF（Append Only File）
记录每次写操作（如执行 `SET name 张三`，记录 `SET name 张三`）。优点：数据安全；缺点：文件较大。

## 八、Redis高可用

### 1. 主从复制
Master 负责写，Slave 负责读。

### 2. 哨兵模式（Sentinel）
监控主节点，如果 Master 挂了自动选新 Master。

### 3. Redis Cluster（集群模式）
多个节点分片存储（如节点1: 0-5000，节点2: 5001-10000），解决单机容量限制。

## 九、结合你的项目回答

**问：豆包社区 Redis 怎么使用？**
> 社区业务中用户信息、帖子详情等高频访问数据适合使用 Redis 缓存。请求优先查询 Redis，缓存未命中后查询 MySQL 并回填。更新数据时采用更新 MySQL 后删除缓存方式，降低缓存和数据库不一致风险。对于热点数据设置合理 TTL，避免缓存压力集中。

**问：为什么不用直接查 MySQL？**
> 社区存在大量读请求，如果全部访问 MySQL 会造成数据库压力。Redis 基于内存访问，适合作为热点数据缓存层。

**问：如何保证缓存一致？**
> 采用 Cache Aside 模式，数据库作为真实数据源。写操作先更新数据库，再删除缓存。必要情况下通过延迟双删或异步消息进一步保证最终一致性。

**问：如何设计排行榜？**
> 积分排行榜可以使用 Redis ZSet，score 保存积分值，member 保存用户 ID，通过 ZREVRANGE 获取排名靠前用户。

## 十、和你的简历匹配的重点

简历中写了：MySQL、Redis、数据库建模、索引设计、事务处理、缓存策略及常见数据一致性方案。

**MySQL 必须掌握**：表设计、三范式、索引、B+Tree、联合索引、explain、事务、锁。

**Redis 必须掌握**：String/Hash/ZSet、Cache Aside、穿透、击穿、雪崩、分布式锁、一致性方案。

达到这个程度足够支撑高级前端 / 全栈偏前端岗位面试。你的简历定位是**全栈（偏前端）**，ES、Kafka、Docker、Linux 不需要后端基础设施专家级深度，但要达到：能解释为什么用、能说出核心原理、能结合项目场景设计、能处理常见问题。

---

# ES / Kafka / Docker / Linux 面试知识体系

## 一、Elasticsearch（ES）

### 1. Elasticsearch是什么？
基于 Lucene 的**分布式搜索与分析引擎**。特点：全文检索、分布式存储、高性能搜索、实时分析。常见应用：搜索系统、日志分析、商品检索、用户搜索、数据分析。

### 2. 为什么不用MySQL搜索而使用ES？
MySQL 适合结构化查询（如 `select * from user where id=1;`），但全文搜索（如搜索"苹果手机"希望匹配"苹果 iPhone 手机"）需要分词 + 倒排索引，ES 更适合。

### 3. ES核心概念
- **Index（索引）**：类似 MySQL 的数据库，如 `user_index`
- **Document**：类似 MySQL 一行数据（JSON格式）
- **Field**：字段，类似 MySQL 列
- **Shard（分片）**：将数据拆分（如一个索引拆为 shard1/shard2/shard3），提高存储能力
- **Replica（副本）**：数据备份，提高可用性和查询能力

### 4. ES为什么查询快？
核心是**倒排索引**。普通数据库通过字段找数据（扫描），ES 提前建立关键词→文档的映射（如 `ES → 文章1, 文章5, 文章10`），直接定位。

### 5. 倒排索引 vs 正排索引
正排：文章→关键词（文章1: ES, Redis, Docker）；倒排：关键词→文章（ES: 文章1, 文章5, 文章10）。搜索效率高。

### 6. ES分词
如"苹果手机"分词为"苹果"+"手机"，常用中文分词器：IK Analyzer。

### 7. ES数据同步MySQL方案
常见架构：MySQL → binlog → Canal → Kafka → ES。流程：MySQL数据变化 → Canal监听binlog → 发送消息 → ES更新索引。

### 8. ES和Redis区别
| 特性 | Redis | ES |
|------|-------|-----|
| 定位 | 缓存 | 搜索 |
| 数据结构 | KV | 文档 |
| 查询 | 简单 | 复杂 |
| 全文搜索 | 不适合 | 强 |
| 存储 | 内存 | 磁盘 |

### 9. ES项目回答
**问：为什么社区搜索不用MySQL？**
> 社区存在帖子标题、内容搜索需求，MySQL like 查询无法满足复杂全文检索。因此使用 ES 建立帖子索引，用户搜索时查询 ES 获取结果，再根据 ID 查询业务数据。

## 二、Kafka

### 1. Kafka是什么？
**分布式消息队列系统**，主要作用：异步处理。如用户注册，同步要依次完成发送邮件、生成积分、发送通知（耗时），使用 Kafka 后注册→Kafka→其他服务消费，提高响应速度。

### 2. Kafka核心概念
- **Producer（生产者）**：发送消息，如订单服务
- **Consumer（消费者）**：处理消息，如积分服务
- **Topic**：消息分类，如 `order-topic`、`message-topic`
- **Partition（分区）**：将 topic 分为多个分区（partition1/2/3），提高并发
- **Consumer Group（消费者组）**：多个消费者共同消费，一条消息只会被组内一个消费者处理

### 3. Kafka为什么快？
- **顺序写磁盘**：磁盘随机写慢，Kafka 追加写日志文件（1→2→3→4→5），速度快
- **Page Cache**：利用操作系统缓存，减少 IO
- **批量发送**：不是一条条发送，而是批量发送

### 4. Kafka消息可靠性
- **Producer**：配置 `acks=all`，等待所有副本确认
- **Broker**：副本机制，leader 负责读写，follower 同步
- **Consumer**：手动提交 offset，避免消费失败丢消息

### 5. Kafka消息重复消费怎么办？
原因：消费者处理成功但提交 offset 失败，导致再次消费。解决方案：
- **幂等设计**：`insert where order_id 不存在`，而不是直接新增订单
- **唯一ID**：消息携带 messageId，消费记录存到 `processed_message` 表避免重复

### 6. Kafka应用场景（社区项目）
用户发帖 → Kafka → 积分服务 + 消息通知 + 搜索索引更新

## 三、Docker

### 1. Docker是什么？
**容器化技术**，解决环境一致性问题（如开发 Node18 / 服务器 Node16 导致失败），Docker 把环境一起打包。

### 2. Docker核心概念
- **Image（镜像）**：模板，如 `node:20`、`mysql:8`、`redis:7`
- **Container（容器）**：镜像运行实例
- **Dockerfile**：构建镜像文件
```dockerfile
FROM node:20
WORKDIR /app
COPY .
RUN npm install
CMD npm start
```
- **Volume**：数据持久化（如 MySQL 数据不能存在容器内部，需挂载到宿主机）
- **Network**：容器通信（如 api 连接 mysql）

### 3. Docker部署流程
代码 → Dockerfile → `docker build` → 镜像 → `docker run` → 容器

### 4. Docker Compose
管理多个服务。`docker-compose.yml` 定义 mysql、redis、backend、nginx 等，启动命令：`docker compose up`。

### 5. Docker和虚拟机区别
| 特性 | Docker | 虚拟机 |
|------|--------|--------|
| 资源 | 共享内核 | 完整系统 |
| 启动 | 秒级 | 分钟级 |
| 体积 | 小 | 大 |
| 隔离 | 进程级 | 系统级 |

### 6. Docker常见问题
- 容器退出：`docker logs container`
- 进入容器：`docker exec -it container bash`
- 查看运行状态：`docker ps`

## 四、Linux

### 1. 常用命令
文件操作：`ls`（查看）、`cd`（进入）、`cp`（复制）、`mv`（移动）、`rm`（删除）

### 2. 查看进程
`ps aux` 或 `top`（查看 CPU、内存）

### 3. 查看端口
`netstat -tunlp` 或 `lsof -i:8080`

### 4. 查看日志
实时查看：`tail -f app.log`；查看全部：`cat`；搜索：`grep error app.log`

### 5. 权限
`rwx` 对应 755：owner（rwx）、group（rx）、other（rx）

### 6. 启动服务（systemd）
启动：`systemctl start nginx`；查看状态：`systemctl status nginx`；开机启动：`systemctl enable nginx`

### 7. 性能排查
- **CPU高**：`top` 找到 PID，`ps -ef | grep PID` 查看详情
- **内存高**：`free -h`
- **磁盘满**：`df -h` 查看，`du -sh *` 寻找大文件

### 8. 线上问题处理流程
接口响应慢时：`top` 看 CPU → `free` 看内存 → `tail -f` 看日志 → `netstat` 检查网络 → 定位服务。

## 五、结合你的简历回答

**问：Docker项目经验？**
> 项目开发过程中使用 Docker 统一开发环境，通过 Docker Compose 管理 MySQL、Redis 等依赖服务，减少环境差异，提高团队开发效率。

**问：Redis + Kafka + ES 架构设计（社区系统）**
```
用户请求 → Go服务 → MySQL / Redis缓存
用户发帖 → Kafka → 积分服务 + ES搜索
```

## 六、面试重点排序

⭐⭐⭐⭐⭐ **必须掌握**：MySQL、Redis、Docker

⭐⭐⭐⭐ **重点理解**：Kafka、ES

⭐⭐⭐ **基础即可**：Linux

---

你的背景（5年前端 + Go项目）面试时，**不要把自己包装成后端基础设施工程师**，重点表达：
> 我能独立完成业务服务开发，理解数据库、缓存、消息、部署链路，能够和后端架构协作。

这样更符合你的简历定位。
