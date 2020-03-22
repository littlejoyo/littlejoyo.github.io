---
title: Go:内置fmt包的应用
date: 2020-03-21 21:20:00
categories:
  - [Go]
tag:
    - Go
    - fmt
---

- fmt标准库是我们在学习Go语言过程中接触最早最频繁的一个

- fmt包实现了类似C语言printf和scanf的格式化I/O。

- 主要分为向外输出内容和获取输入内容两大部分。一起来看看。

<!-- more -->

# 1.向外输出

## 1.1 Print

- Print系列函数会将内容输出到系统的标准输出，区别:

- `Print`函数直接输出内容
- `Printf`函数支持格式化输出字符串
- `Println`函数会在输出内容的结尾添加一个换行符

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

示例代码：

```go
func main() {
	fmt.Print("打印输出：（不换行）。")
	name := "Joyo"
	fmt.Printf("我是：%s\n", name)
	fmt.Println("在终端打印单独一行显示")
}
```

输出结果：

```go
打印输出：（不换行）。我是：Joyo
在终端打印单独一行显示
```

## 1.2 Fprint

- `Fprint`系列函数会将内容输出到一个`io.Writer`接口类型的变量w中

- 我们通常用这个函数往文件中写入内容。

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

示例代码：

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
	fmt.Println("打开文件出错，err:", err)
	return
}
name := "Joyo"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

> 注意，只要满足io.Writer接口的类型都支持写入。

## 1.3 Sprint

- `Sprint`系列函数会把传入的数据生成并返回一个字符串。

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

示例代码：

```go
s1 := fmt.Sprint("Joyo")
name := "Joyo"
age := 18
s2 := fmt.Sprintf("name:%s,age:%d", name, age)
s3 := fmt.Sprintln("Joyo")
fmt.Println(s1, s2, s3)
```

输出结果：

```go
Joyo name:Joyo,age:18 Joyo
```

## 1.4 Errorf

- `Errorf`函数根据format参数生成格式化字符串并返回一个包含该字符串的错误。

```go
func Errorf(format string, a ...interface{}) error
```

- 通常使用这种方式来自定义错误类型，例如：

```go
err := fmt.Errorf("这是一个错误")
```

- Go1.13版本为`fmt.Errorf`函数新加了一个`%w`占位符用来生成一个可以包裹Error的Wrapping Error

```go
e := errors.New("原始错误e")
w := fmt.Errorf("Wrap了一个错误%w", e)
```

# 2.格式化占位符

- `*printf`系列函数都支持format格式化参数

## 2.1 通用占位符

|占位符|说明|
|:---:|:---:|
|%v|值的默认格式表示|
|%+v|类似%v，但输出结构体时会添加字段名|
|%#v|值的Go语法表示|
|%T|打印值的类型|
|%%|打印出百分号|

示例代码：

```go
fmt.Printf("%v\n", 100)
fmt.Printf("%v\n", false)
o := struct{ name string }{"Joyo"}
fmt.Printf("%v\n", o)
fmt.Printf("%+v\n", o)
fmt.Printf("%#v\n", o)
fmt.Printf("%T\n", o)
fmt.Printf("99%%\n")
```

输出结果：

```go
100
false
{小王子}
{name:小王子}
struct { name string }{name:"小王子"}
struct { name string }
99%
```

## 2.2 布尔型

|占位符|说明|
|:---:|:---:|
|%t|输出对应的布尔值，true或false|

## 2.3 整型

|占位符|说明|
|:---:|:---:|
|%b|	表示为二进制|
|%c|	该值对应的unicode码值|
|%d|	表示为十进制|
|%o|	表示为八进制|
|%x|	表示为十六进制，使用a-f|
|%X|	表示为十六进制，使用A-F|
|%U|	表示为Unicode格式：U+1234，等价于”U+%04X”|
|%q|	该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示|

示例代码：

```go
n := 65
fmt.Printf("65的二进制：%b\n", n)
fmt.Printf("65的Unicode码：%c\n", n)
fmt.Printf("65的十进制：%d\n", n)
fmt.Printf("65的八进制：%o\n", n)
m := 31
fmt.Printf("31的十六进制：%x\n", m)
fmt.Printf("31的十六进制：%X\n", m)
fmt.Printf("31的字符对应：%q\n", m)
```

