# Java后端模拟面试错题记录

> 说明：这里会持续记录本轮面试中回答错误或不完整的问题。

## 错题列表

### 1. 请说一下 `HashMap` 和 `ConcurrentHashMap` 的核心区别，至少从线程安全、底层实现、适用场景这三个角度回答。

回答中需要修正的点：

1. `ConcurrentHashMap` 不是“在 HashMap 的基础上实现线程安全机制”这么简单，更准确地说，它是一个单独设计的并发哈希容器，底层实现和并发控制策略都与 `HashMap` 有明显区别。
2. `ConcurrentHashMap` 在 JDK 1.8 中不是“并发量小用 CAS，并发量高就转换成 synchronized”这样的动态切换逻辑。更准确地说，是在不同操作阶段使用不同手段：
   - 桶位为空等简单场景优先使用 CAS。
   - 发生哈希冲突、需要修改链表或红黑树结构时使用 `synchronized`。


#### 本次模拟回答及修正

**你的回答：**
> 通过CAS + synchronized实现线程安全的，在无锁初始化时，桶中没有任何元素，会通过CAS将元素插入到桶中，而当发生哈希冲突时，就会通过synchronized对链表头或红黑树根节点进行加锁，然后在插入元素。ConcurrentHashMap中的size()方法也不是只通过加锁计数来计算，而是通过baseCount和CountCell数组来实现。当通过CAS更新baseCount失败时，就会采用CountCell数组来进行分散计算，最后汇总baseCount和CountCell

**回答评价：** 核心思路正确，CAS + synchronized 的配合逻辑和 size() 优化都答到了。

**需要补充和修正的点：**

1. **缺少 JDK 1.7 和 1.8 的对比**：建议补充 JDK 1.7 采用分段锁（Segment），每段独立继承 ReentrantLock，锁粒度是段级别；JDK 1.8 抛弃了分段锁，锁粒度细化到**单个桶的头节点**，并发能力大幅提升。

2. **为什么用 synchronized 而不是 ReentrantLock**：JDK 1.8 选择 synchronized 的原因——JVM 对 synchronized 有锁升级优化（偏向锁→轻量级锁→重量级锁），且 synchronized 锁的是桶头节点本身，无需额外对象，内存开销更低。

3. **表述小修正**："无锁初始化"建议改为"无竞争插入"更准确；size() 方法的 CountCell 正确拼写是 CounterCell（注意是 Counter 不是 Count）。

### 2. `ArrayList` 和 `LinkedList` 有什么区别？请从底层结构、随机访问、插入删除、内存占用、适用场景这几个方面回答。

回答中需要修正的点：

1. `LinkedList` 不是“不允许随机访问”，更准确地说是“支持按索引访问，但底层需要遍历链表，随机访问效率低，时间复杂度通常为 O(n)”。
2. `LinkedList` 的插入删除不能直接笼统地说一定是 O(1)`。只有在已经定位到目标节点的前提下，插入删除才接近 O(1)；如果先按索引查找节点，查找过程仍然是 O(n)`，整体未必更优。

### 3. 请解释一下 Java 中的“重载”和“重写”有什么区别？再补充说一下它们分别体现的是编译时多态还是运行时多态。

回答中需要修正的点：

1. 重载不能说成“返回值或者参数不同”。Java 中方法重载的判断依据是参数列表不同，包括参数个数、类型或顺序不同；仅返回值不同不能构成重载。
2. 重写时，更准确的表述应该是“运行时根据对象的实际类型决定调用哪个实现方法”，不是笼统说“根据类型”，否则容易和编译期类型混淆。

### 4. 请说一下 `synchronized` 和 `ReentrantLock` 的区别，以及在什么场景下你会优先选择 `ReentrantLock`？

回答中需要修正的点：

1. `synchronized` 不能简单等同于“底层通过操作系统互斥锁实现”。更准确地说，HotSpot 中 `synchronized` 依赖对象头中的监视器机制，JDK 也做了偏向锁、轻量级锁、自旋等多种优化；只有在竞争激烈时才可能膨胀为重量级锁并依赖操作系统互斥量。
2. `ReentrantLock` 的底层基于 AQS，常配合 CAS 实现状态竞争，这一点答得对，但还应补充它支持可中断加锁、可限时获取锁、可设置公平锁、可绑定多个 `Condition`。
3. “使用场景不知道”这一点需要补齐。通常在以下场景优先选择 `ReentrantLock`：
   - 需要可中断锁：`lockInterruptibly()`
   - 需要尝试获取锁或超时获取锁：`tryLock()`
   - 需要公平锁策略
   - 需要多个条件队列进行更细粒度线程唤醒


#### 本次模拟回答及修正

**你的回答：**
> synchronized不需要手动加锁和释放锁，只要进入同步代码块就会加锁，出了同步代码块就会释放锁。而ReentrantLock则是需要手动加锁和释放锁。ReentrantLock可以通过tryLock进行尝试加锁，如果失败了，则直接放弃获取锁，而synchronized如果没获取到锁，线程就会一直阻塞。ReentrantLock可以在构造时实现公平锁的功能，避免线程饥饿

**回答评价：** 核心区别答得不错，手动/自动、tryLock、公平锁这几个点都说到了。
**需要补充和修正的点：**
1. **底层实现层面**：
   - synchronized 是 JVM 层面的内置锁，依赖对象头的 Monitor 机制，经过锁升级优化
   - ReentrantLock 是 JDK 层面（API 层面）的锁，基于 AQS + CAS 实现
1. **ReentrantLock 的更多高级能力**：
   - **可中断获取锁**：lockInterruptibly()，可以在等待锁的过程中响应中断
   - **超时获取锁**：	tryLock(timeout, unit)，超过指定时间还没拿到就放弃
   - **多 Condition 支持**：可以创建多个 Condition 对象，实现精准唤醒（比如生产者唤醒消费者，消费者唤醒生产者），避免 notifyAll 带来的无效唤醒
   - synchronized 只能通过 wait()/notify()/notifyAll() 实现线程通信，且所有等待线程共用一个等待队列
1. **性能方面**：JDK 1.6 之后，synchronized 经过偏向锁、轻量级锁、重量级锁的锁升级优化，性能和 ReentrantLock 基本持平，不要再说 ReentrantLock 性能一定更好。
2. **公平锁补充**：synchronized 只能是非公平锁；ReentrantLock 默认非公平（性能更好），但可以通过构造参数设置为公平锁（先到先得，避免线程饥饿，但吞吐量会下降）。
### 5. 请说一下 JVM 内存区域有哪些？重点讲一下堆、虚拟机栈、方法区分别存什么。
回答中需要修正的点：
1. 线程私有和线程共享划分答反了。
   - 线程私有：程序计数器、虚拟机栈、本地方法栈
   - 线程共享：堆、方法区
2. 虚拟机栈不属于线程共享区，它是线程私有的。
3. 堆主要存储对象实例和数组，这一点答对了。
4. 虚拟机栈中的确存储栈帧，但最好继续展开：栈帧中通常包含局部变量表、操作数栈、动态链接、方法返回地址等。
5. 方法区的表述不够完整。方法区主要存储类元信息、运行时常量池、静态变量、JIT 编译后的代码等，不只是“静态变量、方法链接”。
#### 本次模拟回答及修正

**你的回答：**
> 虚拟机栈，本地方法栈，程序计数器是线程私有的；堆、元空间是线程共享的。每次线程中有方法调用时，就会在栈中创建一个栈帧，栈帧中存储了局部变量表，方法信息，方法链接，出口信息等

**回答评价：** 线程私有/共享划分正确，栈帧的概念也说到了。
**需要补充和修正的点：**
1. **"元空间"表述修正**：建议说"方法区"更准确。元空间（Metaspace）是 JDK 1.8 之后方法区的具体实现（使用本地内存），但方法区是 JVM 规范中的概念。答题时可以说"方法区，JDK 1.8 后用元空间实现"。
2. **方法区存什么**：建议补充完整——方法区主要存储**类元信息、运行时常量池、静态变量、JIT 编译后的代码**等，不只是笼统的"类信息"。
3. **栈帧内容更准确的表述**：
   - **局部变量表**：存储方法参数和局部变量
   - **操作数栈**：执行字节码指令时的操作空间，类似 CPU 寄存器
   - **动态链接**：指向运行时常量池中该方法的引用，用于方法调用
   - **方法返回地址**：方法执行完后返回的位置
1. **堆的补充**：堆是 JVM 中最大的一块内存，主要存储对象实例和数组，也是 GC 管理的主要区域。
### 6. 请讲一下 `==` 和 `equals()` 的区别，再顺带说一下为什么重写 `equals()` 时通常也要重写 `hashCode()`。
回答中需要修正的点：
1. “默认情况下，`==` 和 `equals` 没有区别”这句话不够严谨。更准确地说：
   - 对于引用类型，如果没有重写 `equals()`，那么 `Object.equals()` 默认比较的是引用地址，看起来和 `==` 类似。
   - 但 `==` 本身既可以比较基本类型的值，也可以比较引用类型的地址；`equals()` 只能用于对象比较，语义上更偏向“逻辑相等”。
#### 本次模拟回答及修正
**你的回答：**
> Java中的Set和Map通常是通过equals和hashCode方法来判断是否存在重复的元素，如果只重写了equals方法而没有重写hashCode方法，就会出现错误
**回答评价：** 方向正确，但回答过于简略，没有说清楚原理和后果。
**需要补充和修正的点：**
1. **hashCode 约定**：Java 规范要求——如果两个对象通过 equals() 比较相等，那么它们的 hashCode() 必须返回相同的值。这是 HashMap、HashSet 等集合正常工作的基础。
2. **只重写 equals 不重写 hashCode 的后果**：
   - 两个 equals() 相等的对象可能返回不同的 hashCode
   - 在 HashMap 中，会先通过 hashCode 定位到桶（bucket），再用 equals() 比较
   - 如果 hashCode 不同，两个对象会被放到不同的桶中，导致：
     - HashSet 无法正确去重（可以存入"相等"的对象）
     - HashMap 中用 key 获取不到对应的 value
1. **为什么重写 equals 时必须重写 hashCode**：为了遵守 hashCode 约定，保证基于哈希表的集合类（HashMap、HashSet、Hashtable 等）能正常工作。
2. **补充说明**：反过来，hashCode 相等不代表 equals 一定相等（哈希冲突是正常的），但 equals 相等则 hashCode 必须相等。
### 7. 请说一下 Spring 中 `BeanFactory` 和 `ApplicationContext` 的区别，以及为什么实际开发里更常用 `ApplicationContext`。
回答中需要修正的点：
1. 只答了“`ApplicationContext` 继承自 `BeanFactory`”，缺少关键差异。
2. 更完整的回答应该包括：
   - `BeanFactory` 是 Spring 最基础的 IoC 容器，功能较轻，偏底层。
   - `ApplicationContext` 在 `BeanFactory` 基础上提供了更多企业级能力，例如国际化、事件发布、资源加载、AOP 和自动注册后处理器等支持。
   - `ApplicationContext` 通常默认预实例化单例 Bean，使用体验更完整，更适合实际项目开发。
1. 实际开发更常用 `ApplicationContext`，是因为它功能更丰富、开箱即用，能覆盖大多数 Spring 应用场景。
### 8. 请说一下数据库索引为什么能提升查询效率？再讲一下什么情况下索引反而可能失效。
回答中需要修正的点：
1. “范围查询会导致索引失效”这个说法不够准确。
   - 更准确地说，范围查询通常仍然可能使用索引。
   - 但在联合索引中，一旦某一列使用了范围条件，范围列后面的列往往无法继续充分利用索引。
2. 如果想答得更完整，还可以补充一些常见索引失效场景：
   - 对索引列做函数、运算或隐式类型转换
   - 使用 `!=`、`<>`、`or` 等导致优化器放弃索引的场景
   - 数据分布问题导致优化器判断全表扫描成本更低
