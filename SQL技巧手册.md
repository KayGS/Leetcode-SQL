# 1. SQL操作
   #### 涵盖SELECT、WHERE、JOIN、GROUP BY、HAVING、LIMIT、OFFSET,GROUP_CONTACT等进阶操作。
    
    
**> SELECT 中使用聚合函数和逻辑判断函数**

````SQL
select
round(sum(case when imm !='0' then 1 else 0 end) / count(*),2) as immediate_percentage
# from (
# select customer_id, 
        sum(case when order_date = customer_pref_delivery_date then 1 else 0 end) as imm
#    from delivery 
#    group by customer_id
# ) as a
````

**> WHERE 语句嵌套**

```SQL
where (product_id, change_date) in
(select product_id, max(change_date) as last_change
```


**> JOIN 的类别**

- Cross Join，通过交叉连接获得所有可能的

- Full Join, 返回左表和右表中的所有行，如果某行在左表中没有匹配，则右表的列为 NULL；如果某行在右表中没有匹配，则左表的列为 NULL。

- LEFT JOIN 返回左表中所有行，以及右表中与左表中行匹配的行。如果右表中没有匹配的行，则为 NULL。 小表left join大表提高运行效率



**> GROUP_CONTACT 处理特殊组合**

- `GROUP_CONCAT` 是 MySQL 中的一个聚合函数，用于将组内的多个值连接成一个字符串。它在生成报告或需要将多个行数据合并为一个字符串时非常有用。下面是`GROUP_CONCAT`的用法示例：

> 示例表结构
假设我们有一张名为 `employees` 的表，结构如下：

| employee_id | employee_name | department_id |
|-------------|---------------|---------------|
| 1           | Alice         | 101           |
| 2           | Bob           | 101           |
| 3           | Charlie       | 102           |
| 4           | David         | 101           |
| 5           | Eve           | 102           |

> 基本用法
我们希望获取每个部门的员工姓名列表：

```sql
SELECT department_id, GROUP_CONCAT(employee_name) AS employee_names
FROM employees
GROUP BY department_id;
```

 输出结果：
| department_id | employee_names      |
|---------------|---------------------|
| 101           | Alice,Bob,David     |
| 102           | Charlie,Eve         |

> 使用分隔符
你可以使用 `SEPARATOR` 关键字来自定义连接字符串的分隔符。比如，我们希望用 `|` 来分隔员工姓名：

```sql
SELECT department_id, GROUP_CONCAT(employee_name SEPARATOR '|') AS employee_names
FROM employees
GROUP BY department_id;
```

> 输出结果：
 
| department_id | employee_names      |
|---------------|---------------------|
| 101           | Alice|Bob|David     |
| 102           | Charlie|Eve         |

> 按特定顺序排序
> 我们还可以通过在 `GROUP_CONCAT` 中使用 `ORDER BY` 来指定字符串连接的顺序。例如，将员工姓名按字母顺序排序：

```sql
SELECT department_id, GROUP_CONCAT(employee_name ORDER BY employee_name ASC) AS employee_names
FROM employees
GROUP BY department_id;
```

 输出结果：
| department_id | employee_names      |
|---------------|---------------------|
| 101           | Alice,Bob,David     |
| 102           | Charlie,Eve         |

>  限制结果长度
`GROUP_CONCAT` 的结果长度有默认的最大值，可以通过 `SET` 语句调整：

```sql
SET SESSION group_concat_max_len = 10000;
```



**> ORDER BY 处理特殊排序**

````SQL
ORDER BY 
    CASE 
        WHEN 订单数 >= 0 AND 订单数 <= 2 THEN 1
        WHEN 订单数 >= 3 AND 订单数 <= 5 THEN 2
        WHEN 订单数 > 5 THEN 3
        ELSE 4 
    END
````

````SQL
SELECT customer_id, COUNT(order_id) AS order_count
FROM orders
GROUP BY customer_id
ORDER BY COUNT(order_id) DESC;
````

**> HAVING 处理特殊筛选条件**

- BUY A and B ，NOT C 找出既购买过 ProductA 和 ProductB，但没有购买过 ProductC 的顾客

````SQL
SELECT 顾客号
FROM 销售订单表
GROUP BY 顾客号
HAVING 
    -- 确保顾客购买过 ProductA
    SUM(CASE WHEN 产品 = 'ProductA' THEN 1 ELSE 0 END) > 0 AND
    -- 确保顾客购买过 ProductB
    SUM(CASE WHEN 产品 = 'ProductB' THEN 1 ELSE 0 END) > 0 AND
    -- 确保顾客没有购买过 ProductC
    SUM(CASE WHEN 产品 = 'ProductC' THEN 1 ELSE 0 END) = 0;
