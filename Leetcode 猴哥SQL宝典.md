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
1. **`[0-9]*`**: 在你的正则表达式中，`[0-9]` 表示匹配任意单个数字（0-9），而 `*` 表示匹配这个数字零次或多次。这意味着 `[0-9]*` 可以匹配任意长度的数字串，包括空串。

  在上下文中，这个部分放在正则表达式的开头，用来匹配电话号码中不重要的部分（除了最后四位），以确保最后四位符合你想要的模式（AABB、ABAB、AAAA）。
2. **`amount > 0`**: 仅查询有消费的记录，即费用不为零的记录。
3. **`phone_number REGEXP '[0-9]*([0-9])\\1([0-9])\\2$'`**: 匹配电话号码的四位尾数符合 AABB 模式。
4. **`phone_number REGEXP '[0-9]*([0-9])([0-9])\\1\\2$'`**: 匹配电话号码的四位尾数符合 ABAB 模式。
5. **`phone_number REGEXP '[0-9]*([0-9])\\1\\1\\1$'`**: 匹配电话号码的四位尾数符合 AAAA 模式。

> ### 反斜杠：
