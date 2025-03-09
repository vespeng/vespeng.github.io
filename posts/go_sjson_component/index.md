# Go-SJSON 组件，JSON 动态修改新方案


在Go语言 json 处理领域，在 json 数据处理中，读取与修改是两个核心需求。前文介绍的 `GJSON` 解决了灵活读取问题，而 `SJSON` 作为其姊妹库，则专注于实现无需结构体定义的 json 动态修改。
&lt;!--more--&gt;
本文将延续对比分析风格，解析 SJSON 的核心价值。

## 一、Go 原生 json 修改方式

Go 原生修改 json 数据，同样需先定义结构体，然后再将 json 数据解析到结构体实例，如：

```go {data-open=true}
package main

import (
	&#34;encoding/json&#34;
	&#34;fmt&#34;
)

type Person struct {
	Name string `json:&#34;name&#34;`
	Age  int    `json:&#34;age&#34;`
}

func main() {
	jsonStr := `{&#34;name&#34;:&#34;张三&#34;,&#34;age&#34;:25}`

	var person Person
	err := json.Unmarshal([]byte(jsonStr), &amp;person)
	if err != nil {
		fmt.Println(&#34;解析错误:&#34;, err)
		return
	}

	person.Age = 35
	newJson, _ := json.Marshal(person)

	fmt.Println(string(newJson))
}
```

## 二、SJSON 组件

### 1、概述：

SJSON 提供通过路径表达式直接修改 json 字符串的能力，与 GJSON 采用相同路径语法，形成读写闭环。

官网地址：[GitHub - tidwall/sjson](https://github.com/tidwall/sjson)

### 2、安装：

使用 Go 的包管理工具 `go get` 安装 SJSON：

```shell
go get -u github.com/tidwall/sjson
```

## 三、SJSON核心用法

### 1. 基础值修改

```go {data-open=true}
package main

import (
	&#34;fmt&#34;
	&#34;github.com/tidwall/sjson&#34;
)

func main() {
	jsonStr := `{&#34;name&#34;:&#34;张三&#34;,&#34;age&#34;:25}`

	// 修改 age 值为 35
	newJson, _ := sjson.Set(jsonStr, &#34;age&#34;, 35)

	fmt.Println(string(newJson))
}
```

### 2. 嵌套结构修改

```go {data-open=true}
package main

import (
	&#34;fmt&#34;
	&#34;github.com/tidwall/sjson&#34;
)

func main() {
	jsonStr := `{
		&#34;name&#34;: &#34;张三&#34;,
		&#34;age&#34;: 25,
		&#34;hobby&#34;: {
			&#34;sing&#34;: &#34;只因你太美&#34;,
			&#34;dance&#34;: &#34;背带裤&#34;,
			&#34;rap&#34;: &#34;kun&#34;,
			&#34;ball&#34;: &#34;篮球&#34;
		}`

	// 修改 hobby.sing 的值: 只因你太美 =&gt; 重生
	newJson, _ := sjson.Set(jsonStr, &#34;hobby.sing&#34;, &#34;重生&#34;)

	fmt.Println(string(newJson))
}
```

### 3. 数组操作

```go {data-open=true}
package main

import (
	&#34;fmt&#34;
	&#34;github.com/tidwall/sjson&#34;
)

func main() {
	jsonStr := `{&#34;hobby&#34;: [&#34;sing&#34;,&#34;dance&#34;,&#34;rap&#34;,&#34;ball&#34;]}`

	// 修改 hobby 数组第4个元素为 play
	newJson, _ := sjson.Set(jsonStr, &#34;hobby.3&#34;, &#34;play&#34;)

	fmt.Println(string(newJson))

	// 追加 hobby 数组第5个元素为 play
	newJson, _ = sjson.Set(jsonStr, &#34;tags.-1&#34;, &#34;play&#34;)
	fmt.Println(string(newJson))
}
```

### 4. 字段删除

```go {data-open=true}
package main

import (
	&#34;fmt&#34;
	&#34;github.com/tidwall/sjson&#34;
)

func main() {
	jsonStr := `{&#34;name&#34;:&#34;张三&#34;,&#34;age&#34;:25}`

	// 删除age字段
	newJson, _ := sjson.Delete(jsonStr, &#34;age&#34;)

	fmt.Println(string(newJson))
}
```

## 四、SJSON 与原生方案对比

- SJSON 摆脱结构体定义束缚，避免了完整解析，保持原始 json 结构完整性，避免了修改后丢失未定义字段的问题。

- SJSON 路径直达修改位置，规避嵌套结构嵌套带来的问题，与 GJSON 组成完整处理链路。

- SJSON 支持运行时动态路径构建，避免了手动拼接路径带来的问题。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_sjson_component/  

