# Go 项目实战：如何优雅的处理日志


在 Go 项目开发中，日志处理是一项至关重要的任务。它不仅有助于我们在开发过程中调试代码，还能在生产环境中帮助我们快速定位问题。本文将详细介绍如何在 Go 项目中优雅地处理日志，包括日志的级别、格式、输出以及如何使用第三方日志库等方面。
&lt;!--more--&gt;
## 一、日志级别的重要性

日志级别是控制日志输出的重要手段。通过设置不同的日志级别，我们可以灵活地控制日志的详细程度。在 Go 语言中，常见的日志级别有`DEBUG`、`INFO`、`WARN`、`ERROR`和`FATAL`。不同级别的日志用于记录不同类型的信息，例如：

* `DEBUG`：用于记录详细的调试信息，仅在开发环境中启用。
* `INFO`：用于记录正常的业务流程信息，例如请求的处理、数据的加载等。
* `WARN`：用于记录可能存在的问题或异常情况，但不影响系统的正常运行。
* `ERROR`：用于记录严重的错误信息，这些错误可能导致系统无法正常运行。
* `FATAL`：用于记录非常严重的错误信息，这些错误会导致程序立即退出。

## 二、日志格式的选择

日志格式的选择对于日志的可读性和分析性至关重要。一个好的日志格式应该包含足够的信息，以便我们能够快速定位问题。常见的日志格式有 JSON、XML 和文本格式等。

在 Go 语言中，我们可以使用第三方库来实现不同的日志格式。例如，使用 `logrus` 库可以轻松地将日志格式化为 JSON 格式：

```go {data-open=true}
package main

import (
    &#34;github.com/sirupsen/logrus&#34;
)

func main() {
    // 设置日志格式为JSON
    logrus.SetFormatter(&amp;logrus.JSONFormatter{})

    // 记录不同级别的日志
    logrus.Debug(&#34;这是一条DEBUG级别的日志&#34;)
    logrus.Info(&#34;这是一条INFO级别的日志&#34;)
    logrus.Warn(&#34;这是一条WARN级别的日志&#34;)
    logrus.Error(&#34;这是一条ERROR级别的日志&#34;)
    logrus.Fatal(&#34;这是一条FATAL级别的日志&#34;)
}

```

## 三、日志输出的方式

日志输出的方式有很多种，例如输出到控制台、文件、数据库等。在 Go 语言中，我们可以使用标准库的`log`包来实现基本的日志输出功能。例如，使用标准库 `log.Println` 方法可以将日志输出到控制台：

```go {data-open=true}
package main

import &#34;log&#34;

func main() {
    // 记录日志到控制台
    log.Println(&#34;这是一条日志信息&#34;)
}

```

如果需要将日志输出到文件，我们可以这么做：

```go {data-open=true}
package main

import (
    &#34;log&#34;
    &#34;os&#34;
)

func main() {
    // 创建日志文件
    file, err := os.OpenFile(&#34;app.log&#34;, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    // 设置日志输出到文件
    log.SetOutput(file)

    // 记录日志
    log.Println(&#34;这是一条日志信息&#34;)
}

```

除了常见的输出到控制台或指定文件，我们还可以将日志输出到数据库、Elasticsearch 等其他存储介质中。具体的实现方式还需根据实际需求进行选择。

## 四、使用第三方日志库

虽然说 Go 语言的标准库提供了基本的日志处理功能，但在实际项目中，往往需要结合第三方更为强大的库来满足日常需求：

* `logrus`：一个功能强大的日志库，支持多种日志格式、日志级别、日志输出方式等。
* `zap`：一个高性能的日志库，具有快速、灵活、可扩展等特点。
* `zerolog`：一个极简主义的日志库，专注于提供高性能和简单的 API。

&gt; *ps: 上述简介来自于AI，注意甄别*

这些第三方日志库都提供了丰富的功能和灵活的配置选项，可以帮助我们更好地处理日志。在这里个人比较推荐 **logrus** ，不过具体需求还是得具体选择。

## 五、实战

介绍再多也是空谈，接下来结合具体的项目，我们优雅的配置一下。

默认项目已经安装 logrus ，没有的话可以执行下如下命令：

```shell
go get github.com/sirupsen/logrus
```

### 1、配置config.yaml

为了方便随时更改切换 log 级别或者输出格式，我们可以单独抽离出来实现配置化：

```yaml
log:
  format: json            # 输出格式
  level: debug            # 日志级别
  report_caller: true     # 是否开启调试
```

