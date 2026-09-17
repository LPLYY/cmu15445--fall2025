## 写在前面
**SQL**语句相比于其他编程语言例如cpp，java等相比简单一些，但是，由于许多原因，包括历史原因，产品的考虑等等，完全标准的SQL语句并不被所有数据库支持（笔者没听说过哪家数据库是完全按照SQL标准设计的，如有差错，恳请指出），每个厂商的产品或多或少都有自己的“方言”，所以下面介绍的标准SQL更多是一种通用方法，细微差别就要读者自行查阅不同数据库的使用手册了。有时一些“违规”走在标准前面，并力求将这些加入标准中，有些则等待完善。
笔者的SQL介绍仅仅作为皮毛，并且内容可能并不完备，更像是为了内容的完整性做出的章节，所以有余力的读者可以参考更权威的SQL教程。但是笔者保证这章内容能通过这门课的第一个homework。

	这章真的好繁杂，笔者自己构想的时候很简单，上手才发觉很多不常用的已经忘记了，写个笔记是对的。

好，我们下面介绍基础SQL，基础较好的读者可以略过，高级SQl我们会放在下一章。

## 基础SQL

#### 固有类型
SQL标准支持多种固有类型，char(n) ，varchar ，int ， smallint ， numeric ， real, double precision ， float(n)。

