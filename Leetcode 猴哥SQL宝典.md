# 正则表达
> 要编写一个 SQL 查询来查找在特定日期范围内有消费记录的电话号码，并且电话号码的四位尾数符合特定的格式（AABB、ABAB、AAAA），可以使用正则表达式匹配的功能。假设你的表名为 `transactions`，包含 `phone_number`、`amount`、`transaction_date` 等字段。下面是一个可能的 SQL 查询：
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
> ### 具体作用：
 - **`[0-9]*`**: 在你的正则表达式中，`[0-9]` 表示匹配任意单个数字（0-9），而 `*` 表示匹配这个数字零次或多次。这意味着 `[0-9]*` 可以匹配任意长度的数字串，包括空串。

  在上下文中，这个部分放在正则表达式的开头，用来匹配电话号码中不重要的部分（除了最后四位），以确保最后四位符合你想要的模式（AABB、ABAB、AAAA）。

 - **`phone_number REGEXP '[0-9]*([0-9])\\1([0-9])\\2$'`**: 匹配电话号码的四位尾数符合 AABB 模式。

 - **`phone_number REGEXP '[0-9]*([0-9])([0-9])\\1\\2$'`**: 匹配电话号码的四位尾数符合 ABAB 模式。

 - **`phone_number REGEXP '[0-9]*([0-9])\\1\\1\\1$'`**: 匹配电话号码的四位尾数符合 AAAA 模式。

> ### 反斜杠：
- **正则表达式中的反斜杠 (`\`)**: 
  - 用来标识特殊字符或构造，如 `\d` 表示任意数字，`\w` 表示任意单词字符，`\b` 表示单词边界。
  - 反向引用（如 `\1`）用来引用正则表达式中前面捕获的组。例如 `([0-9])\1` 意味着匹配一个数字，然后再次匹配这个数字。

- **SQL 语句中的反斜杠**:
  - 在许多 SQL 实现中，反斜杠也被用作转义字符。例如，在字符串中使用双引号或反斜杠本身时，你需要使用反斜杠来转义它们。
  - 为了在 SQL 中正确传递正则表达式中的反斜杠，必须将其转义为 `\\`，这样 SQL 解释器会将其解释为单个反斜杠，再交给正则表达式引擎。
 
> ### 星号 (`*`)
- 是一个量词，用来指定其前面的元素可以重复零次或多次。它的作用是匹配任意数量的字符，甚至可以匹配没有字符的情况

- 星号 (`*`) 的主要作用是灵活地匹配电话号码中除去最后四位之外的任意部分，这样你可以专注于匹配最后四位的特定模式。

> ### `$` 在正则表达式中表示“字符串的结尾”

- **`$`**: 表示字符串的结尾，意味着前面的模式必须匹配到字符串的最后。如果在模式前面有字符，它们必须出现在字符串的末尾部分。

> ### 匹配字母的正则表达式

匹配字母（任意字母）：[A-Za-z]

匹配大写字母：[A-Z]

匹配小写字母：[a-z]

例如，要找到包含字母的记录，你可以使用：

````sql
Copy code
SELECT * FROM table_name WHERE column_name REGEXP '[A-Za-z]';
````

# 窗口函数相关
## Hive窗口函数
### 1. **`ROW_NUMBER()`**
`ROW_NUMBER()` 函数返回当前行在其窗口分区中的序号，排序依据由 `ORDER BY` 子句决定。

```sql
SELECT 
  id,
  username,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY created_at) AS row_num
FROM 
  users;
```

### 2. **`RANK()`**
`RANK()` 函数为每一行返回其排序后的排名，相同排名的行将共享相同的排名值，之后的排名会跳过（即不连续）。

```sql
SELECT 
  id,
  username,
  RANK() OVER (PARTITION BY department ORDER BY created_at) AS rank
FROM 
  users;
```

### 3. **`DENSE_RANK()`**
`DENSE_RANK()` 类似于 `RANK()`，但相同排名的行共享相同的排名值，且后续排名不会跳过（即连续）。

```sql
SELECT 
  id,
  username,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY created_at) AS dense_rank
FROM 
  users;
```

### 4. **`NTILE(n)`**
`NTILE(n)` 函数将行按照排序结果分成 `n` 个几乎相等的桶（组），每个桶有一个桶编号。

```sql
SELECT 
  id,
  username,
  NTILE(4) OVER (ORDER BY created_at) AS bucket
FROM 
  users;