### 9. 请说一下事务的四大特性 ACID 分别是什么，并重点讲一下“隔离性”在数据库里是怎么实现的。
回答中需要修正的点：
1. ACID 的基本定义大体正确，但“一致性”可以更准确地表述为：事务执行前后，数据都应满足业务约束和完整性规则。
2. “持久性”不只是“写到硬盘”这么简单，更准确地说，是事务一旦提交，其结果即使在系统故障后也不会丢失，通常依赖 redo log 等机制保障。
3. “隔离性是通过 MVCC 吗”这个回答不完整。更准确地说，数据库隔离性通常是通过多种机制共同实现的：
   - 锁机制：共享锁、排他锁、间隙锁、临键锁等
   - MVCC：主要用于提高读写并发，避免读写冲突
   - 不同事务隔离级别的控制：读未提交、读已提交、可重复读、串行化
4. 如果是 InnoDB，常见面试表述可以总结为：
   - 读操作很多时候依赖 MVCC 实现一致性读
   - 写操作和当前读依赖锁机制保证隔离
   - 在更高隔离级别下结合间隙锁、临键锁减少幻读问题
### 10. 请讲一下 MySQL 里的 `redo log` 和 `binlog` 有什么区别，分别是做什么用的。
回答中需要修正的点：
1. `redo log` 的作用答对了一部分，它主要用于保证事务提交后的持久性和崩溃恢复，是 InnoDB 层的日志。
2. `binlog` 不只是主从复制使用，它还常用于数据恢复、增量备份等场景，是 Server 层的归档日志。
3. 这题更完整的高频区别还应补充：
   - `redo log` 是物理日志，记录页的修改
   - `binlog` 是逻辑日志，记录语句或行变更
   - `redo log` 循环写
   - `binlog` 追加写
   - `redo log` 用于 crash-safe
   - `binlog` 用于主从复制和归档恢复
#### 本次模拟回答及修正
**你的回答：**
> MySQL的redo log是当数据丢失时，需要回复，就会使用redo log。而bin log是二进制格式的日志文件，当需要进行全库的数据迁移，或者首次主从数据同步时，就会使用binlog

**回答评价：** 回答过于简略且不够准确，只说了用途，没有说清本质区别。

**需要补充和修正的点：**
1. **本质区别（高频考点）**：

| 维度 | redo log | binlog |
|------|----------|--------|
| **所属层级** | InnoDB 引擎层 | MySQL Server 层 |
| **日志类型** | 物理日志（记录页的修改） | 逻辑日志（记录 SQL 语句或行变更） |
| **写入方式** | 循环写（固定大小，写满覆盖） | 追加写（不断追加，不会覆盖） |
| **主要作用** | 保证事务持久性、崩溃恢复（crash-safe） | 主从复制、数据恢复、增量备份 |

2. **redo log 更准确的描述**：它是 InnoDB 特有的，用于保证事务提交后的持久性——即使数据库宕机，重启后可以通过 redo log 恢复已提交但还没刷到磁盘的数据，实现 crash-safe。不是"数据丢失时才用"，而是每次事务提交都会写。
3. **binlog 更准确的描述**：它是 Server 层的归档日志，所有引擎都可以用。不只是主从复制，还常用于：
   - 数据恢复（误删后通过 binlog 回滚）
   - 增量备份
   - 数据同步（如同步到数仓）
2. **补充：两阶段提交**：redo log 和 binlog 配合使用时，采用两阶段提交（prepare → commit），保证两者的一致性。
### 11. 请说一下 Redis 为什么快？不要只说“因为它是内存数据库”，尽量从数据结构、单线程模型、IO 模型几个角度展开。
参考总结：
1. Redis 基于内存操作，避免了磁盘随机 IO，这是它高性能的基础。
2. Redis 的核心数据结构做了专门优化，例如：
   - `SDS` 减少字符串操作开销
   - `dict`、跳表、压缩列表、quicklist、intset 等结构针对不同场景优化了时间和空间效率
3. Redis 的命令执行线程长期采用单线程模型，避免了多线程竞争、锁切换和上下文切换带来的开销，所以执行路径很短。
4. Redis 采用 IO 多路复用机制，同时监听大量 socket 连接，把网络 IO 的等待成本降下来，提高并发处理能力。
5. Redis 的大多数操作时间复杂度较低，很多常用命令接近 O(1)，因此单次请求处理非常快。
6. Redis 快并不等于所有命令都快，如果使用了大 key、慢查询命令或复杂聚合操作，性能一样会下降。
### 12. 请说一下 Redis 的持久化方式有哪些？`RDB` 和 `AOF` 的区别是什么？
回答中需要补充的点：
1. Redis 持久化除了单独使用 `RDB` 或 `AOF`，也可以两者配合使用，实际生产里经常混合使用。
2. `RDB` 的描述基本正确，但更准确地说它是“在某个时间点对内存数据做快照”。
3. `AOF` 重写不是“只会记录最后一次写命令”这么简单。更准确地说，AOF 重写会把恢复当前数据集所需的最少命令重新写入新文件，而不是简单保留每个 key 的最后一条命令。
4. 这题高频对比点还应补充：
   - `RDB` 恢复速度通常更快，文件更紧凑
   - `AOF` 数据通常更完整，丢失数据更少
   - `RDB` 更适合做全量备份
   - `AOF` 更适合提高数据恢复精度
#### 本次模拟回答及修正
**你的回答：**
> RDB记录的是二进制格式的日志，记录了当前状态下Redis中所有数据的快照，但是恢复数据更慢。RDB记录命令的是everysec，所以最多只会丢失一秒的数据。AOF日志记录的是每次写命令，并且是以追加的形式记录命令。当需要恢复数据时，就会执行这些命令。但是由于是多次记录写命令，因为占用磁盘空间会比RDB更大。Redis也对其进行了优化，如果对同一个键的多次写入，会被并成一次写命令，只记录最后一次写命令，从而减少了体积。

**回答评价：** 有几处明显错误需要修正，整体思路方向是对的。

**需要补充和修正的点：**
1. **错误修正：RDB 恢复更快，不是更慢！**
   - RDB 是二进制快照文件，恢复时直接加载到内存，**恢复速度很快**
   - AOF 是命令日志，恢复时需要逐条执行命令，**恢复速度更慢**
   - 你说反了！
1. **错误修正：everysec 是 AOF 的配置，不是 RDB 的！**
   - everysec 是 AOF 的 appendfsync 配置选项，表示每秒刷盘一次
   - RDB 是定时快照（比如 save 900 1 表示 900 秒内有 1 次修改就触发快照），不是 everysec
   - 你把 AOF 的配置安到 RDB 上了！
1. **AOF 重写的表述修正**：
   - 不是"只记录最后一次写命令"这么简单
   - 更准确的说法是：AOF 重写会**把恢复当前数据集所需的最少命令**重新写入新文件
   - 比如对同一个 key 写了 100 次，重写后可能只保留最终状态对应的 1 条命令
   - 但不是简单的"每个 key 保留最后一条"，而是根据当前内存数据重新生成最小命令集
1. **RDB 和 AOF 对比总结**：

| 维度 | RDB | AOF |
|------|-----|-----|
| **原理** | 某时刻的内存快照（二进制） | 记录所有写命令（文本/协议格式） |
| **文件大小** | 紧凑，体积小 | 体积较大 |
| **恢复速度** | 快（直接加载） | 慢（逐条执行命令） |
| **数据安全性** | 差（可能丢最后一次快照后的数据） | 好（everysec 配置下最多丢 1 秒） |
| **写入方式** | fork 子进程写 | 追加写 |

2. **生产环境建议**：通常**混合使用**——RDB 做全量备份，AOF 保证数据不丢失，重启时优先用 AOF 恢复。

### 13. 请说一下 Spring Boot 自动配置的原理，至少讲清楚 `@SpringBootApplication`、自动配置类加载、条件装配这几个点。

回答中需要补充或修正的点：
1. `@SpringBootApplication` 不只是开启自动配置，它本质上是组合注解，通常包含：
   - `@SpringBootConfiguration`
   - `@EnableAutoConfiguration`
   - `@ComponentScan`
2. 自动配置类加载的主线需要答完整：
   - `@EnableAutoConfiguration` 会通过 `@Import` 导入自动配置选择器
   - 由选择器去加载候选自动配置类
3. 你提到 `spring.factories` 是经典答案，适用于较多版本；如果面试官追问新版，还可以补充：
   - Spring Boot 2.7+/3.x 逐步使用 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
4. 条件装配不只有 `@ConditionalOnMissingBean`，还常见：
   - `@ConditionalOnClass`
   - `@ConditionalOnBean`
   - `@ConditionalOnProperty`
   - `@ConditionalOnWebApplication`
1. 条件装配的本质是：只有在满足特定环境、类路径、Bean、配置项等条件时，自动配置类或 Bean 才会生效，避免无效装配和配置冲突。
#### 本次模拟回答及修正

**你的回答：**
> SpringBoot启动时，会扫描resources目录下的META-INF/spring.factories文件，读取文件中的所有全限定类名，并通过SPI机制进行加载。

**回答评价：** 说到了 spring.factories 这个关键点，但回答不够完整，缺少核心注解拆解、条件装配等重要内容。

**需要补充和修正的点：**
1. **@SpringBootApplication 是组合注解**：这是自动配置的入口，本质上包含三个核心注解：
   - @SpringBootConfiguration：标记这是一个 Spring Boot 配置类
   - @EnableAutoConfiguration：**开启自动配置的核心注解**
   - @ComponentScan：扫描当前包及其子包下的组件
1. **自动配置加载主线**：
   - @EnableAutoConfiguration 通过 @Import 导入 AutoConfigurationImportSelector（自动配置选择器）
   - 由选择器去加载候选自动配置类
   - 加载来源：经典版本从 META-INF/spring.factories 读取；Spring Boot 2.7+/3.x 逐步改用 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
1. **条件装配（自动配置的"智能"所在）**：自动配置类不是全部生效，而是根据条件选择性装配，常见条件注解：
   - @ConditionalOnClass：类路径下存在某个类才生效
   - @ConditionalOnMissingBean：容器中没有某个 Bean 才生效（允许用户自定义覆盖）
   - @ConditionalOnProperty：配置文件中有特定属性才生效
   - @ConditionalOnWebApplication：是 Web 应用才生效
1. **一句话总结**：自动配置 = 注解触发 + 加载候选配置类 + 条件过滤，最终只加载符合当前环境需要的配置。
### 14. 请说一下 Java 线程池的核心参数有哪些？`corePoolSize`、`maximumPoolSize`、`workQueue`、拒绝策略分别是做什么的？
回答中需要补充或修正的点：
1. `ThreadPoolExecutor` 的核心参数通常包括：
   - `corePoolSize`
   - `maximumPoolSize`
   - `keepAliveTime`
   - `unit`
   - `workQueue`
   - `threadFactory`
   - `handler`
   你漏掉了 `keepAliveTime` 和 `unit`。
2. `corePoolSize` 的表述不够准确。线程池创建后不会立刻自动创建所有核心线程，通常是“有任务提交时才创建核心线程”，除非显式调用了预启动核心线程的方法。
3. `maximumPoolSize` 不是“核心线程数 + 辅助线程数”这种官方表述。更准确地说，它是线程池允许创建的最大线程数上限。
4. 非核心线程也不是“用完即销毁”，而是当空闲时间超过 `keepAliveTime` 后才会被回收。
5. 拒绝策略部分可以再完整一些，常见的有：
   - `AbortPolicy`
   - `CallerRunsPolicy`
   - `DiscardPolicy`
   - `DiscardOldestPolicy`
#### 本次模拟回答及修正

**你的回答：**
> 核心参数有：核心线程数，最大线程数，非核心线程的存活时间，时间单位，线程工厂，任务队列，拒绝策略。当核心线程满了之后，新任务会先加入到任务队列中，如果任务队列也满了，就会创建非核心线程来处理任务队列中的任务，如果非核心线程也达到最大了，还有新的任务加入，就会触发拒绝策略

**回答评价：** 七大核心参数都列出来了，整体流程也基本正确。

