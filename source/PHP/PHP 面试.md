# 基础

**1： echo 、print()、print_r()、sprintf()、vardump 区别？**

> 1. echo 是语句没有返回值，可输出多个变量，不能输出数组和对象只能打印 int 和 string。
> 2. print 是语句不是函数，有返回值，只能输出一个变量，不需要圆括号，不能输出数组和对象。
> 3. print_r() 是函数，可以打印复合类型。
> 4. printf() 是函数，把文字格式化后输出。
> 5. sprintf() 是函数，把文字格式化后写入一个变量而不是输出。
> 6. var_dump() 是函数，输出变量的内容、类型或字符串的内容、类型、长度。用来调试。

**2：php 获取图像尺寸大小的方法是什么？**

> 1. getimagesize() 获取图片尺寸。
> 2. imagesx() 获取图片宽度。
> 3. imagesy() 获取图片高度。

**3：php 中 PEAR 是什么？**

> PEAR（PHP Extension and Application Repository）PHP 扩展与应用库是一个代码仓库。

**4： php 错误类型有哪些？**

> 1. notice（提示）：比较正常的信息而非错误。
> 2. warning（警告）：一般是一些常规错误，会将警告信息展示给用户，但是不会影响代码输出。
> 3. error（错误）：这是比较严重的错误，会影响整个代码运行。

**5：你知道有哪些排序算法？**

> 1. **冒泡排序**
> 2. **快速排序**
> 3. **选择排序**
> 4. **插入排序**

```php

$arr = [12,13,2,21,6,32,78,36,76,9];
//冒泡排序 比较相邻的元素，如果第一个比第二个大，就交换他们。
function bubbleSort(){
  $len = count($arr);
  //该层循环控制需要冒泡的次数
  for($i=1;$i<$len;$i++){
    //该层循环用来控制每轮冒出一个泡
    for($k=0;$k<$len-$i;$k++){
      $tmp = $arr[$k+1];
      $arr[$k+1] = $arr[$k];
      $arr[$k] = $tmp;
    }
  }
  return $arr;
}
// 选择排序：首先在末尾排序中找最小元素，并将其存放到序列起始位置，然后在从剩余未排序数组中寻找最小元素然后放到排列序列的末尾。
function selectSort($arr){
  $len = count($arr);
  for($i=0;$i<$len-1;$i++){
    $p = $i;
    for($k=$i+1;$k<$len;$k++){
      if($arr[$p]>$arr[$k]){
        $p = $k;
      }
    }
    if($p != $i){
      $tmp = $arr[$p];
      $arr[$p] = $arr[$i];
      $arr[$i] = $tmp;
    }
  }
  return $arr;
}

// 插入排序 通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。
//1、从第一个元素开始，该元素可以认为已经被排序
//2、取出下一个元素，在已经排序的元素序列中从后向前扫描
//3、如果该元素（已排序）大于新元素，将该元素移到下一位置
//4、重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
//5、将新元素插入到该位置中
//6、重复步骤2
function insert_sort($arr) {
	$len=count($arr);
	for ($i=1; $i<$len; $i++) {
		//获得当前需要比较的元素值。
		$tmp = $arr[$i];
		//内层循环控制 比较 并 插入
		for ($j=$i-1; $j>=0; $j--) {
			//$arr[$i];//需要插入的元素; $arr[$j];//需要比较的元素
			if($tmp < $arr[$j]) {
				//发现插入的元素要小，交换位置
				//将后边的元素与前面的元素互换
				$arr[$j+1] = $arr[$j];
				//将前面的数设置为 当前需要交换的数
				$arr[$j] = $tmp;
			} else {
				//如果碰到不需要移动的元素
				//由于是已经排序好是数组，则前面的就不需要再次比较了。
				break;
			}
		}
	}
	//将这个元素 插入到已经排序好的序列内。
	//返回
	return $arr;
}

//快速排序
//1、从数列中挑出一个元素，称为 “基准”（pivot），
//2、重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
//3、递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。
function quick_sort($arr) {
	//判断参数是否是一个数组
	if(!is_array($arr)) return false;
	//递归出口:数组长度为1，直接返回数组
	$length = count($arr);
	if($length<=1) return $arr;
	//数组元素有多个,则定义两个空数组
	$left = $right = array();
	//使用for循环进行遍历，把第一个元素当做比较的对象
	for ($i=1; $i<$length; $i++) {
		//判断当前元素的大小
		if($arr[$i]<$arr[0]) {
			$left[]=$arr[$i];
		} else {
			$right[]=$arr[$i];
		}
	}
	//递归调用
	$left=quick_sort($left);
	$right=quick_sort($right);
	//将所有的结果合并
	return array_merge($left,array($arr[0]),$right);
}

```

**6：nginx 和 apache 有什么区别？**

> 1. **apache 是同步多进程模型一个连接对应一个进程；nginx 是一步的多个连接对应一个进程。**
> 2. **apache 处理动态有优势；nginx 并发性能比较好，CPU 占用内存低。**
> 3. **apache rewrite 模块比 nginx 强大。**

**7：PHP 常见的运行模式有哪些？**

> 1. **CGI**
> 2. **Fast-CGI**
> 3. **cli**
> 4. **apache 模块的 DLL**

**8：PHP 运行模式原理是什么？**

