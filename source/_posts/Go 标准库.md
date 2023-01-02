---
title: Go 标准库
urlname: tndv2v9fgd2y5dsz
date: '2022-12-20 15:44:09 +0800'
tags: []
categories: []
---

tags: [标准库,log,json,io,time,os,sync,math,bytes,net/https,strings,strconv]
categories: [Go]
cover: ''

---

> 标准库里总共有超过 100 个包，这些包被分到 38 个类别里。
> 如果想了解所有包以及更详细的描述，移步[官方文档](http://golang.org/pkg/)、[中文文档](https://studygolang.com/pkgdoc)。

```go
archive  bufio   bytes   compress  container  crypto  database
debug   encoding  errors  expvar   flag    fmt    go
hash   html    image   index   io     log    math
mime   net    os    path    reflect   regexp  runtime
sort   strconv  strings  sync    syscall   testing  text
time   unicode  unsafe
```

## log（简单的日志服务）

> 日志是一种找到这些 bug，更好地了解程序工作状态的方法。

### 使用

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func init()  {
	log.SetPrefix("Trace: ")//设置前缀
	log.SetFlags(log.Ldate|log.Ltime|log.Llongfile)//设置logger的输出选项
	logFile,err:=os.OpenFile("err.log",os.O_CREATE|os.O_WRONLY|os.O_APPEND,0644)
	if err != nil {
		fmt.Println("打开文件异常")
		os.Exit(400)
	}
	log.SetOutput(logFile)
}
func main()  {
	log.Println("测试")//将生成的格式化字符串输出到logger，参数用和fmt.Println相同的方法处理。
	log.Panicln("记录日志并调用 panic")
	//log.Fatalln("记录日志并调用 os.Exit(1) ")
}

```

### 定制日志记录器

> 要想创建一个定制的日志记录器，需要创建一个 Logger 类型值。
> 可以给每个日志记录器配置一个单独的目的地，并独立设置其前缀和标志。

```go
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"log"
	"os"
)

var (
	Trace *log.Logger
	Info *log.Logger
	Warning *log.Logger
	Error *log.Logger
)

func init()  {
	file,err:=os.OpenFile("errors.log",os.O_CREATE|os.O_WRONLY|os.O_APPEND,0666)
	if err != nil {
		fmt.Println("打开文件失败")
		os.Exit(400)
	}

	Trace = log.New(ioutil.Discard, "Trace: ", log.Ldate|log.Ltime|log.Llongfile)
	Info = log.New(os.Stdout, "Info: ",log.Ldate|log.Ltime|log.Llongfile)
	Warning = log.New(os.Stdout,"Warning: ",log.Ldate|log.Ltime|log.Llongfile)
	Error = log.New(io.MultiWriter(file,os.Stderr),"Error: ",log.Ldate|log.Ltime|log.Llongfile)
}

func main()  {
	Trace.Println("Trace 错误")
	Info.Println("Info 错误")
	Warning.Println("Warning 错误")
	Error.Println("Error 错误")
}

```

## json（对 json 对象的编解码）

### 解码

> 使用 NewDecode 和 Decode 解码 JSON 到结构类型。
> 注意结构体中 **字段 **要使用大写字母，否则该字段为该类型零值，结构体名字可以小写。

#### 处理网络或文件中的 json

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"os"
)

type Resp struct {
	Data struct{
		Data []post `json:"data"`
	} `json:"data"`
	Code int `json:"code"`
	Msg string `json:"msg"`
}
type post struct {
	Id int `json:"id"`
	Name string `json:"name"`
	Avatar string `json:"avatar"`
}
func main()  {
	url := "https://wx.sacsboe.cn/api/getPostListByOpenId?openid=oRyu45XHK7ZSCQC-C7F3Ftu19AaQ&page=1"
	resp,err:=http.Get(url)
	if err != nil {
		fmt.Println("访问失败",err)
		os.Exit(400)
	}
	//defer resp.Body.Close()
	//var postList Resp
	//var postList interface{}//如果获取所有直接使用 interface{},任何类型都实现了 interface{} 接口
	var postList map[string]interface{}
	//NewDecoder 解码器会自己进行缓冲,而且可能读取比 Json 更多的数据
	//Decode 从自己的输入中读取下一个编好的 Json 值并存入指向指,这里要修改数据所以使用 & 取址
	//这里不用存入变量
	err1 := json.NewDecoder(resp.Body).Decode(&postList)
	if err1 != nil {
		fmt.Println("解码失败",err1)
		os.Exit(401)
	}
	fmt.Println(postList["msg"])
}

```

