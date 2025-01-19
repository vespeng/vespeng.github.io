# Golang 开发环境搭建



## 下载Golang

官网链接：[https://golang.google.cn/dl/](https://golang.google.cn/dl/)

![3326ab02-a030-4902-8934-f95eca5b628c](./images/3326ab02-a030-4902-8934-f95eca5b628c.png)

## 配置环境

1. 安装好之后须添加如下环境变量：

   | 变更名        | 变量值       | 说明                                      |
      | ---------- | --------- | --------------------------------------- |
   | **GOPATH** | E:\gowork | Go语言的工作目录，存放自己编写的 .go 文件、项目、包、编译的二进制文件等 |
   | **GOROOT** | D:\go1.18 | Go的安装路径                                 |

2. 开启go mod 及配置国内代理：

   ```shell
   go env -w GO111MODULE=on
   go env -w GOPROXY=https://goproxy.cn,direct
   ```

## 配置vscode

安装 Go、vscode-go-syntax 两插件

![loading-ag-117](./images/079a6176-fba0-4c94-abe8-aff760e2b4b4.png)

![loading-ag-119](./images/18e22d51-224a-478b-b134-dc808b31b958.png)

&gt;  届时，vscode 会弹出需要安装其他 go 扩展的提示，只需要 Install All 即可 






---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.tech/posts/golang_development_environment/  

