# 0x00 方法论

- 化归：了解一题多解，但是我们这里只详细收录通解或者上位解

# 0x01 pymysql

## 代码框架

```python
import pymysql

if __name__ == '__main__':
	conn = pymysql.connect(		 # 连接数据库
	    host='localhost',        # 数据库主机地址
	    port=3306,               # MySQL 默认端口
	    user='root',             # 用户名
	    password='123123',       # 密码
	    db='mydb',               # 指定的数据库名, 或者使用conn.select_db(dbName)方法
	    charset='utf8mb4'        # 字符集
	)
    cursor = conn.cursor()		 # 获取操作游标
    sqls=[
        '''
        ''',
        '''
        ''',
        ...
    ]    # 这里填写MySQL命令，由于使用‘’‘因而直接按照MySQL的方式换行输入即可
    # sql='...'
    for sql in sqls:
        cursor.execute(sql)
        # conn.commit()            #0x03 提交操作
    cursor.close()
    conn.close()
```

[03 commit](#03 commit)<a id='03'></a>

## pymsql的insert方法

- 推荐使用参数化查询（`%s` 占位符）而不是直接拼接字符串，以防 **SQL 注入**。

- 示例：

  ```mysql
  sql = "INSERT INTO Course (Cno, Cname, Tno) 
  	   VALUES (%s, %s, %s)"
  data = (101, '数据结构', 'T001')
  cs.execute(sql, data)
  ```

## fetch

- `pymysql`查询`fetch`

  - 为什么要用`fetch`类函数：查询语句会将查询的结果集存储在游标中。游标本质上就是一个指针，初始时指向结果集的第一行。
  - 操作特点：在数据库游标（`cursor`）的运行机制中，提取数据的过程中游标会**自动往后移动**，这是因为游标的工作原理是 **顺序读取结果集**。因此下一次提取时，游标就会从上一次提取的结束位置继续读取，直到结果集的末尾。[^1.移动游标方式]

  | 方法                   | 描述                                                         | 返回值                         |
  | ---------------------- | ------------------------------------------------------------ | ------------------------------ |
  | `fetchone()`           | 每次提取一条记录；如果没有更多记录可提取，返回 `None`。      | 单个元组或 `None`              |
  | `fetchmany(size=None)` | 每次提取指定数量（`size`）的记录；如果没有更多记录可提取，返回空列表 `[]`。如果未指定 `size` 参数，则默认使用游标的 `arraysize` （`PyMySQL`中默认为`1`）属性值。 | 列表（包含最多 `size` 条元组） |
  | `fetchall()`           | 提取查询结果中所有剩余的记录；如果没有更多记录可提取，返回空列表 `[]`。 | 列表（包含所有剩余的记录元组） |

[^1.移动游标方式]:`scroll(value, mode='relative'/'absolute')`，其中`value`的值正向前负向后，`absolute`模式`0`为起始。

# 0x02 MySQL

## 基本操作

```mySQL
列出库
	SHOW DATABASES;
选中库
	USE database_name;
列出库中表
	SHOW TABLES;
创建库 
	CREATE DATABASE [IF NOT EXISTS] database_name
	[CHARACTER SET charset_name]
	[COLLATE collation_name]; # 0x01 charset_name和collation_name
删库
	DROP DATABASE [IF EXISTS] database_name;
建表
	CREATE TABLE tablename (
        column1 datatype [NOT NULL],
        column2 datatype [PRIMARY KEY],
    	···
    ) # 0x02 datatype
存入数据
    INSERT INTO tablename (column1, column2, column3, ...)
    VALUES
        (value1_1, value1_2, value1_3, ...),
        (value2_1, value2_2, value2_3, ...),
        (value3_1, value3_2, value3_3, ...);
更新数据
	UPDATE tablename
	SET 
		column1 = value1, 
		column2 = value2, 
		...
	WHERE 条件;
删除数据
	DELETE 
	FROM tablename 
	WHERE 条件;

```
[01 charset_name和collation_name](#01 charset_name和collation_name)<a id='01'></a>

[02 datatype](#02 datatype)<a id='02'></a>

## 其他操作

### SELECT

#### 基本语法

```mysql
SELECT [columnname或 *] 
FROM tablename a
[join other_tablename b
on a.column=b.column]
[WHERE 条件]
[GROUP BY 分组字段]
[HAVING 分组条件]
[ORDER BY 排序字段 [ASC|DESC]]
[LIMIT 限制行数];
```

- `GROUP BY`：用于对查询结果进行分组。
- `HAVING`：用于对分组后的结果进行过滤。
- `ORDER BY`：指定排序的字段，可以升序（`ASC`，默认）或降序（`DESC`）。
- `LIMIT`：限制返回的记录数量。

#### 连接方式

:star:**这里我们推荐使用`join...on`方法其次是自然连接**

- 其他连接方法

  - `where`语句

  - 多表直接汇总`select * from ininfo, outinfo`

  - 等值连接`select * from ininfo A, outinfo B where A.brand = B.brand`


  - 自然连接`select A.brand, A.indata, A.inNum, B.outdata, B.outNum from ininfo A, outinfo B`

#### 实例

-  01 SQL 查询：统计教师的课程数量，并按教师名称倒序排列

```mysql
SELECT Teacher.Tname, COUNT(Course.Cno) AS CourseCount
FROM Teacher
LEFT JOIN Course ON Teacher.Tno = Course.Tno
GROUP BY Teacher.Tname
ORDER BY Teacher.Tname DESC;
```

`LEFT JOIN`使得左表中有而右表无时，右表输出`NULL`

输出结果：

```
(('陈立前', 1), ('谭春娇', 0),...)
```



# 0x03 注释

## 01 charset_name和collation_name

[返回原文](#01)

- **Python 3.6+** 的 `pymysql` 版本要求必须指定 `charset`，否则会报错。*`form ChatGPT-4o-241120`*
- charset_name：
  - `utf8mb4`(包含了一切字符)
  - `gbk`
- collation_name：
  - `utf8mb4_general_ci`：不区分大小写，通用性能较好，但不适合某些语言化排序。
  - `utf8mb4_unicode_ci`：不区分大小写，支持更多语言特性，排序更精确，但性能稍慢。
  - `utf8mb4_bin`：区分大小写，按二进制值排序，性能很高，但没有语言化特性。

## 02 datatype

[返回原文](#02)

MySQL支持的字段属性包括：

- 日期和时间数据类型
  - data：3字节，日期，格式：2014-09-18
  - time：3字节，时间，格式：08:42:30
  - datetime：8字节，日期时间，格式：2014-09-18 08:42:30
  - timestamp：4字节，自动存储记录修改的时间
  - year：1字节，年份
- 数值数据类型。
  - tinyint：1字节，范围（-128~127）
  - smallint：2字节，范围（-32768~32767）
  - mediumint：3字节，范围（-8388608~8388607）
  - `int`： 4字节，范围（-2147483648~2147483647）
  - bigint：8字节，范围（+-9.22*10的18次方）
- 浮点型。
  - `float(m, d)`：4字节，单精度浮点型，m总个数，d小数位
  - `double(m, d)`：8字节，双精度浮点型，m总个数，d小数位
  - `decimal(m, d)`：decimal是存储为字符串的浮点数
- 字符串数据类型。
  - `char(n)`：固定长度，最多255个字符
  - `varchar(n)`：可变长度，最多65535个字符
  - tinytext：可变长度，最多255个字符
  - text：可变长度，最多65535个字符——与varchar比较支持全文索引，缺点
  - mediumtext：可变长度，最多2的24次方-1个字符
  - longtext：可变长度，最多2的32次方-1个字符

## 03 commit

[返回原文](#03)

- MySQL 默认启用**自动提交模式**（`autocommit=1`），使用`SET autocommit=0;`修改；在默认情况下，PyMySQL 不开启自动提交模式，使用`conn.autocommit(True)`来开启。

- 需要进行commit的操作：
  - 插入数据（`INSERT`）
  - 更新数据（`UPDATE`）
  - 删除数据（`DELETE`）
  - 批量操作或需要事务管理的场景
