# MySQL 日志（binlog / redo log / undo log）

> 来源：`MySQL Log.md`

对于MySQL的日志主要介绍binlog、redo log、undo log

binlog：记录了数据库表结构和表数据变更，比如`update/delete/insert/truncate/create`。它不会记录`select`（因为这没有对表没有进行变更）

`binlog`我们可以简单理解为：存储着每条变更的`SQL`语句（当然从下面的图看来看，不止SQL，还有XID「事务Id」等等）

binlog主要有两个作用：复制和恢复数据

-   MySQL在公司使用的时候往往都是一主多从结构的，从服务器需要与主服务器的数据保持一致，这就是通过binlog来实现的
-   数据库的数据被干掉了，我们可以通过binlog来对数据进行恢复。

MySQL的基本存储结构是页（记录都存在页里面），所以MySQL是先把这条记录所在的页找到，然后把该页加载到内存中，将对应记录进行修改。现在就可能存在一个问题：如果在内存中把数据改了，还没来得及落磁盘，而此时的数据库挂了怎么办？显然这次更改就丢了。如果每个请求都需要将数据立马落磁盘之后，那速度会很慢，MySQL可能也顶不住。所以MySQL是怎么做的呢？

MySQL引入了`redo log`，内存写完了，然后会写一份`redo log`，这份`redo log`记载着这次**在某个页上做了什么修改**。其实写`redo log`的时候，也会有`buffer`，是先写到 buffer 里的。

![图片](../assets/MySQL%20Log/img-01.png)

写`redo log`也是需要写磁盘的，但它的好处就是`顺序IO`（我们都知道顺序IO比随机IO快非常多）。

所以，`redo log`的存在为了：当我们修改的时候，写完内存了，但数据还没真正写到磁盘的时候。此时我们的数据库挂了，我们可以根据`redo log`来对数据进行恢复。因为`redo log`是顺序IO，所以**写入的速度很快**，并且`redo log`记载的是物理变化（xxxx页做了xxx修改），文件的体积很小，**恢复速度很快**。

binlog和redo log区别

`binlog`记载的是`update/delete/insert`这样的SQL语句，而`redo log`记载的是物理修改的内容（xxxx页修改了xxx），`redo log` 记录的是数据的**物理变化**，`binlog` 记录的是数据的**逻辑变化**

`redo log`的作用是为**持久化**而生的。写完内存，如果数据库挂了，那我们可以通过`redo log`来恢复内存还没来得及刷到磁盘的数据，将`redo log`加载到内存里边，那内存就能恢复到挂掉之前的数据了。

`binlog`的作用是复制和恢复而生的。

-   主从服务器需要保持数据的一致性，通过`binlog`来同步数据。
-   如果整个数据库的数据都被删除了，`binlog`存储着所有的数据变更情况，那么可以通过`binlog`来对数据进行恢复。

又看到这里，你会想：”如果整个数据库的数据都被删除了，那我可以用`redo log`的记录来恢复吗？“**不能**

因为功能的不同，`redo log` 存储的是物理数据的变更，如果我们内存的数据已经刷到了磁盘了，那`redo log`的数据就无效了。所以`redo log`不会存储着**历史**所有数据的变更，**文件的内容会被覆盖的**。

`redo log`是MySQL的InnoDB引擎所产生的。

`binlog`无论MySQL用什么引擎，都会有的。

InnoDB是有事务的，事务的四大特性之一：持久性就是靠`redo log`来实现的（如果写入内存成功，但数据还没真正刷到磁盘，如果此时的数据库挂了，我们可以靠`redo log`来恢复内存的数据，这就实现了持久性）。

上面也提到，在修改的数据的时候，`binlog`会记载着变更的类容，`redo log`也会记载着变更的内容。（只不过一个存储的是物理变化，一个存储的是逻辑变化）。那他们的写入顺序是什么样的呢？

`redo log`**事务开始**的时候，就开始记录每次的变更信息，而`binlog`是在**事务提交**的时候才记录。

于是新有的问题又出现了：我写其中的某一个`log`，失败了，那会怎么办？现在我们的前提是先写`redo log`，再写`binlog`，我们来看看：

