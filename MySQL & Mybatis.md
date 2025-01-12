## Mybatis

### sqlSession的实现有什么？有什么区别？

有两个实现：`DefaultSqlSession` & `SqlSessionManager`

前者是mybatis的默认实现，默认直接与Mybatis的核心逻辑交互

后者是更加高级，封装化的实现。增强对`SqlSession`的管理，提供了更好的对线程安全性和事务功能



**线程管理**：

- **`DefaultSqlSession`**：不具备**线程安全**特性，直接使用时可能导致线程安全问题。开发者需要手动确保 `DefaultSqlSession` 在多线程环境中的安全使用。
- **`SqlSessionManager`**：内部使用 `ThreadLocal` 维护每个线程独立的 `SqlSession` 实例，天然支持多线程场景，保证每个线程使用自己的会话，不会互相干扰。

**事务管理**：

- **`DefaultSqlSession`**：需要手动管理事务的开启、提交和回滚。开发者必须在合适的地方调用 `commit()` 和 `rollback()`，并在使用完 `SqlSession` 后手动关闭。
- **`SqlSessionManager`**：提供了更加方便的事务管理功能，如 `startManagedSession()` 方法可以自动处理会话和事务的管理。事务的提交和回滚也更加直观，不需要手动传递 `SqlSession` 对象。





### mybatis中有什么使用了单例模式？

`ErrorContext` & `LogFactory`

- `ErrorContext`是用在线程范围内的单例，记录该线程的执行环境错误信息 **以线程为单位，使用ThreadLocal管理**

- `LogFactory`则是提供给整个Mybatis使用的日志工厂，用于获得针对项目配置好的日志对象。 **以进程为单位**





### Nginx是如何处理一个HTTP请求的？

**多进程 & 事件驱动机制** 注意是进程不是线程

**通过多进程架构管理请求，在工程内部使用事件驱动机制处理请求**

多进程机制：Nginx中存在**主进程**和**工作进程**二概念，主进程负责对工作进程的管理，不直接参与数据交互。工作线程负责对客户端的连接和交互。当Nginx启动时，主进程根据配置中的(`worker_processes`)生成固定数量的工作进程，一般设置为**与CPU核数相同**。在此之中，主线程主要负责**维护 & 监控**等工作，当有工作进程寄了，会发送一个信号`SIGCHLD`。

事件驱动机制：epoll等。当有事件发生的时候，程序即触发对应的程序。同时并不会阻塞在某个事件中（非阻塞IO）。

Nginx中每个工作线程是单线程的，从而避免了并发时所会出现的问题。

**问：与CPU相同数量的工作进程是如何处理高并发的流量的**

答：事件驱动、非阻塞I/O、多进程协作、单线程模型等机制实现。



### Nginx的一些特性

反向代理 L7负载均衡器

嵌入式Perl解释器

动态二进制升级

重新编写url 具非常好的PCRE（Perl的正则式）支持



### ThreadLocal的原理

底层是一个特殊的HashMap，即ThreadLocalMap。key为ThreadLocal类型变量，value为Object类型键值对。当ThreadLocal没有强引用的时候，垃圾回收时，会将这个map中的key回收。也就变成了一个`key为null的键值对`，而ThreadLocal每一次执行`set() get()`等都会判断有无这种空key。如果有，将会全部清理掉。

为什么用完一定需要调用remove()删除呢？

why？因为如果不清理掉。所有的这些值都会一直遗留。而ThreadLocal还是以线程为单位的。所以会造成很大的空间消费。最终内存泄漏。这种做法**就是巧妙的利用了gc的机制 当gc回收键的时候 代表这个threadlocal变量不会再被使用 并且交由Threadlocal回收对应的值 让GC来判断有没有键不需要使用可以回收的**



### MySQL中一行记录是怎么存储的？

![image-20240910170834719](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240910170834719.png)

MYSQL中行之间的存储通过链表进行连接

此部分记录多头信息，包括：

- delete_mask：标识数据的逻辑删除，若逻辑删除则设1，否则0
- next_record：类似链表的存储格式，存储的位置介于**记录头信息和真实数据**之间的位置。
- record_type：标识当前记录的类型。TODO 没明白

在头信息的右边是真实的字段值，左边则是`null值列表`与 **变长字段所占用的真实字节数**（当使用到变长字段时 需要先知道他真实的占用大小才能进行使用）

何为null值列表？数据库中的字段设置为NULL的话，就会创建对应的null值列表。用于记录这些值中，哪些字段的值是真的null，如果是null则设为1，非null设为0。并且在mysql中，是以字节为单位存储这些列的信息的。也就是说，若NULL字段少于8，会补齐前置0至8（单字节容量）。若超过了8个字段，则会使用两个字节进行存储。

那么这个NULL值列表是必须的吗？**否，如果所有字段都是NOT NULL，则没有这一值列表**

并且，需要注意到这个列是逆向存放的，理由稍候。

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/null%E5%80%BC%E5%88%97%E8%A1%A84.png)

同理，从上图可以看出需要同时存储变长字段字节数真实字节数。它使用1-2个字节表示对应变长字段的真实长度。（**0-255时1个字节 255-65535时两个字节**）同理他也不是必须存在的，如果不是变长字段，就不会有变长字段列表了。同时这个也是逆序存放。

所以到底为什么要逆序存放？

![image-20240910172905730](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240910172905730.png)

其实本质上就是读取的时候，由于是以一片数据为单位进行读取的。并且字段是顺序存放。这样子的操作就有更高的几率使他们在同一个窗口中被读取。

**Q：那是不是应该将变长字段放到数据库的前面方便读取？**

A：no，如果放在前面的话，如果变长字段数据更改的时候，修改了整个存储容量，可能会导致整个表此字段往后的字段都需要进行迁移。



### varchar(n) 中 n 最大取值为多少？

**MySQL规定除了TEXT、BLOBs这种大对象类型之外，其他所有的列（不包括隐藏列和记录头信息)占用的字节长度加起来不能超过65535个字节。**

