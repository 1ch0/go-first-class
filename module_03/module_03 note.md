# module_03 note

## 注意点

- go mod tidy下载的第三方包一般在$GOPATH/pkg/mod下面。如果没有设置GOPATH  环境变量，其默认值为你的home路径下的go文件夹。这样第三方包就在go文件夹的pkg/mod下
  面。
- Go Module 本身就支持可再现构建，而无需使用 vendor。 当然 Go Module 机制也保留了 vendor 目录（通过  go mod vendor 可以生成 vendor 下的依赖包，通过 go build -mod=vendor 可以实现  基于 vendor 的构建）。一般我们仅保留项目根目录下的 vendor 目录，否则会造成不必  要的依赖选择的复杂性。

### 包依赖管理

- 如果你没有显式设置 GOPATH 环境变量，Go 会将 GOPATH 设置为默认 值，不同操作系统下默认值的路径不同，在 macOS 或 Linux 上，它的默认值是 $HOME/go。
- go mod tidy 命令会扫描 Go 源码，并自动找出项目依赖的外部 Go Module 以及版本，下载这些依赖并更新本地的 go.mod 文件
- Go 的“语义导入版本”机制，也就是说通过在包导入路径中引入主版本号的方  式，来区别同一个包的不兼容版本
- 在声明它的导入路径的基础上，加上版本号信息

#### 修改依赖
1. go get 对应版本，go mod 会自动替换。
2. go mod edit -require= ; 然后 go mod tidy

#### 移出依赖
go mod tidy

#### 基于 vendor 构建
要基于 vendor 构建，而不是基于本地缓存的 Go Module 构建，我们需要在 go build 后面加上 -mod=vendor 参数

```ad-summary
通过 go get 我们可以升级或降级某依赖的版本，如果升级或降级前后的版本不兼容，这里千万注意别忘了变化包导入路径中的版本号，这是 Go 语义导入版本机制的要求；
通过 go mod tidy，我们可以自动分析 Go 源码的依赖变更，包括依赖的新增、版本变更以及删除，并更新 go.mod 中的依赖信息。
通过 go mod vendor，我们依旧可以支持 vendor 机制，并且可以对 vendor 目录下缓存的依赖包进行自动管理。
```