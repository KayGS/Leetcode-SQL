# 数学判断
__奇偶数__
- mod(id,2) <> 0
- In the SQL script you provided, the % symbol is being used as the modulo operator.
  In the CASE statement, id % 2 is used to determine if the id is odd or even. When id % 2 = 1, it means that the id is odd. Here's how it works:
  id % 2 calculates the remainder when the id is divided by 2.
  If the remainder is 1, it means the id is odd.
  If the remainder is 0, it means the id is even.
  
__累加__
- mysql running tot
  sum(weight)over(order by turn)
  
__移动平均__
- 比如要求是这样的paid in a seven days window (i.e., current day + 6 days before)
```sql
SELECT
    visited_on, 
    ROUND(sum(amount) OVER(ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS amount,
    ROUND(AVG(amount) OVER(ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS average_amount
FROM (select visited_on, sum(amount) as amount from customer group by visited_on ) as a
order by visited_on
OFFSET 6 ROWS;
```

# 排序判断
- rank()over()
  
  计算排名并跳过相同排名的下一个值。如果多个行具有相同的排序条件，则它们将被分配相同的排名，并且下一个行将跳过相同排名的行数.
  
  1，2，2，4，4，5......

- dense_rank()over()
  
  但它不跳过相同排名的下一个值。即使有相同排名的行，下一个行也会继续使用下一个整数值。
  
  1，2，2，3，3，4......

- row_number()
  
  为每一行分配一个唯一的整数值，该值基于指定的排序顺序。如果有相同排序条件的行，则会为它们分配不同的排名。
  
  1，2，3，4，5，1，2，3，4......

- NTILE(n)
  
  函数将结果集分成 n 个桶，并为每个行分配一个桶号。如果行的数量不能整除 n，则前面的桶将比后面的桶多一个行。

- row_number()over(order by visited_on) as rk 开窗函数里也可以直接写order by 不需要 partition


# 语句嵌套
- ```sql
  where (product_id, change_date) in
  (select product_id, max(change_date) as last_change
  ```


# 其他
- __Common Table Expression (CTE)__

是一种临时命名的结果集，它可以在一个查询中定义，并且可以像表一样在后续查询中引用。CTE 可以使复杂的查询更具可读性和可维护性，尤其是当查询需要多次引用相同的子查询时。
一般情况下，CTE 由两部分组成：
1. **WITH 子句**：用于定义 CTE 的名称以及查询 CTE 的内容。通常以 `WITH` 关键字开头，后跟 CTE 的名称和查询定义。
2. **主查询**：在 `WITH` 子句后面使用 CTE，可以像使用任何表一样引用 CTE，进行进一步的数据操作和筛选。
例如，以下是一个简单的示例，展示了如何使用 CTE 查找某个表中某列的平均值：
```sql
WITH AvgSalary AS (
    SELECT AVG(salary) AS avg_salary
    FROM employees
)
SELECT *
FROM employees
WHERE salary > (SELECT avg_salary FROM AvgSalary);
```


- __交换上下两行__

要交换SQL查询结果中的上下两行，可以使用窗口函数和条件语句实现。以下是一种通用的方法，假设表中有一个唯一标识行的列（例如自增ID或时间戳），可以利用这一列来确定要交换的行：

```sql
WITH RowNumbered AS (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY id) AS row_num
    FROM your_table
)
SELECT 
    CASE 
        WHEN r1.row_num = 1 THEN r2.*
        WHEN r2.row_num = 1 THEN r1.*
        ELSE your_table.*
    END
FROM RowNumbered r1
JOIN RowNumbered r2 ON r1.row_num = r2.row_num - 1;
```
这段代码中：
 `RowNumbered` CTE 给每行添加了一个 `row_num` 列，表示行的顺序。
 `SELECT` 语句使用 `CASE` 表达式和 `JOIN` 条件，根据 `row_num` 的值选择上下两行进行交换。
请注意，具体的表名、唯一标识列（如 `id`）、以及具体的排序逻辑需根据实际情况进行调整。

- Length（）
- The % wildcard represents any number of characters, even zero characters
  WHERE conditions LIKE 'DIAB1%' OR conditions LIKE '% DIAB1%'
- ```sql
  DELETE FROM person
  WHERE id NOT IN (
    SELECT id
    FROM (
        SELECT MIN(id) AS id
        FROM person
        GROUP BY email
    ) AS t
  );
  ```
delete 语句不能直接from选择，可用临时表来fromThe error you're encountering is likely due to MySQL's restriction on modifying the same table that you're selecting from in a subquery within the DELETE statement. To work around this, you can use a temporary table or a derived table.
- `LIMIT`：指定返回的行数。
  `OFFSET`：指定从结果集中的第几行开始返回数据，行数从0开始计数（即第一行是0，第二行是1，依此类推）。
   `OFFSET` 的值可以是一个正整数或者是表达式，它决定了查询从结果集的哪一行开始返回数据。
   如果不使用 `OFFSET`，则默认从结果集的第一行开始返回数据。
   使用 `LIMIT` 和 `OFFSET` 时，要确保查询结果的顺序是可预测的，通常需要通过 `ORDER BY` 来指定排序条件。
- `GROUP_CONCAT` 函数用于将组内的多行值连接成一个单个字符串，并以指定的分隔符分隔这些值。在不同的数据库管理系统中，语法和用法可能有所不同，我将为你总结一些常见数据库系统中 `GROUP_CONCAT` 的用法。
   ![image](https://github.com/KayGS/SQL-notes/assets/52340945/7498464e-620c-471a-a417-9a7f6662e744)
- REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode\\.com$': This regular expression pattern checks if the email address meets the specified criteria.

  ^: Start of the string.

  [a-zA-Z]: Matches any single letter (upper or lower case) as the first character.

  [a-zA-Z0-9_.-]*: Matches any combination (zero or more occurrences) of letters (upper or lower case), digits, underscore '_', period '.', and/or dash '-' after the first character.

  @leetcode\\.com: Matches the domain '@leetcode.com'.

  $: End of the string.

# 时间差表达
- ```sql
  DATE_SUB(yesterday.recordDate, INTERVAL 1 DAY)
  ```
- ```sql
  DATEADD(interval, number, date)
  ```
- ```sql
  a.recordDate = b.recordDate -1`
  ```
- ```sql
  a.event_date = date_add(b.event_date, interval -1 day)
  ```
- *mysql -> date_add*

# Join考点
- Cross Join，通过交叉连接获得所有可能的
- Full Join, 返回左表和右表中的所有行，如果某行在左表中没有匹配，则右表的列为 NULL；如果某行在右表中没有匹配，则左表的列为 NULL。
- LEFT JOIN 返回左表中所有行，以及右表中与左表中行匹配的行。如果右表中没有匹配的行，则为 NULL。
*小表left join大表提高运行效率*

# 逻辑判断
- Case when…… and…… then
  Else……
  End as
- ```sql
  ifnull(round(avg(b.experience_years),2),0)
  ```
- IF(condition, value_if_true, value_if_false)
- select class
  from Courses
  group by class
  having count(student) >= 5 *可以在having里count*


# 聚合函数
- ```sql
  sum(case when action = 'confirmed' then 1 else 0 end) / count(*)
  ```
- max() 不需要group
- ```sql
  having count(distinct product_key) = (select count(1) from product)
  ```
  having 语句count可以直接连 （select ......）
- order by 里可以直接写count（），聚合函数




