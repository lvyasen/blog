---
title: markdown 语法
urlname: hy7xskorf7inuc22
date: '2022-12-19 14:52:13 +0800'
tags: []
categories: []
---

tags: [markdown]
categories: [markdown]

---

使用 = 和 - 标记一级、二级标题

# 一级标题

## 二级标题

## 标题

## 段落

段落: 段落的换行是使用两个以上空格加上回车。
发生大是大非撒旦法
算法

### 字体

_斜体文本_
_斜体文本_
**粗体文本**
**粗体文本**
**_粗斜体文本_**
**_粗斜体文本_**

### 分隔线

---

---

---

---

---

### 删除线

RUNOOB.COM
GOOGLE.COM
~~BAIDU.COM~~

### 下划线

带下划线文本

### 脚注

[^要注明的文本]

## 列表

### 无序列表

- 1
- 2
- 3

----分割线----

- 1
- 2
- 3

----分割线----

- 1
- 2
- 3

### 有序列表

1. 1
2. 2
3. 3

### 列表嵌套

1. 第一项:
   - 嵌套 1
   - 嵌套 2

## 区块

### 引用

> 区块引用
> 引用

### 引用中使用列表

> 1. 列表一
> 2. 列表二

## 代码

### 单行代码

`echo 'hello world'`

### 代码区块

```python
class test():
  print("这是个测试代码")
```

## 链接

### 普通链接

[链接名称](%E9%93%BE%E6%8E%A5%E5%9C%B0%E5%9D%80)
或者
<链接地址>

### 高级链接

链接也可以使用变量来代替
这个链接使用 test 作为变量 [百度][test]
最后在文档末尾赋值
[test]: www.baidu.com

## 图片

![可选标题](%E5%9B%BE%E7%89%87%E5%9C%B0%E5%9D%80#crop=0&crop=0&crop=1&crop=1&id=uhuiM&originalType=binary∶=1&rotation=0&showTitle=true&status=done&style=none&title=%E5%8F%AF%E9%80%89%E6%A0%87%E9%A2%98 "可选标题")
不过上边不能指定图片大小,可以使用 img 标签代替

## 表格

`|` 区分不同单元格、`-` 来分割表头和其他行

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |

### 设置对齐方式

- `:-` 左对齐
- `-:` 右对齐
- `:-:` 中间对齐
  | 左对齐 | 中间对齐 | 右对齐 |
  | :--- | :---: | ---:|
  | 单元格 | 单元格 | 单元格 |

## 高级技巧

### 支持的 html 元素

#### kbd 标签

#### 转义字符

使用 `\` 反斜线转义字符
需要转义的字符

```javascript
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```

#### 公式

> 当想插入数学公式时使用 `$$` 包裹

![](<https://g.yuque.com/gr/latex?J_%5Calpha(x)%20%3D%20%5Csum_%7Bm%3D0%7D%5E%5Cinfty%20%5Cfrac%7B(-1)%5Em%7D%7Bm!%20%5CGamma%20(m%20%2B%20%5Calpha%20%2B%201)%7D%20%7B%5Cleft(%7B%20%5Cfrac%7Bx%7D%7B2%7D%20%7D%5Cright)%7D%5E%7B2m%20%2B%20%5Calpha%7D%20%5Ctext%20%7B%EF%BC%8C%E7%8B%AC%E7%AB%8B%E5%85%AC%E5%BC%8F%E7%A4%BA%E4%BE%8B%7D%20%0A#card=math&code=J_%5Calpha%28x%29%20%3D%20%5Csum_%7Bm%3D0%7D%5E%5Cinfty%20%5Cfrac%7B%28-1%29%5Em%7D%7Bm%21%20%5CGamma%20%28m%20%2B%20%5Calpha%20%2B%201%29%7D%20%7B%5Cleft%28%7B%20%5Cfrac%7Bx%7D%7B2%7D%20%7D%5Cright%29%7D%5E%7B2m%20%2B%20%5Calpha%7D%20%5Ctext%20%7B%EF%BC%8C%E7%8B%AC%E7%AB%8B%E5%85%AC%E5%BC%8F%E7%A4%BA%E4%BE%8B%7D%20%0A&id=zISJI>)

## 文字颜色、大小、字体

### 颜色

> 通过 color 控制颜色
> 文字颜色预览

### 大小

size 为 1：size 为 1
size 为 2：size 为 2
size 为 3：size 为 3
size 为 4：size 为 4
size 为 6：size 为 6

### 字体

我是黑体字
我是宋体字
我是楷体字
我是微软雅黑字
我是 fantasy 字
我是 Helvetica 字

### 背景色

| 背景色的设置是按照十六进制颜色值：#F4A460 |
| ----------------------------------------- |

| 背景色的设置是按照十六进制颜色值：#FF6347 |
| ----------------------------------------- |

| 背景色的设置是按照十六进制颜色值：#D8BFD8 |
| ----------------------------------------- |

| 背景色的设置是按照十六进制颜色值：#008080 |
| ----------------------------------------- |

| 背景色的设置是按照十六进制颜色值：#FFD700 |
| ----------------------------------------- |

### Emoji

![](https://gw.alipayobjects.com/os/lib/twemoji/11.2.0/2/svg/1f60a.svg#crop=0&crop=0&crop=1&crop=1&height=18&id=pzqzv&originHeight=150&originWidth=150&originalType=binary∶=1&rotation=0&showTitle=false&status=done&style=none&title=&width=18)
