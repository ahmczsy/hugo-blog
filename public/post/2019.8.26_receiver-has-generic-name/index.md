# Receiver has generic name

#### 问题

Golang的**方法接收者**就是在函数名前的括号内的东西  如

```go
func (self Car) run() // (self car)就是方法接收者
```

在JetBrains系类的开发工具中（IntelliJ、Goland）如果方法接收者名字是**self  me  this**类似的词，IDE会提示*Receiver has generic name*信息 如下图所示

![mW6QpQ.png](https://s2.ax1x.com/2019/08/26/mW6QpQ.png)
<!--more-->


#### Go中的函数

在Go中，无论是方法还是函数都可以把他作为一个普通的函数来调用

Demo:

```go
type Car struct {
	i int
}

func (c Car) run() {
	fmt.Println("my receiver is:", c.i)
}

func TestSelf(t *testing.T) {
	a := Car{1}
	a.run()
	Car.run(Car{22})
}
```

运行结果如下：

```
my receiver is: 1
my receiver is: 22
```

Go的方法只是语法糖而已，本质还是第一个参数是接收者的普通函数

#### 解释

在其他面向对象的语言中**this self me**这些字段都有具体的含义，比如可以访问自己的私有方法或字段或者有其他特殊的含义。

但是在go中最好不用使用这些词，因为方法接收者只是一个普通的参数而，起不到这些词字面上的含义

go官方的代码规范也提到了这一点

> The name of a method's receiver should be a reflection of its identity; often a one or two letter abbreviation of its type suffices (such as "c" or "cl" for "Client"). Don't use generic names such as "me", "this" or "self", identifiers typical of object-oriented languages that gives the method a special meaning. In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.



如果想关闭掉这个提示可以在Setting->Editor->Inspections->Go->Code style issues 中把Receiver has generic name这一项取消

![mWcE34.png](https://s2.ax1x.com/2019/08/26/mWcE34.png)



#### 参考

* https://stackoverflow.com/questions/23482068/in-go-is-naming-the-receiver-variable-self-misleading-or-good-practice

* https://stackoverflow.com/questions/38025743/go-why-shouldnt-use-this-for-method-receiver-name
* https://github.com/golang/go/wiki/CodeReviewComments#receiver-names
