# Go语言简介

## Go语言标准库常用的包及功能

| Go语言标准库包名 |                            功  能                            |
| :--------------- | :----------------------------------------------------------: |
| bufio            |                      带缓冲的 I/O 操作                       |
| bytes            |                         实现字节操作                         |
| container        |                 封装堆、列表和环形列表等容器                 |
| crypto           |                           加密算法                           |
| database         |                       数据库驱动和接口                       |
| debug            |                各种调试文件格式访问及调试功能                |
| encoding         |               常见算法如 JSON、XML、Base64 等                |
| flag             |                          命令行解析                          |
| fmt              |                          格式化操作                          |
| go               | Go语言的词法、语法树、类型等。可通过这个包进行代码信息提取和修改 |
| html             |                     HTML 转义及模板系统                      |
| image            |                   常见图形格式的访问及生成                   |
| io               |               实现 I/O 原始访问接口及访问封装                |
| math             |                            数学库                            |
| net              |        网络库，支持 Socket、HTTP、邮件、RPC、SMTP 等         |
| os               |                操作系统平台不依赖平台操作封装                |
| path             |               兼容各操作系统的路径操作实用函数               |
| plugin           |    Go 1.7 加入的插件系统。支持将代码编译为插件，按需加载     |
| reflect          | 语言反射支持。可以动态获得代码中的类型信息，获取和修改变量的值 |
| regexp           |                        正则表达式封装                        |
| runtime          |                          运行时接口                          |
| sort             |                           排序接口                           |
| strings          |                  字符串转换、解析及实用函数                  |
| time             |                           时间接口                           |
| text             |                   文本模板及 Token 词法器                    |

## Go语言代码风格

* 去掉循环冗余括号

```go
for a := 0;a<10;a++ {
    // 循环代码
}
```

* 去掉表达式冗余括号

```go
if 表达式 {
    // 表达式成立
}
```

* 强制的代码风格

Go语言中，左括号必须紧接着语句不换行。其他样式的括号将被视为代码编译错误。

* 不再纠结于` i++` 和 `++i`，在Go语言中自增只有一种写法：

```go
i++
```

## Go语言工程结构

一个Go语言项目的目录一般包含以下三个子目录：

- src 目录：放置项目和库的源文件；
- pkg 目录：放置编译后生成的包/库的归档文件；
- bin 目录：放置编译后生成的可执行文件。

## 第一个go语言程序

```go
// hello_world.go*
package main // 包，表明代码所在的模块（包）

import "fmt" // 引⼊依赖包

// main函数，或主函数，是一个程序的入口函数。
func main() {
  fmt.Println("Hello, World!")
}
```

Go语言以“包”作为管理单位，每个Go源文件必须先声明它所属的包，每个Go 源文件的开头都是一个 package 声明`package name`。

Go语言的包与文件夹是一一对应的，它具有以下几点特性：

- 一个目录下的同级文件属于同一个包。
- 包名可以与其目录名不同。
- main 包是Go语言程序的入口包，一个Go语言程序必须**有且仅有一个** main 包。如果一个程序没有 main 包，那么编译时将会出错，无法生成可执行文件。
- 文件名不一定是 main.go。例如上面hello_world.go。

## Go语言程序的编译和运行

可以通过Go语言提供的`go build`或者`go run`命令对Go语言程序进行编译：

- `go build`命令可以将Go语言程序代码编译成二进制的可执行文件，但是需要我们手动运行该二进制文件；
- `go run`命令则更加方便，它会在编译后直接运行Go语言程序，编译过程中会产生一个临时文件，但不会生成可执行文件，这个特点很适合用来调试程序。

`go build`命令用来启动编译，它可以将Go语言程序与相关依赖编译成一个可执行文件。

以上面`hello_world.go`为例：

```shell
PS D:\workplace\go\src\github.com\zgkaii\go-learning\day01\hello> go build hello_world.go
PS D:\workplace\go\src\github.com\zgkaii\go-learning\day01\hello> ls
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         2021/3/7     14:16        2098176 hello_world.exe
-a----         2021/3/6     16:36            283 hello_world.go
PS D:\workplace\go\src\github.com\zgkaii\go-learning\day01\hello> ./hello_world
Hello, World!
```

