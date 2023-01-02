---
title: Go 测试
urlname: smf0wbkqelnhuz6h
date: '2022-12-30 09:46:37 +0800'
tags: []
categories: []
---

tags: [Go 测试]
categories: [Go]
cover: ''

---

> 标准库的 testing 包就是这样一个标准包，专门用来进行单元测试及自动化测试，打印日志和错误报告，方便程序员调试代码。

## 单元测试

> 测试程序是独立的文件，必须属于被测试的包，和这个包的其他程序放在一起，并且文件名满足形式\*\_test.go。由于是独立的测试文件，所以测试代码和包中的业务代码是分开的。
> \_test.go 程序不会被普通的 Go 编译器编译。
> 有 go test 会编译所有的程序（普通程序和测试程序）。
> 测试文件中必须导入 testing 包，单元测试函数是名字以 TestXxx 开头的全局函数，Xxx 部分可以为任意的字母、数字组合，但是首字母必须是大写字母，函数名可以用被测试函数的用途和功能描述，如 TestFmtInterface、TestPayEmployees 等。测试用例会按照测试源代码中写的顺序依次执行。

```go
func TestAbcde(t *testing.T)
```

## 其他测试相关报错

### could not launch process: debugserver or lldb-server not found: install Xcode's command line tools or lldb-server

> 可以先打开「终端」输入：xcode-select --install，等待安装完成
> macos 系统问题。