```

### 5. **`LEAD()`**
`LEAD()` 函数允许你访问位于当前行之后的行数据。它通常用于比较当前行与后续行的数据。

```sql
SELECT 
  id,
  username,
  created_at,
  LEAD(username, 1) OVER (ORDER BY created_at) AS next_user
FROM 
  users;
```

### 6. **`LAG()`**
`LAG()` 函数与 `LEAD()` 类似，但用于访问当前行之前的行数据。

```sql
SELECT 
  id,
  username,
  created_at,
  LAG(username, 1) OVER (ORDER BY created_at) AS previous_user
FROM 
  users;
```

### 7. **`FIRST_VALUE()`**
`FIRST_VALUE()` 函数返回窗口分区内按指定排序的第一行的值。

```sql
SELECT 
  id,
  username,
  FIRST_VALUE(username) OVER (ORDER BY created_at) AS first_user
FROM 
  users;
```

### 8. **`LAST_VALUE()`**
`LAST_VALUE()` 函数返回窗口分区内按指定排序的最后一行的值。

```sql
SELECT 
  id,
  username,
  LAST_VALUE(username) OVER (ORDER BY created_at) AS last_user
FROM 
  users;
```

### 9. **`CUME_DIST()`**
`CUME_DIST()`（累积分布）计算小于等于当前行值的行数的比例。返回值在 0 到 1 之间。

```sql
SELECT 
  id,
  username,
  CUME_DIST() OVER (ORDER BY created_at) AS cum_dist
FROM 
  users;
```

### 10. **`PERCENT_RANK()`**
`PERCENT_RANK()` 计算当前行在其窗口分区中的百分排名，返回值在 0 到 1 之间。

```sql
SELECT 
  id,
  username,
  PERCENT_RANK() OVER (ORDER BY created_at) AS percent_rank
FROM 
  users;
```

## 窗口函数作用
- 窗口函数允许你在不改变结果集的行数的情况下，对数据进行聚合计算，你可能需要在显示详细数据的同时，添加额外的统计信息

- 在 SQL 中使用窗口函数（开窗函数）时，通常不需要使用 GROUP BY 子句。窗口函数的作用是对查询结果的某一部分进行排序、分区和聚合，通常用于计算行的排名、移动平均、累计总和等。与 GROUP BY 不同，窗口函数不会将结果按组汇总，而是对每一行应用特定的计算。

- 如果需要额外统计信息，比如累加，需要把所有列都加在over（）分区里

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

# 可能考察的业务计算逻辑
- 营销带货销量分析
 
查询包含优惠券码为 1 且 商品类型为 B 的订单号，并返回其中不同订单号的数量、这些订单号下的销售总额。

注意：订单号下的销售总额 = 相同的订单号的所有支付金额总和

- 库存分析

存销比 = 库存数 / 近 7 天销量。

- 退款分析

退款率 = 退款金额 / 订单金额。

- 人均购买频次

计算人均购买频次：用总购买次数除以用户总数。

# 储存过程的使用
````sql
CREATE PROCEDURE filter_age (
	IN age int
)
BEGIN
# Write your MySQL query statement below
SELECT 姓名, 年龄, 职位
    FROM 职员表
    WHERE 年龄 > age;
END
````

# 行列转换
- 列转行

````sql
select 学号,
sum(case when 课程 = '语文' then 成绩 else null end) as 语文成绩,
sum(case when 课程 = '数学' then 成绩 else null end) as 数学成绩
from  成绩表
group by 学号
````

# order by 处理特殊排序
````sql
ORDER BY 
    CASE 
        WHEN 订单数 >= 0 AND 订单数 <= 2 THEN 1
        WHEN 订单数 >= 3 AND 订单数 <= 5 THEN 2
        WHEN 订单数 > 5 THEN 3
        ELSE 4 
    END
````

# having 处理特殊筛选条件
- BUY A and B ，NOT C  找出既购买过 ProductA 和 ProductB，但没有购买过 ProductC 的顾客

````sql
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

# 日期函数
`WEEKDAY()` 函数用于返回日期对应的星期几的索引，通常范围是从 0 到 6，其中 0 代表星期一，6 代表星期日。

### 使用方法：
```sql
SELECT WEEKDAY('2024-08-02');
````
### 返回值含义：
- 0: 星期一
- 1: 星期二
- 2: 星期三
- 3: 星期四
- 4: 星期五
- 5: 星期六
- 6: 星期日
