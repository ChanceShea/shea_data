# Spring
## Spring框架的核心特性
**IoC容器**：Spring通过控制反转，将创建对象的操作交给容器进行，用户只需要对Bean对象进行定义，由Spring容器对这些Bean对象进行创建和初始化
**AOP**：面向切面编程，允许用户定义横切关注点，在不修改原方法的前提下，对方法进行一些增强功能。例如打印日志操作，传统的写法，在每个需要打印日志的方法中都需要写一遍代码，太过于冗余。通过AOP可以将打印日志的操作抽离出来，通过切入点来定义哪些方法需要打印日志，提高了代码的可维护性
**事务控制**：Spring提供了一系列的事务管理接口，支持声明式和编程式事务，用户可以轻松管理事务，不需要调用一系列复杂的接口
**MVC框架**：Spring MVC 是一个基于Sevelet API 定义的一个Web框架，有模型-视图-控制器三层。支持灵活的URL映射
## IoC
Spring IoC容器是一种创建对象的技术，DI（依赖注入）是其实现这一技术的方法。传统开发中，通常需要通过new关键字来新建对象，对象过多时会导致代码过于冗余，而通过IoC容器的思想，将创建对象的操作交给容器管理，而是通过Bean容器来实例化对象。省去了new对象的麻烦，同时也减少了对象之间的耦合度
## AOP
面向切面编程，Spring AOP会将那些与业务代码无关，且会被多个业务模块共同调用的逻辑封装起来，以减少系统的重复代码，降低冗余度。Spring AOP是基于动态代理的，对于代理对象，如果实现了某个接口，则会使用JDK Proxy 去创建代理对象，而对于没有实现接口的类，则会通过Cglib生成一个被代理对象的子类来作为代理对象
Spring AOP有以下几个核心概念
**横切关注点** 散步在应用程序多个模块的方法，非核心业务，但是会被业务中的一些方法会调用
**切面** 切面是横切关注点的模块化，一个类用于封装多个横切关注点的行为
**连接点** 程序执行过程中可以插入切面的一个点，例如方法调用，异常处理等。在Spring AOP中，仅支持方法级别的连接点
**通知** 切面要完成的工作，即切入的方法，描述了切面在“什么时候”做“什么事情”。通知也分多种，前置通知（Before），后置通知（After），环绕通知（Around）等
**切入点** 切入点是一个表达式，定义了通知在何处使用
**织入** 将切面应用到目标对象中，从而创建代理对象的过程
## BeanFactory
BeanFactory是Spring IoC容器的基础实现，提供了完整的IoC服务支持。BeanFactory是一个工厂模式的实现，负责创建对象，管理和配置Bean对象
BeanFactory加载Bean对象时使用了懒加载模式，只有当使用到Bean对象时才会进行创建，否则一直都只是存储着Bean对象的BeanDefinition
### XmlBeanFactory
XmlBeanFactory是BeanFactory主要实现之一，通过读取xml配置文件来管理Bean对象
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.example.UserService">
        <property name="userDao" ref="userDao"/>
    </bean>

    <bean id="userDao" class="com.example.UserDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test"/>
       <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