#### 处理字符串的 json

> 需要将 `string`转为 []byte 切片，并使用 json 包中的 `Unmarshal` 函数进行处理

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type UserInfo struct {
	Name string `json:"name"`
	Age int `json:"age"`
	Contact contact `json:"contact"`
}
type contact struct {
	Home string `json:"home"`
	Phone string `json:"phone"`
}

var mockData = `{
	"name":"walker",
	"age":23,
	"contact":{
		"home":"上海市浦东新区长泰广场 110 号",
		"Phone":"1853123234"
	}
}`

func main()  {
	//var u UserInfo
	var u map[string]interface{}//有时无法为 json 数据单独声明一个结构,可以使用 map 灵活的处理 json 文档
	err := json.Unmarshal([]byte(mockData),&u)
	if err != nil {
		fmt.Println("解析失败",err)
		os.Exit(400)
	}
	fmt.Println(u)
}

```

### 编码（序列化）

> 序列化（marshal）是指将数据转换为 JSON 字符串的过程。

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

func main()  {
	var userInfo = make(map[string]interface{})
	userInfo["name"] = "张三"
	userInfo["age"] = 30
	userInfo["mobile"] = "1853099999"
	userInfo["contact"] = map[string]interface{}{
		"home":"上海市浦东新区长泰广场 110 号",
	}
	//Marshal 返回一个 []byte 切片,用来保存 json 字符串和一个 error 值
	//data,err := json.Marshal(userInfo)
	//带缩进的序列化,下面是四个缩进
	data,err := json.MarshalIndent(userInfo,"","    ")
	if err != nil {
		fmt.Println("序列化失败",err)
		os.Exit(400)
	}
	fmt.Println(string(data))
}

```

## io（提供了对 I/O 原语的基本接口）

> 该包以流的方式高效处理数据，而不用考虑数据是什么，数据来自哪里，数组要发送到哪里。

### io.Reader

> io.Reader 表示一个读取器，它将数据从某个资源读取到传输缓冲区。在缓冲区中，数据可以被流式传输和使用。
> 对于要用作读取器的类型，它必须实现 io.Reader 接口的唯一一个方法 Read(p []byte)。
> 换句话说，只要实现了 Read(p []byte) ，那它就是一个读取器。

```go
//传入一个 []byte 类型切片，返回一个读到的字节数 int 和 err
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main()  {
	reader := strings.NewReader("my name is zs")
	p := make([]byte,2)
	for  {
		readLen,err:=reader.Read(p)
		if err != nil {
			fmt.Println("EOF:",readLen)
			os.Exit(1)
		}
		fmt.Println(readLen,string(p[:readLen]))
	}
}

```

## time

## strings（操作字符串）

## strconv（字符串转换其他类型）

## sort（切片排序和自定义数据集排序）

## path（斜杠分割路径的操作函数）

## os（操作系统函数的接口）

> os 包提供了操作系统函数的不依赖平台的接口。设计为 Unix 风格的，虽然错误处理是 go 风格的；失败的调用会返回错误值而非错误码

### 定义的常量

#### 文件相关

