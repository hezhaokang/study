#  27、NoSQL数据库Redis

## NoSQL 数据库

数据库主要分为两大类：关系型数据库与 NoSQL 数据库。 

关系型数据库，是建立在关系模型基础上的数据库，其借助于集合代数等数学概念和方法来处理数据库 中的数据。主流的 MySQL、Oracle、MS SQL Server 和 DB2 都属于这类传统数据库。 

NoSQL 数据库，全称为 Not Only SQL，意思就是适用关系型数据库的时候就使用关系型数据库，不适 用的时候可以考虑使用更加合适的数据存储。NoSQL 是对不同于传统的关系型数据库的数据库管理系统 的统称。 

NoSQL用于超大规模数据的存储。（例如谷歌或Facebook每天为他们的用户收集万亿比特的数据）。 这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

### RDBMS和NOSQL对比

**RDBMS** 

- 高度组织化结构化数据 
- 结构化查询语言（SQL） 
- 数据和关系都存储在单独的表中。 
- 数据操纵语言，数据定义语言 
- 严格的一致性 
- 基础事务 

**NoSQL** 

- 代表着不仅仅是SQL, 没有声明性查询语言 
- 没有预定义的模式 
- 最终一致性，而非ACID属性 
- 非结构化和不可预知的数据 
- CAP定理 
- 高性能，高可用性和可伸缩性





NoSQL（Not Only SQL）是一类非关系型数据库，设计用于存储、管理和检索大规模分布式数据。以下是 NoSQL 数据库的主要优点和缺点：

------

### **优点**

#### 1. **高性能**

- **快速读取和写入**：NoSQL 数据库通常为特定的用例优化，比如键值存储、列族存储等，支持高吞吐量的读写操作。
- **低延迟**：由于数据的结构化存储和分布式架构，查询速度快，延迟低。

#### 2. **扩展性**

- **水平扩展**：支持通过增加节点（而不是增加硬件性能）来处理更大的数据量和更高的并发量，成本较低。
- **分布式架构**：天然支持分布式存储和计算，便于构建高可用系统。

#### 3. **灵活的数据模型**

- **模式自由**：NoSQL 数据库不需要固定的表结构，可以动态添加字段，适应频繁变化的业务需求。
- **多种数据模型**：支持键值对、文档、列族、图等多种数据模型，适合不同的场景。

#### 4. **高可用性和故障恢复**

- **数据分片和复制**：NoSQL 通常内置分片和复制功能，实现高可用性和数据冗余。
- **自动故障恢复**：部分 NoSQL 数据库支持自动容错和数据恢复。

#### 5. **支持大规模数据处理**

- **大数据存储和分析**：能够高效处理非结构化或半结构化数据（如 JSON、XML）。
- **实时数据处理**：支持实时数据分析和流处理。

------

### **缺点**

#### 1. **缺乏标准化**

- **查询语言不统一**：NoSQL 没有统一的查询语言（不像 SQL 有标准化的语法），每种数据库都有自己的查询方式，学习成本高。
- **实现差异大**：不同 NoSQL 数据库之间的功能差异较大，迁移数据困难。

#### 2. **事务支持有限**

- **弱一致性**：大多数 NoSQL 数据库遵循 CAP 定理中的 AP（可用性和分区容忍性），可能无法保证强一致性（ACID 属性）。
- **复杂事务支持不足**：NoSQL 数据库对复杂事务（如跨多个文档或表的操作）的支持较弱。

#### 3. **工具和生态不如关系型数据库成熟**

- **调试和监控工具有限**：虽然在快速发展，但许多 NoSQL 数据库的工具生态仍不如关系型数据库成熟。
- **社区支持不足**：一些较新的或小众的 NoSQL 数据库的社区和文档可能不够完善。

#### 4. **适用场景有限**

- **不适合高度结构化数据**：如果数据有复杂的关系和严格的约束（如银行系统），NoSQL 并不是理想选择。
- **不适合复杂查询**：在需要复杂联表查询或统计分析的场景下，NoSQL 可能性能较差或功能不足。

#### 5. **学习曲线**

- **运维复杂**：NoSQL 的分布式架构和不同的数据模型可能增加运维复杂度。
- **技能要求高**：需要针对特定场景选择合适的 NoSQL 类型，并掌握其使用方式。

------

### **总结**

| **优点**                       | **缺点**                         |
| ------------------------------ | -------------------------------- |
| 高性能，低延迟                 | 缺乏标准化，查询语言多样化       |
| 支持大规模数据存储和处理       | 对事务支持有限，弱一致性         |
| 模式灵活，适应频繁变化的需求   | 工具和生态不如关系型数据库成熟   |
| 水平扩展能力强，易于分布式部署 | 不适合高度结构化或复杂查询的场景 |

NoSQL 适合处理大规模、快速变化的非结构化数据，如社交媒体数据分析、实时推荐系统、物联网等场景。但在数据关系复杂、事务要求严格的场景下，传统关系型数据库仍是更好的选择。



## CAP

- **C：Consistency** 

  即**一致性**， 所有节点在同一时间具有相同的数据视图 

  换句话说，如果一个节点在写入操作完成后，所有其他节点都能立即读取到最新的数据。 

  注意，这里的一致性指的是强一致性，也就是数据更新完，访问任何节点看到的数据完全一致，要 和弱一致性，最终一致性区分开来。 

  每次读取的数据都应该是最近写入的数据或者返回一个错误, 而不是过期数据，也就是说，所有节 点的数据是一致的。

- **A：Availability** 

  即**可用性**，所有的节点都保持高可用性,要求服务在接收到客户端请求后，都能够给出响应 

  每个非故障节点都能够在有限的时间内返回有效的响应，即系统一直可用。可用性强调系统对用户 请求的及时响应 

  注意，这里的高可用还包括不能出现延迟，比如如果某个节点由于等待数据同步而阻塞请求，那么 该节点就不满足高可用性。 

  也就是说，任何没有发生故障的服务必须在有限的时间内返回合理的结果集。 

  每次请求都应该得到一个有效的响应，而不是返回一个错误或者失去响应，不过这个响应不需要保 证数据是最近写入的,也就是说系统需要一直都是可用正常使用的，不会引起调用者的异常，但是并 不保证响应的数据是最新的。

- **P：Partiton tolerance**

  **分区容忍性**是指系统中的节点由于网络故障无法相互通信，导致系统被分成多个孤立的子系统 

  在分布式系统中，不同节点之间通过网络进行通信 

  分区容忍性是指当分布式系统中出现网络分区（即系统中的一部分节点无法和其他节点进行通信） 时，系统能够容忍这种情况，并且分离的系统也能够正常运行。这意味着，即使系统中某些节点或 网络分区出现故障或延迟，整个系统仍然能够继续运作，不会受到单点故障的影响。 

  由于网络是不可靠的，所有节点之间很可能出现无法通讯的情况，在节点不能通信时，要保证系统 

  可以继续正常服务。 在分布式系统中，机器分布在各个不同的地方，由网络进行连接。由于各地的网络情况不同，网络 的延迟甚至是中断是不可避免的。 

  因此分区容错性通常是分布式系统的必要条件，即如果不能容忍分区，就只能是一个单一系统，而 非一个真正可用的分布式系统



在服务器之间的网络出现异常的情况下，一致性和可用性是不可能同时满足的，必须要放弃 一个，来保证另一个。这也正是CAP定理所说的，在分布式系统中，P总是存在的。在P发生的前提下， C(一致性)和A（可用性）不能同时满足。这种情况在做架构设计的时候就要考虑到，要评估对业务的影 响，进行权衡决定放弃哪一个。在通常的业务场景下，系统不可用是不能接受的，所以要优先保证可用 性，暂时放弃一致性。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三大类： 

**CA** - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。 

放弃分区容忍性，即不进行分区，不考虑由于网络不通或结点挂掉的问题，则可以实现一致性和可 用性。那么系统将不是一个标准的分布式系统 

比如:单一数据中心数据库,所有节点都位于同一个数据中心，并且节点之间的通信是高可靠的 



**CP** - 满足一致性，分区容忍性的系统，通常性能不是特别高。 

放弃可用性，追求强一致性和分区 容错性 

例如: Zookeeper,ETCD,Consul,MySQL的PXC等集群就是追求的强一致，再比如跨行转账，一次转 账请求要等待双方银行系统都完成整个事务才算完成。 



**AP** - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。 

放弃一致性，追求分区容忍性和可用性。这是很多分布式系统设计时的选择。 

例如：MySQL主从复制，默认是异步机制就可以实现AP，但是用户接受所查询的到数据在一定时 间内不是最新的. 通常实现AP都会保证最终一致性，而BASE理论就是根据AP来扩展的，一些业务场景 

比如：订单退 款，今日退款成功，明日账户到账，只要用户可以接受在一定时间内到账即可。



### Base理论

Base理论是三要素的缩写：基本可用（Basically Available）、软状态（Soft-state）、最终一致性 （Eventually Consistency）

<img src="5day-png\27Base理论.png" alt="image-20250118142333427" style="zoom: 25%;" />

- 基本可用 （Basically Available） 

  相对于CAP理论中可用性的要求：【任何时候，读写都是成功的】，“基本可用”要求系统能够基本 运行，一直提供服务，强调的是分布式系统在出现不可预知故障的时候，允许损失部分可用性。比 如系统通过断路保护而引发快速失败，在快速失败模式下，支持加载默认显示的内容（静态化的或 者被缓存的数据），从而保证服务依然可用。 

  相比于正常的系统，可能是响应时间延长，或者是服务被降级。 

  比如在在秒杀活动中，如果抢购人数太多，超过了系统的QPS峰值，可能会排队或者提示限流。

- 软状态 （Soft state） 

  相对于ACID事务中原子性要求的要么全做，要么全不做，强调的是强制一致性，要求多个节点的数据副本是一致的，强调数据的一致性。这种原子性可以理解为”硬状态“。 

  而软状态则允许系统中的数据存在中间状态，并认为该状态不影响系统的整体可用性，即允许系统 在不同节点的数据副本上存在数据延时。 

  比如粉丝数，关注后需要过一段时间才会显示正确的数据。

- 最终一致性（Eventuallyconsistent） 

  数据不可能一直处于软状态，必须在一个时间期限后达到各个节点的一致性。在期限过后，应当保证所有副本中的数据保持一致性，也就是达到了数据的最终一致性。 

  在系统设计中，最终一致性实现的时间取决于网络延时、系统负载、不同的存储选型，不同数据复 制方案设计等因素。也就是说，不保证用户什么时候能看到更新完成后的数据，但是终究会看到 的。



## NoSQl数据库分类

NoSQL（Not Only SQL）数据库是为了解决传统关系型数据库在处理大规模、分布式数据时的性能瓶颈而诞生的。NoSQL 数据库并不使用传统的关系模型，而是提供了多种不同的数据存储方式，适应不同应用场景。根据数据模型的不同，NoSQL 数据库可以分为以下几种类型：

### 1. **键值数据库（Key-Value Store）**

键值数据库是最简单的 NoSQL 数据库类型，数据以键值对的形式存储。每个数据项由一个唯一的键（Key）和与之对应的值（Value）组成。适用于存储大量的简单、无结构的数据。

**特点：**

- 采用简单的键值对存储方式。
- 高效地支持快速的查找、插入、删除操作。
- 适用于存储缓存、会话信息、用户配置等场景。

**代表数据库：**

- **Redis**：一个高性能的键值存储系统，常用于缓存和实时数据处理。
- **Memcached**：主要用于高速缓存。
- **Riak**：一个高度可扩展的键值存储系统。

### 2. **文档型数据库（Document Store）**

文档型数据库存储的数据是“文档”格式，通常使用 JSON 或 BSON 格式进行存储。每个文档具有一个唯一的 ID，并可以存储任意类型的结构化数据，可以灵活地存储不同字段的数据。

**特点：**

- 数据以文档的形式进行存储（通常是 JSON、BSON 或 XML 格式）。
- 适合存储结构不固定的复杂数据，如用户数据、产品信息等。
- 支持复杂的查询和索引功能。

**代表数据库：**

- **MongoDB**：一个基于文档的数据库，支持高性能、高可扩展性和高可用性。
- **CouchDB**：一个面向文档的数据库，使用 HTTP 协议进行数据操作。
- **Couchbase**：一个高性能的分布式 NoSQL 数据库，支持文档和键值存储。

### 3. **列族数据库（Column-Family Store）**

列族数据库将数据按列进行组织，每一列的数据是独立存储的，而不是按行进行存储。列族数据库特别适用于需要快速读取某一列数据的场景。

**特点：**

- 数据按列而不是按行存储。
- 在处理大规模数据时具有高效的性能。
- 适用于实时数据分析和大规模数据集的存储。

**代表数据库：**

- **HBase**：基于 Google Bigtable 设计的分布式列族存储系统，广泛应用于大数据处理。
- **Cassandra**：一个高可用、分布式的列族数据库，适用于处理大量写入操作的应用。
- **Hypertable**：一个开源的高性能的列族存储系统。

### 4. **图数据库（Graph Database）**

图数据库专门用于存储和操作图数据结构，它能够有效处理节点（实体）和边（关系）之间的复杂关系。图数据库主要用于需要处理大量关联数据的场景，如社交网络、推荐系统等。

**特点：**

- 专注于处理节点和边之间的关系。
- 高效地处理图查询，如最短路径、图遍历等。
- 适用于社交网络分析、推荐系统、欺诈检测等。

**代表数据库：**

- **Neo4j**：一个开源的图数据库，支持 ACID 属性，广泛应用于社交网络、推荐系统等领域。
- **ArangoDB**：一个多模型数据库，支持图形模型、文档模型和键值模型。
- **JanusGraph**：一个分布式图数据库，基于 Apache TinkerPop 规范，适合大规模图形数据处理。

### 5. **时序数据库（Time Series Database）**

时序数据库专门用于存储时间序列数据，通常是连续的、按时间排序的数据。它广泛应用于 IoT（物联网）、监控、金融等领域。

**特点：**

- 处理高吞吐量、频繁变化的时序数据。
- 提供高效的存储和查询时间序列数据的方式。
- 支持聚合、降采样、时间窗口等功能。

**代表数据库：**

- **InfluxDB**：一个开源的时序数据库，适用于高性能的时序数据存储和分析。
- **TimescaleDB**：一个基于 PostgreSQL 的时序数据库，兼具关系型数据库的功能与时序数据的优化。
- **Prometheus**：一个开源监控系统和时序数据库，广泛用于应用监控。

