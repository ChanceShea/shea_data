# MySQL
## SQL
SQL语句主要有以下几大类：
1. DDL：数据定义语言，用来定义数据库对象
2. DML：数据操作语言，用来对数据库表中的数据进行增删改
3. DQL：数据查询语言，用来查询数据库表中的巨鹿
4. DCL：数据控制语言，用来创建数据库用户，控制数据库的访问权限
### 聚合函数
聚合函数是将一列数据做为一个整体，进行纵向计算（null值不会参与聚合函数的运算）
常见的聚合函数有：count、max、min、avg、sum
#### 分组查询
**where和having的区别**：where是分组之前进行过滤，不满足where条件的不参与分组，having是分组之后对结果进行过滤；where不能对聚合函数进行判断，而having可以
```mysql
# 查询年龄小于45岁的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址
select workaddress,count(*) address_count from emp where age < 45 group by workaddress having address_count >= 3;
```