`go run`命令将编译和执行指令合二为一，会在编译之后立即执行Go语言程序，但是不会生成可执行文件。

打开terminal，输入`go run hello_world.go`，运行结果：

```html
Hello, World!
```

需要注意的是:

* Go语言中的main函数不支持任何返回值，而是通过`os.Exit()`来返回状态。

* main函数不⽀持传⼊参数，func main(~~arg []string~~)，在程序中直接通过`os.Args`获取命令行参数。还是上例：

```go
// hello_world.go
package main //包，表明代码所在的模块（包）

import (
  "fmt" // 引⼊多个依赖包
  "os"
)

// main函数，或主函数，是一个程序的入口函数。*
func main() {
  if len(os.Args) > 1 {
      fmt.Println("Hello, World!", os.Args[1])
  }
  os.Exit(-1)
}
```

打开terminal，输入`go run hello_world.go test`，运行结果：

```html
Hello, World! test
exit status 4294967295
```

# Go基础语法

## 关键字

25个关键字：

```go
break     default      func    interface  select
case      defer        go      map        struct
chan      else         goto    package    switch
const     fallthrough  if      range      type
continue  for          import  return     var
```

大致可分为四组：

- `const`、`func`、`import`、`package`、`type`和`var`用来声明各种代码元素。
- `chan`、`interface`、`map`和`struct`用做 一些组合类型的字面表示中。
- `break`、`case`、`continue`、`default`、 `else`、`fallthrough`、`for`、 `goto`、`if`、`range`、 `return`、`select`和`switch`用在流程控制语句中。
- `defer`和`go`也可以看作是流程控制关键字， 但它们有一些特殊的作用。

> 关键字不能作为标识符。
>
> 标识符`_`是一个特殊字符，它叫做**空标识符**。

## 基本类型

Go支持如下内置基本类型：

- 一种内置布尔类型：`bool`。
  * 只有两个相同类型的值才可以进行比较。布尔值可以和 &&（AND）和 ||（OR）操作符结合，并且有短路行为。
- 11种内置整数类型：`int8`、`uint8`、`int16`、`uint16`、`int32`、`uint32`、`int64`、`uint64`、`int`、`uint`和`uintptr`。
- 两种内置浮点数类型：`float32`和`float64`。
  * 常量`math.MaxFloat32`表示 float32 能取到的最大数值，大约是 3.4e38；
  * 常量`math.MaxFloat64`表示 float64 能取到的最大数值，大约是 1.8e308；
- 两种内置复数类型：`complex64`（32 位实数和虚数）和`complex128`（64 位实数和虚数，复数的默认类型）。
- 一种内置字符串类型：`string`。

>  `byte`是`uint8`的内置别名。 我们可以将`byte`和`uint8`看作是同一个类型。
>
> `rune`是`int32`的内置别名。 我们可以将`rune`和`int32`看作是同一个类型。

### 整型

Go语言同时提供了有符号和无符号的整数类型，其中包括 int8、int16、int32 和 int64 四种大小截然不同的有符号整数类型，分别对应 8、16、32、64 bit（二进制位）大小的有符号整数，与此对应的是 uint8、uint16、uint32 和 uint64 四种无符号整数类型。

此外还有两种整数类型 int 和 uint，它们分别对应特定 CPU 平台的字长（机器字大小），其中 int 表示有符号整数，应用最为广泛，uint 表示无符号整数。实际开发中由于编译器和计算机硬件的不同，int 和 uint 所能表示的整数大小会在 32bit 或 64bit 之间变化。

还有一种无符号的整数类型 uintptr，它没有指定具体的 bit 大小但是足以容纳指针。uintptr 类型只有在底层编程时才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。

### 字符串

Go语言的字符有以下两种：

- 一种是 uint8 类型，或者叫 byte 型，代表了 ASCII 码的一个字符。
- 另一种是 rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。rune 类型等价于 int32 类型。

**常用的转义字符**：

- `\n`：换行符；`\r`：回车符；`\t`：tab 键；`\u 或 \U`：Unicode 字符；`\\`：反斜杠自身

**计算字符串长度**：

- ASCII 字符串长度使用 len() 函数。
- Unicode 字符串长度使用 utf8.RuneCountInString() 函数。