**需要补充和修正的点：**
1. **流程表述修正**："创建非核心线程来处理任务队列中的任务"这句话不准确。正确的是——队列满了之后，会创建非核心线程来处理**新提交的任务**，而不是队列中的任务；队列中的任务仍然由核心线程和非核心线程空闲后从队列中取出来执行。
2. **核心线程创建时机**：线程池创建后**不会立即创建所有核心线程**，而是"有任务提交时才创建核心线程"（懒加载），除非显式调用 prestartCoreThread() 或 prestartAllCoreThreads() 预启动。
3. **任务提交流程完整顺序**：
   1. 线程数 < corePoolSize → 创建核心线程执行任务
   2. 线程数 >= corePoolSize → 尝试将任务加入阻塞队列（offer(task)）
   3. 队列也满了 → 创建非核心线程执行任务
   4. 线程数达到 maximumPoolSize → 执行拒绝策略
5. **特殊队列的情况**：如果使用 SynchronousQueue（同步队列，容量为 0），offer 永远失败，所以核心线程满了之后会直接创建非核心线程，不会排队。
6. **非核心线程回收**：非核心线程不是"用完即销毁"，而是空闲时间超过 keepAliveTime 后才会被回收。
7. **常见拒绝策略**：
   - AbortPolicy：直接抛异常（默认）
   - CallerRunsPolicy：由提交任务的线程自己执行
   - DiscardPolicy：直接丢弃任务
   - DiscardOldestPolicy：丢弃队列中最老的任务，然后重试
### 15.自己实现的mybatisplus 插件
MybatisPlus中可以通过两种方式实现自定义插件
1. 实现MybatisPlus的InnerInterceptor接口，自定义接口时只需要实现该接口，并将其添加到MybatisPlusInterceptor中即可，实现方式轻量，且与MP内置插件的协同性更好
2. 实现Mybatis的Interceptor接口，这是Mybatis底层的拦截器接口，在MP中也可以使用，但是它需要处理更底层的代理逻辑，并对拦截点的指定要求更细致
**下面是一个自定义的sql语句性能告警器**
```java
@Intercepts({  
        @Signature(type = StatementHandler.class,method = "query",args = {Statement.class, ResultHandler.class}),  
        @Signature(type = StatementHandler.class,method = "update",args = {Statement.class}),  
        @Signature(type = StatementHandler.class,method = "batch",args = {Statement.class})  
})  
@Slf4j  
public class PerformanceMonitorInterceptor implements Interceptor {  
  
    private long slowSqlThreshold = 1;  
    private boolean showParams = true;  
  
    @Override  
    public void setProperties(Properties properties) {  
        String threshold = properties.getProperty("slowSqlThreshold");  
        if (threshold != null) {  
            this.slowSqlThreshold = Long.parseLong(threshold);  
        }  
        String showParamsProp = properties.getProperty("showParams");  
        if (showParamsProp != null) {  
            this.showParams = Boolean.parseBoolean(showParamsProp);  
        }  
        log.info("性能监控拦截器初始化:慢查询阈值={}s，显示参数={}",slowSqlThreshold,showParams);  
    }  
  
    @Override  
    public Object plugin(Object target) {  
        // 利用Mybatis的Plugin工具生成代理对象  
        return Plugin.wrap(target, this);  
    }  
  
    @Override  
    public Object intercept(Invocation invocation) throws Throwable {  
        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();  
        BoundSql boundSql = statementHandler.getBoundSql();  
        // 获取原始sql语句  
        String originalSql = boundSql.getSql();  
        // 获取参数对象  
        Object parameterObject = boundSql.getParameterObject();  
        long start = Instant.now().toEpochMilli();  
        // 执行sql语句  
        Object proceed = invocation.proceed();  
        // 计算执行耗时  
        long executionTime = Instant.now().toEpochMilli() - start;  
        String fullSql = showParams ? buildFullSql(boundSql,parameterObject):originalSql;  
        String formattedSql = formatSql(fullSql);  
        // 大于阈值，则告警  
        if (executionTime > slowSqlThreshold * 1000) {  
            log.warn("【慢SQL告警】执行耗时：{}ms({}s)，阈值：{}s,SQL:{}",  
                    executionTime,executionTime/1000.0,slowSqlThreshold,formattedSql);  
        } else {  
            log.debug("【SQL执行】耗时：{}ms,SQL:{}",executionTime,formattedSql);  
        }  
        return proceed;  
    }  
  
    private String buildFullSql(BoundSql boundSql, Object parameterObject) {  
        String sql = boundSql.getSql();  
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();  
        if (parameterMappings == null || parameterMappings.isEmpty()) {  
            return sql;  
        }  
        // 获取参数值  
        MetaObject metaObject = MetaObject.forObject(  
                parameterObject,  
                SystemMetaObject.DEFAULT_OBJECT_FACTORY,  
                SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY,  
                new DefaultReflectorFactory()  
        );  
        StringBuilder sb = new StringBuilder(sql);  
        int paramIndex = 0;  
        int questionMarkIndex = sb.indexOf("?");  
        while(questionMarkIndex > 0 && paramIndex < parameterMappings.size()) {  
            ParameterMapping mapping = parameterMappings.get(paramIndex);  
            String property = mapping.getProperty();  
            Object value;  
            if (metaObject.hasGetter(property)) {  
                value = metaObject.getValue(property);  
            } else {  
                value = parameterObject;  
            }  
            // 格式化参数值  
            String paramValue = formatParameterValue(value);  
            // 将?占位符替换为实际的参数值  
            sb.replace(questionMarkIndex,questionMarkIndex + 1,paramValue);  
            // 查找下一个?占位符  
            questionMarkIndex = sb.indexOf("?",questionMarkIndex + paramValue.length());  
            paramIndex++;  
        }  
        return sb.toString();  
    }  
  
    private String formatParameterValue(Object value) {  
        if (value == null) {  
            return "NULL";  
        }  
        if (value instanceof String str) {  
            // 转义单引号  
            str = str.replace("'","''");  
            return "'" + str + "'";  
        }  
        if (value instanceof Date) {  
            return "'" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format((Date) value) + "'";  
        }  
        if (value instanceof LocalDateTime dateTime) {  
            return "'" + dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")) + "'";  
        }  
        if (value instanceof Boolean) {  
            return (Boolean) value ? "1" : "0";  
        }  
        return value.toString();  
    }  
  
    private String formatSql(String sql) {  
        // 取出多余的空白  
        sql = sql.replaceAll("\\s+", " ");  
        // 关键SQL关键字换行  
        sql = sql.replaceAll("(?i)select ", "\nSELECT ");  
        sql = sql.replaceAll("(?i)from ", "\n  FROM ");  
        sql = sql.replaceAll("(?i)where ", "\n  WHERE ");  
        sql = sql.replaceAll("(?i)join ", "\n  JOIN ");  
        sql = sql.replaceAll("(?i)left join ", "\n  LEFT JOIN ");  
        sql = sql.replaceAll("(?i)right join ", "\n  RIGHT JOIN ");  
        sql = sql.replaceAll("(?i)inner join ", "\n  INNER JOIN ");  
        sql = sql.replaceAll("(?i)order by ", "\n  ORDER BY ");  
        sql = sql.replaceAll("(?i)group by ", "\n  GROUP BY ");  
        sql = sql.replaceAll("(?i)having ", "\n  HAVING ");  
        sql = sql.replaceAll("(?i)limit ", "\n  LIMIT ");  
        sql = sql.replaceAll("(?i)union ", "\nUNION ");  
        return sql.trim();  
    }  
}
@Bean  
public PerformanceMonitorInterceptor performanceMonitorInterceptor() {  
    PerformanceMonitorInterceptor interceptor = new PerformanceMonitorInterceptor();  
    Properties prop = new Properties();  
    prop.setProperty("slowSqlThreshold","2");  
    prop.setProperty("showParams","true");  
    interceptor.setProperties(prop);  
    return interceptor;  
}
```
- `@Intercepts`注解用于标记这个类是一个Mybatis拦截器，其属性是一个`@Signature`数组，可以同时拦截多个方法
- `@Signature`注解用于定义要拦截的一个具体方法，有三个属性
	type用于说明要拦截的类，Mybatis中允许拦截的核心类有四个，分别是`Executor`，SQL执行器，负责整体执行流程；`StatementHandler`，处理JDBC Statement的创建和执行；`ParameterHandler`，处理参数设置；`ResultHandler`，处理结果集映射
	method用于说明要拦截的方法名
	args用于说明方法的参数类型列表