> 1. **CGI ：Common Gateway Interface（通用网关接口），他把网页和 WEB 服务器中的执行程序相连，从 HTML 接受过来的指令传递给服务器执行，在把服务器执行程序的结果返回给 HTML；CGI 跨平台性能极佳。
>    CGI 方式在遇到连接请求先创建 CGI 子进程，激活一个 CGI 进程，然后处理请求，处理结束后结束这个子进程。这就是 fork-and-execute 模式。所以有多少连接就有多少 CGI 子进程，子进程反复加载是 CGI 性能低下的主要原因。当请求用户数量非常多时，会大量挤占系统资源如：内存、CPU 时间等造成性能低下。**
> 2. **FastCGI：是 CGI 的升级版，FastCGI 可以看成是一个常驻内存的 CGI ，他可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次。PHP 使用 PHP-FPM（FastCGI Process Manage）全称“PHP FastCGI 进程管理器”进行管理。
>    工作原理：
>    1：Web Server 启动时载入 FastCGI 进程管理器。
>    2：FastCGI 进程管理器自身初始化，启动多个 CGI 解释器（多个 php-cgi）并等待来自 Web Server 的连接。
>    3：当客户端请求到达 Web Server 时，FastCGI 进程管理器选择并连接到一个 CGI 解释器，WebServer 将 CGI 环境变量和标准输入发送到 FastCGI 子进程 php-cgi。
>    4：FastCGI 子进程处理完成后将标准输出和错误信息从同一连接返回 WebServer。当 FastCGI 子进程关闭连接时，请求便告知处理完成。FastCGI 进程接着等待并处理来自 FastCGI 继承管理器的下一个连接。当在 CGI 模式中，php-cgi 在此便退出。
>    CGI 每个 Web 请求 PHP 都必须重新解析 php.ini 、重新载入全部扩展并初始化全部数据结构。而使用 FastCGI ，所有操作只在进程启动时发生一次，一个额外好处，持续的数据库连接可以工作。**
> 3. **CLI 是 PHP 命令行模式**
> 4. **apache 模块 DLL：在模块化（DLL）中，PHP 是与 Web 服务器一同启动并运行（是 apache 在 CGI 基础上的一种扩展，加快 PHP 运行效率）**

**9：写一个函数遍历文件夹下所有文件和文件夹。**

```php
function getDirInfo($path){
  $handle = opendir($path);
  while (($content = readdir($handle))!==false) {
    $newDir = $path.DIRECTORY_SEPARATOR.$content;
    if ($content == '..' || $content == '.') {
      continue;
    }
    if (is_dir($newDir)) {
      echo PHP_EOL.'目录'.$newDir;
      getDirInfo($newDir);
    }else{
      echo PHP_EOL.'文件'.$path.$content;
    }
  }
}
getDirInfo('/Users/mac/project/learning/php');
```

# PHP 8

### JIT

**1：JIT 是什么？**

> 简介：Just In Time 是一种编译策略，将代码在运转时转换为依赖于体系结构的机器码，并即时执行。

**2：JIT 有什么好处？**

> 1. 目前很难通过常规手段提升 php 性能，JIT 基本上是性能提升的唯一手段。
> 2. JIT 带来的性能提升可以使 php 在 CPU 密集型发挥作用。
> 3. 可以使用 PHP 开发内置函数，而不用担心性能方面问题。

# 框架篇

1：**为什么使用 PHP 框架？**

> 1. PHP 框架令开发更快。
> 2. 框架使开发人员能够轻易地扩展系统。
> 3. 代码的维护更容易。
> 4. MVC 模式可以确保快速开发。
> 5. 框架更利于保护 WEB 的应用程序免受安全威胁。

2：**各种框架之间有什么区别？**

> 1. symfony 是一套可重复使用的 PHP 组件，它允许开发者创建可扩展、高性能的应用程序。Laravel 基于 symfony 建立。
>    - 提供一个 LTS 版本
>    - 带有负载功能
>    - 是目前最稳定的框架
>    - 是基于构件的框架，提供了丰富的模块化
>    - 具有一个出色的社区，提供丰富的学习资源
> 2. Laravel 被称为是“网络工程师的 PHP 框架”，它提供了一个出色的社区并赢得“最流行框架”的称号。
>    - 为设计者提供支持包管理
>    - 出色完成单元测试
>    - 提供丰富的包，用于扩展框架功能
>    - 具有一个出色的社区，提供丰富的学习资源
> 3. Yii 是一个安全，快速和高效的应用/网站开发框架。Yii 是在 2008 年由薛强创建的。
>    - 自带 Ajax 支持
>    - 十分适合用于开发实时应用程序，因为它的操作更快
>    - 是高度可扩展的
>    - 可准确无误地处理错误
>    - 适合用来创建平静的 Web 服务
>    - 具有一个出色的社区，提供丰富的学习资源

# Swoole 篇

**1：什么是 swoole ？**

> **答**：面向生产环境的 PHP 异步网络通信引擎，Swoole 它是一个网络应用的开发工具，它支持 Http、TCP、UDP、WebSocket。

**2：关于 phper 常用的全局变量（global）为什么在 onRequest 函数中不能使用？**

> **答：**因为 swoole 是多线程编程，global 是不能在多个进程间共享的

\*\*什么是异步、什么是回调？

\*\*
**3：为什么 onReceive 收到的数据这么大？**

> **答：**客户端发送的多次请求，服务端是可以一次性接收的。并不是客户端发送一次，服务端接收一次

**4：swoole 有什么特色？**

> **答：**网络通信 框架、异步、多线程。

**5：那为什么要使用 Swoole？**

> 1. 常驻内存，避免重复加载带来的性能损耗，提升海量性能。
> 2. 协程异步，提高对 I/O 密集型场景并发处理能力（如：微信开发、支付、登录等）。
> 3. 方便地开发 Http、WebSocket、TCP、UDP 等应用，可以与硬件通信。
> 4. PHP 高性能微服务架构成为现实。

\*\*
