# Go-SJSON 组件，JSON 动态修改新方案


在Go语言 json 处理领域，在 json 数据处理中，读取与修改是两个核心需求。前文介绍的 [`GJSON`](https://vespeng.com/posts/go_gjson_component/) 解决了灵活读取问题，而 `SJSON` 作为其姊妹库，则专注于实现无需结构体定义的 json 动态修改。
<!--more-->
本文将延续对比分析风格，解析 SJSON 的核心价值。

## 一、Go 原生 json 修改方式

Go 原生修改 json 数据，同样需先定义结构体，然后再将 json 数据解析到结构体实例，如：

```go {data-open=true}
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	jsonStr := `{"name":"张三","age":25}`

	var person Person
	err := json.Unmarshal([]byte(jsonStr), &person)
	if err != nil {
		fmt.Println("解析错误:", err)
		return
	}

	person.Age = 35
	newJson, _ := json.Marshal(person)

	fmt.Println(string(newJson))
}
```

## 二、SJSON 组件

### 1. 概述：

SJSON 提供通过路径表达式直接修改 json 字符串的能力，与 GJSON 采用相同路径语法，形成读写闭环。

官网地址：[GitHub - tidwall/sjson](https://github.com/tidwall/sjson)

### 2. 安装：

使用 Go 的包管理工具 `go get` 安装 SJSON：

```shell
go get -u github.com/tidwall/sjson
```

## 三、SJSON 核心用法

### 1. 基础值修改

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/sjson"
)

func main() {
	jsonStr := `{"name":"张三","age":25}`

	// 修改 age 值为 35
	newJson, _ := sjson.Set(jsonStr, "age", 35)

	fmt.Println(string(newJson))
}
```

### 2. 嵌套结构修改

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/sjson"
)

func main() {
	jsonStr := `{
		"name": "张三",
		"age": 25,
		"hobby": {
			"h1": "sing",
            "h2": "dance",
            "h3": "rap",
            "h4": "basketball"
		}`

	// 修改 hobby.h4 的值: basketball => football
	newJson, _ := sjson.Set(jsonStr, "hobby.h4", "football")

	fmt.Println(string(newJson))
}
```

### 3. 数组操作

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/sjson"
)

func main() {
	jsonStr := `{"hobby": ["sing","dance","rap","basketball"]}`

	// 修改 hobby 数组第4个元素为 football
	newJson, _ := sjson.Set(jsonStr, "hobby.3", "football")

	fmt.Println(string(newJson))

	// 追加 hobby 数组第5个元素为 game
	newJson, _ = sjson.Set(jsonStr, "tags.-1", "game")
	fmt.Println(string(newJson))
}
```

### 4. 字段删除

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/sjson"
)

func main() {
	jsonStr := `{"name":"张三","age":25}`

	// 删除age字段
	newJson, _ := sjson.Delete(jsonStr, "age")

	fmt.Println(string(newJson))
}
```

## 四、SJSON 与原生方案对比

- SJSON 摆脱结构体定义束缚，保持原始 json 结构完整性，避免修改后丢失未定义字段的问题。

- SJSON 路径直达修改位置，规避嵌套结构嵌套带来的问题，与 GJSON 组成完整处理链路。

- SJSON 支持运行时动态路径构建，避免硬编码路径带来的问题。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/go_sjson_component/  