### 16.WebSocket改用微服务如何实现，如何识别WebSocket连接的用户id，session中如何获取到用户id
### 17.Spring容器是什么数据结构
Spring容器分为两层，第一层是BeanDefinition的注册表，告诉Spring这个Bean该如何被创建，其数据结构是`Map<String,BeanDefinition> beanDefinitionMap`，这是一个ConcurrentHashMap，通常容量设置为256，确保高并发下的线程安全，key是Bean的名称，value是BeanDefinition对抗
第二层是单例对象的缓存，当Bean被实例化、组装好之后，为了能够重复使用，就需要将其存储起来。该容器是用于缓存实例对象的，其数据结构是`Map<String,OBject>singletonObjects`，key是Bean的名称，value就是已经创建好的、可以直接使用的Bean实例对象
### 18.进程之间如何通信、线程之间如何通信
进程之间有独立的虚拟空间，互不干扰，它们之间的通信需要通过内核或借助特殊的内存区域
1. 管道：父子进程之间通信，数据单向流动。本质是内核里的环形缓冲区
2. 命名管道：允许无亲缘关系的进程通信，通过文件系统的特殊文件进行数据交换
3. 消息队列：内核维护一个消息链表，进程往里写消息，其他进程按类型读取。优点是可以多条消息异步收发，但是数据拷贝开销较大
4. 共享内存：将同一款物理内存映射到多个进程的虚拟地址空间，进程可以直接读写，无需内核拷贝。但是本身不提供同步机制，需要配合信号量或互斥锁来防止数据竞争
5. 套接字：最通用的方式，既可用于同一台机器，也可用于跨网络。数据传输稳定，适合复杂通信
同一进程下的线程共享堆内存、全局变量和静态变量，通信的核心在于如何正确地读写共享数据
- 直接共享变量：线程A修改全局变量count，线程B直接读取，无需向进程一样申请特殊机制。但是需要配合同步机制，否则可能会导致脏读
- 线程之间的共享数据，需要同步机制来防止数据读取出错
### 19.针对微服务改造，用Redis共享数据，以WebSocket长连接无法序列化并跨进程发消息
### 20.（username,is_deleted,delete_time）未删除的数据，删除时间该如何设置，设置为null时是否会参与索引
NULL值会被计入索引记录中，在索引的内部存储中，NULL会被视为最小值。InnoDB对NULL值的存储有优化，NULL值只占1个bit，不会占用实际数据的存储空间
### 21. Redis为什么会比其他缓存更快
1. Redis的数据全都存储在内存层面，读写速度极快。同时Redis中实现了SDS、跳表、压缩列表等特殊编码。读取时CPU缓存命中率极高，减少了内存碎片
2. 单线程+非阻塞IO。Redis使用单线程处理网络请求，天然无锁，省去了线程切换带来的CPU开销。Redis使用epoll模型，让一个线程可以同时监听成百上千个socket连接。只有当连接有数据到达时，才会交给Redis处理，否则线程就阻塞等待。使得Redis能把性能集中在处理逻辑上，而不是消耗在等待网络数据上
3. 持久化策略：Redis把写磁盘做成了后台的异步任务。RDB快照，通过fork子进程来写磁盘，主进程继续处理命令，写时复制保证了主进程不受影响。AOF追加，主线程只是把数据写到操作系统的PageCache中就立即返回了。真正的磁盘IO交给了操作系统后台异步处理
### 22.Caffeine缓存为什么会快于Redis缓存
- Caffeine是引用进程的一部分，获取缓存数据时就是一次本地内存寻址。数据在JVM的堆内存中，不需要网络通信。而Redis是独立的进程，通常需要网络IO开销
- Caffeine存储的就是对象本体的引用，存入什么类型，取出就是什么类型，完全不需要编解码，直接操作内存中的对象字段。Redis存入时必须将对象序列化为字节数组，读取时再反序列化回对象。序列化和反序列化的过程极其消耗CPU，在大Value场景下，开销可能超过网络传输
### 23.消息积压该如何处理，快速处理和长期处理的方案
**快速处理**
- **流量管控与降级**：首先停止生产者的写入，或者切换降级逻辑（同步写DB、本地缓存），然后开启生产者限流，将生产速率限制在消费端可承载的范围内；熔断非核心请求，保障核心消息的生产和消费。暂停重试队列的消费投递，避免消费失败的消息无限充实，加重压力
- **消费能力扩容**：水平扩容，优先扩容消费者实例数，上限为Topic的队列/分区数（一个队列同一时间只能被一个消费者线程消费，超过队列数的扩容无效）。垂直扩容，紧急调大消费者线程核心/最大线程数，缩小等待队列长度，降低单次批量消息数量，提升单实例消费并行度。逻辑瘦身，紧急关闭消费逻辑中的非核心操作，最小化单条消息处理耗时
### 24. Linux常用的指令
`ls` 列出目录内容 `ls -la`列出目录所有文件，包括隐藏文件
`cd` 切换目录
`pwd` 显示当前所在路径
`mkdir` 创建文件夹 `mkdir -p a/b/c` 递归创建多级目录
`rm` 删除文件 `rm -rf` 强制递归删除文件夹
`cp` 复制文件或目录 `cp -r src target` 复制文件夹加-r
`mv` 移动或重命名
`touch` 创建空文件或更新时间戳
`cat` 查看完整文件内容
`ps` 查看进程快照 `ps -ef` 查看所有进程
### 25. 多个进程之间能否使用同一个端口，如果是不同协议，能否使用同一个端口
相同协议的情况下，多个进程不能使用同一个端口，但是如果是不同协议，则可以使用同一个端口。例如一个TCP进程和一个UDP进程可以绑定并使用同一个端口号，不会产生冲突
操作系统内核中的TCP和UDP是两个完全独立的软件模块。当数据包到达时，系统会先根据IP包头中的协议号来判断数据包属于哪个协议，然后再根据端口号，把数据交给对应协议下监听的进程
### 26. TCP如何保证可靠传输，TCP三次握手和四次挥手
TCP通过以下核心机制，确保可靠传输
- **校验和**：每个TCP报文段都包含校验和，用于监测数据在传输过程中是否发生比特错误，。如果接收方计算出的校验和不匹配，就会丢弃该报文
- **序列号(Seq)和确认应答(ACK)**：TCP将数据流分割成多个报文段，并为每个字节分配唯一的序列号，接收方接收到数据后，会发送确认号，表示已收到该序列号之前的所有数据。如果发送方在一定时间内没有收到ACK，就会出发重传
- **超时重传**：发送方在发送数据后会启动一个定时器。如果定时器超时前未收到ACK，就会重新发送该报文段。RTO（超时重传时间）会根据网络状况动态调整
**TCP三次握手**
- 第一次，客户端先发送一个SYN报文，携带客户端的初始序列号seq=x。客户端进入`SYN-SENT`状态
- 第二次，服务端回复一个ACK报文， 并发送一个SYN报文给客户端，携带服务端的初始序列号seq=y，确认号ack=x+1，服务端进入`SYN-RCVD`状态
- 第三次，客户端回复一个ACK报文，序列号seq=x+1,确认号ack=y+1，客户端进入`ESTABLISHED`状态，服务端收到ACK报文后，也进入`ESTABLISHED`状态
**为什么不是两次握手**
因为两次握手无法让服务端确认自己发送的SYN报文是否被客户端正确收到，也无法防止已失效的连接请求报文段突然又传到服务端，从而产生错误连接
**TCP四次挥手**
- 第一次，客户端发送FIN报文，序列号seq=u，表示不再发送数据，客户端进入`FIN_WAIT_1`状态
- 第二次，服务端回复一个ACK报文，确认号ack=u+1，服务器进入`CLOSE_WAIT`状态，此时TCP连接处于半关闭状态：客户端不再发送数据，但是服务端还可以继续发送数据
- 第三次，服务端也准备好关闭时，发送FIN报文，序列号seq=w，服务端进入`LAST_ACK`状态
- 第四次，客户端回复一个ACK报文，确认号ack=w+1，客户端进入`TIME_WAIT`状态，等待2MSL（最长报文段寿命）后关闭，服务端收到ACK后进入`CLOSED`状态
**为什么不是三次挥手**
因为TCP是全双工的，发送FIN只表示不再发送数据，但是接收功能依然可以正常工作，被动关闭方（服务端）可能还有数据要发送，所以它的ACK和FIN不能合并，只能分两次发送
**客户端为什么要等待2MSL**
确保最后一个ACK报文能到达服务器，如果服务器没收到ACK，会重发FIN，客户端可以重发ACK
让本连接中所有的旧报文段都在网络中消失，避免干扰后续使用相同端口的新连接
### 27. 网络层有哪些协议
网络层的核心协议有
- `IP协议`：定义了IP地址，并为数据包提供无连接、不可靠的传输服务。主要任务是寻址和路由，决定数据包该发往何处
- `ICMP协议`：用于在IP网络中传递控制消息和错误报告，不传输用户数据，而是用于诊断网络问题
- `ARP协议`：将IP地址解析为MAC地址，因为数据最终需要在物理网络上传输
- `IGMP协议`：用于管理IP多播组成员。它让主机可以向路由器报告希望加入或离开某个多播组，是实现网络视频会议，在线直播等一对多通信的基础
**OSI模型**
- **物理层**：定义网络硬件的电器、机械和物理特性。网线、网卡、接口标准等
- **数据链路层**：在相邻网络节点之间传输数据帧，负责物理寻址和差错控制
- **网络层**：负责数据包从源到目的地的路由选择和转发，是网络层协议工作的核心位置
- **传输层**：提供端到端的可靠或不可靠数据传输服务，并负责流量控制和差错校验，核心协议是TCP和UDP
- **会话层**：管理和维护两个通信节点之间的会话连接、负责建立、管理和终止通信
- **表示层**：负责数据格式的转换、加密和解密，确保一个系统发送的数据能被另一个系统识别
- **应用层**：直接为用户的应用程序提供网络服务，是用户和网络交互的窗口
### 28. 线程池核心参数，核心线程消耗完了该如何处理，任务队列
核心线程消耗完之后，会将任务放到阻塞队列中，当阻塞队列满了之后，线程池才会继续创建非核心线程来处理新的任务。如果线程池中的线程数达到了最大线程数，还有任务提交，就会出发线程池的拒绝策略
线程池的核心参数有以下几个，`核心线程数`，`最大线程数（核心线程数+非核心线程数）`，`阻塞队列`，`拒绝策略`，`线程工厂（用于给线程池命名）`，`非核心线程数存活时间`，`时间单位`
JDK源码中，任务入队和创建非核心线程的顺序并非绝对固定，而是有一个条件分支
- 如果核心线程满了，会先尝试入队`offer(task)`
- 如果入队失败，才会去创建非核心线程
如果队列是`SynchronousQueue`（同步队列，大小为0），offer永远失败，所以只要核心线程满了，新任务就会立即去尝试创建非核心线程
### 29. 流式输出除了SSE还有什么
SSE实现的流式输出，仅服务器向客户端推送数据，特点是轻量、基于HTTP协议，支持自动重连，只能传文本
WebSocket也可以实现流式输出，WebSocket是全双工通信，服务器与客户端可以互相发送消息。特点是持久连接、低延迟、支持二进制数据，需要手动实现重连
长轮询，客户端发送请求，服务端等待有数据时在返回响应。特点是基于HTTP协议，兼容性好、但是实时性较差，由于需要一直保持连接，所以比较浪费资源
### 30. Mybatis是如何执行sql语句的，xml文件是如何转换成方法的
Mybatis启动时，会读取并解析所有的XML映射文件，为后面的SQL执行做好准备
- 构建SqlSessionFactory，Mybatis通过`SqlSessionFactoryBuilder`加载主配置文件和所有mapper文件，解析后生成一个全局唯一的`SqlSessionFactory`对象，这个工厂对象就是整个应用与数据库的连接池管理器
- 解析Mapper文件，在解析过程中，每个`<select>`，`<insert>`等SQL标签都会被解析成一个`MappedStatement`对象，这个对象封装了一条SQL的所有信息，解析完后，所有的`MappedStatement`对象都会被缓存到`Configuration`对象中，这是Mybatis的知识库，包含了所有的配置信息
- 生成代理对象，先获取Mapper代理对象，通过sqlSession.getMapper方法获取Mapper的代理对象，Mybatis会使用JDK动态代理为Mapper接口生成一个代理对象
- 调用代理对象，当调用代理对象的方法时，会被它的`MapperProxy`拦截，它会根据方法名，从之前解析好的信息中找到对应的`MappedStatement`对象，然后决定是执行查询、更新还是其他操作
- 执行SQL，`MapperProxy`最终会委托给`SqlSession`的对应方法，而`SqlSession`又会将工作交给`Executor`。`Executor`是Mybatis的调度中心，负责管理缓存并协调整个SQL的执行过程
### 31. MybatisPlus如何实现分页插件的
分页插件通过拦截器在SQL执行前接入，自动生成并执行一条COUNT来查询获取总条数，在修改原SQL拼接分页语句，动态地拼接数据库方言对应的分页语句
### 32. MySQL里SELECT的执行顺序
1. `FROM`，确定数据来源，包括JOIN关联的表
2. `ON`，在JOIN连接时，执行ON条件，过滤掉不符合连接条件的行
3. `JOIN`，执行外连接时，会补上被丢弃的NULL行，生成新的临时表，如果是内连接，通常和ON合并处理
4. `WHERE`，对临时表进行行级过滤，这里不能使用SELECT定义的别名，也不能使用聚合函数
5. `GROUP BY`，将数据按照指定字段分组
6. `HAVING`，对分组后的结果进行过滤，这里不能使用SELECT中定义的别名
7. `SELECT`，提取指定的列，并计算表达式，列的别名被正式定义
8. `DISTINCT`，使用了DISTINCT，会在SELECT投影之后，去除结果集中重复行
9. `UNION`，如果有集合操作，将多个SELECT结果合并
10. `ORDER BY`，对最终结果进行排序
11. `LIMIT/OFFSET`，截取指定数量的数据行返回给用户
### 33. synchronized和ReentrantLock的区别
**synchronized**是JVM层面内置的锁，底层通过监视器实现。它会自动加锁和释放锁，在进入同步代码块时会自动加锁，出了同步代码块后就会释放锁
**ReentrantLock**是Java并发包下提供的互斥锁，底层是基于AQS和CAS操作实现。它必须要手动加锁，配合`lock()`和`unlock()`函数一起使用，ReentrantLock提供了许多高级的能力，例如`tryLock()`尝试获取锁，如果锁被占用了则可以直接返回false，并且tryLock可以设置超时时间。
ReentrantLock可以在构造时指定`fair=true`，让锁变成公平锁（先到先得），避免线程饥饿的情况，而synchronized是非公平锁，性能更高，但是可以导致某些线程长期获取不到锁
jdk1.6之后，JVM通过锁升级对synchronized进行了大量的优化，使得两者性能基本持平
ReentrantLock可以创建多个Condition对象，实现精准唤醒，可以避免掉不必要的线程唤醒开销。synchronized只能通过`wait()`、`notify()`方法实现线程通信，且所有等待线程共用一个等待队列
### 34. 锁升级
锁升级是jdk1.6之后引入的对synchronized锁进行性能优化的一个特性，锁升级按照以下步骤执行，并且锁升级过程是不可逆的
- **无锁**：对象刚被创建时，还没有任何线程访问同步代码块，此时对象头里的锁标志位是`01`
- **偏向锁**：当有一个线程进入到同步代码块时，锁就会升级成为偏向锁。这一个线程可以反复进入同步代码块。因为当线程A第一次获取锁时，JVM会利用CAS操作，把线程A的ID记录在对象的`Mark Word`（对象头）里，接下来只需要比较对象头里的ID是不是自己，如果是就直接进入同步块
- **轻量级锁**：当有其他线程尝试获取锁时，偏向模式结束。JVM会等待线程A执行到安全点，暂停A并检查A是否活着。如果A退出同步块，则撤销偏向锁，恢复为无锁状态，再让B重新走流程。如果A仍在同步块中，则升级为轻量级锁。轻量级锁是两个线程交替执行同步块（A用完释放，B再来使用）。JVM会在当前线程的栈帧中建立一个名为`Lock Record`的空间，用于存储对象头的拷贝。然后通过CAS自旋尝试把对象头指向该Lock Record。CAS成功则线程拿到轻量级锁。CAS失败，则说明锁被其他线程占用了，此时线程不会阻塞，而是继续做CAS尝试。在原地循环重试几十次
- **重量级锁**：当有两个或多个线程同时争抢锁，且自旋次数超过阈值，或者一个线程持有锁的时间过长。就会升级为重量级锁。JVM会停止自旋，此时锁的Mark Word会指向操作系统的Monitor对象，拿不到锁的线程被阻塞，进入等待队列。后续线程来了也会直接阻塞挂起。线程的唤醒和挂起都需要CPU从用户态切换到内核态，所以开销很大
#### 本次模拟回答及修正

