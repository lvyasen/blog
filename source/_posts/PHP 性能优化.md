---
title: PHP 性能优化
urlname: khv3rxrq7fabgq11
date: '2022-11-18 08:22:08 +0800'
tags:
  - 性能优化
categories:
  - PHP
---

## **1 代码优化**

### 1 尽量静态化

如果一个方法能被静态，那就声明它为静态的，速度可提高 1/4，甚至我测试的时候，这个提高了近三倍。
当然了，这个测试方法需要在十万级以上次执行，效果才明显。
其实静态方法和非静态方法的效率主要区别在内存：静态方法在程序开始时生成内存，实例方法（非静态方法）在程序运行中生成内存，所以静态方法可以直接调用，实例方法要先成生实例再调用，静态速度很快，但是多了会占内存。
任何语言都是对内存和磁盘的操作，至于是否面向对象，只是软件层的问题，底层都是一样的，只是实现方法不同。静态内存是连续的，因为是在程序开始时就生成了，而实例方法申请的是离散的空间，所以当然没有静态方法快。
静态方法始终调用同一块内存，其缺点就是不能自动进行销毁，而实例化可以销毁。

### 2 echo 效率高于 print

因为 echo 没有返回值，print 返回一个整型。测试：
echo 0.000929 - 0.001255 s (平均 0.001092 seconds) print 0.000980 - 0.001396 seconds (平均 0.001188 seconds)
相差 8%左右，总体上 echo 是比较快的。
注意：echo 输出大字符串的时候，如果没有调整就会严重影响性能。打开 Apache 的 mod_deflate 进行压缩，或者打开 ob_start 将内容放进缓冲区可以改善性能问题。

### 3 循环最大次数

在循环之前设置循环的最大次数，而非在在循环中。

### 4 及时销毁变量

数组和对象在 PHP 中特别占内存的，这个由于 PHP 的底层的 zend 引擎引起的。一般来说，PHP 数组的内存利用率只有 1/10, 也就是说，一个在 C 语言里面 100M 内存的数组，在 PHP 里面就要 1G。
特别是在 PHP 作为后台服务器的系统中，经常会出现内存耗费太大的问题。

### 5 避免使用像**get、**set、\_\_autoload 等魔术方法

（仅供参考，有待商榷）
对于**开头的函数就命名为魔术函数，此类函数都在特定的条件下触发的。总得来说，有下面几个魔术函数**construct()、**destruct()、**get()、**set()、**unset()、**call()、**callStatic()、**sleep()、**wakeup()、**toString()、**set_state()、**clone()、**autoload()。
其实，如果**autoload() 不能高效的将类名与实际的磁盘文件（注意，这里指实际的磁盘文件，而不仅仅是文件名）对应起来，系统将不得不做大量的文件是否存在判断（需要在每个 include path 中包含的路径中去寻找），而判断文件是否存在需要做磁盘 I/O 操作，众所周知磁盘 I/O 操作的效率很低，因此这才是使得 autoload 机制效率降低的原因。
因此，我们在系统设计时，需要定义一套清晰的将类名与实际磁盘文件映射的机制。这个规则越简单越明确，autoload 机制的效率就越高。
结论：autoload 机制并不是天然的效率低下，只有滥用 autoload，设计不好的自动装载函数才会导致其效率的降低.
所以说尽量避免使用**autoload 魔术方法，有待商榷。

### 6 requiere_once() 和 include_once() 比较耗资源

