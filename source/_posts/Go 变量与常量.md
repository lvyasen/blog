---
title: Go 变量与常量
urlname: lelkxd4py1w71bir
date: '2022-12-19 14:36:24 +0800'
tags:
  - go 变量与常量
categories:
  - Go
---

她是类型常量的默认类型分别是 bool，rune，int，float64，complex128 或 string，具体取决于它是布尔值、字符、整数、浮点数、复数还是字符串常量 go 语言有四类标记:

1. 标识符（identitfiers）。就是变量名称
2. 关键字（keywords）。内置的关键字
3. 运算符（operators）。
4. 字面量（literals）。常量和变量的初始值
   > 下滑线也被认为是字母 是
   > Go 编译器会做逃逸分析，所以由 Go 的编译器决定在哪里（堆或栈）分配内存，保证程序的正确性。

## 关键字

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670546558531-ec30889e-b10e-4ded-94df-0d095034ef5d.png#averageHue=%23f4f4f4&clientId=u46533321-723c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=131&id=ue7702be3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=262&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=52445&status=done&style=none&taskId=ub5f99445-5ea5-404a-a52c-cc859567af3&title=&width=720)

## 变量

> 变量初始化过程分为两个阶段
>
> 1. 声明阶段 指明变量名称和数据类型 变量名称是一个标识符
> 2. 赋值阶段 为变量分配一个有效值
>
> 中文也可以作为标识符
> Go 语言有个强制规定，在函数内一定要使用全部声明的变量，若存在未使用的变量，则代码将编译失败。
> 在 Go 语言中，指针属于引用类型，其他的引用类型还包括切片、字典和通道，如果传递引用类型参数或者赋值给引用类型变量，原始数据有改动时它们也会发生变化。
> Go 语言中的数组是值类型。
> 被引用的变量一般存储在堆内存中，以便系统进行垃圾回收（GC），且比栈拥有更大的内存空间

```go
//变量声明格式，多用于函数体外，单个声明不加括号
var <标识符> <数据类型>
//声明并赋值
var <标识符> <数据类型> = <值>
//自动推断并赋值，简式声明也叫短变量声明，这种声明没有类型（隐式类型，会根据使用环境推断他所具备的类型）
//简式声明一般用在函数体内，但是注意全局变量声明不要和简式声明同名，否则会出现偶然的变量隐藏。
//所谓变量隐藏就是因为变量作用域不同导致的现象。
<标识符> := <值>

//批量声明
var (
	<标识符> = <值>
    <标识符> = <值>
)

//组合赋值
var <标识符 1>,<标识符 2> := <值 1>,<值 2>

//匿名变量（空白标识符） 将变量的标识符命名为 _ 那么他就成为了匿名变量,该变量将会被舍弃

//如果想交换两个变量的值
a,b =b,a

//执行go tool compile命令，可以看到变量逃逸分析。

```

go 的作用域规则：
■ 在 Go 语言中，在顶层声明的常量、类型、变量或函数的标识符的范围是包块。
■ 导入包名称范围是包含导入声明的文件的文件块。
■ 方法接收器、函数参数或结果变量的标识符的范围是函数体。
■ 在函数内声明的常量或变量标识符的范围从声明语句的末尾开始，到最内层包含块的末尾结束。
■ 在函数内声明的类型标识符的范围从标识符开始，到最内层包含块的末尾结束。
■ 块中声明的标识符可以在内部块中重新声明。

### 零值（nil）

> 当一个变量被 var 声明后，如果没有明确的指定其初始值，Go 语言会自动初始化其值类型为此类型的初始值。
> 对于复合类型的零值，Go 会自动递归将每个元素初始化为其类型对应的零值。
> 在一个 nil 的切片中添加元素是没问题的，但需要注意的是对一个字典做同样的事将会生成一个运行时的异常。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670547631223-27f04872-239a-4e01-8256-f27f19197228.png#averageHue=%23f5f5f5&clientId=u46533321-723c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=156&id=u578d201c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=311&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=42799&status=done&style=none&taskId=u47b3e32a-5c03-4257-b68f-e079e55f7c6&title=&width=720)

## 常量

> 用于存储不会改变的数据。
> 常量不能被重新赋予任何值。
> 声明必须使用 const 关键字,初始化方法与变量相同。
> GO 可以省略说明的数据类型，因为编译器可以根据常量的值进行类型推断。
> 无类型常量的默认类型分别是 bool，rune，int，float64，complex128 或 string，具体取决于它是布尔值、字符、整数、浮点数、复数还是字符串常量。
> 在常量定义中，没有强制要求常量名全部大写，但一般都会全部字母大写，以便阅读。

```go
const <标识符> <数据类型> = <值>

//批量声明
const (
	<标识符> = <值>
)
```

> 变量声明后如果未进行赋值,应用程序会为变量分配一个默认值

### iota（极小的，希腊语第九个字母）

> iota 比较特殊，他是一个可以被编译器修改的常量，
> 在每一个 const 关键字出现时被重置为 0 ，在一次 const 出现前，每出现一次 iota，其代表的数字自动加 1。
> 常用于枚举。

```go
const(
    a=iota//0
    b=iota//1
    c=iota//2
)
//简写形式
const(
    a=iota//0
    b//1
    c//2
)

//如果新的常量声明后，iota 不再向下赋值。后面如果没有赋值，则继承上一个常量值。
const(
    a=iota//0
    b=8//8
    c//8
)
//应用场景
const(     _ = iota// 通过赋值给空白标识符来忽略值
    KB ByteSize = 1<<(10*iota)
    MB
	GB
    TB
    PB
    EB
    ZB
    YB
)
```

