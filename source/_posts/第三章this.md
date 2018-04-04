---
title: this
date: 2017-12-5
categories:
- javascript
---

*  1 [导言](#1)
*  2 [定义](#2)
*  3 [全局环境下的this值](#3)
*  4 [函数中的this值](#4)
    *  1 [引用类型](#4.1)
    *  2  [函数调用与非引用类型](#4.2)
    *  3  [引用类型与this的值为null](#4.3)
    *  4  [构造器函数中的this值](#4.4)
    *  5  [手动设置函数调用时this的值](#4.5)
<!--more-->


## <span id="1">导言</span>

在这篇文章中我们将讨论更多和[执行上下文](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)相关的细节.这次我们讨论的话题为this关键字.  

实践表明,this在不同执行上下文的值常常引起讨论. 

许多的开发者常常认为关键字this在编程语言中只和面向对象编程相关,也就是this只是指向由构造函数创建的对象.在ECMAScript中this概念也的确是这样实现了，但是如我们所见到了,this并不止止于新创建的对象.  

让我们看看this值在ECMAScript中的具体细节.  


## <span id="2">定义</span> 

this是执行上下文的一个属性.它是上下文代码执行时的一个特别对象.  

```
activeExecutionContext = {
    VO: {...},
    this: thisValue
};
```

在这里VO是[变量对象](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/),它在我们先前章节中有讨论.  

this直接和这个上下文的[可执行代码类型](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/#types-of-executable-code)相关.它的值在进入上下文时决定,且在代码运行时不可改变.  

让我们仔细看这些例子.


## <span id='3'>在全局环境下this的值</span>

在这里所有的东西都足够简单.在全局环境下,this的值总是全局对象本身.因此可以直接引用它.  

```
// explicit property definition of
// the global object
this.a = 10; // global.a = 10
console.log(a); // 10
 
// implicit definition via assigning
// to unqualified identifier
b = 20;
console.log(this.b); // 20
 
// also implicit via variable declaration
// because variable object of the global context
// is the global object itself
var c = 30;
console.log(this.c); // 30
```


## <span id="4">在函数中this的值</span>

当this在函数中就变得有趣了,这种情况将更难和引起更多讨论.  

this在这种代码里最主要的特点是它的值是动态绑定到函数的.  

如上面所提到的一样,this值是在进入上下文时决定的.并且在函数代码中this值可以每次都绝对不一样.  

但是,在运行时的this值是不变的.i.e你绝对不可能对它赋予一个新的值,那是因为它不是一个变量值(相反的是,在Python中它被明确定义为self对象,且这个对象可以在运行时改变):  

```
var foo = {x: 10};
 
var bar = {
  x: 20,
  test: function () {
 
    console.log(this === bar); // true
    console.log(this.x); // 20
     
    this = foo; // error, can't change this value
  
    console.log(this.x); // if there wasn't an error, then would be 10, not 20
 
  }
 
};
 
// on entering the context this value is
// determined as "bar" object; why so - will
// be discussed below in detail
 
bar.test(); // true, 20
 
foo.test = bar.test;
 
// however here this value will now refer
// to "foo" – even though we're calling the same function
 
foo.test(); // false, 10
```

所以什么样的因素决定在函数代码中this的值?这里主要有几个因素.  

首先,通常的一个函数调用,this的值由调用者(激活上下文的代码)决定.i.e.在父级上下文中调用函数，this的值由调用表达式的形式决定(换句话说也是就函数如何语法上调用).  

将这个概念记住和理解是很重要的，因为只有这样你才能对this在任何上下文的值都准确知道. 这个概念准确的说是调用表达式的形式(the form of a call expression),i.e.调用函数的方式影响这个上下文this值.  

(我们在一些js文章甚至书上看到,this的值依赖于函数是如何定义的：如果一个全局函数那么this值就设置为全局对象,如果函数是一个对象的方法那么this的值总是设置为这个对象--多么错误的描述).我们继续,我们看到甚至普通全局函数通过不同形式的调用表达式this的也会有不同.  

```
function foo() {
  console.log(this);
}
 
foo(); // global
 
console.log(foo === foo.prototype.constructor); // true
 
// but with another form of the call expression
// of the same function, this value is different
 
foo.prototype.constructor(); // foo.prototype
```

这和对象上定义的方法相似，this的值不总是为那个对象: 

```
var foo = {
  bar: function () {
    console.log(this);
    console.log(this === foo);
  }
};
 
foo.bar(); // foo, true
 
var exampleFunc = foo.bar;
 
console.log(exampleFunc === foo.bar); // true
 
// again with another form of the call expression
// of the same function, we have different this value
 
exampleFunc(); // global, false
```


所以调用表达式是如何影响this的值?为了完全的理解如何影响this值,我们需要了解一种内部类型-- 引用类型( Referece type ).  


### <span id="4.1">引用类型</span>

用伪代码表示引用类型可以把它当成一个拥有两个属性的对象:base(i.e.属性所属与对象)和属性名：  

```
var valueOfReferenceType = {
  base: <base object>,
  propertyName: <property name>
};
```

注意：从ES5开始引用包含属性可以使用"严格(stirct)"--这个标识决定引用是否在严格模式里求值.  

```
'use strict';
 
// Access foo.
foo;
 
// Reference for `foo`.
const fooReference = {
  base: global,
  propertyName: 'foo',
  strict: true,
};
```

为引用类型(Reference type)的值只有两种情况中：

*  1 当处理一个标识符时
*  2 或者处理一个属性访问符  

标识符在标识符求值过程决定. 且具体可以参考[第4章.作用域链](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/),最终会返回一个为引用类型的值(它对this很重要).  

标识符有这些类型：变量名,函数名，函数参数，在全局对象上的不规范属性,如下例:

```
var foo = 10;
function bar() {}
```

作为处理标识符的中间结果引用类型值,如下:  

```
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
var barReference = {
  base: global,
  propertyName: 'bar'
};
```

为了从引用类型的到对象真正的值，还需要经过getValue方法,伪代码如下: 

```
function GetValue(value) {
 
  if (Type(value) != Reference) {
    return value;
  }
 
  var base = GetBase(value);
 
  if (base === null) {
    throw new ReferenceError;
  }
 
  return base.[[Get]](GetPropertyName(value));
```

其中的内部方法[[Get]]返回这个对象属性的真正值,这和从原型链上获取属性的分析相同.  

```
GetValue(fooReference); // 10
GetValue(barReference); // function object "bar"
```

属性访问器也有两种类型：点标记(当提前知道属性名为一个合理的标识符名时),或者方括号标记:  

```
foo.bar();
foo['bar']();
```

在返回的中间计算过程中,我们可以得到也可以得到引用类型的值

```
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
 
GetValue(fooBarReference); // function object "bar"
```

所以在函数上下文中引用类型是如何与this的值相关联的?--重要的时刻来到了.在函数上下文中this值的一般准信规则如下:  

* 在函数上下文中this的值是头调用者提供,并且由当前调用表达式形式决定(函数是如何语法上调用).  

如果在调用圆括号(...)的右边,是一个引用类型的值那么this的值就被设定为引用类型的base对象.  

剩余的其他情况(i.e:和引用类型不一样的值),this值被设置为null.但是因为this的值为null没有任何道理,所以就隐式转为全局对象.  

```
function foo() {
  return this;
}
 
foo(); // global
```

在调用圆括号的左边是一个引用类型值(因为foo是一个标识符):  

```
var fooReference = {
  base: global,
  propertyName: 'foo'
};
```

对应的this值为引用类型的bese对象,type i.e. 全局对象.  

和属性访问器相似的:   

```
var foo = {
  bar: function () {
    return this;
  }
};
 
foo.bar(); // foo
```

我们这个引用类型值的base就是foo对象.  

```
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
```

但是,对同一个函数用另一调用表达式调用那么this的值就不一样了:  

```
var test = foo.bar;
test(); // global
```

因为tests是一个标识符,也就会产生另外的引用类型,它的base对象(全局对象)就会被当做this的值:  

```
var testReference = {
  base: global,
  propertyName: 'test'
};
```

* 注意:在ES5中的严格模式,this的值在不是全局对象而是undefined

现在我们可以准确的知道,为什么同样的函数用不同的调用表达式激活,拥有不同的this值--答案是有不同中间引用类型值:  

```
function foo() {
  console.log(this);
}
 
foo(); // global, because
 
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
console.log(foo === foo.prototype.constructor); // true
 
// another form of the call expression
 
foo.prototype.constructor(); // foo.prototype, because
 
var fooPrototypeConstructorReference = {
  base: foo.prototype,
  propertyName: 'constructor'
};
```

其他this值例子:  

```
function foo() {
  console.log(this.bar);
}
 
var x = {bar: 10};
var y = {bar: 20};
 
x.test = foo;
y.test = foo;
 
x.test(); // 10
y.test(); // 20
```

### <span id="4.2">函数调用与非引用类型</span>

如我们所提到的,在调用圆括号的左边不是一个引用类型值时,this的值自动设置为null,并且最后被设置为全局对象.  

让我们看看下面表达式:  

```
(function () {
  console.log(this); // null => global
})();
```

在上面的情况中,我们的函数对象那个不是引用类型(不是标识符也不是属性访问器),相应的this的值最终被设置为全局对象.  

看一些复杂的例子:  

```
var foo = {
  bar: function () {
    console.log(this);
  }
};
 
foo.bar(); // Reference, OK => foo
(foo.bar)(); // Reference, OK => foo
 
(foo.bar = foo.bar)(); // global?
(false || foo.bar)(); // global?
(foo.bar, foo.bar)(); // global?
```

为什么一个属性访问器,在某些时候this的值不是bese对象而是全局对象?

最后三个调用,在进行某些操作后,调用圆括号的左边已经不是引用类型了.  

第一个调用比较明显就不说了.

第二种情况有一个圆括号运算符,没有什么副作用,调用圆括号的左边任然是引用类型值,所以返回foo.  

第三种情况中,赋值运算符不和圆括号运算符一样,它会调用GetValue方法([11.13.1](http://bclary.com/2004/11/07/#a-11.13.1)的第三步),返回一个函数对象本身(而不是引用类型值)这意味着this的值为设置为null最终设置为全局.  

和第四种和第五种情况相似的--逗号运算符与逻辑或表达式调用GetValue方法,响应的会返回函数本身值而不是引用类型值,this最终设置为全局对象.  

### <span id="4.3">引用类型与this的值为null</span>

这里存在一种情况：当调用圆括号左边是一个引用类型值,但是this值任然全局对象.这是因为引用类型的base对象为一个[激活对象(activation object)](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/#variable-object-in-function-context).   

我们可以在内部函数调用发现这种情况,如我们所知本地变量,内部函数,形参都被存储在函数的一个激活对象中:  

```
function foo() {
  function bar() {
    console.log(this); // global
  }
  bar(); // the same as AO.bar()
}
```

激活对象返回this的值总是null(i.e.伪代码AO.bar()与null.bar()等价).最终this值被转换为全局对象.

如果在一个with语句中,那就不一样了. 这是因为with语句将一个对象添加到作用域链的前端.i.e.这个对象那个在激活对象前.所以对应拥有引用类型值(要么通过标识符要么通过属性访问器).它的base对象就不和激活对象的bese对象相同了.

```
var x = 10;
 
with ({
 
  foo: function () {
    console.log(this.x);
  },
  x: 20
 
}) {
 
  foo(); // 20
 
}
 
// because
 
var  fooReference = {
  base: __withObject,
  propertyName: 'foo'
};

```

和catch从句相似,但是ES5与ES3表现不一致.  

```
try {
  throw function () {
    console.log(this);
  };
} catch (e) {
  e(); // __catchObject - in ES3, global - fixed in ES5
}
 
// on idea
 
var eReference = {
  base: __catchObject,
  propertyName: 'e'
};
 
// but, as this is a bug
// then this value is forced to global
// null => global
 
var eReference = {
  base: global,
  propertyName: 'e'
};
```

同样的也发生在具名函数表达式递归调用中. 

```
(function foo(bar) {
 
  console.log(this);
 
  !bar && foo(1); // "should" be special object, but always (correct) global
 
})(); // global
```

### <span id="4.4">构造器函数中的this值</span>

和this相关的更多是在构造函数中this值.  

```
function A() {
  console.log(this); // newly created object, below - "a" object
  this.x = 10;
}
 
var a = new A();
console.log(a.x); // 10
```

在这种情况,new 操作符会调用函数内部方法[[Construct]],依次,在对象创建好后,调用内部方法[[Call]],所有A函数创建的对象都指向this.  


### <span id="4.5">手动设置函数调用时this的值</span>

在Function.prototype定义了两个可以改变this值的方法，call,apply.  

```
var b = 10;
 
function a(c) {
  console.log(this.b);
  console.log(c);
}
 
a(20); // this === global, this.b == 10, c == 20
 
a.call({b: 20}, 30); // this === {b: 20}, this.b == 20, c == 30
a.apply({b: 30}, [40]) // this === {b: 30}, this.b == 30, c == 40
```