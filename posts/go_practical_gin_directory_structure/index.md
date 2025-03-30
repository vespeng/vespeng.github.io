# Go 项目实战：搭建高效的Gin Web目录结构


## 引言

在当今迅速迭代的软件开发领域，挑选合适的工具与框架对于项目能否顺利推进至关重要。Gin 框架，作为 Go 语言生态中备受青睐的 Web 开发框架，凭借其卓越的性能、简洁的设计以及丰富的功能特性，在众多选项中脱颖而出。本文旨在深入剖析如何在使用 Gin 框架的过程中，构建一个既高效又便于管理的项目架构，助力开发者打造既快速响应又易于维护的 Web 应用程序。

## 一、Gin 概述

引入官网的描述：Gin 是一个使用 Go 语言开发的 Web 框架。 它提供类似 Martini 的 API，但性能更佳，速度提升高达40倍。 如果你是性能和高效的追求者, 你会爱上 Gin。

对比 Beego 框架，Gin 框架采用了极简主义的方法，为追求简单和高性能，没有多余文件或目录，他甚至什么也没有，没有集成任何中间件，一个 main 文件即可启动一个web服务。

正因为如上所述，过分精简对于开发一个项目来说，前期的项目搭建工作就显得尤为重要。

## 二、项目结构设计

有过 Java 开发经验的伙伴应该了解，SpringBoot 遵循着 MVC 的设计理念，这一套设计理念一直沿用至今，他的优秀难以言喻，Gin 框架完全可以参照这个模式来做，如下是我个人设计的一套架构：

```html {data-open=true}
├── /cmd  
│   └── main.go  
├── /config
│   └── config.go  
│   └── config.yaml
├── /docs
├── /internal
│   ├── /api
│   │   ├── v1
│   │   │   ├── /routes.go
│   ├── /app
│   │   ├── loader.go
│   │   ├── db.go
│   │   └── ...
│   ├── /controllers
│   │   ├── user_controller.go
│   │   └── ...
│   ├── /middleware
│   │   ├── error.go
│   │   └── ...
│   ├── /models
│   │   ├── user_entity.go
│   │   └── ...  
│   ├── /repositories  
│   │   ├── user_repository.go
│   │   └── ...  
│   ├── /services  
│   │   ├── user_service.go
│   │   └── ...
│   └── /utils
├── /pkg
├── /scripts
├── /tests
├── .env
├── go.mod
├── go.sum
```

## 三、目录职责

- **`/cmd`**
    * 存放应用的入口文件。
    * **`main.go`**：是整个应用的入口，在这里启动应用。
- **`/config`**
    * 存放应用的配置文件和配置加载逻辑。
    * **`config.go`**：包含配置加载和解析的逻辑。
    * **`config.yaml`**：应用的配置文件，通常包含数据库连接信息、服务器设置等。
- **`/docs`**
    * 存放应用的文档，如API文档、用户手册等。
- **`/internal`**
    * 存放应用的内部逻辑，这些代码不能被外部包所引入，可根据实际需求进而拆分目录。
    * **`api`**：包含应用中核心的业务路由等，即URL路径与控制器方法的映射。
    * **`app`**：包含应用的核心逻辑，如初始化、启动等。
    * **`controllers`**：包含控制器逻辑，处理请求并返回响应。
    * **`middleware`**：存放中间件代码，用于在请求处理流程中的特定阶段执行代码。
    * **`models`**：定义应用的数据模型，通常与数据库表结构对应。
    * **`repositories`**：实现数据访问逻辑，与数据库进行交互。
    * **`services`**：实现业务逻辑，调用repositories中的方法来处理业务需求。
    * **`utils`**：包含通用的工具函数，这些函数可以被多个包所共享。
- **`/pkg`**
    * 存放第三方库，如第三方中间件、工具库等。
- **`/scripts`**
    * 存放各种脚本，如项目部署脚本、测试脚本等。
- **`/tests`**
    * 存放测试代码，包括单元测试、集成测试等。
    * 这里的目录结构可以根据需要自行组织，以支持不同类型的测试。

以上目录结构有助于清晰地分离应用的不同部分，使得代码更加模块化、易于理解和维护。同时，我也参照众多优秀开源项目的目录搭建思想，使其完美遵循了 Go 语言的最佳实践。

## 四、实践

目录搭建好后，开始填充代码