### 2.配置config.go

有了参数配置，还缺一步解析：

具体的解析可以参考 [Go 项目实战：搭建高效的 Gin Web 目录结构](https://vespeng.tech/posts/go_practical_gin_directory_structure/)

### 2、新建init\_logger.go

在这里我们统一配置 logrus 参数，包括日志级别，输出格式：

```go {data-open=true}
package initializers

import (
    log &#34;github.com/sirupsen/logrus&#34;
)

// InitializeLogger 设置日志输出
func InitializeLogger() error {
    // 设置日志格式
    switch config.Conf.Log.Format {
    case &#34;json&#34;:
        log.SetFormatter(&amp;log.JSONFormatter{})
    case &#34;text&#34;:
        log.SetFormatter(&amp;log.TextFormatter{})
    default:
        log.SetFormatter(&amp;log.JSONFormatter{})
    }

    // 设置日志级别
    switch config.Conf.Log.Level {
    case &#34;debug&#34;:
        log.SetLevel(log.DebugLevel)
    case &#34;info&#34;:
        log.SetLevel(log.InfoLevel)
    case &#34;warn&#34;:
        log.SetLevel(log.WarnLevel)
    case &#34;error&#34;:
        log.SetLevel(log.ErrorLevel)
    case &#34;fatal&#34;:
        log.SetLevel(log.FatalLevel)
    case &#34;panic&#34;:
        log.SetLevel(log.PanicLevel)
    default:
        log.SetLevel(log.InfoLevel)
    }

    // 设置打印调用信息
    log.SetReportCaller(config.Conf.Log.ReportCaller)

    return nil
}
```

### 3、输出日志到文件

控制台打印日志，肯定是不满足一个项目的正常使用的，我们非常有必要将日志持久化到一个单独文件中。

但是这样还不够，会存在另一个问题：日志文件会越来越大后期不利于日志排查。所以还需要对日志进行一个分割，最好的实践方式就是按天分割，所以我们接着在上述初始化文件中去做设置：

```go {data-open=true}
package initializers

import (
    &#34;github.com/lestrrat-go/file-rotatelogs&#34;
    log &#34;github.com/sirupsen/logrus&#34;
    &#34;os&#34;
    &#34;time&#34;
    &#34;your_project/config&#34;
)

// InitializeLogger 设置日志输出并初始化日志文件
func InitializeLogger() error {
    // 设置日志格式
    ...

    // 设置日志级别
    ...

    // 设置打印调用信息
    ...

    // 创建日志目录
    logDir := &#34;../logs&#34;
    err := os.MkdirAll(logDir, 0755)
    if err != nil {
        log.Fatalf(&#34;创建日志目录失败: %v&#34;, err)
    }

    // 设置日志输出，按天切割
    logFilePath := logDir &#43; &#34;/app.%Y%m%d.log&#34;
    writer, err := rotatelogs.New(
        logFilePath,
        rotatelogs.WithLinkName(logDir&#43;&#34;/app.log&#34;),
        rotatelogs.WithMaxAge(7*24*time.Hour),     // 保留7天
        rotatelogs.WithRotationTime(24*time.Hour), // 每天切割一次
    )
    if err != nil {
        log.Fatalf(&#34;设置日志输出失败: %v&#34;, err)
    }
    log.SetOutput(writer)

    return nil
}
```

### 4、调用InitializeLogger()

```go {data-open=true}
package main

import (
  &#34;fmt&#34;
    &#34;github.com/gin-gonic/gin&#34;
    log &#34;github.com/sirupsen/logrus&#34;
    &#34;your_project/api&#34;
    &#34;your_project/config&#34;
    &#34;your_project/initializers&#34;
)

func main() {
    // 加载配置文件
    err := config.LoadConfig()
    if err != nil {
        log.Error(&#34;配置文件加载错误: %v&#34;, err)
        return
    }

    // 初始化 logger
    err = InitializeLogger()
    if err != nil {
        log.Error(&#34;logger 初始化错误: %v&#34;, err)
        return
    }

    r := gin.Default()
    api.SetupRoutes(r, Engine)

    err = r.Run(fmt.Sprintf(&#34;:%d&#34;, config.Conf.App.Port))
    if err != nil {
        log.Error(&#34;服务启动错误: %v&#34;, err)
        return
    }
}
```

到这里一个完整的日志配置就算是配置好了。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_practical_processing_log/  

