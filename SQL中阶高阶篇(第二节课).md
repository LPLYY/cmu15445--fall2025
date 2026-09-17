### 写在前面
这一部分将介绍SQL稍微高级一些的特性，但不会很难，同时将介绍一些将SQL嵌套到其他编程语言的内容，但这不是重点，请读者重点理连接，视图，授权，完整性约束这些概念上，以及掌握一定函数和过程的阅读和编写能力，这很重要。
其实笔者本来打算将SQL写作一个章节，但是没想到基础篇真的太长了，所以分了两章。

### 进阶特性
#### 连接
连接运算能让程序员写出更自然的SQL语句，并且能完成一些笛卡尔积很难做到的事情。
##### 自然连接
我们先考虑下面两个关系：
```sql
-- 选手表
CREATE TABLE lol_pro_player (
    player_id   VARCHAR(20) PRIMARY KEY,
    player_name VARCHAR(20),
    player_team VARCHAR(20),
    player_kd   NUMERIC(10,5)
);

-- 比赛记录表
CREATE TABLE match_info (
    match_id    VARCHAR(20) PRIMARY KEY,
    player_id   VARCHAR(20),          -- 关联字段
    match_date  DATE,
    champion    VARCHAR(20),
    result      VARCHAR(10)           -- '胜' / '负'
);

INSERT INTO lol_pro_player VALUES 
    ('P001', 'Uzi',    'RNG', 4.5),
    ('P002', 'Faker',  'T1',  5.2),
    ('P003', 'TheShy', 'IG',  3.8),
    ('P004', 'Clearlove', 'EDG', 6.0);   -- 没有比赛记录

INSERT INTO match_info VALUES 
    ('M001', 'P001', '2026-09-01', '亚索', '胜'),
    ('M002', 'P001', '2026-09-02', '劫',   '负'),
    ('M003', 'P002', '2026-09-03', '剑姬', '胜'),
    ('M004', 'P003', '2026-09-04', '锐雯', '胜'),
    ('M005', 'P005', '2026-09-05', '盲僧', '胜');  -- 选手不存在
```
自然连接运算作用于两个关系，产生一个关系，过程有些类似笛卡尔积，但是只考虑在两个关系的模式中都出现的那些属性上取值相同的元组对。
```sql
SELECT *
FROM lol_pro_player
NATURAL JOIN match_info;
```
结果：

| player_id | player_name | player_team | player_kd | match_id | match_date | champion | result |
| --------- | ----------- | ----------- | --------- | -------- | ---------- | -------- | ------ |
| P001      | Uzi         | RNG         | 4.5       | M001     | 2026-09-01 | 亚索       | 胜      |
| P001      | Uzi         | RNG         | 4.5       | M002     | 2026-09-02 | 劫        | 负      |
| P002      | Faker       | T1          | 5.2       | M003     | 2026-09-03 | 剑姬       | 胜      |
| P003      | TheShy      | IG          | 3.8       | M004     | 2026-09-04 | 锐雯       | 胜      |


自然连接会把重复的 `player_id` 合并成一列，它是隐式的，不可控
自然连接的连接条件由数据库"猜"——只要列名相同就自动连。这带来问题：如果两张表有多个同名列（比如都有 `match_date`、`name` 等），会被全部当成连接条件。为了避免这种问题，SQL提供方法让你显示指定连接条件。
```sql
sql
SELECT *
FROM lol_pro_player
NATURAL JOIN match_info USING (player_id);
```

SQL还支持另外一种形式的连接，其中可以指定任意的连接条件。
```sql
FROM lol_pro_player p
INNER JOIN match_info m
    ON p.player_id = m.player_id   --这就是连接条件
```
on条件可以表达任何SQL谓词，因而使用on条件的连接表达式就可以表示比自然连接更为丰富的连接条件。比如：
```sql
-- 既要求是同一选手，又要求是胜场
SELECT p.player_name, m.match_id, m.champion
FROM lol_pro_player p
JOIN match_info m
    ON p.player_id = m.player_id
   AND m.result = '胜';
```
**using ， natrue join** 也是连接条件。
##### 外连接
共有三种形式的外连接：
左外连接，只保留出现在左外连接运算之前（左边）的关系中的元组。
右外连接，只保留出现在右外连接运算之后（右边）的关系中的元组。
全外连接，保留出现在两个关系中的元组。
我们以左外连接举例：
```sql
SELECT p.player_id, p.player_name, p.player_team,
       m.match_id, m.champion, m.result
FROM lol_pro_player p
LEFT JOIN match_info m ON p.player_id = m.player_id;
```
**结果：**

|player_id|player_name|player_team|match_id|champion|result|
|---|---|---|---|---|---|
|P001|Uzi|RNG|M001|亚索|胜|
|P001|Uzi|RNG|M002|劫|负|
|P002|Faker|T1|M003|剑姬|胜|
|P003|TheShy|IG|M004|锐雯|胜|
|P004|Clearlove|EDG|**NULL**|**NULL**|**NULL**|

我们会保留左表没有成功连接的行，右表的属性这些行会置空。
#### CTE
还记得上一章我们讲的 with 子句吗，CTE（公用表表达式）通常和 with 子句一块出现，我们之前其实已经使用了，只是今天才介绍这个概念，我们举个例子：
对比一下，假设要"先统计每位选手的场次，再筛选出场次 ≥ 2 的选手"。
A：

