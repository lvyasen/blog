---
title: Go 基础类型
urlname: xxu76i3ev36f4bf5
date: '2022-12-19 14:36:24 +0800'
tags: []
categories: []
---

> 日期和时间都封装在 time 包中

```go
time.Now()//获取当前时间

time.May //month 类型是在 time 包中定义的常量直接使用即可
var date = time.Date(2021,11,25,0,0,0,0,time.Local)
//var date = time.Now()
fmt.Printf("年:%v\n",date.Year())//获取年
fmt.Printf("月:%v\n",date.Month())//获取月
fmt.Printf("日:%v\n",date.Day())//获取日
fmt.Printf("时:%v\n",date.Hour())//获取小时
fmt.Printf("分:%v\n",date.Minute())//获取分钟
fmt.Printf("秒:%v\n",date.Second())//获取秒
fmt.Printf("纳秒:%v\n",date.Nanosecond())//获取纳秒
fmt.Printf("周:%v\n",date.Weekday())//获取星期几
fmt.Printf("当地时间:%v\n",date.Local())//获取当地时间
fmt.Printf("获取 UTC 时间:%v\n",date.UTC())//获取 UTC 时间
fmt.Printf("格式化时间:%v\n",date.Format("2006-15-02 15:04:05"))//将日期转换为字符串
fmt.Printf("时间戳:%v\n",date.Unix())//获取当前时间戳
fmt.Printf("时间戳(纳秒):%v\n",date.UnixNano())//获取当前时间戳
fmt.Printf("三个小时后日期:%v\n",date.Add(time.Hour*3))//三个小时后
fmt.Printf("三个小时前日期:%v\n",date.Add(-time.Hour*3))//三个小时前
```

### Duration 类型

> Duration 是 int64 位类型,以纳秒为单位

```go
a := 25 * time.Second //25 秒
b := 25 * time.Minute //25 分钟
c := 25 * time.Hour //25 小时

//转换
c.Minutes()//小时转为分钟
b.Secinds()//分钟转为秒
```

### Time 类型

> Time 结构体表示时间实例,精度位纳秒使用下面两个函数来获取 Time 实例
>
> 1. time.Now() 获取当前系统时间,返回 Time 实例
> 2. time.Date(年,月,日,时,分,秒,纳秒,时区),返回 Time 实例

```go
var now = time.Now()
year,month,day := now.Date() //使用 Date 方法获取年月日部分的值
hour,,minute,second := now.ClocK() //使用 Clock 获取时分秒的值
```

### Sleep 函数

> 调用 Sleep 函数会是当前协程暂停执行,并等待一段时间

```go
time.Sleep(time.Second*3) //暂停三秒
```

### Timer 类型

> Timer 是一种特殊的计时器,当指定时间到期后,会将当前时间发送到 C 字段中
> C 字段是只读的通道类型 其他协程通过 C 字段接受 Timer 实例
> 典型用途在异步编程中处理超时行为

```go
	var timer = time.NewTimer(5*time.Second)
	var c = make(chan bool)
	defer close(c)

	go func() {
		time.Sleep(time.Second*10)
		c<-true
	}()

	select {
	case <-c:
		fmt.Println("任务完成")
	case <-timer.C:
		fmt.Println("任务超时")
	}
//select 语句中每个 case 子句被执行的概率相等
```

### 指针

> 指针是一种特殊类型
> 它的实例将保存被引用的向的内存地址,而不是对象本身的值
> 指针变量在声明后如果没有有效的内存地址,他么他的默认值就是 nil

```go
var name = "张三"
fmt.Println("获取内存地址:%p",&name)
```

#### 什么时候使用指针

1. 参数在处理的过程中需要修改
2. 定义方法的时候如果修改了结构体那么就要使用指针类型

#### new 函数

> new 函数为指定的类型分配内存空间,并使用类型的默认值进行初始化,最后返回新分配的内存空间
> func new(Type) \*Type
> 也就是说使用 new 函数返回的是指针

⚠️ 注意

1. new(string) 所分配的值不是 nil 而是空字符串
2. 结构体分配内存空间后,会自动为其字段分配默认值
3. 接口类型使用 new 函数分配内存后,其默认值为 nil

### Iota 常量

> iota 定义如下
> const iota = 0
> 这是特殊用途常量,在批量定义常量代码中, 如果常量 A 使用了 iota 常量作为基础整数值,那么 A 之后定义的常量会自动累加
