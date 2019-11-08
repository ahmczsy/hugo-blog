---
title: "MarshalJSON没有调用、无效？"
date: 2019-11-08T19:07:02+08:00
keywords: []
description: ""
tags: []
categories: []
draft: true
---

## 问题

今天我在golang自定义json的序列化时，发现我的struct实现`MarshalJSON()`竟然没有效果，还是按照默认的方式来序列化。这让我非常郁闷☹，后来我查找了一些资料发现这个问题不简单。。

下面是我有问题的代码

```go
type str string

func (s *str) MarshalJSON() (data []byte, err error) {
	return json.Marshal("bbb")
}

func TestStr(t *testing.T) {
	var s str = "aaa"
	bytes, _ := json.Marshal(s)
	fmt.Println(string(bytes))
    //我想让他序列化为"bbb"但是输出结果还是"aaa"
}
```

##  pointer receiver  和value receiver

```go
type T struct{}
func (t *T) pointerMethod() { } // pointer receiver
func (t T) valueMethod() { } // value receiver
```

上面两个方法分别就是pointer receiver和value receiver

由于golang没有面向对象那一套东西，上面的两行代码可以视为下面的形式

```go
func  pointerMethod(t *T) { } 
func  valueMethod(t T) { } 
```

所以在golang中value变量只能调用value receiver的方法，pointer只能调用pointer receiver的方法。比如：

```go
var t1 T
t1.valueMethod()//正确
t1.pointerMethod()//错误?

var t2 *T
t2.valueMethod()//错误？
t2.pointerMethod()//正确
```

其实我们实际可以相互调用的

```go
var t1 T
t1.valueMethod()//正确
t1.pointerMethod()//正确

var t2 *T
t2.valueMethod()//正确
t2.pointerMethod()//正确
```

因为golang的底层帮我们做了一层隐式转换

`t1.pointerMethod()` == `(&t1).pointerMethod()`

`t2.valueMethod()`==`(*t2).valueMethod()`

**总的来说**

* `T`可以调用value receiver和pointer receiver方法
* `*T`如果是可以取地址(*addressable*)的变量，也是可以调用value receiver和pointer receiver方法
* 如果`*T`是不可去地址的变量，那只能调用pointer receiver方法

关于什么是可以去地址的变量，什么是不可以取地址。可以参考这篇文章[链接]( https://colobu.com/2018/02/27/go-addressable/ )

## 参考

*  https://stackoverflow.com/questions/33587227/golang-method-sets-pointer-vs-value-receiver 
*  https://stackoverflow.com/questions/21390979/custom-marshaljson-never-gets-called-in-go 
*  http://shanks.leanote.com/post/Untitled-55ca439338f41148cd000759-17 
*  https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/ 
*  https://colobu.com/2018/02/27/go-addressable/ 

*  https://tonybai.com/2015/09/17/7-things-you-may-not-pay-attation-to-in-go/ 