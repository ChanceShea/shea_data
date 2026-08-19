# Mybatis

## Mapper接口的代理对象

SqlSessionFactory读取Mybatis的配置文件（数据库连接的配置文件），然后通过SqlSessionFactory创建SqlSession对象

Mapper接口的代理对象是通过调用SqlSession的getMapper方法，通过jdk的反射，来创建其代理对象，从而可以注入到Spring容器中

## Mybatis是如何执行sql语句的，xml文件是如何转换成方法的
Mybatis启动时，会读取并解析所有的XML映射文件，为后面的SQL执行做好准备
- 构建SqlSessionFactory，Mybatis通过`SqlSessionFactoryBuilder`加载主配置文件和所有mapper文件，解析后生成一个全局唯一的`SqlSessionFactory`对象，这个工厂对象就是整个应用与数据库的连接池管理器
- 解析Mapper文件，在解析过程中，每个`<select>`，`<insert>`等SQL标签都会被解析成一个`MappedStatement`对象，这个对象封装了一条SQL的所有信息，解析完后，所有的`MappedStatement`对象都会被缓存到`Configuration`对象中，这是Mybatis的知识库，包含了所有的配置信息
- 生成代理对象，先获取Mapper代理对象，通过sqlSession.getMapper方法获取Mapper的代理对象，Mybatis会使用JDK动态代理为Mapper接口生成一个代理对象
- 调用代理对象，当调用代理对象的方法时，会被它的`MapperProxy`拦截，它会根据方法名，从之前解析好的信息中找到对应的`MappedStatement`对象，然后决定是执行查询、更新还是其他操作
- 执行SQL，`MapperProxy`最终会委托给`SqlSession`的对应方法，而`SqlSession`又会将工作交给`Executor`。`Executor`是Mybatis的调度中心，负责管理缓存并协调整个SQL的执行过程
## Mybatis是如何执行查询语句的

对于一个mapper，首先会获取到mapper的代理对象。然后调用这个mapper代理对象的方法时，都会触发`MapperProxy`的`invoke()`方法

在`MapperProxy.invoke()`中，首先会忽略Object类自身的方法，然后从缓存中获取或创建一个`MapperMethod`对象，`MapperMethod`封装了SQL命令信息和方法签名信息。最终调用`mapperMethod.executr(sqlSession,args)`方法执行SQL

`MapperMethod`会根据SQL类型调用`SqlSession`的相应方法，如`selectOne()`或`selectList()`。在SpringBoot集成中，实际调用的是`SqlSessionTemplate`，它最终将任务委托给`DefalutSqlSession`

`DefalutSqlSession`并非真实的执行者，它会将任务交给`Executor`。如果配置了二级缓存，会使用`CachingExecutor`，它首先会尝试从二级缓存中获取结果。无论是否使用二级缓存，最终都会调用`BaseExecutor`或其子类，`BaseExecutor`会先查询一级缓存，若未命中则继续向下执行

`Executor`会创建`StatementHandler`。`StatementHandler`负责使用`ParameterHandler`将用户传入的Java参数转换成JDBC可用的参数。通过JDBC的`PreparedStatement`执行SQL

数据返回`ResultSet`后，`ResultSetHandler`负责将结果集映射为Java对象并返回




