# Spring 事务

> 合并自：`Spring事务.md`、`@Transactional注解常见的坑.md`。原文未删改（仅修正已知错误、调整格式）。

## 一、Spring 事务的工作机制

> 来源：`Spring/Spring事务.md`

Spring 事务本质上是把「数据库事务」交给 Spring 通过 AOP 管理（开启 → 提交/回滚）。但**「开启 Spring 事务」并不等于「加数据库锁」**：在 RR/RC 隔离级别下普通 `SELECT` 走 MVCC 快照读、并不加锁；只有写操作（以及 `SELECT ... FOR UPDATE` / `LOCK IN SHARE MODE` 这类当前读）才会加行锁。所以把它理解成"开启事务就用锁"是不准确的。

spring事务实际使用AOP拦截注解方法，然后使用动态代理处理事务方法，捕获处理过程中的异常，spring事务其实是把异常交给spring处理；

spring事务只有捕获到异常才会终止或回滚，如果你在程序中try/catch后自己处理异常而没有throw，那么事务将不会终止或回滚，失去事务本来的作用；

Spring 事务的回滚规则是：**默认只对 `RuntimeException` 和 `Error` 回滚**，受检异常（如 `IOException`）默认**不回滚**；要让受检异常也回滚，需要显式声明 `@Transactional(rollbackFor = Exception.class)`。另外被 catch 掉、没有继续抛出的异常，事务同样不会回滚。；

spring事务会回滚同一事务中的所有数据库操作，本质上是回滚同一数据库连接上的数据库操作；

Spring事务传播属性

| 常量名称 | 常量解释 |
| --- | --- |
| REQUIRED | 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是 Spring 默认的事务的传播。 |
| REQUIRES_NEW | 新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作 |
| SUPPORTS | 支持当前事务，如果当前没有事务，就以非事务方式执行。 |
| MANDATORY | 支持当前事务，如果当前没有事务，就抛出异常。 |
| NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 |
| NEVER | 以非事务方式执行，如果当前存在事务，则抛出异常。 |
| NESTED | 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。 |

Spring的隔离级别

| 常量 | 解释 |
| --- | --- |
| ISOLATION_DEFAULT | 这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与 JDBC 的隔离级别相对应。 |
| ISOLATION_READ_UNCOMMITTED | 这是事务最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 |
| ISOLATION_READ_COMMITTED | 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。 |
| ISOLATION_REPEATABLE_READ | 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。 |
| ISOLATION_SERIALIZABLE | 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。 |

## 二、@Transactional 常见的坑

> 来源：`Spring/@Transactional注解常见的坑.md`

@Transactional注解常见的坑

**注解失效**

1、加@Transaction的方法必须是public，否则失效

spring AOP    jdk代理反射情况无法访问private

cglib情况无法被继承

2、在同一个类里，两个@Transaction方法直接嵌套调用会失效

A方法调用B方法，B方法加上@Transaction注解，如果A，B方法在同一个类里，则方法B的@Transaction注解失效。因为@Transaction注解实现原理是AOP，如果是对象this自己的调用就不会走aop的代理对象。

解决方法：将A，B方法放在两个类中，或是使用AopContext.currentProxy()

### 事务超时

Spring事务超时判断只是在执行数据库操作时候，所以如果超时后没有数据库操作，则无法准确超时。

```java
    @Transactional(timeout = 5)
    public void timeout() throws InterruptedException {
        Thread.sleep(1000000);
        insert();//会报超时异常
    }

    @Transactional(timeout = 5)
    public void timeout() throws InterruptedException {
        insert();
        Thread.sleep(1000000);//这个方法不会超时异常，因为超时后没有数据库操作
    }
```

@Transactional注解的方法

当Transaction内发生unchecked exception的时候，会自动rollback，但是当Transaction内发生checked exception时，是不会自动rollback的

如果使用try-catch捕获异常，则该事务不会回滚（catch中手动抛出runtimeexcetpion会回滚）

spring aop异常捕获原理：被拦截的方法需显式抛出异常，并不能经任何处理，这样aop代理才能捕获到方法的异常，才能进行回滚
