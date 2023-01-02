---
title: Java 语言基础
urlname: kp6xvtygnbeuwibk
date: '2022-12-26 12:13:46 +0800'
tags: []
categories: []
---

tags: []
categories: [Java]
cover: ''

---

> 由于 Java 是一种面向对象的编程（OOP）语言，因此理解 OOP 至关重要。
> Java 语言是面向对象的程序设计语言，Java 程序的基本组成单元是类 阿斯达

## ASCII 和 Unicode

> 传统上，使用英语国家的计算机只使用 ASCII（美国信息交换标准代码）字符集来表示字母、数字、字符。
> ASCII 中的每个字符由 7 位二进制数表示。
> 因此，这个字符集有 128 个字符，包括小写拉丁字母、大写拉丁字母、数字和标点符号等。
> 然而，并不是每种语言都使用拉丁字母，汉语和日语就是使用不同字符集的语言的例子。
> 例如，汉语中的每个字符代表一个单词，而不是一个字母，这些字符的个数成千上万，8 位二进制数不足以表示字符集中的所有字符。
> 日语同样如此，也使用了不同的字符集。总的来说，世界各国语言包含数百种不同的字符集，为了统一所有这些字符集，一个名为 Unicode 的计算标准应运而生。
> Unicode 是一个由非营利性组织 Unicode Consortium 开发的字符集。
> 它试图将世界上所有语言中的所有字符包含到一个字符集中。
> 在 Unicode 中，每个字符使用唯一编码表示。Unicode 目前的版本是 11，用在 Java、XML、ECMAScript、LDAP 等语言中。

## 主类结构

> Java 语言是面向对象的程序设计语言，Java 程序的基本组成单元是类，类体中又包括属性与方法两部分（本书将在第 6 章中逐一介绍）。
> 每一个应用程序都必须包含一个 main()方法，含有 main()方法的类称为主类。

```java
package Number;//声明包
public class Frist {
    static String s1= "你好";
    public static void main(String[] args) {
        String s2 = "Java";
        System.out.println(s1);
        System.out.println(s2);
	}
}
```

## 基本类型

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672035475352-4bf27737-5fc9-4978-a042-f74c2105e0e7.png#averageHue=%23e6ede1&clientId=u6f8f5600-1f4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=458&id=u8e70c85c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=597&originWidth=799&originalType=binary∶=1&rotation=0&showTitle=false&size=162711&status=done&style=none&taskId=uf3abb548-1949-4496-8557-d43b1256342&title=&width=613.5)
