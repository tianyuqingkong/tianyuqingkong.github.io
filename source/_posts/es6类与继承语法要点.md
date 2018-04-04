title: es6类与继承语法要点
date: 2017/11/8
categories:
- javscript
---
## es6类

### es6类与es5类对比
es6和es5的类声明很相似但是也有很多区别：
* 1 es6类声明是没有变量提升和不能重复声明和let一样，也是就说在执行没到达声明前不可以使用
* 2 es6类声明中所有代码自动在严格模式下执行
* 3 es6类声明中的所用的方法都是不可枚举的，在es5中可以使用Object.defineProperty实现
* 4 es6类声明中的所有的方法都缺失内部[[Construct]]属性，也就是说不能当做构造函数使用
* 5 es6类必须使用new调用，否则报错
* 6 不能在es6类声明中修改类名
* 7 es6明确声明class内部只能有静态方法，没有静态属性
<!--more-->
```
//es6 class
class PersonType {
    constructor(name){
        this.name = name;
    }

    static sayClassName(){
        console.log(this.name);// PersonType
    }

    sayName(){
        console.log(this.name);
    }
}
PersonType.family = "chen";

// es5 calss(大部分是es5)
let personType = (function(){
    "use strict";

    // 确保函数是使用new调用的
    const PersonType = function(name){
        if(typeof new.target === "undefined"){
            throw new Error("构造函数必须使用new调用")
        }

        this.name = name;
    };
    
    Object.defineProperty(PersonType,"sayClassName",{
        value: function(){
            // 确保原型上的方法不是被new调用
            if(typeof new.target !== "undefined"){
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name)
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    PersonType.family = "chen";

    Object.defineProperty(PersonType.prototype,"sayName",{
        value: function(){
            // 确保原型上的方法不是被new调用
            if(typeof new.target !== "undefined"){
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name)
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonType
}())
```


### es6的一些新应用

私有方法
```
const sayName = Symbol('sayName');

export default class personType{
    // 共有方法
    sayAge(){
        console.log('18岁');
    }

    // 私有方法
    [sayName](){
        console.log('tianyu')
    }
};
```

存取函数：实现拦截该属性的存取
```
// es5
let CustomHTMLElement = (function() {

    "use strict";

    const CustomHTMLElement = function(element) {

        // 确保函数是被new调用
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.element = element;
    }

    Object.defineProperty(CustomHTMLElement.prototype, "html", {
        enumerable: false,
        configurable: true,
        get: function() {
            return this.element.innerHTML;
        },
        set: function(value) {
            this.element.innerHTML = value;
        }
    });

    return CustomHTMLElement;
}());

// es6
class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get html() {
        return this.element.innerHTML;
    }

    set html(value) {
        this.element.innerHTML = value;
    }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor);   // true
console.log("set" in descriptor);   // true
console.log(descriptor.enumerable); // false
```
这里实现了对原型上html属性的拦截，且这个属性不可枚举,get和set里强制执行严格模式 


## es6继承

### es5和es6继承对比
```
//ES5
function Rectanle(length, width){
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function(){
    return this.length*this.width;
}
function Square(length){
    Rectangle.call(this,length,length);
}

Square.prototype = Object.create(Rectangle.prototype,{
    constructor: {
        value: Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
});

var sqare = new Square(3);

console.log(square.getArea());            // 9
console.log(square instanceof Square);    // true
console.log(square instanceof Rectangle); // true


// ES6
class Rectangle{
    constructor(length, width){
        this.length = length;
        this.width = width;
    }

    getArea(){
        return this.length * this.width;
    }
}

class Square extends Rectangle{
    constructor(lenth){
        //和调用 Ractangle.call(this, length, length)相似
        super(length, length);
    }
}
var square = new Square(3);
console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```
这里的es6通过关键字extends来实现继承，通过在contructor里的super()来调用父类的构造函数。  
表名上es5和es6继承很相似但实际上大有不同：
* es5中的this是先创造子类的实例对象this，然后再将父类的方法和属性添加到this上。而es6则刚好相反。
* es5中一般只实现了子类原型对父类构造函数原型的继承也就是单链继承，es6中不但实现了es5中继承还实现了子类构造函数对父类构造函数的继承，这就是说子类构造函数可以使用父类构造函数上的静态方法，结果上来说是双链继承。
```
console.log( Square.__proto__ === Rectangle); // true
console.log( Square.prototype.__proto__ === Rectangle.prototype ); // true
```

### es6继承注意项

#### constuctor
constructor可以省略：
```
class Square extends Rectangle{
    // no contructor
}

class Square extends Rectangle{
    constructor(...agrs){
        super(...args)
    }
}
```

constructor不省略时：
* super()只能在继承子类constructor里使用
* super()必须调用来创造this，除非直接从constructor中返回一个对象
* 因为super()是用来初始化this的，在super()调用之前不能使用this

#### super关键字
super关键字既可以当做函数使用也可以当做对象使用，这两种情况完全不一样，且在出现关键字super时必须明确super是对象或者函数否则会报错。

super函数：
* 作为父类的构造函数，子类构造函数中一般必须执行一次super函数，具体使用如上
* 在super()执行时内部的this默认指向子类
```
class A{
	constructor(){
		console.log(new.target.name);
	}
}

class B extends A{
	constructor(){
		super();
	}
}
new A(); // A
new B(); // B
```

super对象：
* 在普通方法中，指向父类原型对象；在静态方法中指向父类
```
class A {
  p() {
    return 2;
  }
}
​
class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
​
  p(){
    return 1;
  }
}
​
let b = new B();
```
* 同样super当做对象时this默认指向子类
* super.x = 3 相当于this.x = 3

#### 参考：
* [understanding es6](https://github.com/nzakas/understandinges6/tree/master/manuscript)
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/)