````
- HVING中加判断条件

````SQL
having count(distinct product_key) = (select count(1) from product)
````

**> LIMIT**

- 指定返回的行数

**> OFFSET**

- 指定从结果集中的第几行开始返回数据，行数从0开始计数（即第一行是0，第二行是1，依此类推）。 OFFSET 的值可以是一个正整数或者是表达式，它决定了查询从结果集的哪一行开始返回数据。 如果不使用 OFFSET，则默认从结果集的第一行开始返回数据。 使用 LIMIT 和 OFFSET 时，要确保查询结果的顺序是可预测的，通常需要通过 ORDER BY 来指定排序条件。


---



# 2. **窗口函数**
   #### 详细解析窗口函数的基本语法、使用场景以及常见的问题类型。

  - ### 窗口函数允许你在不改变结果集的行数的情况下，对数据进行聚合计算，你可能需要在显示详细数据的同时，添加额外的统计信息

  - ### 在 SQL 中使用窗口函数（开窗函数）时，通常不需要使用 GROUP BY 子句。窗口函数的作用是对查询结果的某一部分进行排序、分区和聚合，通常用于计算行的排名、移动平均、累计总和等。与 GROUP BY 不同，窗口函数不会将结果按组汇总，而是对每一行应用特定的计算。

  - ### 窗口函数合集

### > **`sum()over()`**
分层累加

```sql
sum（）over（partition by...... order by）
```

### > **`count()over()`**
分层聚合

| employee_id | team_id |
|:-------- |:--------:| 
| 1           | 8       |
| 2           | 8       |
| 3           | 8       |
| 4           | 7       |
| 5           | 9       |
| 6           | 9       |

| employee_id | team_size |
|:-------- |:--------:| 
| 1           | 3         |
| 2           | 3         |
| 3           | 3         |
| 4           | 1         |
| 5           | 2         |
| 6           | 2         |
```MySQL []
select employee_id, count(employee_id)over(partition by team_id) as team_size
from employee
group by employee_id
order by employee_id
````


### > **`ROW_NUMBER()`**
`ROW_NUMBER()` 函数返回当前行在其窗口分区中的序号，排序依据由 `ORDER BY` 子句决定。

为每一行分配一个唯一的整数值，该值基于指定的排序顺序。如果有相同排序条件的行，则会为它们分配不同的排名。

1，2，3，4，5，1，2，3，4......

```sql
SELECT 
  id,
  username,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY created_at) AS row_num
FROM 
  users;
```

### > **`RANK()`**
`RANK()` 函数为每一行返回其排序后的排名，相同排名的行将共享相同的排名值，之后的排名会跳过（即不连续）。

计算排名并跳过相同排名的下一个值。如果多个行具有相同的排序条件，则它们将被分配相同的排名，并且下一个行将跳过相同排名的行数.

1，2，2，4，4，5......

```sql
SELECT 
  id,
  username,
  RANK() OVER (PARTITION BY department ORDER BY created_at) AS rank
FROM 
  users;
```

### > **`DENSE_RANK()`**
`DENSE_RANK()` 类似于 `RANK()`，但相同排名的行共享相同的排名值，且后续排名不会跳过（即连续）。

但它不跳过相同排名的下一个值。即使有相同排名的行，下一个行也会继续使用下一个整数值。

1，2，2，3，3，4......

`dense_rank()`比`rank()`计算快一点

```sql
SELECT 
  id,
  username,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY created_at) AS dense_rank
FROM 
  users;
```

### > **`NTILE(n)`**
`NTILE(n)` 函数将行按照排序结果分成 `n` 个几乎相等的桶（组），每个桶有一个桶编号。

```sql
SELECT 
  id,
  username,
  NTILE(4) OVER (ORDER BY created_at) AS bucket
FROM 
  users;
