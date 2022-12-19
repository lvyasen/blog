---
title: Go 数据类型
urlname: yliyqsrdmnu0h2hv
date: '2022-12-19 14:36:24 +0800'
tags:
  - 数据类型
  - go字符串，切片，通道，接口
categories:
  - Go
---

## 基础数据类型 2

1. 布尔类型。
2. 数字类型。
3. 字符串类型。
4. 复合（派生）类型。
5. 错误类型。

## 字符与字符串

> 字符表示单个字符,其类型是 rune
> 字符串表示多个字符,其类型是 string

### rune 类型

> 表示一个字符
> rune 表达式必须在一对单引号之中

```go
var c rune = 'f'
var s = 's'
```

### string 类型

> 字符串常量表达式需要在一对双引号中
> 段落式字符串 使用 ` 字符表示
> 使用 range 迭代字符串，返回两个值，一个是被迭代的 utf-8 的第一个字节在字符串中的索引，另一个是对应的字符且类型为 rune

```go
var name = "张三"
var md = `
1.需求分析
2.技术开发
3.测试 bug
4.上线部署

```

## 数值类型

> 提供了数学运算的基础元素,包括
>
> 1. 整数
> 2. 浮点数
> 3. 复数

### 整数类型

| 类型                      | 描述                      | 范围                                    |
| ------------------------- | ------------------------- | --------------------------------------- |
| int8                      | 8 位有符号整数            | -128~127                                |
| uint8                     | 8 位无符号整数            | 0~255                                   |
| int16                     | 16 位有符号整数           | -32768~32767                            |
| uint16                    | 16 位无符号整数           | 0~65535                                 |
| int32                     | 32 位有符号整数           | -2147483648~2157482647                  |
| uint32                    | 32 位无符号整数           | 0~4284867295                            |
| int64                     | 64 位有符号整数           | -922337203685477580~9223372036854775807 |
| uint64                    | 64 位无符号整数           | 0~18446744073709551615                  |
| int                       | 在 32 位处理器上为 int32  |
| 在 64 位处理器上为 int64  |                           |
| uint                      | 在 32 位处理器上为 uint32 |
| 在 64 位处理器上为 uint64 |                           |
| byte                      | uint8 别称                | 0~255                                   |
| float32                   | 32 位浮点数               |                                         |
| float64                   | 64 位浮点数               |                                         |
| complex64                 | 64 位复数                 |                                         |
| complex128                | 128 位复数                |                                         |

```go
//获取变量所占内存
unsafe.Sizeof(<变量>)
//数字转字符串
string := strconv.Itoa(int)
string := strconv.FormatInt(int64,10)
//字符串转数字
int，err := strconv.Atoi(string)
int64,err :=
```

### 浮点类型

> float32 大约可以提供小数点后 6 位的精度，而 float64 可以提供小数点后 15 位的精
> 通常情况应该优先选择 float64，因为 float32 的精度较低，在累积计算时误差扩散很快，而且因为浮点数和整数的底层解释方式完全不同，float32 能精确表达的最小正整数并不大。

| 类型    | 描述        | 范围      |
| ------- | ----------- | --------- |
| float32 | 32 位浮点数 | 6 位精度  |
| float64 | 64 位浮点数 | 15 位精度 |

复数

### 复数类型

| 类型       | 描述       | 范围 |
| ---------- | ---------- | ---- |
| complex64  | 64 位复数  |      |
| complex128 | 128 位复数 |      |

## 复合数据类型

1. 数组（Array）。
2. 切片（slice）。
3. 字典（map）。
4. 结构体（struct）。
5. 指针（pinter）。
6. 函数（function）。
7. 接口（interface）。
8. 通道（channel）。

### 数组（值类型）

> 数组（array）是一种集合，它把一定数量且类型相同的对象放到一起，形成一个整体。
> 数组在定义时就会确定元素个数，初始化之后无法更改元素数量。
> 其占用的内存是连续分配的，由于内存连续，CPU 能把正在使用的数据缓存更久的时间。
> 内存连续很容易计算索引，可以快速迭代数组里的所有元素
> 可以使用空接口 interface{}作为类型。但使用值时，必须先做一个类型判断。
> 由于数组传递，使用的是值传递，所以参数是数组时，参数长度不能过大，因为消耗很多内存。
> 使用\*运算符就可以访问元素指针所指向的值。
> 数组变量的类型包括数组长度和每个元素的类型。只有这两部分都相同的数组，才是类型相同的数组，才能互相赋值
> 复制数组指针，只会复制指针的值，而不会复制指针所指向的值。
> 如何替代数组传递？

> 1. 使用数组的指针
> 2. 使用切片

```go
//数组声明有三种方式
//第一种
var arr [3]int
//第二种
var arr = [3]int{1,2,3}
//第三种 常用，可自动推断长度
var arr := [...]int{1,2,3}

//如果将数组的类型声明为 interface 空接口，那么其中类型就可以是任意类型,
arr := [...]interface{1,"2",3}


//获取数组长度
len(arr)
//获取容量
cap(arr)

//访问数组元素
arr[1]

//遍历数组

```

#### 数组值的复制

> Go 语言中的数组在赋值和函数调用时的形参都是值复制。

```go
//下面就是值复制，无论怎么修改 b、c都不会影响 a。
arr := [...]int{1,2,3}
b=a
func Change(c [3]int){}

```

#### 底层原理

> 数组在编译时就需要确定其长度和类型。
> 数组在编译时构建抽象语法树阶段的数据类型为 TARRAY，通过 NewArray 函数进行创建，AST 节点的 Op 操作为 OARRAYLIT

