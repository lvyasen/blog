---
title: Go 并发与并行
urlname: seaztczxvmxxeiat
date: '2022-12-30 09:13:50 +0800'
tags: []
categories: []
---

tags: [并发&并行]
categories: [Go]
cover: ''

---

## 进程&线程

### 进程

> 当运行一个应用程序（如一个 IDE 或者编辑器）的时候，操作系统会为这个应用程序启动一个进程。
> 可以将这个进程看作一个包含了应用程序在运行中需要用到和维护的各种资源的容器。
> **进程是操作系统分配资源的单位，线程是操作系统调度的基本单位，线程之间共享进程资源**。
> 一个包含所有可能分配的常用资源的进程。这些资源包括但不限于内存地址空间、文件和设备的句柄以及线程。
> 一个进程至少包含一个线程，每个进程的初始线程被称作主线程。
> 操作系统将线程调度到某个处理器上运行，这个处理器并不一定是进程所在的处理器。
> 狭义定义：进程是正在运行的程序的实例（an instance of a computer program that is being executed）。
> 广义定义：进程是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是[操作系统](https://baike.baidu.com/item/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/192?fromModule=lemma_inlink)动态执行的[基本单元](https://baike.baidu.com/item/%E5%9F%BA%E6%9C%AC%E5%8D%95%E5%85%83?fromModule=lemma_inlink)，在传统的[操作系统](https://baike.baidu.com/item/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F?fromModule=lemma_inlink)中，进程既是基本的[分配单元](https://baike.baidu.com/item/%E5%88%86%E9%85%8D%E5%8D%95%E5%85%83?fromModule=lemma_inlink)，也是基本的执行单元。
> 进程的概念主要有两点：
>
> - 第一，进程是一个实体。每一个进程都有它自己的地址空间，一般情况下，包括[文本](https://baike.baidu.com/item/%E6%96%87%E6%9C%AC?fromModule=lemma_inlink)区域（text region）、数据区域（data region）和[堆栈](https://baike.baidu.com/item/%E5%A0%86%E6%A0%88?fromModule=lemma_inlink)（stack region）。文本区域存储处理器执行的代码；数据区域存储变量和进程执行期间使用的动态分配的内存；堆栈区域存储着活动过程调用的指令和本地变量。
> - 第二，进程是一个“执行中的程序”。程序是一个没有生命的实体，只有[处理](https://baike.baidu.com/item/%E5%A4%84%E7%90%86?fromModule=lemma_inlink)器赋予程序生命时（操作系统执行之），它才能成为一个活动的实体，我们称其为[进程](https://baike.baidu.com/item/%E8%BF%9B%E7%A8%8B?fromModule=lemma_inlink)。 [3]

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672362977832-d7698575-9ad9-49fe-973b-534bcad6fddf.png#averageHue=%23f1f1f1&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=322&id=u8e7c47c3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=644&originWidth=1102&originalType=binary∶=1&rotation=0&showTitle=false&size=194673&status=done&style=none&taskId=uee28419e-446b-47cb-abf7-7c6cd2c8647&title=&width=551)

### 线程

> **线程是操作系统调度的基本单位，线程之间共享进程资源**。
> 线程运行的本质其实就是函数的执行。
> 一个线程是一个执行空间，这个空间会被操作系统调度来运行函数中所写的代码。
> 操作系统会在物理处理器上调度线程来运行，而 Go 语言的运行时会在逻辑处理器上调度 goroutine 来运行。
> 每个逻辑处理器都分别绑定到单个操作系统线程。
> Go 语言的运行时默认会为每个可用的物理处理器分配一个逻辑处理器。
> 既然线程运行的本质就是函数的执行，那么函数执行都有哪些信息呢？
> 函数运行时的信息保存在栈帧中，栈帧中保存了函数的返回值、调用其它函数的参数、该函数使用的局部变量以及该函数使用的寄存器信息，如图所示，假设函数 A 调用函数。
> CPU 执行指令的信息保存在一个叫做程序计数器的寄存器中，通过这个寄存器我们就知道接下来要执行哪一条指令。由于操作系统随时可以暂停线程的运行，因此我们保存以及恢复程序计数器中的值就能知道线程是从哪里暂停的以及该从哪里继续运行了。

> **线程**（英语：thread）是[操作系统](https://baike.baidu.com/item/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F?fromModule=lemma_inlink)能够进行运算[调度](https://baike.baidu.com/item/%E8%B0%83%E5%BA%A6?fromModule=lemma_inlink)的最小单位。它被包含在[进程](https://baike.baidu.com/item/%E8%BF%9B%E7%A8%8B?fromModule=lemma_inlink)之中，是[进程](https://baike.baidu.com/item/%E8%BF%9B%E7%A8%8B?fromModule=lemma_inlink)中的实际运作单位。一条线程指的是[进程](https://baike.baidu.com/item/%E8%BF%9B%E7%A8%8B?fromModule=lemma_inlink)中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。在 Unix System V 及[SunOS](https://baike.baidu.com/item/SunOS?fromModule=lemma_inlink)中也被称为轻量进程（lightweight processes），但轻量进程更多指内核线程（kernel thread），而把用户线程（user thread）称为线程。
> 线程是独立调度和分派的基本单位。线程可以为操作系统内核调度的内核线程，如[Win32](https://baike.baidu.com/item/Win32?fromModule=lemma_inlink)线程；由用户进程自行调度的用户线程，如[Linux](https://baike.baidu.com/item/Linux/27050?fromModule=lemma_inlink)平台的[POSIX](https://baike.baidu.com/item/POSIX/3792413?fromModule=lemma_inlink) Thread；或者由[内核](https://baike.baidu.com/item/%E5%86%85%E6%A0%B8?fromModule=lemma_inlink)与用户进程，如[Windows 7](https://baike.baidu.com/item/Windows%207?fromModule=lemma_inlink)的线程，进行混合调度。
> 同一进程中的多条线程将共享该进程中的全部系统资源，如虚拟地址空间，[文件描述符](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6?fromModule=lemma_inlink)和[信号处理](https://baike.baidu.com/item/%E4%BF%A1%E5%8F%B7%E5%A4%84%E7%90%86?fromModule=lemma_inlink)等等。但同一进程中的多个线程有各自的[调用栈](https://baike.baidu.com/item/%E8%B0%83%E7%94%A8%E6%A0%88?fromModule=lemma_inlink)（call stack），自己的寄存器环境（register context），自己的线程本地存储（thread-local storage）。
> 一个进程可以有很多线程，每条线程并行执行不同的任务。
> 在多核或多[CPU](https://baike.baidu.com/item/CPU/120556?fromModule=lemma_inlink)，或支持 Hyper-threading 的 CPU 上使用多线程程序设计的好处是显而易见，即提高了程序的执行吞吐率。在单 CPU 单核的计算机上，使用[多线程技术](https://baike.baidu.com/item/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%8A%80%E6%9C%AF/5764231?fromModule=lemma_inlink)，也可以把进程中负责 I/O 处理、人机交互而常被阻塞的部分与密集计算的部分分开来执行，编写专门的 workhorse 线程执行密集计算，从而提高了程序的执行效率。

#### 栈区

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672363617413-4a375f03-d9ca-4e90-92d2-d14722e9b8c8.png#averageHue=%23f9d1b8&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=216&id=u284e02d1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=432&originWidth=640&originalType=binary∶=1&rotation=0&showTitle=false&size=82754&status=done&style=none&taskId=uc77cb9b0-6d46-413d-9215-5f83d57921b&title=&width=320)

> 由于线程运行的本质就是函数运行，函数运行时信息是保存在栈帧中的，因此每个线程都有自己独立的、私有的栈区。
> 同时函数运行时需要额外的寄存器来保存一些信息，像部分局部变量之类。这些寄存器也是线程私有的，一个线程不可能访问到另一个线程的这类寄存器信息。
> 操作系统调度线程需要随时中断线程的运行并且需要线程被暂停后可以继续运行，操作系统之所以能实现这一点，依靠的就是线程上下文信息。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672363740103-056cd50b-01c7-4290-bbd2-404582e84e1b.png#averageHue=%23fefefc&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=293&id=ubf52b971&margin=%5Bobject%20Object%5D&name=image.png&originHeight=585&originWidth=640&originalType=binary∶=1&rotation=0&showTitle=false&size=92943&status=done&style=none&taskId=u8b7578ed-f932-461c-8977-9b776bb86e0&title=&width=320)

> 除此之外，剩下的都是线程间共享资源。
> 这其实就是进程地址空间的样子，也就是说线程共享进程地址空间中除线程上下文信息中的所有内容，意思就是说线程可以直接读取这些内容。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672363829273-b34d48c4-b603-4f73-b3d2-5278a5d12f87.png#averageHue=%238fb7f5&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=252&id=udb15e439&margin=%5Bobject%20Object%5D&name=image.png&originHeight=504&originWidth=339&originalType=binary∶=1&rotation=0&showTitle=false&size=44589&status=done&style=none&taskId=u8864434f-7b6a-47d6-aca2-d8eadc3db0a&title=&width=169.5)

#### 代码区

> 这里保存着代码，更准确的说是**编译后的可执行的机器指令。**
> 这些指令从哪来的呢？是从可执行文件加载到内存中来的，可执行程序中的代码区就是用来初始化进程地址空间中的代码区的。
> 线程之间共享代码区，这就意味着程序中的任何一个函数都可以放到线程中去执行，不存在某个函数只能被特定线程执行的情况。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672364233869-a7b9a1c4-8c39-4cfb-87a4-641e92cf481a.png#averageHue=%23fafaf8&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=108&id=u6bcdb8d3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=216&originWidth=640&originalType=binary∶=1&rotation=0&showTitle=false&size=57752&status=done&style=none&taskId=u8a330290-feae-490b-8de6-c92f5095b29&title=&width=320)

#### 数据区

> 进程地址空间中的数据区，这里存放的就是所谓的全局变量。

#### 堆区

> 堆区也是线程共享的属于进程的资源。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672364360342-5ff59163-20c7-4a5f-a528-85d2fb6bc537.png#averageHue=%23f6f7f4&clientId=udafd4176-d74f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=180&id=u893b4252&margin=%5Bobject%20Object%5D&name=image.png&originHeight=360&originWidth=640&originalType=binary∶=1&rotation=0&showTitle=false&size=68821&status=done&style=none&taskId=u1e64a3dd-7ec8-47d3-b822-87553317f08&title=&width=320)

## goroutine（协程）

> 使用 `go`关键字启动。
> `goroutine`会以很小的栈启动（2KB 或 4KB） ，当栈空间不足时 `goroutine`会动态的伸缩栈大小（最大值可能是 1GB），因为启动代价很小，所以能启动成千上万个。

```go
package goroutine_test


import (
	"runtime"
	"sync"
	"testing"
)

var wg sync.WaitGroup
func TestSimpleGoroutine( t *testing.T)  {
	//声明使用的逻辑处理器数量
	runtime.GOMAXPROCS(1)
	t.Log("开始")
	//往 WaitGroup 中添加计数,阻塞等待,如果计数器小于 0 ,方法 panic,该方法需要再 Wait 方法前.
	wg.Add(10)
	for i := 1; i < 11; i++ {
		//使用 go 关键字创建协程
		i := i
		go func() {
			//减少 WaitGroup 计数器
			defer wg.Done()
			t.Logf("这是第%d个 goroutine",i)
			//这里打印英文 26 个字母
			var chatStart='a'
			if i%2==0 {
				chatStart='A'
			}
			for char := chatStart; char < chatStart+26; char++ {
				t.Logf("%c",char)
			}
		}()
	}
	wg.Wait()
	t.Log("程序执行完毕")
}

```

### 竞争状态

> 如果两个或者多个 goroutine 在没有互相同步的情况下，访问某个共享的资源，并试图同时读和写这个资源，就处于相互竞争的状态，这种情况被称作竞争状态（race candition）。
> 竞争状态的存在是让并发程序变得复杂的地方，十分容易引起潜在问题。
> 对一个共享资源的读和写操作必须是原子化的，换句话说，同一时刻只能有一个 goroutine 对共享资源进行读和写操作。

#### 竞态的例子

> 每个 goroutine 都会覆盖另一个 goroutine 的工作。这种覆盖发生在 goroutine 切换的时候。每个 goroutine 创造一个 counter 变量的副本，之后就切换到另一个 goroutine。
> 当这个 goroutine 再次运行的时候，counter 变量的值已经改变了，但是 goroutine 并没有更新自己的那个副本的值，而是继续使用这个副本的值，用这个值递增，并存回 counter 变量，结果覆盖了另一个 goroutine 完成的工作。

```go
package goroutine_test


import (
	"runtime"
	"sync"
	"testing"
)

var (
	wg sync.WaitGroup
	counter int
)

func TestRaceStatus(t *testing.T) {
	t.Log("程序开始执行")
	// runtime.GOMAXPROCS(1) //当使用一个逻辑处理器时不会出现竞争状态
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for i := 0; i < 10; i++ {
				val := counter
				t.Logf("当前计数器为%d",counter)
				//Gosched使当前go程放弃处理器，以让其它go程运行。它不会挂起当前go程，因此当前go程未来会恢复执行。
				//当再次进来时 counter 的值已经被修改,但是副本没有被修改所以相当于原先已经修改的值会被覆盖掉.
				//每个goroutine都会覆盖另一个goroutine的工作。这种覆盖发生在goroutine切换的时候。每个goroutine创造了一个counter
                //变量的副本，之后就切换到另一个goroutine。当这个goroutine再次运行的时候，counter
                //变量的值已经改变了，但是goroutine并没有更新自己的那个副本的值，而是继续使用这个副本的值，用这个值递增，并存回counter
                //变量，结果覆盖了另一个goroutine完成的工作。
                runtime.Gosched()
				val++
				counter = val
			}
		}()
	}
	wg.Wait()
	t.Logf("程序执行完毕,计数器为%d",counter)
}

```

### 检测竞争状态

> `go build -race`

### 锁住竞争状态

#### 原子函数

> 使用 `sync/atomic `中的 `atomic `函数锁住竞争状态。
> 这个函数会同步整型值的加法，方法是强制同一时刻只能有一个 goroutine 运行并完成这个加法操作。

```go
package goroutine_test

import (
	"runtime"
	"sync"
	"sync/atomic"
	"testing"
)

var (
	wg sync.WaitGroup
	counter int64
)

func TestAtom(t *testing.T) {

	//runtime.GOMAXPROCS(1)
	t.Log("程序开始")
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for k := 0; k < 10; k++ {
				//使用 atomic 函数锁住竞争状态
				atomic.AddInt64(&counter,1)
				//从线程中退出,放回到队列
				runtime.Gosched()
			}
		}()
	}
	wg.Wait()
	t.Logf("程序运行结束,counter=%d",counter)
}

```

#### 互斥锁（mutex）

> 互斥锁这个名字来自互斥（mutual exclusion）的概念。
> 互斥锁用于在代码上创建一个临界区，保证同一时间只有一个 goroutine 可以执行这个临界区代码。

```go
package goroutine_test

import (
	"runtime"
	"sync"
	"sync/atomic"
	"testing"
)

var (
	wg sync.WaitGroup
	counter int64
	mutex sync.Mutex
)
//使用互斥锁处理竞争状态
func TestMutex(t *testing.T) {
	t.Log("程序运行")
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for k := 0; k < 10; k++ {
				//使用临界区
				mutex.Lock()
				{
                    //这里代码被保护起来
					val := counter
					runtime.Gosched()
					val++
					counter = val
				}
				mutex.Unlock()
				//释放锁,允许其他正在等待的 goroutine 进入临界区
			}
		}()
	}
	wg.Wait()
	t.Logf("程序结束,counter=%d",counter)
}
```

#### 通道（channel）

> 原子函数和互斥锁都能工作，但是依靠它们都不会让编写并发程序变得更简单，更不容易出错，或者更有趣。
> 在 Go 语言里，你不仅可以使用原子函数和互斥锁来保证对共享资源的安全访问以及消除竞争状态，还可以使用通道，通过发送和接收需要共享的资源，在 goroutine 之间做同步。
> 当一个资源需要在 goroutine 之间共享时，通道在 goroutine 之间架起了一个管道，并提供了确保同步交换数据的机制。
> 声明通道时，需要指定将要被共享的数据的类型。可以通过通道共享内置类型、命名类型、结构类型和引用类型的值或者指针。
> 在计算机科学中，CSP（Communicating Sequential Processes，通信顺序进程）是用于描述并发系统中交互模式的形式化语言，其通过通道传递消息。
> 通道分为两种：
>
> - 有缓冲通道。
> - 无缓冲通道
>
> 向通道发送值或者指针需要用到<-操作符，

```go
//无缓冲通道
unbuffered := make(chan int)
//有缓冲通道
buffered := make(chan string,10)

channel := make(chan int,10)
//向通道发送数据
channel<-99
//从通道取出数据
val := <-channel
//关闭通道
close(channel)
```

##### 无缓冲通道

> 无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。
> 这种类型的通道要求发送 goroutine 和接收 goroutine 同时准备好，才能完成发送和接收操作。
> 如果两个 goroutine 没有同时准备好，通道会导致先执行发送或接收操作的 goroutine 阻塞等待。
> 这种对通道进行发送和接收的交互行为本身就是同步的。
> 其中任意一个操作都无法离开另一个操作单独存在。

```go
package goroutine_test

import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
	"testing"
)

var (
	wg sync.WaitGroup
	mutex sync.Mutex
	counter int64
	counter2 = make(chan int64)
)

func incrCounter(c chan int64)  {
	var newVal  int64
	val := <-c
	if val!=100 {
		newVal = val+1
		fmt.Println("counter=",newVal)
		go incrCounter(c)
	}
	if val>100 {
		wg.Done()
		return
	}
	c<-newVal
}
//使用无缓冲通道,必须发送者和接受者同时出现
func TestUnbufferedChannel(t *testing.T) {
	t.Log("程序开始")
	channel := make(chan int64)

	wg.Add(1)
	go incrCounter(channel)
	channel<-1
	defer close(channel)
	wg.Wait()
	//t.Logf("程序结束,count=%d",counter)
}
```

##### 有缓冲通道

```go
package goroutine_test

import (
	"fmt"
	"math/rand"
	"runtime"
	"sync"
	"sync/atomic"
	"testing"
	"time"
)

var (
	wg sync.WaitGroup
	mutex sync.Mutex
	counter int64
	counter2 = make(chan int64)
)

func handleTask(tasks chan string,worker int)  {
	defer wg.Done()
	for  {
		//等待分配工作
		task,ok:=<-tasks
		if !ok {
			fmt.Println("通道被清空")
			return
		}
		fmt.Printf("机器人--%d 开始做--%s工作",worker,task)
		sleep := rand.Int63n(100)
		time.Sleep(time.Duration(sleep)*time.Millisecond)
		fmt.Println("工作完成")
	}
}
//使用有缓冲通道
func TestBufferedChannel(t *testing.T) {
	tasks := make(chan string,10)
	//启动四个 goroutine 处理任务
	wg.Add(4)
	for i := 0; i < 4; i++ {
		go handleTask(tasks,i)
	}
	for post := 0; post < 10; post++ {
		tasks<-fmt.Sprintf("任务:%d",post)
	}
	close(tasks)
	wg.Wait()
}
```

### 协程

> 是一种轻量级的线程。协程就是用户态的线程。

#### 优点

- 节省 CPU，避免系统内核级的线程频繁切换，造成的 CPU 资源浪费。好钢用在刀刃上。而协程是用户态的线程，用户可以自行控制协程的创建于销毁，极大程度避免了系统级线程上下文切换造成的资源浪费。
- 节约内存，在 64 位的 Linux 中，一个线程需要分配 8MB 栈内存和 64MB 堆内存，系统内存的制约导致我们无法开启更多线程实现高并发。而在协程编程模式下，可以轻松有十几万协程，这是线程无法比拟的。
- 稳定性，前面提到线程之间通过内存来共享数据，这也导致了一个问题，任何一个线程出错时，进程中的所有线程都会跟着一起崩溃。
- 开发效率，使用协程在开发程序之中，可以很方便的将一些耗时的 IO 操作异步化，例如写文件、耗时 IO 请求等。

#### 协程与线程区别

> 一般每个系统级线程都会有一个固定大小的栈（一般为 2MB），这个栈用来保存函数递归调用时的参数和局部变量。固定大小的栈会面临两问题：
>
> - 如果线程需要的空间很小这将会是很大的浪费。
> - 如果线程需要的空间很大又面临栈溢出的风险。
>
> 但是 `goroutine`会以很小的栈启动（2KB 或 4KB） ，当栈空间不足时 `goroutine`会动态的伸缩栈大小（最大值可能是 1GB），因为启动代价很小，所以能启动成千上万个。

#### 上下文的切换速度

> 协程的速度要快于线程，其原因在于协程切换不用经过操作系统用户态与内核态的切换，并且 Go 语言中的协程切换只需要保留极少的状态和寄存器变量值（SP/BP/PC），而线程切换会保留额外的寄存器变量值（例如浮点寄存器）。上下文切换的速度受到诸多因素的影响，这里列出一些值得参考的量化指标：线程切换的速度大约为 1 ～ 2 微秒，Go 语言中协程切换的速度比它快数倍，为 0.2 微秒左右[3]。

#### 调度方式

> 协程是用户态的。协程的管理依赖 Go 语言运行时的调度器。同时，Go 语言中的协程是从属于某一个线程的，协程与线程的对应关系为 M：N，即多对多。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672366460340-33786a2b-7d00-4794-b4b0-1090242f16fd.png#averageHue=%23fcfcf9&clientId=uc2a83074-594a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=299&id=u1ce4bb51&margin=%5Bobject%20Object%5D&name=image.png&originHeight=598&originWidth=1127&originalType=binary∶=1&rotation=0&showTitle=false&size=166958&status=done&style=none&taskId=u4469be26-0a36-49a2-8658-d1617af37b0&title=&width=563.5)

#### 调度策略

> 线程的调度在大部分时间是抢占式的，操作系统调度器为了均衡每个线程的执行周期，会定时发出中断信号强制执行线程上下文切换。
> 而 Go 语言中的协程在一般情况下是协作式调度的，当一个协程处理完自己的任务后，可以主动将执行权限让渡给其他协程。
> 这意味着协程可以更好地在规定时间内完成自己的工作，而不会轻易被抢占。
> 当一个协程运行了过长时间时，Go 语言调度器才会强制抢占其执行（详见第 15 章）。

## 并发&并行

> 并发并不意味着同一时刻所有任务都在执行，而是在一个时间段内，所有的任务都能执行完毕。
> 并发（concurrency）指的是程序的逻辑结构。如程序代码结构中的某些函数逻辑上可以同时运行，但物理上未必会同时运行。
>
> 并行是让不同的代码片段同时在不同的物理处理器上执行。
> 并行是指程序的运行状态。在物理层面也就是使用了不同 CPU 同时在执行不同或者相同的任务
> 并行的关键是同时做很多事情，而并发是指同时管理很多事情，这些事情可能只做了一半就被暂停去做别的事情了。

> 在单核处理器中，任意一个时刻只能执行一个具体的线程，而在一个时间段内，线程可能通过上下文切换交替执行。
> 多核处理器是真正的并行执行，因为在任意时刻，可以同时有多个线程在执行。
> 由于 Go 语言中的协程依托于线程，所以即便处理器运行的是同一个线程，在线程内 Go 语言调度器也会切换多个协程执行，这时协程是并发的。
> 如果多个协程被分配给了不同的线程，而这些线程同时被不同的 CPU 核心处理，那么这些协程就是并行处理的。因此在多核处理场景下，Go 语言的协程是并发与并行同时存在的。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672366712073-cf6bc81a-713f-49a4-9c71-534de669c6cf.png#averageHue=%23f9f8cf&clientId=uc2a83074-594a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=364&id=ude602b46&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=841&originalType=binary∶=1&rotation=0&showTitle=false&size=124987&status=done&style=none&taskId=u77d8144d-f9c8-473f-b7bd-b48b305ec08&title=&width=420.5)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672367029403-f15aca9c-13a6-44ff-aedd-3ed47d65c118.png#averageHue=%23f4f4f4&clientId=uc2a83074-594a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=268&id=ue89fad8a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=535&originWidth=994&originalType=binary∶=1&rotation=0&showTitle=false&size=175273&status=done&style=none&taskId=uf18453c3-1ee9-4d2f-8e2f-596d7f24083&title=&width=497)
