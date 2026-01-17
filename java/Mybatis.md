# Mybatis

## Mapper接口的代理对象

SqlSessionFactory读取Mybatis的配置文件（数据库连接的配置文件），然后通过SqlSessionFactory创建SqlSession对象

Mapper接口的代理对象是通过调用SqlSession的getMapper方法，通过jdk的反射，来创建其代理对象，从而可以注入到Spring容器中










