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
### 多表查询
#### 内连接
内连接只包含A、B两张表的交集部分
![](assets/MySQL/file-20260130212628093.png)
#### 左外连接
左外连接包含了左表的所有数据以及两表之间的交集的数据
![](assets/MySQL/file-20260130212708629.png)
#### 右外连接
右外连接包含了右表的所有数据以及两表之间的交集的数据
![](assets/MySQL/file-20260130212744541.png)