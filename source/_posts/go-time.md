---
title: Go:内置time包的应用
date: 2020-03-21 17:41:33
categories:
  - [Go]
tag:
    - Go
    - time
---

- 时间和日期在业务编程中我们经常要用到，Go提供了内置的time包

- time包提供了时间的显示和测量用的函数。日历的计算采用的是公历。

- 本文主要介绍了Go语言内置的time包的基本用法。

<!-- more -->

# 1.时间类型

- `time.Time`表示时间类型

- `time.Now()`函数获取当前的时间对象，然后获取时间对象的年月日时分秒等信息

获取时间的代码：

```go
func main() {
	now := time.Now() //获取当前时间
	fmt.Printf("current time:%v\n", now)

	year := now.Year()     //年
	month := now.Month()   //月
	day := now.Day()       //日
	hour := now.Hour()     //小时
	minute := now.Minute() //分钟
	second := now.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
	fmt.Println("year:",year)
	fmt.Println("month:",month)
	fmt.Println("day:",day)
	fmt.Println("hour:",hour)
	fmt.Println("minute:",minute)
	fmt.Println("second:",second)
}
```

输出结果：

```go
current time:2020-03-21 17:52:01.603274 +0800 CST m=+0.001475054
2020-03-21 17:52:01
year: 2020
month: March
day: 21
hour: 17
minute: 52
second: 1
```

# 2.时间戳

- 时间戳是自1970年1月1日（08:00:00GMT）至当前时间的总毫秒数

- 它也被称为Unix时间戳（UnixTimestamp）

## 2.1 获取当前时间戳

```go
func main() {
	now := time.Now()            //获取当前时间
	timestamp1 := now.Unix()     //时间戳
	timestamp2 := now.UnixNano() //纳秒时间戳
	fmt.Printf("current timestamp1:%v\n", timestamp1)
	fmt.Printf("current timestamp2:%v\n", timestamp2)
}
```

输出结果：

```go
current timestamp1:1584784509
current timestamp2:1584784509755072000
```

## 2.2 时间戳转化为时间类型

- 使用time.Unix()函数可以将时间戳转为时间格式。

```go
func demo(timestamp int64) {
	timeObj := time.Unix(timestamp, 0) //将时间戳转为时间格式
	fmt.Println(timeObj)
	year := timeObj.Year()     //年
	month := timeObj.Month()   //月
	day := timeObj.Day()       //日
	hour := timeObj.Hour()     //小时
	minute := timeObj.Minute() //分钟
	second := timeObj.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
}
```

输出结果：

```go
2020-03-21 17:58:07 +0800 CST
2020-03-21 17:58:07
```

# 3.时间间隔

- `time.Duration`是`time`包定义的一个类型，它代表两个时间点之间经过的时间，以纳秒为单位。

- `time.Duration`表示一段时间间隔，可表示的最长时间段大约290年。

- time包中定义的时间间隔类型的常量如下：

```go
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

`time.Duration`表示1纳秒，`time.Second`表示1秒。

# 4.时间操作

## 4.1 Add 增加时间隔间

```go
func (t Time) Add(d Duration) Time
```

求一个小时之后的时间：

```go
func main() {
	now := time.Now()
	later := now.Add(time.Hour) // 当前时间加1小时后的时间
	fmt.Println(later)
}
```

## 4.2 Sub 求两个时间差值

```go
func (t Time) Sub(u Time) Duration
```

- 返回一个时间段t-u。

- 如果结果超出了Duration可以表示的最大值/最小值，将返回最大值/最小值。

- 要获取时间点t-d（d为Duration），可以使用t.Add(-d)。

## 4.3 Equal 判断两个时间是否相同


```go
func (t Time) Equal(u Time) bool
```

- 判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。

- 本方法和用t==u不同，这种方法还会比较地点和时区信息。

## 4.4 Before 

```go
func (t Time) Before(u Time) bool
```

- 如果t代表的时间点在u之前，返回真；否则返回假。

## 4.5 After

```go
func (t Time) After(u Time) bool
```

- 如果t代表的时间点在u之后，返回真；否则返回假。

## 5.定时器

- 使用`time.Tick`(时间间隔)来设置定时器，定时器的本质上是一个通道（channel）。

```go
func tickDemo() {
	ticker := time.Tick(time.Second) //定义一个1秒间隔的定时器
	for i := range ticker {
		fmt.Println(i)//每秒都会执行的任务
	}
}
```

# 6.时间格式化

- 时间类型有一个自带的方法Format进行格式化

- 需要注意的是Go语言中格式化时间模板不是常见的`Y-m-d H:M:S`而是使用Go的诞生时间`2006年1月2号15点04`分（记忆口诀为`2006 1 2 3 4`）。

> 补充：如果想格式化为12小时方式，需指定PM。

```go
func formatDemo() {
	now := time.Now()
	// 格式化的模板为Go的出生时间2006年1月2号15点04分 Mon Jan
	// 24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	// 12小时制
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan"))
	fmt.Println(now.Format("2006/01/02 15:04"))
	fmt.Println(now.Format("15:04 2006/01/02"))
	fmt.Println(now.Format("2006/01/02"))
}
```

输出结果：

```go
2020-03-21 18:20:25.933 Sat Mar
2020-03-21 06:20:25.933 PM Sat Mar
2020/03/21 18:20
18:20 2020/03/21
2020/03/21
```

# 7.解析时间字符串为时间类型

```go
now := time.Now()
	fmt.Println(now)
	// 加载时区
	loc, err := time.LoadLocation("Asia/Shanghai")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 按照指定时区和指定格式解析字符串时间
	timeObj, err := time.ParseInLocation("2006/01/02 15:04:05", "2020/03/22 14:15:20", loc)
	if err != nil {
		fmt.Println(err)
		return
	}
    fmt.Println(timeObj)
    // 打印出跟当前时间的时间间隔
	fmt.Println(timeObj.Sub(now))
```

输出结果：

```go
2020-03-21 18:24:15.428607 +0800 CST m=+0.002097406
2020-03-22 14:15:20 +0800 CST
19h51m4.571393s
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)