```go
//用于包装底层系统的参数用于Open函数，不是所有的flag都能在特定系统里使用的。
const (
    O_RDONLY int = syscall.O_RDONLY // 只读模式打开文件
    O_WRONLY int = syscall.O_WRONLY // 只写模式打开文件
    O_RDWR   int = syscall.O_RDWR   // 读写模式打开文件
    O_APPEND int = syscall.O_APPEND // 写操作时将数据附加到文件尾部
    O_CREATE int = syscall.O_CREAT  // 如果不存在将创建一个新文件
    O_EXCL   int = syscall.O_EXCL   // 和O_CREATE配合使用，文件必须不存在
    O_SYNC   int = syscall.O_SYNC   // 打开文件用于同步I/O
    O_TRUNC  int = syscall.O_TRUNC  // 如果可能，打开时清空文件
)


//指定Seek函数从何处开始搜索（即相对位置）
const (
    SEEK_SET int = 0 // 相对于文件起始位置seek
    SEEK_CUR int = 1 // 相对于文件当前位置seek
    SEEK_END int = 2 // 相对于文件结尾位置seek
)

//DevNull是操作系统空设备的名字。在类似Unix的操作系统中，是"/dev/null"；在Windows中，为"NUL"。
const (
    PathSeparator     = '/' // 操作系统指定的路径分隔符
    PathListSeparator = ':' // 操作系统指定的表分隔符
)
const DevNull = "/dev/null"
```

### 定义的变量

```go
//一些可移植的、共有的系统调用错误。
var (
    ErrInvalid    = errors.New("invalid argument")
    ErrPermission = errors.New("permission denied")
    ErrExist      = errors.New("file already exists")
    ErrNotExist   = errors.New("file does not exist")
)

//Stdin、Stdout和Stderr是指向标准输入、标准输出、标准错误输出的文件描述符。
var (
    Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
    Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
    Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)

//Args保管了命令行参数，第一个是程序名。
var Args []string
```

### 环境相关

#### os.[Hostname](https://github.com/golang/go/blob/master/src/os/doc.go?name=release#92)()：获取主机名

```go
func Hostname() (name string, err error)

//如： macdeMacBook-Pro.local
```

#### os.[Getpagesize](https://github.com/golang/go/blob/master/src/os/types.go?name=release#13)()：返回底层的系统内存页的尺寸

```go
func Getpagesize() int

//如: 4096
```

#### os.[Environ](https://github.com/golang/go/blob/master/src/os/env.go?name=release#101)()：返回表示环境变量的格式为"key=value"的字符串的切片拷贝

> Environ 返回表示环境变量的格式为"key=value"的字符串的切片拷贝。

```go
func Environ() []string

//返回的是字符串切片

//这个是遍历的详情
// PATH=/Users/mac/flutter/bin:/opt/anaconda3/bin:/usr/local/opt/openjdk/bin:/usr/local/opt/php@7.4/sbin:/usr/local/opt/php@7.4/bin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public:/usr/local/go/bin:/opt/X11/bin:/Library/Apple/usr/bin:/usr/local/go/bin:/Users/mac/go/bin
// TERM=xterm-256color
// PUB_HOSTED_URL=https://pub.flutter-io.cn
// COMMAND_MODE=unix2003
// ANDROID_HOME=/Users/mac/Library/Android/sdk
// DISPLAY=/private/tmp/com.apple.launchd.IUWWeeoc3q/org.xquartz:0
// LOGNAME=mac
// XPC_SERVICE_NAME=application.com.jetbrains.goland.25364946.25365700
// PWD=/Users/mac/go/src/go-test
// __CFBundleIdentifier=com.jetbrains.goland
// ANDROID_SDK_HOME=/Users/mac/Library/Android/sdk
// SHELL=/bin/zsh
// PAGER=less
// LSCOLORS=Gxfxcxdxbxegedabagacad
// OLDPWD=/
// GOPATH=/Users/mac/go
// USER=mac
// GOROOT=/usr/local/go
// ZSH=/Users/mac/.oh-my-zsh
// TMPDIR=/var/folders/bt/w1kbbf0s5lq749d93gsf7dcr0000gn/T/
// GO111MODULE=auto
// SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.kJIt9jwn2j/Listeners
// XPC_FLAGS=0x0
// __CF_USER_TEXT_ENCODING=0x1F5:0x19:0x34
// LESS=-R
// FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
// LC_CTYPE=zh_CN.UTF-8
// HOME=/Users/mac

```

