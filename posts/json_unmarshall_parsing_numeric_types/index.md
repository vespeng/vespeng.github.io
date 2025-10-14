# Json.Unmarshal 解析数值类型（踩坑）


首先我们先明确下 json 包下 Unmarshal() 函数是什么：

它是 Go 语言标准库 `encoding/json` 中的一个函数，用于将 JSON 数据解析为 Go 语言中的数据结构。它的作用是将一个 JSON 格式的字节切片（`[]byte`）转换为对应的 Go 语言数据类型，如结构体、切片、映射等。
<!--more-->
其次了解了它的作用后，再来看下这个坑点：

假设有一个 json 串如下：

```json
{
    "id": 1,
    "name": "张三",
    "age": 20
}
```

现在要将它解析成一个 map，拿到 json 原始的数据，方便后续处理：

```go {data-open=true}
func main() {
	str := "{\"id\":1,\"name\":\"张三\",\"age\":20}"
	jsonMap := make(map[string]interface{})
	json.Unmarshal([]byte(str), &amp;jsonMap)
	// 遍历map
	for key, value := range jsonMap {
		fmt.Printf("key: %s, value: %v\n", key, value)
	}
}

// 输出：
// key: id, value: 1
// key: name, value: 张三
// key: age, value: 20
```

这样看着确实没什么问题，每个 key、value 值都是按照预期输出；

现在我把 json 调整一下，假设 id 是一个毫秒级时间戳 **1736325205000（13 位）：**

```go {data-open=true}
func main() {
	str := "{\"id\":1736325205000,\"name\":\"张三\",\"age\":20}"
	jsonMap := make(map[string]interface{})
	json.Unmarshal([]byte(str), &amp;jsonMap)
	// 遍历map
	for key, value := range jsonMap {
		fmt.Printf("key: %s, value: %v\n", key, value)
	}
}

// 输出
// key: id, value: 1.736325205e+12
// key: name, value: 张三
// key: age, value: 20
```

此时坑来了， id 的值变成了一个科学计数法的字符串，显然这不符合我的预期；

那么为什么会变成这样呢？

首先观察到我使用了 %v 进行处理，然而 json 中原本的数据是一个 int，我应该用处理 int 的占位符 %d ：

```go {data-open=true}
func main() {
	str := "{\"id\":1736325205000,\"name\":\"张三\",\"age\":20}"
	jsonMap := make(map[string]interface{)
	json.Unmarshal([]byte(str), &amp;jsonMap)
	fmt.Printf("%d",jsonMap["id"])
}

// 输出
// %!d(float64=1.736325205e+12)
```

到这里本以为是 ok 的，结果输出了这么个玩意，仔细读一下发现 float64 ，输出这个的原因是我要把一个 float64 的元素强行用 int 类型的占位符进行处理；

所以现在进一步清晰了，json.Unmarshal 函数会把 id 转为 float64；

那么问题又来了，为什么它会把 id 转为 float64 类型呢？id == 1 的时候为什么能正常输出呢？

进源码，看看函数内部做了什么：

```go {data-open=true}
func Unmarshal(data []byte, v any) error {
	// Check for well-formedness.
	// Avoids filling out half a data structure
	// before discovering a JSON syntax error.
	var d decodeState
	err := checkValid(data, &amp;d.scan)
	if err != nil {
		return err
	}

	d.init(data)
	return d.unmarshal(v)
}

// 可以看到 checkValid() 方法引用了 decodeState 结构体
// 进结构体里看下：

// decodeState represents the state while decoding a JSON value.
type decodeState struct {
	data                  []byte
	off                   int // next read offset in data
	opcode                int // last read result
	scan                  scanner
	errorContext          *errorContext
	savedError            error
	useNumber             bool
	disallowUnknownFields bool
}

// 初步观察有个 bool 类型的 useNumber 属性
// 接着看下这个结构体具体的实现方法：

// convertNumber converts the number literal s to a float64 or a Number
// depending on the setting of d.useNumber.
func (d *decodeState) convertNumber(s string) (any, error) {
	if d.useNumber {
		return Number(s), nil
	}
	f, err := strconv.ParseFloat(s, 64)
	if err != nil {
		return nil, &amp;UnmarshalTypeError{Value: "number " + s, Type: reflect.TypeOf(0.0), Offset: int64(d.off)}
	}
	return f, nil
}

// 到这里大概能清楚，是这个方法把我的 id 转成了 float64，但是再转之前还有一层 if 会把原始值输出；
// 接下来就回去上一级，看看 d.scan 到底做了什么：
// 努力中...
// ——————看不懂
```

经过多方查找：

理论上 json 会把超过 int64 长度的数值转成 float64，但是这个说法经实践不成立，毫秒级时间戳 13 位，远没有超过 int64 的最大长度；

多次翻阅资料后，有一个说法比较靠谱：

当处理非常大的整数（如毫秒级的时间戳）时，如果直接使用 Go 语言中的整数类型（如 `int` 或 `int64`），可能会因为超出这些类型的表示范围而导致溢出。虽然 `int64` 类型在大多数情况下可以容纳毫秒级的时间戳，但为了确保能够处理所有可能的 JSON 数字，`encoding/json` 包选择了 `float64` 类型作为默认解析结果。

到这里其实我们最初的目的也能够轻松处理：

```go {data-open=true}
func main() {
	str := "{\"id\":1736325205000,\"name\":\"张三\",\"age\":20}"
	jsonMap := make(map[string]interface{})
	json.Unmarshal([]byte(str), &amp;jsonMap)

	// 断言类型为 float64
	fmt.Println(jsonMap["id"])
	if f, ok := jsonMap["id"].(float64); ok {
		fmt.Println(int(f))
	}
}

// 输出
// 1736325205000
```


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/json_unmarshall_parsing_numeric_types/  