这是因为 requiere_once()和 include_once()需要判断该文件是否被引用过，所以能不用尽量不用。常用 require/include 方法避免。[鸟哥在其博客中](https://link.zhihu.com/?target=http%3A//www.laruence.com/2012/09/12/2765.html)就多次声明尽量不要用 require_once 和 include_once。

### 7 在 include 和 require 中使用绝对路径

如果包含相对路径，PHP 会在 include_path 里面遍历查找文件。
用绝对路径就会避免此类问题，因此解析操作系统路径所需的时间会更少。

### 8 使用$\_SERVER['REQUSET_TIME']

如果你需要得到脚本执行的时间，$\_SERVER['REQUSET_TIME']优于 time()。
可以想象，一个是现成就可以直接用，一个还需要函数得出的结果。

### 9 用内置函数替代正则表达式

能用 PHP 内部字符串操作函数的情况下，尽量用他们，不要用正则表达式， 因为其效率高于正则。
没得说，正则最耗性能。
有没有你漏掉的好用的函数？例如：[strpbrk()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strpbrk.php)、[strncasecmp()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strncasecmp.php)、[strpos()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strpos.php)、[strrpos()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strrpos.php)、[stripos()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.stripos.php)、[strripos()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strripos.php)。
[strtr()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strtr.php)函数用于转换指定字符，如果需要转换的全是单个字符的时候，用字符串而不是数组：

<?php $addr = strtr($addr, "abcd", "efgh");       // good $addr = strtr($addr, array('a' => 'e', ));  // bad
效率提升：10 倍。

### 10 用strtr作字符替换
str_replace字符替换比正则替换preg_replace快，但strtr比str_replace又快1/4。
另外，不要做无谓的替换，即使没有替换，str_replace也会为其参数分配内存。很慢！
解决办法：用 strpos 先查找(非常快)，看是否需要替换，如果需要，再替换。
效率：如果需要替换，效率几乎相等，差别在 0.1% 左右。如果不需要替换：用 strpos 快 200%。

### 11 用字符串而不是数组作为参数
如果一个函数既能接受数组，又能接受简单字符做为参数，那么尽量用字符作为参数。例如字符替换函数，参数列表并不是太长，就可以考虑额外写一段替换代码，使得每次传递参数都是一个字符，而不是接受数组做为查找和替换参数。大事化小，1+1>2。

### 12 最好不用@
用@掩盖错误会降低脚本运行速度，并且在后台有很多额外操作。用@比起不用，效率差距 3 倍。特别不要在循环中使用@，在 5 次循环的测试中，即使是先用error_reporting(0)关掉错误，在循环完成后再打开，都比用@快。

### 13 数组元素加引号
$row['id']比$row[id]速度快7倍，建议养成数组键名加引号的习惯。

### 14 别在循环里用函数
例如：
for($x=0; $x < count($array); $x++) { }
这种写法在每次循环的时候都会调用 count() 函数，效率大大降低，建议这样：
$len = count($array); for($x=0; $x < $len; $x++) { }
让函数在循环外面一次获得循环次数。

### 16 方法里建立局部变量
在类的方法里建立局部变量速度最快，几乎和在方法里调用局部变量一样快。

### 17 局部变量比全局变量快2倍
由于局部变量是存在栈中的，当一个函数占用的栈空间不是很大的时候，这部分内存很有可能全部命中cache，这时候CPU访问的效率是很高的。
相反，如果一个函数里既使用了全局变量又使用了局部变量，那么当这两段地址相差较大时，cpu cache需要来回切换，那么效率会下降。

### 18 局部变量而不是对象属性
建立一个对象属性（类里面的变量，例如：$this->prop++）比局部变量要慢3倍。

### 19 提前声明局部变量
建立一个未声明的局部变量要比一个已经定义过的局部变量慢9-10倍。

### 20 谨慎声明全局变量
声明一个未被任何一个函数使用过的全局变量也会使性能降低（和声明相同数量的局部变量一样）。PHP可能去检查这个全局变量是否存在。

### 21 类的性能和其方法数量没有关系
新添加10个或多个方法到测试的类后，性能没什么差异。

### 22 在子类里方法的性能优于在基类中

### 23 函数快于类方法
调用只有一个参数、并且函数体为空的函数，花费的时间等于7-8次$localvar++运算，而同一功能的类方法大约为15次$localvar++运算。

### 24 用单引号代替双引号会快一些
因为PHP会在双引号包围的字符串中搜寻变量，单引号则不会。
PHP 引擎允许使用单引号和双引号来封装字符串变量，但是它们的速度是有很大的差别的！使用双引号的字符串会告诉 PHP 引擎，首先去读取字符串内容，查找其中的变量，并改为变量对应的值。一般来说字符串是没有变量的，所以使用双引号会导致性能不佳。最好是使用字符串连接而不是双引号字符串。
$output = "This is a plain string";  // 不好的实践 $output = 'This is a plain string';  // 好的实践 $type = "mixed";                     // 不好的实践 $output = "This is a $type string"; $type = 'mixed';                     // 好的实践 $output = 'This is a ' . $type .' string';

### 25 echo字符串用逗号代替点连接符更快些
echo可以把逗号隔开的多个字符串当作“函数”参数传入，所以速度会更快。（说明：echo是一种语言结构，不是真正的函数，故把函数加上了双引号）。例如：
echo $str1, $str2;       // 速度快 echo $str1 . $str2;      // 速度稍慢

### 26 尽量静态化
Apache/Nginx解析一个PHP脚本的时间，要比解析一个静态HTML页面慢2至10倍，所以尽量使页面静态化，或使用静态HTML页面。

### 28 使用缓存
Memchached或者Redis都可以。
高性能的分布式内存对象缓存系统，提高动态网络应用程序性能，减轻数据库的负担。
也对运算码 (OP code)的缓存很有用，使得脚本不必为每个请求做重新编译。

### 29 使用整型保存IP
使用ip2long()和long2ip()函数把IP地址转成整型后，再存放进数据库，而保存非字符型。
这几乎能降低1/4的存储空间。同时可以很容易对地址进行排序和快速查找;

### 30 检查email有效性
使用[checkdnsrr()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.checkdnsrr.php)通过域名存在性来确认email地址的有效性，这个内置函数能保证每一个的域名对应一个IP地址。

### 31 使用MySQLi或PDO
mysql_*函数已经不被建议使用，建议使用增强型的mysqli_*系列函数或者直接使用PDO。

### 32 试着喜欢使用三元运算符(?:)

### 33 是否需要组件
在你想在彻底重做你的项目前，看看是否有现成的组件（在[Packagist](https://link.zhihu.com/?target=https%3A//packagist.org/)上）可用，通过composer安装。组件是别人已经造好的轮子，是个巨大的资源库，很多php开发者都知道。

### 35 屏蔽敏感信息
使用[error_reporting()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.error-reporting.php)函数来预防潜在的敏感信息显示给用户。
理想的错误报告应该被完全禁用在php.ini文件里。可是如果你在用一个共享的虚拟主机，php.ini你不能修改，那么你最好添加[error_reporting()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.error-reporting.php)函数，放在每个脚本文件的第一行(或用require_once()来加载)这能有效的保护敏感的SQL查询和路径在出错时不被显示;

### 36 压缩大的字符串
使用[gzcompress()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.gzcompress.php)和[gzuncompress()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.gzuncompress.php)对容量大的字符串进行压缩/解压，再存进/取出数据库。这种内置的函数使用gzip算法，能压缩字符串90%。

### 37 引用传递参数
通过参数地址引用使函数有多个返回值，在参数变量前加个“&”表示按地址传递，而非按值传递。

### 38 完全理解魔术引用和SQL注入的危险。
Fully understand “magic quotes” and the dangers of SQL injection. I’m hoping that most developers reading this are already familiar with SQL injection. However, I list it here because it’s absolutely critical to understand. If you’ve never heard the term before, spend the entire rest of the day googling and reading.

### 39 某些地方使用isset代替strlen
当操作字符串并需要检验其长度是否满足某种要求时，你想当然地会使用[strlen()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.strlen.php)函数。此函数执行起来相当快，因为它不做任何计算，只返回在zval结构（C的内置数据结构，用于存储PHP变量）中存储的已知字符串长度。但是，由于strlen()是函数，多多少少会有些慢，因为函数调用会经过诸多步骤，如字母小写化（译注：指函数名小写化，PHP不区分函数名大小写）、哈希查找，会跟随被调用的函数一起执行。在某些情况下，你可以使用[isset()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.isset.php)技巧加速执行你的代码。
例如：
if (strlen($foo) < 5) {     echo "Foo is too short"; } // 使用isset() if (!isset($foo{5})) {     echo "Foo is too short"; }

### 40 使用++$i递增
当执行变量$i的递增或递减时，$i++会比++$i慢一些。这种差异是PHP特有的，并不适用于其他语言，所以请不要修改你的C或Java代码并指望它们能立即变快，没用的。++$i更快是因为它只需要3条指令(opcodes)，$i++则需要4条指令。后置递增实际上会产生一个临时变量，这个临时变量随后被递增。而前置递增直接在原值上递增。这是最优化处理的一种，正如Zend的PHP优化器所作的那样。牢记这个优化处理不失为一个好主意，因为并不是所有的指令优化器都会做同样的优化处理，并且存在大量没有装配指令优化器的互联网服务提供商（ISPs）和服务器。

### 40 不要随便复制变量
有时候为了使 PHP 代码更加整洁，一些 PHP 新手（包括我）会把预定义好的变量复制到一个名字更简短的变量中，其实这样做的结果是增加了一倍的内存消耗，只会使程序更加慢。试想一下，在下面的例子中，如果用户恶意插入 512KB 字节的文字到文本输入框中，这样就会导致 1MB 的内存被消耗！
// 不好的实践 $description = $_POST['description']; echo $description; // 好的实践  echo $_POST['description'];

### 41 使用选择分支语句
switch、case好于使用多个if、else if语句,并且代码更加容易阅读和维护。

### 42 用file_get_contents替代file、fopen、feof、fgets
在可以用[file_get_contents()](https://link.zhihu.com/?target=http%3A//php.net/manual/zh/function.file-get-contents.php)替代file()、fopen()、feof()、fgets()等系列方法的情况下，尽量用file_get_contents()，因为他的效率高得多！但是要注意,file_get_contents()在打开一个URL文件时候的PHP版本问题。

### 43 尽量的少进行文件操作，虽然PHP的文件操作效率也不低的

### 44 优化Select SQL语句
在可能的情况下尽量少的进行insert、update操作(在update上，我被恶批过)。

### 45 尽可能的使用PHP内部函数

### 46 循环内部不要声明变量，尤其是大变量：对象
这好像不只是PHP里面要注意的问题吧？

### 47 多维数组尽量不要循环嵌套赋值

### 48 循环用foreach效率更高
尽量用foreach代替while和for循环

### 50 对global变量，应该用完就unset()掉

### 51 并不是事必面向对象(OOP)
面向对象往往开销很大，每个方法和对象调用都会消耗很多内存。

### 52 方法不要细分得过多
仔细想想你真正打算重用的是哪些代码？

### 53 耗时函数考虑用C扩展的方式实现
如果在代码中存在大量耗时的函数，你可以考虑用C扩展的方式实现它们。

### 54 mod_deflate压缩输出
打开apache的mod_deflate模块，可以提高网页的浏览速度。（提到过echo 大变量的问题）

### 55、数据库连接当使用完毕时应关掉，不要用长连接

### 56、split比exploade快
split() 0.001813 - 0.002271 seconds (avg 0.002042 seconds) explode() 0.001678 - 0.003626 seconds (avg 0.002652 seconds)
以上都是关于，下面是从整体结构方面优化PHP性能：

## **2 整体结构优化PHP性能**

### 1 将PHP升级到最新版
提高性能的最简单的方式是不断升级、更新PHP版本。

### 2 使用分析器
网站运行缓慢的原因颇多，Web应用程序极其复杂，让人扑朔迷离。而一种可能性在于PHP代码本身。这个分析器可以帮助你快速找出造成瓶颈的代码，提高网站运行的总体性能。
Xdebug PHP extension提供了强大的功能，可以用来调试，也可以用来分析代码。方便开发人员直接跟踪脚本的执行，实时查看综合数据。还可以将这个数据导入到可视化的工具 KCachegrind中。

### 3 检错报告
PHP支持强大的检错功能，方便你实时检查错误，从比较重要的错误到相对小的运行提示。总共支持13种独立的报告级别，你可以根据这些级别灵活匹配，生成用户自定义的检测报告。

### 4 利用PHP的扩展
一直以来，大家都在抱怨PHP内容太过繁杂，最近几年来开发人员作出了相应的努力，移除了项目中的一些冗余特征。即便如此，可用库以及其它扩展的数量还是很可观。甚至一些开发人员开始考虑实施自己的扩展方案。

### 5 PHP缓存，使用PHP加速器：APC
一般情况下，PHP脚本被PHP引擎编译后执行，会被转换成机器语言，也称为操作码。如果PHP脚本经过反复编译而得到相同的结果，那为什么不完全跳过编译过程呢?
通过PHP加速器，你完全可以实现这一点，它缓存了PHP脚本编译后的机器码，允许代码根据要求立即执行，而不经过繁琐的编译过程。
对PHP开发人员而言，目前提供了两种可用的缓存方案，一种是APC(Alternative PHP Cache，可选PHP缓存)，它是一个可以通过PEAR安装的开源加速器。另一种流行的方案是Zend Server，它不仅提供了操作码缓存技术，也提供了相应页面的缓存工具。

### 6 内存缓存
PHP通常在检索和数据分析方面扮演着重要角色，这些操作可能会导致性能降低。实际上有些操作是完全没有必要的，特别是从数据库中反复检索一些常用的静态数据。不妨考虑一下短期使用Redis或Memcached extension来缓存数据。Memcached的扩展缓存与libMemcached库协同工作，在RAM中缓存数据，也允许用户定义缓存的期限，有助于确保用户信息的实时更新。

### 7 内容压缩
几乎所有的浏览器都支持Gzip的压缩方式，gzip可以降低80%的输出，付出的代价是大概增加了10%的cpu计算量。但是赚到的是不仅占用的带宽减少了，而且你的页面加载会变得很快，优化了你的PHP站点性能。你可以在PHP.ini中开启它：
zlib.output_compression = On zlib.output_compression_level = (level)
level可能是1-9之间的数字，你可以设置不同的数字使得他适合你的站点。
如果你使用apache，你也可以激活mod_gzip模块，他是高度可定制的。

### 8 服务器缓存
主要是基于web反向代理的静态服务器nginx和squid，还有apache2的mod_proxy和mod_cache模块

### 9 数据库优化，缓存等
通过配置数据库缓存，如开启QueryCache缓存，当查询接收到一个和之前同样的查询， 服务器将会从查询缓存种检索结果，而不是再次分析和执行上次的查询以及数据存储过程，连接池技术等。
**以上内容希望帮助到大家，**很多PHPer在进阶的时候总会遇到一些问题和瓶颈，业务代码写多了没有方向感，不知道该从那里入手去提升，对此我整理了一些资料，包括但不限于：分布式架构、高可扩展、高性能、高并发、服务器性能调优、TP6，laravel，YII2，Redis，Swoole、Swoft、Kafka、Mysql优化、shell脚本、Docker、微服务、Nginx等多个知识点高级进阶干货需要的可以免费分享给大家**，需要戳下方**

## 



用户进入表单授权后无法跳转到表单页
身份证正面没
