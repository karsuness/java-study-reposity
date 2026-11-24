# java-study-reposity

> Java 后端学习笔记与知识体系整理

## 简介

本仓库整理 Java 后端开发的核心知识，覆盖从语言基础、并发编程、虚拟机原理，到主流框架、中间件、数据库及分布式技术方案。

每篇笔记按「原理 → 实现 → 实践」的思路组织，力求说清楚**为什么这样设计**，而不只是罗列结论。内容既适合系统复习，也适合作为面试准备与日常问题排查的参考手册。

**内容规模**：47 篇笔记，涵盖 6 大知识模块、18 个细分主题。

## 知识模块

| 模块 | 说明 |
| --- | --- |
| Java | 语言基础、集合框架、并发编程、JVM |
| MySQL | 存储引擎、事务与锁、索引优化、日志 |
| Spring | 核心容器、AOP、事务、SpringBoot、SpringCloud |
| 中间件 | Redis、MQ、Dubbo、ZooKeeper、MyBatis |
| 技术方案 | 分布式事务、限流算法 |
| 计算机 | 计算机网络 |

---

## 目录

### 一、Java

#### [语言基础](Java/基础)

- [Java 语言基础](Java/基础/Java语言基础.md)
- [Java 语言特性与底层实现](Java/基础/Java语言特性与底层实现.md)
- [String](Java/基础/String.md)
- [反射与动态代理](Java/基础/反射与动态代理.md)
- [过滤器与拦截器](Java/基础/过滤器和拦截器.md)
- [五大 IO 模型](Java/基础/五大IO模型.md)
- [树的比较](Java/基础/树-比较.md)

#### [集合框架](Java/List)

- [集合框架与 ArrayList](Java/List/集合框架与ArrayList.md)
- [ArrayDeque](Java/List/ArrayDeque.md)
- [SkipList 与堆](Java/List/SkipList与堆.md)
- [Map](Java/Map/Map.md)
- [ConcurrentHashMap 与线程安全的 Map](Java/Map/ConcurrentHashMap与线程安全Map.md)
- [fail-fast 与 fail-safe](Java/Map/fail-fast和fail-safe.md)
- [同步容器与并发问题](Java/List/同步容器并发问题.md)

#### [并发编程](Java/多线程)

- [多线程基础](Java/多线程/多线程基础.md)
- [线程安全与锁](Java/多线程/线程安全与锁.md)
- [线程池](Java/多线程/线程池.md)
- [ThreadLocal](Java/多线程/ThreadLocal.md)
- [分布式锁](Java/多线程/分布式锁.md)

#### [JVM](Java/JVM)

- [JVM 内存结构](Java/JVM/JVM内存结构.md)
- [垃圾收集与收集器](Java/JVM/垃圾收集与收集器.md)
- [类加载](Java/JVM/类加载.md)
- [JVM 调优](Java/JVM/JVM调优.md)

### 二、MySQL

- [InnoDB 存储引擎、事务与锁](MySQL/InnoDB存储引擎与事务锁.md)
- [索引与查询优化](MySQL/索引与查询优化.md)
- [MySQL 日志（binlog / redo log / undo log）](MySQL/MySQL%20Log.md)
- [表设计与字段变更](MySQL/表设计与字段变更.md)

### 三、Spring

#### [Spring 核心](Spring/Spring)

- [Spring 核心容器与 Bean](Spring/Spring/Spring核心容器与Bean.md)
- [Spring AOP 与注解](Spring/Spring/Spring%20AOP与注解.md)
- [Spring 事务](Spring/Spring/Spring事务.md)
- [Spring 循环依赖](Spring/Spring/Spring循环依赖.md)
- [Spring MVC](Spring/Spring/SpringMVC.md)

#### [SpringBoot](Spring/SpringBoot)

- [SpringBoot](Spring/SpringBoot/SpringBoot.md)

#### [SpringCloud](Spring/SpringCloud)

- [SpringCloud 核心组件](Spring/SpringCloud/SpringCloud组件.md)
- [SpringCloud 与 Dubbo 的区别](Spring/SpringCloud/SpringCloud与dubbo区别.md)

### 四、中间件

#### [Redis](中间件/Redis)

- [Redis 数据类型与底层结构](中间件/Redis/Redis数据类型与底层结构.md)
- [Redis 持久化与高可用](中间件/Redis/Redis持久化与高可用.md)
- [Redis 缓存问题与分布式锁](中间件/Redis/Redis缓存问题与分布式锁.md)

#### [消息队列](中间件/MQ)

- [Kafka](中间件/MQ/Kafka.md)
- [RabbitMQ 与消息队列](中间件/MQ/RabbitMQ与消息队列.md)

#### [Dubbo](中间件/dubbo)

- [Dubbo](中间件/dubbo/Dubbo.md)
- [Java SPI 与 Dubbo SPI](中间件/dubbo/Java%20SPI与Dubbo%20SPI.md)

#### [ZooKeeper](中间件/zookeeper)

- [ZooKeeper](中间件/zookeeper/ZooKeeper.md)

#### [MyBatis](中间件/MyBatis)

- [MyBatis](中间件/MyBatis/Mybatis.md)

### 五、技术方案

- [分布式事务](技术方案/分布式事务.md)
- [限流的算法](技术方案/限流的算法.md)

### 六、计算机网络

- [计算机网络](计算机/网络/计算机网络.md)

---

## 说明

- 笔记中的图示统一存放在 [assets](assets) 目录，按所属笔记分子目录组织，例如 `assets/类加载/img-01.png`。
- 文档使用相对路径引用图片，在 GitHub 网页端、本地编辑器及各类 Markdown 阅读器中均可正常显示。
- 内容持续补充与修订，欢迎交流指正。

## 许可

本项目基于 [LICENSE](LICENSE) 中声明的条款授权。