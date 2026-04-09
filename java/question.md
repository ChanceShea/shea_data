# Java基础面试题

1. Java 有哪些核心特点？
2. Java 为什么能够实现跨平台？
3. JVM、JRE、JDK 分别是什么？它们之间有什么关系？
4. Java 为什么既可以说是编译型语言，又可以说是解释型语言？
5. Java 中参数传递为什么只有值传递？
6. Java 中基本数据类型有哪些？分别占多少字节？
7. 自动类型转换和强制类型转换有什么区别？
8. 为什么金融场景中通常不建议使用 `float` 或 `double` 做精确计算？
9. `BigDecimal` 为什么推荐使用字符串构造？
10. 什么是自动装箱和自动拆箱？它们会带来什么问题？
11. 为什么 Java 中需要 `Integer` 这样的包装类？
12. `int` 和 `Integer` 有哪些区别？
13. 什么是 `Integer` 缓存？范围是多少？
14. 面向对象的三大特性是什么？
15. 封装的意义是什么？
16. 继承有什么优点和限制？
17. 多态是什么？它的实际价值是什么？
18. 方法重载和方法重写有什么区别？
19. 向上转型和向下转型分别是什么？向下转型有什么风险？
20. 抽象类和普通类有什么区别？
21. 接口和抽象类应该如何选择？
22. `String`、`StringBuilder`、`StringBuffer` 有什么区别？
23. 为什么 `String` 适合作为 `HashMap` 的 key？
24. `HashMap` 的底层数据结构是什么？
25. `HashMap` 的 `put` 流程大致是怎样的？
26. `HashMap` 的 `get` 流程大致是怎样的？
27. `HashMap` 为什么要求 key 正确重写 `equals()` 和 `hashCode()`？
28. `HashMap` 的 key 可以为 `null` 吗？value 可以为 `null` 吗？
29. `HashMap` 为什么线程不安全？
30. `HashMap` 的扩容机制是怎样的？默认负载因子为什么是 `0.75`？
31. `HashMap` 的容量为什么通常是 2 的幂？
32. `HashMap` 和 `Hashtable` 有什么区别？
33. `ConcurrentHashMap` 是如何保证线程安全的？
34. JDK 1.7 和 JDK 1.8 的 `ConcurrentHashMap` 实现有什么区别？
35. 为什么 `ConcurrentHashMap` 中既使用了 `synchronized`，也使用了 CAS？
36. Set 集合为什么可以保证元素不重复？
37. JVM 的类加载过程包括哪些阶段？其中“加载、链接、初始化”分别做了什么？
38. 什么是双亲委派机制？它解决了什么问题？有没有场景会主动打破它？
39. Java 运行时数据区域有哪些？堆、虚拟机栈、方法区分别存什么？
40. 可达性分析为什么比引用计数法更适合 Java 的垃圾回收？
41. JVM垃圾回收中，哪些对象可以作为GC Roots？请详细说明每种GC Roots的具体来源。
42. Spring框架中，Bean的生命周期包括哪些关键阶段？请描述从Bean定义到销毁的完整过程。
43. MySQL的InnoDB存储引擎中，MVCC（多版本并发控制）是如何实现的？请解释undo log、read view和版本链的关系。
44. Spring AOP中，JDK动态代理和CGLIB动态代理有什么区别？分别在什么场景下使用？
45. MySQL的InnoDB存储引擎中，索引的底层数据结构是什么？请解释B+树相比B树的优势。
46. JVM中，垃圾回收算法有哪些？请分别说明标记-清除、标记-整理、复制算法的原理和优缺点。
47. Spring框架中，事务的传播行为有哪些？请详细说明PROPAGATION_REQUIRED和PROPAGATION_REQUIRES_NEW的区别。
48. Redis有哪些持久化方式？请对比RDB和AOF的优缺点及适用场景。
49. 什么是Java内存模型（JMM）？请解释happens-before原则及其作用。
