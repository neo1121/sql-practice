# SQL-Practice

收录做过的 SQL 练习，方便以后查看复习。

参考资料：《MySQL 必知必会》



## 题目

1. 题目：[组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

   知识点：[外联结](#外联结（OUTER JOIN）)

   解决方案：

   ```mysql
   SELECT firstName,lastName,city,state 
   FROM Person LEFT JOIN Address ON Person.personId = Address.personId;
   ```

   

2. 题目：[第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

   知识点：[限制结果行数](#限制结果行数（LIMIT）)，[检索不同的行](#检索不同的行（DISTINCT）)，[排序检索数据](#排序检索数据（ORDER BY）)，[子查询](#子查询)

   解决方案：

   ```mysql
   -- 子查询 + LIMIT
   SELECT (
       SELECT DISTINCT salary 
       FROM Employee 
       ORDER BY salary DESC 
       LIMIT 1 OFFSET 1
   ) AS SecondHighestSalary;
   
   -- IFNULL + LIMIT
   SELECT IFNULL(
       (
           SELECT DISTINCT salary 
        	FROM Employee 
        	ORDER BY salary DESC 
        	LIMIT 1 OFFSET 1
       ), NULL
   ) AS SecondHighestSalary;
   ```

   

3. 题目：[分数排名](https://leetcode-cn.com/problems/rank-scores/)

   知识点：[在聚集函数中使用 DISTINCT](#在聚集函数中使用 DISTINCT)，[排序检索数据](#排序检索数据（ORDER BY）)，[子查询](#子查询)

   解决方案：
   
   ```mysql
   SELECT score, (
       SELECT COUNT(DISTINCT score) 
       FROM Scores 
       WHERE score>=a.score
   ) AS 'rank' FROM Scores AS a ORDER BY score DESC;
   ```
   
   
   
4. 题目：[连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

   知识点：[检索不同的行](#检索不同的行（DISTINCT）)

   解决方案：

   ```mysql
   SELECT DISTINCT a.Num AS ConsecutiveNums 
   FROM Logs a, Logs b, Logs c 
   WHERE a.Id = b.Id - 1 AND a.Id = c.Id - 2 AND a.Num = b.Num AND a.Num = c.Num;
   ```

   

5. 题目：[超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

   知识点：[外联结](#外联结（outer JOIN）)

   解决方案：

   ```mysql
   SELECT a.name AS Employee 
   FROM Employee a LEFT JOIN Employee b ON a.managerId = b.id 
   WHERE a.salary > b.salary;
   ```

   

6. 题目：[查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

   知识点：[检索不同的行](#检索不同的行（DISTINCT）)

   解决方案：

   ```mysql
   SELECT DISTINCT a.Email 
   FROM Person a, Person b 
   WHERE a.Email = b.Email AND a.Id != b.Id;
   ```

   

7. 题目：[从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

   知识点：[外联结](#外联结（OUTER JOIN）)

   解决方案：
   
   ```mysql
   SELECT a.Name AS Customers 
   FROM Customers a LEFT JOIN Orders b ON b.CustomerId = a.Id
   WHERE b.CustomerId is NULL;
   ```
   
   
   
8. 题目：[部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

   知识点：[外联结](#外联结（OUTER JOIN）)，[子查询](#子查询)，[数据分组](#数据分组（GROUP BY）)，[汇总数据](#汇总数据)

   解决方案：
   
   ```mysql
   -- 将部门工资排名，再选出排名第一的行
   -- 性能较差
   SELECT Department,Employee,Salary 
   FROM (
       SELECT b.name AS Department, a.name AS Employee, a.salary,(
           SELECT COUNT(DISTINCT salary) 
           FROM Employee 
           WHERE salary>=a.salary AND departmentId=a.departmentId
       ) AS 'rank' 
       FROM Employee a LEFT JOIN Department b ON a.departmentId = b.id
   ) c 
   WHERE c.rank = 1;
   
   -- 用子查询查出各部门最高工资
   SELECT Department.name AS Department, Employee.name AS Employee, Employee.salary AS Salary
   FROM Employee LEFT JOIN Department ON Employee.departmentId = Department.id
   WHERE (salary, departmentId) IN (
       SELECT MAX(salary) AS salary,departmentId
       FROM Employee
       GROUP BY departmentId
   );
   ```
   
9. 题目：[部门工资前三高的所有员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

   知识点：[外联结](#外联结（OUTER JOIN）)，[子查询](#子查询)，[数据分组](#数据分组（GROUP BY）)，[汇总数据](#汇总数据)

   解决方案：

   ```mysql
   SELECT Department, Employee, Salary
   FROM (
       SELECT b.name AS Department,a.name AS Employee, a.salary AS Salary,(
           SELECT COUNT(DISTINCT salary)
           FROM Employee
           WHERE Employee.salary >= a.salary AND Employee.departmentId = a.departmentId
       ) AS 'rank'
       FROM Employee a LEFT JOIN Department b ON a.departmentId = b.id
   ) c
   WHERE c.rank<=3;
   ```

10. 题目：[删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

   知识点：

   解决方案：

   ```mysql
   DELETE
   FROM Person
   WHERE Person.id IN (
       SELECT id 
       FROM (
           SELECT a.id
           FROM Person a, Person b
           WHERE a.email = b.email AND a.id > b.id
       ) c
   );
   ```

11. 题目：[上升的温度](https://leetcode-cn.com/problems/rising-temperature/)

    知识点：[日期和时间处理函数](#日期和时间处理函数)

    解决方案：

    ```mysql
    -- DateDiff() 函数计算两个日期之差
    SELECT w1.id
    FROM Weather w1, Weather w2
    WHERE DateDiff(w1.recordDate, w2.recordDate) = 1 and w1.Temperature > w2.Temperature;
    ```

12. 题目：[行程和用户](https://leetcode-cn.com/problems/trips-and-users/)

    知识点：[外联结](#外联结（OUTER JOIN）)，[子查询](#子查询)，[数值处理函数](#数值处理函数)

    解决方案：

    ```mysql
    -- 禁止用户的id
    # SELECT users_id
    # FROM Users
    # WHERE banned = 'Yes';
    
    -- 非禁止用户生成的订单总数
    # SELECT request_at, COUNT(*)
    # FROM Trips
    # WHERE client_id NOT IN (
    #     SELECT users_id
    #     FROM Users
    #     WHERE banned = 'Yes'
    # ) AND driver_id NOT IN (
    #     SELECT users_id
    #     FROM Users
    #     WHERE banned = 'Yes'
    # )
    # GROUP BY request_at;
    
    -- 被司机或乘客取消的非禁止用户生成的订单数量
    # SELECT request_at, COUNT(*)
    # FROM Trips
    # WHERE client_id NOT IN (
    #     SELECT users_id
    #     FROM Users
    #     WHERE banned = 'Yes'
    # ) AND driver_id NOT IN (
    #     SELECT users_id
    #     FROM Users
    #     WHERE banned = 'Yes'
    # ) AND status like 'cancelled%'
    # GROUP BY request_at;
    
    -- 完整题解
    SELECT b.request_at AS Day, ROUND(IFNULL(a.cnt,0)/b.cnt,2) AS 'Cancellation Rate'
    FROM (
        SELECT request_at, COUNT(*) AS cnt
        FROM Trips
        WHERE client_id NOT IN (
            SELECT users_id
            FROM Users
            WHERE banned = 'Yes'
        ) AND driver_id NOT IN (
            SELECT users_id
            FROM Users
            WHERE banned = 'Yes'
        ) AND status LIKE 'cancelled%'
        GROUP BY request_at
    ) a RIGHT JOIN (
        SELECT request_at, COUNT(*) AS cnt
        FROM Trips
        WHERE client_id NOT IN (
            SELECT users_id
            FROM Users
            WHERE banned = 'Yes'
        ) AND driver_id NOT IN (
            SELECT users_id
            FROM Users
            WHERE banned = 'Yes'
        )
        GROUP BY request_at
    ) b ON a.request_at = b.request_at
    WHERE b.request_at BETWEEN '2013-10-01' AND '2013-10-03';
    ```

    

## SELECT 子句及其顺序

**SQL 语句中仅 SELECT 子句是必须存在的，其他子句可按需使用**

```mysql
-- 子句书写顺序
SELECT
FROM 
WHERE 
GROUP BY
HAVING
ORDER BY
LIMIT
```



## 联结（JOIN）

### 为什么要使用联结？

> 关系表的设计就是要保证把信息分解成多个表，一类数据一个表。各表通过某些常用的值（即关系设计中的关系（relational）互相关联）。
>
> 关系数据可以有效地存储和方便地处理。因此，关系数据库的可伸缩性远比非关系数据库要好。
>
> **可伸缩性（Scale）**能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好（scale well）。

在开发中，常以关系表的形式设计数据库。如果希望使用一条语句查询出被存储在多张表中的数据，就会用到联结。

联结不存在于实际的数据库表中，联结只会存在于查询的执行过程中。

### 外联结（OUTER JOIN）

> 联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。

如果需要将 A、B 两张表根据某字段合并，且保留 B 表中相关字段不存在的行，应使用外联结。



## 限制结果行数（LIMIT）

> SELECT 语句返回所有匹配的行，它们可能是指定表中的每个行。为了返回第一行或前几行，可使用 LIMIT 子句。
>
> LIMIT 5, 5 指示 MySQL 返回从行 5 开始的 5 行。第一个数为开始位置，第二个数为要检索的行数。
>
> MySQL 5 支持 LIMIT 的另一种替代语法。LIMIT 4 OFFSET 3 意为从行 3 开始取 4 行，就像 LIMIT 3, 4 一样。



## 检索不同的行（DISTINCT）

`DISTINCT` 作用于所有的列，仅当整行完全相同才会去除该行。



## 排序检索数据（ORDER BY）

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
> 这些函数是高效设计的，它们返回的结果一般比你在自己的客户机应用程序中计算要快得多。

### MySQL 常用的 5 个聚集函数

| 函数    | 说明             |
| ------- | :--------------- |
| AVG()   | 返回某列的平均值 |
| COUNT() | 返回某列的行数   |
| MAX()   | 返回某列的最大值 |
| MIN()   | 返回某列的最小值 |
| SUM()   | 返回某列值之和   |

此外，MySQL 还支持一系列的标准偏差聚集函数，此处省略。

### 在聚集函数中使用 DISTINCT

**MySQL 5 及后期版本支持在聚集函数中使用 DISTINCT**

在聚集函数中使用 DISTINCT 时必须指定列名，且不能与计算或表达式一起使用。

> **将 DISTINCT 用于 MIN() 和 MAX()** 虽然 DISTINCT 从技术上可用于 MIN() 和 MAX()，但这样做实际上没有价值。一个列中的最小值和最大值不管是否包含不同值都是相同的。

### 组合聚集函数

> SELECT 语句可根据需要包含多个聚集函数。



## 分组数据

### 数据分组（GROUP BY）

> 分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算。
>
> GROUP BY 子句指示 MySQL 分组数据，然后对每个组而不是整个结果集进行聚集。
>
> **GROUP BY 子句必须出现在 WHERE 子句之后，ORDER BY 子句之前。**

### 过滤分组（HAVING）

> 所有类型的 WHERE 子句都可以用 HAVING 来替代。唯一的差别是 WHERE 过滤行，而 HAVING 过滤分组。

### HAVING 和 WHERE 的差别？

> WHERE 在分组前过滤，HAVING 在数据分组后进行过滤。
>
> WHERE 排除的行不包括在分组中。这可能会改变计算值，从而影响 HAVING 子句中基于这些值过滤掉的分组。



## 子查询

**MySQL 4.1 及以上版本才支持子查询**

> 对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。
>
> **逐渐增加子查询来建立查询** 首先，建立和测试最内层的查询。然后，用硬编码数据建立和测试外层查询，并且仅在确认它正常后才嵌入子查询。

**如果子查询没有查询到数据，会返回 NULL**

### 利用子查询进行过滤

**列必须匹配** 在 WHERE 子句中使用子查询于 IN 操作符时，应确保子查询返回的列与 WHERE 子句中的列匹配。

> 通常，子查询将返回单个列并与单个列匹配，但如果需要也可以使用多个列。

### 作为计算字段使用子查询

> **相关子查询（correlated subquery）** 涉及外部查询的子查询。任何时候只要列名可能有多义性，就必须使用这种语法（表名和列名由一个句点分隔）。

**必须注意限制有歧义性的列名** 如果 A 表与 B 表有相同的列名(id)，则在子查询中不能直接使用 `WHERE id = id`，而必须使用 `WHERE A.id = b.id`。



## 数据处理函数

**以下只列出常用函数**

### 文本处理函数

| 函数        | 说明              |
| ----------- | ----------------- |
| Left()      | 返回串左边的字符  |
| Right()     | 返回串右边的字符  |
| Length()    | 返回串的长度      |
| Locate()    | 找出串的一个子串  |
| LTrim()     | 去掉串左边的空格  |
| RTrim()     | 去掉串右边的空格  |
| Soundex()   | 返回串的SOUNDEX值 |
| SubString() | 返回子串的字符    |
| Lower()     | 将串转换为小写    |
| Upper()     | 将串转换为大写    |

### 日期和时间处理函数

| 函数          | 说明                           |
| ------------- | ------------------------------ |
| AddDate()     | 增加一个日期（天、周等）       |
| AddTime()     | 增加一个时间（时、分等）       |
| CurDate()     | 返回当前日期                   |
| CurTime()     | 返回当前时间                   |
| Now()         | 返回当前日期和时间             |
| Date()        | 返回日期时间的日期部分         |
| Year()        | 返回一个日期的年份部分         |
| Month()       | 返回一个日期的月份部分         |
| Day()         | 返回一个日期的天数部分         |
| DayOfWeek()   | 对于一个日期，返回对应的星期几 |
| Time()        | 返回一个日期时间的时间部分     |
| Hour()        | 返回一个时间的小时部分         |
| Minute()      | 返回一个时间的分钟部分         |
| Second()      | 返回一个时间的秒部分           |
| DateDiff()    | 计算两个日期之差               |
| Date_Add()    | 高度灵活的日期运算函数         |
| Date_Format() | 返回一个格式化的日期或时间串   |

### 数值处理函数

| 函数        | 说明                     |
| ----------- | ------------------------ |
| Cos()       | 返回一个角度的余弦       |
| Sin()       | 返回一个角度的正弦       |
| Tan()       | 返回一个角度的正切       |
| Abs()       | 返回一个数的绝对值       |
| Exp()       | 返回 e (自然对数) 的幂   |
| Mod()       | 返回除操作的余数         |
| Pi()        | 返回圆周率               |
| Sqrt()      | 返回一个数的平方根       |
| Rand()      | 返回一个随机数           |
| Round(x, d) | 将 x 四舍五入为 d 位小数 |

