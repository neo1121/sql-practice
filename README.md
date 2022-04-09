# SQL-Practice

收录做过的 SQL 练习，方便以后查看复习。

参考资料：《MySQL 必知必会》



## 题目

1. 题目：[组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

   知识点：[外联结](#外联结（outer join）)

   解决方案：

   ```mysql
   select firstName,lastName,city,state 
   from Person left join Address on Person.personId = Address.personId;
   ```

   

2. 题目：[第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

   知识点：[限制结果行数](#限制结果行数（limit）)，[检索不同的行](#检索不同的行（distinct）)，[排序检索数据](#排序检索数据（order by）)，[子查询](#子查询)

   解决方案：

   ```mysql
   -- 子查询 + limit
   select (
       select distinct salary 
       from Employee 
       order by salary DESC 
       limit 1 offset 1
   ) as SecondHighestSalary;
   
   -- ifnull + limit
   select ifnull(
       (
           select distinct salary 
        	from Employee 
        	order by salary DESC 
        	limit 1 offset 1
       ), null
   ) as SecondHighestSalary;
   ```

   

3. 题目：[分数排名](https://leetcode-cn.com/problems/rank-scores/)

   知识点：[在聚集函数中使用 distinct](#在聚集函数中使用 distinct)，[排序检索数据](#排序检索数据（order by）)，[子查询](#子查询)

   解决方案：
   
   ```mysql
   select score, (
       select count(distinct score) 
       from Scores 
       where score>=a.score
   ) as 'rank' from Scores as a order by score DESC;
   ```
   
   
   
4. 题目：[连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

   知识点：[检索不同的行](#检索不同的行（distinct）)

   解决方案：

   ```mysql
   select distinct a.Num as ConsecutiveNums 
   from Logs a, Logs b, Logs c 
   where a.Id = b.Id - 1 and a.Id = c.Id - 2 and a.Num = b.Num and a.Num = c.Num;
   ```

   

5. 题目：[超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

   知识点：[外联结](#外联结（outer join）)

   解决方案：

   ```mysql
   select a.name as Employee 
   from Employee a left join Employee b on a.managerId = b.id 
   where a.salary > b.salary;
   ```

   

6. 题目：[查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

   知识点：[检索不同的行](#检索不同的行（distinct）)

   解决方案：

   ```mysql
   select distinct a.Email 
   from Person a, Person b 
   where a.Email = b.Email and a.Id != b.Id;
   ```

   

7. 题目：[从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

   知识点：[外联结](#外联结（outer join）)

   解决方案：
   
   ```mysql
   select a.Name as Customers 
   from Customers a left join Orders b on b.CustomerId = a.Id
   where b.CustomerId is null;
   ```
   
   
   
8. 题目：[部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

   知识点：[外联结](#外联结（outer join）)，[子查询](#子查询)，[数据分组](#数据分组（group by）)，[汇总数据](#汇总数据)

   解决方案：
   
   ```mysql
   -- 将部门工资排名，再选出排名第一的行
   -- 性能较差
   select Department,Employee,Salary 
   from (
       select b.name as Department, a.name as Employee, a.salary,(
           select count(distinct salary) 
           from Employee 
           where salary>=a.salary and departmentId=a.departmentId
       ) as 'rank' 
       from Employee a left join Department b on a.departmentId = b.id
   ) c 
   where c.rank = 1;
   
   -- 用子查询查出各部门最高工资
   select Department.name as Department, Employee.name as Employee, Employee.salary as Salary
   from Employee left join Department on Employee.departmentId = Department.id
   where (salary, departmentId) in (
       select max(salary) as salary,departmentId
       from Employee
       group by departmentId
   );
   ```
   
9. 题目：[部门工资前三高的所有员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

   知识点：[外联结](#外联结（outer join）)，[子查询](#子查询)，[数据分组](#数据分组（group by）)，[汇总数据](#汇总数据)

   解决方案：

   ```mysql
   select Department, Employee, Salary
   from (
       select b.name as Department,a.name as Employee, a.salary as Salary,(
           select count(distinct salary)
           from Employee
           where Employee.salary >= a.salary and Employee.departmentId = a.departmentId
       ) as 'rank'
       from Employee a left join Department b on a.departmentId = b.id
   ) c
   where c.rank<=3;
   ```

10. 题目：[删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

   

## select 子句及其顺序

**SQL 语句中仅 select 子句是必须存在的，其他子句可按需使用**

```mysql
-- 子句书写顺序
select
from 
where 
group by
having
order by
limit
```



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



## 汇总数据

### 为什么要使用聚集函数？

> 经常需要汇总数据而不用把它们实际检索出来，为此 MySQL 提供了专门的函数。
>
> 这些函数是高校设计的，它们返回的结果一般比你在自己的客户机应用程序中计算要快得多。

### MySQL 常用的 5 个聚集函数

| 函数    | 说明             |
| ------- | :--------------- |
| AVG()   | 返回某列的平均值 |
| COUNT() | 返回某列的行数   |
| MAX()   | 返回某列的最大值 |
| MIN()   | 返回某列的最小值 |
| SUM()   | 返回某列值之和   |

此外，MySQL 还支持一系列的标准偏差聚集函数，此处省略。

### 在聚集函数中使用 distinct

**MySQL 5 及后期版本支持在聚集函数中使用 distinct**

在聚集函数中使用 distinct 时必须指定列名，且不能与计算或表达式一起使用。

> **将 DISTINCT 用于 MIN() 和 MAX()** 虽然 DISTINCT 从技术上可用于 MIN() 和 MAX()，但这样做实际上没有价值。一个列中的最小值和最大值不管是否包含不同值都是相同的。

### 组合聚集函数

> SELECT 语句可根据需要包含多个聚集函数。



## 分组数据

### 数据分组（group by）

> 分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算。
>
> GROUP BY 子句指示 MySQL 分组数据，然后对每个组而不是整个结果集进行聚集。
>
> **GROUP BY 子句必须出现在 WHERE 子句之后，ORDER BY 子句之前。**

### 过滤分组（having）

> 所有类型的 WHERE 子句都可以用 HAVING 来替代。唯一的差别是 WHERE 过滤行，而 HAVING 过滤分组。

### having 和 where 的差别？

> WHERE 在分组前过滤，HAVING 在数据分组后进行过滤。
>
> WHERE 排除的行不包括在分组中。这可能会改变计算值，从而影响 HAVING 子句中基于这些值过滤掉的分组。



## 子查询

**MySQL 4.1 及以上版本才支持子查询**

> 对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。
>
> **逐渐增加子查询来建立查询** 首先，建立和测试最内层的查询。然后，用硬编码数据建立和测试外层查询，并且仅在确认它正常后才嵌入子查询。

**如果子查询没有查询到数据，会返回 null**

### 利用子查询进行过滤

**列必须匹配** 在 where 子句中使用子查询于 in 操作符时，应确保子查询返回的列与 where 子句中的列匹配。

> 通常，子查询将返回单个列并与单个列匹配，但如果需要也可以使用多个列。

### 作为计算字段使用子查询

> **相关子查询（correlated subquery）** 涉及外部查询的子查询。任何时候只要列名可能有多义性，就必须使用这种语法（表名和列名由一个句点分隔）。

**必须注意限制有歧义性的列名** 如果 A 表与 B 表有相同的列名(id)，则在子查询中不能直接使用 `where id = id`，而必须使用 `where A.id = b.id`。