```

### > **`LEAD()`**
`LEAD()` 函数允许你访问位于当前行之后的行数据。它通常用于比较当前行与后续行的数据。

LEAD 函数访问当前行之后的第1行的 log_id 值。

OVER (ORDER BY log_id) 指定了如何对行进行排序，以便确定哪一行是下一行。

LEAD(visit_date, 1, '2021-01-01') OVER (PARTITION BY user_id ORDER BY visit_date) AS next_visit_date

使用 LEAD 函数获取每个用户的下一个访问日期。如果没有下一个日期，则默认设置为 '2021-01-01'。

```sql
SELECT 
  id,
  username,
  created_at,
  LEAD(username, 1) OVER (ORDER BY created_at) AS next_user
FROM 
  users;
```

### >  **`LAG()`**
`LAG()` 函数与 `LEAD()` 类似，但用于访问当前行之前的行数据。

LAG(log_id) OVER (ORDER BY log_id) AS prev_log_id 返回前一行的值

使用 LAG 函数获取每个用户的上一个访问日期。如果没有上一个日期，则默认设置为 '2020-01-01'（这可以设置为一个早于所有访问日期的日期）。

LAG(visit_date, 1, '2020-01-01') OVER (PARTITION BY user_id ORDER BY visit_date) 用来获取每个用户的上一个访问日期。

```sql
SELECT 
  id,
  username,
  created_at,
  LAG(username, 1) OVER (ORDER BY created_at) AS previous_user
FROM 
  users;
```

### >  **`FIRST_VALUE()`**
`FIRST_VALUE()` 函数返回窗口分区内按指定排序的第一行的值。

```sql
SELECT 
  id,
  username,
  FIRST_VALUE(username) OVER (ORDER BY created_at) AS first_user
FROM 
  users;
```

### >  **`LAST_VALUE()`**
`LAST_VALUE()` 函数返回窗口分区内按指定排序的最后一行的值。

```sql
SELECT 
  id,
  username,
  LAST_VALUE(username) OVER (ORDER BY created_at) AS last_user
FROM 
  users;
```

### > **`CUME_DIST()`**
`CUME_DIST()`（累积分布）计算小于等于当前行值的行数的比例。返回值在 0 到 1 之间。

```sql
SELECT 
  id,
  username,
  CUME_DIST() OVER (ORDER BY created_at) AS cum_dist
FROM 
  users;
```

### >  **`PERCENT_RANK()`**
`PERCENT_RANK()` 计算当前行在其窗口分区中的百分排名，返回值在 0 到 1 之间。

```sql
SELECT 
  id,
  username,
  PERCENT_RANK() OVER (ORDER BY created_at) AS percent_rank
FROM 
  users;
```

- ### 如果需要额外统计信息，比如累加，需要把所有列都加在over（）分区里， 如果不这样写，不会按行累加，而是在相同数值时直接返还累加后的相同数据

````sql
select 学号,课程号,成绩, sum(成绩)over( order by 成绩 desc,学号,课程号) as 累计成绩
from 学生成绩表
order by 成绩 desc, 学号
````

| 学号   | 课程号  | 成绩 | 累计成绩 |
| ---- | ---- | -- | ---- |
| 1001 | 1003 | 99 | 99   |
| 1001 | 1002 | 90 | 189  |
| 1001 | 1001 | 80 | 269  |
| 1002 | 1003 | 80 | 349  |
| 1003 | 1001 | 80 | 429  |
| 1003 | 1002 | 80 | 509  |

这样写的结果对于80的时候，会按行累加

````sql
select 学号,课程号,成绩, sum(成绩)over( order by 成绩 desc,学号) as 累计成绩
from 学生成绩表
order by 成绩 desc, 学号
````

| 学号   | 课程号  | 成绩 | 累计成绩 |
| ---- | ---- | -- | ---- |
| 1001 | 1003 | 99 | 99   |
| 1001 | 1002 | 90 | 189  |
| 1001 | 1001 | 80 | 269  |
| 1002 | 1003 | 80 | 349  |
| 1003 | 1001 | 80 | 589  |
| 1003 | 1002 | 80 | 589  |
| 1003 | 1003 | 80 | 589  |
| 1002 | 1002 | 60 | 649  |

这样写在成绩80那里不会按行累加而是直接返还相同数据

---


# 3. **Common Table Expression (CTE)**
  #### 包含普通CTE,递归CTE的原理和使用场景，CTE实例。

```sql
with A as (),
	B as ()
		select
