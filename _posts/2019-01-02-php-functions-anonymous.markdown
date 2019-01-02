---
layout:       post
title:        "浅析PHP匿名函数"
subtitle:     ""
date:         2019-01-02 14:44:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - 语法基础
    - PHP
    - 匿名函数
    - 闭包
---

>PHP自5.3版本开始引入对闭包以及匿名函数的支持，说到闭包，一开始接触的时候会觉得很难，不容易理解，下面我们就一起来探究一下PHP的闭包。（本章节案例内容来源 [PHP官方文档](http://php.net/functions.anonymous)）

# 匿名函数
匿名函数也叫做闭包函数，允许临时创建一个没有指定名称的函数。最常用作回调函数参数的值，当然，也有其他应用的情况。

## 匿名函数简单示例

``` php
<?php
echo preg_replace_callback('~-([a-z])~', function ($match) {
    return strtoupper($match[1]);
}, 'hello-world');
// 输出 helloWorld
```
首先我们先来了解一下`preg_replace_callback`这个函数的定义
``` php
mixed preg_replace_callback ( mixed $pattern , callable $callback , mixed $subject [, int $limit = -1 [, int &$count ]] )
```

这个函数执行一个正则表达式搜索并且使用一个回调进行替换。

---
__参数说明__
- pattern
要搜索的模式，案例中的`'~-([a-z])~'` (正则表达式的定界符一般使用`'/'`，这里使用的是`'~'`)

- callback
一个回调函数，在每次需要替换时调用，调用时函数得到的参数是从subject中匹配的结果

- subject
要搜索替换的目标字符串或字符串数组

这里`preg_replace_callback`将正则表达式匹配的结果作为回调函数的参数`match`传入，然后在回调函数中进行字符串替换。

# 匿名函数变量赋值示例
``` php
<?php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
```
这里我们将匿名函数赋值给一个变量`$greet`，然后我们就可以通过这个变量来调用这个匿名函数。

# 从父作用域继承变量
闭包可以使用`use`语言结构从父作用域中继承变量
``` php
<?php
$message = 'hello';

// 没有 "use"

$example = function () {
    var_dump($message);
    //会提示错误:undefine varibale

};
echo $example();

// 继承 $message

$example = function () use ($message) {
    var_dump($message);
};
echo $example();
//输出hello

// Inherited variable's value is from when the function is defined, not when called

// 继承的变量值是从函数定义开始的而不是函数调用开始。

$message = 'world';
echo $example();
//依旧输出hello

// Reset message

// 重置 message 变量

$message = 'hello';

// Inherit by-reference

// 使用 引用 继承

$example = function () use (&$message) {
    var_dump($message);
};
echo $example();
//输出hello

// The changed value in the parent scope

// is reflected inside the function call

$message = 'world';
echo $example();
// 由于函数继承的是变量的应用，所以当变量值修改，

// 继承的变量也会修改，此处输出world

// Closures can also accept regular arguments

// 闭包函数也可以接受参数 

$example = function ($arg) use ($message) {
    var_dump($arg . ' ' . $message);
};
$example("hello");
//输出hello world
```

# 闭包和作用域
Inheriting variables from the parent scope is not the same as using global variables. Global variables exist in the global scope, which is the same no matter what function is executing. The parent scope of a closure is the function in which the closure was declared (not necessarily the function it was called from). See the following example

从父作用域继承变量和使用全局变量是不同的，全局变量无论哪个函数去调用他都是相同的因为他存在于全局作用域中，一个闭包的父作用域是定义该闭包的函数（与调用它的函数无关），示例如下。
``` php
<?php
// 一个基本的购物车，包括一些已经添加的商品和每种商品的数量。

// 其中有一个方法用来计算购物车中所有商品的总价格，该方法使

// 用了一个 closure 作为回调函数。

class Cart
{
    const PRICE_BUTTER  = 1.00;
    const PRICE_MILK    = 3.00;
    const PRICE_EGGS    = 6.95;

    protected   $products = array();
    
    public function add($product, $quantity)
    {
        $this->products[$product] = $quantity;
    }
    
    public function getQuantity($product)
    {
        return isset($this->products[$product]) ? $this->products[$product] :
               FALSE;
    }
    
    public function getTotal($tax)
    {
        $total = 0.00;
        
        $callback =
            function ($quantity, $product) use ($tax, &$total)
            {
                $pricePerItem = constant(__CLASS__ . "::PRICE_" .
                    strtoupper($product));
                $total += ($pricePerItem * $quantity) * ($tax + 1.0);
            };
        
        array_walk($this->products, $callback);
        // 使用自定义函数对数组中的每个元素做回调处理

        return round($total, 2);
        // 四舍五入保留两位小数

    }
}

$my_cart = new Cart;

// 往购物车里添加条目

$my_cart->add('butter', 1);
$my_cart->add('milk', 3);
$my_cart->add('eggs', 6);

// 打出出总价格，其中有 5% 的销售税.

print $my_cart->getTotal(0.05) . "\n";
// 最后结果是 54.29

```
这里我们简单介绍一下`array_walk`这个函数
``` php
bool array_walk ( array &$array , callable $callback [, mixed $userdata = NULL ] )
```
`array_walk`的用途是使用自定义函数对数组中的每个元素做回调处理

---
__参数说明__
* array
输入的数组。

* callback
回调函数，典型情况下`callback`接受两个参数，`array`参数的值作为第一个，键名作为第二个。

上面这个例子中的闭包函数从父作用域中继承父作用域函数的`$tax`变量已经使用引用继承了`&$total`变量，这样在我们在闭包函数中可以直接修改它的值。

# 写在最后（与本文无关的内容）
今天是2019年的第二天，已经差不多有半年没有更新自己的博客了，回首过去的一年，感觉自己过的很是失败，接下来的一年除了要经常更新博客之外，希望自己能够明确好目标，走出生活的舒适区，不负自己！