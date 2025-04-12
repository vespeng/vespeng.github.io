# Go-GJSON 组件，解锁 JSON 读取新姿势


在 Go 语言开发领域，json 数据处理是极为常见的任务。Go 标准库提供了 `encoding/json` 包用于处理 json 数据，同时第三方库 `GJSON` & `SJSON` 也在 json 处理方面表现出色。
<!--more-->
本文将深入探讨下 GJSON 组件，通过与原生处理方式对比，它存在什么特别之处，它的优势体现在哪。

## 一、Go 原生 json 读取方式

Go 原生读取 json 数据，通常需先定义结构体，然后再将 json 数据解析到结构体实例，如：

```json
{
    "name": "张三",
    "age": 25
}
```

具体处理逻辑：

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
    if err!= nil {
        fmt.Println("解析错误:", err)
        return
    }

    fmt.Println("Name:", person.Name)
    fmt.Println("Age:", person.Age)
}
```

这种方式虽能准确解析 json 数据，但如果 json 存在多层嵌套，层级过度包装，那么结构体定义以及解析过程就会变得相当繁琐。

## 二、GJSON 组件

### 1.概述：

GJSON 是一个轻量级且高性能的 JSON 解析库，它允许开发者通过简洁的语法，无需定义结构体，就能快速提取 JSON 数据中的特定值。

官网地址：[GitHub - tidwall/gjson](https://github.com/tidwall/gjson)

### 2.安装：

使用 Go 的包管理工具 `go get` 安装 GJSON：

```shell
go get -u github.com/tidwall/gjson
```

## 三、GJSON 基本用法

### 1.简单 json 数据获取

对于简单的 json，像前面那个例子，直接用 `gjson.Get` 方法，传入 json 字符串和要获取的字段名，就能拿到对应的值。比如获取 `name` 字段，`gjson.Get(jsonStr, "name")` 就可以搞定，例如：

```go {data-open=true}
package main

import (
    "fmt"
    "github.com/tidwall/gjson"
)

func main() {
    jsonStr := `{"name":"张三","age":25}`

    name := gjson.Get(jsonStr, "name")
    age := gjson.Get(jsonStr, "age")

    fmt.Println("Name:", name.String())
    fmt.Println("Age:", age.Int())
}
```

### 2.嵌套 json 数据获取

上述提到，原生的处理方式对于多层级的 json 很不友好，然而 gjon 可以直接通过点号分隔路径定位数据，这时候它的优势就逐渐明显，例如：

```json {data-open=true}
{
    "name": "张三",
    "age": 25,
    "hobby": {
        "sing": "只因你太美",
        "dance": "背带裤",
        "rap": "kun",
        "ball": "篮球"
    }
}
```

具体处理逻辑：

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/gjson"
)

func main() {
	jsonStr := `{
		"name": "张三",
		"age": 25,
		"hobby": {
			"sing": "只因你太美",
			"dance": "背带裤",
			"rap": "kun",
			"ball": "篮球"
		}`

	name := gjson.Get(jsonStr, "name")
	ball := gjson.Get(jsonStr, "hobby.ball")

	fmt.Println("Name:", name.String())
	fmt.Println("ball:", ball.String())
}
```

相比原生方式，无需复杂结构体定义，操作更简便。

### 3. json 数组获取

如果在 json 中嵌套了数组，对于这种的处理也比较简单，直接通过数组下标来定位数据即可，如：

```go {data-open=true}
package main

import (
	"fmt"
	"github.com/tidwall/gjson"
)

func main() {
	jsonStr := `{"hobby": ["sing","dance","rap","ball"]}`

	hobby := gjson.Get(jsonStr, "hobby.3")
  
    // 输出第4个爱好
	fmt.Println("hobby:", hobby.String())
}
```

相比于原生方式处理数组，得先解析成切片，操作起来就没这么直接。

## 四、GJSON 与原生 JSON 处理方式对比

- GJSON 语法简单直观，熟悉 json 结构即可快速上手，无需学习结构体定义及标签使用等知识。而原生方式在结构体定义上相对复杂，尤其是处理复杂 json 结构时。

- GJSON 无需将整个 json 数据解析为结构体，在处理大型 json 数据时，内存占用少，解析速度快。原生方式在解析复杂 json 数据时，结构体构建和内存分配开销较大。

- GJSON 对各种复杂 json 结构都能灵活应对，根据需求按路径获取数据，无需频繁修改代码结构。原生方式则需根据 json 结构变化，频繁修改结构体定义，灵活性较差。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_gjson_component/  