### 6. **多模型数据库（Multi-Model Database）**

多模型数据库支持多种数据模型，可以在同一个数据库中处理文档、图、键值等多种数据格式，提供更高的灵活性。

**特点：**

- 同时支持多种数据模型。
- 可以根据不同的应用需求选择最合适的数据模型。
- 适用于多种复杂场景。

**代表数据库：**

- **ArangoDB**：同时支持文档、图和键值存储的多模型数据库。
- **OrientDB**：一个支持文档、图和对象存储的多模型数据库。
- **Couchbase**：支持文档存储，同时也具有键值存储功能。

### 总结

NoSQL 数据库通过提供不同的存储模型，适应了各种复杂的业务需求。具体选择哪种类型的 NoSQL 数据库，应该根据应用的具体场景来决定：

- **键值数据库**适用于高效缓存和简单的键值映射。
- **文档型数据库**适合灵活的数据存储，特别是结构不固定的应用场景。
- **列族数据库**适合大规模数据集和需要高效读取某一列的场景。
- **图数据库**适用于复杂关系的数据存储，尤其是社交网络等。
- **时序数据库**适用于存储和分析时序数据。
- **多模型数据库**适用于需要灵活选择数据模型的复杂应用。

## Redis 特性

Redis（**Remote Dictionary Server**）是一个开源的、基于内存的高性能键值对存储数据库。它以其高效的读写性能、丰富的数据类型和简单易用的接口，成为了许多现代应用中的首选缓存和数据存储解决方案。

Redis 的主要特性如下：

------

### 1. **高性能**

Redis 是一个内存数据库，所有数据都存储在内存中，这使得它的读写操作非常快速。它能够处理每秒数百万次的请求，适用于需要高速读写的场景，如缓存系统、会话管理、实时数据处理等。

### 2. **持久化支持**

虽然 Redis 是基于内存的数据库，但它提供了持久化机制来保存数据，以便在重启或宕机时恢复数据。Redis 提供了两种持久化方式：

- **RDB（Redis 数据库快照）**：定期将内存中的数据保存到磁盘中，适合对数据一致性要求不高的场景。
- **AOF（追加日志文件）**：将每个写操作都记录到日志文件中，提供更高的数据持久性，适合对数据一致性要求较高的场景。

### 3. **丰富的数据结构**

Redis 不仅支持传统的字符串（String）类型，还支持多种复杂数据类型：

- **字符串（String）**：可以是文本或二进制数据，支持常见的字符串操作。
- **列表（List）**：双端队列，支持队列、栈操作，如 `LPUSH`、`RPUSH`。
- **集合（Set）**：无序集合，支持交集、并集、差集等操作。
- **有序集合（Sorted Set）**：每个元素都有一个分数，可以根据分数排序，适用于排行榜、评分系统等场景。
- **哈希（Hash）**：键值对集合，适合存储对象类型数据。
- **位图（Bitmap）**：用于处理二进制数据，如用户签到、访问统计等。
- **HyperLogLog**：用于近似计算集合的基数，适合计数和去重场景。
- **地理空间（Geo）**：支持存储和查询地理位置数据，适用于地理位置服务。

### 4. **事务支持**

Redis 提供了简单的事务功能，允许一组命令作为一个原子操作执行。使用 `MULTI` 命令开始事务，使用 `EXEC` 提交事务。事务中的命令会按照提交顺序依次执行，但 Redis 事务不支持回滚，因此不能撤销已执行的命令。

### 5. **发布/订阅（Pub/Sub）**

Redis 提供了发布/订阅消息传递模型，允许客户端订阅某个频道，当有消息发布到该频道时，所有订阅该频道的客户端都会接收到消息。适用于实时消息通知和事件驱动的应用。

### 6. **Lua 脚本支持**

Redis 允许通过 Lua 脚本执行原子操作。通过 `EVAL` 命令，用户可以将 Lua 脚本提交给 Redis 服务器执行，所有的操作都将在服务器端执行，确保了操作的原子性。

### 7. **高可用和分布式**

Redis 提供了多种高可用性和分布式部署选项：

- **Redis Sentinel**：提供自动故障转移、监控和通知功能，确保 Redis 集群的高可用性。
- **Redis Cluster**：实现了数据分片和集群管理，可以自动将数据分布到多个 Redis 实例中，提高扩展性和可用性。

### 8. **支持主从复制**

Redis 支持主从复制（Master-Slave），一个主节点可以有多个从节点，主节点接受写请求，从节点进行数据复制，提供读取负载均衡和数据冗余。主从复制机制可以提高数据的可用性和性能。

### 9. **内存管理**

Redis 提供了灵活的内存管理机制，支持通过 **LRU（最少最近使用）** 算法来淘汰不常用的数据。可以配置内存上限，确保在内存资源不足时 Redis 的正常运行。

- **内存限制**：可以通过 `maxmemory` 配置内存上限，超过该上限时 Redis 会根据设置的策略（如 `volatile-lru`、`allkeys-lru` 等）来清除数据。

### 10. **支持多种编程语言客户端**

Redis 提供了多种编程语言的客户端支持，几乎所有流行的编程语言都有 Redis 客户端库，如 Python、Java、Go、Node.js、C、C++ 等，可以轻松集成到应用程序中。

### 11. **支持多种数据持久化策略**

- **RDB（Redis 数据库快照）**：通过定期快照的方式，将内存数据保存到硬盘中。
- **AOF（Append Only File）**：通过记录所有写操作的日志，将操作追加到日志文件中。
- **混合持久化（RDB + AOF）**：通过结合 RDB 快照和 AOF 日志，提供更高的数据安全性。

### 12. **支持多种序列化格式**

Redis 支持多种数据格式的序列化和反序列化，例如 JSON、MsgPack、Protobuf 等。对于复杂的数据结构，可以灵活选择不同的序列化格式。

------

### Redis 的优点：

- **高性能**：由于 Redis 主要将数据存储在内存中，提供了超高的读写速度。
- **丰富的数据结构**：支持多种数据结构，能适应不同的业务需求。
- **持久化选项**：支持数据持久化，既能保证数据不丢失，也能保证高性能。
- **简易的部署和使用**：Redis 配置简单，使用方便，适合各种开发需求。
- **广泛的客户端支持**：支持多种编程语言的客户端，便于集成。
- **高可用性和分布式功能**：支持主从复制、哨兵和集群等高可用和分布式特性。

### Redis 的缺点：

- **内存消耗大**：由于 Redis 是一个内存数据库，如果数据量过大，可能需要大量的内存资源。
- **持久化机制的开销**：尽管 Redis 提供了持久化机制，但开启持久化会引入一定的性能开销。
- **不适合复杂的查询**：Redis 不支持 SQL 类似的复杂查询，适用于需要快速读写的场景，但不适合需要复杂查询和操作的场景。

------

### Redis 使用场景：

1. **缓存系统**：Redis 可以用于缓存热点数据，减少数据库压力，加速应用响应速度。
2. **会话存储**：用于存储用户会话信息，支持快速读取和写入。
3. **排行榜和计数器**：使用 Redis 的有序集合（Sorted Set）存储用户分数、排名等。
4. **任务队列**：使用 Redis 的列表（List）类型，提供高效的任务队列管理。
5. **实时数据分析**：利用 Redis 提供的高性能数据存储，适用于实时数据处理和分析。
6. **消息队列**：使用 Redis 的发布/订阅模式，进行异步消息推送和事件通知。
7. **地理位置应用**：使用 Redis 的 Geo 数据类型实现基于位置的数据存储与查询。

### 总结

- 速度快: 10W QPS,基于内存,C语言实现 
- 单线程：引号的”单线程“ 
- 持久化：RDB，AOF 
- 支持多种数据类型 
- 支持多种编程语言 
- 功能丰富: 支持Lua脚本,发布订阅,事务,pipeline等功能 
- 简单: 代码短小精悍(单机核心代码只有23000行左右),单线程开发容易,不依赖外部库,使用简单 
- 主从复制 
- 支持高可用和分布式

## “单线程”

- 单线程为何如此快? 

- 纯内存 

- 非阻塞 

- 避免线程切换和竞态消耗 

- 基于Epoll实现IO多路复用

<img src="5day-png\27单线程.png" alt="image-20250118143429429" style="zoom: 33%;" />

<img src="5day-png\27单线程1.png" alt="image-20250118143537180" style="zoom: 25%;" />

**注意事项:**

- 一次只运行一条命令  
- 避免执行长(慢)命令:keys *, flushall, flushdb, slow lua script, mutil/exec, operate big value(collection) 
- 其实不是单线程: 早期版本是单进程单线程,3.0 版本后实际还有其它的线程, 实现特定功能,如: fysnc  file descriptor,close file descriptor

### Redis 与 Memcached 的对比

Redis 和 Memcached 都是高性能的缓存系统，但它们在功能、特性和适用场景上有显著差异。以下是两者的详细对比：

------

### 1. **数据结构支持**

- **Redis**：
  Redis 支持多种复杂的数据结构，包括：

  - 字符串（String）
  - 列表（List）
  - 集合（Set）
  - 有序集合（Sorted Set）
  - 哈希（Hash）
  - 位图（Bitmap）
  - 地理空间数据（Geo）
  - HyperLogLog

  这些丰富的数据类型使 Redis 能够支持复杂的场景，如排行榜、计数器、队列、实时分析等。

- **Memcached**：
  Memcached 只支持简单的键值对存储，数据类型为字符串或二进制，适合用于简单的缓存需求。

------

### 2. **持久化支持**

- **Redis**：
  支持持久化，可以将内存中的数据保存到磁盘中，提供两种持久化方式：
  - RDB（快照）：定期保存内存数据。
  - AOF（追加日志）：记录每个写操作，恢复数据更加可靠。 Redis 也支持混合持久化（RDB + AOF）。
- **Memcached**：
  不支持持久化，数据仅存储在内存中，一旦服务重启或宕机，数据会丢失。

------

### 3. **高可用和分布式**

- **Redis**：
  - 支持主从复制，提供高可用性。
  - 提供 Redis Sentinel 实现自动故障转移。
  - 支持 Redis Cluster 实现数据分片和分布式存储，能够管理大规模集群。
- **Memcached**：
  Memcached 本身不支持高可用性和分布式功能，但可以通过客户端库实现分布式数据存储（如一致性哈希）。

------

### 4. **性能对比**

- **Redis**：
  性能略低于 Memcached，特别是在简单的键值存储场景下，因为 Redis 支持更多数据结构和功能。但在支持复杂操作（如排行榜、队列等）时，Redis 的性能非常高效。
- **Memcached**：
  在简单的键值存储场景下，Memcached 性能优异，内存利用率也较高。

------

### 5. **内存管理**

- **Redis**：
  Redis 采用内存映射机制，将所有数据存储在内存中，同时支持内存淘汰策略（如 LRU）。当内存达到上限时，可以根据策略清理旧数据。
- **Memcached**：
  Memcached 使用 Slab Allocation 机制，将内存划分为固定大小的块以管理对象。Memcached 的内存利用率较高，但如果存储对象的大小和分配的块不匹配，可能导致内存浪费。

------

### 6. **线程模型**

- **Redis**：
  单线程模型（主线程处理请求），通过 I/O 多路复用机制（如 epoll）实现高效的并发处理。适合大多数场景，且避免了多线程竞争带来的复杂性。
- **Memcached**：
  多线程模型，可以充分利用多核 CPU，在高并发情况下表现优异。

------

### 7. **功能扩展**

- **Redis**：
  - 支持 Lua 脚本执行，提供原子操作。
  - 提供发布/订阅（Pub/Sub）功能，用于消息队列。
  - 支持事务功能。
  - 提供丰富的客户端支持。
- **Memcached**：
  功能简单，专注于缓存服务，不支持复杂功能。

------

### 8. **典型应用场景**

- **Redis**：
  - 缓存系统（热点数据、查询结果缓存）
  - 排行榜和计数器（有序集合）
  - 实时数据分析（HyperLogLog、位图）
  - 队列系统（列表）
  - 地理位置服务（Geo 数据类型）
  - 发布/订阅消息队列
- **Memcached**：
  - 缓存系统（简单键值对存储）
  - 临时数据存储（会话数据缓存）
  - 适用于高并发、简单查询的缓存场景

------

### 9. **社区和生态**

- **Redis**：
  活跃的社区支持，文档和教程丰富，客户端库覆盖多种编程语言（如 Python、Java、Go、C 等），生态系统完善。
- **Memcached**：
  社区支持较少，功能稳定但更新频率较低，适合简单应用需求。

------

### 10. **优缺点对比**

| 特性           | Redis 优点                                  | Memcached 优点                           |
| -------------- | ------------------------------------------- | ---------------------------------------- |
| **功能丰富**   | 支持多种数据类型，适合复杂场景              | 专注于简单的键值对缓存，轻量级，性能更高 |
| **持久化**     | 支持数据持久化，保证数据安全                | 数据存储仅在内存中，简单高效             |
| **分布式**     | 原生支持高可用和分布式（Sentinel、Cluster） | 无原生支持，但客户端实现简单             |
| **内存利用率** | 支持多种内存淘汰策略，灵活管理              | 内存分配机制高效，适合大规模缓存场景     |

------

### 总结

- 如果需要支持复杂的数据结构、持久化、高可用、分布式等功能，**Redis** 是更好的选择。
- 如果只是实现简单的键值对缓存，并且对高性能和内存利用率要求更高，**Memcached** 更加适合。

## Redis 常见应用场景

- 缓存：缓存RDBMS中数据,比如网站的查询结果、商品信息、微博、新闻、消息 
- Session 共享：实现Web集群中的多服务器间的 session 共享 
- 计数器：商品访问排行榜、浏览数、粉丝数、关注、点赞、评论等和次数相关的数值统计场景 
- 社交：朋友圈、共同好友、可能认识他们等 
- 地理位置: 基于地理信息系统GIS（Geographic Information System)实现摇一摇、附近的人、外卖 等功能 
- 消息队列：ELK等日志系统缓存、业务的订阅/发布系统

## 缓存的实现流程

### 数据更新操作流程：

<img src="5day-png\27数据更新操作流程.png" style="zoom:50%;" />

### 数据读操作流程

