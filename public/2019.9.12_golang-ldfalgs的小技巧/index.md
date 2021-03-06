# golang ldfalgs的小技巧

## 问题

如果想要在go build生成的可执行文件中注入编译时间，git hash等信息。可以在编译的时候使用-ldflags -X参数来注入变量

-ldfflags -X 可以在go install 、go build、go run  、go test中使用

**go build -ldflags "-X  ' packageName.varName=cmd ' "**

<!--more-->

## 样例

```go
package main

import "fmt"

var (
	time = "not set"
	git  = "not set"
)

func main() {
	fmt.Println("build time:", time)
	fmt.Println("git log:", git)
}
```

想要在build的时候改变time和git的值，可以这么做：

powershell(**cmd有问题**):

```shell
go build -ldflags "-X 'main.buildTime=$(date)' -X 'main.gitHash=$(git log --pretty=format:"%h" -1)'"  main.go
```

linux:

```shell
go build  -ldflags "-X 'main.time=$(date -u --rfc-3339=seconds)' -X 'main.git=$(git log --pretty=format:"%h" -1)'"  main.go
```

运行结果:

```
build date: 2019-09-12 06:57:45+00:00
git hash: e1ac7a6
```

## 注意

* 只能给string赋值,不能是bool,int
* 只能是变量，不能是常量
* `-X importpath.name=val`, 其中`importpath`是变量所在包的的路径， `name`是包中定义的变量， val 是需要在编译时设置的变量的值`(string)`
* 如果val是命令的结果，命令中可能有空格之类的特殊符号，建议 `importpath.nmae=val`  用**单引号**括起来,如：\``importpath.nmae=val`\`  

  

## 参考

* https://stackoverflow.com/questions/47509272/how-to-set-package-variable-using-ldflags-x-in-golang-build
* [https://2young.2simple.top/article/golang/golang%E5%9C%A8%E7%BC%96%E8%AF%91%E6%97%B6%E4%BD%BF%E7%94%A8ldflags%E5%8A%A8%E6%80%81%E8%AE%BE%E7%BD%AE%E5%8F%98%E9%87%8F%E7%9A%84%E5%80%BC.html](https://2young.2simple.top/article/golang/golang在编译时使用ldflags动态设置变量的值.html)


