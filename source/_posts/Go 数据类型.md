---
title: Go 数据类型
urlname: yliyqsrdmnu0h2hv
date: '2022-12-19 14:36:24 +0800'
tags:
  - 数据类型
  - go字符串
  - 切片
  - 通道
  - 接口
  - 函数
categories:
  - Go
---

> 因为所有的类型包括自定义类型其实都已经实现了空接口 interface{}，所以空接口 interface{}可以被当作任意类型的数值。

## 基础数据类型

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
   > 引用类型和值类型区别：
   >
   > - 值在传递参数时，传递的是值的 copy。
   > - 引用类型传递时传递的是地址。

### 引用类型

1. 切片（slice）。
2. 字典（map）。
3. 函数（function）。
4. 接口（interface）。
5. 通道（channel）。

### 值类型

1. 数组（Array）
2. 基本类型：int、string、float、bool。
3. 结构体。

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
> 数组是构造切片和映射的基石。
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
> Go 语言里切片经常用来处理数据的集合。
> 将切片或者映射传递给函数成本很小，并且不会复制底层的数据结构

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
> 在同一个 map 对象中，元素的值可以重复出现，但 key 必须是唯一。
> 映射用来处理具有键值对结构的数据。
> 映射的增长没有容量或者任何限制。

#### 声明

> 仅仅声明变量是不能够操作的。
> 调用 `make`函数可以创建映射对象实例。
> 在声明时不需要知道字典的长度，字典是可以动态增长的。
> 创建映射时，更常用的方法是使用映射字面量

```go
map[<keyType>]<valType>
var userInfo map[string]interface{}
userInfo["name"] = "张三"
//使用 make 创建实例
var userInfo = make(map[string]string)
```

#### 检查 key 是否存在

> 如果访问某个 key 在映射中不存在，会返回元素类型的默认值。

```go
var userInfo = map[string]string
name,ok := userInfo["name"]
if(!ok){
    fmt.Println("元素不存在")
}
```

#### 遍历

> 使用 range 遍历字典

```go
userInfo := map[string]string{
    "name":"张三"
}
for key,val in range userInfo{
    fmt.Pringln(key,val)
}
```

#### 内部实现

> 映射是一个集合，可以使用类似处理数组和切片的方式迭代映射中的元素。
> 但映射是无序的集合，意味着没有办法预测键值对被返回的顺序。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671430908220-71632850-de37-41c9-b720-d63896f6a164.png#averageHue=%23e5e5e5&clientId=u6fc25da0-5b53-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=513&id=u06b6f0c3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1026&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=264867&status=done&style=none&taskId=u0a1af87d-241e-4ab1-9289-25bc7e74f00&title=&width=720)
映射的散列表包含一组桶。在存储、删除或者查找键值对的时候，所有操作都要先选择一个桶。把操作映射时指定的键传给映射的散列函数，就能选中对应的桶。这个散列函数的目的是生成一个索引，这个索引最终将键值对分布到所有可用的桶里
随着映射存储的增加，索引分布越均匀，访问键值对的速度就越快。如果你在映射里存储了 10 000 个元素，你不希望每次查找都要访问 10 000 个键值对才能找到需要的元素，你希望查找键值对的次数越少越好。对于有 10 000 个元素的映射，每次查找只需要查找 8 个键值对才是一个分布得比较好的映射。映射通过合理数量的桶来平衡键值对的分布。
Go 语言的映射生成散列键的过程比图 4-25 展示的过程要稍微长一些，不过大体过程是类似的。在我们的例子里，键是字符串，代表颜色。这些字符串会转换为一个数值（散列值）。这个数值落在映射已有桶的序号范围内表示一个可以用于存储的桶的序号。之后，这个数值就被用于选择桶，用于存储或者查找指定的键值对。对 Go 语言的映射来说，生成的散列键的一部分，具体来说是低位（LOB），被用来选择桶。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671520332026-63d074bb-f055-4dae-b78f-d792cfa067fe.png#averageHue=%23e4e4e4&clientId=u25ca1dea-7cb8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=220&id=ud4d4dd9e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=440&originWidth=1036&originalType=binary∶=1&rotation=0&showTitle=false&size=229531&status=done&style=none&taskId=u62db486a-1b03-42ac-9ea1-7f1b6d6b2f6&title=&width=518)