<img src="5day-png\27数据读操作流程.png" style="zoom:50%;" />

## 缓存穿透,缓存击穿,缓存雪崩和缓存 crash

在分布式缓存系统中（如 Redis 和 Memcached），我们常常会遇到三种缓存相关的问题：**缓存穿透**、**缓存击穿**和**缓存雪崩**。这些问题可能导致缓存失效或性能下降，了解这些问题并采取相应的措施可以有效地优化缓存系统的稳定性和性能。

### 1. **缓存穿透（Cache Penetration）**

#### 定义：

缓存穿透指的是查询一个不存在的数据，而缓存系统没有保存该数据的记录，导致每次都直接访问数据库，绕过了缓存。比如： 发起为id为 “-1” 的数据或id 为特别大不存在的数据。

#### 发生原因：

- 用户查询的数据在缓存中不存在。
- 系统未对不存在的数据进行有效的缓存，导致每次请求都会查询数据库。

#### 解决方法：

- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截 
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效 时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复 用同一个id暴力攻击

- **使用布隆过滤器（Bloom Filter）**：布隆过滤器是一个空间效率高的数据结构，可以用于快速判断一个元素是否在某个集合中。通过在查询缓存之前先使用布隆过滤器判断数据是否存在，可以有效避免缓存穿透。若布隆过滤器也未命中，可以直接查询数据库，并将结果加入缓存。
- **对无效数据缓存**：对查询为空的数据也进行缓存（如设置一个短暂的失效时间），避免频繁查询数据库。

------

### 2. **缓存击穿（Cache Breakdown）**

#### 定义：

缓存击穿指的是在缓存中某个数据过期（或者被删除）时，恰好有大量请求同时访问该数据，导致这些请求直接查询数据库，造成数据库压力激增。

#### 发生原因：

- 设置热点数据永远不过期。

- 缓存中某个数据失效或过期。
- 短时间内有大量的请求同时访问这个数据，造成对数据库的高并发访问，可能会导致数据库崩溃或性能降低。

#### 解决方法：

- **加锁机制**：当缓存失效时，通过加锁的方式保证同一时刻只有一个请求去查询数据库并更新缓存，其他请求等待缓存更新完成后再读取缓存。
- **异步更新缓存**：在缓存失效后，后台异步更新缓存，而不阻塞其他请求的处理。
- **缓存预热**：在缓存失效之前，提前将数据重新加载到缓存中，避免请求直接访问数据库。

------

### 3. **缓存雪崩（Cache Avalanche）**

#### 定义：

缓存雪崩指的是缓存中的大量数据在同一时间失效，导致大量请求同时访问数据库，造成数据库瞬间压力增大，引起数据库压力过大甚至down机。和 缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到 从而查数据库。

#### 发生原因：

- 大量缓存数据设置了相同的过期时间，导致在某个时刻同时失效。
- 大量的请求同时到达这些失效的缓存数据，造成对数据库的巨大压力。

#### 解决方法：

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生 
- 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中 
- 设置热点数据永远不过期

- **设置不同的缓存过期时间**：避免大量数据同时过期，可以将缓存的过期时间设置为一个随机值，避免集中失效。
- **使用多级缓存**：将缓存分为不同的级别，如 Redis + 本地缓存（如 Caffeine），即使 Redis 缓存失效，本地缓存仍然可以起到一定的缓解作用。
- **备份缓存**：使用多节点或集群的方式保证缓存系统的稳定性，避免单点故障造成雪崩。
- **熔断机制**：如果数据库压力过大，可以采取熔断机制，短时间内停止对数据库的访问，给数据库减轻负担。

------

### 4. **缓存崩溃（Cache Crash）**

#### **定义**：

缓存崩溃是指缓存系统由于故障、资源问题或者配置错误等原因导致完全无法提供服务，甚至在系统崩溃的情况下，所有缓存数据都丢失。这通常会导致应用的性能急剧下降，因为所有请求都会回落到数据库查询。

#### **原因**：

- **资源耗尽**：如内存不足、磁盘空间不足等，导致缓存系统无法正常运行。
- **节点故障**：如果缓存系统是分布式部署，当某个节点发生故障或者不可用时，可能会影响到整个缓存系统的正常运行。
- **配置错误**：例如错误的内存配置或不当的过期策略，可能导致缓存系统负载过重或无法正常工作。

#### **解决方案**：

- **资源监控**：定期监控缓存系统的内存、磁盘空间、负载等，及时调整资源配置，防止资源耗尽。
- **高可用架构**：部署缓存集群，确保系统具有高可用性。在单点故障的情况下，其他节点可以继续提供服务。
- **缓存持久化**：启用缓存系统的持久化机制（如 Redis 的 RDB 或 AOF），避免数据丢失，缓存系统崩溃后可以快速恢复。

### 总结对比

| 问题类型      | 定义                                                         | 解决方法                                       |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| **缓存穿透**  | 查询一个不存在的数据，缓存系统没有保存该数据的记录，绕过缓存直接访问数据库。 | 使用布隆过滤器；对空数据缓存（短时间内）       |
| **缓存击穿**  | 缓存中某个数据过期，多个请求同时访问，导致数据库压力激增。   | 加锁机制；异步更新缓存；缓存预热               |
| **缓存雪崩**  | 大量缓存数据同时失效，导致大量请求访问数据库，造成数据库崩溃。 | 设置不同过期时间；多级缓存；备份缓存；熔断机制 |
| **缓存Cache** | 缓存系统由于故障、资源问题或者配置错误等原因导致完全无法提供服务，甚至在系统崩溃的情况下，所有缓存数据都丢失 | 高可用架构；资源监控；缓存持久化               |

## Redis安装

### 包安装

**基于官方仓库包安装**

```bash
https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/
```

```powershell
apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
apt-get update
```

**Install on Ubuntu/Debian**

```powershell
apt-get update
apt-get install redis
systemctl enable redis-server
systemctl start redis-server
```

**Install on Red Hat/Rocky**

```
yum install redis
systemctl enable redis
systemctl start redis
```

### 编译安装

```
https://redis.io/docs/getting-started/installation/install-redis-from-source/
```

```powershell
[root@ubuntu2404 ~]#apt update && apt -y install make gcc libjemalloc-dev libsystemd-dev
[root@ubuntu2404 ~]#wget http://download.redis.io/releases/redis-6.2.4.tar.gz
[root@ubuntu2404 ~]#tar xvf redis-6.2.4.tar.gz
[root@ubuntu2404 redis-6.2.4/~]#cd redis-6.2.4/
[root@ubuntu2404 redis-6.2.4/~]#make -j 2 USE_SYSTEMD=yes PREFIX=/apps/redis install
[root@ubuntu2404 redis-6.2.4/~]#echo 'PATH=/apps/redis/bin:$PATH' >> /etc/profile
[root@ubuntu2404 redis-6.2.4/~]#mkdir /apps/redis/{etc,log,data,run} 
[root@ubuntu2404 redis-6.2.4/~]#cp redis.conf /apps/redis/etc/
#前台启动
[root@ubuntu2404 ~]#redis-server /apps/redis/etc/redis.conf
```

**创建 Redis 用户和设置数据目录权限及修改默认配置文件**

```powershell
[root@ubuntu2404 ~]#useradd -r -s /sbin/nologin redis
#设置目录权限
[root@ubuntu2404 ~]#chown -R redis:redis /apps/redis/
#默认配置文件有问题，需要修改，数据目录默认是根目录，权限不足会导致服务无法启动
[root@ubuntu2404 ~]#vim /apps/redis/etc/redis.conf
#dir ./
dir /apps/redis/data
#pidfile /var/run/redis_6379.pid
pidfile /apps/redis/run/redis_6379.pid
```

**创建 Redis 服务 Service 文件**

```powershell
[root@ubuntu2404 ~]#vim /lib/systemd/system/redis.service
[root@ubuntu2404 ~]#cat /lib/systemd/system/redis.service
# /usr/lib/systemd/system/redis.service
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID			#此行可省略
Type=notify									#如果支持systemd可以启用此行
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000							 #指定此值才支持更大的maxclients值

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload 
[root@ubuntu2404 ~]#systemctl start redis
[root@ubuntu2404 ~]#systemctl status redis
```

```powershell
# /usr/lib/systemd/system/redis.service
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000	

[Install]
WantedBy=multi-user.target
```



### 脚本安装

```powershell
#!/bin/bash
#

#本脚本支持在线和离线安装

REDIS_VERSION=redis-7.4.2
#REDIS_VERSION=redis-7.2.5
#REDIS_VERSION=redis-7.2.4
#REDIS_VERSION=redis-7.2.3
#REDIS_VERSION=redis-7.2.1
#REDIS_VERSION=redis-7.0.11
#REDIS_VERSION=redis-7.0.7
#REDIS_VERSION=redis-7.0.3
#REDIS_VERSION=redis-6.2.6
#REDIS_VERSION=redis-4.0.14

PASSWORD=123456

INSTALL_DIR=/apps/redis

CPUS=`lscpu |awk '/^CPU\(s\)/{print $2}'`

. /etc/os-release

color () {
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"
    SETCOLOR_FAILURE="echo -en \\033[1;31m"
    SETCOLOR_WARNING="echo -en \\033[1;33m"
    SETCOLOR_NORMAL="echo -en \E[0m"
    echo -n "$1" && $MOVE_TO_COL
    echo -n "["
    if [ $2 = "success" -o $2 = "0" ] ;then
        ${SETCOLOR_SUCCESS}
        echo -n $"  OK  "    
    elif [ $2 = "failure" -o $2 = "1"  ] ;then 
        ${SETCOLOR_FAILURE}
        echo -n $"FAILED"
    else
        ${SETCOLOR_WARNING}
        echo -n $"WARNING"
    fi
    ${SETCOLOR_NORMAL}
    echo -n "]"
    echo 
}


prepare(){
    if [ $ID = "centos" -o $ID = "rocky" ];then
        yum  -y install gcc make jemalloc-devel systemd-devel
    else
        apt update 
        apt -y install  gcc make libjemalloc-dev libsystemd-dev
    fi
    if [ $? -eq 0 ];then
        color "安装软件包成功"  0
    else
        color "安装软件包失败，请检查网络配置" 1
        exit
    fi
}

install() {   
    if [ ! -f ${REDIS_VERSION}.tar.gz ];then
        wget http://download.redis.io/releases/${REDIS_VERSION}.tar.gz || { color "Redis 源码下载失败" 1 ; exit; }
    fi
    tar xf ${REDIS_VERSION}.tar.gz -C /usr/local/src
    cd /usr/local/src/${REDIS_VERSION}
    make -j $CUPS USE_SYSTEMD=yes PREFIX=${INSTALL_DIR} install && color "Redis 编译安装完成" 0 || { color "Redis 编译安装失败" 1 ;exit ; }

    ln -s ${INSTALL_DIR}/bin/redis-*  /usr/local/bin/
    
    mkdir -p ${INSTALL_DIR}/{etc,log,data,run}
  
    cp redis.conf  ${INSTALL_DIR}/etc/

    sed -i -e 's/bind 127.0.0.1/bind 0.0.0.0/'  -e "/# requirepass/a requirepass $PASSWORD"  -e "/^dir .*/c dir ${INSTALL_DIR}/data/"  -e "/logfile .*/c logfile ${INSTALL_DIR}/log/redis-6379.log"  -e  "/^pidfile .*/c  pidfile ${INSTALL_DIR}/run/redis_6379.pid" ${INSTALL_DIR}/etc/redis.conf


    if id redis &> /dev/null ;then 
         color "Redis 用户已存在" 1 
    else
         useradd -r -s /sbin/nologin redis
         color "Redis 用户创建成功" 0
    fi

    chown -R redis.redis ${INSTALL_DIR}

    cat >> /etc/sysctl.conf <<EOF
net.core.somaxconn = 1024
vm.overcommit_memory = 1
EOF
    sysctl -p 
    if [ $ID = "centos" -o $ID = "rocky" ];then
        echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local
        chmod +x /etc/rc.d/rc.local
        /etc/rc.d/rc.local 
    else 
        echo -e '#!/bin/bash\necho never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.local
        chmod +x /etc/rc.local
        /etc/rc.local
    fi


cat > /lib/systemd/system/redis.service <<EOF
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=${INSTALL_DIR}/bin/redis-server ${INSTALL_DIR}/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT \$MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target

EOF
     systemctl daemon-reload 
     systemctl enable --now  redis &> /dev/null 
     if [ $? -eq 0 ];then
         color "Redis 服务启动成功,Redis信息如下:"  0 
     else
        color "Redis 启动失败" 1 
        exit
     fi
     sleep 2
     redis-cli -a $PASSWORD INFO Server 2> /dev/null
}

prepare 

install 

```

### 消除启动时的三个Warning提示信息(可选)

**1 Tcp backlog**

```
WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
```

```powershell
Tcp backlog 是指TCP的第三次握手服务器端收到客户端 ack确认号之后到服务器用Accept函数处理请求前的队列长度，即全连接队列
注意：Ubuntu22.04以上版本默认值满足要求，不再有此告警
#半连接队列
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/tcp_max_syn_backlog
128
[root@ubuntu2204 ~]#cat /proc/sys/net/ipv4/tcp_max_syn_backlog
128

#全连接队列默认值
[root@ubuntu2404 ~]#cat /proc/sys/net/core/somaxconn
4096
[root@ubuntu2204 ~]#cat /proc/sys/net/core/somaxconn
4096
[root@ubuntu2004 ~]#cat /proc/sys/net/core/somaxconn
4096
[root@rocky8 ~]#cat /proc/sys/net/core/somaxconn
128

解决办法
#修改配置
#vim /etc/sysctl.conf
net.core.somaxconn = 1024
#sysctl -p
```

**2 overcommit_memory**

```
WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
```

```powershell

内核参数overcommit_memory 实现内存分配策略,可选值有三个：0、1、2
0 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则内存申请失败，并把错误返回给应用进程
1 表示内核允许分配所有的物理内存，而不管当前的内存状态如何
2 表示内核允许分配超过所有物理内存和交换空间总和的内存

默认值为0
[root@ubuntu2404 ~]#sysctl vm.overcommit_memory
vm.overcommit_memory = 0

解决办法
修改为1
[root@ubuntu2404 ~]#vim /etc/sysctl.conf
[root@ubuntu2404 ~]#sysctl -p
vm.overcommit_memory = 1			#新版只允许1，不支持2

```

