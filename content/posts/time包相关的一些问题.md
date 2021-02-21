---
title: "golang time包相关的一些问题"
date: 2019-07-25T14:42:11+08:00
---

### 获取毫秒时间戳

```go
//第一种
time.Now().UnixNano() / int64(time.Millisecond)
//第二种
time.Now().UnixNano() / 1e6
```
<!--more-->


### time.Unix()用法

```go
//函数定义
func Unix(sec int64, nsec int64) Time 
```

* unix有两个参数

* 第一个参数秒级的时间戳
* 第二个参数是纳秒
* 当使用sec时，把nsec置为0，使用nsec时把sec置为0

```go
//秒时间戳->Time
time.Unix(1564063799, 0)
//纳秒时间戳->Time
time.Unix(0, 1564063799695197200)
//毫秒时间戳->Time
time.Unix(0, 1564063799695*int64(time.Millisecond))
```



### Time.Format

format默认使用的时本地时区，如果要指定时区方式如下

```go
var cstZone = time.FixedZone("CST", 8*3600)       // 东八
time.Now().In(cstZone).Format("2006-01-02 15:04:05")
```



### Time.Parse

`time.Parse()` 只会在参数里有指明时区信息、时区信息以 zone offset 形式（如 2018-01-01 12:11:11 +0800 CST）表示、表示结果与本地时区等价时，才会使用本地时区，否则使用读出的时区。若参数里没有指明时区信息，**则使用 UTC 时间**。所以一般建议使用time.ParseInLocation()

```go
time.ParseInLocation("2006-01-02 15:04:05", "2018-01-01 12:11:11",time.Local)
```

