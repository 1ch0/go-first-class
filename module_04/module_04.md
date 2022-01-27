# module_04
## 10 | 变量声明
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

## 11 | 代码块与作用域
### 变量遮蔽（Variable Shadowing）

```go
var a = 11
func foo(n int) {
	a := 1
    a += n
}

func main() {
    fmt.Println("a =", a) // 11
    foo(5)
    fmt.Println("after calling foo, a =", a) // 11
}
```

在这段代码中，函数 foo 调用前后，包级变量 a 的值都没有发生变化。这是因为，虽然 foo 函数中也使用了变量 a，但是 foo 函数中的变量 a 遮蔽了外面的包级变量a，这使得包级变量 a 没有参与到 foo 函数的逻辑中，所以就没有发生变化了。

#### 代码块与作用域

- Go 语言中的代码块是包裹在一对大括号内部的声明和语句序列，如果一对大括号内部没有任何声明或其他语句，我们就把它叫做空代码块。

![image-20220127095918039](C:\Users\86176\AppData\Roaming\Typora\typora-user-images\image-20220127095918039.png)

- 宇宙代码块
- 包代码块
- 文件代码块

- 一个标识符要成为导出标识符需同时具备两个条件：一是这个标识符声明在包代码块中，或者它是一个字段名或方法名；二是它名字第一个字符是一个大写的Unicode 字符。

### 避免变量遮蔽的原则

变量遮蔽错误示例

```go
package main

import (
	"errors"
	"fmt"
)

var a int = 2020

func checkYear() error {
	err := errors.New("wrong year")

	switch a, err := getYear(); a {
	case 2020:
		fmt.Println("it is", a, err)
	case 2021:
		fmt.Println("it is", a)
		err = nil
	}
	fmt.Println("after check, it is", a)
	return err
}

type new int

func getYear() (new, error) {
	var b int16 = 2021
	return new(b), nil
}

func main() {
	err := checkYear()
	if err != nil {
		fmt.Println("call checkYear error:", err)
		return
	}
	fmt.Println("call checkYear ok")
}

```

- 第一个问题：遮蔽预定义标识符。  new
- 第二个问题：遮蔽包代码块中的变量。 switch
- 第三个问题：遮蔽外层显式代码块中的变量。 err

### 利用工具检测变量遮蔽问题

#### go vet

```sh
go install golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow@latest

go vet -vettool=$(which shadow) -strict demo.go
```

```ad-summary
代码块有显式与隐式之分，显式代码块就是包裹在一对配对大括号内部的语句序列，而隐式代码块则不容易肉眼分辨，它是通过 Go 语言规范明确规定的。隐式代码块有五种，分别是宇宙代码块、包代码块、文件代码块、分支控制语句隐式代码块，以及 switch/select的子句隐式代码块
作用域的概念是 Go 源码编译过程中标识符（包括变量）的一个属性。Go 编译器会校验每个标识符的作用域，如果它的使用范围超出其作用域，编译器会报错。
```

## 12 | 基本数据类型

### 被广泛使用的整型

#### 平台无关整型

##### 有符号整型（int8~int64）

| 类型  | 长度（byte） | 取值范围                                  |
| ----- | ------------ | ----------------------------------------- |
| int8  | 1            | -128——127                                 |
| int16 | 2            | -32768——32767                             |
| int32 | 4            | -2147483648——2147483647                   |
| int64 | 8            | -9223372036854775808——9223372036854775807 |



##### 无符号整型（uint8~uint64）

| 类型   | 长度（byte） | 取值范围                |
| ------ | ------------ | ----------------------- |
| uint8  | 1            | 0——255                  |
| uint16 | 2            | 0——65535                |
| uint32 | 4            | 0——4294967295           |
| uint64 | 8            | 0——18446744073709551615 |

在编写有移植性要求的代码时，千万不要强依赖这些类型的长度。如果你不知道这三个类型在目标运行平台上的长度，可以通过 unsafe 包提供的 SizeOf 函数来获取，比如在 x86-64 平台上，它们的长度均为 8

### 整型的溢出问题

无论哪种整型，都有它的取值范围，也就是有它可以表示的值边界。如果这个整型因为参与某个运算，导致结果超出了这个整型的值边界，我们就说发生了整型溢出的问题。由于整型无法表示它溢出后的那个“结果”，所以出现溢出情况后，对应的整型变量的值依然会落到它的取值范围内，只是结果值与我们的预期不符，导致程序逻辑出错

```go
var s int8 = 127
s += 1 // 预期128，实际结果-128
var u uint8 = 1
u -= 2 // 预期-1，实际结果255
```

#### 字面值与格式化输出

数值字面值（Number Literal）