**3 transparent hugepage**

```
WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
警告您的内核中启用了透明大页 (THP) 支持。这将导致 Redis 出现延迟和内存使用问题。要解决此问题，请以 root 身份运行命令“echo never > /sys/kernel/mm/transparent_hugepage/enabled”，并将其添加到 /etc/rc.local 中，以便在重新启动后保留设置。禁用THP后必须重新启动Redis。
```

```powershell
注意：Ubuntu24.04 .22.04 默认值满足要求，不再有此告警
[root@ubuntu2404 ~]#cat /sys/kernel/mm/transparent_hugepage/enabled
always [madvise] never
[root@ubuntu2204 ~]#cat /sys/kernel/mm/transparent_hugepage/enabled
always [madvise] never
[root@ubuntu2004 ~]#cat /sys/kernel/mm/transparent_hugepage/enabled
always [madvise] never
[root@rocky8 ~]#cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@centos7 ~]#cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never

解决办法
#ubuntu开机配置
[root@ubuntu2004 ~]#cat /etc/rc.local 
#!/bin/bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@ubuntu2004 ~]#chmod +x /etc/rc.local

#CentOS开机配置
[root@centos8 ~]#echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local 
[root@centos8 ~]#cat /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@centos8 ~]#chmod +x /etc/rc.d/rc.local
```



## 开启 Redis 多实例

```powershell
编译安装实现多实例
[root@ubuntu2404 etc]#cp redis.conf redis6379.conf 
[root@ubuntu2404 etc]#cp redis.conf redis6380.conf 
[root@ubuntu2404 etc]#cp redis.conf redis6381.conf
[root@ubuntu2404 etc]#grep 6379 redis6379.conf 
# Accept connections on the specified port, default is 6379 (IANA #815344).
port 6379
# tls-port 6379
pidfile /apps/redis/run/redis_6379.pid
logfile /apps/redis/log/redis-6379.log
# cluster-config-file nodes-6379.conf
# cluster-announce-tls-port 6379
[root@ubuntu2404 etc]#sed -i 's#6379#6380#g' redis6380.conf 
[root@ubuntu2404 etc]#grep 6380 redis6380.conf 
# Accept connections on the specified port, default is 6380 (IANA #815344).
port 6380
# tls-port 6380
pidfile /apps/redis/run/redis_6380.pid
logfile /apps/redis/log/redis-6380.log
# cluster-config-file nodes-6380.conf
# cluster-announce-tls-port 6380
# cluster-announce-bus-port 6380
[root@ubuntu2404 etc]#sed -i 's#6379#6381#g' redis6381.conf 
[root@ubuntu2404 etc]#grep 6381 redis6381.conf
# Accept connections on the specified port, default is 6381 (IANA #815344).
port 6381
# tls-port 6381
pidfile /apps/redis/run/redis_6381.pid
logfile /apps/redis/log/redis-6381.log
# cluster-config-file nodes-6381.conf
# cluster-announce-tls-port 6381

[root@ubuntu2404 ~]#cat /lib/systemd/system/redis6379.service 
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis6379.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#cat /lib/systemd/system/redis6380.service 
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis6380.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#cat /lib/systemd/system/redis6381.service 
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/apps/redis/bin/redis-server /apps/redis/etc/redis6381.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```

## 安装的相关程序介绍

```powershell
[root@ubuntu2404 ~]#ll /apps/redis/bin/
total 31168
drwxr-xr-x 2 redis redis     4096 Jan 18 16:33 ./
drwxr-xr-x 7 redis redis     4096 Jan 18 16:33 ../
-rwxr-xr-x 1 redis redis  6705656 Jan 18 16:33 redis-benchmark*						#性能测试程序
lrwxrwxrwx 1 redis redis       12 Jan 18 16:33 redis-check-aof -> redis-server*		#AOF文件检查程序
lrwxrwxrwx 1 redis redis       12 Jan 18 16:33 redis-check-rdb -> redis-server*		#RDB文件检查程序
-rwxr-xr-x 1 redis redis  7708704 Jan 18 16:33 redis-cli*							#客户端程序
lrwxrwxrwx 1 redis redis       12 Jan 18 16:33 redis-sentinel -> redis-server*		#哨兵程序，软链接到服务主程序
-rwxr-xr-x 1 redis redis 17484352 Jan 18 16:33 redis-server*						#服务主程序
```

```powershell
#默认为本机无密码连接
redis-cli
#远程客户端连接,注意:Redis没有用户的概念
redis-cli -h <Redis服务器IP> -p <PORT> -a <PASSWORD> --no-auth-warning
```

### shell连接数据库

```powershell
#!/bin/bash

NUM=100
PASS=123456
HOST=127.0.0.1
PORT=6379
DATABASE=0


for i in `seq $NUM`;do
    redis-cli -h ${HOST}  -a "$PASS" -p ${PORT} -n ${DATABASE} --no-auth-warning  set key${i} value${i}
    #redis-cli -h ${HOST} -p ${PORT} -n ${DATABASE} --no-auth-warning  set key${i} value${i}
    echo "key${i} value${i} 写入完成"
done
echo "$NUM个key写入到Redis完成" 
```

### python连接数据库

```python
#!/usr/bin/python3 
import redis
pool = redis.ConnectionPool(host="127.0.0.1",port=6379,password="123456",decode_responses=True)
r = redis.Redis(connection_pool=pool)
for i in range(100000):
    r.set("k%d" % i,"v%d" % i)
    data=r.get("k%d" % i)
    print(data)
```

### golang连接数据库

```golang
package main

import (
        "context"
        "fmt"
        "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
        rdb := redis.NewClient(&redis.Options{
                Addr:     "127.0.0.1:6379",
                Password: "123456",
                DB:       0,
        })

        // 测试连接
        _, err := rdb.Ping(ctx).Result()
        if err != nil {
                fmt.Printf("连接 Redis 出错，错误信息：%v\n", err)
                return
        }

        // 批量插入数据
        for i := 1; i <= 10000; i++ {
                key := fmt.Sprintf("key%d", i)
                value := fmt.Sprintf("value%d", i)

                err := rdb.Set(ctx, key, value, 0).Err()
                if err != nil {
                        fmt.Printf("设置键值对失败: %v\n", err)
                        return
                }
        }

        // 使用 SCAN 遍历键
        var cursor uint64
        for {
                keys, newCursor, err := rdb.Scan(ctx, cursor, "key*", 100).Result()
                if err != nil {
                        fmt.Printf("扫描键失败: %v\n", err)
                        return
                }
                for _, key := range keys {
                        fmt.Println(key)
                }
                if newCursor == 0 {
                        break
                }
                cursor = newCursor
        }
}
```

## Redis 基础功能

### INFO

```powershell
127.1:6379> INFO
# Server
redis_version:7.4.2
redis_git_sha1:00000000
redis_git_dirty:1
redis_build_id:84401268a63069d0
redis_mode:standalone
os:Linux 6.8.0-48-generic x86_64
arch_bits:64
monotonic_clock:POSIX clock_gettime
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:13.3.0
process_id:8111
process_supervised:systemd
run_id:3bc94deed07b6dc92f7d38f56f1c7b1199ca1ec1
tcp_port:6379
server_time_usec:1737205338274646
uptime_in_seconds:16099
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:9152090
executable:/apps/redis/bin/redis-server
config_file:/apps/redis/etc/redis.conf
io_threads_active:0
listener0:name=tcp,bind=0.0.0.0,bind=-::1,port=6379

# Clients
connected_clients:1
cluster_connections:0
maxclients:10000
client_recent_max_input_buffer:20480
client_recent_max_output_buffer:0
blocked_clients:0
tracking_clients:0
pubsub_clients:0
watching_clients:0
clients_in_timeout_table:0
total_watched_keys:0
total_blocking_keys:0
total_blocking_keys_on_nokey:0

# Memory
used_memory:9371328
used_memory_human:8.94M
used_memory_rss:17825792
used_memory_rss_human:17.00M
used_memory_peak:9589384
used_memory_peak_human:9.15M
used_memory_peak_perc:97.73%
used_memory_overhead:6397592
used_memory_startup:946400
used_memory_dataset:2973736
used_memory_dataset_perc:35.30%
allocator_allocated:10095568
allocator_active:10444800
allocator_resident:13451264
allocator_muzzy:0
total_system_memory:2013294592
total_system_memory_human:1.88G
used_memory_lua:31744
used_memory_vm_eval:31744
used_memory_lua_human:31.00K
used_memory_scripts_eval:0
number_of_cached_scripts:0
number_of_functions:0
number_of_libraries:0
used_memory_vm_functions:32768
used_memory_vm_total:64512
used_memory_vm_total_human:63.00K
used_memory_functions:192
used_memory_scripts:192
used_memory_scripts_human:192B
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.03
allocator_frag_bytes:273200
allocator_rss_ratio:1.29
allocator_rss_bytes:3006464
rss_overhead_ratio:1.33
rss_overhead_bytes:4374528
mem_fragmentation_ratio:1.91
mem_fragmentation_bytes:8477352
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_total_replication_buffers:0
mem_clients_slaves:0
mem_clients_normal:1928
mem_cluster_links:0
mem_aof_buffer:0
mem_allocator:jemalloc-5.3.0
mem_overhead_db_hashtable_rehashing:0
active_defrag_running:0
lazyfree_pending_objects:0
lazyfreed_objects:0

# Persistence
loading:0
async_loading:0
current_cow_peak:0
current_cow_size:0
current_cow_size_age:0
current_fork_perc:0.00
current_save_keys_processed:0
current_save_keys_total:0
rdb_changes_since_last_save:8968
rdb_bgsave_in_progress:0
rdb_last_save_time:1737205107
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
rdb_saves:4
rdb_last_cow_size:1396736
rdb_last_load_keys_expired:0
rdb_last_load_keys_loaded:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_rewrites:0
aof_rewrites_consecutive_failures:0
aof_last_write_status:ok
aof_last_cow_size:0
module_fork_in_progress:0
module_fork_last_cow_size:0

# Stats
total_connections_received:108
total_commands_processed:211317
instantaneous_ops_per_sec:0
total_net_input_bytes:6653890
total_net_output_bytes:2745642
total_net_repl_input_bytes:0
total_net_repl_output_bytes:0
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
instantaneous_input_repl_kbps:0.00
instantaneous_output_repl_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_subkeys:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
expire_cycle_cpu_milliseconds:327
evicted_keys:0
evicted_clients:0
evicted_scripts:0
total_eviction_exceeded_time:0
current_eviction_exceeded_time:0
keyspace_hits:100000
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
pubsubshard_channels:0
latest_fork_usec:459
total_forks:4
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0
total_active_defrag_time:0
current_active_defrag_time:0
tracking_total_keys:0
tracking_total_items:0
tracking_total_prefixes:0
unexpected_error_replies:0
total_error_replies:2
dump_payload_sanitizations:0
total_reads_processed:211426
total_writes_processed:211326
io_threaded_reads_processed:0
io_threaded_writes_processed:0
client_query_buffer_limit_disconnections:0
client_output_buffer_limit_disconnections:0
reply_buffer_shrinks:6
reply_buffer_expands:0
eventloop_cycles:371474
eventloop_duration_sum:32486901
eventloop_duration_cmd_sum:421010
instantaneous_eventloop_cycles_per_sec:9
instantaneous_eventloop_duration_usec:203
acl_access_denied_auth:0
acl_access_denied_cmd:0
acl_access_denied_key:0
acl_access_denied_channel:0

# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:046087863b712edcb7878ac7bbe584ea85df2c22
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:19.482926
used_cpu_user:19.029672
used_cpu_sys_children:0.028023
used_cpu_user_children:0.061131
used_cpu_sys_main_thread:24.943453
used_cpu_user_main_thread:13.568445

# Modules

# Errorstats
errorstat_ERR:count=2

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=110000,expires=0,avg_ttl=0,subexpiry=0
```

```
redis-cli -a 123456 info server
127.1:6379> info server
```

### select

**切换数据库，相当于在MySQL的 USE DBNAME 指令**

```powershell
127.1:6379> select 0
OK
127.1:6379> select 1
OK
127.1:6379[1]> 
```

```bash
#redis默认16个数据库，可以修改
[root@ubuntu2404 ~]#vim /apps/redis/etc/redis.conf 
databases 16
```



### KEYS

| 命令   | 时间复杂度 |
| ------ | ---------- |
| keys   | O(n)       |
| dbsize | O(1)       |
| del    | O(1)       |
| exists | O(1)       |
| expire | O(1)       |
| type   | O(1)       |



```powershell
127.0.0.1:6379[15]> SELECT 0
OK
127.0.0.1:6379> KEYS *
1) "9527"
2) "9526"
3) "course"
4) "list1"
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> KEYS *
(empty list or set)
127.0.0.1:6379[1]> 
redis>MSET one 1 two 2 three 3 four 4  # 一次设置 4 个 key
OK
redis> KEYS *o*
1) "four"
2) "two"
3) "one"
redis> KEYS t??
1) "two"
redis> KEYS t[w]*
1) "two"
redis> KEYS *  # 匹配数据库内所有 key
1) "four"
2) "three"
3) "two"
4) "one"
```

### DBSIZE

**返回当前库下的所有key 数量**

```powershell
127.0.0.1:6379> select 0
OK
127.0.0.1:6379> dbsize
(integer) 110000
```

### FLUSHDB

**强制清空当前库中的所有key，此命令慎用！**

```
127.0.0.1:6379[1]> mset name1 zs name2 li name3 wu
OK
127.0.0.1:6379[1]> dbsize
(integer) 3
127.0.0.1:6379[1]> flushdb
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
```

### FLUSHALL

**强制清空当前Redis服务器所有数据库中的所有key，即删除所有数据，此命令慎用！**

```powershell
127.0.0.1:6379> FLUSHALL
OK
#生产建议修改配置使用rename-command禁用此命令
vim /etc/redis.conf
rename-command FLUSHALL ""   #flushdb和和AOF功能冲突，需要设置 appendonly no,不区分命令大小写，但和flushall （v7.2.3不冲突）
```

### SHUTDOWN