```

### >  普通CTE

#### Common Table Expression (CTE)是一种临时命名的结果集，它可以在一个查询中定义，并且可以像表一样在后续查询中引用。CTE 可以使复杂的查询更具可读性和可维护性，尤其是当查询需要多次引用相同的子查询时。 一般情况下，CTE 由两部分组成：

- WITH 子句：用于定义 CTE 的名称以及查询 CTE 的内容。通常以 WITH 关键字开头，后跟 CTE 的名称和查询定义。

- 主查询：在 WITH 子句后面使用 CTE，可以像使用任何表一样引用 CTE，进行进一步的数据操作和筛选。 例如，以下是一个简单的示例，展示了如何使用 CTE 查找某个表中某列的平均值：

```sql
WITH AvgSalary AS (
    SELECT AVG(salary) AS avg_salary
    FROM employees
)
SELECT *
FROM employees
WHERE salary > (SELECT avg_salary FROM AvgSalary);
```



### >  普通CTE和递归CTE的区别

-  普通CTE：只用于一次性查询，并不自引用。也就是说，它生成的结果集不会再进行递归扩展。

-  递归CTE：允许CTE自引用，即在CTE的定义中可以引用自身，从而进行递归计算。递归CTE可以重复执行查询逻辑，直到不再满足递归条件。

### >  递归CTE

```sql 
WITH RECURSIVE ..... as （）
```

递归CTE允许你创建一个自引用的CTE，它可以用来执行递归查询。递归CTE由两个部分组成：

-  锚定成员（基础情况）：这是递归的起点。它定义了递归CTE的初始结果集。 递归成员：这是递归的扩展。它引用自身来扩展结果集，直到不再满足递归条件为止。

-  这部分通过从CTE中引用自身（AllSubtasks st），并将subtask_id加1来生成下一个子任务。比如：条件WHERE st.subtask_id < t.subtasks_count确保我们只生成合法的子任务，即子任务ID不超过subtasks_count。

-  递归过程： 初始执行锚定成员查询，生成每个任务的第一个子任务。

-  递归成员查询执行，将子任务ID加1并生成新的子任务。

-  不断重复递归过程，直到条件不再满足。

-  示例说明
> 假设有如下任务：
> 
> task_id	|subtasks_count
>  |:-------- |:--------:| 
> 1	|3
> 使用正确条件 WHERE st.subtask_id < t.subtasks_count 的递归CTE会生成以下子任务：
> 
> 锚定成员：task_id = 1, subtask_id = 1
> 
> 递归1次：task_id = 1, subtask_id = 2（1 + 1）
> 
> 递归2次：task_id = 1, subtask_id = 3（2 + 1）
> 
> 最终结果是：
>
> | task_id | subtask_id |
> |:-------- |:--------:| 
> 1|	1
> 1	|2
> 1|	3
> 如果使用错误条件 WHERE st.subtask_id = t.subtasks_count：
> 
> 锚定成员：task_id = 1, subtask_id = 1
> 
> 递归0次：subtask_id 从未达到 subtasks_count，条件不满足，递归终止。
>
```MySQL []
-- 生成所有的任务和子任务
WITH RECURSIVE AllSubtasks AS (
    SELECT task_id, 1 AS subtask_id
    FROM Tasks
    WHERE subtasks_count > 0

    UNION ALL

    SELECT t.task_id, st.subtask_id + 1
    FROM AllSubtasks st
    JOIN Tasks t ON st.task_id = t.task_id
    WHERE st.subtask_id < t.subtasks_count
)
```

### >  CTE可解决问题——上下行交换

- 要交换SQL查询结果中的上下两行，可以使用窗口函数和条件语句实现。以下是一种通用的方法，假设表中有一个唯一标识行的列（例如自增ID或时间戳），可以利用这一列来确定要交换的行：

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

> 这段代码中： RowNumbered CTE 给每行添加了一个 row_num 列，表示行的顺序。 SELECT 语句使用 CASE 表达式和 JOIN 条件，根据 row_num 的值选择上下两行进行交换。 请注意，具体的表名、唯一标识列（如 id）、以及具体的排序逻辑需根据实际情况进行调整。


---



# 4. **REGXP 正则表达式和字符串处理**
   #### 展示如何使用SQL进行复杂的字符串匹配和处理，特别是使用正则表达式解决实际问题的技巧。

### >  正则中符号意义

- 正则表达式中的反斜杠 `(\)`:

用来标识特殊字符或构造，如 `\d` 表示任意数字，`\w` 表示任意单词字符，`\b` 表示单词边界。

反向引用（如 \1）用来引用正则表达式中前面捕获的组。例如 ([0-9])\1 意味着匹配一个数字，然后再次匹配这个数字。

SQL 语句中的反斜杠:

在许多 SQL 实现中，反斜杠也被用作转义字符。例如，在字符串中使用双引号或反斜杠本身时，你需要使用反斜杠来转义它们。

为了在 SQL 中正确传递正则表达式中的反斜杠，必须将其转义为 `\\`，这样 SQL 解释器会将其解释为单个反斜杠，再交给正则表达式引擎。

- 星号 `(*)`

是一个量词，用来指定其前面的元素可以重复零次或多次。它的作用是匹配任意数量的字符，甚至可以匹配没有字符的情况

星号 `(*)` 的主要作用是灵活地匹配电话号码中除去最后四位之外的任意部分，这样你可以专注于匹配最后四位的特定模式。

- `^`

在正则表达式中，`^` 有两个常见的含义，具体取决于它的使用位置：

1. **匹配字符串的开头**：
   - 当 `^` 放在正则表达式的开头时，它表示匹配字符串的开头。这样，表达式会从字符串的开头开始匹配。
   - 例如，`^abc` 会匹配以 "abc" 开头的字符串，如 "abcdef" 但不会匹配 "123abc"。

   ```regex
   ^abc
   ```

   匹配示例：
   - "abc123" (匹配)
   - "123abc" (不匹配)

2. **否定字符集**：
   - 当 `^` 放在字符集（方括号 `[]`）的开头时，它表示取反，即匹配不在字符集中的任何字符。
   - 例如，`[^0-9]` 表示匹配任何非数字字符。

   ```regex
   [^0-9]
   ```

   匹配示例：
   - "abc" (匹配)
   - "123" (不匹配)

总结一下，`^` 在正则表达式中主要用于表示匹配字符串的开头，或者在字符集内部用于否定匹配。

- 匹配字母/数字/特殊符号的正则表达式

匹配字母（任意字母）：`[A-Za-z]`

匹配大写字母：`[A-Z]`

匹配小写字母：`[a-z]`

例如，要找到包含字母的记录，你可以使用：

```sql
SELECT * FROM table_name WHERE column_name REGEXP '[A-Za-z]';
```

`[0-9]` 表示匹配任意单个数字（0-9），而 `*` 表示匹配这个数字零次或多次。这意味着 `[0-9]*` 可以匹配任意长度的数字串，包括空串。

`[a-zA-Z0-9_.-]*`表示匹配任意字母，0-9，_,.等特殊符号

- `$` 在正则表达式中表示“字符串的结尾”

`$`表示字符串的结尾，意味着前面的模式必须匹配到字符串的最后。如果在模式前面有字符，它们必须出现在字符串的末尾部分。

### >  正则表达来匹配固定格式

> 要编写一个 SQL 查询来查找在特定日期范围内有消费记录的电话号码，并且电话号码的四位尾数符合特定的格式（AABB、ABAB、AAAA），可以使用正则表达式匹配的功能。假设你的表名为 transactions，包含 phone_number、amount、transaction_date 等字段。下面是一个可能的 SQL 查询：

```sql
SELECT phone_number
FROM transactions
WHERE 
   transaction_date BETWEEN '2017-01-01' AND '2017-10-31'
   AND amount > 0
   AND (
       phone_number REGEXP '[0-9]*([0-9])\\1([0-9])\\2$'  -- 匹配 AABB
       OR phone_number REGEXP '[0-9]*([0-9])([0-9])\\1\\2$'  -- 匹配 ABAB
       OR phone_number REGEXP '[0-9]*([0-9])\\1\\1\\1$'  -- 匹配 AAAA
   );
