---
title: 设计模式 UML
urlname: ssyie9zd2tkm0s1s
date: '2022-12-26 14:44:00 +0800'
tags: []
categories: []
---

tags: [设计模式基础知识]
categories: [设计模式]
cover: ''

---

## 概述

> 设计模式通常也结合一些图形来进行描述，其中最常用、使用最广泛的图形描述技术就是 UML（Unified Modeling Language，统一建模语言）。

### 特性

- 统一（Unified）：UML 融合了多种优秀的面向对象建模方法以及多种得到认可的软件工程方法，消除了因方法林立且相互独立而带来的种种不便，集百家之所长，UML 通过统一的表示方法，让不同知识背景的领域专家、系统分析设计人员和开发人员以及用户可以方便地交流。
- 建模（Modeling）：不同于编程语言，它通过一些标准的图形符号和文字来对系统进行建模，用于对软件进行描述、可视化处理、构造和建立软件系统制品的文档。UML 适用于各种软件开发方法、软件生命周期的各个阶段、各种应用领域以及各种开发工具，UML 是一种总结了以往建模技术的经验并吸收了当今最优秀成果的标准建模方法。
- 语言（Language）：也就意味着它有属于自己的标准表达规则，它不是一种类似 Java、C++、C＃的编程语言，而是一种分析设计语言，也就是一种建模语言。

### 结构

#### 视图（View）

> UML 视图用于从不同的角度来表示待建模系统。
> 视图是由许多图形组成的一个抽象集合，在建立一个系统模型时，只有通过定义多个视图，每个视图显示该系统的一个特定方面，才能构造出该系统的完整蓝图，视图也将建模语言链接到开发所选择的方法和过程。
> UML 视图包括：
>
> - 用户视图：是所有视图的核心，用于描述系统的需求。
> - 结构视图：表示系统的静态行为，描述系统的静态元素，如包、类与对象，以及它们之间的关系。
> - 行为视图：表示系统的动态行为，描述系统的组成元素（如对象）在系统运行时的交互关系。
> - 实现视图：表示系统中逻辑元素的分布，描述系统中物理文件以及它们之间的关系。
> - 环境视图：描述系统中硬件设备以及它们之间的关系。

#### 图（Diagram）

> UML 图是描述 UML 视图内容的图形。最新的 UML 2.0 提供了 13 种图，
> 分别是
>
> - 用例图（Use CaseDiagram）
> - 类图（Class Diagram）
> - 对象图（Object Diagram）
> - 包图（Package Diagram）
> - 组合结构图（Composite StructureDiagram）
> - 状态图（State Diagram）
> - 活动图（Activity Diagram）
> - 顺序图（Sequence Diagram）
> - 通信图（CommunicationDiagram）
> - 定时图（Timing Diagram）
> - 交互概览图（Interaction Overview Diagram）
> - 组件图（Component Diagram）
> - 部署图（Deployment Diagram）

## 图示

### 类

> 类（Class）封装了数据和行为，是面向对象的重要组成部分。
> 它是具有相同属性、操作、关系的对象集合的总称。
> 在系统中，每个类都具有一定的职责。
> 职责指的是类要完成什么样的功能，要承担什么样的义务。
> 一个类可以有多种职责，设计得好的类一般只有一种职责。
> 在定义类的时候，将类的职责分解成为类的属性和操作（即方法）。
> 类的属性即类的数据职责，类的操作即类的行为职责。
> 设计类是面向对象设计中最重要的组成部分，也是最复杂和最耗时的部分
> 类将被实例化成对象（Object），对象对应于某个具体的事物，是类的实例（Instanc）。
> 类图（Class Diagram）是用出现在系统中的不同类来描述系统的静态结构，主要用来描述不同的类以及它们之间的关系。

#### 图示