```bash
可用版本： >= 1.0.0
时间复杂度： O(N)，其中 N 为关机时需要保存的数据库键数量。
SHUTDOWN 命令执行以下操作：

关闭Redis服务,停止所有客户端连接

如果有至少一个保存点在等待，执行 SAVE 命令

如果 AOF 选项被打开，更新 AOF 文件

关闭 redis 服务器(server)

如果持久化被打开的话， SHUTDOWN 命令会保证服务器正常关闭而不丢失任何数据。

另一方面，假如只是单纯地执行 SAVE 命令，然后再执行 QUIT 命令，则没有这一保证 —— 因为在执行SAVE 之后、执行 QUIT 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 QUIT 就会造成数据丢失。

#建议禁用此指令
vim /etc/redis.conf
rename-command shutdown ""
```

## Redis 配置文件说明

```powershell
[root@ubuntu2404 ~]#cat /apps/redis/etc/redis.conf  | grep -Ev '^$|^#'
bind 0.0.0.0 -::1	#指定监听地址，支持用空格隔开的多个监听IP
protected-mode yes	#redis3.2之后加入的新特性，在没有设置bind IP和密码的时候,redis只允许访问127.0.0.1:6379，可以远程连接，但当访问将提示警告信息并拒绝远程访问,redis-7版本后，只要没有密码就不能远程访问
port 6379			#监听端口,默认6379/tcp
tcp-backlog 511		#三次握手的时候server端收到client ack确认号之后的队列值，即全连接队列长度
timeout 0			#客户端和Redis服务端的连接超时时间，默认是0，表示永不超时
tcp-keepalive 300	#tcp 会话保持时间300s
daemonize no		#默认no,即直接运行redis-server程序时,不作为守护进程运行，而是以前台方式运行，如果想在后台运行需改成yes,当redis作为守护进程运行的时候，它会写一个 pid 到/var/run/redis.pid 文件
supervised no		#和OS相关参数，可设置通过upstart和systemd管理Redis守护进程，centos7后都使用systemd
pidfile /apps/redis/run/redis_6379.pid		#pid文件路径,可以修改为/apps/redis/run/redis_6379.pid
loglevel notice		#日志级别
logfile /apps/redis/log/redis-6379.log		#日志路径,示例:logfile "/apps/redis/log/redis_6379.log"
databases 16		#设置数据库数量，默认：0-15，共16个库
always-show-logo no	#在启动redis 时是否显示或在日志中记录记录redis的logo
save 900 1 			#在900秒内有1个key内容发生更改,就执行快照机制
save 300 10	 		#在300秒内有10个key内容发生更改,就执行快照机制
save 60 10000  		#60秒内如果有10000个key以上的变化，就自动快照备份
save 900 1 300 10 60 10000 	#新版本支持这样写	
set-proc-title yes	#作用：指定是否启用 Redis 设置进程标题的功能。启用后，Redis 会在系统进程列表（如通过 ps 命令查看）中显示自定义的进程标题，以便于管理员识别和管理多个 Redis 实例。取值：yes：启用自定义进程标题。no：禁用该功能。默认值：yes示例：bashps aux | grep redis如果启用，进程标题可能类似于：redis-server 127.0.0.1:6379
proc-title-template "{title} {listen-addr} {server-mode}"		#作用：定义 Redis 的进程标题模板。该模板支持使用占位符动态生成进程标题，以便区分不同实例。占位符说明：{title}：Redis 的默认标题（通常是 redis-server）。{listen-addr}：Redis 监听的地址和端口（如 127.0.0.1:6379）。{server-mode}：Redis 的运行模式，例如 standalone（单机模式）或 sentinel（哨兵模式）。示例：proc-title-template "{title} {listen-addr} {server-mode}"在启用了 set-proc-title yes 后，进程标题可能显示为：redis-server 127.0.0.1:6379 standalone
locale-collate ""		#作用：设置 Redis 的语言环境用于字符串排序（collation）。如果不指定或设置为空，Redis 将使用操作系统的默认语言环境。取值：任何有效的语言环境值，例如 en_US.UTF-8、zh_CN.UTF-8。空字符串（""）：表示使用系统的默认语言环境。注意事项：修改此选项可能影响 Redis 的字符串比较行为，特别是与排序相关的操作。确保系统中支持指定的语言环境，否则可能导致 Redis 启动失败。示例：locale-collate "en_US.UTF-8"
stop-writes-on-bgsave-error yes		#默认为yes时,可能会因空间满等原因快照无法保存出错时，会禁止redis写入操作，生产建议为no
									#此项只针对配置文件中的自动save有效
rdbcompression yes	#持久化到RDB文件时，是否压缩，"yes"为压缩，"no"则反之
rdbchecksum yes		#是否对备份文件开启RC64校验，默认是开启
dbfilename dump.rdb	#快照文件名
rdb-del-sync-files no
dir /apps/redis/data/		#快照文件保存路径，示例：dir "/apps/redis/data"

#主从复制相关
# replicaof <masterip> <masterport> #指定复制的master主机地址和端口，5.0版之前的指令为slaveof 
# masterauth <master-password> #指定复制的master主机的密码
replica-serve-stale-data yes	#当从库同主库失去连接或者复制正在进行，从机库有两种运行方式：
								#1、设置为yes(默认设置)，从库会继续响应客户端的读请求，此为建议值
								#2、设置为no，除去特定命令外的任何请求都会返回一个错误"SYNC with master in progress"。
replica-read-only yes			#是否设置从库只读，建议值为yes,否则主库同步从库时可能会覆盖数据，造成数据丢失
repl-diskless-sync yes			#是否使用socket方式复制数据(无盘同步)，新slave第一次连接master时需要做数据的全量同步，redis server就要从内存dump出新的RDB文件，然后从master传到slave，有两种方式把RDB文件传输给客户端：
								#1、基于硬盘（disk-backed）：为no时，master创建一个新进程dump生成RDB磁盘文件，RDB完成之后由父进程（即主进程）将RDB文件发送给slaves，此为默认值
								#2、基于socket（diskless）：master创建一个新进程直接dump RDB至slave的网络socket，不经过主进程和硬盘
								#推荐使用基于硬盘（为no），是因为RDB文件创建后，可以同时传输给更多的slave，但是基于socket(为yes)， 新slave连接到master之后得逐个同步数据。只有当磁盘I/O较慢且网络较快时，可用diskless(yes),否则一般建议使用磁盘(no)

repl-diskless-sync-delay 5		#diskless时复制的服务器等待的延迟时间，设置0为关闭，在延迟时间内到达的客户端，会一起通过diskless方式同步数据，但是一旦复制开始，master节点不会再接收新slave的复制请求，直到下一次同步开始才再接收新请求。即无法为延迟时间后到达的新副本提供服务，新副本将排队等待下一次RDB传输，因此服务器会等待一段时间才能让更多副本到达。推荐值：30-60
repl-ping-replica-period 10 	#slave根据master指定的时间进行周期性的PING master,用于监测master状态,默认10s
repl-timeout 60	 				#复制连接的超时时间，需要大于repl-ping-slave-period，否则会经常报超时
repl-diskless-sync-max-replicas 0	#作用：限制允许使用无盘同步（diskless replication）的最大从节点数量。无盘同步是指在主节点不将数据快照（RDB 文件）写入磁盘的情况下，直接通过网络将数据流式传输给从节点，从而提高性能。取值：0（默认值）：表示不限制从节点数量，即所有从节点都可以使用无盘同步。正整数：限制可以使用无盘同步的最大从节点数量。当超过该数量时，其余从节点将使用磁盘同步。场景：在网络较快且磁盘 I/O 成本较高的环境中，无盘同步可以显著提高性能。设置一个非零值可以避免无盘同步时占用过多的内存（因为无盘同步直接在内存中生成数据快照）。示例：repl-diskless-sync-max-replicas 3 表示最多允许 3 个从节点使用无盘同步。	
repl-diskless-load disabled		#作用：控制从节点在接收主节点发送的 RDB 文件时，是否使用无盘加载（diskless load）。无盘加载是指在从节点接收到 RDB 文件后，直接将数据加载到内存，而不需要将其写入磁盘。取值：disabled（默认值）：禁用无盘加载。从节点会将接收到的 RDB 文件先写入磁盘，再加载到内存。on-empty-db：如果从节点的数据库为空，则使用无盘加载；否则禁用无盘加载。force: 始终启用无盘加载，无论数据库是否为空。场景：优点：避免从节点对磁盘的频繁写入操作，适合磁盘 I/O 成本高的环境。缺点：如果网络不稳定或加载过程中失败，数据会丢失，因为没有磁盘备份。推荐：在高性能网络环境中（如本地数据中心）使用 on-empty-db，兼顾性能和安全性。在对数据可靠性要求较高的场景，保留默认的 disabled。示例：repl-diskless-load on-empty-db
repl-disable-tcp-nodelay no		#是否在slave套接字发送SYNC之后禁用 TCP_NODELAY，如果选择"yes"，Redis将合并多个报文为一个大的报文，从而使用更少数量的包向slaves发送数据，但是将使数据传输到slave上有延迟，Linux内核的默认配置会达到40毫秒，如果 "no" ，数据传输到slave的延迟将会减少，但要使用更多的带宽
repl-backlog-size 512mb 		#复制缓冲区内存大小，当slave断开连接一段时间后，该缓冲区会累积复制副本数据，因此当slave 重新连接时，通常不需要完全重新同步，只需传递在副本中的断开连接后没有同步的部分数据即可。只有在至少有一个slave连接之后才分配此内存空间,建议建立主从时此值要调大一些或在低峰期配置,否则会导致同步到slave失败
repl-backlog-ttl 3600 			#多长时间内master没有slave连接，就清空backlog缓冲区
replica-priority 100			#当master不可用，哨兵Sentinel会根据slave的优先级选举一个master，此值最低的slave会优先当选master，而配置成0，永远不会被选举，一般多个slave都设为一样的值，让其自动选择
								#min-replicas-to-write 3 #至少有3个可连接的slave，mater才接受写操作
								#min-replicas-max-lag 10 #和上面至少3个slave的ping延迟不能超过10秒，否则master也将停止写操作
requirepass foobared 			#设置redis连接密码，之后需要AUTH pass,如果有特殊符号，用" "引起来,生产建议设置
rename-command 					#重命名一些高危命令，示例：rename-command FLUSHALL "" 禁用命令
   								#示例: rename-command del kang
maxclients 10000 				#Redis最大连接客户端,默认值10000
maxmemory <bytes> 				#redis使用的最大内存，单位为bytes字节，0为不限制，建议设为物理内存一半，8G内存的计算方式8(G)*1024(MB)1024(KB)*1024(Kbyte)，需要注意的是缓冲区是不计算在maxmemory内,生产中如果不设置此项,可能会导致OOM
maxmemory-policy 				# MAXMEMORY POLICY：当达到最大内存时，Redis 将如何选择要删除的内容。您可以从以下行为中选择一种：
									#
									# volatile-lru -> Evict 使用近似 LRU（最近最少使用算法），只有设置了过期时间的键
									# allkeys-lru -> 使用近似 LRU 驱逐任何键。
									# volatile-lfu -> 使用近似 LFU 驱逐，只有设置了过期时间的键。
									# allkeys-lfu -> 使用近似 LFU 驱逐任何键。
									# volatile-random -> 删除设置了过期时间的随机密钥。
									# allkeys-random -> 删除一个随机密钥，任何密钥。
									# volatile-ttl -> 删除过期时间最近的key（次TTL）
									# noeviction -> 不要驱逐任何东西，只是在写操作时返回一个错误。此为默认值
									#
									# LRU 表示最近最少使用
									# LFU 表示最不常用
									#
									# LRU、LFU 和 volatile-ttl 都是使用近似随机算法实现的。
									#
									# 注意：使用上述任何一种策略，当没有合适的键用于驱逐时，Redis 将在需要更多内存的写操作时返回错误。这些通常是创建新密钥、添加数据或修改现有密钥的命令。一些示例是：SET、INCR、HSET、LPUSH、SUNIONSTORE、SORT（由于 STORE 参数）和 EXEC（如果事务包括任何需要内存的命令）。
appendonly no 						#是否开启AOF日志记录，默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了，但是redis如果中途宕机，会导致可能有几分钟的数据丢失(取决于dump数据的间隔时间)，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性，Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。默认不启用此功能
appendfilename "appendonly.aof" 	#文本文件AOF的文件名，存放在dir指令指定的目录中
appendfsync everysec 					#aof持久化策略的配置
										#no表示由操作系统保证数据同步到磁盘,Linux的默认fsync策略是30秒，最多会丢失30s的数据
										#always表示每次写入都执行fsync，以保证数据同步到磁盘,安全性高,性能较差
										#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据,此为默认值,也生产建议值
										#同时在执行bgrewriteaof操作和主进程写aof文件的操作，两者都会操作磁盘，而bgrewriteaof往往会涉及大量磁盘操作，这样就会造成主进程在写aof文件的时候出现阻塞的情形,以下参数实现控制
no-appendfsync-on-rewrite no 			#在aof rewrite期间,是否对aof新记录的append暂缓使用文件同步策略,主要考虑磁盘IO开支和请求阻塞时间。
										#默认为no,表示"不暂缓",新的aof记录仍然会被立即同步到磁盘，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题
										#为yes,相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？Linux的默认fsync策略是30秒，最多会丢失30s的数据,但由于yes性能较好而且会避免出现阻塞因此比较推荐
										#rewrite 即对aof文件进行整理,将空闲空间回收,从而可以减少恢复数据时间		
acllog-max-len 128						#作用：控制 ACL 日志的最大条目数。Redis 记录访问控制列表（ACL）相关的日志，包括用户访问权限违规的情况。取值：数字：指定最大日志条目数。默认值：128 条。注意：超过此数量的日志条目会被自动删除（先进先出）
requirepass 123456						#作用：设置 Redis 的全局访问密码。客户端连接 Redis 时，需要提供此密码进行身份验证。取值：字符串：指定密码。空字符串（未设置）：表示不启用密码保护。示例：requirepass "mypassword"注意：推荐将密码设置为复杂的随机字符串，增强安全性。如果使用 Redis 6.0 或更高版本，建议通过 ACL 设置用户权限，替代简单的 requirepass。
lazyfree-lazy-eviction no				#作用：控制 Redis 是否在内存驱逐（eviction）时使用异步删除。取值：yes：使用异步删除。no：使用同步删除（默认）。
lazyfree-lazy-expire no					#作用：控制 Redis 是否在键过期时使用异步删除。取值：yes：使用异步删除。no：使用同步删除（默认）
lazyfree-lazy-server-del no				#作用：控制 Redis 是否在执行 DEL 命令时使用异步删除。取值：yes：使用异步删除。no：使用同步删除（默认）
replica-lazy-flush no					#作用：控制从节点在加载主节点数据时是否使用异步删除旧数据。取值：yes：使用异步删除。no：使用同步删除（默认）。
lazyfree-lazy-user-del no				#作用：控制 Redis 是否在删除 ACL 用户时，使用异步删除相关权限数据。
lazyfree-lazy-user-flush no				#作用：控制 Redis 是否在清空所有 ACL 用户数据时，使用异步删除。
oom-score-adj no						#作用：控制是否允许调整 Redis 进程的 OOM (Out-Of-Memory) 分数。OOM 分数由操作系统用来决定当系统内存不足时，哪些进程应该被杀掉。取值：yes：允许调整。no：不调整（默认）。
oom-score-adj-values 0 200 800			#作用：定义不同 OOM 分数的调整值，适用于 Redis 主进程及其子进程。示例：0：Redis 主进程的 OOM 分数（不会被优先杀掉）。200：RDB/AOF 写子进程的 OOM 分数。800：其他子进程的 OOM 分数。
disable-thp yes							#作用：禁用透明大页（Transparent Huge Pages, THP）。THP 会影响 Redis 的性能，因此建议禁用。
appendonly no							#作用：启用或禁用 AOF 持久化功能。取值：yes：启用 AOF。no：禁用 AOF（默认）
appendfilename "appendonly.aof"			#作用：设置 AOF 文件的名称
appenddirname "appendonlydir"			#作用：设置 AOF 文件所在的目录。
appendfsync everysec					#作用：设置 AOF 文件的同步频率。取值：always：每次写入都立即同步（性能差，数据最安全）。everysec（默认）：每秒同步一次，折中性能与数据安全性。no：由操作系统决定同步时机（性能最佳，但风险最高）。
no-appendfsync-on-rewrite no			#作用：指定在 AOF 重写期间是否禁用同步。取值：yes：禁用同步，减少 I/O 开销（风险高）。no（默认）：正常同步。
auto-aof-rewrite-percentage 100		#当Aof log增长超过指定百分比例时，重写AOF文件，设置为0表示不自动重写Aof日志，重写是为了使aof体积保持最小，但是还可以确保保存最完整的数据
auto-aof-rewrite-min-size 64mb		#触发aof rewrite的最小文件大小
aof-load-truncated yes				#是否加载由于某些原因导致的末尾异常的AOF文件(主进程被kill/断电等)，建议yes
aof-use-rdb-preamble yes			#redis4.0新增RDB-AOF混合持久化格式，在开启了这个功能之后，AOF重写产生的文件将同时包含RDB格式的内容和AOF格式的内容，其中RDB格式的内容用于记录已有的数据，而AOF格式的内容则用于记录最近发生了变化的数据，这样Redis就可以同时兼有RDB持久化和AOF持久化的优点（既能够快速地生成重写文件，也能够在出现问题时，快速地载入数据）,默认为no,即不启用此功能	
aof-timestamp-enabled no			#作用：控制是否在 AOF（Append-Only File）文件中记录时间戳。如果启用，Redis 会在每条 AOF 记录之前附加时间戳信息，方便追踪和调试。取值：yes：在 AOF 文件中启用时间戳记录。no（默认）：不记录时间戳
lua-time-limit 5000 				#lua脚本的最大执行时间，单位为毫秒
cluster-enabled yes 				#是否开启集群模式，默认不开启,即单机模式
cluster-config-file nodes-6379.conf #由node节点自动生成的集群配置文件名称
cluster-node-timeout 15000 			#集群中node节点连接超时时间，单位ms,超过此时间，会踢出集群
cluster-replica-validity-factor 10 	#单位为次,在执行故障转移的时候可能有些节点和master断开一段时间导致数据比较旧，这些节点就不适用于选举为master，超过这个时间的就不会被进行故障转移,不能当选master，计算公式：(node-timeout * replica-validity-factor) + repl-ping-replica-period 
cluster-migration-barrier 1 		#集群迁移屏障，一个主节点至少拥有1个正常工作的从节点，即如果主节点的slave节点故障后会将多余的从节点分配到当前主节点成为其新的从节点。
cluster-require-full-coverage yes 	#集群请求槽位全部覆盖，如果一个主库宕机且没有备库就会出现集群槽位不全，那么yes时redis集群槽位验证不全,就不再对外提供服务(对key赋值时,会出现CLUSTERDOWN The cluster is down的提示,cluster_state:fail,但ping 仍PONG)，而no则可以继续使用,但是会出现查询数据查不到的情况(因为有数据丢失)。生产建议为no
cluster-replica-no-failover no 	#如果为yes,此选项阻止在主服务器发生故障时尝试对其主服务器进行故障转移。 但是，主服务器仍然可以执行手动强制故障转移，一般为no
								#Slow log 是 Redis 用来记录超过指定执行时间的日志系统，执行时间不包括与客户端交谈，发送回复等I/O操作，而是实际执行命令所需的时间（在该阶段线程被阻塞并且不能同时为其它请求提供服务）,由于slow log 保存在内存里面，读写速度非常快，因此可放心地使用，不必担心因为开启 slow log 而影响Redis 的速度
slowlog-log-slower-than 10000	#以微秒为单位的慢日志记录，为负数会禁用慢日志，为0会记录每个命令操作。默认值为10ms,一般一条命令执行都在微秒级,生产建议设为1ms-10ms之间
slowlog-max-len 128				#最多记录多少条慢日志的保存队列长度，达到此长度后，记录新命令会将最旧的命令从命令队列中删除，以此滚动删除,即,先进先出,队列固定长度,默认128,值偏小,生产建议设为1000以上
latency-monitor-threshold 0		#作用：控制 Redis 延迟监控的阈值，记录操作延迟超过指定毫秒数的事件。取值：0（默认）：禁用延迟监控。正整数：启用延迟监控，并记录延迟超过此值（单位：毫秒）的事件。适用场景：如果需要分析 Redis 性能问题，可以设置为非零值（例如 100），以监控高延迟操作。
notify-keyspace-events ""		#作用：配置 Redis 的键空间通知功能，用于向客户端发布键相关的事件。取值：空字符串（默认）：禁用键空间知。字符串：指定监听的事件类型，如：K：键空间事件。E：键事件。x：过期事件。g：通用命令。示例：notify-keyspace-events "Ex"
hash-max-listpack-entries 512	#作用：设置 Redis 哈希对象（hash）的最大 listpack 条目数。如果超过该值，哈希对象会从紧凑的 listpack 转为普通哈希表。默认值：512 条目。
hash-max-listpack-value 64		#作用：设置哈希对象中单个字段值的最大字节数。超过该值时，哈希对象会从 listpack 转为普通哈希表。默认值：64 字节。
list-max-listpack-size -2		#作用：控制列表对象的最大紧凑表示（listpack）大小。取值：负值：以 2 的绝对值次幂为基础的限制，如 -2 表示 2^2。正值：以字节为单位的绝对大小。
list-compress-depth 0			#作用：控制列表对象压缩的深度。列表两端保留未压缩的节点数，其余部分压缩存储。默认值：0（禁用压缩）。
set-max-intset-entries 512		#作用：设置整数集合（intset）转换为普通集合（hashtable）的最大条目数。默认值：512 条目。
set-max-listpack-entries 128	#作用：类似 hash-max-listpack-* 配置项，但适用于集合（set）数据结构。
set-max-listpack-value 64		#作用：类似 hash-max-listpack-* 配置项，但适用于集合（set）数据结构。
zset-max-listpack-entries 128	#作用：类似 hash-max-listpack-* 配置项，但适用于有序集合（zset）。
zset-max-listpack-value 64		#作用：类似 hash-max-listpack-* 配置项，但适用于有序集合（zset）。
hll-sparse-max-bytes 3000		#作用：设置 HyperLogLog 数据结构的稀疏表示最大字节数。默认值：3000 字节。
stream-node-max-bytes 4096		#作用：设置 Stream 数据结构中单个节点的最大字节数。
stream-node-max-entries 100		#作用：设置 Stream 数据结构中单个节点的最大条目数。
activerehashing yes				#作用：启用 Redis 的主动 rehash 功能，以避免哈希表扩容和收缩时阻塞。默认值：yes
client-output-buffer-limit normal 0 0 0					
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60		#作用：设置不同类型客户端的输出缓冲区限制。参数：、
													#第一组：正常客户端，默认 0 0 0（无限制）。
													#第二组：从节点，默认 256mb 64mb 60。
													#第三组：发布/订阅客户端，默认 32mb 8mb 60。
hz 10							#作用：控制 Redis 的运行循环频率（每秒运行多少次任务）。默认值：10
dynamic-hz yes					#作用：启用动态调整 hz 值的功能，以平衡性能和延迟。默认值：yes
aof-rewrite-incremental-fsync yes	#作用：启用增量 fsync 机制，在 AOF 重写时分步同步文件，减少 I/O 峰值。
rdb-save-incremental-fsync yes		#作用：启用增量 fsync 机制，在 RDB 保存时分步同步文件。
jemalloc-bg-thread yes				#作用：启用 jemalloc 的后台线程，用于异步清理内存碎片。默认值：yes
```

