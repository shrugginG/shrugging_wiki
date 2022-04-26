数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户。 在使用left jion时，on和where条件的区别如下：

1、on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。

2、where条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。



# Create

- 1.1 直接创建表：

```sql
CREATE TABLE
[IF NOT EXISTS] tb_name -- 不存在才创建，存在就跳过
(column_name1 data_type1 -- 列名和类型必选
  [ PRIMARY KEY -- 可选的约束，主键
   | FOREIGN KEY -- 外键，引用其他表的键值
   | AUTO_INCREMENT -- 自增ID
   | COMMENT comment -- 列注释（评论）
   | DEFAULT default_value -- 默认值
   | UNIQUE -- 唯一性约束，不允许两条记录该列值相同
   | NOT NULL -- 该列非空
  ], ...
) [CHARACTER SET charset] -- 字符集编码
[COLLATE collate_value] -- 列排序和比较时的规则（是否区分大小写等）
```

- 1.2 从另一张表复制表结构创建表： `CREATE TABLE tb_name LIKE tb_name_old`
- 1.3 从另一张表的查询结果创建表： `CREATE TABLE tb_name AS SELECT * FROM tb_name_old WHERE options`
- 2.1 修改表：`ALTER TABLE 表名 修改选项` 。选项集合：

```sql
    { ADD COLUMN <列名> <类型>  -- 增加列
     | CHANGE COLUMN <旧列名> <新列名> <新列类型> -- 修改列名或类型
     | ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT } -- 修改/删除 列的默认值
     | MODIFY COLUMN <列名> <类型> -- 修改列类型
     | DROP COLUMN <列名> -- 删除列
     | RENAME TO <新表名> -- 修改表名
     | CHARACTER SET <字符集名> -- 修改字符集
     | COLLATE <校对规则名> } -- 修改校对规则（比较和排序时用到）
```

- 3.1 删除表：`DROP TABLE [IF EXISTS] 表名1 [ ,表名2]`。

# Read

## count()的区别

《Mysql必知必会》P77

> COUNT()函数有两种使用方式。
>
> * 使用COUNT(*)对表中行的数目进行计数，不管表列中包含的是空 值（NULL）还是非空值。 
> * 使用COUNT(column)对特定列中具有值的行进行计数，忽略 NULL值。 

## 多表关联的语法

```sql
... FROM table1 INNER|LEFT|RIGHT JOIN table2 ON condition INNER|LEFT|RIGHT JOIN table3 ON condition ...
```

## join配合using使用

```sql
select tag,difficulty,round((sum(score)-max(score)-min(score))/(count(score)-2),1)
from exam_record join examination_info using (exam_id)
where tag='SQL' and difficulty='hard';
```





# Update 
## 如何通过一张表更新另一张表

四种插入方式：

- 普通插入（全字段）：INSERT INTO table_name VALUES (value1, value2, ...)
- 普通插入（限定字段）：INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)
- 多条一次性插入：INSERT INTO table_name (column1, column2, ...) VALUES (value1_1, value1_2, ...), (value2_1, value2_2, ...), ...
- **从另一个表导入：INSERT INTO table_name SELECT * FROM table_name2 [WHERE key=value]**

