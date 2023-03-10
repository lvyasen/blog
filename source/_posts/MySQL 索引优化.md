---
title: MySQL 索引优化
urlname: irclt42der8hmum3
date: '2022-12-06 08:42:53 +0800'
tags:
  - mysql
categories:
  - mysql
---

> 索引，在 MySQL 中也叫作键（key），是存储引擎用于快速找到记录的一种数据结构。
> 索引是在存储引擎实现的，而不是在服务层实现的。
> B-tree 是按照索引列中的数据大小顺序存储的，所以很适合按照范围来查询。

## 索引基础

### 使用索引优点

- 索引大大减少了服务器需要扫描的数据量。
- 索引可以帮助服务器避免排序和临时表。
- 索引可以将随机 I/O 变为顺序 I/O。

### 索引类型

#### 聚簇索引（主键索引）

> 一般是通过主键聚集索引，如果没有主键那么会选择一个非空唯一索引代替主键。
> 聚簇索引占用空间最大，因为保存了全部的数据。
> 因为高并发的瓶颈在于磁盘 IO ，如果树高为 3 则只需进行三次 IO 就能找到数据。
> 数据挂载到叶子节点，叶子节点使用双向链表关联。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671152713469-a208cc41-bb43-4eef-a103-ca4b82b003b5.png#averageHue=%23f7f7f7&clientId=udb13fb17-79d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=265&id=ud3788764&margin=%5Bobject%20Object%5D&name=image.png&originHeight=529&originWidth=1018&originalType=binary∶=1&rotation=0&showTitle=false&size=89063&status=done&style=none&taskId=ua036846e-1322-41b2-b1ba-ab29a100ef1&title=&width=509)

#### 非聚簇索引（非主键索引、二级索引、辅助索引）

> 辅助索引，也称为二级索引，单张表可以有多个。
> InnoDB 辅助索引的叶子节点只存放对应索引字段的键值和主键 ID。
> 为了减少回表次数，可以将语句中经常使用到的所有列以合适的顺序建立一个二级联合索引。
> 有时创建二级索引可能比没创建索引执行的时间要长！因为如果索引选择性不高，先扫描索引再回表查数据，没有直接不用索引的好，如性别字段建立索引。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671172419662-e49decd5-d9e8-4660-8636-21ff3816ecfb.png#averageHue=%23f8f8f8&clientId=udb13fb17-79d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=228&id=u0be7feea&margin=%5Bobject%20Object%5D&name=image.png&originHeight=455&originWidth=933&originalType=binary∶=1&rotation=0&showTitle=false&size=64630&status=done&style=none&taskId=u868071d9-0672-4e8b-a3b4-b146c93c03d&title=&width=466.5)

##### 唯一索引

> 非组件索引中索引选择性最高。
> 唯一索引是一个不包含重复值的二级索引。

##### 联合索引

> 单个字段唯一性很低，需要联合多个字段才能达到最优效果，这种由多个字段组成的二级索引称为联合索引。
> 可以避免回表。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671172939017-c3845183-4ec5-4c33-a554-442c3eadd2e7.png#averageHue=%23f8f8f8&clientId=udb13fb17-79d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=282&id=u1ab79ef2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=563&originWidth=1215&originalType=binary∶=1&rotation=0&showTitle=false&size=100302&status=done&style=none&taskId=uaf709742-541d-4656-a6f7-e66cfbb511e&title=&width=607.5)

### 索引实现方式

> 在 MySQL 中，索引是在存储引擎层而不是服务器层实现的。
> 自适应哈希索引。InnoDB 存储引擎有一个被称为自适应哈希索引的特性。当 InnoDB 发现某些索引值被非常频繁地被访问时，它会在原有的 B-tree 索引之上，在内存中再构建一个哈希索引。

#### 为什么使用 B+ 树作为底层数据结构？

> 想了解这个问题必须了解数据结构，如不是用 B+树作为底层数据结构会怎样？

##### 顺序查找（最简单的算法）

> 如果从一组数据中找到对应记录，通常一个个扫描，知道找到对应的记录。
> 算法空间复杂度 O(n)

如一组数据 [1,2,3,4,5,6]，需要遍历数组对比。

##### 二分查找（在顺序查找中引入中间值，对比中间值）

> 先将记录按照顺序排列，查找时将中间节点作为比较对象。
> 算法复杂度 O(logn)
> 对比顺序查找快了很多。

##### 二叉树（binary tree 在二分查找基础上构建一棵树）

> 将一组无序的数据构建成一颗有序的树

**特点：**

- 每个节点最多有两个子节点。
- 每个节点都大于自己的左侧子节点。
- 每个节点都小于自己的右侧子节点。

**形态：**

- **空二叉树。**
- **只有一个节点的二叉树。**
- 右子树为空的二叉树（右腿断了）
- 左子树为空的二叉树（左腿断了）
- 左右子树都非空的的二叉树（既有左子树又有右子树，）

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1671157676179-c2881cdb-7c74-485f-9275-d6b95e65b9f7.png#averageHue=%23f2f2f2&clientId=udb13fb17-79d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=125&id=u259fbddb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=249&originWidth=414&originalType=binary∶=1&rotation=0&showTitle=false&size=37152&status=done&style=none&taskId=ucbac1d5a-0c66-41cd-8a39-fe2a40c1ac5&title=&width=207)
**缺点：**

- 顺序存储可能会浪费空间(在非完全二叉树的时候)，但是读取某个指定的节点的时候效率比较高 O(0)
- 链式存储相对二叉树比较大的时候浪费空间较少，但是读取某个指定节点的时候效率偏低 O(nlogn)
- 最坏的情况下会退化为一个链表

##### 平衡二叉树（为了解决二叉树退化为链表）

> 平衡二叉树是二叉树的改进版，除了满足二叉树的定义，还必须满足任意节点的平衡因子（两个树的高度差）的绝对值最大为 1.

##### B 树（多叉平衡二叉树）

> B 树可以理解为平衡二叉树的拓展，也是一棵平衡树，但是是多叉的。

特点：

1. 每个节点都存储了真实的数据。
2. B 树的查询效率与键在 B 树中的位置有关，最大时间复杂度与 B+树的相同（数据在叶子节点上），最小时间复杂度为 1（数据在根节点上）。

##### B+树

- B+树的键都出现在叶子节点上，可能在内节点上重复出现。
- B+树的内节点存储的都是键值，键值对应的具体数字都存储在叶子节点上。
- B+树比 B 树占的空间更多，因为 B+树的叶子节点包含所有数据，而 B 树是整棵树包含所有数据。多出的部分就是 B+树的内节点，但是 B+树的内节点具有索引的作用，因此，B+树的查询效率比 B 树的查询效率更高

#### B-Tree 索引

> B-tree 通常意味着所有的值都是按顺序存储的，并且每一个叶子页到根的距离相同。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25799318/1670296493254-3885946c-5770-428b-af63-d25156d98ce1.png#averageHue=%23f6f6f6&clientId=u4fff3aad-9a6a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=278&id=ude19af5f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1000&originWidth=1440&originalType=binary∶=1&rotation=0&showTitle=false&size=282910&status=done&style=none&taskId=uef98574d-0103-4559-9881-8f661e09f9f&title=&width=400)

#### B+Tree 索引
