---
title: 词法环境：ES
date: 2018/8/13
tags: ES 
---

# 详解ECMA-262-5 3.2章  词法环境：ECMAScript的实现


## 导言(Intruduction)

在这一章中我们将继续词法环境的考察.在先前的[3.1章](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/#environment-frames-model)我们澄清了关于这一话题的通用理论.特别是我们已经学习了环境与静态作用域以及闭包间的紧密关系.  

我们已经知道ECMAScript使用了环境帧链的模型,在这个章节中我们将着手于词法环境在ECMAScript中的具体实现.特别是我们将讨论ES中使用的和通用理论中相对应结构和术语.  

我们将从定义开始，尽管我们已经在通用理论中给出了词法环境的定义，但是这里我们将给出和ECMA262-5相关的定义.  


<!--more-->
## 定义(Definitions)

正如我们在通用理论中说的，环境是用于管理在逻辑上代码嵌套的数据(变量，函数，等).同样的ECMAScript也是相同的作用.  

* 一个词法环境定义了在词法嵌套结构的代码情况中，标识符与变量值以及函数的关联.  

在ES中的词法环境包含两部分：一是环境记录以及一个对外部环境的引用.I.e.定义中环境和先前模型中讨论的单帧相对应.因此:  

* 一个环境记录表示标识符与创建时词法作用域相的绑定

这就是一个环境记录，它保存着当前上下文显示的变量.  

看下面例子:

```
var x = 10;
 
function foo() {
  var y = 20;
}
```

我们知道这里有连个抽象的环境，它们分别对应着全局上下文和foo函数上下文:  

```
// environment of the global context
 
globalEnvironment = {
 
  environmentRecord: {
 
    // built-ins:
    Object: function,
    Array: function,
    // etc ...
 
    // our bindings:
    x: 10
 
  },
 
  outer: null // no parent environment
 
};
 
// environment of the "foo" function
 
fooEnvironment = {
  environmentRecord: {
    y: 20
  },
  outer: globalEnvironment
};
```

这个outer引用通常用于连接当前环境的父级环境,当然父级函数也有自己的oute连接.并且全局环境的outer指向null.  

全局环境是这个作用域链的顶端.这与ES原型继承很像:如果一个属性没在当前对象找到,那么它就搜寻其原型,接着原型的原型以此类推.直到找到为止,或者找到原型的顶部.环境和它一样：上下文中出现的变量(或标识符)像属性一样,outer连接代表着原型.  

如先前所说的一个词法环境可能包含多个内部词法环境.e.g,如果一个函数包含两个嵌套函数那么每个嵌套函数的outer引用指向相同的外部环境.

```
function foo() {
 
  var x = 10;
 
  function bar() {
    var y = 20;
    console.log(x + y); // 30
  }
 
  function baz() {
    var z = 30;
    console.log(x + y); // 40
  }
 
}
 
//Environments
 
// "foo" environmnet
 
fooEnvironment = {
  environmentRecord: {x: 10},
  outer: globalEnvironment
};
 
// both "bar" and "baz" have the same outer
// environment -- the environment of "foo"
 
barEnvironment = {
  environmentRecord: {y: 20},
  outer: fooEnvironment
};
 
bazEnvironment = {
  environmentRecord: {z: 30},
  outer: fooEnvironment
};
```

ECMAScript 定义了两种类型的环境记,它们主要是为是其实现目的.但是在某些细节上我们认为它们完全相同. 


## 环境记录类型(Environment record types)

在ES5文档中指明了两种类型的环境记录：声明环境记录和对象环境记录  


### 声明环境记录(Declarative environment record)

声明环境记录通常用于出现在函数作用域的变量,函数,形参等等.(这种情况下和es3中活动变量很像)和catch语句.  

如下例：  

```
// all: "a", "b" and "c"
// bindings are bindings of
// a declarative record
 
function foo(a) {
  var b = 10;
  function c() {}
}
```

在catch从句中绑定是是期待参数:  

```
try {
  ...
} catch (e) { // "e" is a binding of a declarative record
  ...
}
```

通常情况下声明记录默认存储在低阶工具中(如虚拟机的寄存器,因此能提供快速访问),这是和原来ES3中的活动对象(activeaction object)主要的不同.  

规范没有要求(甚至间接不建议)声明记录的实现以一种低效率的简单对象.这样的结果导致声明环境不会直接默认暴露给用户级别,这也意味着我们不能像记录的属性一样直接访问这些绑定，同样的甚至在ES3中也不能访问(除非在Rhino实现的利用\_\_parent\_\_访问).  

隐形的,声明记录允许其使用[词法处理](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-35.html#%_sec_5.5.6)技术,这就意味着可以不通过任何作用域链直接获取需要的变量--不管作用域嵌套多深(如果存储是固定的或者不变的,所有的变量处理都可以在编译之前就可知道).但是ES5规范并没直接提到这个事实.  

再说一次,关于使用声明环境记录替代老的活动对象概念一切都是为了实现上的高效.  

因此,如Brendan Eich[提到的](https://mail.mozilla.org/pipermail/es-discuss/2010-April/010915.html)(最后一段)--在ES3中活动对象的实现简直就是一个'a bug'  

理论上,环境的声明记录可以通过这种方式实现(原型的type不是规范里的,这里只是我用作说明)   

```
environment = {
  // storage
  environmentRecord: {
    type: "declarative",
    // storage
  },
  // reference to the parent environment
  outer: <...>
};
```

#### Eval和内部函数也许会破坏优化(Eval and inner functions may break optimizations)  

注意,eval函数可以破坏环境记录的高效，这是因为使用eval的话根本不知道哪些应该绑定.  

比如说V8实现的引擎(我猜其他的也一样)会优化函数如不会创建arguments对象(如果它没在函数体内出现),也不会保存没用的父级变量.这样的函数会更加轻量,它自会保存使用词法变量.I.e如果父级变量没有使用--函数甚至不会有闭包.(截屏自chrome dev-tools)：  
![withoutEval](词法环境ES/withoutEval.png) 

然后相同的函数,但是有一个空的eval调用:

![withEval](词法环境ES/withEval.png)  

因为使用了eval就不能提前知道哪些数据将会被使用,我们将看到所有的"重量东西", argumnets对象,和闭包属性，ie父级环境.  

进一步，看一看后一种情况的outerFn,它同样创建了arguments对象因为拥有内部函数因此很难分析出内部函数是否使用argumnets.  

然而,这只是一种实现，它让我们看到了有哪些优化可以提供，也可以怎么取消这些优化.  
让我们看看第二种环境记录类型--对象环境记录.  


### 对象环境记录(Object environment record)

相反的,对象环境记录被用来定义在全局上下文下的变量，函数以及内部使用with语句.这些东西都以低效的方式存储在一个简单对象来实现.如上面提到的,这种情况的绑定都是以对象属性的形式.  

* 这种存储着绑定的上下文被称作绑定对象  

在全局上下文的情况下,变量和全局对象相关联,正因如此我们才可以像全局对象属性一样引用它们:  

```
var a = 10;
console.log(a); // 10
 
// "this" in the global context
// is the global object itself
console.log(this.a); // 10
 
// "window" is the reference to the
// global object in the browser environment
console.log(window.a); // 10
```

在使用with语句的情况下,变量和with--对象属性相关联:

```
with ({a: 10}) {
  console.log(a); // 10
}
```

每次执行with语句时一个新的词法环境就被创建，这词法环境的环境记录是对象环境记录,而outer环境指向当前运行的环境.然后将当前运行环境替换为with创建的新环境.当with执行完毕当前运行环境又恢复为原来状态.

```
var a = 10;
var b = 20;
 
with ({a: 30}) {
  console.log(a + b); // 50
}
 
console.log(a + b); // 30, restored
```

伪代码如下：  

```
// initial state
context.lexicalEnvironment = {
  environmentRecord: {a: 10, b: 20},
  outer: null
};
 
// "with" executed
previousEnvironment = context.lexicalEnvironment;
 
withEnvironment = {
  environmentRecord: {a: 30},
  outer: context.lexicalEnvironment
};
 
// replace current environment
context.lexicalEnvironment = withEnvironment;
 
// "with" completed, restore the environment back
context.lexicalEnvironment = previousEnvironment;
```

效果上来看和catch从句一样,同样的将当前运行环境替换为新创建的那个环境，这不过这个环境的环境记录为声明环境记录.  

```
var e = 10;
 
try {
  throw 20;
} catch (e) { // replace the environment
  console.log(e); // 20
}
 
// and now it's restored back
console.log(e); // 10
```

下面我将看看这些由with语句和catch从句创建的暂时环境使用函数表达式时的效果.   

因为对象环境记录的低效，在ES5严格模式下with语句被移除掉了.  

更多原因是，with语句在许多场合造成混淆(因为变量和函数声明的提升)并且其中一些真很让人困惑,这也是为什么with在ES5严格模式下去除.  

将全局对象从作用域链的底部移除是下一代ES的计划,全局环境记录也将从对象转为声明,使用modules这样的体系,全局绑定如parseInt,Math等等都将只是引入到全局上下文中,但是技术上来说它们将不再是全局对象的属性了,因为已经没有全局对象了.   

理论上来说,一个拥有对象环境记录的环境,可以以这种方式呈现:  

```
environment = {
  // storage
  environmentRecord: {
    type: "object",
    bindingObject: {
      // storage
    }
  },
  // reference to the parent environment
  outer: <...>
};
```

规范有说明到,bingdingObject是真正对象(e.g. 全局对象)的某种映射,但不是所有在原始对象的属性都被当做绑定对象的属性,比如说,属性名不能为标识符的变量就没有包含在绑定对象上，这是相当符合逻辑的这是因为它们不能被当做变量引用:   

```
// global properties
this['a'] = 10; // included in the binding object
this['hello world'] = 20; // isn't included
 
console.log(a); // 10, can refer
console.log(hello world); // cannot, syntax error
```

然而,如何将原始对象同步的绑定规范并没有具体指出.  


## 执行上下文结构(Structure of execution context)

这里我们将简单提及ES5中执行上下文的结构.它和ES3中的有一点不同,并且有如下属性:  

```
ExecutionContextES5 = {
  ThisBinding: <this value>,
  VariableEnvironment: { ... },
  LexicalEnvironment: { ... },
}
```

我们可以看到上下文中有变量环境和词法环境,这对阅读规范的读者常常造成疑惑，我们将简单的澄清它们，但是这里只是简单注意这是主要是来区分[函数声明](http://dmitrysoshnikov.com/ecmascript/chapter-5-functions/#function-declaration)和[函数表达式](http://dmitrysoshnikov.com/ecmascript/chapter-5-functions/#function-expression)的[[Scope]]值.  

让我们来看一看执行环境的属性.  


### this绑定(This binding) 

This的值现在称为this绑定,然而除了术语改变了其他的从语义上(除了在严格模式上，this可能为undefined)没有什么改变,在全局模式下, this任然绑定的是全局对象本身：  
```
(function (global) {
  global.a = 10;
})(this);
 
console.log(a); // 10
```

并且在函数上下文中this的值仍由函数如何调用所决定.如果他是通过一个[引用](http://dmitrysoshnikov.com/ecmascript/chapter-3-this/#-reference-type)调用,那么this的基本值由引用决定,其他情况--要么全局对象或者在严格模式下的undefined.  

```
var foo = {
  bar: function () {
    console.log(this);
  }
};
 
// --- Reference cases ---
 
// with a reference
foo.bar(); // "this" is "foo" - the base
 
var bar = foo.bar;
 
// with the reference
bar(); // "this" is the global, implicit base
this.bar(); // the same, explicit base, the global
 
// with also but another reference
bar.prototype.constructor(); // "this" is "bar.prototype"
 
// --- non-Reference cases ---
 
(foo.bar = foo.bar)(); // "this" is "global" or "undefined"
(foo.bar || foo.bar)(); // "this" is "global" or "undefined"
(function () { this; })(); // "this" is "global" or "undefined"
```

再次注意,在严格模式,不能通过如下模式获得全局对象:  

```
(function () {
  "use strict";
  var global = (function () { return this; })();
  console.log(global); // undefined!
})();
```

如和在这种情况下(包括间接调用和eval)处理这种技术将在[严格模式](http://dmitrysoshnikov.com/ecmascript/es5-chapter-2-strict-mode/#codethiscode-value-restrictions)章节介绍.  

让我们回到环境来看规范中上下文中的变量环境和词法环境,这两部分常常造成误解并且在一些解释中描述常常不正确.  

### 变量环境(Variable environment) 

变量环境部分存储着初始化变量和函数的上下文.准确的说它的环境存储包含着进入上下文是填充的数据.这和ES3中的[变量对象](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/#variable-object)很像.  

当进入一个函数的上下文时,回想一下这时一个特别的argumnets对象用于表示传入的形参值被创建.在严格模式下arguments对象经历了[很多改变](http://dmitrysoshnikov.com/ecmascript/es5-chapter-2-strict-mode/#codeevalcode-and-codeargumentscode-restrictions),其中arguments将不能通过其属性改变其真正的参数值.还有callee(调用函数本身的引用)将在严格模式下废除.  

在这个代码里:  

```
function foo(a) {
  var b = 20;
}
 
foo(10);
```

理论上我们有如下VariableEnvironment构成foo函数的上下文:  

```
fooContext.VariableEnvironment = {
  environmentRecord: {
    arguments: {0: 10, length: 1, callee: foo},
    a: 10,
    b: 20
  },
  outer: globalEnvironment
};
```

那LexicalEnvironment是如何组成的?好笑的是词法环境初始化只是VariableEnvironment的复制.  


### 词法环境(lexical environment)  

词法环境和变量环境本质上都是的词法环境(不管它们怎么命名)i.e.都是在内部函数创建时静态(词法)捕获外部绑定.  

正如我们先前提到的初始化LexicalEnvironment只是对VariableEnvironment的蓝本复制,看看下面例子:  

```
fooContext.LexicalEnvironment = copy(fooContext.VariableEnvironment);
```

然而接下来会发生什么,在[代码执行阶段](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/#code-execution),提前在原有的词法环境通过with语句和catch从句来扩展(尽管如我们所说,在ES5中提前替换上下文环境，但是在ES3中没有扩展).    

with语句和catch从句如上面所示的在执行时替换上下文环境,并且这种情况和函数表达式有关.  

从[函数创建的规则](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/#rules-of-function-creation-and-application)的讨论中,我们知道闭包将在函数创建时将词法环境保存.  

如果一个函数表达式在一个with语句(或者cacth从句)创建,那么它应该保存(替换)当前词法环境.  

我们是直接将VariableEnvironment替换(而不是它的副本LexicalEnvironment),然后再其with语句执行结束后恢复过来,然而这就意味着FE将不能引用创建时的绑定,但是FE需要with-绑定.  
 
其实我们不会替换VariableEnvironment本身,因为FD也可以在with语句内调用,FD与FE相反的是可以使用初始化的状态,而不是从with-对象(我们将看下面的例子)

* 这就是为什么,当函数声明(FD)时保存VariableEnvironment部分作为其[[Scope]]属性,而函数表达式(ES)保存其LexicalEnvironment部分.这是唯一可以区分它们两的地方.  

这个事实好笑的是在下一代ES或者ES5严格模式with语句将彻底消失，这样ES规范将在这个方面少了很多误解.  

让我们在看一下我们的分析：FE保存LexicalEnvironment,因为它需要在with执行时动态绑定,但是FD将保存VariableEnvironment,因为根据规范FD不能在一个块级内创建，并且它会[提升](http://dmitrysoshnikov.com/notes/note-4-two-words-about-hoisting/)到顶部.
```
var a = 10;
 
// FD
function foo() {
  console.log(a);
}
 
with ({a: 20}) {
 
  // FE
  var bar = function () {
    console.log(a);
  };
 
  foo(); // 10!, from VariableEnvrionment
  bar(); // 20,  from LexicalEnvrionment
 
}
 
foo(); // 10
bar(); // still 20
```

理论上可以如下面: 

```
// "foo" is created
foo.[[Scope]] = globalContext.[[VariableEnvironment]];
 
// "with" is executed
previousEnvironment = globalContext.[[LexicalEnvironment]];
 
globalContext.[[LexicalEnvironment]] = {
  environmentRecord: {a: 20},
  outer: previousEnvironment
};
 
// "bar" is created
bar.[[Scope]] = globalContext.[[LexicalEnvironment]];
 
// "with" is completed, restore the environment
globalContext.[[LexicalEnvironment]] = previousEnvironment;
```

为了从实例中更明显的看清区别,我们可以采用非标准模式在块级内创建FD,根据我们[所记得](http://dmitrysoshnikov.com/ecmascript/chapter-5-functions/#implementations-extension-function-statement)规范将会报异常,但是如今的实现环境没有一个会报错.Firefox的函数语句有自己的非标准方式执行很多年,并且在其他实现环境如v8引擎也遵循下面行为:  

```
var a = 10;
 
with ({a: 20}) {
 
  // FD
  function foo() { // do not test in Firefox!
    console.log(a);
  }
 
  // FE
  var bar = function () {
    console.log(a);
  };
 
  foo(); // 10!, from VariableEnvrionment
  bar(); // 20,  from LexicalEnvrionment
 
}
 
foo(); // 10
bar(); // still 20
```

上面的代码这样运行的原因是因为FD(被提升到顶部)保存VariableEnvironment当做其[[Scope]],而FE保存其LexicalEnvironment(当with或catch执行时将被替换).  

注意：因为ES6标准的块级函数声明，这个例子洪foo函数就保存词法环境


## 标识符求值(Identifier resolution)

* 标识符的求值是判断哪个标识符出现在那个词法环境绑定上  

换句话说,它和变量的[作用域链查询](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/#scope-chain)很像.如上说的也和原型链查询很像,只是将prototype的连接换成了连接着环境的outer连接.  

看如下代码：

```
var a = 10;
 
(function foo() {
 
  var b = 20;
 
  (function bar() {
 
    var c = 30;
    console.log(a + b + c); // 60
     
  })();
 
})();
```

标识符求值过程如a绑定一样的迭代求值. 

```
function resolveIdentifier(lexicalEnvironment, identifier) {
  
  // if it's the final link, and we didn't find
  // anything, we have a case of a reference error
  if (lexicalEnvironment == null) {
    throw ReferenceError(identifier + " is not defined");
  }
  
  // return the binding (reference) if it exists;
  // later we'll be able to get the value from the reference
  if (lexicalEnvironment.hasBinding(identifier)) {
    return new Reference(lexicalEnvironment, identifier);
  }
  
  // else try to find in the parent scope,
  // recursively analyzing the outer environment
  return resolveIdentifier(lexicalEnvironment.outer, identifier);
 
}
 
resolveIdentifier(bar.[[LexicalEnvironment]], "a") ->
 
-- bar.[[LexicalEnvironment]] - not found,
-- bar.[[LexicalEnvironment]].outer (i.e. foo.[[LexicalEnvironment]]) -> not found
-- bar.[[LexicalEnvironment]].outer.outer -> found reference, value 10
```

## 结论

这一章中我们澄清了ECMAScript中词法环境的概念.我们也讲述了为什么要将原来的 变量/激活 对象以及作用域链换成词法环境链--它们大多数的改变是为相关的高效实现.  

我们也看到了词法环境与闭包的相关性(再次注意,ED合FE在ECMScript都会参数闭包),我们再次回忆了this绑定是如何在不同的执行上下文取值.  

此外我们在提到了下一代ES关于环境的计划,如去除全局对象.