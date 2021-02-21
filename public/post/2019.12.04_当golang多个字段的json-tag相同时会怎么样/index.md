# 当golang多个字段的json tag相同时会怎么样?


## 问题

今天我在写代码的时候遇到了标题所说的问题，我用google搜索了下也没有找到想要的答案。后来通过几个测试用例试了下这个发现问题有点意思

## 情况1  都是普通字段

```go
type sameTag struct {
	A     string `json:"a"`
	B     string `json:"a"`
	Other string `json:"other"`
}

func TestSameJson(t *testing.T) {
	tag := sameTag{"a", "b", "other filed"}
	bytes, e := json.Marshal(tag)
	fmt.Println(string(bytes))
	fmt.Println(e)
}
```

输出结果:

```
{"other":"other filed"}
<nil>
```

上面代码的A、B字段的tag相同。该struct在序列化后只保留了other字段，AB字段因为冲突直接消失了

## 情况2  普通字段和匿名字段

```go
type innerStruct struct {
	A string `json:"a"`
}
type sameTag struct {
	innerStruct
	B     string `json:"a"`
	Other string `json:"other"`
}

func TestSameJson(t *testing.T) {
	tag := sameTag{innerStruct{"inner"}, "outter", "other filed"}
	bytes, e := json.Marshal(tag)
	fmt.Println(string(bytes))
	fmt.Println(e)
}
```

输出结果：

```
{"a":"outter","other":"other filed"}
<nil>
```

这次的AB的tag都相同，但是序列化结果采用了B字段。因为B字段的优先级比较高，一般字段的优先级比匿名字段高

## 情况3 都是匿名字段

```go
type innerA struct {
	A string `json:"a"`
}
type innerB struct {
	B string `json:"a"`
}
type sameTag struct {
	innerA
	innerB
	Other string `json:"other"`
}

func TestSameJson(t *testing.T) {
	tag := sameTag{innerA{"innerA"}, innerB{"innerB"}, "other filed"}
	bytes, e := json.Marshal(tag)
	fmt.Println(string(bytes))
	fmt.Println(e)
}
```

输出结果:

```
{"other":"other filed"}
<nil>
```

这次输出和情况1一样，还是优先级的问题。innerA和innerB都是匿名字段所以AB字段的优先级一样，这样就冲突了，序列化忽略这两个字段。

## 总结

总的来说就是因为字段优先级的原因，会采用优先级高的字段。当优先级相同时跳过这些相同的字段。

这段逻辑时json的源码中也有体现

encode.go文件中的`dominantField`方法就是来判断存不存在优先级相同的字段

![golang源码](1575458496902.png)

>```
>dominantField looks through the fields, all of which are known to have the same name, to find the single field that dominates the others using Go's embedding rules, modified by the presence of JSON tags. If there are multiple top-level fields, the boolean will be false: This condition is an error in Go and we skip all the fields.
>```
