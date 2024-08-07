# CTE
### 基础写法
>
```MySQL []
with A as (),
	B as ()
		select
```

### 递归CTE原理：
> 递归CTE允许你创建一个自引用的CTE，它可以用来执行递归查询。递归CTE由两个部分组成：
> 
> 锚定成员（基础情况）：这是递归的起点。它定义了递归CTE的初始结果集。
> 递归成员：这是递归的扩展。它引用自身来扩展结果集，直到不再满足递归条件为止。
> 
> 这部分通过从CTE中引用自身（AllSubtasks st），并将subtask_id加1来生成下一个子任务。条件WHERE st.subtask_id < t.subtasks_count确保我们只生成合法的子任务，即子任务ID不超过subtasks_count。
> 
> 递归过程：
> 初始执行锚定成员查询，生成每个任务的第一个子任务。
> 
> 递归成员查询执行，将子任务ID加1并生成新的子任务。
> 
> 不断重复递归过程，直到条件不再满足。
> 
WITH RECURSIVE ..... as （）
> 普通CTE和递归CTE的区别
> 
> ### 普通CTE：只用于一次性查询，并不自引用。也就是说，它生成的结果集不会再进行递归扩展。
> 
> ### 递归CTE：允许CTE自引用，即在CTE的定义中可以引用自身，从而进行递归计算。递归CTE可以重复执行查询逻辑，直到不再满足递归条件。
> 
> 为什么需要使用WITH RECURSIVE
> 
> 在你的需求中，你需要生成每个任务的所有子任务，并且子任务的数量是动态的，这需要递归地进行计算。只有递归CTE能够实现这种需求。
> 
> 如果使用 WHERE st.subtask_id = t.subtasks_count，则只会生成那些正好等于 subtasks_count 的子任务，这样就无法递增地生成所有子任务。例如：
> 
> 初始情况下，subtask_id 为 1。
> 
> 如果条件是 st.subtask_id = t.subtasks_count，那么只有在递归条件首次被满足时才会生成结果（即当 subtask_id 正好等于 subtasks_count 时）。
> 
> 这意味着你无法生成从 1 到 subtasks_count-1 的子任务。递归条件必须允许生成递增的子任务ID，直到达到 subtasks_count 为止。
> 
> 示例说明
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
-- 找出未执行的子任务
SELECT ast.task_id, ast.subtask_id
FROM AllSubtasks ast
LEFT JOIN Executed e ON ast.task_id = e.task_id AND ast.subtask_id = e.subtask_id
WHERE e.subtask_id IS NULL
ORDER BY ast.task_id, ast.subtask_id;
```
>
>


# 聚合函数
### 行列转换
>
> 列转行  union
>
> 行转列  category，sum(if(x, a, b)))


### 返回前一行数据`lag()`，返回后一行数据`lead()`
 LEAD(log_id, 1) OVER (ORDER BY log_id)
> 
> LEAD 函数访问当前行之后的第1行的 log_id 值。

> OVER (ORDER BY log_id) 指定了如何对行进行排序，以便确定哪一行是下一行。

> LEAD(visit_date, 1, '2021-01-01') OVER (PARTITION BY user_id ORDER BY visit_date) AS next_visit_date

> 使用 LEAD 函数获取每个用户的下一个访问日期。如果没有下一个日期，则默认设置为 '2021-01-01'。

LAG(log_id) OVER (ORDER BY log_id) AS prev_log_id 返回前一行的值

> 使用 LAG 函数获取每个用户的上一个访问日期。如果没有上一个日期，则默认设置为 '2020-01-01'（这可以设置为一个早于所有访问日期的日期）。
> 
> LAG(visit_date, 1, '2020-01-01') OVER (PARTITION BY user_id ORDER BY visit_date) 用来获取每个用户的上一个访问日期。

### 分层累加
>
```MySQL []
sum（）over（partition by...... order by）
```

### 分层聚合
>
> count()over()

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

### 直接在select中使用
>
```MySQL []
# select
round(sum(case when imm !='0' then 1 else 0 end) / count(*),2) as immediate_percentage
# from (
# select customer_id, 
        sum(case when order_date = customer_pref_delivery_date then 1 else 0 end) as imm
#    from delivery 
#    group by customer_id
# ) as a
````
### 在having中使用
>
```MySQL []
# SELECT seat_id 
# from grps
# group by seat_id 
having max(grp) = (select max(grp) from grps)
```

# 连续判断
### 数字： 错行left join，避免最后一行null值，可用`abs()`解决
>
```MySQL []
# select
#	distinct c1.seat_id 
# from cinema c1 left join cinema c2
on abs(c1.seat_id - c2.seat_id) = 1 
# where c1.free = 1 and c2.free = 1
# order by c1.seat_id
```

### 时间
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



# 连接
###  所有值之间计算一遍，自连，连接条件：<>
>
```MySQL []
SELECT MIN(ABS(p1.x - p2.x)) AS shortest
FROM Point p1
JOIN Point p2 ON p1.x <> p2.x;
```

###  包含与非包含，找到主表，然后判断 `in()`, `not in ()`

###  反向调换后的`union all` + `having（）`可以找全from - to所有出现节点并做适当判断
>
```MySQL []
select user1_id as id1, user2_id as id2
    from friendship
    where user1_id = 1
    union all
    select user2_id as id1, user1_id as id2
    from friendship
    where user2_id = 1
````

# 排序
###  不跳过相同排名，`dense_rank()`比`rank()`计算快一点
>
```MySQL []
# select a.name as Department, b.name as Employee, salary as Salary
# from Department as a 
# left join
# (select name, salary, departmentId,
dense_rank()over(partition by departmentId order by salary desc) as rk 
# from employee) as b 
# on a.id = b.departmentId 
# where rk = 1
```
