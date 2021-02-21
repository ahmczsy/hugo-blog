# ClickHouse 中用 int 类型作为字典的效率高，还是用LowCardinality(String)


## 前言

今天有个业务是需要在某个表中新增一列。新增的列是个字典列，值的种类很少。一开始我想要使用 `LowCardinality(String)` 来存储，但有个同事说用`uInt32` 来存储。我的理由是看着直观，不用维护 int 的映射关系，他的理由是存储和查询的效率高。但是我从clickhouse 的官方文档中看到：

>The efficiency of using `LowCardinality` data type depends on data diversity. If a dictionary contains less than 10,000 distinct values, then ClickHouse mostly shows higher efficiency of data reading and storing. If a dictionary contains more than 100,000 distinct values, then ClickHouse can perform worse in comparison with using ordinary data types.

文档说数据的种类在十万以下时，效率很高，但不知道有多高。于是我打算 benchmark 测试下 `LowCardinality(String)` 和 `uInt32 `的性能差多少。



## 数据准备

### 建表语句

```sql
CREATE TABLE Dict
(
    id      UInt32,
    int_dic UInt32,
    str_dic LowCardinality(String)
)
    ENGINE = MergeTree() ORDER BY (id)
```

### 产生数据

```sql
INSERT INTO Dict
SELECT number                                                                                           AS id,
       [1, 2, 3, 4, 5, 6,7,8,9,10][rand() % 10 + 1]                                                     AS int_dic,
       ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'][rand() % 10 + 1] AS str_dic
FROM numbers(10000000)
```



## 测试结果

#### group 查询

##### int 类型 

```shell
SELECT
    int_dic,
    count()
FROM Dict
GROUP BY int_dic

┌─int_dic─┬─count()─┐
│       4 │  999772 │
│       3 │  999515 │
│       2 │  999412 │
│       5 │ 1000329 │
│       1 │ 1000159 │
│       6 │ 1000936 │
│       7 │  999680 │
│       9 │ 1000776 │
│       8 │  999922 │
│      10 │  999499 │
└─────────┴─────────┘

10 rows in set. Elapsed: 0.343 sec. Processed 10.00 million rows, 40.00 MB (29.19 million rows/s., 116.74 MB/s.)
```

##### LowCardinality(String) 类型

```shell
SELECT
    str_dic,
    count()
FROM Dict
GROUP BY str_dic

┌─str_dic─┬─count()─┐
│ nine    │ 1000776 │
│ six     │ 1000936 │
│ two     │  999412 │
│ three   │  999515 │
│ one     │ 1000159 │
│ four    │  999772 │
│ five    │ 1000329 │
│ ten     │  999499 │
│ seven   │  999680 │
│ eight   │  999922 │
└─────────┴─────────┘

10 rows in set. Elapsed: 0.159 sec. Processed 10.00 million rows, 10.02 MB (62.71 million rows/s., 62.84 MB/s.)
```



#### in  查询

##### int 类型

```shell
SELECT count()
FROM Dict
WHERE int_dic IN (1, 2, 3)

┌─count()─┐
│ 2999086 │
└─────────┘

1 rows in set. Elapsed: 0.094 sec. Processed 10.00 million rows, 40.00 MB (106.04 million rows/s., 424.17 MB/s.)
```



##### LowCardinality(String) 类型

```shell
SELECT count()
FROM Dict
WHERE str_dic IN ('one', 'two', 'three')

┌─count()─┐
│ 2999086 │
└─────────┘

1 rows in set. Elapsed: 0.041 sec. Processed 10.00 million rows, 10.02 MB (240.81 million rows/s., 241.32 MB/s.)
```



#### 单个值查询

##### int 类型

```shell
SELECT count()
FROM Dict
WHERE int_dic = 1

┌─count()─┐
│ 1000159 │
└─────────┘

1 rows in set. Elapsed: 0.047 sec. Processed 10.00 million rows, 40.00 MB (213.06 million rows/s., 852.26 MB/s.)

```



##### LowCardinality(String) 类型

```shell
SELECT count()
FROM Dict
WHERE str_dic = 'one'

┌─count()─┐
│ 1000159 │
└─────────┘

1 rows in set. Elapsed: 0.032 sec. Processed 10.00 million rows, 10.02 MB (310.69 million rows/s., 311.35 MB/s.)
```



## 结论

从测试结果来看，`LowCardinality(String)` 类型的性能大大超过我的预期。一开以为会比 int 类型稍微慢点

谁知却比 int 的性能好上一倍。通过这个测试顺利说服了同事😁