```

---




# 5. **高级查询与优化**
  #### 涉及子查询、复杂查询的分解与优化、索引的使用等高级主题。

### >  行列转换

- 列转行

```sql
select 学号,
sum(case when 课程 = '语文' then 成绩 else null end) as 语文成绩,
sum(case when 课程 = '数学' then 成绩 else null end) as 数学成绩
from  成绩表
group by 学号
```

- 行转列

union


### >  连续判断，找组别中的规律后，进行分组group，找到每组的连续时间/数值

- 通常情况，有连续的组别内，通过相减的差值是一样的，所以可根据差值进行判断

| date       | period_state | row_num | grp | row_num - grp |
|:-------- |:--------:|:-------- |:--------:|  :-------- |
| 2019-01-01 | succeeded    | 1       | 1   | 0             |
| 2019-01-02 | succeeded    | 2       | 2   | 0             |
| 2019-01-03 | succeeded    | 3       | 3   | 0             |
| 2019-01-04 | failed       | 4       | 1   | 3             |
| 2019-01-05 | failed       | 5       | 2   | 3             |
| 2019-01-06 | succeeded    | 6       | 4   | 2             |

> 对于连续的日期和相同的状态，`row_num - grp` 是一个常数。
> 
> 例如，从 `2019-01-01` 到 `2019-01-03`，`row_num - grp` 都是 0。
> 
> 从 `2019-01-04` 到 `2019-01-05`，`row_num - grp` 都是 3。
> 
> 这样我们就可以用 `row_num - grp` 作为分组标识，将连续的日期区间分组在一起。
>
> 通过 `row_num - grp` 分组后，我们可以使用 `MIN(date)` 和 `MAX(date)` 来获取每个组的开始和结束日期

```sql
GroupedDates AS (
    SELECT period_state,
           MIN(date) AS start_date,
           MAX(date) AS end_date
    FROM NumberedDates
    GROUP BY period_state, row_num - grp
)
```
| period_state | start_date | end_date   |
|:-------- |:--------:|:-------- |
| succeeded    | 2019-01-01 | 2019-01-03 |
| failed       | 2019-01-04 | 2019-01-05 |
| succeeded    | 2019-01-06 | 2019-01-06 |

### >  错行判断是避免NULL值

- 错行left join，避免最后一行null值，可用abs()解决

```sql
# select
#	distinct c1.seat_id 
# from cinema c1 left join cinema c2
on abs(c1.seat_id - c2.seat_id) = 1 
# where c1.free = 1 and c2.free = 1
# order by c1.seat_id
```
### >  from - to 类似问题如何找全所有结点

- 反向调换后的union all + having（）可以找全from - to所有出现节点并做适当判断

```sql
select user1_id as id1, user2_id as id2
    from friendship
    where user1_id = 1
    union all
    select user2_id as id1, user1_id as id2
    from friendship
    where user2_id = 1
