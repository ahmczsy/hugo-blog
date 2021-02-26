# mysql 如何查看，修改 collate


## 问题

今天偶然发现 mysql 在比较字符时会忽略大小写，这让我非常意外。

比如 `select 'a' = 'A'` 这行运行的结果是 `1`也就是在 mysql 看来 `a` 和 `A` 是相等的

经过一番的查找原来是 mysql 的 `collate` 属性造成的



## collate

我们在创建数据库是一般都是直接 `create database <name>` 

如果需要使用 utf8 字符集会在后面指定 database 的字符集`create database <name> character set utf8mb4`

但除了`charset` 属性还有`collate` 属性是可以设置（如下），`collate` 就是和大小写敏感有关

```sql
CREATE DATABASE <name>
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
```



####  collate 的作用

对于mysql中那些字符类型的列，如`VARCHAR`，`CHAR`，`TEXT`类型的列，都需要有一个`COLLATE`类型来告知mysql如何对该列进行排序和比较。简而言之，**COLLATE会影响到ORDER BY语句的顺序，会影响到WHERE条件中大于小于号筛选出来的结果，会影响`DISTINCT`、`GROUP BY`、`HAVING`语句的查询结果**。另外，mysql建索引的时候，如果索引列是字符类型，也**会影响索引创建**，只不过这种影响我们感知不到。总之，**凡是涉及到字符类型比较或排序的地方，都会和COLLATE有关**。



#### collate 的后缀

`collate` 有三种后缀

* `_ci` ： case insensitive 的缩写，即大小写无关。例如：`utf8mb4_general_ci`
* `_cs` ： case sensitive 的缩写 ，即大小写有关
* `_bin` ：把字符看作二进制串，然后从最高位往最低位比对。所以很显然它是区分大小写的。例如 `utf8mb4_bin`



#### 其他

* 在 mysql 5.7 中 utf8 编码的默认 `collate` 是 `utf8mb4_general_ci`
* `show collation` 可以查看 mysql 所有支持的 `collate`



## 查看 collate

从上面的信息 得出`collate` 是影响查询大小写敏感的关键，所以接下来肯定是想要看看目前的数据库，表的 `collate ` 值是什么，是不是以 `_ci` 结尾

#### 查询 database 的 collate

```sql
SELECT SCHEMA_NAME                'database',
       DEFAULT_CHARACTER_SET_NAME 'charset',
       DEFAULT_COLLATION_NAME     'collation'
FROM information_schema.SCHEMATA
where SCHEMA_NAME = 'collate_test_db';
```

结果：

```sh
+-----------------+---------+--------------------+
| database        | charset | collation          |
+-----------------+---------+--------------------+
| collate_test_db | utf8mb4 | utf8mb4_general_ci |
+-----------------+---------+--------------------+
```



#### 查询表的 collate

```sql
select TABLE_SCHEMA, TABLE_COLLATION
from information_schema.TABLES
where TABLE_NAME = 'collate_test_table'
```

结果：

```shell
+-----------------+--------------------+
| TABLE_SCHEMA    | TABLE_COLLATION    |
+-----------------+--------------------+
| collate_test_db | utf8mb4_general_ci |
+-----------------+--------------------+
```



#### 查询列的 collate

```sql
SHOW FULL COLUMNS FROM <table_name>;
```



## 修改 collate

从上面看出的数据库和表都是 `_ci` 后缀的，所以下面就是把后缀改为 `_bin`



#### database 层级的修改

```sql
ALTER DATABASE collate_test_db DEFAULT CHARACTER SET utf8mb4 COLLATE = utf8mb4_bin;
```

这只是修改了 database 的默认 `collate` ，不会对已经存在的表进行修改



#### table 层级的修改

> :warning: **注意下面有两个 sql** ：第一个 sql 只会修改 table 的默认 `collate` ，不会修改已经存在字段的 `collate` 。第二个 sql 不但会修改默认的 `collate` ，还会修改已存在的字段。所以推荐使用第二个 sql

```sql
ALTER TABLE collate_test_table  CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

```sql
ALTER TABLE collate_test_table CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```



#### 批量生成修改 collate 的sql

如果一个 database 内的 table 过多，一个个写 alter sql 太麻烦了，可以批量生成

```sql
SELECT CONCAT('ALTER TABLE `', TABLE_NAME,
              '` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;')
           AS target_tables
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'collate_test_db'
  AND TABLE_TYPE = 'BASE TABLE'
```

结果：

```shell
+------------------------------------------------------------------------------------------+
| target_tables                                                                            |
+------------------------------------------------------------------------------------------+
| ALTER TABLE `collate_test_table` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;   |
| ALTER TABLE `collate_test_table_1` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin; |
| ALTER TABLE `collate_test_table_2` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin; |
| ALTER TABLE `collate_test_table_3` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin; |
| ALTER TABLE `collate_test_table_4` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin; |
+------------------------------------------------------------------------------------------+
```



## 参考

* https://stackoverflow.com/questions/6115612/how-to-convert-an-entire-mysql-database-characterset-and-collation-to-utf-8  
* https://blog.csdn.net/ghosind/article/details/83692869
* https://cloud.tencent.com/developer/article/1366841 


