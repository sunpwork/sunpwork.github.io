---
layout:       post
title:        "认识 JavaScript 中的闭包"
subtitle:     ""
date:         2019-05-28 15:05:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - JavaScript
    - 闭包
    - closure

---
>年前立了个flag说2019年要经常更博，结果三四个月过去了，一篇没更。说来惭愧，最近正在准备web前端面试，发现自己对JavaScript中闭包这个概论不是很了解，到网上看了介绍，这里留个记录。本章节内容来源[思否老哥的文章](https://segmentfault.com/a/1190000006875662#articleHeader3)再加上自己的一些理解。

# 什么是闭包(Closure)
- 闭包是由函数引用其周边状态绑定在一起形成的组合结构，在JavaScript中，闭包在每个函数被创建时形成。
- 闭包能够从一个函数内部访问其外部函数的作用域。
- 要使用闭包，只需要简单地将一个函数定义在另一个函数内部，并将它暴露出来。要暴露一个函数，可以将它返回或者传给其他函数

# 闭包使用
创建闭包最常用方式，就是在一个函数内部创建另一个函数。下面例子中的`closure`就是一个闭包
``` JavaScript
function func(){
    var a=1, b=2;
    return () => a+b;
}
```
闭包的作用域包含着它自己的作用域，以及它的函数的作用域。

# 闭包的注意事项
- 通常，函数的作用域及其所有变量都会在函数执行结束后被销毁。但是，在创建了一个闭包以后，这个函数的作用域就会一直保存到闭包不存在为止。

``` JavaScript
function makeAdder(x){
    return (y) => x+y;
}
var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2)); // 7
console.log(add10(2)); // 12

// 释放闭包
add5 = null;
add10 = null;
```

- 闭包只能获取包含函数中任何变量的最后一个值，这是因为闭包所保存的是整个变量对象，而不是某个特殊的变量
``` JavaScript
function test(){
    var arr = [];
    for(var i=0; i<10; i++){
        arr[i] = () => i;
    }
    for(var a=0; a<10; a++){
        console.log(arr[a]()); // 执行到此处时。i是10，所以输出的全部是10
    }
}
test(); // 连续输出10个10
```

对于上面的情况，我们改变代码如下：
``` JavaScript
function test(){
    var arr = [];
    for(let i=0; i<10; i++){ // var改为let let每次循环都是一个单独的块级作用域
        arr[i] = () => i;
    }
    for(var a=0; a<10; a++){
        console.log(arr[a]());
    }
}
test(); // 输出 0~9
```
至于`let`与`var`的区别，这里我们不做篇幅去讲解了。

- 闭包中的this对象

``` JavaScript
var name = "The Windows";

var obj = {
    name: "My Object",

    getName: function(){
        return this.name
    }
}
console.log(obj.getName()()) // The Window
```

`obj.getName()()`实际上是在全局作用域中调用了匿名函数，`this`指向了window。匿名函数的指向环境具有全局性，因此其`this`对象通常指向window

``` JavaScript
var name = "The Window";

var obj = {
  name: "My Object",
  
  getName: function(){
    var that = this;
    return function(){
      return that.name;
    };
  }
};

console.log(obj.getName()());  // My Object
```

# 闭包的应用

应用闭包的主要场合是：设计私有的变量和方法。任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数外部访问这些变量。私有变量包括函数的参数、局部变量和函数内定义的其他函数。我们把有权限访问私有变量的公有方法称为`特权方法`
``` JavaScript
function Animal(){
    // 私有变量
    var series = "哺乳动物";
    function run(){
        console.log("Run!");
    }

    // 特权方法
    this.getSeries = () => {
        return series;
    }

}
```
下面来介绍两个概念
```
模块模式（The Module Pattern）：为单例创建私有变量和方法。
单例（singleton）：指的是只有一个实例的对象。JavaScript 一般以对象字面量的方式来创建一个单例对象。
```

普通模式创建单例：
``` JavaScript
var singleton = {
  name: "percy",
  speak:function(){
    console.log("speaking!!!");
  },
  getName: function(){
    return this.name;
  }
};
```
模块模式创建单例
``` JavaScript
var singleton = (() => {
  
  // 私有变量
  var age = 22;
  var speak = function(){
    console.log("speaking!!!");
  };
  
  // 特权（或公有）属性和方法
  return {
    name: "percy",
    getAge: function(){
      return age;
    }
  };
})();
```

匿名函数最大的用途就是创建闭包，并且可以构建命名空间，减少全局变量的使用。
``` JavaScript
var ObjEvent = objEvent || {};
(function()){
    var addEvent = function(){
        // some code
    }
    function removeEvent(){
        // some code
    }
    objEvent.addEvent() = addEvent;
    objEvent.removeEvent() = removeEvent;
}();
```

一个闭包计数器
``` JavaScript
var countNumber = (function(){
    var num = 0;
    return function(){
        return ++num;
    }
})();

```

# 闭包的缺陷
- 常驻内存会增大内存使用量，并且使用不当很容易造成内存泄漏
- 如果不是因为某些特殊任务而需要闭包，在没有必要的情况下，在其他函数中创建函数是不明智的，因为闭包对脚本性能具有负面影响，包括处理速度和内存消耗。