## config 命令实现动态修改配置（热加载）

config 命令用于查看当前redis配置、以及不重启redis服务实现动态更改redis配置等

**注意：不是所有配置都可以动态修改,且此方式无法持久保存**

```powershell
CONFIG SET parameter value
时间复杂度：O(1)
CONFIG SET 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启。

可以使用它修改配置参数，或者改变 Redis 的持久化(Persistence)方式。
CONFIG SET 可以修改的配置参数可以使用命令 CONFIG GET * 来列出，所有被 CONFIG SET 修改的配置参数都会立即生效。

CONFIG GET parameter
时间复杂度： O(N)，其中 N 为命令返回的配置选项数量。
CONFIG GET 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)，在
Redis 2.4 版本中， 有部分参数没有办法用 CONFIG GET 访问，但是在最新的 Redis 2.6 版本中，所有配置参数都已经可以用 CONFIG GET 访问了。

CONFIG GET 接受单个参数 parameter 作为搜索关键字，查找所有匹配的配置参数，其中参数和值以“键-值对”(key-value pairs)的方式排列。
比如执行 CONFIG GET s* 命令，服务器就会返回所有以 s 开头的配置参数及参数的值：
```

范例：版本差异

```powershell
#redis-7支持动态修改端口
127.0.0.1:6379> config set port 8888
OK

#redis-7 不支持动态修改日志文件路径
127.0.0.1:6379> config set logfile /tmp/redis.log
(error) ERR CONFIG SET failed (possibly related to argument 'logfile') - can't 
set immutable config

#redis-5不支持动态修改端口
127.0.0.1:6379> config set port 8888
(error) ERR Unsupported CONFIG parameter: port
```

### 设置客户端连接密码

```powershell
#设置连接密码
127.0.0.1:6379> CONFIG SET requirepass 123456
OK
#查看连接密码
127.0.0.1:6379> CONFIG GET requirepass  
1) "requirepass"
2) "123456"
```

### 获取当前配置

```powershell
#奇数行为键，偶数行为值
127.0.0.1:6379> CONFIG GET *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) ""
  5) "masterauth"
  6) ""
  7) "cluster-announce-ip"
  8) ""
  9) "unixsocket"
10) ""
11) "logfile"
12) "/var/log/redis/redis.log"
13) "pidfile"
14) "/var/run/redis_6379.pid"
15) "slave-announce-ip"
16) ""
17) "replica-announce-ip"
18) ""
19) "maxmemory"
20) "0"

#查看bind 
127.0.0.1:6379> CONFIG GET bind
1) "bind"
2) "0.0.0.0"

#Redis5.0有些设置无法修改,Redis6.2.6版本支持修改bind
127.0.0.1:6379> CONFIG SET bind 127.0.0.1
(error) ERR Unsupported CONFIG parameter: bind
```

### 设置 Redis 使用的最大内存量

```powershell
127.0.0.1:6379> CONFIG SET maxmemory 8589934592 或 1g|G # 默认以字节为单位,1G以10的n次
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "8589934592"
```

### 慢查询

<img src="5day-png\27慢查询.png" alt="image-20250119162346239" style="zoom: 33%;" />

范例: SLOW LOG

```powershell
[root@ubuntu2404 ~]#vim /apps/redis/etc/redis.conf
slowlog-log-slower-than 1    #单位为us，指定超过1us即为慢的指令，默认值为10000us，10ms
slowlog-max-len 1024         #指定只保存最近的1024条慢记录，默认值为128
127.0.0.1:6379> SLOWLOG LEN  #查看慢日志的记录条数
(integer) 14
127.0.0.1:6379> SLOWLOG GET [n] #查看慢日志的最近n条记录，默认为10
1) 1) (integer) 14
2) (integer) 1544690617       #第2）行表示命令执行的时间戳，距离1970-1-1的秒数，date -d 
+@1544690617 可以转换
3) (integer) 4                #第3)行表示每条指令的执行时长
4) 1) "slowlog"
127.0.0.1:6379> SLOWLOG GET 3
1) 1) (integer) 7
   2) (integer) 1602901545
   3) (integer) 26
   4) 1) "SLOWLOG"
      2) "get"
   5) "127.0.0.1:38258"
   6) ""
2) 1) (integer) 6
   2) (integer) 1602901540
   3) (integer) 22
   4) 1) "SLOWLOG"
      2) "get"
      3) "2"
   5) "127.0.0.1:38258"
   6) ""
3) 1) (integer) 5
   2) (integer) 1602901497
   3) (integer) 22
   4) 1) "SLOWLOG"
      2) "GET"
   5) "127.0.0.1:38258"
   6) ""
127.0.0.1:6379> SLOWLOG RESET #清空慢日志
OK
```

## Redis 持久化

Redis 是基于内存型的NoSQL, 和MySQL是不同的,使用内存进行数据保存

如果想实现数据的持久化,Redis也也可支持将内存数据保存到硬盘文件中 

Redis支持两种数据持久化保存方法 

- RDB:Redis DataBase 

- AOF:AppendOnlyFile

<img src="5day-png\27redis持久化" alt="image-20250119164007390" style="zoom:33%;" />