**遍历字符串元素**：

- ASCII 字符串遍历直接使用下标。
- Unicode 字符串遍历用 for range。

```go
func TestString1(t *testing.T) {
	theme := "我 me"
	// 按ASCII字符遍历字符串
	for i := 0; i < len(theme); i++ {
		fmt.Printf("ascii: %c  %d\n", theme[i], theme[i])
	}

	// 按Unicode字符遍历字符串
	for _, s := range theme {
		fmt.Printf("Unicode: %c  %d\n", s, s)
	}
}
```

**截取字串**：

- `strings.Index`：正向搜索子字符串。
- `strings.LastIndex`：反向搜索子字符串。
- 搜索的起始位置可以通过切片偏移制作。

```go
	tracer := "我是谁, 我来自哪里 please"
	// 获取中文的逗号位置
	comma := strings.Index(tracer, ", ")
	// tracer[comma:]从tracer的 comma位置开始到 tracer字符串的结尾构造一个子字符串
	// 在这新的字符串中查找“哪里”的位置
	pos := strings.Index(tracer[comma:], "哪里")
	
	fmt.Println(comma, pos, tracer[comma+pos:]) // 9 11 哪里 please
```

**修改字符串**:

- Go 语言的字符串是不可变的。
- 修改字符串时，可以将字符串转换为 []byte 进行修改。
- []byte 和 string 可以通过强制类型转换互转。

```go
	str := "Hello my friend!"
	strBytes := []byte(str)

	for i := 5; i < 8; i++ {
		strBytes[i] = ' '
	}

	fmt.Println(string(strBytes)) // Hello    friend!
```

**字符串拼接**:

`bytes.Buffer` 是可以缓冲并可以往里面写入各种字节数组的。字符串也是一种字节数组，使用 `WriteString() `方法进行写入。

将需要连接的字符串，通过调用 `WriteString() `方法，写入 `stringBuilder` 中，然后再通过 `stringBuilder.String()` 方法将缓冲转换为字符串。

```go
	first := "你好啊"
	second := "朋友"

	// 声明字节缓冲
	var stringBuilder bytes.Buffer

	// 把字符串写入缓冲
	stringBuilder.WriteString(first)
	stringBuilder.WriteString(second)

	// 将缓冲以字符串形式输出
	fmt.Println(stringBuilder.String()) // 你好啊朋友
```

**fmt.Sprintf（格式化输出）**:

| 动  词 | 功  能                                                       |
| ------ | ------------------------------------------------------------ |
| `%v`   | 按值的本来值输出                                             |
| `%+v`  | 在 %v 基础上，对结构体字段名和值进行展开                     |
| `%#v`  | 输出 Go 语言语法格式的值                                     |
| `%T`   | 将替换为对应实参的类型的字符串表示形式。                     |
| `%%`   | 输出 % 本体                                                  |
| `%b`   | 整型以二进制方式显示                                         |
| `%o`   | 整型以八进制方式显示                                         |
| `%d`   | 整型以十进制方式显示                                         |
| `%x`   | 将替换为对应实参的十六进制表示。实参的类型可以为字符串、整数、整数数组（array）或者整数切片（slice）等。 |
| `%X`   | 整型以十六进制、字母大写方式显示                             |
| `%U`   | Unicode 字符                                                 |
| `%f`   | 浮点数                                                       |
| `%p`   | 指针，十六进制方式显示                                       |
| `%s`   | 将被替换为对应实参的字符串表示形式。实参的类型必须为字符串或者字节切片（byte slice）类型。 |
| `%c`   | 格式化说明符`%c`用于表示字符                                 |

### 数据类型转换

Go语言不存在隐式类型转换，因此所有的类型转换都必须显式的声明：`类型 B 的值 = 类型 B(类型 A 的值)`,例如`b := int(a)`。

只有相同底层类型的变量之间可以进行相互转换（如将 int16 类型转换成 int32 类型）。

## 字符串和数值类型的相互转换

strconv 包为我们提供了字符串和基本数据类型之间的转换功能。strconv 包中常用的函数包括 Atoi()、Itia()、parse 系列函数、format 系列函数、append 系列函数等。