用户自定义数据：**char(n)** 自定义固定长度为n的字符串，**varchar(n**具有自定义最大长度的可变字符串，**float(n)** 自定义精度至少为n的浮点数.
**numeric(p.d)**：具有用户指定精度的定点数。这个数有p位数字（加上一个符号位），并且小数点右边有p位中的d位数字，比如numeric(3.1) 可以精确存 _ _ ._ 的浮点数。

依赖于机器的数据：**int ， smallint** 整数，小整数，**real , double precision**浮点数与双精度浮点数

其中char，和varchar区别在于补空，使用char时，比如char(10) 中插入“TOM”，会在后面补满空格，varchar则不会。

每种类型都可能包含一个被称作空(null)值的特殊值，空值表示一个缺失的值。

#### 定义关系
我们使用 create table 命令来定义基础关系，如果读者学习过c语言，这个过程有点像定义一个结构体，比如：
```sql
create table lol_pro_player ( 
    player_id   varchar(20),
    player_team varchar(20),
    player_kd   numeric(10,5)
);

```

当然，我们在上一张提到过约束,格式如:

```sql
	create table L ( 
	A1 D1,
    A2 D2,
    ...,
    An Dn，
    <完整性约束1>,
    ...,
    <完整性约束k> 
			);
```


常见的约束比如：
**primary key**(A1,A2....) 主码约束，为关系指定主码，主码唯一且不可为null。

**foreign key**(A1,A2....) **references** s 外码约束 ，表示该关系中的A1，A2 ....元素必须对应在s关系中某元组的主码，比如我们可以在lol_pro_player关系中将player_id作为外码，s可以是大名单的关系，这样就可以避免我们插入一个完全不存在的职业选手，还是很有用的是吧。

**no null** 不允许出现null，比如可以这样 

```sql
create table lol_pro_player (
			......
			player_id varchar(20) not null --这样我们就可以约束player_id不可以为空。
			.......)			
```
当然还有很多其他约束，笔者这里紧跟教材大旗，放在中级SQL介绍。
当用户试图改变数据时，如果违反了约束，数据库会阻止用户，具体的行为不同数据库会有差别。我们会在后面介绍改变数据的具体操作。

我们使用指令**drop table** 来删除关系，比如 **drop table**  s ，如果仅仅想删除关系中的元组而想保留关系本身可以这样 **delete from** r 。

我们使用**alter table**为关系增加属性 ，比如**alter table** a **add** A D；其中A为属性名称，D是属性类型。

#### 查询
##### 单关系查询
查询语句形式如下
```sql
SELECT  player_id FROM instructor;
```
这样不会去掉重复.。

如果想去掉重复，我们可以：
```sql
SELECT DISTINCT player_id FROM instructor;
```
我们可以使用**all**来显式指明不去除重复：select all ...  这样，虽然本身就是默认的。

我们可以通过**where**子句只选择符合特定谓词逻辑的元组。比如我们可以选出在A队效力并且kda大于2的职业选手：
```sql
SELECT player_id 
FROM instructor
WHERE player_team = 'A' AND player_kd > 2 ;
```
##### 多关系查询

目前为止我们只接触了单关系查询，我们接下来要处理多关系查询，在这之前我们要引入一个概念**笛卡尔积**，举个例子：
```sql
-- 选手表
CREATE TABLE lol_pro_player ( 
    player_id   VARCHAR(20),
    player_team VARCHAR(20),
    player_kd   NUMERIC(10,5)
);

-- 比赛表
CREATE TABLE match_info (
    match_id    VARCHAR(20),
    match_date  DATE,
    champion    VARCHAR(20)
);

-- 插入选手数据
INSERT INTO lol_pro_player VALUES 
    ('Uzi', 'RNG', 4.5),
    ('Faker', 'T1', 5.2),
    ('TheShy', 'IG', 3.8);

-- 插入比赛数据
INSERT INTO match_info VALUES 
    ('M001', '2026-09-01', '亚索'),
    ('M002', '2026-09-02', '劫'),
    ('M003', '2026-09-03', '剑姬');
```
对应的笛卡尔积应当：
```text
player_id | player_team | player_kd | match_id | match_date | champion
----------|-------------|-----------|----------|------------|----------
Uzi       | RNG         | 4.50000   | M001     | 2026-09-01 | 亚索
Uzi       | RNG         | 4.50000   | M002     | 2026-09-02 | 劫
Uzi       | RNG         | 4.50000   | M003     | 2026-09-03 | 剑姬
Faker     | T1          | 5.20000   | M001     | 2026-09-01 | 亚索
Faker     | T1          | 5.20000   | M002     | 2026-09-02 | 劫
Faker     | T1          | 5.20000   | M003     | 2026-09-03 | 剑姬
TheShy    | IG          | 3.80000   | M001     | 2026-09-01 | 亚索
TheShy    | IG          | 3.80000   | M002     | 2026-09-02 | 劫
TheShy    | IG          | 3.80000   | M003     | 2026-09-03 | 剑姬
```
如果读者学习过离散数学，这部分应该很好理解，

在多关系查询中，from 子句会先对所有关系做笛卡尔积，然后通过 where 子句从中筛选出满足条件的元组。例如，要查询每位选手对应的比赛信息，可以在 where 中添加连接条件：
```sql
SELECT *
FROM lol_pro_player, match_info
WHERE lol_pro_player.player_id = match_info.match_id;  -- 示例连接条件
```
不难看出 where 子句需要先建立笛卡尔积，所以 where 子句要写在 from 子句后面，这种顺序很重要，我们会在后面讨论更多。

但是问题还有，如果两个关系的联系很紧密，笛卡尔积产生的元组都是有意义的，那很好。但是很可惜大多时候都事与愿违，好在数据库会通过特定的优化来优化多关系查询，可是同样提醒读者在构建SQl语句的时候一定要谨慎对待多关系查询。

##### 附加查询
这一部分是 “方言” 重灾区，笔者只会按照标准介绍，具体的实现请参考不同数据库操作手册，不过也不会有特别大的出入。
###### as子句
SQL允许使用as在 select 和 from 子句中给属性起个别名
比如：
```sql
SELECT P.player_id, P.player_kd
FROM lol_pro_player AS P;
```
这是个很好的点子，在早期数据库系统是不支持 as 子句的，那如果我们想要比较一个关系下不同元组就会很麻烦，但是我们现在就可以这样 from A as B， A as C ， where B.p > C.p  ， 注意这样就不能使用 A.p 了。
比如：
```sql
-- 查询 KDA 高于 RNG 战队选手的所有其他选手
SELECT P1.player_id, P1.player_team, P1.player_kd
FROM lol_pro_player AS P1, lol_pro_player AS P2
WHERE P1.player_kd > P2.player_kd --这里不可以 lol_pro_player.player_kd
  AND P2.player_team = 'RNG';
```

##### 字符串运算
这一部分和正则表达式很相似，具体规则可以查阅相关帖子，比如：
```sql
SELECT *
FROM lol_pro_player
WHERE player_id = "%Z%";
```
其中' * ' 在SELECT子句中表示全部属性。

##### where 子句
SQL标准提供了 between 和 and 以简化 where 子句，比如：查询kd在1-2之间的选手
```sql
SELECT player_id
FROM lol_pro_player
WHERE player_kd BEtWEEN 1.0 AND 2.0;
```
这等价于：
```sql
SELECT player_id
FROM lol_pro_player
WHERE player_kd >= 1.0 AND player_kd <= 2.0;
```
类似的SQL也支持 not between 。

#### 集合运算

SQL 作用在关系上的 union（并）、interect（交）和 except（差）运算对应于数学中的 ∪、∩ 和 − 运算。

我们先来看两个查询：
```sql
-- 查询 RNG 战队的选手
SELECT player_id
FROM lol_pro_player
WHERE player_team = 'RNG';
-- 查询 KDA 大于 4.0 的选手
SELECT player_id
FROM lol_pro_player
WHERE player_kd > 4.0;
```
假设数据如下：

| player_id | player_team | player_kd |
| --------- | ----------- | --------- |
| Uzi       | RNG         | 4.5       |
| Faker     | T1          | 5.2       |
| TheShy    | IG          | 3.8       |
| Ming      | RNG         | 3.2       |
| Rookie    | IG          | 4.1       |

第一个查询返回 {Uzi, Ming}，第二个查询返回 {Uzi, Faker, Rookie}。
##### 并运算

找出 RNG 战队选手或 KDA 大于 4.0 的选手(或两者都满足）的所有选手：
```sql
(SELECT player_id
 FROM lol_pro_player
 WHERE player_team = 'RNG')
 
UNION

(SELECT player_id
 FROM lol_pro_player
 WHERE player_kd > 4.0);
```
结果：{Uzi, Ming, Faker, Rookie}
union 自动去除重复,如果想保留重复，使用 union all：
```sql

(SELECT player_id
 FROM lol_pro_player
 WHERE player_team = 'RNG')
 
UNION ALL

(SELECT player_id
 FROM lol_pro_player
 WHERE player_kd > 4.0);
```

结果：{Uzi, Ming, Uzi, Faker, Rookie}
##### 交运算
找出 既是 RNG 战队选手，同时 KDA 又大于 4.0 的选手：
```sql
(SELECT player_id
 FROM lol_pro_player
WHERE player_team = 'RNG');

INTERSECT

(SELECT player_id
 FROM lol_pro_player
 WHERE player_kd > 4.0);
```

结果：{Uzi}
intersect 自动去除重复。
##### 差运算
找出 **RNG 战队选手** 但 **KDA 不高于 4.0** 的选手：
```sql

(SELECT player_id
 FROM lol_pro_player
 WHERE player_team = 'RNG')
 
EXCEPT

(SELECT player_id
 FROM lol_pro_player
 WHERE player_kd > 4.0);
```
结果：{Ming}
except 自动去除重复。

#### 空值
空值并不是 0 的含义，所以如果有这样的语句 1  >  null ，这个结果并不是 true，而是 **unknown** ，所以有关 null 的运算结果都是 unknown 。where 子句中的布尔运算符可以处理 unknown 结果。
and : true and unknown -- unknown 
     false and unknown -- false
     unknown and unknown -- unknown
or : true or unknown -- true
    false or unknown -- unknown
    unknown or unknown -- unknown
not : not unknown -- unknown
我们使用 null 来检测空值，比如我们想找到kd为空值的选手：
```sql
SELECT player_id
FROM lol_pro_player
WHERE player_id IS NULL;
```
类似的我们也可以使用空值 unknown 的特性。
```sql
SELECT player_id
FROM lol_pro_player
WHERE player_id > 1 IS UNKNOWN;
```

#### 聚集函数
聚合函数旨在从已收集的数据中进一步推导出新的数据或信息，主要思路在于将多个数据提取并通过聚合函数整理为一个单一的值，简单来说就是把多个值操作为一个值。
聚合函数基本上只能用于SQL查询的输出列表，除此之外还可以在嵌套查询里，我们暂且忽略掉。
##### 基本聚集
我们首先介绍比较特殊的计数函数 **count** 。
比如：
```sql
SELECT COUNT(*) AS num
FROM lol_pro_player
WHERE player_team = 'IG';
```
我们会得到符合 where 语句谓词的元组数，其中count()内并不影响结果，他只是在这里统计匹配到的元组数量。所以count(1) 或者 count（aaa) 都会得到相同的结果。如果想去除重复项，可在聚集表达式中使用关键字 distinct 。比如：SELECT COUNT(distinct ID) AS num。
类似的SQL还提供的几聚合函数 平均值：avg。最小值：min, 最大值：max.总和：sum ，使用方法类似，但要注意此时（）内容与结果有关。对于空值，count（ * ）不会忽略，但是其他聚集函数都会忽略，所以聚集函数输入的值可能为空，规定count处理空集结果为0。

##### 分组聚集
有时我们不希望将所有元组聚集为一个值，使用**group by** 子句可以实现按用户的要求分组聚集，比如：我们想获得每个队伍的平均kd。
```sql
SELECT AVG(player_kd) AS team_avg_kd
FROM lol_pro_player
GROUP BY player_team;
```
当SQL查询使用分组时，任何没有出现在 group by 子句中的属性如果出现在 select 子句中，它只能作为聚集函数的参数。比如：
```sql
SELECT  player_id , AVG(player_kd) AS team_avg_kd --错误的！！！
FROM lol_pro_player
GROUP BY player_team;
```
player_id不可以出现在 select 子句中，因为我们分组之后，一个team只能输出一个数，无法确定player_id该输出什么。

##### having 子句
having 子句是针对 group by 分组的属性来做出筛选，比如：
```sql
SELECT player_team,AVG(player_kd) AS team_avg_kd
FROM lol_pro_player
GROUP BY player_team
HAVING AVG(player_kd) > 2;
```
读者可能会疑惑为什么要引入 having 子句，通通写在 where 子句不可以吗，这就要介绍SQL的逻辑执行顺序（不同数据库在物理上可能会做出不同的优化）:
FROM - JOIN - WHERE - GROUP BY  - 聚合函数 - HAVING - SELECRT - DISTINCT - ORDER BY - LIMIT
注意到 where 子句执行在 group by 之前，对其分组后的限制不可能发生在分组之前，所以引入 having子句。与 select 子句类似，任何出现在 having子句中，但是没有被聚集的属性必须出现在 group by 子句中。

#### 嵌套子查询
子查询可以出现在任何位置，我们先从最简单的 from 子句嵌套子查询开始，先引入连接词 **in** , **not in** ，比如：找出KDA高于自己战队平均KDA的选手。
```sql

SELECT p1.player_id, p1.player_team, p1.player_kd
FROM lol_pro_player p1
WHERE p1.player_kd > (
    SELECT avg_kd
    FROM (
        SELECT p2.player_team, AVG(p2.player_kd) AS avg_kd
        FROM lol_pro_player p2
        WHERE p2.player_team = p1.player_team 
        GROUP BY p2.player_team
    ) AS team_avg
);
```
注意到子查询的元素属性结果也可以在外层使用。

但是  from子句嵌套的子查询中不能使用来自同一from子句的其他关系的变量，比如：
```sql
FROM A1 , (子查询)
```
其中子查询不能包含A1的相关变量，但是最新的SQL提供**lateral**关键字解决这个问题。
```sql
FROM A1 , LATERAL(子查询)
```
这样就可以了。

同时 in 也支持枚举的检测，比如：
```sql
SELECT * FROM lol_pro_player
WHERE player_id IN ('Uzi', 'TheShy', 'Faker');
```
还有一种写法很贴近自然语言，比如 : 找出KDA至少比RNG战队某一位选手高的所有选手
```sql
SELECT DISTINCT T.player_id, T.player_team, T.player_kd
FROM lol_pro_player AS T
WHERE T.player_kd > SOME (
    SELECT S.player_kd
    FROM lol_pro_player AS S
    WHERE S.player_team = 'RNG'
);
```
这里表示大于其中任意一个就行，类似的还有< . <= , >= , = , <> 。

**not exists** 结构可以来测试子查询结果集中是否不存在元组，使用结构大概：where not exits（子查询），类似的 还有 exists。
**unique** 结构测试是否有重复元素 where unique（子查询）。
##### with 子句

**WITH子句** 用于定义**临时命名结果集**，这个定义只对包含with子句的查询有效。它使复杂查询更易读、更模块化，且可以被多次引用。
比如：先计算各战队平均KDA，然后找出高于平均值的选手
```sql
WITH team_avg AS (
    SELECT 
        player_team,
        AVG(player_kd) AS avg_kd
    FROM lol_pro_player
    GROUP BY player_team
)
SELECT 
    p.player_id,
    p.player_team,
    p.player_kd,
    t.avg_kd,
    ROUND(p.player_kd - t.avg_kd, 2) AS diff_from_avg
FROM lol_pro_player p
JOIN team_avg t ON p.player_team = t.player_team
WHERE p.player_kd > t.avg_kd
ORDER BY diff_from_avg DESC;
```
前面我们主要介绍了子查询出现在 **from** 子句中，SQL允许子查询出现在返回单个值的表达式能够出现的任何地方，标量子查询可以出现在 select 、where 和 having子句中。

#### 修改数据库

##### delete（删除）
SQL只支持删除整个元组而不支持删除单独某个属性的值，格式如下：
```sql
DELETE FROM lol_pro_player
WHERE player_id = 'the shy';
```
##### insert（插入）
最简单的插入就是插入一个完整元组
```sql
INSERT INTO lol_pro_players
	VALUES('wei','player_id','IG','player_team','2','player_kd';
```
顺序可以和原结构不同，但是顺序相同时可以省略属性值名称。
insert 也可以作用于查询结果。
```sql
INSERT INTO lol_pro_player;
	SELECT *
	FROM match_info;
```
##### update (更新)
在某些情况下，我们可能希望在不改变一个元组所有值情况下改变某个属性的值。比如：
```sql
UPDATE lol_player_kd 
SET player_kd = player_kd * 1.05;
```
SQL提供case结构，我们可以利用它在单条update语句中执行前面的更新，以避免更新次序引发的问题:
```sql
UPDATE lol_player_kd 
SET player_kd = CASE;
WHEN player_kd < 2 then player_kd * 1.5
WHEN player_kd >= 2 then player_kd * 2
```

好的，这就是SQL基础篇的全部内容量了，接下来我们将在中阶篇介绍进阶一点的特性。