## 字面量

> 字面量是指由字母、数字等构成的字符串或者数值，它只能作为右值出现。

### 整数字面量

> 表示整数常量的数字序列。可选前缀设置非十进制基数：0 表示八进制，0x 或 0X 表示十六进制。在十六进制数中，字母 a ～ f 和 A ～ F 表示值 10 到 15

### 浮点数字面量（Floating-point literals）

> 浮点数字面量是浮点常量的十进制表示。它有一个整数部分、一个小数点、一个小数部分和一个指数部分。整数和小数部分包括十进制数字，指数部分是 e 或 E，后跟可选的带符号的十进制指数。可以省略整数部分或小数部分中的一个，可以省略小数点或指数之一。

```go
0.
72.
40 072
.40  //
 == 72.40 2.7
1828 1.e
+0 6.6
7428e-1
1 1E6
 .25
 .12
345E+5

```

### 虚数字面量（Imaginary literals）

> 虚数字面量是复数常数的虚部的十进制表示。它由浮点字面量或十进制整数后跟小写字母 i 组成。

```go
0i
011
i  // == 11i 0.i
 2.7
1828i 1.e
+0i 6.6
7428e-11i 1E6
i .25
i .12
345E+5
```

### Rune 字面量（Rune literals）

> Rune 字面量是标识 Unicode 代码点的整数值。Rune 字面量表示用单引号括起来的一个或多个字符，如'x'或'\n'。在单引号内，除了换行符和未转义的单引号外，任何字符都可以出现。单引号字符表示字符本身的 Unicode 值，而以反斜杠开头的多字符序列表示各种格

### 字符串字面量（String literals）

> 原始字符串字面量
> 解释的字符串字面量。
> 原始字符串字面量是反引号之间的字符序列，如`foo`何字符都可能出现
> 解释的字符串文字是双引号之间的字符序列，如"bar"。

## 运算符

■ 算术运算符
■ 关系运算符
■ 逻辑运算符
■ 位运算符
■ 赋值运算符
■ 其他运算
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551631543-0f726528-9c20-49da-8c38-6a06d268c9f1.png#averageHue=%23f8f8f8&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=157&id=u0fa1827d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=314&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=38511&status=done&style=none&taskId=ue3fcae8a-6142-41c8-9634-012bd984d03&title=&width=720)

### 运算符优先级

> 优先级数字越大表示优先级越高

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670552106913-1ec01f7d-a661-4114-ae90-38b5009b6c6e.png#averageHue=%23f7f7f7&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=ufd2c6dbd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=308&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=32507&status=done&style=none&taskId=u2d85ec51-eb86-4276-8a8b-d1a75a4bb14&title=&width=720)

### 算术运算符

> 四个`标准算术运算符`（+，-，\*，/）适用于整数、浮点和复数类型;+也适用于字符串。
> 按位逻辑和移位运算符仅适用于整数

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551784253-bb851b7b-429c-409d-a365-d64d18036198.png#averageHue=%23f8f8f8&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=206&id=u5629921e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=411&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=58144&status=done&style=none&taskId=u00d7276e-ef6d-411c-b885-d4d2f368c48&title=&width=720)

### 关系运算符

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551822080-5cfbd7b6-dc47-4839-98ab-57921542e268.png#averageHue=%23efefef&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=181&id=u948acca5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=361&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=145590&status=done&style=none&taskId=ud0f2dbcd-61f0-4af6-b5ae-9320ca12964&title=&width=720)

### 逻辑运算符

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551860774-7637edf7-0166-4226-b7e7-9ea9db040dee.png#averageHue=%23ededed&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=107&id=ue0e0e146&margin=%5Bobject%20Object%5D&name=image.png&originHeight=214&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=72121&status=done&style=none&taskId=ub46568bb-8019-454e-951b-c13ff99c8e7&title=&width=720)

### 位运算符

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551883984-107c9014-aad1-4df2-90d8-e2ded21a1fce.png#averageHue=%23ececec&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=176&id=uebd6246e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=351&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=188931&status=done&style=none&taskId=uec050d14-9b52-4f4b-81d0-639a0d2ac2e&title=&width=720)

### 赋值运算符

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670551928018-91533f1e-009d-405a-8cfb-c0084f0c83ff.png#averageHue=%23f6f6f6&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=205&id=u286f8a6f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=92405&status=done&style=none&taskId=uee318818-47a1-44ea-93bd-6156731ec6c&title=&width=720)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670552033489-00eb8a3b-5d26-4d52-9bff-161f4cffcadf.png#averageHue=%23f2f2f2&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=131&id=ua819411c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=261&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=63408&status=done&style=none&taskId=u598ca559-64e5-4741-a9f9-f7467184b56&title=&width=720)

### 其他运算符

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670552056230-270b33c8-a9ae-4426-89bb-771f0beae055.png#averageHue=%23eeeeee&clientId=u779c3df1-dfea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=88&id=u3d504569&margin=%5Bobject%20Object%5D&name=image.png&originHeight=175&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=41330&status=done&style=none&taskId=u6aaf5771-c697-4172-a514-1734635afd8&title=&width=720)

### 特殊运算符

#### 位清除&^

> 将指定位置上的值设置为 0。
> 将运算符左边数据相异的位保留，相同位清零。

```go
X=2
Y=4
x&^y==x&(^y)

```

#### ^异或（XOR）