-   如果写`redo log`失败了，那我们就认为这次事务有问题，回滚，不再写`binlog`。
-   如果写`redo log`成功了，写`binlog`，写`binlog`写一半了，但失败了怎么办？我们还是会对这次的**事务回滚**，将无效的`binlog`给删除（因为`binlog`会影响从库的数据，所以需要做删除操作）
-   如果写`redo log`和`binlog`都成功了，那这次算是事务才会真正成功。

简单来说：MySQL需要保证`redo log`和`binlog`的**数据是一致**的，如果不一致，那就乱套了。

-   如果`redo log`写失败了，而`binlog`写成功了。那假设内存的数据还没来得及落磁盘，机器就挂掉了。那主从服务器的数据就不一致了。（从服务器通过`binlog`得到最新的数据，而主服务器由于`redo log`没有记载，没法恢复数据）
-   如果`redo log`写成功了，而`binlog`写失败了。那从服务器就拿不到最新的数据了。

MySQL通过**两阶段提交**来保证`redo log`和`binlog`的数据是一致的。

过程：

-   阶段1：InnoDB`redo log` 写盘，InnoDB 事务进入 `prepare` 状态
-   阶段2：`binlog` 写盘，InooDB 事务进入 `commit` 状态
-   每个事务`binlog`的末尾，会记录一个 `XID event`，标志着事务是否提交成功，也就是说，恢复过程中，`binlog` 最后一个 XID event 之后的内容都应该被 purge。

![图片](../assets/MySQL%20Log/img-02.png)

当MySQL写完redolog并将它标记为prepare状态时，并且会在redolog中记录一个XID，它全局唯一的标识着这个事务。而当你设置`sync_binlog=1`时，做完了上面第一阶段写redolog后，mysql就会对应binlog并且会直接将其刷新到磁盘中。

binlog结束的位置上也有一个XID。只要这个XID和redolog中记录的XID是一致的，MySQL就会认为binlog和redolog逻辑上一致。就上面的场景来说就会commit，而如果仅仅是rodolog中记录了XID，binlog中没有，MySQL就会RollBack

`undo log`主要有两个作用：回滚和多版本控制(MVCC)

在数据修改的时候，不仅记录了`redo log`，还记录`undo log`，如果因为某些原因导致事务失败或回滚了，可以用`undo log`进行回滚

`undo log`主要存储的也是逻辑日志，比如我们要`insert`一条数据了，那`undo log`会记录的一条对应的`delete`日志。我们要`update`一条记录时，它会记录一条对应**相反**的update记录。

这也应该容易理解，毕竟回滚嘛，跟需要修改的操作相反就好，这样就能达到回滚的目的。因为支持回滚操作，所以我们就能保证：“**一个事务包含多个操作，这些操作要么全部执行，要么全都不执行**”。【原子性】

因为`undo log`存储着修改之前的数据，相当于一个**前版本**，MVCC实现的是读写不阻塞，读的时候只要返回前一个版本的数据就行了。

## binlog刷盘时机

对于 `InnoDB` 存储引擎而言，只有在事务提交时才会记录 `biglog` ，此时记录还在内存中，那么 `biglog`
是什么时候刷到磁盘中的呢？ `mysql` 通过 `sync_binlog` 参数控制 `biglog` 的刷盘时机，取值范围是 `0-N`：

-   0：不去强制要求，由系统自行判断何时写入磁盘；
-   1：每次 `commit` 的时候都要将 `binlog` 写入磁盘；
-   N：每N个事务，才会将 `binlog` 写入磁盘。

从上面可以看出， `sync_binlog` 最安全的是设置是 `1` ，这也是 `MySQL 5.7.7`
之后版本的默认值。但是设置一个大一些的值可以提升数据库性能，因此实际情况下也可以将值适当调大，牺牲一定的一致性来获取更好的性能。

### binlog日志格式

`binlog` 日志有三种格式，分别为 `STATMENT` 、 `ROW` 和 `MIXED` 。

> 在 `MySQL 5.7.7` 之前，默认的格式是 `STATEMENT` ， `MySQL 5.7.7` 之后，默认值是 `ROW` 。日志格式通过 `binlog-format` 指定。