### string 与 int 类型之间的转换

* `Itoa()`：整型转字符串，函数签名`func Itoa(i int) string`

* `Atoi()`：字符串转整型，函数签名 `func Atoi(s string) (i int, err error)`、

```go
	num := 100
	str := "200"
	str1 := strconv.Itoa(num)
	num1, err := strconv.Atoi(str)
	
	fmt.Printf("type: %T value: %#v\n", str1, str1) // type: string value: "100"

	if err != nil {
		fmt.Printf("%v 转换失败！", str1)

	} else {
		fmt.Printf("type: %T value: %#v\n", num1, num1) // type: int value: 200
	}
```

### Parse 系列函数

Parse 系列函数用于将字符串转换为指定类型的值，其中包括` ParseBool()、ParseFloat()、ParseInt()、ParseUint()`。

* `ParseBool()` 函数用于将字符串转换为 bool 类型的值，只能接受 `1、0、t、f、T、F、true、false、True、False、TRUE、FALSE`，其它的值均返回错误，函数签名`func ParseBool(str string) (value bool, err error)`。

* `ParseInt() `函数用于返回字符串表示的整数值（可以包含正负号），函数签名`func ParseInt(s string, base int, bitSize int) (i int64, err error)`。

  * base 指定进制，取值范围是 2 到 36。如果 base 为 0，则会从字符串前置判断，“0x”是 16 进制，“0”是 8 进制，否则是 10 进制。
  * bitSize 指定结果必须能无溢出赋值的整数类型，`0、8、16、32、64 `分别代表 `int、int8、int16、int32、int64`。
  * 返回的 err 是` *NumErr `类型的，如果语法有误，`err.Error = ErrSyntax`，如果结果超出类型范围 `err.Error = ErrRange`。

* `ParseUint() `函数的功能类似于 `ParseInt() `函数，但 `ParseUint() `函数**不接受正负号**，用于无符号整型，函数签名

  `func ParseUint(s string, base int, bitSize int) (n uint64, err error)`。

* `ParseFloat() `将一个表示浮点数的字符串转换为 float 类型，函数签名`func ParseFloat(s string, bitSize int) (f float64, err error)`。

  * bitSize 指定了返回值的类型，32 表示 float32，64 表示 float64。

```go
	str1, str2, str3, str4 := "111", "-10", "10", "3.1415926"

	bool, err1 := strconv.ParseBool(str1)
	num1, err2 := strconv.ParseInt(str2, 10, 0)
	num2, err3 := strconv.ParseUint(str3, 10, 0)
	num3, err4 := strconv.ParseFloat(str4, 64)

	if err1 != nil {
		fmt.Printf("str1: %v\n", err1)
	} else {
		fmt.Println(bool)
	}

	if err2 != nil {
		fmt.Printf("str2: %v\n", err2)
	} else {
		fmt.Println(num1)
	}

	if err3 != nil {
		fmt.Printf("str3: %v\n", err3)
	} else {
		fmt.Println(num2)
	}

	if err4 != nil {
		fmt.Printf("str4: %v\n", err4)
	} else {
		fmt.Println(num3)				
	}
    // str1: strconv.ParseBool: parsing "111": invalid syntax
	// -10
	// 10
	// 3.1415926
```

### Format 系列函数

Format 系列函数实现了将给定类型数据格式化为字符串类型的功能，其中包括`FormatBool()、FormatInt()、FormatUint()、FormatFloat()`。

* `FormatBool()`可将一个 bool 类型的值转换为对应的字符串类型，函数签名`func FormatBool(b bool) string`。
* `FormatInt()`将整型数据转换成指定进制并以字符串的形式返回，函数签`func FormatInt(i int64, base int) string`。
* `FormatUint()`函数与 FormatInt() 函数的功能类似，但是参数 i 必须是无符号的 uint64 类型，函数签名`func FormatUint(i uint64, base int) string`。
* `FormatFloat()`函数用于将浮点数转换为字符串类型，函数签名`func FormatFloat(f float64, fmt byte, prec, bitSize int) string`。
  * `fmt` 表示格式，可以设置为“f”表示`-ddd.dddd`、“b”表示 `-ddddp±ddd`，指数为二进制、“e”表示` -d.dddde±dd `十进制指数、“E”表示` -d.ddddE±dd` 十进制指数、“g”表示指数很大时用“e”格式，否则“f”格式、“G”表示指数很大时用“E”格式，否则“f”格式。