&gt; 下边简单实现集成数据库，配置路由，启动服务

### 1.配置config

在 config.yaml 文件下配置端口和数据库连接，这里选择xorm：

```yaml
# 基础配置
app:
  port: 8080
database:
  driver: mysql
  source: root:123456@tcp(127.0.0.1:3306)/xxx_table?charset=utf8mb4&amp;parseTime=True&amp;loc=Local
```

在 config.go 下解析配置

```go {data-open=true}
package config

import (
    &#34;fmt&#34;
    &#34;github.com/spf13/viper&#34;
)

type Config struct {
    App      AppConfig      `yaml:&#34;app&#34; mapstructure:&#34;app&#34;`
    Database DatabaseConfig `yaml:&#34;database&#34; mapstructure:&#34;database&#34;`
}

type AppConfig struct {
    Port int `mapstructure:&#34;port&#34;`
}

type DatabaseConfig struct {
    Driver string `yaml:&#34;driver&#34; mapstructure:&#34;driver&#34;`
    Source string `yaml:&#34;source&#34; mapstructure:&#34;source&#34;`
}

var Conf *Config

// LoadConfig 加载配置文件
func LoadConfig() error {

    // 设置配置文件路径和名称
    viper.AddConfigPath(&#34;./config&#34;)
    viper.SetConfigName(&#34;config&#34;)
    viper.SetConfigType(&#34;yaml&#34;)

    // 读取配置文件
    err = viper.ReadInConfig()
    if err != nil {
        return fmt.Errorf(&#34;读取配置文件失败: %v&#34;, err)
    }

    // 将配置文件内容解析到 Conf 变量中
    Conf = &amp;Config{}
    err = viper.Unmarshal(Conf)
    if err != nil {
        return fmt.Errorf(&#34;解析配置文件失败: %v&#34;, err)
    }

    return nil
}
```

### 2.配置init

数据库及其他的初始化统一放置到 app 目录下，在这里新建 loader.go 来初始化 mysql，但是为了之后方便管理，我们另单独创建 db.go 文件：

&gt; 如需要加载其他如 redis，那就新建 redis.go 文件

```go {data-open=true}
package app

import (
    _ &#34;github.com/go-sql-driver/mysql&#34;
    &#34;github.com/go-xorm/xorm&#34;
    log &#34;github.com/sirupsen/logrus&#34;
    &#34;yourProject/config&#34;
)

var Engine *xorm.Engine

// InitializeMySQL 数据库初始化
func InitializeMySQL() error {
    var err error
    // 创建数据库引擎
    Engine, err = xorm.NewEngine(config.Conf.Database.Driver, config.Conf.Database.Source)
    if err != nil {
        log.Error(&#34;数据库初始化失败: %v&#34;, err)
        return err
    }

    // 测试数据库连接
    if err = Engine.Ping(); err != nil {
        log.Error(&#34;数据库连接失败: %v&#34;, err)
        return err
    }

    return nil
}
```

app.go 中调用 InitializeMySQL()

```go {data-open=true}
package app

import (
    &#34;fmt&#34;
)

// InitializeAll 初始化所有模块
func InitializeAll() error {
    err := InitializeMySQL()
    if err != nil {
        return fmt.Errorf(&#34;MySQL初始化错误: %v&#34;, err)
    }

    return nil
}
```

### 3.配置model

在 models 下新建 user_entity.go，注意：这个需要和数据库对应

```go {data-open=true}
package models

type User struct {
    Id          int64  `xorm:&#34;pk autoincr &#39;id&#39;&#34;`
    UserID      int64  `xorm:&#34;not null &#39;user_id&#39;&#34;`
    Password    string `xorm:&#34;varchar(50) not null &#39;password&#39;&#34;`
    UserName    string `xorm:&#34;varchar(30) &#39;user_name&#39;&#34;`
    Email       string `xorm:&#34;varchar(50) &#39;email&#39;&#34;`
    PhoneNumber int64  `xorm:&#34;&#39;phone_number&#39;&#34;`
    Sex         string `xorm:&#34;char(1) &#39;sex&#39;&#34;`
    Remark      string `xorm:&#34;varchar(500) &#39;remark&#39;&#34;`
}

// TableName 方法用于返回表名
func (u User) TableName() string {
    return &#34;user&#34;
}
```

### 4.配置controller

在 controllers 下新建 user_controllers.go

```go {data-open=true}
package controllers

import (
    &#34;your_project/internal/services&#34;
    &#34;github.com/gin-gonic/gin&#34;
    &#34;net/http&#34;
)

type UserController struct {
    UserService *services.UserService
}

func NewUserController(UserService *services.UserService) *UserController {
    return &amp;UserController{UserService: UserService}
}

func (uc *UserController) GetUsers(c *gin.Context) {
    users, err := uc.UserService.GetUsers()
        if err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{&#34;error&#34;: &#34;Failed to fetch users&#34;})
            return
        }
    c.JSON(http.StatusOK, gin.H{&#34;users&#34;: users})
}
```

### 5.配置service

在 sevices 下新建 user_service.go

```go {data-open=true}
package services

import (
    &#34;your_project/internal/models&#34;
    &#34;your_project/internal/repositories&#34;
    &#34;github.com/go-xorm/xorm&#34;
)

type UserService struct {
    userRepo *repositories.UserRepository
}

func NewUserService(engine *xorm.Engine) *UserService {
    return &amp;UserService{userRepo: repositories.NewUserRepository(engine)}
}

func (us *UserService) GetUsers() ([]*models.User, error) {
    return us.userRepo.GetUsers()
}
```

### 6.配置repositorie

在 repositories 下新建 user_repo.go

```go {data-open=true}
package repositories

import (
    &#34;your_project/internal/models&#34;
    &#34;github.com/go-xorm/xorm&#34;
)

type UserRepository struct {
    engine *xorm.Engine
}

func NewUserRepository(engine *xorm.Engine) *UserRepository {
    return &amp;UserRepository{engine: engine}
}

// GetUsers 获取所有用户
func (r *UserRepository) GetUsers() ([]*models.User, error) {
    var users []*models.User
    err := r.engine.Table(models.User{}.TableName()).Find(&amp;users)
    return users, err
}
```

### 7.配置api

routes.go 中设置路由，这里设置路由组，为方便日后迭代

```go {data-open=true}
package v1

import (
    &#34;github.com/gin-gonic/gin&#34;
    &#34;github.com/go-xorm/xorm&#34;
    &#34;your_project/internal/controllers&#34;
    &#34;your_project/internal/services&#34;
)

func SetupRoutes(r *gin.Engine, engine *xorm.Engine) {
    // 定义用户路由组
    user := r.Group(&#34;/user&#34;)
    {
        // 创建 UserService 实例
        UserService := services.NewUserService(engine)
        // 创建 UserController 实例
        UserController := controllers.NewUserController(UserService)

        user.GET(&#34;/&#34;, UserController.GetUsers)
    }
}
```

### 8.配置main

```go {data-open=true}
package main

import (
    &#34;fmt&#34;
    &#34;github.com/gin-gonic/gin&#34;
    log &#34;github.com/sirupsen/logrus&#34;
    &#34;your_project/config&#34;
    &#34;your_project/internal/api/v1&#34;
    &#34;your_project/internal/app&#34;
)

func main() {
    // 加载配置文件
    err := config.LoadConfig()
    if err != nil {
        log.Error(&#34;配置文件加载错误: %v&#34;, err)
        return
    }

    // 初始化所有模块
    err = InitializeAll()
    if err != nil {
        log.Error(&#34;模块初始化错误: %v&#34;, err)
        return
    }

    r := gin.Default()
    v1.SetupRoutes(r, Engine)

    err = r.Run(fmt.Sprintf(&#34;:%d&#34;, config.Conf.App.Port))
    if err != nil {
        log.Error(&#34;服务启动错误: %v&#34;, err)
        return
    }
}
```

截至这里，一个基本的查询请求就已经构建完成

### 9.启动项目

cmd 目录下直接运行 main 函数，正常会输出如下信息：

```shell
Listening and serving HTTP on :8080
```

接着访问 [http://localhost:8080/user](http://localhost:8080/user) 正常查询结果回显 json 如下：

```json
{
    &#34;users&#34;: [
        {
            &#34;Id&#34;: 1,
            &#34;UserID&#34;: &#34;000001&#34;,
            &#34;Password&#34;: &#34;123456&#34;,
            ...
        }
    ]
}
```

上述示例，我已经上传至 [GitHub - vespeng/gin-pathway](https://github.com/vespeng/gin-pathway)，欢迎 fork 和 star。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/go_practical_gin_directory_structure/  