-   `STATMENT` ： 基于 `SQL` 语句的复制( `statement-based replication, SBR` )，每一条会修改数据的sql语句会记录到 `binlog` 中 。

    ```
    * 优点： 不需要记录每一行的变化，减少了` binlog ` 日志量，节约了 ` IO ` , 从而提高了性能；
    * 缺点： 在某些情况下会导致主从数据不一致，比如执行` sysdate() ` 、 ` slepp() ` 等 。
    ```

-   `ROW` ： 基于行的复制( `row-based replication, RBR` )，不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了 。
    -   优点： 不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题 ；
    -   缺点： 会产生大量的日志，尤其是 `alter table` 的时候会让日志暴涨
-   `MIXED` ： 基于 `STATMENT` 和 `ROW` 两种模式的混合复制( `mixed-based replication, MBR` )，一般的复制使用 `STATEMENT` 模式保存 `binlog` ，对于 `STATEMENT` 模式无法复制的操作使用 `ROW` 模式保存 `binlog`

`redo log` 包括两部分：一个是内存中的日志缓冲( `redo log buffer` )，另一个是磁盘上的日志文件( redo log file `)。` mysql `每执行一条` DML `语句，先将记录写入` redo log buffer ，后续某个时间点再一次性将多个操作记录写到 `redo log file` 。

在计算机操作系统中，用户空间( `user space` )下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间( kernel space `)缓冲区(` OS Buffer `)。因此，` redo log buffer `写入` redo log file `实际上是先写入` OS Buffer `，然后再通过系统调用` fsync() `将其刷到` redo log file 中，过程如下：

`mysql` 支持三种将 `redo log buffer` 写入 `redo log file` 的时机，可以通过 `innodb_flush_log_at_trx_commit ` 参数配置，各参数值含义如下：

| 参数值 | 含义 |
| --- | --- |
| 0（延迟写） | 事务提交时不会将 `redo log buffer` 中日志写入到 `os buffer` ，而是每秒写入 `os buffer` 并调用 `fsync()` 写入到 `redo log file` 中。也就是说设置为0时是(大约)每秒刷新写入到磁盘中的，当系统崩溃，会丢失1秒钟的数据。 |
| 1（实时写，实时刷） | 事务每次提交都会将 `redo log buffer` 中的日志写入 `os buffer` 并调用 `fsync()` 刷到 `redo log file` 中。这种方式即使系统崩溃也不会丢失任何数据，但是因为每次提交都写入磁盘，IO的性能较差。 |
| 2（实时写，延迟刷） | 每次提交都仅写入到 `os buffer` ，然后是每秒调用 `fsync()` 将 `os buffer` 中的日志写入到 `redo log file` 。 |

### redo log记录形式

前面说过， `redo log` 实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此 `redo log`
实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志。如下图：

同时我们很容易得知， 在innodb中，既有 `redo log` 需要刷盘，还有 `数据页` 也需要刷盘， `redo log` 存在的意义主要就是降低对 `数据页` 刷盘的要求 。在上图中， `write pos` 表示 `redo log` 当前记录的 `LSN` (逻辑序列号)位置， `check point` 表示 数据页更改记录 刷盘后对应 `redo log` 所处的 `LSN` (逻辑序列号)位置。 `write pos` 到 `check point` 之间的部分是 `redo log` 空着的部分，用于记录新的记录； `check point` 到 `write pos` 之间是 `redo log` 待落盘的数据页更改记录。当 `write pos` 追上 `check point` 时，会先推动 `check point` 向前移动，空出位置再记录新的日志。

启动 `innodb` 的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。因为 `redo log` 记录的是数据页的物理变化，因此恢复的时候速度比逻辑日志(如 `binlog` )要快很多。 重启 `innodb` 时，首先会检查磁盘中数据页的 `LSN` ，如果数据页的 `LSN` 小于日志中的 `LSN` ，则会从 `checkpoint` 开始恢复。 还有一种情况，在宕机前正处于`checkpoint` 的刷盘过程，且数据页的刷盘进度超过了日志页的刷盘进度，此时会出现数据页中记录的 `LSN` 大于日志中的 `LSN`，这时超出日志进度的部分将不会重做，因为这本身就表示已经做过的事情，无需再重做。
