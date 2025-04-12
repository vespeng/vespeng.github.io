# Go 项目实战：如何部署多环境开发


在 Go 项目的开发过程中，能够在不同的环境（如开发、测试、生产）中进行灵活部署是至关重要的。不同环境通常需要不同的配置，如服务器端口、数据库连接信息、缓存设置等。
<!--more-->
对于 Java 的 SpringBoot 框架来说，可以直接在 `application.yml ` 中指定一个环境配置文件，通常`application_dev.yml ` 代表开发环境，那么 go 可否参考这种方式呢？

接下来本文将详细介绍如何使用多种方式来实现多环境开发部署，重点围绕 `config.yaml` 文件和 `config.go` 文件来进行配置读取和环境区分。

## Go 中的系统环境变量

先来解释一个概念，在 Go 语言中，系统环境变量是操作系统为每个进程提供的键值对集合。这些环境变量可以用于配置应用程序的行为、连接数据库、设置日志级别等。Go 提供了标准库 `os` 来读取和操作这些环境变量。

## 实战

### 1.编写.env文件

在项目根目录下新建一个 .env 文件，配置如下：

```shell
APP_ENV=dev
```

### 2.获取环境变量

使用 `os.Getenv` 函数可以获取指定名称的环境变量值。

```go
env := os.Getenv("APP_ENV")
```

- **`APP_ENV`** 是一个环境变量名，用于标识应用程序的运行环境（如开发、测试、生产等）。
- 如果 `APP_ENV` 未设置，`os.Getenv("APP_ENV")` 将返回空字符串。

### 3.设置默认值

我们需要一个默认的环境，如果 `APP_ENV` 未设置，将其设为 `"dev"`：

```go
env := os.Getenv("APP_ENV")
if env == "" {
    env = "dev" // 默认环境为 dev
}
```

这样可以确保即使没有显式设置 `APP_ENV`，程序也能有一个合理的默认行为。

### 4.加载 .env 文件

使用 `github.com/joho/godotenv` 包来加载 `.env` 文件中的环境变量。`.env` 文件通常用于本地开发环境，避免将敏感信息硬编码到代码中，这里其实挺像 vue 的环境加载方式。

```go
err := godotenv.Load()
if err != nil {
    return fmt.Errorf("加载 .env 文件失败: %v", err)
}
```

这行代码会读取项目根目录下的 `.env` 文件，并将其中定义的环境变量加载到当前进程中。

### 5.动态选择配置文件

根据 `APP_ENV` 的值，动态选择不同的配置文件：

```go
viper.AddConfigPath("./config")
viper.SetConfigName(fmt.Sprintf("config_%s", env))
viper.SetConfigType("yaml")

err = viper.ReadInConfig()
if err != nil {
    return fmt.Errorf("读取配置文件失败: %v", err)
}
```

这段代码会根据 `APP_ENV` 的值（例如 `dev` 或 `production`），选择对应的配置文件（如 `config_dev.yaml` 或 `config_prod.yaml`）。这样可以根据不同的环境加载不同的配置。

### 6.解析配置文件

使用 `viper.Unmarshal` 将配置文件的内容解析到结构体中：

```go
Conf = &Config{}
err = viper.Unmarshal(Conf)
if err != nil {
    return fmt.Errorf("解析配置文件失败: %v", err)
}
```

`viper.Unmarshal` 会将配置文件中的键值对映射到结构体字段上，前提是结构体字段标签（如 `yaml` 和 `mapstructure`）与配置文件中的键匹配。

通过上述方式，我们可以根据项目的实际需求和情况，确保项目在不同环境下都能正确配置并稳定运行。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_practical_multi_environment_development/  