很好记 2的16次方。但是这个65535指的是包含null值列表信息以及变长字段长度信息的。

```sql
CREATE TABLE test( 
`name` VARCHAR(65535) NULL
) ENGINE InnoDB DEFAULT CHARACTER SET ascii ROW FORMAT COMPACT;
```

如图，由于这个是变长字段，且大于255，并且是设置为NULL可空。所以上方的建表语句是不可行，需要2字节存储变量长度，1字节为NULL值列表。

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/65532.png)

多行数时也是同理

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/%E5%A4%9A%E5%AD%97%E6%AE%B5%E7%9A%84%E6%83%85%E5%86%B5.png)

**MYSQL中的NULL值并不会存放在真实数据部分 那么如果是NULL值 真实数据部分存储的是什么？**

什么都不存，数据库跳过此字段



### MySQL中行溢出是怎么做的？

如果遇见blob等类型的时候，数据一下子存不完。此时InnoDB存储引擎会将数据存放到**溢出页**当中。

Ciompact会存一部分数据到当前页中，将剩余的数据放在溢出页。通过20字节的存储指向溢出页的地址，据此地址找到剩余数据所在的页。

Compressed和Dynamic这两个行格式和Compact非常类似，主要的区别在于处理行溢出数据时有些区别。这两种格式采用完全的行谥出方式，记录的真实数据不会存储该列的一部分数据，只存储20个字节的指针来指向溢出页。而实际的数据都存储在溢出页中

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/%E8%A1%8C%E6%BA%A2%E5%87%BA.png)

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/%E8%A1%8C%E6%BA%A2%E5%87%BA2.png)

**如果溢出页也不够存怎么办？**

此时，在溢出页中，三种行格式除了存放数据，同时存放下一个存放页的指针。形式一个多存放页的链表。





### MySQL InnoDB 选择B+tree作为索引的数据结构

其实就是问B+的优点：

vs B树：

首先需要知道，在树的节点中，一般来说每个节点是由**空间上的大小限制**的。而B+树和B树最直观的区别就是，B+树的模式不在非叶子节点上存储数据，B树does。所以相同的情况下，同一个节点使用B+树可以促成农户更多的指针（分支），而相同情况下。更多分支意味着高度越小，高度越小则意味着磁盘IO的次数更小。从而提高了运行的效率。

同时，B+树的特点是有两个指针，一个指向根节点（root），一个指向最小的节点。分别赋给了B+树**范围查询和由小到大查询**的能力。B树明显无法做到这点。

vs 二叉树：

同理，对于二叉树。当数据的量级较大，二叉树的书稿会非常高。磁盘IO次数会更多。而B+树由于上述所说的特点，在数量级很大的前提下（千万级）。高度依然可以维持在3-4层左右。

vs Hash：

hash在获取等值，等值的时候效率非常高（O（1））。但是使用Hash没办法操作范围查询。



### 什么时候使用索引

- 经常使用where进行查询的字段
- 经常用于GROUP BY/ORDER BY的字段
- 有唯一性编码的字段



### 什么时候不需要创建索引？

- 使用不多（同上 WHERE / GROUPBY / ORDER BY中使用不到的字段）
- 存在的数据区分性不强， 重复较多（当重复性较强，mysql中的查询优化器甚至会选择不适用索引进行查询）

- 经常更新的字段
- 表数据太小（索引带来的收益 不如空间带来的开销）



### 索引什么时候会失效？

- 使用左模糊搜索，两边模糊搜索的情况下。（多种索引包括前缀索引等都会失效）

- 在where范围查询中，如果使用到了OR，而左方字段是索引列，右方字段不是。此时无法结合索引进行查询

- 在查询的时候，对索引列进行了计算，导致索引失效（*个人理解：底层mysql将age + 1看成一个具体值 并且与索引中记录的值不同 所以无法使用* 此处应为执行函数的时候，不是在原表上进行查询改动，而是创建一个新的临时表进行计算，这是为了保证索引值的有序性）
- 使用索引的时候，不遵循最左匹配原则（简单来说就是，尽量使索引中字段与查询语句中约束字段顺序保持一致）
- 索引列非常建议使用NOT NULL进行约束



### 有什么优化索引的方法？

- 前缀索引优化（减少索引字段空间大小 orderby无法使用 无法作为覆盖索引）
- 覆盖索引优化（覆盖所有使用where等使用时会涉及到的字段）
- 主键索引自增（在最小程度改动树结构的前提下 增加新的数据 因为往树里新数据一般情况下只是加一个新字段的事）
- 防止索引失效 如上



## 事务

### 事务有什么特性？

ACID

A 4 Atomic **原子性**。

事务开始运行后数据只有两个状态，成功修改和失败修改（即修改前）的状态。

通过undo_log实现，why？undo_log记录用户进行的操作，一旦事务执行失败，则通过undo_log中记录的语句进行回滚。

C 4 Consistency **一致性**

原子性保证了数据只会有两种状态。而一致性保证事务中所涉及的修改数据会一起修改。**从一个一致状态修改至另一个状态**。通过其他三个特性一起保证。

I 4 Isolation **隔离性**

隔离性即，事务启动时，与本次事务无关的变量不会受到影响而改变。每个事务与数据库中正在执行的事务是隔离的。通过MVCC的多版本并发控制实现，MVCC通过它的事务控制来进行。除了MVCC还有锁机制实现，与**事务具体的隔离级别**控制。

D 4 Durability **持久性**

持久性即，事务对数据进行的修改是永远的。即使存储介质等出现故障也不会消失。通过redo_log实现，redo_log中保存了mysql曾经进行过的操作。通过这个重做文件重做，保证数据持久性。



并发的数据处理会导致并发的事务出现。意味着MySQL会出现同时处理多个事务的情况。



### 并行事务时会出现什么问题？

**脏读**

运行的事务有A B。若A事务修改了X字段，但是还没提交。此时B事务读到了已经被A事务修改的X字段。A事务又回滚了。就会造成B事务读到的是错误的值（尚未提交修改的值）