```sql
SELECT *
FROM (
    SELECT p.player_name, COUNT(m.match_id) AS 场次
    FROM lol_pro_player p
    LEFT JOIN match_info m ON p.player_id = m.player_id
    GROUP BY p.player_name
) AS t
WHERE t.场次 >= 2;
```
B：
```sql
WITH player_stats AS (
    SELECT p.player_name, COUNT(m.match_id) AS 场次
    FROM lol_pro_player p
    LEFT JOIN match_info m ON p.player_id = m.player_id
    GROUP BY p.player_name
)
SELECT *
FROM player_stats
WHERE 场次 >= 2;
```
可能读者会觉得差不多，但是当你写更复杂的SQL语句时 CTE 能很清晰的表达，而嵌套查询会很难读，这一点等读者做homework 1时相信会有体会。

#### 窗口函数
窗口函数能在不合并行的前提下，对与当前行相关的一组行（称为"窗口"）进行计算。与 GROUP BY  聚合不同，窗口函数会为每一行返回一个结果，同时保留原始行的所有信息。
标准格式：
```sql
函数名() OVER (
    PARTITION BY 分组列      -- 可选：把数据分成多个窗口
    ORDER BY 排序列          -- 可选：窗口内排序
    窗口框架                 -- 可选：限定窗口边界
)
```
举个例子：我们假设表 `match_info`

|player_id|match_id|match_date|result|
|---|---|---|---|
|P001|M001|2026-09-01|胜|
|P001|M002|2026-09-02|负|
|P001|M003|2026-09-03|胜|
|P002|M004|2026-09-04|胜|
|P002|M005|2026-09-05|负|
```sql
SELECT 
    player_id,
    match_id,
    match_date,
    result,
    SUM(CASE WHEN result = '胜' THEN 1 ELSE 0 END) 
        OVER ()                                          AS 全局胜场,
    SUM(CASE WHEN result = '胜' THEN 1 ELSE 0 END) 
        OVER (PARTITION BY player_id)                    AS 本选手总胜场,
    SUM(CASE WHEN result = '胜' THEN 1 ELSE 0 END) 
        OVER (PARTITION BY player_id ORDER BY match_date) AS 本选手累计胜场,
    SUM(CASE WHEN result = '胜' THEN 1 ELSE 0 END) 
        OVER (PARTITION BY player_id ORDER BY match_date
              ROWS BETWEEN 1 PRECEDING AND CURRENT ROW)   AS 近2场胜场
FROM match_info
ORDER BY player_id, match_date;
```
结果：

|player_id|match_id|match_date|result|全局胜场|本选手总胜场|本选手累计胜场|近2场胜场|
|---|---|---|---|---|---|---|---|
|P001|M001|09-01|胜|3|2|1|1|
|P001|M002|09-02|负|3|2|1|1|
|P001|M003|09-03|胜|3|2|2|1|
|P002|M004|09-04|胜|3|1|1|1|
|P002|M005|09-05|负|3|1|1|1|

| 部分                     | 作用      | 省略时        |
| ---------------------- | ------- | ---------- |
| PARTITION BY player_id | 按选手分组   | 全部选手一组     |
| ORDER BY match_date0   | 组内按日期排序 | 不排序        |
| ROWS BETWEEN ...       | 看哪几行    | 默认从头累计到当前行 |
#### 授权
在SQL中有时我们不希望所有人都可以随意访问数据，我们可以通过授权来限制对关系的访问，在介绍授权之前我们先引入 **视图** 的概念，SQL中使用 **create view** 命令来定义视图。
比如：
```sql
CREATE VIEW v_player_stats AS
SELECT p.player_id,
       p.player_name,
       p.player_team,
       COUNT(m.match_id)                                AS 总场次,
       SUM(CASE WHEN m.result = '胜' THEN 1 ELSE 0 END) AS 胜场
FROM lol_pro_player p
LEFT JOIN match_info m ON p.player_id = m.player_id
GROUP BY p.player_id, p.player_name, p.player_team;
```
创建后，直接当表查：
```sql
SELECT * FROM v_player_stats WHERE 胜场 >= 1;
```
对于视图能否修改，不同数据库的规定有明显差异，大体上：

单表、无聚合、无 distinct、无 group by 通常可以。多表 join，不一定，看具体数据库。含聚合/ distinct / group by /窗口函数，不可以。 

视图常和权限配合：给用户视图权限，不给基表权限，实现数据隔离。
常见权限：

| 权限               |      |
| ---------------- | ---- |
| `SELECT`         | 查询   |
| `INSERT`         | 插入   |
| `UPDATE`         | 修改   |
| `DELETE`         | 删除   |
| `ALL PRIVILEGES` | 全部权限 |
```sql
-- 允许 analyst 查询选手表
GRANT SELECT ON lol_pro_player TO analyst;
--回收权限
REVOKE SELECT ON lol_pro_player FROM analyst;
```
更具体的就要读者自行根据不同数据库操作手册学习了。

#### 特殊数据类型
SQL除了基本数据类型，还有一些特殊的数据类型：
比如：日期 ，时间 ， 时间戳。
SQL允许为属性设置默认值，当插入元组没有指定元组属性时，自动设定为默认值。
比如：
```sql
CREATE TABLE match_info (
    match_id    VARCHAR(20) PRIMARY KEY,
    player_id   VARCHAR(20),
    match_date  DATE        DEFAULT CURRENT_DATE,      -- 不填就取当天
    champion    VARCHAR(20) DEFAULT '未知英雄',         -- 不填就用这个
    result      VARCHAR(10) DEFAULT '未定'              -- 不填就用这个
);
```
#### 函数与过程与触发器
SQL支持用户自己定义函数，但是不同书库语法差异很大，绝大部分数据库跟SQL提供的标准也大相径庭，所以这部分交由读者自行对不同数据库操作手册阅读。

#### 使用程序设计语言访问SQL
前文我们提到过SQL可以很好的嵌入到其他编程语言中，关于这部分课程并没有具体介绍，感兴趣的读者可以自行了解。