#### os.[Getenv](https://github.com/golang/go/blob/master/src/os/env.go?name=release#79)()：检索并返回名为 key 的环境变量的值

> Getenv 检索并返回名为 key 的环境变量的值。如果不存在该环境变量会返回空字符串。

```go
func Getenv(key string) string

//如：
	pwd := os.Getenv("PWD")
	fmt.Println(pwd)
```

#### os.[Setenv](https://github.com/golang/go/blob/master/src/os/env.go?name=release#86)()：设置环境变量

> 设置名为 key 的环境变量。如果出错会返回该错误。

```go
func Setenv(key, value string) error
```

#### os.[Clearenv](https://github.com/golang/go/blob/master/src/os/env.go?name=release#95)()：删除所有环境变量

```go
func Clearenv()
```

#### os.[Exit](https://github.com/golang/go/blob/master/src/os/proc.go?name=release#36)()：退出

> Exit 让当前程序以给出的状态码 code 退出。一般来说，状态码 0 表示成功，非 0 表示出错。
> 程序会立刻终止，defer 的函数不会被执行。

```go
func Exit(code int)
```

### 进程相关

#### os.[Getuid](https://github.com/golang/go/blob/master/src/os/proc.go?name=release#15)()：返回调用者的用户 ID

> 返回调用者的用户 ID

```go
func Getuid() int

//Geteuid返回调用者的有效用户ID。
func Geteuid() int
```

#### os.[Getgid](https://github.com/golang/go/blob/master/src/os/proc.go?name=release#21)()：返回调用者的组 ID

> Getgid 返回调用者的组 ID。

```go
func Getgid() int

//Getegid返回调用者的有效组ID。
func Getegid() int
```

#### os.Getpid()：获取当前进程 id

> Getpid 返回调用者所在进程的进程 ID。

```go
func Getpid() int
```

#### os.Getppid()：获取当前进程父进程 id

> Getppid 返回调用者所在进程的父进程的进程 ID。

```go
func Getppid() int
```

### 文件相关

> `type FileMode uint32`
> FileMode 代表文件的模式和权限位。
> 这些字位在所有的操作系统都有相同的含义，因此文件的信息可以在不同的操作系统之间安全的移植。
> 不是所有的位都能用于所有的系统，唯一共有的是用于表示目录的 ModeDir 位。

```go
const (
    // 单字符是被String方法用于格式化的属性缩写。
    ModeDir        FileMode = 1 << (32 - 1 - iota) // d: 目录
    ModeAppend                                     // a: 只能写入，且只能写入到末尾
    ModeExclusive                                  // l: 用于执行
    ModeTemporary                                  // T: 临时文件（非备份文件）
    ModeSymlink                                    // L: 符号链接（不是快捷方式文件）
    ModeDevice                                     // D: 设备
    ModeNamedPipe                                  // p: 命名管道（FIFO）
    ModeSocket                                     // S: Unix域socket
    ModeSetuid                                     // u: 表示文件具有其创建者用户id权限
    ModeSetgid                                     // g: 表示文件具有其创建者组id的权限
    ModeCharDevice                                 // c: 字符设备，需已设置ModeDevice
    ModeSticky                                     // t: 只有root/创建者能删除/移动文件
    // 覆盖所有类型位（用于通过&获取类型位），对普通文件，所有这些位都不应被设置
    ModeType = ModeDir | ModeSymlink | ModeNamedPipe | ModeSocket | ModeDevice
    ModePerm FileMode = 0777 // 覆盖所有Unix权限位（用于通过&获取类型位）
)
```

###

## math（数学函数）

## bytes（操作 []byte 的常用函数）

## sync（提供基本的同步基元）

## net/http（提供了 http 客户端和服务端实现）

## fmt
