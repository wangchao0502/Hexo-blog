title: 对于javascript关键字this的理解
date: 2015-11-25 21:14:13
tags:
- 前端
- javascript
---

## 前言

今天上班时候突然接到了面试电话，然后在环境很不好的情况下被问到了this问题，之前以为自己理解的还可以，但是当面对真正的考验的时候还是表现的不尽如人意。于是今天下班回家，痛定思痛，重新复习一下javascript中非常重要的基础概念——关键字`this`。

## What is `this`?

简单说this就是函数当前的运行环境，但是难点就在于**运行环境**是可以变的。
例如下面一个例题：

```javascript
var name = "Bob";  
var nameObj ={  
    name : "Tom",  
    showName : function(){  
        alert(this.name);  
    },  
    waitShowName : function(){  
        setTimeout(this.showName, 1000);  
    }  
};  

nameObj.waitShowName();
```

```javascript
var subway={
     name:'1号线',
     speed:0,
     run:function(speed){
         this.speed=speed;  //绑定到对象本身
         function test(speed){
             this.speed=speed+50;//竟然绑定到全局变量了，真是匪夷所思啊
         }
         test(speed);
     }
 };
 subway.run(100);
 console.log(subway.speed);//100
 console.log(speed);//150
```

先把这两道题放着，最后再来分析。


> In JavaScript, as in most object-oriented programming languages, this is a special keyword that is used within methods to refer to the object on which a method is being invoked.
> The this keyword is relative to the execution context, not the declaration context.

> this指向函数执行时的当前对象(或理解为调用函数的对象)。该关键字在Javascript中和执行环境，而非声明环境有关。

对this的定义相当简单，但是我们要知道的是如何确定是那个对象调用了函数，在不同的运行环境下，this到底指向谁？

## Where is `this`?

[Understanding JavaScript’s this keyword](https://javascriptweblog.wordpress.com/2010/08/30/understanding-javascripts-this/)

通过Twitter的Angus Croll的博客，我们来梳理下this在不同上下文中的指向。
<table><tr><th>Execution Context</th><th>Syntax of function call</th><th>Value of this</th></tr><tr><td>Global</td><td>n/a</td><td>global object(e.g. `window`)</td></tr><tr><td>Function</td><td>Method call: `myObject.foo();`</td><td>`myObject`</td></tr><tr><td>Function</td><td>Baseless function call: `foo();`</td><td>global object(e.g. `window`) (`undefined` in strict mode)</td></tr><tr><td>Function</td><td>Using call: `foo.call(context, myArg, myArg)` | `foo.call(context, [myArgs])`</td><td>`context`</td></tr><tr><td>Function</td><td>Constructor with new: `var newFoo = new Foo();`</td><td>the new instance(e.g. `newFoo`)</td></tr><tr><td>Evaluation</td><td>n/a</td><td>value of `this` in parent context</td></tr></table>

### 1. Global context

`this`指向全局对象（浏览器是`window`，nodejs是`global`）

```javascript
alert(this); //window
```

### 2. Function context

至少有4种情况通过函数应用改变this的指向

#### a) 作为方法调用

方法和函数的区别就是，前者能找到一个明确的归属（对象），而后者是全局的
`this`指向调用者

```javascript
var a = {
    b: function() {
        return this;
    }
};

a.b(); //a;
a['b'](); //a;

var c = {};
c.d = a.b;
c.d(); //c
```
#### b) 作为没有归属的方法调用

`this` is the global object (or ｀undefined｀ in strict mode)

```javascript
var a = {
    b: function() {
        return this;
    }
};

var foo = a.b;
foo(); //window

var a = {
    b: function() {
        var c = function() {
            return this;
        };
        return c();
    }
};

a.b(); //window
```

`a.b()`的例子不太好理解，因为当我们执行a.b()的时候，返回内容是c方法的执行结果，但是c并不是谁的属性，他是没有归属的，因此c的this表示全局环境。

```javascript
var a = {
    b: function() {
        return (function() {return this;})();
    }
};

a.b(); //window
```
自执行函数的调用者都是全局环境——window

#### c) 通过Function.prototype.call, Function.prototype.apply调用
call, apply函数可以将`this`指向第一个参数

```javascript
var a = {
    b: function(x1, x2, x3) {
        return this;
    }
};

var d = {};

a.b.call(d, x1, x2, x3);	//d
a.b.apply(d, [x1, x2, x3]); //d
```
#### d) 通过new调用一个构造函数
`this`指向新建对象

```javascript
var A = function() {
    this.toString = function(){return "I'm an A"};
};

new A(); //"I'm an A"
```


### 3. Evaluation context
eval表达式中的`this`和eval执行环境下的`this`指向相同

```javascript
alert(eval('this==window')); //true - (except firebug, see above)

var a = {
    b: function() {
        eval('alert(this==a)');
    }
};

a.b(); //true;
```

## 总结

以上各种情况其实都满足**this指向函数执行时的当前对象**这句话。无论是全局环境还是对象内部调用，我们只要判断在程序执行上下文中，是谁让代码执行的。

比如，在Global context下，函数执行时没有明显的调用对象，那肯定就是window调用的，因为函数的执行离不开context，及一个调用者的作用域。

有些在语句块内执行的函数，虽然我们可以发现它存在于对象内部，看起来就是那个对象调用了它。但是，仔细分析，就发现自执行函数，和代码块内部定义了函数再执行，跟包裹它的对象毫无关系。最简单的方法就是，把这个可以的代码段从这个对象体内拿出来，如果他仍然能执行，证明他就是被global context调用的函数。

apply和call方法其实就是借用了第一个参数的执行环境，执行一个未被这个对象定义过的函数。仔细想想，就会发现，这两个方法就是干着**借窝生蛋**的勾当，而执行上下文仍然在那个窝里。

最后eval实际上和apply，call没什么两样，也是**借窝生蛋**，窝还是原来的窝，只不过功能更强大了，不仅可以生蛋，还可以做任何事情。