**你的回答：**
> 首先是无锁，没有任何线程进入，当只有一个线程进入代码块时，就会升级成偏向锁的状态。当有两个线程同时进入，就会升级成轻量级锁，未获取到锁的线程就会自旋等待，而不是阻塞。因为线程从运行切换到阻塞需要从用户态切换到内核态，更消耗性能。当有多个线程同时进入或者自旋的时间过长，就会升级成重量级锁，所有未获取到锁的线程都会阻塞

**回答评价：** 整体流程和思路正确，四个阶段都说到了，自旋的原因也解释了。
**需要补充和修正的点：**
1. **偏向锁细节补充**：
   - 第一个线程进入同步块时，JVM 用 CAS 把线程 ID 记录到对象头的 Mark Word 中
   - 之后这个线程再次进入时，只需要比较 Mark Word 中的线程 ID 是不是自己，如果是就直接进入，无需 CAS
   - 偏向锁的思想是：如果一直只有一个线程用锁，那就连 CAS 都省了
1. **轻量级锁细节补充**：
   - 偏向模式结束后，JVM 会等待持有偏向锁的线程到达安全点，检查线程状态
   - 如果线程已经退出同步块 → 撤销偏向锁，恢复无锁
   - 如果线程还在同步块中 → 升级为轻量级锁
   - 轻量级锁的获取：JVM 在当前线程栈帧中创建 Lock Record，拷贝对象头 Mark Word 到 Lock Record，然后用 CAS 尝试把对象头指向 Lock Record
   - 轻量级锁的场景是**两个线程交替执行**同步块，不是同时竞争
1. **重量级锁细节补充**：
   - 自旋超过一定次数（或自旋过程中又有线程来竞争）→ 升级为重量级锁
   - 升级后，Mark Word 会指向操作系统的 Monitor 对象
   - 拿不到锁的线程会被真正阻塞挂起，进入等待队列
   - 线程阻塞和唤醒需要用户态→内核态切换，开销大
1. **重要结论**：锁升级是**不可逆**的，只能从低到高升级，不能降级。
### 35. Spring Bean的生命周期
1. **实例化**：通过构造器或工厂方法创建出Bean的实例对象
2. **属性赋值**：Spring根据`@Autowired`注解或XML配置，完成依赖注入，把当前Bean依赖的其他Bean全部注入进来
3. **初始化**：执行各种初始化回调，例如`@PostContructor`、`InitializingBean`接口等，此时Bean才真正可用
4. **销毁**：容器关闭时，执行销毁回调`@PreDestroy`、`Disposable`接口
### 36. Spring事务的传播机制和隔离级别
事务传播机制解决的是业务方法之间相互调用时，事务该如何传递的问题
隔离级别解决的是事务并发时，数据读一致性的问题
**事务传播机制**
- **REQUIRED（默认）**：如果当前存在事务，则加入；如果没有，则新建一个事务。业务场景中最常用的传播机制，比如用户下单扣减库存，必须在同一个事务里，要么全部成功，要么全部回滚
- **REQUIRES_NEW（新建）**：总是新建一个事务。如果当前存在事务，则把当前事务挂起，等待新事务执行完后再回复。例如用户下单失败，但是失败日志必须写入成功，不能因为主业务失败导致日志回滚丢失
- **NESTED（嵌套）**：如果当前存在事务，则在嵌套事务内执行；如果没有，则新建一个。比如批量导入100条数据，希望第50条失败时只回滚第50条，前面49条提交。NESTED是外层事务的子事务，外层回滚，内层也回滚。而REQUIRES_NEW是完全独立的，外层回滚不影响内存
- **SUPPORTS（支持）**：如果当前有事务则加入，没有则以非事务的方式执行。适用于只读查询，有无事务影响不大，但加入事务可以享用连接池的复用
- **NOT_SUPPORTS（不支持）**：以非事务的方式执行，如果当前有事务则挂起。例如发送短信/邮件等通知，不应该因为数据库事务失败而导致通知发送被回滚。但是挂起事务会增加连接占用时间
- **MANDATORY（强制）**：必须存在事务，否则抛异常。强制要求调用方必须开启事务的方法。比如金融系统中的核心扣款子方法
- **NEVER（绝不）**：必须不存在事务，否则抛异常。比如某些制度缓存刷新操作，在事务中执行可能导致锁竞争
**事务隔离级别**
事务隔离级别解决的是**脏读**、**幻读**、**不可重复读**三大并发问题
- **ISOLATION_DEFAULT**：是否导致脏读、幻读、不可重复读，全都由数据库决定
- **READ_UNCOMMITTED**：读未提交，会读到事务未提交的数据。三种问题都可能会出现
- **READ_COMMITTED**：读已提交，只能读到事务已经提交的数据。不会导致脏读，但是可能会导致幻读和不可重复读
- **REPEATABLE_READ**：可重复读，事务开始时会创建一个快照，整个事务执行期间都只读这个快照里的数据。不会导致脏读和幻读，但是可能会导致不可重复读。是MySQL默认的事务隔离级别
- **SERIALIZABLE**：串行化，所有事务串行执行，解决了所有事务并发问题。但是性能较差
### 37.  RDB和AOF的区别
**RDB**：RDB是快照式的持久化，是将Redis某一时刻的数据以二进制形式全量存储到磁盘中。通过fork子进程来进行持久化，不会影响Redis主进程处理命令。优点是占用空间小，恢复的速度快。但是因为是定时快照，如果Redis意外宕机，可能会丢失最后一次快照之前的所有数据。并且fork子进程的时候，内存会临时翻倍，导致主进程短暂卡顿
**AOF**：AOF是以日志形式记录每一次写操作命令，并以追加的方式写入文件。Redis重启时，会重新执行这些命令来重建数据。写日志有三种频率：`always`，每次写命令都同步刷盘，最安全，但是性能最差；`everysec`：每秒同步一次，最多丢失1秒的数据，兼顾性能和安全性；`no`：交由操作系统决定什么时候该进行刷盘，性能最佳但是不可控。AOF是纯文本命令格式，即使数据损坏了，也方便人工编辑修复。但是AOF记录了所有的写命令，会导致比RDB文件大很多，而且重启时也需要依次执行命令，所以大数据量的场景下恢复速度较慢。**为了解决AOF日志文件过大的问题，Redis会进行AOF重写，即定期在后台将内存中的最新数据转化为写命令写入AOF，假设前面对同一个键写了100次，只会保留最后一次**
### 38. 如何保证数据一致性
在分布式环境下，无法保证决定意义上的强一致性，只能保证最终一致性
最常用的就是**旁路缓存**策略，对于读请求，先查Redis，命中则返回；没命中则查MySQL，然后将查询到的结果写入到Redis中并返回；对于写请求，先更新MySQL，成功之后删除Redis当中的旧缓存数据。
**为什么是删除缓存而不是更新缓存**
因为更新缓存是写操作，如果频繁写数据就会导致Redis资源浪费；且如果线程A更新了MySQL还没来得及写缓存，线程B读到了旧数据并写入缓存，就会产生脏数据。删除缓存让下一次读请求去拉取最新的数据，更加稳妥
对于写请求，一定要**先更新数据库，再删缓存**
如果是先删除缓存，再更新数据库，高并发环境下可能会导致数据不一致，因为更新数据库是一个耗时的操作，如果数据库没更新完，就有另一个线程来读数据，并且从MySQL中读到了旧数据，并将旧数据写回缓存，这时候数据库才更新完，就会导致数据不一致
而先更新数据库，再删缓存，删缓存是一个快速的操作。假设数据库还没更新，就有其他线程来读，此时读到的也是旧数据，数据会出现短暂的不一致性（但这是可接受范围内的）。更新完数据库后，再删除缓存，删除缓存较快，通常不会在这个短暂的时间有其他线程来读数据

#### 本次模拟回答及修正

**你的回答：**
> 当读数据时，如果Redis中数据过期了，就去MySQL中读取，然后写回Redis中，如果没过期，就直接返回数据。写数据时，采用的是先更新数据库，再删除缓存。因为如果采用先删除缓存，再更新数据库，可能在更新数据库前，有另一个线程来从MySQL中读到了旧数据，然后写回到Redis中，就会导致数据不一致。因为更新是一个比较耗时的操作，而删缓存耗时更短

**回答评价：** 核心思路正确，旁路缓存模式和先更库再删缓存的原因都说到了。

**需要补充和修正的点：**
1. **模式名称**：这是经典的 **Cache Aside Pattern（旁路缓存模式）**，可以先说模式名称再展开，显得更专业。
2. **读流程更准确的描述**：不是"数据过期了才去 MySQL 读"，而是：
   - 先查 Redis，命中则直接返回
   - 没命中（缓存 miss）→ 查 MySQL → 把结果写入 Redis → 返回
   - 缓存可能因为过期、被删除、从未加载过等原因不存在，不只是"过期"
1. **为什么是删除缓存而不是更新缓存**：
   - 更新缓存是写操作，如果写多读少，会导致大量缓存更新是无效的，浪费资源
   - 并发场景下更新缓存有风险：线程 A 更新数据库还没来得及更新缓存，线程 B 也更新数据库并更新了缓存，然后线程 A 的缓存更新才到，导致缓存是旧数据
   - 删除缓存更简单稳妥，让下一次读请求去拉取最新数据
1. **先更新数据库再删缓存的原因（你答到了，但可以更完整）**：
   - 如果先删缓存再更新数据库：在更新数据库的过程中，有其他线程来读，会读到旧数据并写回缓存 → 缓存脏数据
   - 先更新数据库再删缓存：删缓存很快，在这个短暂的时间窗口内有读请求的概率很低，即使有也只是短暂不一致，最终会一致
   - 这是**最终一致性**的方案，不是强一致
1. **进阶方案补充**：
   - **延迟双删**：更新数据库 → 删缓存 → 延迟一会再删一次（解决缓存更新期间的并发问题）
   - **消息队列保证最终一致性**：更新数据库后发消息，由消费者异步删除缓存，保证重试
   - **订阅 binlog**：监听 MySQL binlog 来异步更新/删除缓存