输出代码：

```go
65的二进制：1000001
65的Unicode码：A
65的十进制：65
65的八进制：101
31的十六进制：1f
31的十六进制：1F
31的字符对应：'\x1f'
```

## 2.4 浮点数和复数

|占位符|说明|
|:---:|:---:|
|%b|	无小数部分、二进制指数的科学计数法，如-123456p-78|
|%e|	科学计数法，如-1234.456e+78|
|%E|	科学计数法，如-1234.456E+78|
|%f|	有小数部分但无指数部分，如123.456|
|%F|	等价于%f|
|%g|	根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）|
|%G|	根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）|

示例代码：

```go
f := 13.14
fmt.Printf("%b\n", f)
fmt.Printf("%e\n", f)
fmt.Printf("%E\n", f)
fmt.Printf("%f\n", f)
fmt.Printf("%g\n", f)
fmt.Printf("%G\n", f)
```

输出结果：

```go
7397162387956040p-49
1.314000e+01
1.314000E+01
13.140000
13.14
13.14
```

## 2.5 字符串和[]byte

|占位符|说明|
|:---:|:---:|
|%s|	直接输出字符串或者[]byte|
|%q|	该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示|
|%x|	每个字节用两字符十六进制数表示（使用a-f)|
|%X|	每个字节用两字符十六进制数表示（使用A-F）|

示例代码：

```go
s1 := "Joyo"
fmt.Printf("%s\n", s1)
fmt.Printf("%q\n", s1)
fmt.Printf("%x\n", s1)
fmt.Printf("%X\n", s1)
s2:= "小A18岁"
fmt.Printf("%s\n", s2)
fmt.Printf("%q\n", s2)
fmt.Printf("%x\n", s2)
fmt.Printf("%X\n", s2)
```

输出结果：

```go
Joyo
"Joyo"
4a6f796f
4A6F796F
小A18岁
"小A18岁"
e5b08f413138e5b281
E5B08F413138E5B281
```

## 2.6 指针

|占位符|说明|
|:---:|:---:|
|%p|	表示为十六进制，并加上前缀0x|
|%#p|	表示为十六进制,不加上前缀0x|

示例代码：

```go
a := 10
fmt.Printf("%p\n", &a)
fmt.Printf("%#p\n", &a)
```

输出代码：

```go
0xc000094000
c000094000
```

## 2.7 宽度标识符

- 宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。

- 精度通过（可选的）宽度后跟点号后跟的十进制数指定。

- 如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。

|占位符|说明|
|:---:|:---:|
|%f|	默认宽度，默认精度|
|%9f|	宽度9，默认精度|
|%.2f|	默认宽度，精度2|
|%9.2f|	宽度9，精度2|
|%9.f|	宽度9，精度0|

注意：精度表示小数点后几位

示例代码：

```go
n := 13.14
fmt.Printf("%f\n", n)
fmt.Printf("%9f\n", n)
fmt.Printf("%.2f\n", n)
fmt.Printf("%9.2f\n", n)
fmt.Printf("%9.f\n", n)
```

输出结果：

```go
13.140000
13.140000
13.14
    13.14
       13
```

## 2.8 其他falg

|占位符|说明|
|:---:|:---:|
|’+’|	总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）；|
|’ ‘|	对数值，正数前加空格而负数前加负号；对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格|
|’-’|	在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）；|
|’#’|	八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值；|
|‘0’|	使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面；|

示例代码：

```go
s := "一二三"
	fmt.Printf("%s\n", s)
	fmt.Printf("%5s\n", s)
	fmt.Printf("%-5s\n", s)
	fmt.Printf("%5.7s\n", s)
	fmt.Printf("%-5.7s\n", s)
	fmt.Printf("%5.2s\n", s)
	fmt.Printf("%05s\n", s)
	fmt.Println("============>")
	a:= "Joyo"
	fmt.Printf("%s\n", a)
	fmt.Printf("%5s\n", a)
	fmt.Printf("%-5s\n", a)
	fmt.Printf("%5.7s\n", a)
	fmt.Printf("%-5.7s\n", a)
	fmt.Printf("%5.2s\n", a)
	fmt.Printf("%05s\n", a)
```

输出代码:

```go
一二三
  一二三
一二三  
  一二三
一二三  
   一二
00一二三
============>
Joyo
 Joyo
Joyo 
 Joyo
Joyo 
   Jo
0Joyo
```