```go
	var num1 bool = true
	var num2 int64 = 100
	var num3 uint64 = 110
	var num4 float64 = 3.1415926

	str1 := strconv.FormatBool(num1)
	str2 := strconv.FormatInt(num2, 16)
	str3 := strconv.FormatUint(num3, 16)
	str4 := strconv.FormatFloat(num4, 'E', -1, 64)

	fmt.Printf("type:%T,value:%v\n ", str1, str1) // type:string,value:true
	fmt.Printf("type:%T,value:%v\n ", str2, str2) // type:string,value:64
	fmt.Printf("type:%T,value:%v\n ", str3, str3) // type:string,value:6e
	fmt.Printf("type:%T,value:%v\n ", str4, str4) // type:string,value:3.1415926E+00
```

### Append 系列函数

Append 系列函数用于将指定类型转换成字符串后追加到一个切片中，其中包含`AppendBool()、AppendFloat()、AppendInt()、AppendUint()`。Append 系列函数和 Format 系列函数的使用方法类似，只不过是将转换后的结果追加到一个切片中。

```go
	// 声明一个slice
	b10 := []byte("int (base 10):")

	// 将转换为10进制的string，追加到slice中
	b10 = strconv.AppendInt(b10, -42, 10)
	fmt.Println(string(b10))				// int (base 10):-42
	b16 := []byte("int (base 16):")
	b16 = strconv.AppendInt(b16, -42, 16)	
	fmt.Println(string(b16))				// int (base 16):-2a
```

## 变量

### 变量的声明

使用var关键字。标准模式**`var 变量名 变量类型`**，变量声明以关键字 var 开头，后置变量类型，行尾无须分号。例如`var a int`。

**批量声明变量**：

```go
var (
    a int
    b string
    c []float32
    d func() bool
    e struct {
        x int
    }
)
```

**简短格式**：`名字 := 表达式`。例如`i, j := 0, 1`。

需要注意的是，简短模式（short variable declaration）有以下限制：

* 定义变量，同时显式初始化。
* 不能提供数据类型。
* 只能用在函数内部。

### 变量的初始化

每个变量会初始化其类型的默认值，例如：

- 整型和浮点型变量的默认值为 0 和 0.0。
- 字符串变量的默认值为空字符串。
- 布尔型变量默认为 bool。
- 切片、函数、指针变量的默认为 nil。

**变量初始化的标准格式**：`var 变量名 类型 = 表达式`，例如：`var a int = 10`。

**编译器推导类型的格式**：编译器会尝试根据等号右值推导变量的类型，例如：`var a = 10`。

**短变量声明并初始化**：Go语言的**推导声明写法**，编译器会自动根据右值类型推断出左值的对应类型，例如：`a := 10`。

> 推导声明写法的左值变量必须是没有定义过的变量，定义过会报编译错误。

### 匿名变量

匿名变量的特点是一个下画线“`_`”，“`_`”本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个标识符作为变量对其它变量进行赋值或运算。

```go
func GetData() (int, int) {
    return 100, 200
}
func main(){
    a, _ := GetData()
    _, b := GetData()
    fmt.Println(a, b) // 100 200
}
```

> 匿名变量不占用内存空间，不会分配内存。匿名变量与匿名变量之间也不会因为多次声明而无法使用。

### 类型别名

类型别名是 Go 1.9 版本添加的新功能，主要用于解决代码升级、迁移中存在的类型兼容性问题。定义类型别名的写法为：

```go
type Type别名 = Type
```

类型别名与类型定义差别：

```go
// 将NewInt定义为int类型，这是Go1.9 版本之前定义
type NewInt int

// 将int取一个别名叫IntAlias
type IntAlias = int

func TestType(t *testing.T) {
	// 将a声明为NewInt类型
	var a NewInt
	// 查看a的类型名
	fmt.Printf("a type: %T\n", a) // a type: main.NewInt
	// 将a2声明为IntAlias类型
	var a2 IntAlias
	// 查看a2的类型名
	fmt.Printf("a2 type: %T\n", a2) // a2 type: int
}
```

