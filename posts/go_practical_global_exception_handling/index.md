# Go 项目实战：全局异常处理


在 Go 项目开发中，有效的异常处理是确保程序健壮性和稳定性的关键因素之一。全局异常处理机制能够统一处理项目中可能出现的各种异常情况，提高代码的可读性、可维护性以及错误处理的一致性。
&lt;!--more--&gt;
## 一、Go 中的错误处理机制

在 Go 语言中，并没有像其他语言那样的传统异常机制。而是期望开发者主动去识别处理这种“异常”，通过返回值来表示可能出现的错误。

通常情况下，函数会返回一个结果集和一个错误值，我们需要判断错误值是否为 `nil`，如果不为 `nil` 则表示出现了“异常”。

```go {data-open=true}
package main

import (
    &#34;fmt&#34;
)

// 模拟一个会返回错误的函数
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf(&#34;除数不能为 0&#34;)
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println(&#34;出错啦:&#34;, err)
        return
    }
    fmt.Println(&#34;结果是:&#34;, result)
}
```

## 二、Go 中的 panic

当程序遇到无法处理的错误时，就会被提示`panic`，程序会直接崩溃。

`recover` 函数用于捕获 `panic` 抛出的信息，让程序从 panic 状态恢复继续正常执行，前提 recover 只能在 `defer` 函数中使用。

```go {data-open=true}
package main

import (
    &#34;fmt&#34;
)

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println(&#34;已捕获到恐慌:&#34;, r)
        }
    }()

    // 手动触发一个 panic
    panic(&#34;这是一个恐慌！&#34;)
}


// 输出：
// 已捕获到恐慌:这是一个恐慌！
```

三、实现全局异常处理
----------

根据上述其实不难发现，错误处理是显式的，我们可以做前置判断，根据具体情况进行处理，但是`panic` 处理通常是隐式的，一旦被调用 panic 函数，程序的执行流程会被打乱，需捕获 panic 才能恢复程序的正常执行。

所以针对这种隐式的、在编程过程中无法提前预知的错误，就很有必要做一层异常的处理，最好可以是全局处理。

为了实现全局异常处理，我们可以创建一个中间件或者全局的异常处理函数。

```go {data-open=true}
func GlobalErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if err := recover(); err!= nil {
                log.Printf(&#34;Recovered from panic: %v&#34;, err)
                c.JSON(500, gin.H{
                    &#34;message&#34;: &#34;Internal Server Error&#34;,
                })
                c.Abort()
            }
        }()
        c.Next()
    }
}
```

## 四、在项目中的应用

在实际的项目中，我们可以将这个全局异常处理中间件应用到 HTTP 服务器的路由处理中。

```go {data-open=true}
package main

import (
    &#34;github.com/gin-gonic/gin&#34;
    &#34;log&#34;
)

func main() {
    r := gin.Default()
    // 应用全局异常处理中间件
    r.Use(GlobalErrorHandler())

    r.GET(&#34;/ping&#34;, func(c *gin.Context) {
        // 模拟异常
        panic(&#34;Something went wrong!&#34;)
    })

    r.Run(&#34;:8080&#34;)
}
```

这样下来，在程序的后续处理中，一旦遇到 panic 就会被捕获，从而不影响程序的继续运行。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_practical_global_exception_handling/  

