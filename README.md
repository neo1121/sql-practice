# SQL-Practice

收录做过的 SQL 练习，方便以后查看复习

参考资料：《MySQL 必知必会》

## 联结（join）

### 为什么要使用联结？

> 关系表的设计就是要保证把信息分解成多个表，一类数据一个表。各表通过某些常用的值（即关系设计中的关系（relational）互相关联）。
>
> 关系数据可以有效地存储和方便地处理。因此，关系数据库的可伸缩性远比非关系数据库要好。
>
> **可伸缩性（Scale）**能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好（scale well）。

在开发中，常以关系表的形式设计数据库。如果希望使用一条语句查询出被存储在多张表中的数据，就会用到联结。

联结不存在于实际的数据库表中，联结只会存在于查询的执行过程中。

### 外联结（outer join）

> 联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。

如果需要将 A、B 两张表根据某字段合并，且保留 B 表中相关字段不存在的行，应使用外联结。

1. 题目：[组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

   解决方案：`select firstName,lastName,city,state from Person left join Address on Person.personId = Address.personId;`

