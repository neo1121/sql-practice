# SQL-Practice

收录做过的 SQL 练习，方便以后查看复习

参考资料：《MySQL 必知必会》

## 题目

1. 题目：[组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

   解决方案：

   ```mysql
   select firstName,lastName,city,state from Person left join Address on Person.personId = Address.personId;
   ```

   知识点：[外联结](#外联结（outer join）)

2. 题目：[第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

   解决方案：

   ```mysql
   -- 子查询 + limit
   select (
       select distinct salary from Employee order by salary DESC limit 1 offset 1
   ) as SecondHighestSalary;
   
   -- ifnull + limit
   select ifnull(
       (select distinct salary from Employee order by salary DESC limit 1 offset 1), null
   ) as SecondHighestSalary;
   ```

   知识点：[限制结果行数](#限制结果行数（limit）)，[检索不同的行](#检索不同的行（distinct）)，[排序检索数据](#排序检索数据（order by）)

   注：如果子查询没有查找到数据，会返回 null

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

## 限制结果行数（limit）

> SELECT 语句返回所有匹配的行，它们可能是指定表中的每个行。为了返回第一行或前几行，可使用 LIMIT 子句。
>
> LIMIT 5, 5 指示 MySQL 返回从行 5 开始的 5 行。第一个数为开始位置，第二个数为要检索的行数。
>
> MySQL 5 支持 LIMIT 的另一种替代语法。LIMIT 4 OFFSET 3 意为从行 3 开始取 4 行，就像 LIMIT 3, 4 一样。

## 检索不同的行（distinct）

`distinct` 作用于所有的列，仅当整行完全相同才会去除该行。

## 排序检索数据（order by）

> 检索出的数据并不是以存粹的随机顺序显示的。如果不排序，数据一般将以它在底层表中出现的顺序显示。
>
> 关系数据库设计理论认为，如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有意义。
>
> **用非检索的列排序数据是完全合法的**

在按照多个列进行排序时，排序会完全按照所规定的顺序进行。如果前一字段在表中是唯一的，则不会再按照后面的字段排序。