```go
a := 53 // 十进制
b := 0700 // 八进制，以"0"为前缀
c1 := 0xaabbcc // 十六进制，以"0x"为前缀
c2 := 0Xddeeff // 十六进制，以"0X"为前缀

d1 := 0b10000001 // 二进制，以"0b"为前缀
d2 := 0B10000001 // 二进制，以"0B"为前缀
e1 := 0o700 // 八进制，以"0o"为前缀
e2 := 0O700 // 八进制，以"0O"为前缀

# Go 1.13
a := 5_3_7 // 十进制: 537
b := 0b_1000_0111 // 二进制位表示为10000111
c1 := 0_700 // 八进制: 0700
c2 := 0o_700 // 八进制: 0700
d1 := 0x_5c_6d // 十六进制：0x5c6d


var a int8 = 59
fmt.Printf("%b\n", a) //输出二进制：111011
fmt.Printf("%d\n", a) //输出十进制：59
fmt.Printf("%o\n", a) //输出八进制：73
fmt.Printf("%O\n", a) //输出八进制(带0o前缀)：0o73
fmt.Printf("%x\n", a) //输出十六进制(小写)：3b
fmt.Printf("%X\n", a) //输出十六进制(大写)：3B
```

### 浮点型

#### 浮点型的二进制表示

IEEE 754 标准规定了四种表示浮点数值的方式：单精度（32 位）、双精度（64 位）、扩展单精度（43 比特以上）与扩展双精度（79 比特以上，通常以 80 位实现）。后两种其实很少使用

Go 语言提供了 float32 与 float64 两种浮点类型，它们分别对应的就是 IEEE 754 中的单
精度与双精度浮点数值类型。

无论是 float32 还是 float64，它们的变量的默认值都为 0.0，不同的是它们占用的内存空
间大小是不一样的，可以表示的浮点数的范围与精度也不同。

浮点数在内存中的二进制表示分三个部分：符号位、阶码（即经过换算的指
数），以及尾数
$$
(-1)^S * 1.M * 2^{E-offset}
$$

---



% \f is defined as #1f(#2) using the macro
\f\relax{x} = \int_{-\infty}^\infty
    \f\hat\xi\,e^{2 \pi i \xi x}
    \,d\xi

---

$$
% \f is defined as #1f(#2) using the macro
\f\relax{x} = \int_{-\infty}^\infty
    \f\hat\xi\,e^{2 \pi i \xi x}
    \,d\xi
$$

---

#### 字面值与格式化输出

- 直白地用十进制表示的浮点值形式

  ```go
  3.1415
  .15 // 整数部分如果为0，整数部分可以省略不写
  81.80
  82. // 小数部分如果为0，小数点后的0可以省略不写
  ```

- 科学计数法形式

  ```go
  6674.28e-2 // 6674.28 * 10^(-2) = 66.742800
  .12345E+5 // 0.12345 * 10^5 = 12345.000000
  
  # 十六进制科学计数法形式的浮点数
  # 十六进制科学计数法的整数部分、小数部分用的都是十六进制形式，但指数部分依然是十进制形式，并且字面值中的 p/P 代表的幂运算的底数为 2。
  0x2.p10 // 2.0 * 2^10 = 2048.000000
  0x1.Fp+0 // 1.9375 * 2^0 = 1.937500
  
  
  var f float64 = 123.45678
  fmt.Printf("%f\n", f) // 123.456780
  
  # %e 输出的是十进制的科学计数法形式，而 %x 输出的则是十六进制的科学计数法形式
  fmt.Printf("%e\n", f) // 1.234568e+02
  fmt.Printf("%x\n", f) // 0x1.edd3be22e5de1p+06
  ```

### 复数类型

```go
# 第一种，我们可以通过复数字面值直接初始化一个复数类型变量：
var c = 5 + 6i
var d = 0o123 + .12345E+5i // 83+12345i

# 第二种，Go 还提供了 complex 函数，方便我们创建一个 complex128 类型值
var c = complex(5, 6) // 5 + 6i
var d = complex(0o123, .12345E+5) // 83+12345i

# 第三种，你还可以通过 Go 提供的预定义的函数 real 和 imag，来获取一个复数的实部与虚部，返回值为一个浮点类型
var c = complex(5, 6) // 5 + 6i
r := real(c) // 5.000000
i := imag(c) // 6.000000
```



### 创建自定义的数值类型

```go
type MyInt int32

var m int = 5
var n int32 = 6
var a MyInt = m // 错误：在赋值中不能将m（int类型）作为MyInt类型使用
var a MyInt = n // 错误：在赋值中不能将n（int32类型）作为MyInt类型使用

# 要避免这个错误，我们需要借助显式转型，让赋值操作符左右两边的操作数保持类型一致
var m int = 5
var n int32 = 6
var a MyInt = MyInt(m) // ok
var a MyInt = MyInt(n) // ok

# 也可以通过 Go 提供的类型别名（Type Alias）语法来自定义数值类型
type MyInt = int32
var n int32 = 6
var a MyInt = n // ok
```

```go
var f1 float32 = 16777216.0
var f2 float32 = 16777217.0
f1 == f2 // true

# 16777216.0 = 2^24 = (1+.0) * 2^24
# 因为float32的尾数只有23bit，能够表示的下一个数是 (1+2^(-23))*2^24 = 2^24+2 = 16777218.0
# 而16777217.0 = 2^24 + 1 = (1+2^(-24)) * 2^24，尾数得是2^(-24)，需要24bit才能表示
```

### 字符串

```go
const (
GO_SLOGAN = "less is more" // GO_SLOGAN是string类型常量
s1 = "hello, gopher" // s1是string类型常量
)
var s2 = "I love go" // s2是string类型变量
```

- 第一点：string 类型的数据是不可变的，提高了字符串的并发安全性和存储利用率。