### 39. ConcurrentHashMap的原理
ConcurrentHashMap是并发安全的Map，JDK1.7之前，主要是通过分段锁的形式实现的，每一段都独立继承一个ReentrantLock，多线程读写时，每段会独自加锁，多段之间互不影响，提高了并发能力
JDK1.8之后，转为使用CAS + synchronized实现，和HashMap一样采用Node数组+链表+红黑树实现，初始化时采用无锁，只要桶上没有元素，就可以直接将数据插入进去。只有当发生哈希冲突，才会在链表或红黑树的头节点通过syncrhonized进行加锁。这意味着，只要多个线程不是操作同一个桶，就可以完全并发执行
ConcurrentHashMap对size()方法进行了优化，不再是简单的加锁统计数量，而是采用了`baseCount`和`CountCell`数组，当通过CAS更新`baseCount`失败时，就会采用`CountCell`数组去分散计算，最后再将baseCount和CountCell进行求和，从而提高了并发下获取数据的效率
### 40. 为什么Redis是单线程，但是性能却很好
Redis的单线程指的是命令的执行使用的是主线程。而之所以主线程能够支撑高并发读写，是因为Redis底层使用是IO多路复用，即一个主线程可以监听多个客户端事件，只有有事件发送过来才会去处理。这使得Redis可以同时处理成千上万的并发连接，而这使得Redis的性能瓶颈不在于CPU，而在于网络IO。而单线程就是解决网络IO瓶颈的最好的方式，它避免了多线程下共享数据需要的复杂的锁机制。而Redis6.0之后使用了多线程，是因为现在网络带宽更大，使得网络IO不再是性能瓶颈，所以多线程能够更好的处理数据包，并对其进行解析。而且Redis的多线程只在网络IO中使用，在执行命令时使用的还是单线程
### 41. jdk8对HashMap有什么优化
1. 引入红黑树，在jdk8之后，HashMap中引入了红黑树的数据结构，在数组长度大于64且链表长度大于8时就会进行数据结构的转换，链表会转换成红黑树，优化前，如果一个链表过长，在进行搜索时，链表上遍历搜索元素的时间复杂度是O(n)，而红黑树是O(logn)，优化了查询效率
2. 采用尾插法进行插入元素。jdk7及之前，HashMap在多线程扩容的场景下，使用头插法，可能会导致链表首尾相连，导致死循环。而jdk8之后就改用了尾插法，避免了上述场景的出现
3. 更高效的rehash，在jdk7及之前，当HashMap扩容时，需要为每个元素重新计算hash来获取新的下标位置，而jdk8之后，HashMap扩容时，利用HashMap数组长度为2的次幂的特性，新元素要么在原位置不变，要么就变为原位置+旧容量的位置
## 42. 对于一个`Map<Object,Object>hashMap`，put了一个Integer类型的数据，get(new Long(...))能否获取到
不能，HashMap在get时会先利用equals进行判断，当equals相等时，再利用hashCode进行判断。而对于eqauls方法，会进行类型检查，Integer和Long的类型不一致，所以获取到的值是null
### 43. WebSocket建立连接的具体过程
- 发起升级请求：客户端向服务端发送一个标准的HTTP GET请求。这个请求包含了特定的头信息，告诉服务器想把连接升级为WebSocket协议
- 服务器响应升级：服务器接收到请求后，如果支持WebSocket协议，就会返回一个`101 Switching Protocols`状态码的HTTP响应
- 连接升级完成：一旦客户端收到101响应，双方便完成握手，协议正式从HTTP升级为WebSocket。此时，之前用于握手的TCP连接会被保持，不会断开。这条连接从此刻开始作为全双工通信的通道
- 开始全双工通信：连接建立后，客户端和服务器就可以通过这条持久化的连接，随时、主动地向对方开始发送数据，实现了真正的双向实时通信
**请求头信息**
客户端发起GET请求中必须包含以下核心头部信息
- `Connection: Upgrade`和`Upgrade: websocket`：这两个请求头是必须的，它们组合在一起，明确告诉服务器这是一个升级请求，目标协议是websocket
- `Sec-WebSocket-Key`：这是一个Base64编码的16字节随机值。用于安全验证，服务器会用它来计算一个特定的值返回给客户端，以证明自己确实理解了WebSocket协议
- `Sec-WebSocket-Version`：告诉服务器客户端所使用的WebSocket协议版本
- `Host`：标准HTTP头部，指明服务器的主机名和端口号
**响应头信息**
- `HTTP/1.1 101 Switching Protocols`：状态行，不是头部，但它是握手成功最直接的标志
- `Connection: Upgrade`和`Upgrade: websocket`：与请求头对应，确认协议正在切换
- `Sec-WebSocket-Accept`：必须返回，它的值是服务器根据客户端发来的`Sec-WebSocket-Key`计算得出。客户端会验证这个值，以取保连接是安全的
## 44. MySQL锁
### 全局锁
全局锁主要是对整个数据库实例进行加锁。加锁后，整个数据库进入只读状态，所有数据更新、表结构变更等操作都会被阻塞
主要用于全局逻辑备份，以确保备份数据在时间点上的一致性
### 表级锁
表级锁是对整张表进行加锁。开销小、加锁快，但是并发度低。表级锁主要分为以下几种
- 表锁：可以分为读锁和写锁。加读锁时，其他用户也可以读，但是不能写数据。加写锁时，其他用户的读写都会被阻塞
- 元数据锁：主要是用于保护表结构不会变更。分为MDL读锁和MDL写锁。当对表进行DML（增删改查）时，自动加MDL读锁；当对表进行DDL（表结构更改）时，自动加MDL写锁
- 意向锁：InnoDB内部使用的表级锁，主要用于判断当前表有没有加行级锁。当事务想要给某些行加共享锁，需要先加上意向共享锁。当对行数据加排他锁前，需要先加意向排他锁
### 行级锁
行级锁是MySQL中锁定粒度最细的锁，只针对数据行。并发度高，锁冲突概率小，但是开销大，加锁慢。行级锁主要分为以下几种
- 记录锁：锁定某一行数据
- 间隙锁：锁定索引记录之间的间隙，主要为了防止幻读
- 临键锁：记录锁+间隙锁，既锁定记录也锁定间隙，是InnoDB默认的行锁算法，用于防止幻读
## 45. 为什么RocketMQ不使用Zookeeper
### CAP理论
Zookeeper是一个CP系统，在网络分区或Leader选举时，会牺牲可用性来保证数据强一致性。而RocketMQ的NameServer是一个AP系统，允许数据在节点间存在短暂的不一致，且保证了服务的持续可用
对于RocketMQ来说，注册中心的首要职责是服务发现，短暂的路由信息不一致是可以接受的。相比之下，因为Leader选举导致整个集群在几十秒内无法使用，是难以接受的
### 性能
NameServer在设计上比Zookeeper更轻量、性能更优
- 写入性能：Zookeeper的每次写入都需要多数节点确认，写入性能存在瓶颈且不可水平扩展。而NameServer节点间互不通信、无数据同步，Broker并行向所有NameServer发送心跳，单节点吞吐量可达10W+/s
- 架构扩展：NameServer集群无状态、去中心化，增加节点只需直接启动新实例，非常简便。而Zookeeper集群的扩容涉及复杂的数据同步和Leader选举
### 功能
Zookeeper是一个通用的分布式协调服务，功能强大但复杂，NameServer是专门为RocketMQ设计的组件，功能十分精简
- NameServer只关注**路由信息的注册与发现**，省去了Zookeeper的Leader选举、分布式锁、Watcher等RocketMQ不需要的复杂功能
- NameServer无需独立部署，可以随Broker一起部署；几乎没有复杂的配置；节点故障影响范围小，监控指标也更少
## 46. MySQL是如何实现事务的
MySQL只有InnoDB存储引擎支持事务。InnoDB实现事务的四大特性，主要依赖于**Undo Log、Redo Log、MVCC、锁机制**
### 原子性——Undo Log
原子性要求事务要么全部成功，要么全部失败。InnoDB通过Undo Log实现回滚功能
在修改数据之前，InnoDB会先将旧数据记录到UndoLog中。例如`UPDATE`操作会记录旧值；`DELETE`操作会记录旧行的完整数据
UngoLog存放在共享表空间的回滚段中。若事务执行失败或执行`ROLLBACK`，InnoDB会利用UndoLog中的数据将物理页恢复到修改前的状态
UndoLog不仅用于回滚，还用于为MVCC提供旧版本数据
### 持久性——RedoLog+Doublewrite
持久性要求事务提交之后，数据必须永久保存在磁盘中。InnoDB通过RedoLog+Doublewirte Buffer实现
数据页存储在磁盘上是随机读写，速度慢。InnoDB采用WAL技术，事务提交时，先修改Buffer Pool中的内存数据页，并将修改操作顺序写入RedoLog中，这时事务视为提交成功。即使数据库宕机，重启后也可以根据RedoLog重放操作，恢复已提交但未写入磁盘的数据
RedoLog还配合Binlog实现分布式事务一致性。RedoLog写入分为Prepare和Commit两阶段，确保MySQL主从数据一致性，防止主库重启恢复与从库重放Binlog产生的数据不一致
由于MySQL数据页大小(16KB)大于操作系统单次写入原子单位，为防止页写入过程中中途崩溃导致数据页损坏，InnoDB先将脏页拷贝到Doublewrite Buffer，刷盘成功后再写入数据文件
### 隔离性——锁机制+MVCC
隔离性要求多个事务并发执行时互不干扰。InnoDB采用MVCC和锁结合的策略，实现了非阻塞读操作
InnoDB为每行数据新增了隐藏字段`trx_id`和`roll_point`。通过UndoLog构建数据的历史版本链，结合ReadView决定事务能看到哪些版本的数据。对于快照读（普通SELECT语句）不加锁，通过MVCC获取事务开始时的快照数据，实现了读写不阻塞，极大地提高了并发性能；对于当前读`SELECT ... FOR UPDATE`或`UPDATE`、`DELETE`操作则读取最新版本，配合锁机制
InnoDB支持行级锁，并使用间隙锁和临键锁来解决幻读问题。对于RC级别，每次查询都重新生成ReadView，仅能查到已提交的数据。对于RR级别，事务只在第一次查询时生成ReadView，后续查询服用该视图，配合临键锁防止幻读
### 一致性——综合保障
一致性是事务层面的最终目的，它不仅依赖数据库自身的机制，也依赖应用层逻辑
- 数据库层面通过原子性和持久性保证数据状态不丢失；通过隔离性保证并发数据不混乱；同时依赖唯一键、外键、非空等约束来防止非法数据写入
- 数据库无法保证业务逻辑上的完整性，例如“转账金额不能为负”这类逻辑，需要利用应用程序的SQL判断来保证
## 47. SQL 语句在 MySQL 中的执行过程
### 查询SQL执行流程
假设执行`SELECT id,name FROM user WHERE age > 18 ORDER BY id LIMIT 10;`
- **连接认证**：客户端通过TCP/IP或Socket连接，连接器负责进行用户名/密码认证和SSL加密协商。认证通过后，连接器会读取该用户拥有的全局权限和库表权限。
- **查询缓存**：在MySQL8.0之前，Server层会将以SQL文本为Key的结果集缓存在内存中。MySQL8.0之后，由于表结构或数据一发生变化就会失效，命中率极低，官方就移除了这个功能
- **词法和语法分析**：将SQL字符串拆解成一个个Token。识别出`SELECT`、`user`、`age`、`>18`等关键字和非关键字。根据MySQL语法规则检查SQL语句是否符合范式，并生成一颗抽象语法树AST。检查AST中的表名、字段名是否存在，以及当前用户是否有操作这些对象的执行权限
- **查询优化（优化器）**：这时决定SQL快慢的核心环节。优化器收到语法树后，负责生成执行计划。它会进行成本估算，决定是全表扫描还是二级索引；如果存在多表连接，决定谁做驱动表，谁做被驱动表；ORDER BY走索引排序，还是在内存/磁盘中进行额外的排序操作
- **执行器调用引擎**：执行器根据优化器生成的执行计划，一步步调用存储引擎提供的读写接口。假设age二级索引。请求读取`age`索引树的第一条满足`>18`的叶子节点；根据叶子节点存储的主键ID，回到聚簇索引中查找完整的行数据；将完整行放入Server层的结果集缓冲中，并判断是否满足`LIMIT 10`的数量要求；继续调用接口读取下一条，知道扫描完所有符合条件的行或打到LIMIT阈值
- **返回结果**：执行器将最终的结果期以边读边发的方式通过网络协议返回给客户端，避免一次性占用过多的Server内存
### 更新SQL执行流程
假设执行`UPDATE user SET balance = balance - 100 WHERE id = 1`
更新流程的前四步（连接、分析、优化、执行器开启）与查询一致，但执行器内部涉及日志体系和内存脏页
- **加载数据页到Buffer Pool**：执行器调用InnoDB接口，根据`id=1`查找聚簇索引。如果该数据页不在内存中，InnoDB会从磁盘`.ibd`文件读取该页并加载到Buffer Pool
- **写入Undo Log**：在修改内存数据之前，先将该行数据的旧值写入UndoLog的回滚段中
- **更新内存数据**：将Buffer Pool中的该行数据修改为新的`balance`值。此时，该数据页变为脏页
- **写入Redo Log**：将修改操作以顺序追加的方式写入Redo Log Buffer，并将Redo Log状态标记为Prepare
- **写入binlog**：Server层将SQL语句写入Binlog缓存中，最终写入磁盘文件
- **提交事务**：InnoDB接收到Server层的提交指令后，将刚才的Redo Log状态从Prepare更新为Commit，并强制刷盘。为了保证RedoLog和Binlog逻辑一致，防止主从数据不一致。例如，写完Redo Log但Binlog写入失败，则回滚；反之若Binlog写入成功但Redo Log未Commit，则重启恢复时依据Binlog重做
- **返回客户端**：事务提交成功后，执行器会返回`OK`包给客户端
- **后台异步刷脏页**：后台的`Page Cleaner`线程在系统空闲或Redo Log空间不足时，将Buffer Pool中的脏页异步刷回磁盘数据文件。这一步不影响事务是否提交成功
## 48. 为什么Java8移除了永久代并引入了元空间
永久代在JDK7及更早的版本中存在一些根本性的问题
- **大小固定，极易OOM**：永久代大小在JVM启动时通过`-XX:PermSize`和 `-XX:MaxPermSize`固定。但应用中需要加载多少类很难预估，设置过小就会导致经典的`java.lang.OutOfMemoryError: PermGen space`错误。特别是在大量使用反射、动态代理、JSP等动态生成类的场景下，问题尤为明显
- **GC性能差且复杂**：永久代的垃圾回收效率低下且机制复杂。它和堆内存的GC是分开进行的，增加了复杂度。要回收一个无用的类，条件非常苛刻，导致类卸载困难，一旦永久代膨胀就很难收缩，进而影响应用性能
- **职责划分不清**：永久代存储的内容比较杂乱，除了类的元数据，还混杂了字符串常量池和静态变量等。这种设计不够清晰，也给内存管理带来了麻烦
针对上述问题，JDK8引入的元空间，对上述问题进行了改进
- **使用本地内存，动态扩展**：元空间的最大变化是不再使用JVM堆内存，而是使用本地内存。这意味着它的大小只受限于系统的可用内存，默认无上限。这极大地降低了因类元数据导致的内存溢出风险，并省去了复杂耗时的调优工作
- **简化GC，提升性能**：使用本地内存后，元空间的回收管理得到简化。当一个类加载器被卸载时，其对应的元空间内存可以被更高效地回收或返回给操作系统，从而提升了GC的整体效率
- **职责更清晰**：元空间专注于管理类的元数据，而字符串常量池和静态变量等被移动到了Java堆中，是内存布局更合理
## 49. MySQL事务的两阶段提交
MySQL中，两阶段提交通常指**InnoDB存储引擎为了确保崩溃恢复时的数据一致性，在内部Redo log和Binlog之间进行的一套协调机制**
核心作用是：保证**数据库数据**与**用于复制和恢复的Binlog**逻辑一致，避免主从数据不一致
在MySQL中，数据的持久化主要依赖于两种日志
- Redo Log（InnoDB引擎层）：记录物理修改，用于崩溃恢复，保证食物的持久性
- Binlog（Server层）：记录逻辑修改，用于主从复制和基于时间点的数据恢复
在事务提交时，Redo log和Binlog需要分别写入磁盘。如果这两次写入不是原子的，就会出现以下问题
- Redo Log写完了并提交，但Binlog还没来得及写，MySQL宕机了。恢复后数据在，但是Binlog中没有这条记录，导致从库丢失数据
- Binlog写完了，但是Redo Log还没提交，MySQL宕机了。恢复后数据回滚，但Binlog中却有这条记录，导致从库中多出数据
### 两阶段提交执行流程
假设事务执行`UPDATE t set a=1 WHERE id=1`，提交过程如下
**准备阶段**
- 写Redo Log：InnoDB将修改写入Redo Log，并将改日志的状态标记为`prepare`
- 刷盘：Redo Log的`prepare`日志落盘
**提交阶段**
- 写Binlog：MySQL Server层将事务的逻辑修改写入Binlog，并完成刷盘落盘
- 写提交标记：InnoDB将刚才那条Redo Log的状态从`prepare`修改为`commit`，表示事务正式提交成功
- 返回成功：客户端收到事务提交成功的返回
## 50. RocketMQ的事务消息
RocketMQ的事务消息是通过两阶段提交的方式来实现的
**第一阶段**
- 生产者先将消息发送到RocketMQ的Topic，此时消息的状态为半消息，消费者不可见
- 然后，生产者执行本地事务逻辑，并根据本地事务的执行结果来决定下一步的操作
**第二阶段**
- 如果本地事务成功，生产者会向RocketMQ提交`commit`操作，将半消息变为正式消息，消费者可见
- 如果本地事务失败，生产者会向RocketMQ提交`rollback`操作，RocketMQ会丢弃这个半消息
- 如果生产者没有及时提交`commit`或`rollback`操作，RocketMQ会定时回查生产者本地事务状态，决定是否提交或回滚消息
## 51. MVCC
MVCC是MySQL InnoDB存储引擎用于控制数据并发访问的核心机制。其核心思想是为每一行数据维护多个历史版本，让读操作通过读取某个时间点的快照来获取数据，从而避免枷锁带来的性能损耗，实现非阻塞的读操作
InnoDB中的每行数据除了用户定义的列之外，还包含两个隐藏列`trx_id`、`roll_ptr`
- `trx_id`：记录了最后一次修改该行的事务ID，全局递增
- `roll_ptr`：回滚指针，指向undo log中该行的上一个版本
当一个事务修改某行数据时，InnoDB不会立刻覆盖旧数据，而是将修改前的数据保存到undo log中，然后更新该行的`trx_id`为当前事务ID，`roll_ptr`指向undo log中的旧版本
多次修改后就会形成一条版本链
ReadView是MVCC的决策大脑，它决定了当前事务能看到版本链中的哪一个版本
一个ReadView包含了四个字段`m_ids`、`min_trx_id`、`max_trx_id`、`creator_trx_id`
- `m_ids`：生成ReadView时，当前系统中活跃的未提交的事务
- `min_trx_id`：m_ids中的最小值
- `max_trx_id`：生成当前ReadView时，系统要为下一个事务分配的事务ID
- `creator_trx_id`：生成该ReadView的事务自己的ID
**判断可见性**
针对版本链中的某一个版本，判断是否对当前事务可见，主要依赖于以下规则
- `trx_id < min_trx_id`：说明这个版本的事务已经提交，数据可见
- `trx_id > max_trx_id`：说明这个版本的事务是在ReadView生成之后才创建的，不可见
- `min_trx_id < trx_id < max_trx_id`：分为两种情况
	- `trx_id`在`m_ids`中：当前版本的事务还未提交，数据不可见
	- `trx_id`不在`m_ids`中：当前版本的事务已经提交，数据可见