</beans>
```
例如上述代码，XmlBeanFactory就可以从中读取到对应的Bean对象，并将其加载到IoC容器中
### DefaultListableBeanFactory
DefaulListableBeanFactory是通过读取Java代码来进行Bean对象的管理，也是较为完善的一种实现
```java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        return dataSource;
    }

}
```
通过注解的形式，@Configuration和@Bean配合使用，从中读取到Bean对象并加载到IoC容器中  **@Configuration所定义的类，如果没有被扫描到，则需要试用@ComponentScan来扫描该包下的配置对象**
### BeanFactoryPostProcessor
BeanFactoryPostProcessor是BeanFactory的扩展点之一，它允许BeanFactory在实例化，配置和初始化Bean之前，对BeanFactory做一些修改
#### ConfigurationClassPostProcessor
这是一个用于处理@Configuration注解的类，通过扫描所有加了@Configuration注解的类，并解析这个类上的注解（@Bean，@ComponentScan等），并将其注册成BeanDefinition，最后通过CGLIB进行代理增强
### BeanPostProcessor
BeanPostPostProcessor是BeanFactory的扩展点之一，它允许在Bean实例化，依赖注入和属性设置之后对Bean进行一些处理
#### AutowiredAnnotationBeanPostProcessor
这是一个处理@Autowired注解的后置处理器，从而实现依赖自动注入
## Spring Bean
![](C:\Users\xgw\AppData\Roaming\marktext\images\2025-11-30-13-36-19-image.png)
Spring Bean的生命周期主要分为四个阶段
1. **实例化** ：当需要创建一个Bean时（例如，第一次被请求或容器启动时对于单例Bean），容器会通过反射调用其构造函数来创建对象实例。此时，对象只是一个“裸”对象，其属性还没有被设置，也还没有执行任何初始化逻辑
2. **依赖注入** ：Spring解析Bean之间的依赖关系，并通过反射将值（其他Bean的引用、配置值等）设置到新创建的Bean实例的相应字段或Setter方法中。如果Bean实现了 `Aware` 系列接口，Spring会在此阶段调用这些接口的方法，将一些容器相关的对象“注入”给Bean
3. **初始化** ：所有 BeanPostProcessor 的 postProcessBeforeInitialization方法会被调用。这是Spring提供的一个非常强大的扩展点，可以用于在初始化之前修改Bean（例如，进行代理包装）。@PostConstruct 注解的处理就是通过一个 BeanPostProcessor (CommonAnnotationBeanPostProcessor) 在此阶段完成的。
   **@PostConstruct 注解方法**： 
   如果Bean的方法上标注了 @PostConstruct 注解，该方法会被调用。**InitializingBean.afterPropertiesSet**： 
   如果Bean实现了 InitializingBean 接口，其 afterPropertiesSet() 方法会被调用
4. **销毁** ：当Spring容器被关闭时（例如，调用 ApplicationContext.close()），容器会管理Bean的销毁过程
   **@PreDestroy 注解方法**： 
   如果Bean的方法上标注了 @PreDestroy 注解，该方法会被调用
### Bean的三种初始化方法
1. @PostConstruct：Spring的扩展功能，在Spring中需要手动配置对应的后处理器
2. InitializingBean：实现InitializingBean接口，并重写afterPropertiesSet方法
3. initMethod：在@Bean注解中给出initMethod属性，并在xml文件中配置相应的初始化方法
执行顺序：@PostConstruct  >  InitializingBean  >  initMethod
销毁方法：@PreDestroy  >  DisposableBean  >  destroyMethod
## Aware
Aware接口是Spring提供的一组标记接口，用于让Bean感知Spring容器的某些特定对象或资源  Spring在初始化容器时，会检查容器是否实现了Aware接口，如果是，则调用setter方法注入相应的依赖
### BeanNameAware
用于注入Bean的名字
```java
@Slf4j
public class MyBean implements BeanNameAware {
    @Override
    public void setBeanName(String name) {
        log.info("获取到Bean的名称: {}", name);
    }
}
```
### tips
使用@Autowired，@PostContruct等注解注入，通常需要后处理器来对注解进行处理，属于扩展功能，而对于Aware接口，属于Spring内置功能，在某些情况下，扩展功能可能会失效，而Aware接口不会失效
## 作用域
**singleton：** 每次从容器中获取对象时，获取到的都是同一个对象，即Bean对象是单例的
**prototype：** 每次从容器中获取对象时，都会在容器中重新创建一个新的Bean对象，并返回
**request：** 每次发送一个新的请求时，就会创建一个新的Bean对象
**session：** 每次打开一个新的会话，就会创建一个新的Bean对象
**application：** 只要是同一个Web应用程序，就是同样的Bean对象，不同的Web应用程序就会创建不同的Bean对象 
# Spring Cloud
## 配置中心
## 远程调用
## 负载均衡
负载均衡就是将负载分摊到多个操作单元上进行执行，负载均衡可以分为客户端负载均衡和服务端负载均衡
客户端负载均衡指的是发生在服务提供者一方，比如常见的nginx负载均衡，服务端负载均衡指的是发生在服务请求的一方，也就是在发送请求之前就已经选好了由哪个实例来处理请求
### 策略
LoadBalancer内置的策略包括：
1. 轮询（RoundRobin）：默认策略，按顺序依次分配到各个实例
2. 随机（Random）：随机选择可用实例，避免单一节点压力过大
3. 区域优先（Zone-Preference）：优先调用同一区域的实例，降低网络延迟
4. 权重分配（Nacos）：基于Nacos配置的服务实例权重值分配请求比例
## 超时连接
为了避免因网络延迟或者服务端阻塞导致客户端无限等待，从而导致服务雪崩，可以采用OpenFeign的超时控制机制，用于保障系统稳定性
### 重试机制
远程调用超时失败时，还可以进行多次尝试，如果某次成功，则返回成功结果，如果多次尝试依然失败，则结束调用，返回错误
## 服务雪崩
服务雪崩指的是微服务架构中，某个服务节点因故障或资源耗尽导致调用链路级联失败，最终引发整个系统不可用的现象
### 解决方案
1. 超时处理：设置接口请求的最大等待时间，超时后释放资源并返回错误，防止客户端无限制等待。但是在高并发场景下仍可能因为资源快速耗尽而导致服务雪崩
2. 线程隔离：为每个服务或接口分配独立线程池，限制资源使用上限，为每一个服务都设置一个独立的线程池，故障被隔离在特定资源池内，避免扩散到其他服务
3. 熔断降级：通过断路器统计异常比例或慢调用比例，若超过阈值则熔断服务，后续请求直接返回预设的降级逻辑
4. 流量控制（限流）：限制接口的QPS，预防突发流量压垮服务，通过Sentinel配置单机阈值，适用于秒杀，高并发API等流量突增的场景
### Sentinel
## 分布式事务
当完成某一个业务需要横跨多个模块（即需要调用其他模块接口），或操作多个数据库时，使用声明式事务@Transactional就无法保证事务。这时候就需要使用分布式事务
### 分布式事务理论
分布式事务可以有多种分类，比如柔性事务和强一致性事务，这些事务操作会遵循一定的定理，比如CAP原理、BASE理论
### CAP原理
CAP原理包含了以下三个元素：
1. C：Consistency，一致性。任何一个读操作总是能够读到之前完成的写操作的结果，即分布式环境中，多点的数据是一致的。所有节点在同一时刻读取的数据都是最新的数据副本
2. A：Availability，可用性。好的响应性能，完全的可用性指的是在任何故障模型下，服务器都会在有限的时间内处理完成并进行相应，保证每个请求不管成功或失败都有响应，即读取的数据有可能不是最新的数据副本
3. P：Partition tolerance，分区容忍性。指的是当出现网络分区的情况时（即系统中一部分节点无法和其他节点进行通信），分离的系统也能正常运行，也就是说，系统中任意信息的丢失或失败不会影响系统的继续运作
CAP原理指的是，CAP三个元素最多只能满足两个，而对于分布式系统，分区容忍性是基本要求。因此设计分布式数据系统，就是在一致性和可用性之间去一个平衡。而对于大多数web应用，并不需要强一致性（不需要强一致性，不代表不需要一致性），因此可以牺牲一致性去换取高可用性。
### BASE理论
BASE理论指的是，Basically Available（基本可用）、Soft-state（软状态/柔性事务）、Eventual Consistency（最终一致性）。核心思想：即使无法做到强一致性，但每个业务根据自身的特点，采用适当的方式来使系统达到最终一致性
1. 基本可用BA：Basically Available。指分布式系统在出现故障的时候，允许损失部分可用性，保证核心可用。即可以提供降级服务
2. 软状态S：Soft-state。软状态指的是允许系统存在中间状态，并且该中间状态不会影响整体可用性。即允许系统在不同节点间副本同步的时候存在延时，即状态在一段时间内不同步
3. 最终一致性E：Eventually Consistency。系统中所有数据副本经过一定时间后，最终可以达到一致的状态，不需要实时保证系统数据的强一致性。最终一致性是弱一致性的一种特殊状态
### 刚柔事务
刚柔事务分为刚性事务和柔性事务。刚性事务是原子的，要么都成功要么都失败，也就是需要保障ACID理论，而柔性事务只需要保障数据最终一致即可，需遵循BASE理论。
柔性事务分为：两阶段型，补偿型，异步确保型，最大努力通知型
#### 常用事务解决方案模型
1. **2PC**，2PC即两阶段提交，2PC是一个非常经典的强一致，中心化的原子提交协议。
   中心化指的是协议中有两类节点，一个是中心化协调者节点和N个参与者节点。
   两个阶段：投票阶段和提交/执行阶段
   **1PC**：协调者向所有的参与者**发送事务预处理请求**，称之为Prepare，并开始等待各参与者的响应。各参与者节点**执行本地事务操作**，但在执行完之后并**不真正提交数据库本地事务**。如果参与者成功执行了事务操作，那么就返回给协调者Yes响应，表示事务可以执行，否则返回No，表示事务不可以执行
   **2PC**：所有的参与者反馈给协调者的信息都是Yes，就会执行事务提交操作，协调者向所有的参与者节点**发出Commit请求**。参与者接受Commit请求之后，就会正式执行本地事务Commit操作，并在完成提交之后释放整个事务执行期间占用的事务资源。
2. **3PC**，即三阶段提交，其在2PC的基础上增加了CanCommit阶段，并引入了超时机制，一旦事务参与者迟迟没有收到协调者的Commit请求，就会自动进行本地commit，这样相对有效解决了协调者单点故障的问题
   **1PC**：这个阶段类似于2PC中的第二个阶段中的Ready阶段，事务协调者向所有参与者询问**是否可以完成本次事务**，如果参与者节点认为可以完成则返回Yes，否则返回No。实际场景中，参与者节点会对自身逻辑进行事务尝试，简单来说就是检查自身状态的健康性，看有没有能力进行事务操作
   **2PC**：阶段一中，如果所有参与者都返回Yes的话，那么就会进入PreCommit阶段**进行事务预提交**。此时分布式事务协调者会向所有参与者节点发送PreCommit请求，参与者接收到后**开始执行事务操作**，并将Undo和Redo信息记录到事务日志中。参与者执行完事务操作后，就会向协调者**反馈Ack表示已经准备好提交事务**，并等待协调者下一步指令。如果阶段一中有任何一个参与者节点返回No，或者协调者在等待参与者节点反馈的过程中**超时**，整个分布式事务就会中断，协调者就会向所有参与者**发送abort请求**
   **3PC**：阶段二中如果所有参与者节点都可以进行PreCommit提交，那么协调者就会从**预提交状态转化成提交状态**。然后向所有参与者节点**发送doCommit请求**，参与者节点在收到提交请求后就会各自执行事务提交操作，并向协调者节点**反馈Ack消息**，协调者收到所有参与者的Ack消息后**完成事务**。反之，如果有一个参与者节点**未完成PreCommit的反馈或反馈超时**，那么协调者会向所有的参与者节点**发送abort请求**，从而**中断事务**
        相比于2PC而言，3PC对于协调者和参与者都设置了超时时间，而2PC只有协调者才有超时时间。这样就可以避免参与者在长时间无法与协调者节点通讯的情况下（协调者挂了），无法释放资源的问题。参与者会在超时后，自动进行本地commit从而进行释放资源
3. **TCC**：TTC又成补偿事务，其核心思想是针对每个操作都要注册一个与其对应的确认和补偿，它分为三个操作
   **Try阶段**：主要是对业务系统做检测及资源预留
   **Confirm阶段**：确认执行业务操作
   **Cancel阶段**：取消执行业务操作
4. **MQ分布式事务**：当我们对数据的强一致性要求没那么高，则可以采用MQ来实现业务的最终一致性
### Seata
在使用Seata分布式事务管理框架时，通常涉及到多个服务的协调和事务的提交/回滚，Seata通过使用全局事务ID，来管理多个服务的事务。
Seata架构中，有三个角色：
TC（Transaction Coordinator）事务协调器：Server端，要单独部署，维护全局事务的运行状态，负责协调并驱动全局事务的提交和回滚
TM（Transaction Manager）事务管理器：Client端，控制全局事务便捷，负责开启一个全局事务，并最终发起全局提交和全局回滚的决议
RM（Resource Manager）资源管理器：Client端，由业务系统集成，控制分支事务，负责分支注册，状态汇报，并接受事务协调器的指令，驱动分支（本地）事务的提交和回滚