# 3.获取输入

- Go语言fmt包下有`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数

- 可以在程序运行过程中从标准输入获取用户的输入

## 3.1 fmt.Scan

```go
func Scan(a ...interface{}) (n int, err error)
```

- `Scan`从标准输入扫描文本，读取由`空白符`分隔的值保存到传递给本函数的参数中，换行符视为空白符。

- 本函数返回成功扫描的数据个数和遇到的任何错误。

- 如果读取的数据个数比提供的参数少，会返回一个错误报告原因。

示例代码：

```go
func main() {
	var (
		name    string
		age     int
		isMan   bool
	)
	fmt.Scan(&name, &age, &isMan)
	fmt.Printf("扫描结果 name:%s age:%d isMan:%t \n", name, age, isMan)
}
```

输入参数：

```go
joyo
18
true
```

输出结果：

```go
扫描结果 name:Joyo age:18 isMan:true 
```

## 3.2 fmt.Scanf

```go
func Scanf(format string, a ...interface{}) (n int, err error)
```

- `Scanf`从标准输入扫描文本，根据format参数指定的格式去读取由空白符分隔的值保存到传递给本函数的参数中。

- 本函数返回成功扫描的数据个数和遇到的任何错误。

示例代码：

```go
var (
		name    string
		age     int
		isMan bool
	)
	fmt.Scanf("1:%s 2:%d 3:%t", &name, &age, &isMan)
	fmt.Printf("扫描结果 name:%s age:%d isMan:%t \n", name, age, isMan)
```

输入参数：

```go
1:Joyo 2:18 3:true
```

输出结果：

```go
扫描结果 name:Joyo age:18 isMan:true
```

- `fmt.Scanf`不同于fmt.Scan简单的以空格作为输入数据的分隔符

- `fmt.Scanf`为输入数据指定了具体的输入内容格式，只有按照格式输入数据才会被扫描并存入对应变量。（比如上面规定了`1:%s 2:%d 3:%t`的输入格式）

- 如果不按照规定的格式中以空格分隔的方式输入，`fmt.Scanf`就不能正确扫描到输入的数据

输出参数：

```go
Joyo 18 true
```

输出结果：

```go
扫描结果 name: age:0 isMan:false 
```

> 获取不到扫描的内容，最后输出了默认值。

## 3.3 fmt.Scanln

```go
func Scanln(a ...interface{}) (n int, err error)
```

- `Scanln`类似`Scan`，**它在遇到`换行`时才停止扫描**。

- 最后一个数据后面必须有换行或者到达结束位置。

- 本函数返回成功扫描的数据个数和遇到的任何错误。

示例代码：

```go
var (
		name    string
		age     int
		isMan   bool
	)
	fmt.Scanln(&name, &age, &isMan)
	fmt.Printf("扫描结果 name:%s age:%d isMan:%t \n", name, age, isMan)
```

输入参数：

```go
Joyo 18 false
```

> `fmt.Scanln`遇到回车就结束扫描。

输出结果：

```go
扫描结果 name:Joyo age:18 isMan:false 
```

## 3.4 bufio.NewReader

- 有时候我们想完整获取输入的内容，而输入的内容可能包含空格，这种情况下可以使用`bufio`包来实现。

示例代码:

```go
func bufioDemo() {
	reader := bufio.NewReader(os.Stdin) // 从标准输入生成读对象
	fmt.Print("请输入内容：")
	text, _ := reader.ReadString('\n') // 读到换行
	text = strings.TrimSpace(text)
	fmt.Printf("%#v\n", text)
}
```

输入参数:

```go
请输入内容：  测试打印出好多输入内容  内容中包含了空格。
```

输出结果：

```go
"测试打印出好多输入内容  内容中包含了空格。"
```

## 3.5 Fscan系列

- `Fscan`系列的几个函数功能分别类似于`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数

- 只不过它们不是从标准输入中读取数据而是从`io.Reader`中读取数据

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

## 3.6 Sscan系列

- `Sscan`系列这几个函数功能分别类似于`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数

- 只不过它们不是从标准输入中读取数据而是**从指定字符串中读取数据**。

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

> 文章参考：https://www.liwenzhou.com/posts/Go/go_fmt/

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)