![](https://cdn.xiaolincoding.com//mysql/other/10b513008ea35ee880c592a88adcb12f.png)

**不可重复读**

运行的事务同理A B。若A事务需要多次读取同一个值X。但是在两次读取之间B事务对其进行了修改。就会造成两次读取中数据不一样的问题

![](https://cdn.xiaolincoding.com//mysql/other/f5b4f8f0c0adcf044b34c1f300a95abf.png)

**幻读**

运行的事务同理A B。若A事务需要多次进行统一约束下的查询。而在两次查询的过程中，B事务对其数据进行了修改，增加或减少了A事务查询的结果。就会导致A事务两次查询中得出的记录数量不一致。

![](https://cdn.xiaolincoding.com//mysql/other/d19a1019dc35dfe8cfe7fbff8cd97e31.png)

注意：

不可重复读指的是，多次读取同一个字段的**数据值**不一致。

而幻读指的是，同一个约束条件查询出的**记录条数**不一样

` select *`和`select count(*)`的区别



三种问题衍生出了SQL中的不同隔离级别

### 数据库中数据事务有什么隔离级别？

**读未提交**：事务未提交时，做的变更对其他事务可见（此时会导致脏读）

**读提交**：事务提交之后，所作的变更才能被其他事务可见（未提交时获取的还是之前的值）

**可重复读**：事务启动之后，就确定了这一次所操作的事务的版本。在之后的操作将只会操作这一个版本的数据。是**MySQL InnoDB 引擎的默认隔离级别**

**串行化**：会对记录加上读写锁。若A事务对字段进行了读写操作会堵塞住直至事务提交。B事务或其他都无法对其进行读写。

每个级别间都有一定的区别。读未提交就是最原始的。到了读提交，保证了一定的一致性。但是若A事务在两次读取同一个字段间，B事务对其进行了修改。就会导致A事务所操作的数据并非同一个版本。所有产生了可重复读的隔离级别。

可以看到越往下它的一致性就越好。当时相对应，它的执行效率会更低。（CAP一致性权衡）

![](https://cdn.xiaolincoding.com//mysql/other/4e98ea2e60923b969790898565b4d643.png)

### MySQL在可重复读下解决幻读的举措

快照读：普通的select语句，通过**MVCC**保证不幻读和一致性。本质上就是MVCC中存储了很多事务以及他们的id，当一个事务开启的时候，会记录这个事务所涉及到的字段的id。并且在后续的查询语句的时候，只根据这一id标识的值进行修改。

当前读：涉及数据修改或加锁的语句。通过 **next-key lock（记录锁+间隙锁）**方式解决了幻读。当执行的时候，若其他事务在锁的范围内插入了记录，这个插的行为会被**阻塞**而无法插入。从而避免幻读产生。

> 典型语句：`SELECT ... FOR UPDATE`、`SELECT ... LOCK IN SHARE MODE`、`UPDATE`、`DELETE`、`INSERT`等涉及数据修改或加锁的语句。



## 锁

### 全局锁

使用？ 

```sql
flush tables with read lock;
```

释放？

```sql
unlock tables;
```

当使用了全局锁，就只能读，所有的写操作都会被阻塞



### 全局锁的应用场景？

数据库的**全库逻辑备份**。锁住后，就不会因为数据或表结构的更新，而导致备份的数据的文件出错。用户只能进行**读操作**。

有什么坏处？因为是全程不可用的。所以会将所有的更新操作都阻塞住。影响业务。

有什么解决办法？如果数据库的存储引擎支持的事务支持**可重复读**的隔离级别，可以创建一个快照（Read View）。

如果结合使用的话，就可以备份某一个版本的数据的同时，其他业务同时进行更新操作。

![image-20240911195023867](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240911195023867.png)



### 表级锁

- 表锁

- 元数据锁（MDL）
- 意向锁
- AUTO-INC锁

**表锁**

```sql
//表级别的共享锁，也就是读锁：
lock tables t_student read;
//表级别的独占锁，也就是写锁：
lock tables t_student write;
```

即**读锁与写锁**，同样可以使用unlock tables释放所有锁

**当当前会话退出时，也会释放所有锁**



### 元数据锁

对MDL的使用一般是数据库自动加上的。

例如说在对表进行CRUD数据操作，加上MDL读锁。防止在读的时候修改了表的结构

在对表结构做变更操作时，加的是MDL写锁。防止在单次修改的时候，又同时进行了其他结构上的修改

### 那么MDL锁是在什么时候释放的？

事务提交后，意味着**事务执行期间，MDL是一直持有的**

![image-20240911200600930](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240911200600930.png)

产生这个的原因就是因为。首先读取的时候上了读锁。但是此事务不提交，读锁不被释放。此时就只能读，如果来了一个修改的请求，需要获取写锁，此时因为已有读锁。所以操作也会阻塞住。并且又因为获取锁的这一操作本身是队列形式的，所以当获取了写锁之后，之后所有读的请求都需要等待写锁成功获取并释放后才能进行。也就是一直阻塞住了。

![image-20240911200841685](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240911200841685.png)



### 意向锁

本质上的运行过程就是，想往一个颗粒度更大的层面上尝试加锁。做一个类似兼容性检查，如果这个兼容性检查通过了，就代表接下来加锁的整个过程是合法的。



### 行级锁 

下方所设计的所有的锁都有S锁和X锁之分

S锁-读锁 X锁-写锁

#### Record Lock

记录锁 锁住特定的一条记录

#### Gap Lock

间隙锁 锁住特定值间的一个范围

#### 插入意向锁

即如上 锁住前的一个意向锁





## 算法

141. 环形链表

Leetcode164 最大间距

[93. 复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)

## 其他

### Mysql中使用唯一索引和普通索引有什么区别（查询）

Mysql通过索引进行查询的时候，都是先从树的树根进行查，查到对应的数据页当中。而之后，唯一索引在查找到对应的合法语句时，会直接返回，停止检索。普通索引则会接着往下查，直到不符合为止（由于不是唯一索引，无法保证唯一性的前提下，有可能存在多个合法语句）。

而查询的时候，也是一页一页查的，查到了。之后的所有查询操作原则上都是在此页上进行。但是万一所需要查询的语句刚好是某一页的最后一条记录，此时则需要再次对下一页进行查询。而唯一索引就没有这个问题

两者的效率差别就在这



### 介绍一下change_buffer

change_buffer用于执行普通索引时，对更新的效率做优化。它本身是可以持久化到硬盘当中的。在使用普通索引更新时，由于普通索引相关的数据不需要做前置校验（类似格式校验，唯一性校验等）。所以可以直接将单次更新的操作写到change_buffer之中。在用户下次读取这一个页的时候，再将数据拿出来之后，先对照change_buffer进行数据的更新（或者是定时更新）。在更新完之后再进行其他操作。

这里有几个问题，为什么需要这么做？change_buffer创建的意义何在？

因为数据库中最耗时的操作就是涉及磁盘的随机IO。当对数据进行多次更新，但是未读的前提下。change_buffer的存在节约了很多次更新磁盘的IO成本。

（不严谨的类比一下 其实可以类比成懒加载和饿加载）

并且，存储到change_buffer中的更新相关语句 是处理之后的，无需再次进行计算等操作。所以耗时会进一步减少。

同时，关于更多细节可以一句话总结为：Change Buffer 记录的是单条索引记录的更新操作，但在合并到磁盘时，操作是以 **页为单位** 进行，并使用 **页级锁** 进行锁定。



然后就引出了下面一个问题

### Mysql中使用唯一索引和普通索引有什么区别（更新）

结合上方的介绍，可以知道。

**普通索引**使用change_buffer辅助更新。当更新的时候，不直接将对应的值更新到磁盘中。而是打入change_buffer中。在下一次读取该资源的时候，就先执行change_buffer中相关语句的更新（if exist）

而对应**唯一索引**，由于再更新之前，需要进行前置的唯一性校验。所以不能保证这一次的更新操作是否成功。而校验这一次的操作需要将内存中的数据读取出来，而都已经将数据读取出来了，直接更新的成本比写入change_buffer的时间成本是更小的。（因为本质上在什么时候更新，都需要将数据加载到内存中。而此时已经加载完毕了，就可以趁机更新。少一次随机读取的时间成本）





综合上方的作用，我们就可以知道change_buffer带来的收益主要是：减少在更新时读取磁盘的IO成本。本质上，也就是修改的次数越多，更新的次数越多。change_buffer带来的收益就越多（**写多读少场景**）。

反之，如果修改的次数并不多，读取的时候越多，由于change_buffer的一个固定刷新时间是读取。那么本质上没有减少读取时所耗费的时间。反而增加了change_buffer的维护时空成本。





###  Change Buffer 与 Redo Log 的区别

| 特性         | Change Buffer          | Redo Log                           |
| ------------ | ---------------------- | ---------------------------------- |
| **作用对象** | 二级索引变更           | 所有页的变更                       |
| **主要目标** | 减少随机写读 I/O       | 减少随机写 I/O                     |
| **记录内容** | 二级索引的变更操作     | 物理级别的页修改操作               |
| **优化方向** | 避免频繁加载页进行更新 | 提交事务前只需写日志，减少刷盘压力 |
| **合并时机** | 查询访问或后台任务触发 | 崩溃恢复或检查点时直接回放日志     |



### CheckPoint刷新时机

CKP是标记脏页是否写入磁盘的一个标志位（将对应的redo_log持久化）

1. redo_log 接近满的时候，就会触发自动刷盘。假如刷盘操作未完毕的前提下redo_log写满了，此时会阻塞住所有的写入操作（新的操作无法写入日志）

2. 缓冲池中脏页的比例过高（此处的缓冲池中包含脏页，干净页，锁索引信息等常用缓存）

3. 定期触发

4. 手动触发

5. 数据库关闭

| 特性                       | **Redo Log 变满**                                          | **缓冲池中的脏页比例过高**                               |
| -------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **触发点**                 | Redo Log 空间接近耗尽，无法记录新的事务日志。              | 脏页占用的缓冲池空间比例超过阈值（默认 75%）。           |
| **本质原因**               | 日志写入压力大，Red Log 写入速度快于 Checkpoint 刷盘速度。 | 写入操作过多导致脏页比例增加，而后台 I/O 刷盘速度不足。  |
| **主要影响**               | 新的更新操作被阻塞，事务无法提交。                         | 查询可能受到影响，加载新页会被迫等待脏页刷盘完成。       |
| **缓解方式**               | 增大 `innodb_log_file_size`，优化事务提交逻辑。            | 调整 `innodb_max_dirty_pages_pct` 或提升磁盘 I/O 性能。  |
| **触发 Checkpoint 的逻辑** | 提前触发 Checkpoint 以释放 Redo Log 空间。                 | 提前触发 Checkpoint 以将脏页刷新到磁盘，释放缓冲池空间。 |

其实就可以理解成为一个滑动窗口，滑动窗口本身满了，或者达到最大刷新时延了，就将他传过去。



### count(*)效率很慢 为什么慢 有什么解决办法 和count(1)这些有什么区别？

count()函数的作用是计数。若其中字段为`not null`，函数直接遍历每一行并加1

若为`null`，函数会先将其拿出来判断是否为null，如果非null再加1

所以首先notnull修饰的效率会更高一点

而这个函数的本质作用机制是遍历表中的每一行，耗费时间O(n)

而除了count(*)之外，一般还有count(1),count(id),count(column)等方式

count(column)  & count(id) 的时候会先获取到单行的记录（可以走普通索引）

而count(1)则是在查询到对应行的时候，看能不能塞个1进去，只是单个字段，所以他的效率优于其他二者

count(*)则是在此之上专门做过优化的，查询的时候，不需要将字段取出来，所以他的查询效率更优于count(1)。在数据量大的前提下会更加明显

即`count(*) > count(1) > count(id) > count(column)` 后二者可以走普通索引优化查询，减少查询时所涉及的字段。

**怎么解决？**

可以记录一个数字，代表当前记录的行数。

**记在哪？**

Redis的话需要维护缓存，单机下就需要维护缓存与DB的一致性。并且分布式环境下也需要维护多节点Redis间值的一致性。

牺牲一部分性能的前提下，可以记录在Mysql中，同样再开一个表，再开一个事务，在**数据写入语句运行前，开启事务同时更新计数表中的记录数**。

那么假如现在同时有**数据写入语句和更新计数语句**，开启事务的时候，应该把谁放前面？

答案是**数据写入语句**，先插入再更新可以最大程度的减少事务之间的锁等待（**插入操作的耗时差异**），有效的错开事务之间的锁竞争，从而提升并发度。



### 两阶段提交（redo_log和bin_log）

###### 是什么？

在需要对数据进行写入、修改操作的时候，会触发的操作。

redo_log负责崩溃恢复操作。bin_log负责数据库备份。

本质上就是两阶段提交就是为了保证了两个日志的逻辑状态一致所作的措施。

首先先写入redo_log，并且标记为prepare状态。

然后写入bin_log，无误的话就标记为commit状态。

事务提交。

###### 为什么需要两阶段提交？如果只需要保证原子性直接一个事务不就可以了吗？

因为如果在写入redo_log后 在事务内系统直接崩溃了。

若没有两阶段提交 就导致redo_log的状态ahead于bin_log

而存在两阶段提交的话 就可以将redo_log中未同步的部分重新写入bin_log当中。

并且，两者是两套独立的日志系统：

redo_log属于存储引擎层，用于物理恢复。

bin_log属于服务层，用于逻辑恢复和复制。

###### 如果在写入redo_log之前或写入bin_log之后呢？此时会发生什么？

该丢的数据还是会丢，但是此时至少保证了redo_log和bin_log的状态是一致的。







### 跳表

Redis中的跳表是实现Zset这一存储数据类型的底层结构，他主要由多层链表组成。上层链表跨越多个元素，下层则跨越更少。查询的时候通过判断元素的权重值是否大于或小于当前值。假如在同一层链表节点中找不到对应的节点，则于当前节点的下一层开始查询，直至找到对应权重相同的节点。权重值相同后，判断他们的具体值(SDS)是否相同。假如相同，则是同一个元素，非相同则在此基础上往前查询

注意，一旦判断到权重值相同，则对值进行判断。而判断值是否相同的逻辑和判断权重是完全一样的，也是基于跳表进行查询。直至不断往下层查询查询到目标值。	





### 分布式锁

分布式锁的实现是由中间件进行实现的 分布式环境下 锁的对象有可能是一个库或多个库 在这个基础上 我们首先需要保证操作单个库的线程直接没有竟态问题 然后需要考虑被操作的库间是否同步 

分布式锁确实是一个工具，用来帮助达成分布式环境下多个数据库或资源之间的同步

我们是通过分布式锁作为工具达成数据库之间的同步

> ### **分布式锁由中间件实现**
>
> - 确认点
>
>   ：分布式锁是通过中间件实现的，目的是协调多个节点对共享资源的访问。
>
>   - 例如，Redis 的 `SETNX` 命令、ZooKeeper 的临时节点都是实现分布式锁的基础。
>   - 中间件的作用是提供全局唯一的锁标识，并在多个节点之间同步锁的状态。
>
> ------
>
> ### **2. 锁的对象可能是一个库或多个库**
>
> - **单个库**：
>   - 在分布式场景中，即使操作的只是单个数据库，多个线程或节点可能同时尝试对该库进行修改。此时，需要分布式锁来确保同一时间只有一个节点能修改数据。
>   - 例如：多个服务实例试图修改某个商品的库存。
> - **多个库（或资源）**：
>   - 如果需要对多个数据库或资源进行操作（如跨数据库的事务），分布式锁可以确保这些操作是原子的。
>   - 比如：同时更新两个分库中的表记录，必须通过锁确保在两个更新完成前其他节点不会介入。
>
> ------
>
> ### **3. 保证单个库的线程没有竞态问题**
>
> - **单机内部的线程竞争**：
>   - 如果是单机环境，普通的互斥锁（如 `synchronized` 或 `ReentrantLock`）足以解决线程间的竞态问题。
>   - 但在分布式环境中，仅靠单机的锁机制无法覆盖整个系统，因为多个节点上的线程会各自独立运行，不共享同一个锁。
> - **分布式环境的线程竞争**：
>   - 在分布式环境下，中间件（如 Redis）会负责协调多个节点的锁竞争，确保跨节点的全局互斥性。此时，分布式锁可以看作跨越单机线程锁的补充机制。
>
> ------
>
> ### **4. 被操作的库间是否同步**
>
> 这是分布式锁的一个关键点，尤其在多个库或资源之间有依赖关系时，操作同步性显得尤为重要。
>
> - **场景 1：单库操作**
>
>   - 如果只操作单个库，通过分布式锁即可控制对该库的并发访问，无需额外的同步机制。
>   - 例如：在 Redis 中实现一个锁，对一个 MySQL 数据表的更新进行保护。
>
> - **场景 2：跨库操作**
>
>   - **数据同步问题**：多个库之间可能存在数据依赖，如分布式事务。如果锁不能保证操作的原子性，可能会导致数据不一致。
>
>   - 解决方案
>
>     ：
>
>     - 使用分布式事务框架（如 `XA` 或基于 TCC 的事务管理）。
>     - 将跨库操作合并到一个分布式锁的临界区内，确保所有资源的访问都由同一个锁控制。
>
> ------
>
> ### **扩展补充**
>
> 1. **分布式锁和同步的关系**
>    - 分布式锁解决的是多个节点并发操作同一个资源的问题，但它本身并不直接解决资源间的同步问题。
>    - 如果资源之间存在依赖关系，仍需要通过事务、数据复制或逻辑协调来实现同步。
> 2. **操作单个库的线程是否存在竞态**
>    - 当使用分布式锁时，单个库的线程间一般不会有竞态问题，因为锁机制已经将并发控制提升到了分布式层面。
>
> ------
>
> ### **总结**
>
> 1. **分布式锁本质**：
>    它通过中间件提供了一种全局的互斥机制，解决分布式环境下多个节点竞争同一资源的问题。
> 2. **应用范围**：
>    - 当操作单个库时，分布式锁主要用来协调节点间的访问，库内线程的竞态问题可忽略。
>    - 当操作多个库或资源时，除了分布式锁，还需要考虑资源间的同步问题（如分布式事务）。
> 3. **关键点**：
>    - 确保单机线程安全后，分布式锁主要负责跨节点的并发控制。
>    - 数据库之间的同步（如一致性、事务性）需要在分布式锁基础上额外设计。



### 怎么人为设计一个死锁

人为实现一个死锁 A持有一个资源X B持有一个资源Y 此时A开启了事务需要操作资源Y 但是同时B也开启了事务需要操作资源X 双方操作对方的资源 但是都被锁住了 从而造成了死锁



### 缓存穿透的原理

用户重复查找不存在数据库中的数据，缓存没有DB也没有。所以每一次查询的缓存都会打到DB上，带来很大的压力。严重会击垮数据库。

### 缓存雪崩

某个时刻中大量的缓存同时失效，大量的查询同时打到DB上。数据库压力激增。

### 缓存击穿

某个**热点**key失效，对他的所有查询全部达到DB上。



### Redis更新缓存的策略

旁路缓存 来的时候先更新缓存 过了一段时间内把缓存中的数据写入数据库当中

更新的时候 更新数据库 并且同时把缓存删掉 重新回写



### InnoDB的特点

支持事务 行级锁 崩溃恢复能力（WAL）



### Cookie Session Token

- **Cookie** 用于在客户端存储小型数据，如 `sessionId` 或 Token。

- **Session** 是服务器端的会话管理，适合状态化场景，通过 `sessionId` 与客户端建立关联。

- **Token** 是一种现代化的认证方式，不依赖服务器存储会话，适合无状态和分布式架构。



### 分布式环境下的 Session 共享绑定

#### 什么是Session共享问题？

在分布式环境之下，有多个机器存储用户及其状态信息。在这一前提下。假如A机器上被用户做了修改。但是用户读取的时候，读取的是另外一个机器。此时他就丢失了他所作过的更改。

#### 怎么解决？

本质的解决思路是：

1. 让大家都访问同一个机器
2. 同步多个机器之间的状态
3. 传输的时候，客户端将所需的信息一并传到服务器中。客户端本身无状态

1. 使用Redis, MySQL等存储介质，将信息一并集中存储到该机器上。
2. 设定一个后台定时任务，类似XXL-JOB等框架，同步机器之间的信息。保持其之间的逻辑状态一致。存在延迟问题，可能需要有全局时钟保证任务同步执行。
3. 将所需的信息全部加密后放到Token传输, Session本身无状态。（此方案下的Token大小可能会很大，几百个字节也有可能）对传输本身带来负担



### 介绍一下策略模式

单个业务中可能可以有多个实现方式或者实现渠道。譬如说支付可以有零钱，银行卡。

此时可以抽象出一个支付渠道的枚举类，将不同方式下的方案都抽象成方法。根据传进来的不同策略选择不同方法进行执行。

使得他们可以互相替换，不影响使用他们的客户端代码。



### 线程池的核心线程数如何配置

判断任务是CPU密集型还是IO密集型

如果是IO密集型可以将核心数设置为远大于CPU核数

如果是CPU密集可以将核心数设置为近似CPU核数

这里所谓的CPU核数是本电脑的核数

具体的设置可以进行压力测试配合实施监控进行测试，得出最优结果

 

### JWT的组成和运行方式

头部 负载 签名

头部 metadata 标识签名的算法和类型

负载 data 实际传输的消息体

签名 signature 校验时需比对的加密后的密钥值

在校验的时候，首先分割其中的头部和负载。

然后通过本端的密钥和加密算法，再次对密钥进行加密，并且与原字符串拼接，判断是否相同

如果相同的话，就代表此JWT是合法的。

不同，则此JWT可能已经被纂改了

由于他的比对方式，可以知道他是不可逆的加密，而具体的加密方式可以根据使用者进行自行选择。





### Redis哨兵机制

Redis的哨兵机制指的是Redis会有多个独立的监控进程，实时监控现有的Redis节点，通过Ping他判断它是否可以正常工作。如果单个哨兵判断某一个节点已经无法工作，他会判断他为主观宕机（主观下线）。如果多个哨兵都认为它无法工作了，他就是客观下线。此时如果下线的节点不是主节点的话。他就不会尝试恢复这个节点（仅通知管理员）。如果下线的是主节点的话，Redis的多个哨兵就会协商选举一个新的主节点出来代替它。 

而选择新的节点的策略（选举策略）可以由用户指定优先级指定。除此之外的选举因素包括数据的完整性，节点的响应速度，节点的稳定性。在多个相同的前提下，则随机抽选一个。



### 介绍一下跨域问题以及解决方法

跨域问题的产生主要是因为浏览器的“同源策略”。

一般情况下，由于不安全，浏览器是不会允许页面向不同源的网站发起请求的。

所谓的同源就是 **域名相同 协议相同 端口号相同** 缺一不可

而如果其中一个条件不满足的话，浏览器会返回报错。

解决方法主要可以从两个角度：

1. Nginx —— 配置代理服务器

假如产生了跨域问题，产生的原因是因为接口的访问路径与前端的静态资源不在同一**源**下。此时可以以当前域名的路径访问，再经由Nginx进行反向代理，将所有的请求反向代理到服务器中。

2. CORS —— 配置跨域资源共享

在服务器中编写对应的配置，在后端处规定有哪些前端的域名是允许访问的。成功配置之后，就会在响应头中返回一个字段 `Access-Control-Allow-Origin`。其中包含的就是合法的，可跨域访问的资源路径。



### select / poll / epoll

#### select 

Linux中分为用户态和内核态。一般情况下，都只是在用户态执行操作，而select作为系统调用。用户执行的时候，操作系统会将涉及的所有fd，即文件描述符拷贝到内核态当中。之后select的所有操作都是在内核态中运行的。

拷贝之后，select会遍历所有socket，看看有没有事件。如果没有的话，此时会阻塞等待。

> FD与Socket之间的关系：
>
> **文件描述符（FD）** 是一个标识符，用来表示一个打开的 I/O 资源。它是一个整数，可以代表套接字、文件、管道等。
>
> **套接字（Socket）** 是一种特定类型的 I/O 资源，用于进程间的通信，尤其是网络通信。

如果遍历之后某个时间点客户端发送了数据，对应的socket需要通知select。此处需说明，每个socket都会维护一个睡眠队列，其中有一个entry专门用于通知客户端。假如发送了数据，socket就会遍历整个队列，直至调用到整个entry，触发它的回调函数。

类似的，每个socket都会有个entry，所以无论是 哪个线程来活了，socket都可以在回调函数触发后立马被唤醒，然后开始干活。

但是这个回调函数唤醒的逻辑中并没有包括唤醒者的标识（fd），所以select还是需要重新遍历一次所有的fd，直至找到来活的select，实现逻辑。

> **Q：**以上是select的执行逻辑 现在我有个问题 假如说某个时刻A和B都来活了，A先通知并唤醒了select，B后唤醒，但是当select语句运行的时候，它却先select遍历到了B。这种情况是合法出现的吗
>
> **A：**合法且在事件驱动架构中常见的，文件描述符的遍历顺序不依赖于回调的触发顺序，仅会依赖于某些事件是否发生。

但是由于整个过程包括拷贝，遍历都是在内核态中运行的，所以整个大小不宜太大。每个select能拷贝的fds集合只有**1024**



#### poll

相比于select，这种方式主要做的改进是更换了存储的介质。

在select中，存储每个fd及其值的方式是**位图**bitmap。而poll则是更换为pollfd的结构体数组。相比之下，bitmap的数据结构规定了在拷贝的时候需要提前规定内存的限制（不能动态拓展），并且它的最大值是1024。并且在每一次被唤醒后，他都需要把整个位图都遍历一遍，才能获得所需信息。可以说是严格O(n)

而poll的结构体动态数组可以动态分配内存，并且s在检测到就绪事件后，可以继续遍历而不是强制遍历整个集合。无需严格O(n)



#### epoll TBD

可以看到在poll和select中，一旦发生了新事件，都需要将全部fd都遍历一遍。而Epoll中本身维护了一个事件队列，假如有事件发生，操作系统会知道是直接知道是谁发生的（类比一下可以看成直接知道是谁唤醒了select），直接将查询时间O(n)变为了O(1)。





#### 关于三者的 **用户态 & 内核态** 拷贝细节

本质上其实三者的拷贝时机是类似的：

**更新需要遍历查询的数组 返回就绪并事件处理完毕的数组**

对于**select** & **poll**

他们在每一次更新数组的时候都需要全量更新，即在原来的基础上，将所需要标记的fds**全量拷贝**到内核态当中。

而对于**epoll**

只需要在原来的基础上增加或减少对于的fd就可以了，可以理解为**增量拷贝**



#### ET & LT (TBD)





### 生产中什么情况下mysql性能会很低

#### 连接数多

##### 减少连接 kill掉没有执行任务的连接

事务外空闲的线程是最优先的 然后到事务内空闲

提高最大连接数的数量 (max_connections)

但是需要酌情加 因为提高连接数同时 相当于把更多的CPU资源分配到连接上了

变相也会减少用在执行语句上的CPU

##### 跳过权限校验阶段

兜底方案



#### 慢查询性能问题

通过慢查询日志逐步调查是哪里出的问题

slow_log中的long_query_time一步步检查

而慢查询一般情况下分为三种

**索引没设计好 sql语句没写好 mysql执行错了索引**

1. 索引：关掉bin_log后 在从库上创建索引 执行主从切换 关掉bin_log 再次创建索引
2. sql：使用query_rewrite进行重写 这个是基于正则查找符合重写条件的语句的 所以**有可能会影响到其他符合同样正则条件的语句** 
3. 错索引：mysql是通过统计信息估算出 扫描行数 并根据这个扫描行数判断使用什么索引的。所以会有这个概率用错 这时候可以使用force index 强制该语句使用对应的索引。但是在此之前需要使用explain 查看使用计划 是否索引使用错误的问题

总结一下就是：慢查询日志定位慢查询位置

索引慢创索引主从切换

sql错使用rewrite重写sql

错索引使用force index 也可以借助这个rewrite插件执行



#### QPS突增

在应用层 这种问题一般都使用老生常谈的**熔断 限流**来解决。

mysql没有提供上述的模块，但是也可以采用另一措施实现类似熔断的效果。

如果qps徒增 业务方也需要采取措施 假设业务方会下掉该功能模块（分布式环境） DB可以从白名单中ban掉这个ip

同理 如果这个业务是单独开启一个角色连接的 DB也可以临时删除掉这个角色

上两种方法都是通过临时禁用 拒绝它的查询请求实现的。

还有一种方案是通过上方的query_rewrite将对应的热点语句修改为“select 1”,虽然保证了执行效率，但是这种方案也会导致该业务的下游业务（如果有的话）全部乱套，或者也可以在业务层对这个做一个特殊判断。

总结就是：

谁出问题ban谁

临时修改语句为负担小语句





### 简单介绍一下Kafka的架构

kafka是一个消息队列 他主要将消息都存储到broker中 对于每一个broker 首先不同的主题的消息会分发到不同的topic之中 对于每一个topic 可能他的信息量还会很大 所以在此基础上又分为多个分区partition 注意 此处的多个分区可以挂载在不同的主机上 对于每一个分区 为了防止数据的丢失 通常会给他们都设置数据的备份 也就是老生常谈的leader 和 follower架构。



### 介绍一下BufferPool的设计

BufferPool是MySQL中的缓存方案。

它存储的内容可以分为：

**数据页，索引页，锁信息，动态哈希索引页，插入缓存页，undo页**

前三者很容易理解，就是对所接触到的热点数据的一个缓存，

而对于动态哈希索引，他是对常用的热索引再构建一个哈希索引。我们知道普通的索引查询是通过B+树进行遍历的，而在运行之中，MySQL会自动辨别是否有部分索引的使用频率高，如果有的话，会在原b+树的基础上再次构建一个哈希索引。将查树时O(logn)的时间耗费进一步减为O(1)。

对于undo页，它本质上是和undo_log一起作用的。在事务回滚的时候，或者崩溃恢复的时候，它用于回滚至对于事务未执行前的状态，保证数据的逻辑一致性。

对于插入缓存页，其实就是change_buffer... 作用差不多就是对更新的操作做修改之后**，记录下修改的操作**，先不刷回磁盘，定期刷回去。脏页



上方是根据存储的数据类型分辨出它是什么页，而如果按照修改操作对他们分类的话，我们可以得到

**脏页（修改过的），空闲页（没被修改过的）**

假如此时数据库需要将新的数据存入BufferPool，他需要知道哪里是空的。而页在内存中的分布是随机的。所以MySQL设计了一个链表将他们绑定起来。并且每一个页都有一个类似控制块的东西，存储了这个页的原信息。并且在链表中作为标识符。

对于空闲页的链表可以成为FreeList，MySQL只需要将空闲页连起来，有数据插入的时候，再从链表中将他删去就可以了。

对于脏页，也有这样的一个链表，标识了脏页需刷盘的时机。本质上也是为了让MySQL知道有哪一些页是脏的，好把他们都刷到内存中。

**LRUList**

在此之外还会有一个LRU链表

通过LRU算法来区分冷热数据，并且将冷数据逐出BufferPool中。**无论是脏页还是空闲页，都会出现在这个LRU list当中**。

但是普通的LRU算法不足以满足需求。因为按照LRU的话，会有两个问题：预读污染，BufferPool污染。（**污染可以大致理解为无效数据**）

**预读时读出的页，会抢占热数据的位置，将原来的热点数据逐出了BufferPool**

> 什么是预读？
>
> 有页A B C
>
> 根据局部性原理，程序读取的时候，一般都是读取某小部分的数据。此时假如读B，会将相邻的页，则A\C都一并读取进来

还有一个问题是BufferPool污染，假如做了一个模糊查询的时候，查询范围很大，需要占了很大的BufferPool空间。但是所需的有效数据只有极少数。此时LRU链表中，都是读进来做模糊查询的无效页。

所以根据这个问题，MySQL在原来的基础上对其做了修改。

首先，将原始的LRU链表分为OLD,YOUNG两个部分。

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/mysql/innodb/lrutwo.drawio.png)

针对预读时所载入的无效页，**本质上是尽量别让它顶替了原热点数据的位置**

预读时的相邻页，在读进来的时候，插入到OLD区域的第一个位置。

而如果bufferpool设定合理的话，所有的热点数据就会一直在young区域中。不会被顶替掉。

**同时保证了局部性原理和热点数据**



对于上方所说的bufferpool污染，本质上也是判断它是否热点数据。

而对于模糊查询情景时的附带无效页，一般来说只会模糊查询一次。

所以MySQL设计为假如该页的后续访问和当前访问相隔时间过短，就不会将这个页放到young的头部，而是进入old的头部。而这个门槛时间可以通过控制 `innodb_old_blocks_time`来修改。

> ps: 这样的设计可能会出现问题：
>
> 假如我现在读取数据的时候，将一个页以及它的相邻页都读取到了内存当中（预读）。
>
> 此时它的预读页是在old区域的头部
>
> 然后我对这三个页，进行极高频率的范围查询（在innodb_old_blocks_time内），此时会不断的读取这三个页，但是他们不会移到young区域的头部，只会移到old区域的头部。
>
> 如果三个页的话不会导致被淘汰什么的，但是假如某个业务的主要功能是这个，没准会导致大面积页都只能到old区域，逐出bufferpool了
>
> 某种程度上是MySQL对**冷热数据的误判**
>
> 但是可以通过设计好`innodb_old_blocks_time`解决

GPT如是说道：

> ###### 问：
>
> 所以这个是不是MYSQL将其误判成为了冷数据 有什么解决办法吗
>
> 
>
> ###### ChatGPT 说：
>
> 是的，你的理解是对的。在你描述的场景中，MySQL 可能会将频繁访问的范围查询的相邻页误判为 **冷数据**，并将它们放入 **旧链表** 中。这可能会导致它们在内存中的位置不够优先，进而影响查询性能，特别是在高频访问的范围查询场景中。
>
> ### 为什么会误判为冷数据？
>
> 在 MySQL 的 **Buffer Pool** 设计中，频繁访问的页会被移入 **旧链表**，而不直接进入 **LRU链表头部**，这是为了避免热点数据频繁进出头部带来的额外开销。**频繁访问的范围查询** 会使一批相邻的数据页都频繁被加载和访问，但这些页并不会马上重新进入 **LRU链表头部**，而是被移动到 **旧链表**，从而被误认为是“冷数据”。
>
> 这种误判的根本原因在于，MySQL 的缓存机制是基于 **访问模式**（例如，短期频繁访问）来决定数据页是否应进入 **LRU链表头部** 或 **旧链表头部**，而不是基于查询的 **访问频率** 或 **数据页的实际活跃度**。



