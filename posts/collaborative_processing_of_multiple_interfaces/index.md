# Go 并发实战：利用协程处理多个接口数据


高效地处理多个数据源并将其整合为有意义的结果是开发中一项重要的任务。Go 语言，以其强大的并发特性，为我们提供了优雅而高效的解决方案。那么我们探讨一下如何利用 Go 语言的协程，同时调用多个接口获取数据，并将这些数据无缝地合并为一个完整的数据集。

先假定一个场景：

> _**现有一需求，需要请求n个接口（暂定为3个）获取接口数据，然后对数据进行二次处理并返回。**_

按照过往的经验，我们会依次请求接口拿到数据暂存，最后对数据进行包装处理，这种自上而下的处理方式其实并无不妥，现在想要提高下效率，利用牺牲 cpu 资源来换取查询性能。

先模拟创建几个接口，分别返回(k1,v1)、(k2,v2)、(k3,v4)：

```go
// 模拟接口A
func getDataFromA() map[string]interface{} {
   return map[string]interface{}{
       "key1": "value1",
   }
}

// 模拟接口B
func getDataFromB() map[string]interface{} {
   return map[string]interface{}{
       "key2": "value2",
   }
}

// 模拟接口C
func getDataFromC() map[string]interface{} {
   return map[string]interface{}{
       "key3": "value3",
   }
}
```

开启协程分别请求上述接口：

首先得思考一个问题，协程执行不保证顺序，请求到的数据应该怎么保存？怎么判断全部协程都执行完毕？怎么拿到全部的数据？

上述接口定义中返回的数据均是map，那么我完全可以用map来保存数据，所以我定义方法就可以这么定义：

```go
func getAllData() map[string]interface{} {
	return nil    // 暂时先不做处理
}
```

为了防止主协程先于其他执行结束，需要引入 **sync.WaitGroup** 包控制；所有协程返回的数据，可以用通道来暂存，make 一个容量为 3 的 channel：

```go
func getAllData() map[string]interface{} {
    var wg sync.WaitGroup
    resultChan := make(chan map[string]interface{}, 3)
    return nil    // 暂时先不做处理
}
```

接下来就可以开启协程去调用:

```go
func getAllData() map[string]interface{} {
    var wg sync.WaitGroup
    resultChan := make(chan map[string]interface{}, 3)

    wg.Add(3)
    go func() {
        defer wg.Done()
        resultChan <- getDataFromA()
    }()

    go func() {
        defer wg.Done()
        resultChan <- getDataFromB()
    }()

    go func() {
        defer wg.Done()
        resultChan <- getDataFromC()
    }()

    wg.Wait()
    close(resultChan)
    return nil // 暂时先不做处理
}
```

最后可以对数据做个简单处理，封装成一个大map返回，实际业务当然按需处理：

```go
newMap := make(map[string]interface{})
    for res := range resultChan {
        for k, v := range res {
            newMap [k] = v
    }
}
return newMap
```

执行验证返回结果：

```bash
> [Running] go run "main.go"
> map[key1:value1 key2:value2 key3:value3]
```


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/collaborative_processing_of_multiple_interfaces/  

