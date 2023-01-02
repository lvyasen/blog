---
title: Hexo 常用命令
urlname: ug4b1uz17qtgbnag
date: '2022-12-19 14:50:48 +0800'
tags:
  - hexo
  - 配置
categories:
  - hexo
---

## 常用命令

### 创建项目

`hexo init blog` 初始化创建项目

### 创建文章

`hexo new title` 创建一篇文章

### 生成静态文件

`hexo generate` 可以简写成 `hexo g`

### 启动本地服务器

`hexo s` 或 `hexo server`
可以加些参数

- `-p` 端口,默认是 4000
- `-i` 指定服务器地址，默认是 0.0.0.0
- `-s` 静态模式 ，仅提供 public 文件夹中的文件并禁用文件监视

### 生成静态文件并启动

`hexo g && hexo s`

### 部署网站

`hexo d`

### 清除缓存文件

`hexo clean`

### 禁用插件

`hexo --safe`

### 调试模式

`hexo --debug`

## 主题配置

### 使用 butterfly 主题

在根目录中执行
`git clone -b 3.7.1 https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly`

#### 应用主题

修改站点配置文件\_config.yml，把主题改为 butterfly

```yaml
theme: butterfly
```

#### 安装插件

如果你没有 pug 以及 stylus 的渲染器，请下载安装：

```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

#### 配置

把主题文件夹中的 \_config.yml 复制到 Hexo 根目录里，同时重新命名为 \_config.butterfly.yml。

以后只需要在 \_config.butterfly.yml 进行配置就行。

Hexo 会自动合併主题中的\_config.yml 和 \_config.butterfly.yml 里的配置，如果存在同名配置，会使用\_config.butterfly.yml 的配置，其优先度较高。

### 语雀

### 从语雀同步到本地

`yuque-hexo sync`