### RDB

<img src="5day-png\27RDB工作原理.png" alt="image-20250119165113016" style="zoom:33%;" />

RDB(Redis DataBase)：是基于某个时间点的快照，注意RDB只保留当前最新版本的一个快照 

相当于MySQL中的完全备份

RDB 持久化功能所生成的 RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成该 RDB 文 件时数据库的状态。因为 RDB 文件是保存在磁盘中的，所以即便 Redis 服务进程甚至服务器宕机，只要 磁盘中 RDB 文件存在，就能将数据恢复

**RDB 支持save和bgsave两种命令实现数据文件的持久化**

注意： save 指令使用主进程进行备份，而不生成新的子进程，但是也会生成临时文件temp-<主进程 PID>.rdb文件

**范例： save 执行过程会使用主进程进行快照，并生成临时文件temp-<主进程PID>.rdb文件**

```powershell
#生成临时文件temp-<主进程PID>.rdb文件
[root@centos7 data]#redis-cli -a 123456 save&
[1] 28684
[root@centos7 data]#pstree -p |grep redis ;ll /apps/redis/data
           |-redis-server(28650)-+-{redis-server}(28651)
           |                     |-{redis-server}(28652)
           |                     |-{redis-server}(28653)
           |                     `-{redis-server}(28654)
           |           |                         `-redis-cli(28684)
           |           `-sshd(23494)---bash(23496)---redis-cli(28601)
total 251016
-rw-r--r-- 1 redis redis 189855682 Nov 17 15:02 dump.rdb
-rw-r--r-- 1 redis redis  45674498 Nov 17 15:02 temp-28650.rdb
```

#### **RDB bgsave 实现快照的具体过程:**

<img src="5day-png\27RDB的bgsave.png" alt="image-20250119171202446" style="zoom: 33%;" />

首先从redis 主进程先fork生成一个新的子进程,此子进程负责将Redis内存数据保存为一个临时文件tmp- <子进程pid>.rdb 

当数据保存完成后,再将此临时文件改名为RDB文件,如果有前一次保存的RDB文件则会被替换，最后关闭 此子进程 

由于Redis只保留最后一个版本的RDB文件,如果想实现保存多个版本的数据,需要人为实现

```powershell
查看bgsave是否备份成功
rdb_bgsave_in_progress:0
0：备份成功
1：正在备份
```

#### **RDB相关配置**

```powershell
#在配置文件中的 save 选项设置多个保存条件，只有任何一个条件满足，服务器都会自动执行 BGSAVE 命令
#Redis7.0以后支持写在一行，如：save 3600 1 300 100 60 10000，此也为默认值
save 900 1         #900s内修改了1个key即触发保存RDB
save 300 10        #300s内修改了10个key即触发保存RDB
save 60 10000      #60s内修改了10000个key即触发保存RDB

dbfilename dump.rdb
dir ./             #编泽编译安装时默认RDB文件存放在Redis的工作目录,此配置可指定保存的数据目录
stop-writes-on-bgsave-error yes  #当快照失败是否仍允许写入,yes为出错后禁止写入,建议为no
rdbcompression yes
rdbchecksum yes
```

```powershell
[root@ubuntu2004 ~]#grep save /apps/redis/etc/redis.conf
# save <seconds> <changes>
# Redis will save the DB if both the given number of seconds and the given
# save ""
# Unless specified otherwise, by default Redis will save the DB:
# save 3600 1
# save 300 100
# save 60 10000
#以上是默认值
[root@ubuntu2004 ~]#redis-cli config get save
1) "save"
2) "3600 1 300 100 60 10000"
#表示有三个自动保存条件，它们的含义如下：
#3600 1：如果在 3600 秒（即 1 小时）内至少发生 1 次写操作，则触发一次数据快照保存。
#300 100：如果在 300 秒（即 5 分钟）内至少发生 100 次写操作，则触发一次数据快照保存。
#60 10000：如果在 60 秒（即 1 分钟）内至少发生 10000 次写操作，则触发一次数据快照保存。

#禁用系统的自动快照
[root@ubuntu2004 ~]#vim /apps/redis/etc/redis.conf
save ""			#不备份
# save 3600 1
# save 300 100
# save 60 10000
#支持动态修改，注意：需要添加双引号
127.0.0.1:6379> config set save "60 3"
OK
127.0.0.1:6379> config get save
1) "save"
2) "60 3"
```

#### **实现 RDB 方法**

- save: 同步,不推荐使用，使用主进程完成快照，因此会阻塞其它命令执行 

- bgsave: 异步后台执行,不影响其它命令的执行，会开启独立的子进程，因此不会阻赛其它命令执行 

- 配置文件实现自动保存: 在配置文件中制定规则,自动执行bgsave

`SAVE` 和 `BGSAVE` 是 Redis 中用于触发 **RDB (Redis Database)** 持久化操作的命令。它们之间的主要区别在于它们触发持久化的方式以及对 Redis 性能的影响。

#### 1. **`SAVE`**

- **作用**：
  `SAVE` 命令会阻塞当前 Redis 进程，并立即执行 **同步** RDB 保存操作。

- **执行过程**：

  - 当执行 `SAVE` 命令时，Redis 会创建一个新的 RDB 快照文件（`dump.rdb`）。
  - 在此过程中，Redis 会暂停处理所有的其他命令，直到 RDB 文件写入磁盘完成为止。
  - 这个操作是 **同步的**，因此 Redis 会在保存过程中无法响应其他请求。

- **影响**：

  - **阻塞**：由于 Redis 会在保存过程中阻塞，可能会影响系统的响应时间，尤其是在数据量较大的时候。
  - **适用场景**：适合在一些不太关心性能的特殊场景中使用，例如对数据持久化有较高要求且可以容忍短暂停顿的操作。

- **使用示例**：

  ```
  SAVE
  ```

#### 2. **`BGSAVE`**

- **作用**：
  `BGSAVE` 命令会触发 **异步** RDB 保存操作，不会阻塞 Redis 进程。

- **执行过程**：

  - 当执行 `BGSAVE` 命令时，Redis 会通过 **fork** 出一个子进程来进行 RDB 文件的保存操作。
  - 主进程继续处理其他命令，子进程会负责将数据持久化到磁盘中。
  - 子进程完成保存后，生成的 RDB 文件将会替代旧的 RDB 文件。

- **影响**：

  - **非阻塞**：不会影响主进程的执行，因此 Redis 可以继续响应客户端的请求。
  - **性能优化**：使用 `BGSAVE` 时，持久化操作在后台进行，减少了对客户端请求的阻塞影响，因此更适合高性能的生产环境。
  - 由于是通过 `fork` 创建子进程，子进程和主进程是独立运行的，只有在写入磁盘时，主进程才会受到一定影响（例如磁盘 I/O 占用）。

- **使用示例**：

  ```
  BGSAVE
  ```

#### **区别总结**：

| 特性         | `SAVE`                       | `BGSAVE`                         |
| ------------ | ---------------------------- | -------------------------------- |
| **阻塞**     | 是，Redis 会暂停响应其他命令 | 否，Redis 继续处理其他请求       |
| **执行方式** | 同步操作，直到完成才返回     | 异步操作，fork 子进程执行持久化  |
| **性能影响** | 高，可能导致性能瓶颈         | 低，适合高性能生产环境           |
| **适用场景** | 对停顿没有要求的情况         | 高并发环境，要求低延迟的生产场景 |

#### **建议**：

- **生产环境**：推荐使用 `BGSAVE`，因为它能避免阻塞 Redis 的主进程，对系统性能影响较小。
- **开发/测试环境**：如果希望确保数据持久化立即完成，且能容忍停顿，可以使用 `SAVE`。
- **备份方案**：如果需要频繁地进行备份操作，使用 `BGSAVE` 会更为合适，尤其是在数据量较大的情况下。

#### **注意事项**：

- 在执行 `BGSAVE` 时，Redis 会 **fork** 出一个子进程，若系统内存较小或数据量过大，频繁使用 `BGSAVE` 可能会导致高内存消耗。
- Redis 会在执行 `BGSAVE` 或 `SAVE` 后，检查当前数据库的持久化状态，并根据配置文件中的规则判断是否重新执行持久化操作。

#### RDB 模式的优缺点

**RDB缺点**

- RDB快照只保存某个时间点的数据，恢复的时候直接加载到内存即可，不用做其他处理，这种文件 适合用于做灾备处理.可以通过自定义时间点执行redis指令bgsave或者save保存快照，实现多个版 本的备份 

- 比如: 可以在最近的24小时内，每小时备份一次RDB文件，并且在每个月的每一天，也备份一个 RDB文件。这样的话，即使遇上问题，也可以随时将数据集还原到指定的不同的版本。 

- RDB在大数据集时恢复的速度比AOF方式要快

**RDB优点**

- 不能实时保存数据，可能会丢失自上一次执行RDB备份到当前的内存数据 如果需要尽量避免在服务器故障时丢失数据，那么RDB并不适合。虽然Redis允许设置不同的保存 点（save point）来控制保存RDB文件的频率，但是，因为RDB文件需要保存整个数据集的状态， 所以它可能并不是一个非常快速的操作。因此一般会超过5分钟以上才保存一次RDB文件。在这种 情况下，一旦发生故障停机，就可能会丢失较长时间的数据。 

- 在数据集比较庞大时，fork()子进程可能会非常耗时，造成服务器在一定时间内停止处理客户端请 求,如果数据集非常巨大，并且CPU时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒 或更久。另外子进程完成生成RDB文件的时间也会花更长时间.

#### 手动备份RDB文件的脚本

```powershell
#配置文件
[root@centos7 ~]#vim /apps/redis/etc/redis.conf
save ""
dbfilename dump_6379.rdb
dir "/data/redis"
appendonly no 
#脚本
[root@centos8 ~]#cat redis_backup_rdb.sh 
#!/bin/bash
#

BACKUP=/backup/redis-rdb
DIR=/data/redis
FILE=dump_6379.rdb
PASS=123456
color () { 
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"   # 设置光标的位置
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"    # 设置颜色为绿色（成功）
    SETCOLOR_FAILURE="echo -en \\033[1;31m"    # 设置颜色为红色（失败）
    SETCOLOR_WARNING="echo -en \\033[1;33m"    # 设置颜色为黄色（警告）
    SETCOLOR_NORMAL="echo -en \E[0m"           # 设置颜色恢复为默认

    echo -n "$1" && $MOVE_TO_COL
    echo -n "["

    if [ $2 = "success" -o $2 = "0" ] ;then   # 成功时显示 OK
        ${SETCOLOR_SUCCESS}
        echo -n $" OK "    
    elif [ $2 = "failure" -o $2 = "1" ] ;then # 失败时显示 FAILED
        ${SETCOLOR_FAILURE}
        echo -n $"FAILED"
    else
        ${SETCOLOR_WARNING}                # 如果是警告时显示 WARNING
        echo -n $"WARNING"
    fi
    ${SETCOLOR_NORMAL}
    echo -n "]"
    echo
}
redis-cli -h 127.0.0.1 -a $PASS --no-auth-warning bgsave 
result=`redis-cli -a $PASS --no-auth-warning info Persistence |grep rdb_bgsave_in_progress| sed -rn 's/.*:([0-9]+).*/\1/p'`
until [ $result -eq 0 ] ;do
    sleep 1
    result=`redis-cli -a $PASS --no-auth-warning info Persistence |awk -F: '/rdb_bgsave_in_progress/{print $2}'`
done
DATE=`date +%F_%H-%M-%S`
[ -e $BACKUP ] || { mkdir -p $BACKUP ; chown -R redis.redis $BACKUP; }
scp $DIR/$FILE $BACKUP/dump_6379-${DATE}.rdb backup-server:/backup/

color "Backup redis RDB" 0
```



### AOF

####  AOF 工作原理

<img src="5day-png\27AOF工作原理.png" alt="image-20250119173239708" style="zoom:33%;" />

AOF 即 AppendOnlyFile，AOF 和 RDB 都采有COW机制 

AOF 可以指定不同的保存策略,默认为每秒钟执行一次 fsync,按照操作的顺序地将变更命令追加至指定的 AOF日志文件尾部 

在第一次启用AOF功能时，会做一次完全备份，后续将执行增量性备份，相当于完全数据备份+增量变化 

如果同时启用RDB和AOF,进行恢复时,默认AOF文件优先级高于RDB文件,即会使用AOF文件进行恢复 

在第一次开启AOF功能时,会自动备份所有数据到AOF文件中,后续只会记录数据的更新指令 

**注意: AOF 模式默认是关闭的,第一次开启AOF后,并重启服务生效后,会因为AOF的优先级高于RDB,而 AOF默认没有数据文件存在,从而导致所有数据丢失**

范例: 错误开启AOF功能,会导致数据丢失

```powershell
[root@ubuntu1804 ~]#redis-cli 
127.0.0.1:6379> dbsize
(integer) 10010011

[root@ubuntu1804 ~]#vim /apps/redis/etc/redis.conf
appendonly yes #修改此行