[Example](https://www.nowcoder.com/practice/9681abf28745468c8adacb3b029a18ce):

```sql
INSERT INTO exam_record_before_2021(uid, exam_id, start_time, submit_time, score)
SELECT uid, exam_id, start_time, submit_time, score
FROM exam_record
WHERE YEAR(submit_time) < 2021;
```

## REPLACE用法

```sql
REPLACE INTO examination_info``VALUES(NULL,``9003``,``'SQL'``,``'hard'``,``90``,``'2021-01-01 00:00:00'``);
```

1. 关键字NULL可以用DEFAULT替代。
2. 掌握replace into···values的用法

replace into 跟 insert into功能类似，不同点在于：replace into 首先尝试插入数据到表中，

1. 如果发现表中已经有此行数据**（根据主键或者唯一索引判断）**则先删除此行数据，然后插入新的数据；
2. 否则，直接插入新数据。

**要注意的是：插入数据的表必须有主键或者是唯一索引！否则的话**

**，replace into 会直接插入数据，这将导致表中出现重复的数据。**

# Delete

## [删除表中所有数据并重置自增列](https://www.nowcoder.com/practice/3abefc6fc73e4f219dad0ab66e6b1e3f)

两种方式：

- 利用TRUNCATE可以快速删除一张明确要删除的表，并且重置。但是不可回滚！

```sql
TRUNCATE exam_record;
```

- 手动重置自增ID，但是DELETE操作是可以回滚的

```sql
DELETE FROM exam_record;
ALTER TABLE exam_record auto_increment=1;
```

# Other

## 关于时间的计算

### 时间差

- TIMESTAMPDIFF(interval, time_start, time_end)可计算time_start-time_end的时间差，单位以指定的interval为准，常用可选：

  - SECOND 秒
  - MINUTE 分钟（返回秒数差除以60的整数部分）
  - HOUR 小时（返回秒数差除以3600的整数部分）
  - DAY 天数（返回秒数差除以3600*24的整数部分）
  - MONTH 月数
  - YEAR 年数

### Date

Day(date)

### 如何计算某个月份的天数

```sql
DAYOFMONTH(last_day(curdate()));

DAY(last_day(curdate()));
```


## if的使用

```sql
select
    count(exam_id) as total_pv,
    count(submit_time) as complete_pv,
    count(distinct if(submit_time is not null, exam_id, null)) as complete_exam_cnt
from exam_record
```

### date_format()和time_format()的区别



## 窗口函数
### 专用窗口函数

#### ranking

#### 理解group by与窗口函数配合使用的关系

https://www.nowcoder.com/practice/255aa1863fe14aa88694c09ebbc1dbca?tpId=240&tqId=2183404&ru=/exam/oj&qru=/ta/sql-advanced/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240

#### 三种不同rank窗口函数的使用方法

* RANK()

* ROW_NUMBER()

* DENSE_RANK()

  https://www.nowcoder.com/practice/4a3acb02b34a4ecf9045cefbc05453fa?tpId=240&tqId=2183407&ru=%2Fpractice%2F9dcc0eebb8394e79ada1d4d4e979d73c&qru=%2Fta%2Fsql-advanced%2Fquestion-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240

### 聚合窗口函数

#### [max() over() 逐行计算最大最小值](https://www.nowcoder.com/practice/2b7acdc7d1b9435bac377c1dcb3085d6?tpId=240&tqId=2183410&ru=/exam/oj&qru=/ta/sql-advanced/question-ranking&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240) 

#### [sum() over() 逐行累加](https://www.nowcoder.com/practice/5f1cbe74c682485aa73e4c2b30f04a62?tpId=240&tags=&title=&difficulty=0&judgeStatus=0&rp=0&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240)

## 数据处理

### 小数的处理

[mysql 保留两位小数](https://www.cnblogs.com/xiaomai333/p/7647381.html)

1、round(x,d) :用于数据的四舍五入,round(x)  ,其实就是round(x,0),也就是默认d为0；

这里有个值得注意的地方是，d可以是负数，这时是指定小数点左边的d位整数位为0,同时小数位均为0；

SELECT ROUND(100.3465,2),ROUND(100,2),ROUND(0.6,2),ROUND(114.6,-1);

结果分别：100.35,100，0.6,110



2、TRUNCATE(x,d)：函数返回被舍去至小数点后d位的数字x。若d的值为0，则结果不带有小数点或不带有小数部分。若d设为负数，则截去（归零）x小数点左起第d位开始后面所有低位的值。

SELECT TRUNCATE(100.3465,2),TRUNCATE(100,2),TRUNCATE(0.6,2),TRUNCATE(114.6,-1);

结果分别：100.34,100，0.6,110

 

3、FORMAT（X,D）：强制保留D位小数，整数部分超过三位的时候以逗号分割，并且返回的结果是string类型的

　SELECT FORMAT(100.3465,2),FORMAT(100,2),FORMAT(,100.6,2);

结果分别：100.35,100.00，100.60

 

4、convert（value，type）;类型转换，相当于截取``

type:

- 二进制，同带binary前缀的效果 : BINARY   
- 字符型，可带参数 : CHAR()   
- 日期 : DATE   
- 时间: TIME   
- 日期时间型 : DATETIME   
- 浮点数 : DECIMAL    
- 整数 : SIGNED   
- 无符号整数 : UNSIGNED 



## 题型整理

### [计算月活用户](https://www.nowcoder.com/practice/1ce93d5cec5c4243930fc5e8efaaca1e?tpId=240&tags=&title=&difficulty=0&judgeStatus=0&rp=0&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240)

