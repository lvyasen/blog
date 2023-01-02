---
title: Java 安装
urlname: sgesiz0cgmpd0fzw
date: '2022-12-26 11:56:32 +0800'
tags: []
categories: []
---

tags: []
categories: [Java]
cover: ''

---

## 下载和安装

> Java 源代码可在[OpenJDK 官方网站](https://openjdk.org/)下载。代码库包含组成 JDK 的项目，包括类库、虚拟机（代号为 HotSpot）、Java 编译器和其他工具。要使用此源代码，必须将其构建到 Java 开发所需的软件套件中。尽管可以自己构建 JDK，但这并不容易，即使对于经验丰富的 Java 程序员也是如此。优选的方法是下载一个可用的 JDK 构建。Oracle、AdoptOpenJDK 开源社区和其他提供商都提供免费下载的 JDK 构建版本。这些构建具有不同的许可协议，但通常都可以免费用于开发。

### Linux 安装

> 在 Linux 平台上，JDK 有以下两种安装文件格式：一种是 RPM，支持 RPM 包管理系统的 Linux 平台，如 Red Hat 和 SuSE；另一种是自解压包，其中包含安装包的压缩文件。

```bash
#如果使用RPM，请按以下步骤操作。
#（1）使用su 命令成为root用户。
#（2）解压下载的文件。
#（3）将路径改为下载文件所在的位置，输入下面的命令：
chmod a + x rpmFile
#这里的rpmFile是RPM文件。
#（4）运行RPM文件：
./ rpmFil


#如果使用自解压二进制安装文件，请按以下步骤操作：
#（1）解压下载的文件。
#（2）用chmod
#为该文件提供执行权限：
chmod a + x binFile
#这里的binFile是为读者的平台下载的bin文件。
#（3）将路径更改为要安装文件的位置。
#（4）运行自解压二进制文件。将路径加到下载的文件名前面来执行它。例如，如果文件在当前目录中，在文件前面加上“./”。
./ binFile
```

### MacOS 安装

> 要在 macOS 上安装 JDK 11，需要一台基于英特尔的计算机运行 macOS，还需要管理员特权。安装的具体步骤如下。
> （1）双击下载的.dmg 文件。
> （2）在出现的 Finder 窗口中，双击包图标。
> （3）在出现的第一个窗口中，单击 Continue 按钮。
> （4）出现安装类型窗口，单击 Install 按钮。
> （5） 这时会出现一个窗口，显示“Installer is trying to install new software. Type your password to allow this”，输入读者的 Admin 密码。
> （6）单击 Install Software 按钮启动安

### 设置系统环境变量

#### windows

> 要在 Windows 上设置 PATH 环境变量，请按下面的步骤操作。
> （1）如果使用的是 Windows 10，请在工具栏上的搜索框中输入“environment”，然后单击 Windows 找到的第一个搜索结果。
> （2）如果使用的是 Windows 7 或 Windows 8，请依次选择 Start>Settings>Control Panel>System。
> （3）如果还没有打开，选择 Advanced 选项卡，然后单击 Environment Variables 按钮。
> （4）在用户变量或系统变量窗口中找到 PATH 环境变量。PATH 值是由分号分隔的一系列目录。现在，单击 Edit 按钮将$JAVA_HOME/bin（Java 安装目录下 bin
> 目录的完整路径）添加到 PATH
> 现有值的末尾，或者在资源管理器中找到该目录——类似于如下样式：
> C:\Program Files\Java\jdk-11\bin
> （5）单击 Set、OK 或 Apply 按钮

#### 在 UNIX 和 Linux 上设置 PATH 环境变量

> 在 UNIX 和 Linux 操作系统上设置 PATH 环境变量取决于所使用的 shell。对于 C shell，要在~/ cshrc
> 文件后面添加以下内容：
> set path=(path/to/jdk/bin $path)
> 这里的path/to/jdk/bin是JDK安装目录下的bin目录。
> 对于Bourne Again shell，要在~/.bashrc
> 或~/.bash_profile
> 文件之后添加下面这行代码：
> export PATH=/path/to/jdk/bin:$PATH
> 这里的 path/to/jdk/bin 是 JDK 安装目录下的 bin 目录。

## 编写

> 所有 Java 源文件都必须以 java 作为扩展名。

```java
class MyFirst{
  public static void main(String[] args){
    System.out.println("hello world");
  }
}
//文件 MyFirst.java

```

## 编译

> 使用 JDK 安装目录 bin 目录中的 javac 编译 Java 程序。假设已经修改了计算机的 PATH 环境变量，就可从任何目录调用 javac。要编译清单 1.1 中的 MyFirstProgram 类，请执行以下操作。
> （1）打开终端或命令提示符，并将目录更改为保存 MyFirstProgram.java 文件的目录。
> （2）输入以下命令：
> `javac MyFirstProgram.java`

## 执行

> 要执行 Java 程序，要用到 JDK 中的 java 程序。在设置了 PATH 环境变量之后，就能从任意目录调用 java
> 解释器。在工作目录中，输入以下内容并按 Enter 键：
> `java MyFirst `
> 还可以给 Java 程序传递参数，例如，如果有一个名为 Calculator 的类，想给它传递两个参数，可以这样执行程序：
> `java Calculator arg-1 arg-2`

## 规范

## JShell

> JShell 是 Java 9 的一个新特性，它是一个环境，称为 REPL（Read-Eval-Print Loop）或语言外壳，用于读取用户输入、计算输入并打印输出结果。
> 可以使用 JShell 输入一小段程序并在没有编译的情况下运行它。
> 在实际应用中，通常用 JShell 执行一些琐碎的任务，比如测试一个方法或做简单的数学计算。

```bash
#输入 jshell
jshell
jshell> int i = 6;var k=8;
i ==> 6
k ==> 8

```
