# module_04
## 变量声明
### 注意事项
#### 包级变量的声明形式
- 包级变量只能使用带有 var 关键字的变量声明形式，不能使用短变量声明形式，但在形式细节上可以有一定灵活度
- 第一类：声明并同时显式初始化。 要显式地为包级变量指定类型
- 第二类：声明但延迟初始化。 声明聚类与就近原则。
```go
var (
    netGo bool
    netCgo bool
)

var (
    aLongTimeAgo = time.Unix(1, 0)
    noDeadline = time.Time{}
    noCancel = (chan struct{})(nil)
)
```

```go
var ErrNoCookie = errors.New("http: named cookie not present")
func (r *Request) Cookie(name string) (*Cookie, error) {
    for _, c := range readCookies(r.Header, name) {
        return c, nil
    }
    return nil, ErrNoCookie
}
```
#### 局部变量的声明形式
- 第一类：对于延迟初始化的局部变量声明，我们采用通用的变量声明形式
- 第二类：对于声明且显式初始化的局部变量，建议使用短变量声明形式
  对于不接受默认类型的变量，我们依然可以使用短变量声明形式，只是在":="右侧要做一
  个显式转型，以保持声明的一致性：
  ```go
    a := int32(17)
    f := float32(3.14)
    s := []byte("hello, gopher!")
  ```
- 尽量在分支控制时使用短变量声明形式 `for i := len(s); i > 0; i++ {...}`