> 类使用包含类名、属性和操作且带有分隔线的长方形来表示。
> 在 UML 类图中，类一般由三部分组成。
> （1）类名：每个类都必须有一个名字，类名是一个字符串。
> （2）类的属性（Attributes）：属性是指类的性质，即类的成员变量。一个类可以有任意多个属性，也可以没有属性。
> UML 规定属性的表示方式为：
> 可见性 名称：类型[ = 默认值]

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672037973252-1a73b7f3-2812-448d-9903-9aba40fddd01.png#averageHue=%23e0e0e0&clientId=ufbfed401-b14f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=97&id=ud7cbf547&margin=%5Bobject%20Object%5D&name=image.png&originHeight=194&originWidth=283&originalType=binary∶=1&rotation=0&showTitle=false&size=25214&status=done&style=none&taskId=ue9592ba6-ddf7-4cf3-b7fe-fb9cfb3acd9&title=&width=141.5)

```java
// 可见性 属性名：类型[ = 默认值]
public class Employee {
  private String name;//
  private int age;//私有属性 private 用 - 表示
  protected String email;//受保护属性 protected 用 # 号表示
  public void modifyInfo() {//共有（public）属性用 + 表示。

  }
}
```

### 类之间关系

#### 关联关系

> 关联（Association）关系是类与类之间最常用的一种关系，它是一种结构化关系，用于表示一类对象与另一类对象之间有联系，如汽车和轮胎、师傅和徒弟、班级和学生等。
> 在 UML 类图中，用实线连接有关联关系的对象所对应的类，在使用 Java、C＃和 C++等编程语言实现关联关系时，通常将一个类的对象作为另一个类的成员变量。
> 在使用类图表示关联关系时可以在关联线上标注角色名，一般使用一个表示二者之间关系的动词或者名词表示角色名（有时该名词为实例对象名），关系的两端代表两种不同的角色。
> 因此，在一个关联关系中可以包含两个角色名，角色名不是必需的，可以根据需要增加，其目的是使类之间的关系更加明确。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672040825684-2106f924-6f8a-4f32-ba76-4222a18a1dc7.png#averageHue=%23efefef&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=68&id=u5a336dab&margin=%5Bobject%20Object%5D&name=image.png&originHeight=136&originWidth=919&originalType=binary∶=1&rotation=0&showTitle=false&size=22935&status=done&style=none&taskId=u1f589f80-62d2-409f-8355-35bd4a2bc36&title=&width=459.5)

##### 双向关联

> 默认情况下，关联是双向的。例如，顾客（Customer）购买商品（Product）并拥有商品，
> 反之，卖出的商品总有某个顾客与之相关联。因此，Customer 类和 Product 类之间具有双向关联关系，

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672040872230-3ee52e53-745e-4feb-8644-b5d722498c45.png#averageHue=%23eeeeee&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=78&id=u25fda8f8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=155&originWidth=1033&originalType=binary∶=1&rotation=0&showTitle=false&size=33651&status=done&style=none&taskId=u17747cbc-d0f7-45e4-9fdb-dfa8a3dda60&title=&width=516.5)

##### 单向关联

> 类的关联关系也可以是单向的，在 UML 中单向关联用带箭头的实线表示

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672040902177-457b43d8-4303-490d-b68b-54fa9c3a4a65.png#averageHue=%23f2f2f2&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=77&id=uff615499&margin=%5Bobject%20Object%5D&name=image.png&originHeight=154&originWidth=951&originalType=binary∶=1&rotation=0&showTitle=false&size=23979&status=done&style=none&taskId=u80514467-3327-4595-9a37-2ca444b094b&title=&width=475.5)

##### 自关联

> 在系统中可能会存在一些类的属性对象类型为该类本身，这种特殊的关联关系称为自关联。例如，一个节点类（Node）的成员又是节点 Node 类型的对象，如图 2-5 所示。

##### 多重性关联

> 多重性关联关系又称为重数性（Multiplicity）关联关系，表示两个关联对象在数量上的对应关系。在 UML 中，对象之间的多重性可以直接在关联直线上用一个数字或一个数字范围表

| 表示方式 | 说明                                                          |
| -------- | ------------------------------------------------------------- |
| 1..1     | 表示另一个类的一个对象只与该类的一个对象有关系                |
| 0..\*    | 表示另一个类的一个对象与该类的零个或多个对象有关系            |
| 1..\*    | 表示另一个类的一个对象与该类的一个或多个对象有关系            |
| 0..1     | 表示另一个类的一个对象没有或只与该类的一个对象有关系          |
| m..n     | 表示另一个类的一个对象与该类最少 m 最多 n 个对象有关系（m≤n） |

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672100872790-b048b840-72f4-44ba-bb57-da653453f178.png#averageHue=%23f2f2f2&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=71&id=ua867f87e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=142&originWidth=887&originalType=binary∶=1&rotation=0&showTitle=false&size=19407&status=done&style=none&taskId=u029d9928-e20c-47d7-bb02-60bf71e103a&title=&width=443.5)

