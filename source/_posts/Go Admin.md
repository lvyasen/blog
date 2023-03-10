---
title: Go Admin
urlname: vtxnheop8ouf1kts
date: '2022-12-22 16:34:20 +0800'
tags: []
categories: []
---

tags: [go-admin ]
categories: [Go]
cover: ''

---

## 介绍

> 基于 Gin + Vue + Element UI OR Arco Design OR Ant Design 的前后端分离权限管理系统,系统初始化极度简单，只需要配置文件中，修改数据库连接，系统支持多指令操作，迁移指令可以让初始化数据库信息变得更简单，服务指令可以很简单的启动 api 服务

1. 多租户：系统默认支持多租户，按库分离，一个库一个租户。
2. 用户管理：用户是系统操作者，该功能主要完成系统用户配置。
3. 部门管理：配置系统组织机构（公司、部门、小组），树结构展现支持数据权限。
4. 岗位管理：配置系统用户所属担任职务。
5. 菜单管理：配置系统菜单，操作权限，按钮权限标识，接口权限等。
6. 角色管理：角色菜单权限分配、设置角色按机构进行数据范围权限划分。
7. 字典管理：对系统中经常使用的一些较为固定的数据进行维护。
8. 参数管理：对系统动态配置常用参数。
9. 操作日志：系统正常操作日志记录和查询；系统异常信息日志记录和查询。
10. 登录日志：系统登录日志记录查询包含登录异常。
11. 接口文档：根据业务代码自动生成相关的 api 接口文档。
12. 代码生成：根据数据表结构生成对应的增删改查相对应业务，全程可视化操作，让基本业务可以零代码实现。
13. 表单构建：自定义页面样式，拖拉拽实现页面布局。
14. 服务监控：查看一些服务器的基本信息。
15. 内容管理：demo 功能，下设分类管理、内容管理。可以参考使用方便快速入门。
16. 定时任务：自动化任务，目前支持接口调用和函数调用。

## 安装

```bash
# 从 github 中获取
# 获取后端代码
git clone https://github.com/go-admin-team/go-admin.git

# 获取前端代码
git clone https://github.com/go-admin-team/go-admin-ui.git

#从码云中获取
git clone https://gitee.com/go-admin-team/go-admin.git &&
git clone https://gitee.com/go-admin-team/go-admin-ui.git
```

### 启动

```markdown
# 进入 go-admin 后端项目

cd ./go-admin

# 更新整理依赖

go mod tidy

# 编译项目

go build

# 修改配置

# 文件路径 go-admin/config/settings.yml

vi ./config/setting.yml

# 1. 配置文件中修改数据库信息

# 注意: settings.database 下对应的配置数据

# 2. 确认 log 路径

//在 windows 环境如果没有安装中 CGO，会出现这个问题；

# 启动

$ ./go-admin server -c config/settings.yml
```

### 使用 docker 编译启动

```bash
# 编译镜像
docker build -t go-admin .

# 启动容器，第一个go-admin是容器名字，第二个go-admin是镜像名称
# -v 映射配置文件 本地路径：容器路径
docker run --name go-admin -p 8000:8000 -v /config/settings.yml:/config/settings.yml -d go-admin-server
```

### 初始化数据库

```bash
# 首次配置需要初始化数据库资源信息
# macOS or linux 下使用
$ ./go-admin migrate -c config/settings.yml

# ⚠️注意:windows 下使用
$ go-admin.exe migrate -c config/settings.yml


# 启动项目，也可以用IDE进行调试
# macOS or linux 下使用,这边用这个
$ ./go-admin server -c config/settings.yml


# ⚠️注意:windows 下使用
$ go-admin.exe server -c config/settings.yml
```

#### sys_api 表的数据如何添加

在项目启动时，使用-a true 系统会自动添加缺少的接口数据
`./go-admin server -c config/settings.yml -a true`

### 生成文档

