# Java后端模拟面试错题记录

> 说明：这里会持续记录本轮面试中回答错误或不完整的问题。

## 错题列表

### 1. 请说一下 `HashMap` 和 `ConcurrentHashMap` 的核心区别，至少从线程安全、底层实现、适用场景这三个角度回答。

回答中需要修正的点：

1. `ConcurrentHashMap` 不是“在 HashMap 的基础上实现线程安全机制”这么简单，更准确地说，它是一个单独设计的并发哈希容器，底层实现和并发控制策略都与 `HashMap` 有明显区别。
2. `ConcurrentHashMap` 在 JDK 1.8 中不是“并发量小用 CAS，并发量高就转换成 synchronized”这样的动态切换逻辑。更准确地说，是在不同操作阶段使用不同手段：
   - 桶位为空等简单场景优先使用 CAS。
   - 发生哈希冲突、需要修改链表或红黑树结构时使用 `synchronized`。

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

### 5. 请说一下 JVM 内存区域有哪些？重点讲一下堆、虚拟机栈、方法区分别存什么。

回答中需要修正的点：

1. 线程私有和线程共享划分答反了。
   - 线程私有：程序计数器、虚拟机栈、本地方法栈
   - 线程共享：堆、方法区
2. 虚拟机栈不属于线程共享区，它是线程私有的。
3. 堆主要存储对象实例和数组，这一点答对了。
4. 虚拟机栈中的确存储栈帧，但最好继续展开：栈帧中通常包含局部变量表、操作数栈、动态链接、方法返回地址等。
5. 方法区的表述不够完整。方法区主要存储类元信息、运行时常量池、静态变量、JIT 编译后的代码等，不只是“静态变量、方法链接”。

### 6. 请讲一下 `==` 和 `equals()` 的区别，再顺带说一下为什么重写 `equals()` 时通常也要重写 `hashCode()`。

回答中需要修正的点：

1. “默认情况下，`==` 和 `equals` 没有区别”这句话不够严谨。更准确地说：
   - 对于引用类型，如果没有重写 `equals()`，那么 `Object.equals()` 默认比较的是引用地址，看起来和 `==` 类似。
   - 但 `==` 本身既可以比较基本类型的值，也可以比较引用类型的地址；`equals()` 只能用于对象比较，语义上更偏向“逻辑相等”。

### 7. 请说一下 Spring 中 `BeanFactory` 和 `ApplicationContext` 的区别，以及为什么实际开发里更常用 `ApplicationContext`。

回答中需要修正的点：

1. 只答了“`ApplicationContext` 继承自 `BeanFactory`”，缺少关键差异。
2. 更完整的回答应该包括：
   - `BeanFactory` 是 Spring 最基础的 IoC 容器，功能较轻，偏底层。
   - `ApplicationContext` 在 `BeanFactory` 基础上提供了更多企业级能力，例如国际化、事件发布、资源加载、AOP 和自动注册后处理器等支持。
   - `ApplicationContext` 通常默认预实例化单例 Bean，使用体验更完整，更适合实际项目开发。
3. 实际开发更常用 `ApplicationContext`，是因为它功能更丰富、开箱即用，能覆盖大多数 Spring 应用场景。

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
5. 条件装配的本质是：只有在满足特定环境、类路径、Bean、配置项等条件时，自动配置类或 Bean 才会生效，避免无效装配和配置冲突。

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
### 29. 流式输出除了SSE还有什么
### 30. Mybatis是如何执行sql语句的，xml文件是如何转换成方法的
### 31. MybatisPlus如何实现分页插件的