### 结构体（值类型）

> 结构体类型可以封装字段列表，使之组成一个整体。
> 结构类型可以用来描述一组数据值，这组值的本质即可以是原始的，也可以是非原始的。
> 一般用来定义数据集。
> 大小写决定对外是否可见。

#### 声明

```go
type <结构体名称> struct{
    <key> <type>
}
type person struct {
    name string
}

//嵌套
type userList struct{
    user *user
}

// 实例化
var p person
或
var p = person{}
或
p := person{}

// 初始化
var p = person{
    name:"张三"
}
// 按照顺序初始化
var p = person{"李四"}
```

### 方法（引用类型）

> 结构体的方法并不是在结构体内部定义的，而是在结构体外部以函数形式定义。
> 方法和函数区别是：方法在函数名前有一个接受参数。该参数是方法所属结构体的实例。
> 方法接受的参数可以是值也可以是指针

```go
type user struct{
    name string
    age int
}
func (u user) getUserInfo(){
    fmt.Println(u.name)
}
func (u *user) updateUserAge(age int){
    u.age = age
}

var u = user{
    name:"zs",
    age:19
}
u.getUserInfo()
u.updateUserAge(10)
```

##### 匿名类型的方法提升

> 当一个匿名类型被嵌入到结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型继承了这些方法。即这个机制提供了一种简单的方式来模拟经典面向对象语言中的子类和继承相关的效果。

###### 匿名类型的方法调用

> 当嵌入一个匿名类型时，这个类型的方法就变成了外部类型的方法，但是当它的方法被调用时，方法的接收器是内部类型（嵌入的匿名类型），而非外部类型。

```go
type People struct {
    Age    int
    gender string
    Name      string
}
type OtherPeople struct {
    People
}

func (p People) PeInfo() {
    fmt.Println("People ", p.Name, ":", p.Age, "岁, 性别:", p.gender)
}

OtherPeople.People.PeInfo()
//可以通过类型名称来访问内部类型的字段和方法。然而，这些字段和方法也同样被提升到了外部类型，可以直接访问
OtherPeople.PeInfo()
```

###### 提升规则

> 给定一个结构体 S 和一个类型 T，Go 语言中匿名嵌入类型的方法集提升规则如下：
>
> 1. 如果 S 包含匿名字段 T，则 S 和*S 的方法集都包含具有接收器 T 的提升方法。*S 的方法集还包含具有接收器\*T 的提升方法。
> 2. 如果 S 包含匿名字段*T，则 S 和*S 的方法集都包含具有接收器 T 或\*T 的提升方法。

### 接口（引用类型）

> 多态是指代码可以根据类型的具体实现采取不同行为的能力。
> 接口就是一组抽象方法的集合，它必须由其他非 interface 类型实现，而不能自我实现。
> 如果一个类型实现了某个接口，所有使用这个接口的地方，都可以支持这种类型的值。
> 与结构体类似，也是使用 type 关键字定义。
> 如果接口的所有方法在某个类型方法集中被实现，则认为该类型实现了这个接口。
> 这种灵活性使 Go 语言不用像 Java 语言那样需要显式实现接口。一旦类型不需要实现某个接口，甚至可以不改动任何代码。

#### 声明

> 在接口中声明方法不需要 func 关键字，也没有实例对象接收参数，只需要提供方法名称、参数、返回值等特征描述。

```go

//声明格式
type <接口名称> interface{
    <方法列表>
}

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

```

#### 实现

> 类型不用显式声明实现接口，只需要实现接口所有方法，这样的隐式实现解耦了实现接口的包和定义接口的包。

```go
type Locker interface{
    Lock() uint16
    Unlock(id uint16)
}

type interLocker struct{
    lockID uint16
}
//实现接口

func (l *interLocker) Lock() uint16 {
    l.lockID++
    return l.lockID
}

func (l *interLocker) Unlock(id uint16){
    if id!=l.lockID {
        fmt.Println("持有锁不正确")
    }
    fmt.Println("系统已解锁")
}
```