## 常量

Go语言中的常量使用关键字 const 定义，用于存储不会改变的数据，常量是在编译时被创建的，即使定义在函数内部也是如此，并且只能是布尔型、数字型（整数型、浮点型和复数）和字符串型。

常量的定义格式和变量的声明语法类似：`const name [type] = value`（type可省略），例如：

```go
const pi = 3.14159 // 相当于 math.Pi 的近似值
```

批量声明多个常量：

```
const (
    e  = 2.7182818
    pi = 3.1415926
)
```

**iota 常量生成器**

常量声明可以使用 iota 常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。在一个 const 声明语句中，在第一个声明的常量所在的行，iota 将会被置为 0，然后在每一个有常量声明的行加一。

```go
type Weekday int

const (
	Sunday Weekday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)

func TestConst(t *testing.T) {
	fmt.Printf("%v %v %v", Sunday, Monday, Wednesday) // 0 1 3
}
```

## 指针

指针（pointer）在Go语言中可以被拆分为两个核心概念：

- **类型指针**，允许对这个指针类型的数据进行修改，传递数据可以直接使用指针，而无须拷贝数据，类型指针不能进行偏移和运算。
- **切片**，由指向起始元素的原始指针、元素数量和容量组成。

一个指针变量可以指向任何一个值的内存地址，它所指向的值的内存地址在 32 和 64 位机器上分别占用 4 或 8 个字节，占用字节的大小与所指向的值的大小无关。当一个指针被定义后没有分配到任何变量时，它的默认值为 nil。指针变量通常缩写为 ptr。

每个变量在运行时都拥有一个地址，这个地址代表变量在内存中的位置。Go语言中使用在变量名前面添加`&`操作符（前缀）来获取变量的内存地址（取地址操作），格式如下：

```go
ptr := &v   // v 的类型为 T
```

取地址操作符`&`和取值操作符`*`是一对互补操作符，`&`取出地址，`*`根据地址取出地址指向的值。

```go
	var b int = 10
	var s string = "hello"
	ptr1, ptr2 := &b, &s
	v1, v2 := *ptr1, *ptr2

	fmt.Printf("%p %p\n", ptr1, ptr2) // 0xc000014330 0xc000044530
	fmt.Printf("%T %T %v %v\n", v1, v2, v1, v2) // int string 10 hello
```

**new()函数创建指针**：

new() 函数可以创建一个对应类型的指针，创建过程会分配内存，被创建的指针指向默认值。

```go
str := new(string)
*str = "hello"

fmt.Println(*str) //hello
```

## 运算符优先级

| 优先级 | 分类           | 运算符                                         | 结合性       |
| ------ | -------------- | ---------------------------------------------- | ------------ |
| 1      | 逗号运算符     | ,                                              | 从左到右     |
| 2      | 赋值运算符     | =、+=、-=、*=、/=、 %=、 >=、 <<=、&=、^=、\|= | **从右到左** |
| 3      | 逻辑或         | \|\|                                           | 从左到右     |
| 4      | 逻辑与         | &&                                             | 从左到右     |
| 5      | 按位或         | \|                                             | 从左到右     |
| 6      | 按位异或       | ^                                              | 从左到右     |
| 7      | 按位与         | &                                              | 从左到右     |
| 8      | 相等/不等      | ==、!=                                         | 从左到右     |
| 9      | 关系运算符     | <、<=、>、>=                                   | 从左到右     |
| 10     | 位移运算符     | <<、>>                                         | 从左到右     |
| 11     | 加法/减法      | +、-                                           | 从左到右     |
| 12     | 乘法/除法/取余 | *（乘号）、/、%                                | 从左到右     |
| 13     | 单目运算符     | !、*（指针）、& 、++、--、+（正号）、-（负号） | 从右到左     |
| 14     | 后缀运算符     | ( )、[ ]、->                                   | 从左到右     |

> 注意：优先级值越大，表示优先级越高。

## 参考

* http://c.biancheng.net/golang/

* [Go 101](https://go101.org/article/101.html) 
* [Go by Example](https://gobyexample.com/) 

