---
title: Go 常用库
urlname: gklhq3uaw0d06tdx
date: '2022-12-21 18:29:09 +0800'
tags: []
categories: []
---

tags: [cobra ]
categories: [Go]
cover: ''

---

# 命令行

## cobra（命令行） 

> Cobra 是一个用[Go 语言](https://so.csdn.net/so/search?q=Go%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)实现的命令行工具。并且现在正在被很多项目使用，例如：Kubernetes,、Hugo 和 Github CLI 等。
> 通过使用 Cobra，不仅可以快速的创建命令行界面，也可以快速开发基于 Cobra 的应用程序。

### 功能：

- 轻松创建基于子命令的 CLI：如 app server、app fetch 等。
- 自动添加-h,--help 等帮助性 Flag
- 自动生成命令和 Flag 的帮助信息
- 创建完全符合 POSIX 的 Flag(标志)（包括长、短版本）
- 支持嵌套子命令
- 支持全局、本地和级联 Flag
- 智能建议（app srver... did you mean app server?）
- 为应用程序自动生成 shell 自动完成功能（bash、zsh、fish、powershell）
- 为应用程序自动生成 man page
- 命令别名，可以在不破坏原有名称的情况下进行更改
- 支持灵活自定义 help、usege 等。
- 无缝集成[viper](https://link.zhihu.com/?target=http%3A//github.com/spf13/viper)构建 12-factor 应用

### 安装

```bash
go get -u github.com/spf13/cobra@latest
```

#### cobra-cli

> cobra-cli 可以帮助我们快速创建出一个 cobra 基础代码结构。

```go
go install github.com/spf13/cobra-cli@latest
```

##### 使用并创建项目

```bash
mkdir cobra-test
cd cobra-test
go mod init app

//使用客户端初始化
cobra-cli  init
	--author "walker 18530893662@163.com"
	--license apache

#生成结构
app
├── cmd
│   └── root.go
├── go.mod
├── go.sum
├── LICENSE
└── main.go


//添加新命令
cobra-cli add ebook
cobra-cli add ping
//运行命令
go run main.go ebook

//添加 ebook 子命令
cobra-cli add ebookLists -p "ebookCmd"
//运行子命令
go run main.go ebbok ebookLists
```

##### 写代码

```go
/*
Copyright © 2022 NAME HERE <EMAIL ADDRESS>

*/
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

// ebookCmd represents the ebook command
var ebookCmd = &cobra.Command{
	Use:   "ebook",
	Short: "获取书籍列表",
	Long: `根据 URL 获取用户的书籍列表`,
	Args: cobra.MinimumNArgs(1),//对参数进行控制
	//noArgs: 如果命令有参数就会报错。
	//MaximumNArgs(int)：设置命令可接受的最大参数数量 。
	//ExactArgs(int)：如果命令的参数没有指定的数量就会报错。
	//RangeArgs(min, max)：命令的参数必须在指定的范围内。
	Run: func(cmd *cobra.Command, args []string) {
		//获取命令行参数
		fmt.Println("args: ",args)
	},
	//运行但是如果错误返回一个 error
	RunE: func(cmd *cobra.Command, args []string) {

	},
}

func init() {
	rootCmd.AddCommand(ebookCmd)

	//局部 flag
	// toggele为flag的名称
	// t为flag的简称
	// false为默认值
	// 最后一个参数为flagde 描述
	ebookCmd.Flags().BoolP("lists", "l", false, "局部简短的描述")
	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:

	//全局 flag
	ebookCmd.PersistentFlags().String("name", "n", "全局简短的描述")

}

```

##### 生成器配置文件（非必须，旨在消除重复信息）

```yaml
author: Steve Francia <spf@spf13.com>
year: 2020
license:
  header: This file is part of CLI application foo.
  text: |
    {{ .copyright }}

    This is my license. There are many like it, but this one is mine.
    My license is my best friend. It is my life. I must master it as I must
    master my life.
```

##

# 数据库

## Gorm

## Xorm