#### 接口嵌入

> 一个接口可以包含一个或多个其他的接口，但是在接口内不能嵌入结构体，也不能嵌入接口自身，否则编译会出错。

```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}

```

#### 接口的继承

> 当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）时，这个类型就可以使用（另一个类型）所有的接口方法。

```go
type ReaderWriter struct {
    io.Reader
    io.Writer
}

```

#### 类型断言

> 可以把实现了某个接口的类型值保存在接口变量中，但反过来某个接口变量属于哪个类型呢？如何检测接口变量的类型呢？这就是类型断言（Type Assertion）的作用。
> 类型断言可能是无效的，虽然编译器会尽力检查转换是否有效，但是它不可能预见所有的情况。

```go
value, ok := varI.(T)       // 类型断言

```

接口类型向普通类型转换有两种方式：comma-ok 断言和 type-switch 测试。

##### 通过 type-switch 做类型判断

```go
// type-switch做类型判断
var value interface{}
switch str := value.(type) {
	case string:
    	fmt.Println("value类型断言结果为string:", str)
  	case Stringer:
    	fmt.Println("value类型断言结果为Stringer:", str)
    default:
    	fmt.Println("value类型不在上述类型之中")
}

```

##### comma-ok 类型断言

```go
// comma-ok断言
var varI I varI = T("Tstring")
if v, ok := varI.(T); ok {
 // 类型断言
    fmt.Println("varI类型断言结果为: ", v) // varI已经转为T类型
    varI.f()
 }
```

### 通道（引用类型）

> Go 语言通过通信来共享内存，而不是共享内存来通信。
> 通道是协程通信的通道，协程之间可以通过它收发消息。
> 通道是进程内的通信方式，因此通过通道传递对象的过程和调用函数时的参数传递行为比较一致，例如也可以传递指针等。
> 通道是类型相关的，一个通道只能传递（发送或接收）一种类型的值，这个类型需要在声明通道时指定。
> 默认情况下，通道发送消息和接收消息都是阻塞的（叫作无缓冲的通道）。

#### 创建

```go
var channel chan int = make(chan int)
// 或
channel := make(chan int)

//发送
// 定义接收的channel
receive_only := make (<-chan int)
// 定义发送的channel
send_only := make (chan<- int)
// 可同时发送接收
send_receive := make (chan int)

```

#### 无缓冲通道

> 无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。
> 这种类型的通道要求发送 goroutine 和接收 goroutine 同时准备好，才能完成发送和接收操作。
> 如果两个 goroutine 没有同时准备好，通道会导致先执行发送或接收操作的 goroutine 阻塞等待。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671525858918-68aec6e7-6741-45d1-a84c-88172c6badb6.png#averageHue=%23fdfdf9&clientId=u412f3f0c-65b6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=435&id=ube929efa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=869&originWidth=979&originalType=binary∶=1&rotation=0&showTitle=false&size=337937&status=done&style=none&taskId=u100a4e52-16a2-4326-a5ce-93c9b607447&title=&width=489.5)

#### 有缓冲通道

> 有缓冲的通道（buffered channel）是一种在被接收前能存储一个或者多个值的通道。
> 这种类型的通道并不强制要求 goroutine 之间必须同时完成发送和接收。
> 通道会阻塞发送和接收动作的条件也会不同。
> 只有在通道中没有要接收的值时，接收动作才会阻塞。
> 只有在通道没有可用缓冲区容纳被发送的值时，发送动作才会阻塞。
> 这导致有缓冲的通道和无缓冲的通道之间的一个很大的不同：无缓冲的通道保证进行发送和接收的 goroutine 会在同一时间进行数据交换；有缓冲的通道没有这种保证。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671525966229-3c883df0-d605-4192-ab74-5efb30d95567.png#averageHue=%23fcfcf8&clientId=u412f3f0c-65b6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=278&id=u49b3dc3e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=556&originWidth=930&originalType=binary∶=1&rotation=0&showTitle=false&size=241067&status=done&style=none&taskId=u2b599578-2e43-4cab-9437-925dd4c1018&title=&width=465)
