---
layout: post
title: js中bind、call、apply函数的用法
categories: [programmer]
tags: [js,call]
description: 关于 js 里 bind call apply 的调用方式分析，以及对 js 里对象调用的分析， this 指针的使用技巧
plotly: false
date: 2015-02-27
permalink: /:year/:month/js_bind_call_apply:output_ext
---
\
最近一直在用 js 写游戏服务器，我也接触 js 时间不长，大学的时候用 js 做过一个 H3C 的 web 的项目，然后在腾讯实习的时候用 js 写过一些奇怪的程序，自己也用 js 写过几个的网站。但真正大规模的使用 js 这还是第一次。我也是初生牛犊不怕虎，这次服务器居然抛弃 C++ 和 lua 的正统搭配，而尝试用 nodejs 来写游戏服务器，折腾的自己要死要活的我也是醉了。

<!-- more -->

\
在给我们项目组的其他程序介绍 js 的时候，我准备了很多的内容，但看起来效果不大，果然光讲还是不行的，必须动手。前几天有人问我关于代码里 **call()** 函数的用法，我让他去看书，这里推荐用 js 写服务器的程序猿看 ​​[《javascript编程精粹》](https://www.amazon.cn/JavaScript%E8%AF%AD%E8%A8%80%E7%B2%BE%E7%B2%B9-%E9%81%93%E6%A0%BC%E6%8B%89%E6%96%AF%E2%80%A2%E5%85%8B%E7%BD%97%E5%85%8B%E7%A6%8F%E5%BE%B7/dp/B0097CON2S/ref=sr_1_1?s=books&ie=UTF8&qid=1425039788&sr=1-1&keywords=JavaScript%E7%B2%BE%E7%B2%B9)​​​ 这本书，crockford 大神果然不是盖的。之后我在 ​​[segmentfault](https://segmentfault.com/q/1010000002519489/a-1020000002521688)​​ 上又看到了类似的问题，那边解答之后干脆这里记一笔。

-----

\
首先，关于 js 定义类或对象的方法，请参看 ​​[w3school](https://www.w3school.com.cn/js/pro_js_object_defining.asp) 的这里​​，写的非常详细和清晰，我不再赘言了。

为了介绍 bind、call、apply 这三个函数的用法，不得不介绍 js 里函数的一些设定。关于这部分推荐通读 ​​《javascript编程精粹》​​ 的第四章，这里我所说的在书里都能找到。

关于这三个函数的详细介绍，可以参看 MDN 的文档：[​​bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)​​​、[​​call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)​​​、[​​apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)​​。

\
下面开始搬砖，修改自我之前在 segmentfault 上的答案：

js 里函数调用有 4 种模式：**方法调用**、**正常函数调用**、**构造器函数调用**、**apply/call 调用**。

同时，无论哪种函数调用除了你声明时定义的形参外，还会自动添加 2 个形参，分别是 **this** 和 **arguments**。

**arguments** 不涉及到上述 3 个函数，所以这里只谈 **this**。**this** 的值，在上面 4 中调用模式下，分别会绑定不同的值。分别来说一说：

-----

方法调用：
-----

这个很好理解，函数是一个对象的属性，比如
```js
var a = {    
    v : 0,    
    f : function(xx) {                
        this.v = xx;    
    }
}
a.f(5);
```
这个时候，上面函数里的 **this** 就绑定的是这个对象 **a**。所以 **this.v** 可以取到对象 **a** 的属性 **v**。

-----

正常函数调用：
-----

依然看代码
```js
function f(xx) {        
    this.x = xx;
}
f(5);
```
这个时候，函数 **f** 里的 **this** 绑定的是全局对象，如果是在浏览器运行的解释器中，一般来说是 **window** 对象。所以这里 **this.x** 访问的其实是 **window.x** ，当然，如果 **window** 没有 **x** 属性，那么你这么一写，按照 js 的坑爹语法，就是给 **window** 对象添加了一个 **x** 属性，同时赋值。

-----

构造器函数调用：
-----

构造函数一直是我认为是 js 里最坑爹的部分，因为它和 js 最初设计的基于原型的面向对象实现方式格格不入，就好像是特意为了迎合大家已经被其他基于类的面相对象实现给惯坏了的习惯。

如果你在一个函数前面带上 **new** 关键字来调用，那么 js 会创建一个 **prototype** 属性是此函数的一个新对象，同时在调用这个函数的时候，把 **this** 绑定到这个新对象上。当然 **new** 关键字也会改变 **return** 语句的行为，不过这里就不谈了。看代码
```js
function a(xx) {        
    this.m = xx;
}
var b = new a(5);
```
上面这个函数和正常调用的函数写法上没什么区别，只不过在调用的时候函数名前面加了关键字 **new** 罢了，这么一来，**this** 绑定的就不再是前面讲到的全局对象了，而是这里说的创建的新对象，所以说这种方式其实很危险，因为光看函数，你不会知道这个函数到底是准备拿来当构造函数用的，还是一般函数用的。所以我们可以看到，在 **jslint** 里，它会要求你写的所有构造函数，也就是一旦它发现你用了 **new** 关键字，那么后面那个函数的首字母必须大写，这样通过函数首字母大写的方式来区分，我个人只有一个看法：坑爹：）

-----

apply/call 调用：
-----

我们知道，在 js 里，函数其实也是一个对象，那么函数自然也可以拥有它自己的方法，有点绕，在 js 里，每个函数都有一个公共的 **prototype** —— **Function**，而这个原型自带有好几个属性和方法，其中就有这里困惑的 **bind**、**call**、**apply** 方法。先说 **apply** 方法，它让我们构造一个参数数组传递给函数，同时可以自己来设置 this 的值，这就是它最强大的地方，上面的 3 种函数调用方式，你可以看到，**this** 都是自动绑定的，没办法由你来设，当你想设的时候，就可以用 **apply()** 了。**apply** 函数接收 2 个参数，第一个是传递给这个函数用来绑定 this 的值，第二个是一个参数数组。看代码
```js
function a(xx) {        
    this.b = xx;
}
var o = {};
a.apply(o, [5]);
alert(a.b);    // undefined
alert(o.b);    // 5
```
是不是很神奇，函数 **a** 居然可以给 **o** 加属性值。当然，如果你 **apply** 的第一个参数传递 **null**，那么在函数 **a** 里面 **this** 指针依然会绑定全局对象。

**call()** 方法和 **apply()** 方法很类似，它们的存在都是为了改变 **this** 的绑定，那 **call()** 和 **apply()** 有什么区别呢？就我个人看来，没啥鸟区别。。。开玩笑！刚刚说了，上面 **apply()** 接收两个参数，第一个是绑定 **this** 的值，第二个是一个参数数组，注意它是一个数组，你想传递给这个函数的所有参数都放在数组里，然后 **apply()** 函数会在调用函数时自动帮你把数组展开。而 **call()** 呢，它的第一个参数也是绑定给 **this** 的值，但是后面接受的是不定参数，而不再是一个数组，也就是说你可以像平时给函数传参那样把这些参数一个一个传递。所以如果一定要说有什么区别的话，看起来是这样的

```js
function a(xx, yy) {    
    alert(xx, yy);    
    alert(this);    
    alert(arguments);
}
a.apply(null, [5, 55]);
a.call(null, 5, 55);
```
仅此而已。

-----

bind 函数：
-----

最后再来说 **bind()** 函数，上面讲的无论是 **call()** 也好， **apply()** 也好，都是立马就调用了对应的函数，而 **bind()** 不会， **bind()** 会生成一个新的函数，**bind()** 函数的参数跟 **call()** 一致，第一个参数也是绑定 **this** 的值，后面接受传递给函数的不定参数。 **bind()** 生成的新函数返回后，你想什么时候调就什么时候调，看下代码就明白了
```js
var m = {   
    "x" : 1
};
function foo(y) {   
    alert(this.x + y);
}
foo.apply(m, [5]);
foo.call(m, 5);
var foo1 = foo.bind(m, 5);
foo1();
```

-----

\
末了来个吐槽，你在 js 里想定义一个函数，于是你会这么写:
```js
function jam() {};
```
其实这是 js 里的一种语法糖，它等价于：
```js
var jam = function() {};
```
然后你想执行这个函数，脑洞大开的你会这么写：
```js
function jam() {}();
```
但是这么写就报错了，其实这种写法也不算错，因为它确实是 js 支持的函数表达式，但是同时 js 又规定以 **function** 开头的语句被认为是函数语句，而函数语句后面是肯定不会带 **()** 的，所以才报错，于是聪明的人想出来，加上一对括号就可以了。于是就变成了这样：
```js
(function jam() {}());
```
这样就定义了一个函数同时也执行它，详情参见 [**​​ECMAScript 的 Expression Statement**](https://262.ecma-international.org/5.1/#sec-12.4) 章节​​。