##### 聚合关系

> 聚合（Aggregation）关系表示整体与部分的关系。
> 在聚合关系中，成员对象是整体对象的一部分，但是成员对象可以脱离整体对象独立存在。
> 在 UML 中，聚合关系用带空心菱形的直线表示。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672100848661-9f361c01-7df3-4a5f-b0a8-e91181b773f9.png#averageHue=%23efefef&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=94&id=u831c2287&margin=%5Bobject%20Object%5D&name=image.png&originHeight=187&originWidth=1045&originalType=binary∶=1&rotation=0&showTitle=false&size=39407&status=done&style=none&taskId=u270ac5d4-9056-4825-b588-83a4771f744&title=&width=522.5)

##### 组合关系

> 组合（Composition）关系也表示类之间整体和部分的关系，但是在组合关系中整体对象可以控制成员对象的生命周期。
> 一旦整体对象不存在，成员对象也将不存在，成员对象与整体对象之间具有同生共死的关系。
> 在 UML 中，组合关系用带实心菱形的直线表示。

#### 依赖关系

> 依赖（Dependency）关系是一种使用关系，特定事物的改变有可能会影响到使用该事物的其他事物，在需要表示一个事物使用另一个事物时使用依赖关系。
> 大多数情况下，依赖关系体现在某个类的方法使用另一个类的对象作为参数。
> 在 UML 中，依赖关系用带箭头的虚线表示，由依赖的一方指向被依赖的一方。
> 赖关系通常通过 3 种方式来实现：
>
> 1. 最常用的一种方式，将一个类的对象作为另一个类中方法的参数；如下图
> 2. 是在一个类的方法中将另一个类的对象作为其局部变量；
> 3. 是在一个类的方法中调用另一个类的静态方法。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672101449149-830ddc70-ad73-4cd7-b8f8-c4591a8dc399.png#averageHue=%23f7f7f7&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=159&id=u63769208&margin=%5Bobject%20Object%5D&name=image.png&originHeight=317&originWidth=1405&originalType=binary∶=1&rotation=0&showTitle=false&size=40775&status=done&style=none&taskId=ube4b0b9c-ef08-4bdc-b02a-b34ca0c562f&title=&width=702.5)

#### 泛化关系

> 泛化（Generalization）关系也就是继承关系，用于描述父类与子类之间的关系，父类又称作基类或超类，子类又称作派生类。
> 在 UML 中，泛化关系用带空心三角形的直线来表示。
> 在代码实现时，使用面向对象的继承机制来实现泛化关系，如在 Java 语言中使用 extends 关键字、在 C++/C＃中使用冒号“：”来实现

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672101636156-386d1c23-884e-4505-8b35-e3941fdd20b5.png#averageHue=%23f5f5f5&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=221&id=u3fb67b50&margin=%5Bobject%20Object%5D&name=image.png&originHeight=441&originWidth=1072&originalType=binary∶=1&rotation=0&showTitle=false&size=65976&status=done&style=none&taskId=uc522855a-11a5-4e75-9bfc-1bdbc3975ed&title=&width=536)

#### 接口与实现关系

> 接口之间也可以有与类之间关系类似的继承关系和依赖关系，但是接口和类之间还存在一种实现（Realization）关系。
> 在这种关系中，类实现了接口，类中的操作实现了接口中所声明的操作。
> 在 UML 中，类与接口之间的实现关系用带空心三角形的虚线来表示。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1672101700046-866d9701-e46b-4cd3-9116-ce66b7b86a8e.png#averageHue=%23f8f8f8&clientId=u6b191b86-3f7e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=221&id=u682449fa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=441&originWidth=914&originalType=binary∶=1&rotation=0&showTitle=false&size=43782&status=done&style=none&taskId=u8edea81c-f414-471e-913e-b536dbe666e&title=&width=457)