[root@ubuntu1804 data]#systemctl restart redis
[root@ubuntu1804 ~]#redis-cli 
127.0.0.1:6379> dbsize
(integer) 0
```

范例：正确启用AOF功能,访止数据丢失

```powershell
[root@centos8 ~]#ll /var/lib/redis/
total 314392
-rw-r--r-- 1 redis redis 187779391 Oct 17 14:23 dump.rdb
[root@centos8 ~]#redis-cli
127.0.0.1:6379> config get appendonly 
1) "appendonly"
2) "no"
127.0.0.1:6379> config set appendonly  yes  #自动触发AOF重写,会自动备份所有数据到AOF文
件
OK
[root@centos8 ~]#ll /var/lib/redis/
total 314392
-rw-r--r-- 1 redis redis 187779391 Oct 17 14:23 dump.rdb
-rw-r--r-- 1 redis redis  85196805 Oct 17 14:45 temp-rewriteaof-2146.aof
[root@centos8 ~]#ll /var/lib/redis/
total 366760
-rw-r--r-- 1 redis redis 187779391 Oct 17 14:45 appendonly.aof
-rw-r--r-- 1 redis redis 187779391 Oct 17 14:23 dump.rdb
[root@centos8 ~]#vim /etc/redis.conf
appendonly yes #改为yes 
#config set appendonly yes 后可以同时看到下面显示
```

```powershell
127.0.0.1:6379> config get appendonly
1) "appendonly"
2) "no"
127.0.0.1:6379> config set appendonly  yes
OK
[root@ubuntu2404 ~]#ll /apps/redis/data/
total 16
drwxr-xr-x 2 redis redis 4096 Jan 19 17:36 appendonlydir/
-rw-r--r-- 1 redis redis   88 Jan 19 16:55 dump.rdb
[root@ubuntu2404 ~]#vim /apps/redis/etc/redis.conf 
appendonly yes
[root@ubuntu2404 ~]#ll /apps/redis/data/appendonlydir/
total 16
drwxr-xr-x 2 redis redis 4096 Jan 19 17:36 ./
drwxr-xr-x 3 redis redis 4096 Jan 19 17:36 ../
-rw-r--r-- 1 redis redis   88 Jan 19 17:36 appendonly.aof.1.base.rdb
-rw-r--r-- 1 redis redis    0 Jan 19 17:36 appendonly.aof.1.incr.aof
-rw-r--r-- 1 redis redis   88 Jan 19 17:36 appendonly.aof.manifest
```

```powershell
# Redis 7.0以上版本
[root@ubuntu2204 ~]#file /apps/redis/data/appendonlydir/*
/apps/redis/data/appendonlydir/appendonly.aof.1.base.rdb: Redis RDB file, version 
0010
/apps/redis/data/appendonlydir/appendonly.aof.1.incr.aof: ASCII text, with CRLF 
line terminators
/apps/redis/data/appendonlydir/appendonly.aof.manifest:   ASCII text
#Redis6.0以前版本只有一个文件
[root@ubuntu2204 ~]#file /var/lib/redis/appendonly.aof
/var/lib/redis/appendonly.aof: Redis RDB file, version 0009
```

#### AOF 相关配置

```powershell
appendonly no 						#是否开启AOF日志记录，默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了，但是redis如果中途宕机，会导致可能有几分钟的数据丢失(取决于dump数据的间隔时间)，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性，Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。默认不启用此功能
appendfilename "appendonly.aof"  	#文本文件AOF的文件名，存放在dir指令指定的目录中
appenddirname "appendonlydir"    	#7.X 版指定目录名称
appendfsync everysec        		#aof持久化策略的配置
									#no表示由操作系统保证数据同步到磁盘,Linux的默认fsync策略是30秒，最多会丢失30s的数据
									#always表示每次写入都执行fsync，以保证数据同步到磁盘,安全性高,性能较差
									#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据,此为默认值,也生产建议值
dir /path
#rewrite相关
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes

```

#### AOF Rewrite 重写(清理)

将一些重复的,可以合并的,过期的数据重新写入一个新的AOF文件,从而节约AOF备份占用的硬盘空间,也 能加速恢复过程 

可以手动执行bgrewriteaof 触发AOF,第一次开启AOF功能,或定义自动rewrite 策略

**AOF rewrite 过程** 

父进程生成一个新的子进程负责生成新的AOF文件，同时父进程将新的数据更新同时写入两个缓冲区 aof_buf和aof_rewrite_buf 

6.X版本之前新的AOF文件覆盖旧的AOF文件 

7.X版本之后版本，新的AOF文件覆盖AOF目录中的RDB文件appendonly.aof.2.base.rdb，并生成一个 新空的AOF文件appendonly.aof.2.incr.aof，此文件的编号会加1，同时更新appendonly.aof.manifest 中的内容

<img src="5day-png\27AOF重写.png" alt="image-20250119180401950" style="zoom: 33%;" />

AOF rewrite 重写相关配置

```powershell
#同时在执行bgrewriteaof操作和主进程写aof文件的操作，两者都会操作磁盘，而bgrewriteaof往往会涉及大量磁盘操作，这样就会造成主进程在写aof文件的时候出现阻塞的情形,以下参数实现控制
no-appendfsync-on-rewrite no 	#在aof rewrite期间,是否对aof新记录的append暂缓使用文件同步策略,主要考虑磁盘IO开支和请求阻塞时间。
								#默认为no,表示"不暂缓",新的aof记录仍然会被立即同步到磁盘，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题
								#为yes,相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？Linux的默认fsync策略是30秒，最多会丢失30s的数据,但由于yes性能较好而且会避免出现阻塞因此比较推荐
								#rewrite 即对aof文件进行整理,将空闲空间回收,从而可以减少恢复数据时间
auto-aof-rewrite-percentage 100 #当Aof log增长超过指定百分比例时，重写AOF文件，设置为0表示不自动重写Aof日志，重写是为了使aof体积保持最小，但是还可以确保保存最完整的数据
auto-aof-rewrite-min-size 64mb 	#触发aof rewrite的最小文件大小
aof-load-truncated yes 			#是否加载由于某些原因导致的末尾异常的AOF文件(主进程被kill/断电等)，建议yes
```

auto-aof-rewrite-percentage 100

<img src="5day-png\27AOF重写日志整理.png" alt="image-20250119181523271" style="zoom: 67%;" />



```powershell
动态修改配置自动生成appendonly.aof文件
127.0.0.1:6379> CONFIG set appendonly yes
```

#### 手动执行AOF重写 BGREWRITEAOF 命令

```
BGREWRITEAOF
时间复杂度： O(N)， N 为要追加到 AOF 文件中的数据数量。
执行一个 AOF文件 重写操作。重写会创建一个当前 AOF 文件的体积优化版本。

即使 BGREWRITEAOF 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 BGREWRITEAOF 成功之前不会被修改。

重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：
如果 Redis 的子进程正在执行快照的保存工作，那么 AOF 重写的操作会被预定(scheduled)，等到保存工作完成之后再执行 AOF 重写。在这种情况下， BGREWRITEAOF 的返回值仍然是 OK ，但还会加上一条额外的信息，说明 BGREWRITEAOF 要等到保存操作完成之后才能执行。在 Redis 2.6 或以上的版本，可以使用 INFO [section] 命令查看 BGREWRITEAOF 是否被预定。

如果已经有别的 AOF 文件重写在执行，那么 BGREWRITEAOF 返回一个错误，并且这个新的
BGREWRITEAOF 请求也不会被预定到下次执行。

从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， BGREWRITEAOF 仅仅用于手动触发重写操作。
```

范例: 手动 bgrewriteaof

```powershell
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
#7.X版本
[root@ubuntu2204 etc]#redis-cli -a 123456 BGREWRITEAOF ; pstree -p |grep 
redis; ls -l /apps/redis/data/ls -l /apps/redis/data/appendonlydir/
           ├─redis-server(41173)─┬─redis-server(41192)
           │                     ├─{redis-server}(41174)
           │                     ├─{redis-server}(41175)
           │                     ├─{redis-server}(41176)
           │                     ├─{redis-server}(41177)
           │                     ├─{redis-server}(41183)
           │                     └─{redis-server}(41194)
-rw-r--r-- 1 redis redis 189855694  6月 26 16:53 appendonly.aof.2.base.rdb
-rw-r--r-- 1 redis redis       267  6月 26 17:04 appendonly.aof.2.incr.aof
-rw-r--r-- 1 redis redis         0  6月 26 17:10 appendonly.aof.3.incr.aof
-rw-r--r-- 1 redis redis       132  6月 26 17:10 appendonly.aof.manifest

#7.X生成一个临时的AOF文件
[root@ubuntu2204 ~]#ll /apps/redis/data/
总计 83688
drwxr-xr-x 3 redis redis     4096  2月 19 09:41 ./
drwxr-xr-x 7 redis redis     4096  2月  2 11:42 ../
drwxr-xr-x 2 redis redis     4096  2月 19 09:41 appendonlydir/
-rw-r--r-- 1 redis redis       88  2月 19 09:24 dump.rdb
-rw-r--r-- 1 redis redis 85680128  2月 19 09:41 temp-rewriteaof-96239.aof

#执行完成后incr文件清空，合并到RDB文件中
[root@ubuntu2204 etc]#ll /apps/redis/data/appendonlydir/
总计 185420
drwxr-xr-x 2 redis redis      4096  6月 26 17:10 ./
drwxr-xr-x 3 redis redis      4096  6月 26 17:10 ../
-rw-r--r-- 1 redis redis 189855711  6月 26 17:10 appendonly.aof.3.base.rdb
-rw-r--r-- 1 redis redis         0  6月 26 17:10 appendonly.aof.3.incr.aof
-rw-r--r-- 1 redis redis        88  6月 26 17:10 appendonly.aof.manifest
```

#### AOF 模式优缺点

**AOF 模式优点**

- 数据安全性相对较高，根据所使用的fsync策略(fsync是同步内存中redis所有已经修改的文件到存 储设备)，默认是appendfsync everysec，即每秒执行一次 fsync,在这种配置下，Redis 仍然可以保 持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据( fsync会在后台线程执行， 所以主线程可以继续努力地处理命令请求) 
- 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中不需要seek, 即使出现 宕机现象，也不会破坏日志文件中已经存在的内容。然而如果本次操作只是写入了一半数据就出现 了系统崩溃问题，不用担心，在Redis下一次启动之前，可以通过 redis-check-aof 工具来解决数据 一致性的问题 
- Redis可以在 AOF文件体积变得过大时，自动地在后台对AOF进行重写,重写后的新AOF文件包含了 恢复当前数据集所需的最小命令集合。整个重写操作是绝对安全的，因为Redis在创建新 AOF文件 的过程中，append模式不断的将修改数据追加到现有的 AOF文件里面，即使重写过程中发生停 机，现有的 AOF文件也不会丢失。而一旦新AOF文件创建完毕，Redis就会从旧AOF文件切换到新 
- AOF文件，并开始对新AOF文件进行追加操作。 AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，也可以通过该文 件完成数据的重建
- AOF文件有序地保存了对数据库执行的所有写入操作，这些写入操作以Redis协议的格式保存，因 此 AOF文件的内容非常容易被人读懂，对文件进行分析(parse)也很轻松。导出（export)AOF文件 也非常简单:举个例子，如果不小心执行了FLUSHALL.命令，但只要AOF文件未被重写，那么只要停 止服务器，移除 AOF文件末尾的FLUSHAL命令，并重启Redis ,就可以将数据集恢复到FLUSHALL执 行之前的状态。

**AOF 模式缺点**

- 即使有些操作是重复的也会全部记录，AOF 的文件大小一般要大于 

- RDB 格式的文件 AOF 在恢复大数据集时的速度比 RDB 的恢复速度要慢 

- 如果 fsync 策略是appendfsync no, AOF保存到磁盘的速度甚至会可能会慢于RDB 

- bug 出现的可能性更多

Redis 的 AOF（Append Only File）模式是一种持久化机制，记录所有对数据库进行写操作的命令。以下是 AOF 模式的优点和缺点：

------

#### **优点**

1. **更高的数据安全性**
   - AOF 通过记录每一条写操作命令，可以设置更短的同步周期（如 `appendfsync always` 或 `appendfsync everysec`），从而在宕机时最大限度地减少数据丢失。
   - 相较于 RDB，AOF 通常可以提供更接近实时的数据持久化。
2. **更灵活的同步策略**
   - AOF 提供了三种数据同步策略（`always`、`everysec` 和 `no`），允许用户根据需求在性能和安全性之间做出权衡。
   - 一般来说，`everysec` 是推荐选项，能够在性能和数据安全之间达到良好的平衡。
3. **日志文件可读性**
   - AOF 文件是一个纯文本文件，存储的是 Redis 执行的写命令日志，因此可读性高。管理员可以方便地通过手动编辑 AOF 文件修复数据库问题。
4. **支持更精细的数据恢复**
   - 如果数据库因意外原因损坏，可以通过回放 AOF 文件中记录的命令，逐条重现操作过程来恢复数据。
5. **不受全量保存限制**
   - RDB 模式需要周期性地保存整个数据库，而 AOF 只记录写操作日志，尤其适用于数据库体积较大、但更新较少的场景。

------

#### **缺点**

1. **文件体积更大**
   - 与 RDB 相比，AOF 文件通常更大，因为它记录了每一个写操作，而 RDB 只保存快照数据。
   - 对于高频写入的场景，AOF 文件增长速度会非常快。
2. **恢复速度较慢**
   - 在 Redis 重启时，AOF 需要回放所有写操作日志来重建数据，速度比直接加载 RDB 文件慢很多，尤其是当 AOF 文件非常大时。
3. **性能开销更高**
   - AOF 需要频繁地将写命令追加到日志文件中，这会导致更多的磁盘 I/O 开销，尤其是在使用 `appendfsync always` 的同步策略时，性能可能会显著下降。
4. **风险：磁盘写满**
   - 在极端情况下，频繁写操作可能会导致 AOF 文件增长过快而填满磁盘，进而影响 Redis 的运行。
5. **重写操作复杂性**
   - AOF 文件的大小会随着时间逐渐增长，为了避免文件过大，Redis 会定期触发 AOF 重写（`BGREWRITEAOF`）。重写会 fork 出子进程进行整理操作，但这可能会导致额外的 CPU 和内存占用。
6. **不适合只读场景**
   - 如果 Redis 主要用作只读缓存，则 AOF 提供的日志记录优势显得无用，而其额外的磁盘开销和性能开销可能不值得。

------

#### **适用场景**

- **AOF 适用场景**：
  - 数据一致性要求高（如金融系统、用户订单等）。
  - 容忍较高的磁盘 I/O 和启动时间开销。
  - 需要更灵活的备份和恢复策略。
- **AOF 不适用场景**：
  - 数据可丢失的缓存场景。
  - 高写入频率且不需要实时数据持久化的场景。
  - 对性能和响应时间有极高要求的场景。

------

#### **总结**

| 特性               | AOF 模式                   | RDB 模式                 |
| ------------------ | -------------------------- | ------------------------ |
| 数据安全性         | 较高，丢失数据较少         | 可能丢失较多数据         |
| 持久化频率         | 更频繁，记录每个写操作     | 定期，按快照时间间隔保存 |
| 文件体积           | 较大                       | 较小                     |
| 恢复速度           | 较慢                       | 较快                     |
| 性能开销           | 高（尤其是同步策略严格时） | 较低                     |
| 适合高性能缓存场景 | 一般                       | 较好                     |

如果数据安全性是首要目标，建议启用 AOF，并设置合适的同步策略；如果对性能和内存开销要求更高，可以仅使用 RDB，或者结合两者以取长补短。 















20250316