```bash
go generate
```

### 其他平台编译

```bash
# windows
env GOOS=windows GOARCH=amd64 go build main.go

# or
# linux
env GOOS=linux GOARCH=amd64 go build main.go
```

### 前端启动

```bash
# 安装依赖
npm install

# 建议不要直接使用 cnpm 安装依赖，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npmmirror.com

# 启动服务
npm run dev

```

#### 相关问题解决

```bash
# calc 问题
yarn add --dev sass-migrator

```

## 目录文件

```bash
.
├── Dockerfile </li>
├── LICENSE.md </li>
├── Makefile </li>
├── README.en.md </li>
├── README.md </li>
├── _config.yml </li>
├── app # 应用文件夹
│   ├── admin # admin应用
│   │   ├── apis # api </li>
│   │   ├── models # 模型 </li>
│   │   ├── router # 路由 </li>
│   │   └── service # 业务逻辑 </li>
│   └── jobs #自动化作业
│       ├── apis # api </li>
│       ├── models # 模型 </li>
│       ├── router # 路由 </li>
│       └──  service # 业务逻辑</li>
├── cmd # 命令 </li>
├── common #公共类 </li>
├── config # 系统配置 </li>
├── docs # 文档 </li>
├── go.mod </li>
├── go.sum </li>
├── logger # 日志包 </li>
├── main.go </li>
├── package-lock.json </li>
├── static # 静态文件 </li>
├── temp # 临时文件 </li>
├── template # 模版文件 </li>
├── test # 测试 </li>
└── tools # 工具 </li>
```

## 编写接口

> 程序目录结构

```bash
go-admin
  app
    admin
      apis //接口
      models //模型
      router //路由
      service
        dto
```

### 编写处理逻辑

> 在 `/app/admin/apis` 中目录中创建 `article.go` 文件

```go
package apis

import (
 "github.com/gin-gonic/gin"
 "go-admin/common/apis"
)

type Article struct {
 apis.Api
}

// GetArticleList 获取文章列表
func (e Article)GetArticleList(c *gin.Context) {
 err := e.MakeContext(c).
  Errors
 if err != nil {
  e.Logger.Error(err)
  return
 }
 e.OK("hello world ！","success")
}
```

### 路由注册

> 在 `/app/admin/router/article.go`中编写路由
> router 注册类型，我们比较常用的就是 GET、POST、PUT、DELETE 等

```go
package router

import (
 "go-admin/app/admin/apis"

 "github.com/gin-gonic/gin"
 jwt "github.com/go-admin-team/go-admin-core/sdk/pkg/jwtauth"
)

func init() {
 routerCheckRole = append(routerCheckRole, registerArticleRouter)
}

// 需认证的路由代码
func registerArticleRouter(v1 *gin.RouterGroup, authMiddleware *jwt.GinJWTMiddleware) {
 api:= apis.Article{}
 r := v1.Group("")
 {
  r.GET("/articleList", api.GetArticleList)
 }
}
```

#### path

path 是一个匹配 URL 的准则（有点正则表达式的意思），当 go-admin 响应一个请求时，它会从注册的 url 第一项开始，按照顺序一次匹配，直到找到匹配项。
这些准则不会匹配 GET 和 POST 参数或域名。例如，URL 在处理请求 [http://www.zhangwj.com/articleList](http://www.zhangwj.com/articleList) 时，它会尝试匹配 articleList 。处理请求 [http://www.zhangwj.com/articleList?page=3](http://www.zhangwj.com/articleList?page=3) 时，也只会尝试匹配 blog/list。
path 也支持带参数的写法，例如 r.GET("/articleList/",apis.GetArticleList), 这个时候会按照这 /articleList/进行匹配可以是字符串，可以是数字等任意字符，当然也是可以限制的，这里我们不再展开。

### 重新编译启动

```bash
go build

./go-admin server -c=config/settings.yml
```