```go
//TARRAY内部的Array结构存储了数组中元素的类型及数组的大小。
type Array struct {
    Elem  ＊Type
    Bound int64
}
```

#### *[n]T 与[n]*T 的区别

> *[n]T 与[n]*T 这两种声明格式看起来很像，但它们的含义是完全不同的。
> （1）*[n]T：指针类型，存放类型为[n]T 的实例内存地址。
> （2）[n]*T：数组类型，其元素类型为指向 int 数值的指针类型（\*int）

### 切片（引用类型）

> 切片（slice）是对底层数组一个连续片段的引用（该数组称为相关数组，通常是匿名的），所以切片是一个引用类型（和数组不一样）。
> 切片便于使用和管理数据集合。
> 切片的长度永远不会超过它的容量。
> 切片是围绕动态数组构建的，可以按照需求自动增长和缩小。
> 切片的动态增长是通过内置函数 append 来实现的，这个函数可以快速且高效地增长切片。还可以通过对切片再次切片来缩小一个切片的大小。
> 切片的底层内存也是在连续块中分配的，切片还能获得索引、迭代以及为垃圾回收优化的好处
> `和数组不同的是，切片不用指定固定长度。`
> 一个切片在运行时由`指针（date）、长度（len）和容量（cap）`3 部分构成
> 使用内置函数 make()可以给切片初始化，该函数指定切片类型、长度和可选容量的参数。
> 由于切片类型是引用类型，所以不需要额外的内存，并且比数组高效，因此切片比数组常用。
> 切片只能访问到其长度内的元素。试图访问超出其长度的元素将会导致语言运行时异常

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670925607311-252f9a78-b6b8-42e6-b67f-240b14d64a73.png#averageHue=%23fdfdfa&clientId=u8e3492c7-8455-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=239&id=uf9d653b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=478&originWidth=921&originalType=binary∶=1&rotation=0&showTitle=false&size=53740&status=done&style=none&taskId=u917ea2fb-58c2-4dd0-acd1-0a65faf458a&title=&width=460.5)

```go
#初始化,数组需要指定长度，因此 [] 中需要指定长度
var slice []string{}#当初始化一个切片时不指定值，会创建一个 nil 切片
#使用 make 初始化
slice := make([]string,10,12)#10表示长度，12 表示容量
#使用数组方式初始化
slice := []int{1,2,3}

#添加元素 append 函数
func append(slice ［］Type, elems ...Type) ［］Type
slice = slice.append(slice,4)

#复制数据
copy(newSlice,slice[0,3])


```

#### 切片截取、复制

> 改变切片长度得到新切片的过程称为切片重组。
> 做法：newSlice = slice[0:end]
> 在一个切片基础上重新划分一个切片时，新的切片会继续引用原有切片相关数组，相当于一个窗口。
> 如果需要复制数据而不是引用数据，使用内置函数 copy(newSlice，slice) 复制数据
> 切片的复制其实也是值复制，但这里的值复制指对于运行时 SliceHeader 结构的复制。如图 7-3 所示，底层指针仍然指向相同的底层数据的数组地址，因此可以理解为数据进行了引用传递。切片的这一特性使得即便切片中有大量数据，在复制时的成本也比较小，这与数组有显著的不同

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671073540276-810c4db4-6172-47e8-9c3c-9dab4e8a9bd2.png#averageHue=%23f4f4d8&clientId=uc7ce50d9-4468-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=297&id=u3cb0505a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=594&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=269668&status=done&style=none&taskId=ucdd9234c-d010-4850-a098-d6a694d9a64&title=&width=720)

#### 切片的增长、扩容

> 相对于数组，使用切片的好处就是按需增加切片的容量。
> 函数 append 总是会增长新切片的长度，而容量有可能会改变，这取决于切片的容量。

#### 字面量的初始化

> 当使用形如[]int{1，2，3}的字面量创建新的切片时，会创建一个 array 数组（[3]int{1，2，3}）存储于静态区中，并在堆区创建一个新的切片，在程序启动时将静态区的数据复制到堆区，这样可以加快切片的初始化过程。

#### make 初始化

> 对形如 make（[]int，3，4）的初始化切片。在类型检查阶段 typecheck1 函数中，节点 Node 的 Op 操作为 OMAKESLICE，并且左节点存储长度为 3，右节点存储容量为 4
> 对于字面量的重要优化是判断变量应该被分配到栈中还是应该逃逸到堆区。如果 make 函数初始化了一个太大的切片，则该切片会逃逸到堆中。如果分配了一个比较小的切片，则会直接在栈中分配。
> 此临界值定义在 cmd/compile/internal/gc.maxImplicitStackVarSize 变量中，默认为 64KB，可以通过指定编译时 smallframes 标识进行更新，因此，make（[]int64，1023）与 make（[]int64，1024）实现的细节是截然不同的。

### 映射（引用类型）

> 映射是一种数据结构，用于存储一系列无序的键值对。

#### 内部实现

> 映射是一个集合，可以使用类似处理数组和切片的方式迭代映射中的元素。
> 但映射是无序的集合，意味着没有办法预测键值对被返回的顺序。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671430908220-71632850-de37-41c9-b720-d63896f6a164.png#averageHue=%23e5e5e5&clientId=u6fc25da0-5b53-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=513&id=u06b6f0c3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1026&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=264867&status=done&style=none&taskId=u0a1af87d-241e-4ab1-9289-25bc7e74f00&title=&width=720)