判断时，会从版本链的最新版本开始，逐渐向前遍历，直到找到第一个满足可见性的版本
对于RC隔离级别下，每次SELECT时都会生成一个新的ReadView，而对于RR隔离级别下，只有事务创建之后，第一次SELECT才会生成新的ReadView，这个ReadView一直使用到事务结束
## 51. MySQL的乐观锁和悲观锁
### 悲观锁
认为每次数据操作都有可能会被其他事务修改。所以在操作数据之前，先主动加锁，确保整个操作过程中数据不被外界干扰，操作完成后再释放锁
在MySQL InnoDB中，通过`SELECT ... FOR UPDATE`语句实现。执行该语句时，数据库会对查询到的行加上排他锁，其他事务的查询、更新、删除操作都会被阻塞，直到当前事务提交或回滚
优点是：数据一致性强，绝对串行化，无脏读幻读风险；实现简单，数据库自动管理锁的获取和释放；适合写入频繁、冲突概率极高的场景
缺点是：并发性能差，大量线程排队等待锁；容易产生死锁（多个事务互相持有对方需要的锁）；若索引失效，行锁可能升级为表锁，性能急剧下降
### 乐观锁
认为数据被并发修改的概率很低。因此不加锁，而是通过业务逻辑来校验数据版本，在提交更新时才检查是否被其他事务修改过
实现时，不依赖数据库锁，通常通过版本号或时间戳机制在应用层面实现。可以在表中新增`version`字段。读取数据时获取当前`version`，更新时`version+1`，同时`where`条件带上旧版本号。若影响行数为0，说明数据已经被修改，业务端重试或抛出异常。或者也可以利用`last_update_time`代替`version`，更新时检查时间戳是否一致
优点是：无锁机制，读写不阻塞，性能极高；无死锁风险；适合读多写少、冲突概率低的场景
缺点是：冲突频繁时，大量事务重试会严重降低性能；数据一致性需由业务层保证，开发复杂度较高；若并发很高，ABA问题理论上需要结合时间戳等额外手段处理
## 52. MySQL中死锁该如何解决
### 定位与分析死锁
- **查看死锁日志**：MySQL会将死锁信息记录在错误日之中。在MySQL客户端执行`SHOW ENGINE INNODB STATUS\G`，在输出中查找`LATEST DETECTED DEADLOCK`部分，这里会显示最近一次死锁的详细信息；也可以在`my.cnf`中添加`innodb_print_all_deadlocks = 1`，这样每一次死锁都会被记录到错误日志中，方便追溯历史问题
- **分析死锁日志**：拿到日志后，重点关注以下内容。
	- **事务与SQL**：找到参与死锁的事务ID以及它们正在执行的SQL语句
	- **持有的锁**：查看每个事务当前`HOLDS THE LOCK(S)`的信息，了解它占有了哪些资源
	- **等待的锁**：查看每个事务`WAITING FOR THIS LOCK TO BE GRANTED`的信息，了解它在等待什么资源
	- **受害者**：日志汇明确指出哪个事务被回滚
### 解决当前死锁
- **自动处理机制**：InnoDB内置了死锁检测机制。一旦检测到死锁，他会自动回滚代价最小的事务（通常是修改行数较少的一个），让另一个事务继续执行。对于偶发的死锁，应用层捕获到`1213`错误后重试事务即可
- **手动干预**：如果死锁持续发生且无法自动解除，可以手动终止会话
	- **查找阻塞会话**：执行`SHOW PROCESSLIST;`或查询`information_schema.INNODB_TRX`表，找到长时间处于锁等待状态的线程ID
	- **终止会话**：执行`KILL <线程ID>;`命令强制终止该会话，释放其持有的锁
	- **调整参数**：可以调整`innodb_lock_wait_timeout`参数，缩短锁等待超市时间。当事务等待超过设定值时会自动回滚，从而避免长时间的死锁僵局
