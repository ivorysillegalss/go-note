# sql中update语句的执行流程

对于更新这一操作，其大概执行链路于查询语句还是差不多的。

例如有下方语句：

```sql
mysql> create table T(ID int primary key, c int);
mysql> update T set c=c+1 where ID=2;
```

执行语句前需连接数据库。

若MySQL版本小于8.0且启用了查询缓存的功能。则如前文所说，对应的表的所有查询缓存信息都会被清空。

通过分析器分析出其为更新语句。使用ID作为索引。执行器最后执行并更新。

在更新过程中涉及到了`redo_log`和`bin_log`两个日志模块。

MySQL整体可分为Server层和引擎层。

前者负责数据库各种功能的具体实现，后者则负责数据的存储

此处所说的`redo_log`是引擎层的，`bin_log`则是Server层的。



**redo_log**

此日志模块为重做日志，是实现MySQL中的WAL技术的重要组成。InnoDB中有一个日志缓冲区(log buffer)，可用于存储此`redo_log`

WAL技术（Write-Ahead Logging），关键点是**先写日志，再写磁盘**，在某种程度上有点类似Redis的AOF。因为每次当数据来临的时候，将其从磁盘中找到并更新的IO成本是较高的，所以MySQL会先将数据更新的信息存储到redo_log当中

默认情况下，InnoDB后台线程会定期将此日志缓冲区中的内容刷新到硬盘，默认**每秒执行一次**。

但是`redo_log`是固定大小且可配置的，譬如说可以配置为4个1GB的文件。此时`redo_log`区域一共可记录4GB的操作。

![image-20240715154921501](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240715154921501.png)

上方图是一个简易的`redo_log`的示意图：

- **write pos**是当前写入的位置（将用户的操作记录到redo_log当中）
- **checkpoint**是当前要擦除的位置（将redo_log中操作持久化到数据文件当中）、
- 一共有4文件，两者写到文件3末尾则回到0号文件开头

根据上图，稍作分析可知此时`logfile_3`和`logfile_0` 是空闲的，可以提供用户进行新的更新操作。但是如果`write_pos`追上了`checkpoint`此时就表示`redo_log`区域满了，不允许进行新的更新。

而当数据库发生异常重启的时候，以往所提交的记录都不会丢失（存储到了redo_log当中），这一能力称为**creash-safe**



**binlog**

binlog的主要作用是归档，并没有类似`redo_log`的**crash-safe**的能力。

此处可再次澄清关于InnoDB与MySQL的关系，前者是后者能适配使用的存储引擎之一。MySQL自带的引擎是MylSAM。而MySQL的架构中，存储引擎是策略化的，可配置的。**InnoDB是第三方以插件形式引入MySQL的**。并且由于binlog没有crash-safe的能力，所以才衍生出了`redo_log`。所以在使用范围上，**redo_log是InnoDB引擎特有的，而binlog是Server层实现的，所以所有引擎都能使用。**

同时，两者在其他方面也有一定的差异：

- redo log是物理日志，记录的是“在某个数据也上做了什么修改”，假如实际使用中有对相同的数据页进行多次修改，则redo_log可以将他们合并到一起；而binlog是逻辑日志，记录的是操作的sql语句的原始逻辑，如“给ID=2的T字段加1”

- redo log是多个文件循环写的，可配置文件的参数，也有可能会被用完。但是binlog是追加写入，指写到一定大小切换下一个。



> 了解完了两个日志，那么他们的执行时机又是什么时候呢？

在执行器调用引擎接口的时候，会同时将此更i性能操作记录到redo log当中，此处redo log状态为**prepare**。最后提交事务就绪。

- 引擎将这行新数据更新到redo log缓冲区中。

- 此时，redo log 处于 **prepare** 状态。redo log 记录的是物理层面的修改。

- InnoDB引擎通知执行器更新操作已经完成，可以准备提交事务。
- 执行器生成这个操作的binlog。binlog 记录的是SQL语句的逻辑层面操作，生成的binlog会首先写入binlog缓存，然后写入磁盘中binlog文件。

- 执行器通知InnoDB引擎可以提交事务。InnoDB引擎将 redo log 从 prepare 状态变为 **commit** 状态。

- InnoDB将 redo log 缓冲区中的内容持久化到磁盘上的 redo log 文件。



从上方的工作流程可以看到日志提交的时候会有两个状态，这是为了确保日志文件间的一致性。