```

### >  所有值间相互计算问题，可用自连解决

- 所有值之间计算一遍，自连，连接条件：<>

```sql
SELECT MIN(ABS(p1.x - p2.x)) AS shortest
FROM Point p1
JOIN Point p2 ON p1.x <> p2.x;
```

### >  包含与未包含问题

- 找到主表，然后判断 `in()`, `not in ()`

---

# 6. **数学/时间计算与分析**
   #### 包含如何在SQL中进行复杂的数学运算以及统计分析的技巧。

### >  移动平均

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

### >  奇偶数判断

mod(id,2) <> 0

In the SQL script you provided, the % symbol is being used as the modulo operator.

In the CASE statement, id % 2 is used to determine if the id is odd or even. When id % 2 = 1, it means that the id is odd. Here's how it works:

id % 2 calculates the remainder when the id is divided by 2.

If the remainder is 1, it means the id is odd.

If the remainder is 0, it means the id is even.

### >  时间差表达

```sql
DATE_SUB(yesterday.recordDate, INTERVAL 1 DAY)
```

```sql
DATEADD(interval, number, date)
```

```sql
a.recordDate = b.recordDate -1`
```

```sql
a.event_date = date_add(b.event_date, interval -1 day)
```

```sql
mysql -> date_add
```

日期函数

`WEEKDAY()` 函数用于返回日期对应的星期几的索引，通常范围是从 0 到 6，其中 0 代表星期一，6 代表星期日。

```sql
SELECT WEEKDAY('2024-08-02');
```
返回值含义：
0: 星期一
1: 星期二
2: 星期三
3: 星期四
4: 星期五
5: 星期六
6: 星期日


---

# 7. **其他**

### >  应用分析
- 营销带货销量分析
查询包含优惠券码为 1 且 商品类型为 B 的订单号，并返回其中不同订单号的数量、这些订单号下的销售总额。

注意：订单号下的销售总额 = 相同的订单号的所有支付金额总和

- 库存分析
存销比 = 库存数 / 近 7 天销量。

- 退款分析
退款率 = 退款金额 / 订单金额。

- 人均购买频次
计算人均购买频次：用总购买次数除以用户总数。

### >  存储逻辑

```sql
CREATE PROCEDURE filter_age (
	IN age int
)
BEGIN
# Write your MySQL query statement below
SELECT 姓名, 年龄, 职位
    FROM 职员表
    WHERE 年龄 > age;
END
```