### 从根源上预防死锁
- **统一加锁顺序**：这是最核心的预防手段。所有事务在修改多个资源时，必须遵循完全相同的顺序。例如，事务A和事务B都先更新id为1的行，再更新id为2的行，就能有效避免循环等待
- **缩短事务**：事务越小、越快，持有锁的时间就越短，冲突概率越低。将复杂的业务逻辑拆分成多个小事务；在事务中只保留最核心的数据库操作，避免用户交互或远程调用等耗时操作
- **优化查询与索引**：如果没有合适的索引，`UPDATE`或`DELETE`操作可能锁住大量行甚至全表。为`WHERE`条件创建精准的索引，能让语句只锁定必要的行，大幅度降低锁竞争
- **选用合适的隔离级别**：隔离级别越高，锁越严格，死锁风险越大。如果业务允许，可以考虑使用`READ COMMITTED`级别，它不使用间隙锁，能有效地减少死锁
## 53. MySQL如何进行SQL调优
### 定位慢SQL
- **开启慢查询日志**：在`my.cnf`中配置`slow_query_log=1`，设置`long_query_time=1`，并指定`slow_query_log_file`路径
- **实时监控**：使用`SHOW PROCESSLIS;`查看当前正在执行且耗时较长的查询；或使用`SHOW FULL PROCESSLIST;`查看完整的SQL文本
### 分析执行计划
利用EXPLAIN查看执行计划，重点查看以下几个字段
- **type**：访问类型，从好到差`system`>`const`>`eq_ref`>`ref`>`range`>`index`>`ALL`。出现`ALL（全表扫描）`或`index（全索引扫描）`时，必须优先优化
- **possible_keys**：可能用到的索引，为`NULL`则说明无索引可以用
- **key**：实际用到的索引，与`possible_keys`不符，或为`NULL`
- **rows**：预估扫描的行数，数字越大，性能越差
- **Extra**：额外信息，出现`Using temporary（用临时表）`或`Using filesort（文件排序）`是性能大忌
- **filtered**：返回结果占扫描行数的比例，百分比越低，说明扫描了大量无用数据
### 索引优化
- **遵循最左前缀法则**
- **优先使用覆盖索引**
- **避免索引失效的常见写法**：在索引列上做运算/函数；隐式类型转换；`LIKE`以通配符开头；`OR`条件
### SQL语句重写
即使有索引，SQL写法不对也会拖垮性能
- **深分页问题**：`LIMIT 1000000,10`需要扫描100万行再丢弃，效率极低。可以尝试以下做法
	- **延迟关联**：先通过覆盖索引查出主键，再关联取数据
	```sql
	SELECT * FROM orders
	JOIN (SELECT id FROM orders WHERE status = 1 LIMIT 1000000,10) AS tmp
	ON orders.id = tmp.id
	```
	- **记录上一次位置**：记住上一页最后一条`id`，用`WHERE id > last_id LIMIT 10`
- **优化`COUNT`和`DISTINCT`**：若业务允许，使用`COUNT(*)`而不是`COUNT(列名)`，前者会优化走最小二级索引；避免在`COUNT`中加`WHERE`后扫描大表，可以考虑缓存或额外汇总表
- **用UNION ALL代替UNION**：`UNION`会去重，需要排序和临时表；如果业务允许重复，使用`UNION ALL`性能提升巨大
- **避免无用排序**：`ORDER BY`字段如果能用到索引，则`Extra`显示`Using index`，否则出现`Using filesort`。减少排序字段或建立联合索引
## 54. Spring的启动流程
Spring的启动过程，本质上就是IoC容器的初始化过程，这个过程的总指挥就是`AbstractApplicationContext`类的`refresh()`方法
**容器刷新前的准备工作**
- **设置启动状态**：记录启动时间，将容器状态标记为活跃
- **初始化属性资源**：初始化上下文环境(`Environment`)占位符，为后续的属性解析做准备
- **验证必要属性**：验证`Environment`中标记为“必需”的属性是否存在
### 创建或获取BeanFactory
获取或创建一个`BeanFactory`实例，它是Spring IoC容器的核心实现，实现类是`DefaultListableBeanFactory`。你可以把它理解为存放所有`BeanDefinition`的仓库
**配置和增强BeanFactory**
- **prepareBeanFactory**：进行通用配置，如设置类加载器、表达式解析器等，并注册两个重要的**Bean后置处理器**
- **postProcessBeanFactory**：这是一个模版方法，留给字类扩展。例如，Web环境下的`ApplicationContext`会利用它注册`request`、`session`等作用域
- **invokeBeanFactoryPostProcessors**：调用所有已注册的`BeanFactoryPostProcessor`。这个阶段可以修改`BeanDefinition`。SpringBoot的自动配置核心`ConfigurationClassPostProcessor`就是在这里被调用的
- **registerBeanPostProcessors**：注册所有`BeanPostProcessor`。它们会监听Bean生命周期，在Bean实例化、初始化等阶段进行干预
**初始化应用上下文基础设施**
这几步为容器增添一些高级功能
- **initMessageSource**：初始化国际化消息
- **initApplicationEventMulticaster**：初始化事件广播器
- **onRefresh**：模版方法。在SpringBoot中，内嵌的Web服务器就是在这里被创建和启动的
- **registerListeners**：将实现了`ApplicationListener`接口的Bean注册到事件广播器中
**完成BeanFactory的初始化**
这是最核心的一步，它会实例化所有非懒加载的单例Bean。在此过程中，会进行依赖注入，并触发之前注册的各种`BeanPostProcessor`，完成AOP代理等复杂功能
**完成刷新**
收尾工作
- **清理资源缓存**：清理一些临时的资源缓存
- **发布事件**：发布`ContextRefreshedEvent`事件，通知所有监听器容器已启动完成
## 55. SpringBoot的启动过程
SpringBoot的启动过程，本质上可以看作是对Spring `refresh()`方法的一层高度封装和自动化扩展。整个过程围绕`SpringApplication`类展开，主要分为**初始化`SpringApplication`实例和执行`run`方法两个阶段**
**准备与规划**
这个阶段会扫描并加载必要的组件，为后续的启动做准备
- **推断Web应用类型**：根据Classpath下是否存在特定类，判断应用是`Servlet`、`Reactive`还是`None`。这决定了后续是否启动以及启动哪种内嵌的Web服务器
- **加载并初始化`ApplicationContextInitializer`**：从`META-INF/spring.factories`文件中加载所有`ApplicationContextInitializer`的实现类并实例化。它们会在IoC容器刷新前被调用，用于对容器进行额外的配置
- **推断主类**：通过堆栈信息找到包含`main`方法的真实启动类
**执行与启动**
实例化完成后，调用`run`方法正式启动应用。这一阶段会触发`refresh()`方法
- **启动计时与Headless模式配置**：创建`StopWatch`开始计时，并将系统设置为`java.awt.headless`模式以适应服务器环境
- **获取并通知`SpringApplicationRunListener`**：从`spring.factories`加载`SpringApplicationRunListener`，并通过它们在不同阶段发布生命周期
- **准备`Environment`**：创建并配置应用的环境，包括加载`application.yml`或`application.properties`配置文件，并设置命令行参数
- **打印Banner**：在控制台打印SpringBoot的启动图标
- **创建ApplicationContext**：根据第一阶段推断出的Web应用类型，反射创建对应的IoC容器实例。例如Servlet环境会创建`AnnotationConfigServletWebServerApplicationContext`
- **准备上下文**：将`Environment`等配置应用到容器，并执行在第一阶段加载的所有`ApplicationContextInitializer`的`initialize`方法
- **刷新上下文**：这是最核心的一步。它调用Spring容器的`refresh`方法，完成Bean的加载、实例化、依赖注入等所有核心工作。同时，内嵌的Web服务器也是在此阶段被创建和启动的
- **刷新后处理**：一个空的模板方法，预留用于在容器刷新后执行自定义逻辑
- **调用`ApplicationRunner`和`CommandLineRunner`**：容器启动成功后，会遍历并执行所有实现了这两个接口的Bean的`run`方法，用于执行一些启动后的初始化逻辑
- **发布就绪事件并结束计时**：发布`ApplicationReadyEvent`事件，标志着应用已经完全启动并可提供服务。最后停止计时并打印启动耗时日志
## 56. TCP和可靠UDP的区别
TCP是操作系统内核提供的，标准的可靠传输协议；而可靠UDP是在UDP的基础上，在应用层自己实现的一套可靠传输机制
### TCP
TCP在发送数据包前会先建立连接，确保接收方有效。数据包发送出去后需要接收方进行响应，如果数据包丢失，就会重新发送一个新的数据包。TCP会确保每个数据包都能够按顺序到达
**特点**：可靠，但是传输速度慢，效率低。建立连接，确认，重传等机制会消耗大量的时间和系统资源
**应用**：网页浏览(HTTP/HTTPS)、文件传输(FTP)、邮件(SMTP)、远程登录(SSH)等
当需要数据必须完整，数据的顺序必须严格，开发简单时，就可以选用TCP协议
### UDP
UDP在发送数据包时，不会关心接收方是谁，有没有收到，也不保证数据包的顺序。发送速度极快
**特点**：速度极快，但是不可靠。没有繁琐的确认和重试机制，开销小
**应用**：实时视频、直播、在线游戏、DNS域名查询等
当需要速度是第一位时，能容忍少量数据丢失，需要极低延迟，需要一对多通信时，可以选用UDP协议
**可靠UDP**
可靠UDP使用UDP的高速传输，但是在应用程序代码中，模仿TCP的确认、重传、排序等机制，实现可靠传输
**特点**：兼容UDP的速度和TCP的可靠性，但实现极其复杂。开发、调试和维护成本高昂
**应用**：对延迟极度敏感的竞技类游戏、需要毫秒级响应的高频金融交易等
选用可靠UDP时，是需要对延迟的要求极其严苛，并且需要保证数据可靠性，且能够投入大量精力去实现和维护时，就可以选用可靠UDP
### 可靠UDP解决了TCP的哪些问题
- **队头阻塞**：TCP要求数据包严格按顺序到达。如果传输过程中第二个包丢了，即使第3、4、5个包已经到达，TCP也会把它们卡在缓冲区里，必须等待第二个包重传成功，才会把后续数据一起交给应用。以QUIC（可靠UDP实现的协议）为例，它支持多流并发。如果传的是3张不同的图片，它们各自走独立的流。第一张图片包丢失了，只会卡住第一张图，而第二、三张照常显示，互不干扰
- **连接建立太慢**：建立可靠连接需要3次握手，建立加密通道需要TLS1.2的2次往返，加起来，客户端发第一个有效数据前，需要来回跑三趟。QUIC在握手时加密和传输一起协商，实现了0-RTT或1-RTT快速恢复。如果之前连接过，再次连接时，客户端可以直接携带加密数据和请求一起发出去，省去了等待握手的纯耗时
- **断线重连能力差**：TCP连接由 **（源IP，源端口，目标IP，目标端口）** 这四元组唯一标识。当手机从Wi-Fi切换到5G时，IP地址变了，四元组失效，TCP连接立即断开。必须重新进行3次握手+TLS握手。可靠UDP使用**Connection ID**，这是一个随机生成的64位数字，独立于IP和端口。即使切换了网络，只要在包里带上这个ID，服务器就能立刻认出你是谁
- **协议升级困难**：想要升级TCP的拥塞控制算法或增加新特性，必须升级操作系统内核。而可靠UDP完全跑在用户空间。开发者想要优化算法、调整重传策略，只需要更新应用程序的代码即可，不需要动地层系统
## 57. Redis实现分布式锁可能遇到的问题
- 加锁和设置过期时间是两个独立的命令，非原子操作。加锁成功后，若在设置过期时间前程序崩溃，锁将永不过期，形成死锁
- 程序因异常或逻辑疏忽，没有执行释放锁的代码。锁无法被主动释放，只能等待其自动过期，这段时间内其他线程无法获取锁，影响系统吞吐量
- 线程A的锁因执行业务超时被Redis释放，随后线程B获取了锁。当线程A执行完毕，错误地释放了线程B的锁。导致线程B的锁被提前误删，破坏了锁的互斥性，可能引发并发安全问题
- 业务逻辑执行时间超过了锁的过期时间，导致Redis锁被提前释放
- 同一个线程在已经持有锁的情况下，再次尝试获取同一把锁时被阻塞。在需要递归调用或嵌套加锁的场景下，会导致死锁（自己等待自己释放锁）
- 获取锁失败后，没有合理的重试或等待机制，直接返回失败。高并发清空下，导致大量请求失败，用户体验感差，资源利用率低
- 高并发下，大量线程/进程激烈竞争同一把锁。造成Redis性能瓶颈，系统吞吐量急剧下降
