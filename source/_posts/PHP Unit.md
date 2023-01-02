---
title: PHP Unit
urlname: fcs6gp649gt2157g
date: '2022-12-28 11:03:14 +0800'
tags: []
categories: []
---

tags: [PHP 单元测试]
categories: [PHP]
cover: ''

---

> [中文手册](https://phpunit.readthedocs.io/zh_CN/latest/index.html)

## 作用

> 单元测试主要是作为一种良好实践来编写的，它能帮助开发人员识别并修复 bug、重构代码，还可以看作被测软件单元的文档。
> 要实现这些好处，理想的单元测试应当覆盖程序中所有可能的路径

## 安装

> 这里默认使用 `composer`安装

```bash
composer require --dev phpunit/phpunit ^latest
```

## 依赖关系

> 使用 `@depents` 标注依赖关系

```php
<?php
declare(strict_types=1);
use PHPUnit\Framework\TestCase;//继承 TestCase

final class StackTest extends TestCase
{
    public function testEmpty(): array
    {
        $stack = [];
        $this->assertEmpty($stack);

        return $stack;
    }

    /**
     * @depends testEmpty
     * 使用  @depends 方法名 标注依赖
     */
    public function testPush(array $stack): array
    {
        array_push($stack, 'foo');
        $this->assertSame('foo', $stack[count($stack)-1]);
        $this->assertNotEmpty($stack);

        return $stack;
    }

    /**
     * @depends testPush
     */
    public function testPop(array $stack): void
    {
        $this->assertSame('foo', array_pop($stack));
        $this->assertEmpty($stack);
    }
}
```

## 断言

> [中文文档](https://phpunit.readthedocs.io/zh_CN/latest/assertions.html)

- 判断对象是否一致：assertSame(obj1,obj2)
-
