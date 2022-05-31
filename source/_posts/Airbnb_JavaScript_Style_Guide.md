title: Aribnb javascript 代码规范
date: 2017/11/10
categories:
- javscript代码规范
---
## 类型(Types)



* 1.1 原始类型：当你获取一个原始类型你应该直接在其值上操作
   * string
   * number
   * boolean
   * null
   * undefined
   * symbol
   ```
   const foo = 1;
   let bar = foo;

   bar = 9;

   console.log(foo, bar); // => 1, 9
   ```
   * Symbols不能完全polyfill,所以在使用应该确定其当前使用的环境原生支持
   <!--more-->

* 1.2 复杂类型：当你获取一个复杂类型时你应该通过其引用来操作其值
   * object
   * array
   * function
   ```
   const foo = [1, 2];
   const bar = foo;

   bar[0] = 9;

   console.log(foo[0], bar[0]); // => 9, 9
   ```


## 参数(References)

* 2.1 对所有参数使用const;避免使用var  eslint：prefer-const，no-const-assign
    > 为什么? 这是避免你对你的参数赋值，参数赋值可以导致许多bug并且难于理解
    ```
    // bad 
    var a = 1;
    var b = 2;

    // good
    const a = 1;
    const b = 2;
    ```

* 2.2 如果你一定要重新对你的参数再次赋值，可以使用let代替var eslit: no-var jscs: disallowVar
    > 为什么? let拥有块级作用而var拥有函数作用域
    ```
    // bad
    var count = 1;
    if(true){
        count += 1;
    }

    // good, use the let.
    let count = 1;
    if(true){
        count += 1;
    }
    ```

* 2.3 值的注意的是let和const都是块级作用域
    ```
    // const和let只存在于定义的块
    {
        let a = 1;
        const b = 1;
    }
    console.log(a); // ReferenceError
    console.log(b); // ReferenceError
    ```


## 对象(Objects)

* 3.1 使用字面量语法创建对象 eslint: no-new-object
    ```
    // bad
    const item = new Object();

    // good
    const item = {};
    ```

3.2 当创建的对象拥有动态属性名时使用计算属性名称
    > 为什么? 这样允许你在统一的一个地方定义对象属性
    ```
    function getKey(k){
        return `a key named ${k}`;
    }

    // bad
    const obj = {
        id: 5,
        name: 'San Francisco'
    };
    obj[getKey('endabled')] = true;

    // good
    const obj = {
        id: 5,
        name: 'San Francisco',
        [getKey('enabled')]: true
    };
    ```

* 3.3 使用对象方法简写 eslint: object-shorthand 
    ```
    // bad
    const atom = {
        value: 1,

        addValue： function(value){
            return atom.value + value
        }
    };

    // good
    const atom = {
        value: 1,
        addValue(value){
            return atom.value + value;
        }
    };
    ```

* 3.4 使用属性值简写 eslint: object-shorthand
    > 为什么? 对于书写和描述更加短小
    ```
    const lukeSkywalker = 'Luke Skywalker';

    // bad 
    const obj = {
        lukeSkywalker: lukeSkywalker
    };

    // good
    const obj = {
        lukeSkywalker
    };
    ```

*  3.5 将简写属性统一放在对象声明的开头
    > 为什么? 这样更容易区分哪些属性是使用的简写
    ```
    const anakinSkywalker = 'Anakin Skywalker';
    const lukeSkywalker = 'Luke Skywalker';

    // bad
    const obj = {
    episodeOne: 1,
    twoJediWalkIntoACantina: 2,
    lukeSkywalker,
    episodeThree: 3,
    mayTheFourth: 4,
    anakinSkywalker,
    };

    // good
    const obj = {
    lukeSkywalker,
    anakinSkywalker,
    episodeOne: 1,
    twoJediWalkIntoACantina: 2,
    episodeThree: 3,
    mayTheFourth: 4,
    };
    ```

* 3.6 只有当非法标识符做属性时才使用引号 eslint: quote-props
    > 为什么? 通常这样主观上更易阅读,它改进了语法突显,并且也更加容易被许多js引擎优化
    ```
    // bad
    const bad = {
    'foo': 3,
    'bar': 4,
    'data-blah': 5,
    };

    // good
    const good = {
    foo: 3,
    bar: 4,
    'data-blah': 5,
    };
    ```

* 3.7 不要直接使用Object.prototype上的方法,如hasOwnProperty,propertyIsEnumerable,isPrototypeOf  
    > 为什么? 这些方法有可能被对象实例上的属性给覆盖掉 如问题中的{hasOwnProperty: false}或者这个对象也许是个null对象(Object.create(null))
    ```
    // bad
    console.log(object.hasOwnProperty(key));

    // good
    console.log(Object.prototype.hasOwnProperty.call(object, key));

    // best
    const has = Object.prototype.hasOwnProperty; // cache the lookup once, in module scope.
    /* or */
    import has from 'has';
    // ...
    console.log(has.call(object, key));
    ```

* 3.8 在覆盖-拷贝对象时应该利用展开运算符(...)而不是Object.assign 为了得到一个已删减属性的新对象应该使用对象解构运算符
    ```
    // very bad
    const original = {a:1, b: 2};
    const copy = Object.assign(origin,{c:3});// 这个改变了 `original` ಠ_ಠ

    // bad 
    const original = {a:1, b: 2};
    const copy = Object.assign({}, original, {c:3}); // copy =>{a:1,b:2,c:3}

    //good
    const original = {a:1, b:2};
    const copy = {...original,c:3}; // copy =>{a:1, b:2, c:3}

    const {a,...noA} = copy; // noA => {b:2, c:3}
    ```


## 数组(Arrays)

* 4.1 使用字面量语法创建数组. eslint: no-array-constuctor
    ```
    // bad
    const items = new Array();

    // good
    const items = [];
    ```

* 4.2 使用数组的push方法添加项而不是直接添加
    ```
    const someStack = [];

    // bad
    someStack[someStack.length] = 'abc';

    // good
    someStack.push('abc');
    ```

* 4.3 使用数组的扩展运算符...来复制数组
    ```
    // bad
    const len = items.length;
    const itemsCopy = [];
    let i;

    for (i = 0; i < len; i += 1) {
    itemsCopy[i] = items[i];
    }

    // good
    const itemsCopy = [...items];
    ```

* 4.4 使用...将类数组转换为数组而不是Array.form
    ```
    const foo = document.querySelectorAll('.foo');

    // good
    const nodes = Array.from(foo);

    // best
    const nodes = [...foo];
    ```

* 4.5 Array.from 而不是展开运算符...来遍历执行方法 ,因为它避免了创建一个中间数组(这里是因为使用map时从来没被赋过值或者使用 delete 删除的索引则不会调用函数。)
    ```
    // bad 
    const baz = [...foo].map(bar);

    // good
    const baz = Array.form(foo, bar);

    //译者添加
    Array.from([1,,2],(i)=>{return i*2;}) // => [2, NaN, 4]

    [1,,2].map((i)=>{return i*2;}) // => [2, empty × 1, 4]
    ```

* 4.6 在数组的回调函数中返回声明.当函数体只包含一个声明语句且是没有副作用的表达式是可以省略return语句的。
    ```
    // good
    [1, 2, 3].map((x) => {
    const y = x + 1;
    return x * y;
    });

    // good
    [1, 2, 3].map(x => x + 1);

    // bad - no returned value means `memo` becomes undefined after the first iteration
    [[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
    const flatten = memo.concat(item);
    memo[index] = flatten;
    });

    // good
    [[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
    const flatten = memo.concat(item);
    memo[index] = flatten;
    return flatten;
    });

    // bad
    inbox.filter((msg) => {
    const { subject, author } = msg;
    if (subject === 'Mockingbird') {
        return author === 'Harper Lee';
    } else {
        return false;
    }
    });

    // good
    inbox.filter((msg) => {
    const { subject, author } = msg;
    if (subject === 'Mockingbird') {
        return author === 'Harper Lee';
    }

    return false;
    });
    ```

* 4.7 如果一个数组拥有多行，在数组的开始括弧后和在结束括弧前加换行符。
    ```
    // bad 
    const arr = [
        [0, 1], [2, 3], [4, 5]
    ];

    const objectInArray = [{
        id: 2,
    },{
        id:2
    }];

    const numberInArray = [
        1,2
    ];

    // good
    const arr = [[0, 1], [2, 3], [4, 5]];

    const ObjectInArray = [
        {
            id: 1,
        },
        {
            id: 2,
        }
    ];
    ```


## 解构(Destructuring)

* 5.1 当使用多属性对象赋值时应该使用对象解构
    > 为什么? 解构可以让你节约掉为那些属性创建临时变量
    ```
    // bad
    function getFullName(user){
        const firstName = user.firstName;
        const lastName = user.lastName;

        return `${firstName} ${lastName}`;
    }

    // good
    function getFullName(user){
        const { firstName, lastName} = user;
        return `${firstName} ${lastName}`;
    }

    // best
    function getFullName({firstName, lastName}){
        return `${firstName} ${lastName}`;
    }
    ```

* 5.2 使用数组解构. eslint: prefer-destrcuturing
    ```
    const arr = [1, 2, 3, 4];

    // bad
    const first = arr[0];
    const second = arr[1];

    // good
    const [first, second] = arr;
    ```

* 5.3 当多个值返回时使用对象解构而不是数组解构
    > 为什么? 你可以随着时间添加一个属性或者改变参数顺序而不用改变原来的调用
    ```
    // bad
    function processInput(input){
         // then a miracle occurs
        return [left, right, top, bottom];
    }

    // 调用者需要知道返回数据的顺序
    const [left, __, top] = pocessInput(input);

    // good
    function precessInput(input){
        // then a miracle occurs
        return {left, right, top, bottom};
    }

    // 调用者只需要选择他需要的数据
    const { left, top } = processInput(input);
    ```

## 字符串(Stirngs)

* 6.1 使用单引号'' eslint: quotes
    ```
    // bad
    const name = "Capt. Janeway";

    // bad - template literals should contain interpolation or newlines
    const name = `Capt. Janeway`;

    // good
    const name = 'Capt. Janeway';
    ```

* 6.2 当字符超过100字符不应该换行连接字符
    > 为什么?换行字符对于代码搜索性不有利
    ```
    // bad
    const errorMessage = 'This is a super long error that was thrown because \
    of Batman. When you stop to think about how Batman had anything to do \
    with this, you would get nowhere \
    fast.';

    // bad
    const errorMessage = 'This is a super long error that was thrown because ' +
    'of Batman. When you stop to think about how Batman had anything to do ' +
    'with this, you would get nowhere fast.';

    // good
    const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
    ```

* 6.3 当编程式的构建字符串时使用字符串模板而不是使用连接 eslint: prefer-template template-curly-spacing
    > 为什么? 字符串模板可读性高，同时对于换行字符串以及插入特性语法简洁
    ```
    // bad
    function sayHi(name) {
        return 'How are you, ' + name + '?';
    }

    // bad
    function sayHi(name) {
        return ['How are you, ', name, '?'].join();
    }

    // bad
    function sayHi(name) {
        return `How are you, ${ name }?`;
    }

    // good
    function sayHi(name) {
        return `How are you, ${name}?`;
    }
    ```

* 6.4 对字符串永远不要用eval(),这里面太多不稳定性 eslint: no-eval

* 6.5 不要添加多余的字符串转义 eslint:no-useless-escape
    > 为什么? 反斜杠破坏可读性，因此不要使用多余的转义
    ```
    // bad
    const foo = '\'this\' \i\s \"quoted\"';

    // good
    const foo = '\'this\' is "quoted"';
    const foo = `my name is '${name}'`;
    ```


## 函数(Functions)

* 7.1 使用命名函数表达式而不是函数声明. eslint: func-style
    > 为什么? 函数声明是要提升的，这就意味着函数相当容易在其定义前引用.这对可读性和可维护性是有害的.如果你发现函数的定义庞大或复杂到理解剩余文件内容时,或许这时是时候将定义提取到自己的模块中(module).不要忘记这个函数表达式明确的名字，不管这个名字是否可以从表达式包含变量中推断.这样会消除任何对于错误调用栈的猜想.
    ```
    // bad
    function foo(){
        // ...
    }

    // bad 
    const foo = function(){
        // ...
    }

    // good
    // 词法名字和变量引用相互区分
    const short = function longUniqueMoreDescriptivelLexicalFoo(
        \\ ...
    )
    ```

* 7.2 用圆括号将立即执行函数表达式包含. eslint: wrap-iife  
    > 为什么? 一个立即执行函数表达式是一个单独的单元加上执行双括号都用空号包裹起来，干净的表达. 值的注意的是在一个拥有module的世界里，你几乎不需要立即执行表达式
    ```
    // immediately-invoked function expression (IIFE)
    (function () {
    console.log('Welcome to the Internet. Please follow me.');
    }());
    ```
* 7.3 永远不要在非函数块级(if,while等)声明一个函数,而是将函数赋值给一个变量.浏览器允许你这样做,但是它们表现并不一致,这是很难接受的. eslint: no-loop-func

* 7.4 注意：ECMA-262将块定义为一些列的声明(satetments).一个函数声明(declaration)不是这种声明(statement)
    ```
    // bad
    if(currentUser){
        function test(){
            console.log('Nope.');
        }
    }

    // good
    let test;
    if(currentUser){
        test = ()=>{
            console.log('Yup.')；
        }
    }
    ```

* 7.5 不要声明一个arguments函数参数,这样会覆盖函数中原有的arguments对象
    ```
    // bad
    function foo(name, options, arguments) {
        // ...
    }

    // good
    function foo(name, options, args) {
        // ...
    }
    ```

* 7.6 不要使用arguments,最好使用... eslint:prefer-rest-params
    > 为什么? ...是明确指明你想要传入的参数.其次剩余参数是一个真正数组，而不像arguments是一个类数组.

    ```
    // bad
    function concatenateAll(){
        const args = Array.prototype.slice.call(arguments);
        return args.join('');
    }

    // good
    function concatenateAll(...args){
        return args.join('']);
    }
    ```

* 7.7 使用默认参数语法而不是改变函数的参数
    ```
    // 真正的糟糕
    function hanleThings(opts){
        // 不要改变函数的参数
        // 更加糟糕: 如果opts是falsy这里将会引入一个微妙的bug
        opts = opts || {}
    }

    // still bad
    function hanldeThings(opts){
        if(opts === void 0){
            opts = {};
        }
        // ...
    }

    // good 
    function handleThings(opts = {}){
        // ...
    }
    ```

* 7.8 避免默认参数有副作用
    > 为什么? 这看起来很疑惑
    ```
    var b = 1;
    // bad
    function count(a = b++) {
    console.log(a);
    }
    count();  // 1
    count();  // 2
    count(3); // 3
    count();  // 3
    ```

* 7.9 一直讲默认参数放在最后
    ```
    // bad
    function hanleThings(opts = {}, name){
        // ....
    }

    // good
    function hanleThings(name, opts = {}){
        // ...
    }
    ```

* 7.10 不要使用函数构造函数创建新函数 eslint:no-new-func
    > 为什么?用这种方式创建函数和使用eval()对字符串求值一样，同样的风险很大.
    ```
    // bad
    var add = new Function('a', 'b', 'return a + b');

    // still bad
    var subtract = Function('a', 'b', 'return a - b');
    ```

* 7.11 在函数签名间加空格. eslint:sapce-before-function-parent
    > 为什么? 一致性总是好的,并且在添加或者去掉函数名时不应该去掉空格
    ```
    // bad
    const f = function(){};
    const g = function (){};
    const h = function() {};

    // good
    const x = function () {};
    const y = function a() {};
    ```

* 7.13 不要改变(mutate)参数. eslint:no-param-reassign
    > 为什么? 改变传入参数对象会对原调用者造成意想不到的变量副作用
    ```
    // bad 
    function f1(obj){
        obj.key = 1;
    }

    // good 
    function f2(obj){
        const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
    }
    ```

* 7.13 不要对参数再次赋值 eslint: no-param-reassign
    > 为什么? 
    ```
    // bad 
    function f1(a){
        a = 1;
    }

    function f2(a){
        if(!a) { a = 1; }
        // ...
    }

    // good f3(a){
        const b = a || 1;
        //...
    }

    function f4(a = 1){
        // ...
    }
    ```

* 7.14 对于可变参数函数使用扩展操作符...
    > 为什么? 因为简洁,你不需要绑定内容上下文(context),并且你将new和apply结合使用也不容易
    ```
    // bad
    const x = [1, 2, 3, 4, 5];
    console.log.apply(console, x);

    // good
    const x = [1, 2, 3, 4, 5];
    console.log(...x);

    // bad
    new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]));

    // good
    new Date(...[2016, 8, 5]);
    ```

* 7.15 对于多行函数签名，和本文档中保持一致
    ```
    // bad
    function foo(bar,
                baz,
                quux) {
    // ...
    }

    // good
    function foo(
        bar,
        baz,
        quux,
    ) {
    // ...
    }

    // bad
    console.log(foo,
        bar,
        baz);

    // good
    console.log(
        foo,
        bar,
        baz,
    );
    ```


## 箭头函数

* 8.1 当使用匿名函数时(同样传入行内回调函数时)使用箭头函数 eslint: prefer-arrow-callback
    > 为什么? 箭头函数创建的执行上下文(this)通常是你想要的,并且语法更加简洁

    > 什么时候不? 当你有一个足够复杂函数时,你也许该将逻辑移到一个具名函数表达式中.
    ```
    // bad 
    [1, 2, 3].map(function(x){
        const y = x + 1;
        return x * y;
    })

    // good
    [1, 2, 3].map((x) => {
        const y = x + 1;
        return x * y;
    })
    ```

* 8.2 如果函数体是一个单行表达式且返回值为一个没有副作用的表达式,这时省略掉括号同时使用隐式返回
    > 为什么? 语法糖.当多个函数链式调用时可读性高.
    ```
    // bad 
    [1, 2, 3].map(number =>{
        const nextNumber = number + 1;
      
    });

    // good 
    [1, 2, 3].map(number => `A string containing the ${nextNumber}`);

    // good
    [1, 2, 3].map((number, index) => ({
        [index]: number,
    }));

    // No implicit return with side effects
    function foo(callback) {
        const val = callback();
        if (val === true) {
            // Do something if callback returns true
        }
    }

    let bool = false;

    // bad
    foo(() => bool = true);

    // good
    foo(() => {
        bool = true;
    });
    ```

* 8.3 当表达式为多行是,使用圆括号包裹起来可读性更好
    > 为什么? 这样可以明显看出函数从哪里开始和结束
    ```
    // bad
    ['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
            httpMagicObjectWithAVeryLongName,
            httpMethod,
        )
    );

    // good
    ['get', 'post', 'put'].map(httpMethod => (
        Object.prototype.hasOwnProperty.call(
            httpMagicObjectWithAVeryLongName,
            httpMethod,
        )
    ));
    ```

* 8.4 如果箭头函数只有一个参数并且没有大括号就可以不要是有圆括号，将括号省略掉,否则为了清晰和一致性一直将参数放在括号里. 注意:也可以一直将参数放在括号里。
```
// bad
[1, 2, 3].map((x) => x * x);

// good 
[1, 2, 3].map(x => x * x);

// good 
[1, 2, 3].map(number => (
  `A long string with the ${number}. It’s so long that we don’t want it to take up space on the .map line!`
));

// bad 
[1, 2, 3].map( x => {
    const y = x + 1;
    return x * y;
});

// good 
[1, 2, 3].map((x) =>{
    const y = x + 1;
    return x * y;
});
```

* 8.5 避免箭头函数(=>)和比较符号(<=,>=)的混淆. eslint: no-confusing-arrow
```
// bad
const itemHeight = item => item.height > 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) => item.height > 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = item => (item.height > 256 ? item.largeSize : item.smallSize);

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item;
  return height > 256 ? largeSize : smallSize;
};
```


## 类和构造函数(Classes & Constructors)

* 9.1 避免直接操作prototype而应该使用class语法
```
//bad 
function Queue( contents = [] ){
    this.queue = [...contents];
}

Queue.prototype.pop = function(){
    onst value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
}


// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
```

* 9.2 使用extends来继承
    > 为什么? 内置的继承方法原型链没有打断instanceof使用
    ```
    // bad
    const inherits = require('inherits');
    function PeekableQueue(contents) {
        Queue.apply(this, contents);
    }
    inherits(PeekableQueue, Queue);
        PeekableQueue.prototype.peek = function () {
         return this.queue[0];
    };

    // good
    class PeekableQueue extends Queue {
        peek() {
            return this.queue[0];
        }
    }
    ```

* 9.3 方法返回this来帮助构造方法调用链
    ```
    // bad
    Jedi.prototype.jump = function () {
        this.jumping = true;
        return true;
    };

    Jedi.prototype.setHeight = function (height) {
        this.height = height;
    };

    const luke = new Jedi();
    luke.jump(); // => true
    luke.setHeight(20); // => undefined

    // good
    class Jedi {
    jump() {
        this.jumping = true;
        return this;
    }

    setHeight(height) {
        this.height = height;
        return this;
    }
    }

    const luke = new Jedi();

    luke.jump()
    .setHeight(20);
    ```
* 9.4 可以写自定义的toString方法，只要保证它能正常工作并且没有副作用.
    ```
    class Jedi {
        constructor(options = {}) {
            this.name = options.name || 'no name';
        }

        getName() {
            return this.name;
        }

        toString() {
            return `Jedi - ${this.getName()}`;
        }
    }
    ```

* 9.5 如果没有指定构造函数类都有一个默认的构造函数，一个空构造函数或者这个构造函数只是委托父级构造函数，这些都是不必要的。
    ```
    // bad
    class Jedi{
        constructor() {}

        getName() {
            return this.name;
        }
    }

    // bad 
    class Reg extends Jedi{
        constructor(...args){
            super(...args);
        }

        // good 
        class Reg extends Jedi{
            constructor(...args){
                super(...args);
                this.name = 'Rey'
            }
        }
    }
    ```

* 9.6 避免使用重复方法名. eslint: bo-dupe-class-members
    > 为什么? 重复的类方法名，会导致先前声明的类名失效
    ```
    // bad
    class Foo {
        bar() { return 1; }
        bar() { return 2; }
    }

    // good
    class Foo {
        bar() { return 1; }
    }

    // good
    class Foo {
        bar() { return 2; }
    }
    ```

## 模块(Modules)

* 10.1 总是使用modules(import/export)而不是非标准模块系统
    > 为什么? moduels是未来，不如现在就使用
    ```
    // bad
    const AirbnbStyleGuide = require('./AirbnbStyleGuide');
    module.exports = AirbnbStyleGuide.es6;

    // ok
    import AirbnbStyleGuide from './AirbnbStyleGuide';
    export default AirbnbStyleGuide.es6;

    // best
    import { es6 } from './AirbnbStyleGuide';
    export default es6;
    ```

* 10.2 不要使用通配符imports
    >为什么?这可以保证你只有一个default exports(不太理解~\(≧▽≦)/~)
    ```
    // bad
    import * as AirbnbStyleGuide from './AirbnbStyleGuide';

    // good
    import AirbnbStyleGuide from './AirbnbStyleGuide';

    10.3 And do not export directly from an import.
    ```

* 10.3 不要直接从一个引入导出
    > 为什么? 保持一致
    ```
    // bad
    // filename es6.js
    export { es6 as default } from './AirbnbStyleGuide';

    // good
    // filename es6.js
    import { es6 } from './AirbnbStyleGuide';
    export default es6;
    ```

* 10.4 从一个地方引入的都放在同一个地方. eslint：no-duplicate-imports
    > 为什么? 保存可维护性
    ```
    // bad
    import foo from 'foo';
    // … some other imports … //
    import { named1, named2 } from 'foo';

    // good
    import foo, { named1, named2 } from 'foo';

    // good
    import foo, {
    named1,
    named2,
    } from 'foo'
    ```
* 10.5 不要导出可以改变的绑定. eslint: import/no-mutable-exports
    >为什么? 异变(mutation)一般都是要避免的，除非在特殊的情况需要，通常只有constant引用才能被导出.
    ```
    // let foo = 3;
    export {foo};

    // good 
    const foo = 3;
    export { foo };
    ```

* 10.6 如果modles里只有一个导出，推荐使用default导出而不是具名导出. eslint:import/prefer-default-export.
    > 为什么? 为了鼓励更多的文件值导出一个模块，这样便于阅读和维护.
    ```
    // bad 
    export function foo() {}

    // good
    export default function foo() {}
    ```

* 10.7 将所有的import语句放在非import语句前. eslint: import/first
    > 为什么? 既然imports都是会提升的(hosited),就都将他们放在顶部避免一些意外的表现行为.
    ```
    // bad
    import foo from 'foo';
    foo.init();

    import bar from 'bar';

    // good
    import foo from 'foo';
    import bar from 'bar';

    foo.init();
    ```

* 10.8 跨行imports应该缩进，这和跨行数组与对象字面量一样.
    ```
    // bad
    import {longNameA, longNameB, longNameC, longNameD, longNameE} from 'path';

    // good
    import {
    longNameA,
    longNameB,
    longNameC,
    longNameD,
    longNameE,
    } from 'path';
    ```

* 10.9 不允许webpack的loader语法也在模块的import语句里. eslint: import/no-wabpack-loader-syntax
    > 为什么?为什么不讲这些语法写在webpack.config.js.
    ```
    // bad
    import fooSass from 'css!sass!foo.scss';
    import barCss from 'style!css!bar.css';

    // good
    import fooSass from 'foo.scss';
    import barCss from 'bar.css';
    ```


## 迭代器和生成器(Iterators and Generators)

* 11.1 不要使用迭代器.应该使用高阶函数来代替循环(for-in或者for-of). eslint:no-iterator no-restricted-stntax
    > 为什么? 这可以强迫我们使用不异变原则(immutable).处理纯函数返回的值相对于有副作用的更容易解释.

    > 使用 map()/ every()/ filter()/ find()/ findIndex()/ reduce()/ some()/... 来遍历数组,同时使用Object.keys()/ Object.value()/ Object.entries()来产生数组让你可以遍历对象.
    ```
    const numbers = [1, 2, 3, 4, 5];

    // bad
    let (let num of numbers){
        sum += num;
    }
    sum === 15;

    // good
    let sum = 0;
    numbers.forEach((num) =>{
        sum += num
    });

    sum === 15;

    // best(使用函数式强制)
    const sum = numbers.reduce((total, num) => tatal + num, 0);

    sum === 15;

    // bad 
    const increaseByOne = [];
    for(let i = 0; i < numbers.length; i++){
        increaseByOne.push(numbers[i] + 1);
    }

    // good
    const increasedByOne = [];
    numbers.forEach((num) =>{
        increasedByOne.push(num + 1);
    });

    // best (保持函数式)
    const increasdByOne = numbers.map(num => num + 1);
    ```

* 11.2 目前不要使用迭代器(generators)
    > 为什么? 它们目前还不能很好的转义到ES5

* 11.3 如果你一定要使用迭代器,或者你不赞成我们的建议，请确定函数标识符正确的空格化. eslint: generator-star-spacing
    > 为什么? function和*两者是同一个概念的关键字, *不是来修饰function, function*是一个独一无二的构造器，这个构造器和普通的functtion不同.
    ```
    // bad
    function * foo() {
        // ...
    }

    // bad
    const bar = function * () {
        // ...
    };

    // bad
    const baz = function *() {
        // ...
    };

    // bad
    const quux = function*() {
        // ...
    };

    // bad
    function*foo() {
        // ...
    }

    // bad
    function *foo() {
        // ...
    }

    // very bad
    function
    *
    foo() {
        // ...
    }

    // very bad
    const wat = function
    *
    () {
        // ...
    };

    // good
    function* foo() {
        // ...
    }

    // good
    const foo = function* () {
        // ...
    };
    ```


## 属性(Properties)

* 12.1 使用点标记符来获取属性, eslint: dot-notation
    ```
    const luke = {
        jedi: true,
        age: 28,
    };

    // bad
    const isJedi = luke['jedi'];

    // good
    const isJedi = luke.jedi;
    ```

* 12.2 当使用一个变量来访问属性时使用[]
    ```
    const luke = {
        jedi: true,
        age: 28,
    };

    function getProp(prop) {
        return luke[prop];
    }

    const isJedi = getProp('jedi');
    ```
* 12.3 幂计算使用**, eslint: no-restricted-properties
    ```
    // bad
    const binary = Math.pow(2, 10);

    // good
    const binary = 2 ** 10;
    ```


##  变量(Variables)

* 13.1 声明变量时使用const或let，如果不这样做将可能导致全局变量,一般我们都避免污染全局作用域空间, eslint: no-undef prefer-const
    ```
    // bad
    superPower = new SuperPower();

    // good
    const superPower = new SuperPower();
    ```

* 13.2 为每个声明的变量单独使用const或let. eslit: one-var
    > 为什么? 这样更加容易添加新声明变量,同时你也不用担心;和,的位置写反了,在打断点debugger时更方便分别检查每个变量.
    ```
    // bad
    const items = getItems(),
        goSportsTeam = true,
        dragonball = 'z';

    // bad
    // (compare to above, and try to spot the mistake)
    const items = getItems(),
        goSportsTeam = true;
        dragonball = 'z';

    // good
    const items = getItems();
    const goSportsTeam = true;
    const dragonball = 'z';
    ```

* 13.3 将你所有的const声明集合在一起写，同样let也是一样
    > 为什么? 当你想再次赋值时可能更加方便
    ```
    // bad
    let i, len, dragonball,
        items = getItems(),
        goSportsTeam = true;

    // bad
    let i;
    const items = getItems();
    let dragonball;
    const goSportsTeam = true;
    let len;

    // good
    const goSportsTeam = true;
    const items = getItems();
    let dragonball;
    let i;
    let length;
    ```

* 13.4 变量在你需要的时候对它们赋值，但是要把它们放在合理的地方
    ```
    // bad 不必要的函数调用
    function checkName(hasName){
        const name = getName();

        if(hasName === 'test'){
            return false;
        }

        if(name === 'test'){
            this.setName('');
            return false;
        }

        return false;
    }

    // good 
    function checkName(hasName) {
        if (hasName === 'test'){
            return false;
        }

        const name = getName();
        if(name === 'test'){
            this.setName('');
            return false;
        }

        return name;
    }
    ```

* 13.5 不要将变量链式赋值. eslint: no-multi-assign
    > 为什么? 变量链式赋值会造成隐式全局变量
    ```
    // bad
    (function example(){
        //js 这样解析
        // let a = ( b = ( c = 1 ) );
        //let关键字只用到了a上其余都成为了全局变量
        let a = b = c = 1;
    }());

    console.log(a); // throws ReferenceError
    console.log(b); // 1
    console.log(c); // 1

    
    // good
    (function example() {
        let a = 1;
        let b = a;
        let c = a;
    }());

    console.log(a); // throws ReferenceError
    console.log(b); // throws ReferenceError
    console.log(c); // throws ReferenceError

    //都是 `const`

    ```

* 13.6 避免使用一元运算符 eslint: no-plusplus
    ```
    // bad

    const array = [1, 2, 3];
    let num = 1;
    num++;
    --num;

    let sum = 0;
    let truthyCount = 0;
    for (let i = 0; i < array.length; i++) {
        let value = array[i];
        sum += value;
        if (value) {
            truthyCount++;
        }
    }

    // good

    const array = [1, 2, 3];
    let num = 1;
    num += 1;
    num -= 1;

    const sum = array.reduce((a, b) => a + b, 0);
    const truthyCount = array.filter(Boolean).length;
    ```

## 提升(Hoising)
    * 14.1 var声明变量时在其未赋值就已经提升到当前作用域顶部.但是const和let有一个新的概念暂时性死区(Temporal Dead Zones).同样这就意味着使用typeof不再安全.
    ```
    // we know this wouldn’t work (assuming there
    // is no notDefined global variable)
    function example() {
        console.log(notDefined); // => throws a ReferenceError
    }

    // creating a variable declaration after you
    // reference the variable will work due to
    // variable hoisting. Note: the assignment
    // value of `true` is not hoisted.
    function example() {
        console.log(declaredButNotAssigned); // => undefined
        var declaredButNotAssigned = true;
    }

    // the interpreter is hoisting the variable
    // declaration to the top of the scope,
    // which means our example could be rewritten as:
    function example() {
        let declaredButNotAssigned;
        console.log(declaredButNotAssigned); // => undefined
        declaredButNotAssigned = true;
    }

    // using const and let
    function example() {
        console.log(declaredButNotAssigned); // => throws a ReferenceError
        console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
        const declaredButNotAssigned = true;
    }
    ```

* 14.2 匿名函数表达式只是提升了它们的变量名，但是函数并没有赋值.
    ```
    function example() {
        console.log(anonymous); // => undefined

        anonymous(); // => TypeError anonymous is not a function

        var anonymous = function () {
            console.log('anonymous function expression');
        };
    }
    ```

* 14.3 具名函数表达式提升了它们的变量名，童谣函数名和函数也没有提升
    ```
    function example() {
        console.log(named); // => undefined

        named(); // => TypeError named is not a function

        superPower(); // => ReferenceError superPower is not defined

        var named = function superPower() {
                console.log('Flying');
        };
    }

    // the same is true when the function name
    // is the same as the variable name.
    function example() {
        console.log(named); // => undefined

        named(); // => TypeError named is not a function

        var named = function named() {
            console.log('named');
        };
    }
    ```

* 14.4 函数声明提升它们名字和函数体
    ```
    function example() {
        superPower(); // => Flying

        function superPower() {
            console.log('Flying');
        }
    }
    ```

* 更多关于这部分的信息 [javascript Scoping & Hositing](http://www.adequatelygood.com/2010/2/JavaScript-Scoping-and-Hoisting/) by [Ben Cherry](http://www.adequatelygood.com/)

## 比较操作符和相等性(Comparison Operators & Equality)

* 15.1 使用===和 !== 来代替 == 和 ！= eslint: eqeqeq

* 15.2 在条件语句中如if语句对表达式求值时都是用强制类型转换，转换都遵循如下规则:
    * 对象类型(Objects)求值为true
    * Undefiend求值为 false
    * Null求值为 false
    * 布尔类型(Booleans)求值为布尔的值
    * 数字类型(Numbers)当为+0，-0或NaN求值为false其余都为true
    * 字符串类型(Strings)只有字符串为空字符串''为false,其余都为true
    ```
    if ([0] && []) {
        // true
        // an array (even an empty one) is an object, objects will evaluate to true
    }
    ```

* 15.3 布尔类型使用简写，其余如strings和numbers必须明确其比较
    ```
    // bad
    if(isValid === true){
        // ...
    }

    // good
    if(isValid){
        //...
    }

    // bad 
    if(name){
        //...
    }

    // if(name !== ''){
        //...
    }

    // bad
    if(collection.length){
        // ...
    }

    // good 
    if(collection.length > 0){
        // ...
    }
    ```

* 15.4 更多信息看 [Truth Equality and javascript](https://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108) by Angus Croll

* 15.5 对于case和default从句它们如果包含词法声明(eg: let const function calss)使用大括号创建块
    > 为什么? 词法声明在整个是switch块可见但是只有在赋值的时候初始化(只有当case到达时).这样可能带着多个case从句都尝试定义相当的东西
    ```
    // bad
    switch (foo) {
        case 1:
            let x = 1;
            break;
        case 2:
            const y = 2;
            break;
        case 3:
            function f() {
            // ...
            }
            break;
        default:
            class C {}
    }

    // good
    switch (foo) {
        case 1: {
            let x = 1;
            break;
        }
        case 2: {
            const y = 2;
            break;
        }
        case 3: {
            function f() {
            // ...
            }
            break;
        }
        case 4:
            bar();
            break;
        default: {
            class C {}
        }
    }
    ```

* 15.6 三元运算符应该保持在一行，不应有使用很混乱 eslint: no-nested-ternary
    ```
    // bad
    const foo = maybe1 > maybe2
        ? "bar"
        : value1 > value2 ? "baz" : null;

    // split into 2 separated ternary expressions
    const maybeNull = value1 > value2 ? 'baz' : null;

    // better
    const foo = maybe1 > maybe2
        ? 'bar'
        : maybeNull;

    // best
    const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
    ```

* 15.7 避免不必要的三元语句. eslint: no-unneeded-ternary
    ```
    // bad
    const foo = a ? a : b;
    const bar = c ? true : false;
    const baz = c ? false : true;

    // good
    const foo = a || b;
    const bar = !!c;
    const baz = !c
    ```

* 15.8 当多个操作符混合在一个语句中使用圆括号将包裹起来，当有混合数学操作符不要将**与%或者+-*&/之间混合 eslint: no-mixed-operators
    > 为什么?这样改善了可读性并且澄清了开发者的意图
    ```
    // bad
    const foo = a && b < 0 || c > 0 || d + 1 === 0;

    // bad
    const bar = a ** b - 5 % d;

    // bad
    if (a || b && c) {
        return d;
    }

    // good
    const foo = (a && b < 0) || c > 0 || (d + 1 === 0);

    // good
    const bar = (a ** b) - (5 % d);

    // good
    if ((a || b) && c) {
        return d;
    }

    // good
    const bar = a + b / c * d;
    ```

* 块(Blocks)

* 16.1 对所用的多行块使用大括号 eslint: nonblock-statement-body-position
    ```
    // bad
    if (test)
        return false;

    // good
    if (test) return false;

    // good
    if (test) {
        return false;
    }

    // bad
    function foo() { return false; }

    // good
    function bar() {
        return false;
    }
    ```

* 16.2 在多行if-else中else放在if结束括号后 eslint:brace-style
    ```
    // bad
    if (test) {
        thing1();
        thing2();
    }
    else {
        thing3();
    }

    // good
    if (test) {
        thing1();
        thing2();
    } else {
        thing3();
    }
    ```
* 16.3 如果if语句里有return语句,接下来的else块是不必要的. eslint: no-else-return
    ```
    // bad
    function foo() {
        if (x) {
            return x;
        } else {
            return y;
        }
    }

    // bad
    function cats() {
        if (x) {
            return x;
        } else if (y) {
            return y;
        }
    }

    // bad
    function dogs() {
        if (x) {
            return x;
        } else {
            if (y) {
            return y;
            }
        }
    }

    // good
    function foo() {
        if (x) {
            return x;
        }

        return y;
    }

    // good
    function cats() {
        if (x) {
            return x;
        }

        if (y) {
            return y;
        }
    }

    //good
    function dogs(x) {
        if (x) {
            if (z) {
            return y;
            }
        } else {
            return z;
        }
    }

    ```

## 控制语句(Control Statements)

* 17.1 在控制语句中如果太长或者超过了一行的最大长度，将每个（汇合）条件放在一个新行.逻辑运算符应该在行的开头(译者注：这个我自己放在行尾)
    > 为什么?要求操作符放在每行的开始，这是为了和方法链调用方式保持一致和对齐.同样这也提高了可读性
    ```
    // bad
    if ((foo === 123 || bar === 'abc') && doesItLookGoodWhenItBecomesThatLong() && isThisReallyHappening()) {
        thing1();
    }

    // bad
    if (foo === 123 &&
    bar === 'abc') {
        thing1();
    }

    // bad
    if (foo === 123
    && bar === 'abc') {
        thing1();
    }

    // bad
    if (
    foo === 123 &&
    bar === 'abc'
    ) {
        thing1();
    }

    // good
    if (
    foo === 123
    && bar === 'abc'
    ) {
        thing1();
    }

    // good
    if (
    (foo === 123 || bar === "abc")
    && doesItLookGoodWhenItBecomesThatLong()
    && isThisReallyHappening()
    ) {
        thing1();
    }

    // good
    if (foo === 123 && bar === 'abc') {
        thing1();
    }
    ```


## 注释(Comments)

* 18.1 对于多行注释使用 /** ... */
    ```
    // bad
    // make() returns a new element
    // based on the passed in tag name
    //
    // @param {String} tag
    // @return {Element} element
    function make(tag) {

        // ...

        return element;
    }

    // good
    /**
    * make() returns a new element
    * based on the passed-in tag name
    */
    function make(tag) {

        // ...

        return element;
    }
    ```
* 18.2 对于单行注释使用//,将单行注释放在新一行的上面，并且在单行注释前加一个空行除非这个注释在块级的头部
    ```
    // bad
    const active = true; // is current tab

    // good 
    // is current tab
    const active = true

    // bad
    function getType() {
        console.log('fetching type...');
        // set the default type to 'no type'
        const type = this.type || 'no type';

        return type;
    }

    // good
    function getType() {
        console.log('fetching type...');

        // set the default type to 'no type'
        const type = this.type || 'no type';

        return type;
    }

    // also good
    function getType() {
        // set the default type to 'no type'
        const type = this.type || 'no type';

        return type;
    }
    ```

* 18.3 在所有的注释前都加一个空格这样更易阅读. eslint: spaced-comment
    ```
    // bad
    //is current tab
    const active = true;

    // good
    // is current tab
    const active = true;

    // bad
    /**
    *make() returns a new element
    *based on the passed-in tag name
    */
    function make(tag) {

        // ...

        return element;
    }

    // good
    /**
    * make() returns a new element
    * based on the passed-in tag name
    */
    function make(tag) {

        // ...

        return element;
    }
    ```

* 18.4 在注释前加一个FIXME 或者 TODO 来帮助其他开发者迅速的定位问题，或者你建议一个解决方案将要实施. 它们都不同于普通的注释因为它们是可实施的. FIXME：这个动作需要理清，TODOL：需要实施

* 18.5 用 // FIXME: 来标明问题
    ```
    class Calculator extends Abacus {
        constructor() {
            super();

            // FIXME: 这里不应该有全局变量
            total = 0;
        }
    }
    ```

* 18.6 使用// TODO: 来标明解决问的方案
    ```
    class Calculator extends Abacus {
        constructor() {
            super();

            // TODO: total应该通过一个选项参数配置
            this.total = 0;
        }
    }
    ```


## 空白符

* 19.1 使用软tabs(空格键)来设置2个空格 eslint: indent
    ```
        // bad
        function foo() {
        ....let name;
        }

        // bad
        function foo() {
        .let name;
        }

        // good 
        function baz() {
        ..let name;
        }
    ```

* 19.2 在大括号之前使用一个空格 eslint: space-before-blocks
    ```
    // bad
    function test(){
        console.log('test');
    }

    // good
    funtcion test() {
        console.log('test');
    }

    // bad 
    dog.set('attr',{
        age: '1 year',
        breed: 'Bernese Mountain Dog'
    })

    // good
    dog.set('attr', {
        age: '1 year',
        breed: 'Bernese Mountain Dog'
    });
    ```

* 19.3 在控制语句中(if,while)的开始圆括号前加一个空格，而在参数列和函数声明与调用不加空格. eslint: keyword-spaceing
    ```
    // bad
    if(isJedi) {
        fight ();
    }

    // good
    if (isJedi) {
        fight();
    }

    // bad
    function fight () {
        console.log ('Swoosh!');
    }

    // good
    function fight() {
        console.log('Swoosh!');
    }
    ```

* 19.4 在操作符前加空格. eslint: space-infix-ops
    ```
    // bad
    const x=y+4;

    // good
    const x = x + 5;
    ```

* 19.5 在文件结束处加一个换行符. eslint:eol-last
    ```
    // bad
    import { es6 } from './AirbnbStyleGuide';
    // ...
    export default es6;
    ```

    ```
    // bad
    import { es6 } from './AirbnbStyleGuide';
    // ...
    export default es6;↵
    ↵
    ```

    ```
    // good
    import { es6 } from './AirbnbStyleGuide';
    // ...
    export default es6;↵
    ```

* 19.6 当方法调用很长时(超过两个方法调用链)使用缩减.开头出使用点调用，这意味着这行是方法调用，相反的是新的语句不用 eslint: newline-per-chained-call no-whitespace-before-property
    ```
    // bad
    $('#items').find('.selected').highlight().end().find('.open').updateCount();

    // bad
    $('#items').
        find('.selected').
            highlight().
            end().
        find('.open').
            updateCount();

    // good
    $('#items')
        .find('.selected')
            .heighLight()
            .end()
        .find('.open')
            .undateCount();

    // bad
    const leds = stage.selectAll('.led').data(data).enter().append('svg:svg').classed('led', true)
        .attr('width', (radius + margin) * 2).append('svg:g')
        .attr('transform', `translate(${radius + margin},${radius + margin})`)
        .call(tron.led);

    // good
    const leds = stage.selectAll('.led')
        .data(data)
    .enter().append('svg:svg')
        .classed('led', true)
        .attr('width', (radius + margin) * 2)
    .append('svg:g')
        .attr('transform', `translate(${radius + margin},${radius + margin})`)
        .call(tron.led);    
    
    // good
    const leds = stage.selectAll('.led').data(data);    
        
    ```

* 19.7 在每个块级结束之后新语句开始之前留一个空包行
    ```
    // bad
    if (foo) {
        return bar;
    }
    return baz;

    // good
    if (foo) {
        return bar;
    }

    return baz;

    // bad
    const obj = {
        foo() {

        },
        bar() {

        }
    };
    return obj;

    // good
    const obj = {
        foo() {

        },

        bar() {

        },
    };

    return obj;

    // bad
    const arr = [
        function foo() {

        },
        function foo() {

        },
    ];
    return arr;

    // good 
    const arr = [
        function foo() {

        },

        function bar() {

        },
    ];

    return arr;
    ```

* 19.8 不要在新的块头部加空白行
    ```
    // bad
    function bar() {

        console.log(foo);

    }

    // bad
    if (baz) {

        console.log(qux);
        } else {
        console.log(foo);

    }

    // bad
    class Foo {

        constructor(bar) {
            this.bar = bar;
        }
    }

    // good
    function bar() {
        console.log(foo);
    }

    // good
    if (baz) {
        console.log(qux);
    } else {
        console.log(foo);
    }
     ```

* 19.9 不要在圆括号里加额外的空格 eslint: space-in-parens
    ```
    // bad
    function bar( foo ) {
        return foo;
    }

    // good
    function bar(foo) {
        return foo;
    }

    // bad
    if ( foo ) {
        console.log(foo);
    }

    // good
    if (foo) {
        console.log(foo);
    }
    ```
* 19.10 不要在方括号里加空格 eslint:array-bracket-spacing
    ```
    // bad
    const foo = [ 1, 2, 3 ];
    console.log(foo[ 0 ]);

    // good
    const foo = [1, 2, 3];
    console.log(foo[0]);
    ```
* 19.11 在大括号里加空格 eslint: object-curly-spacing
    ```
    // bad
    const foo = {clark: 'kent'};

    // good
    const foo = { clark: 'kent' }
    ```
* 19.12 避免每一行的代码超过100个字符(包括空格). 注意: 上面的长字符串(6.2)是这个规则的例外,其余的都不应该破坏这条规则. eslint： max-len
    > 为什么? 这样确保可读性和可维护性
    ```
    // bad
    const foo = jsonData && jsonData.foo && jsonData.foo.bar && jsonData.foo.bar.baz && jsonData.foo.bar.baz.quux && jsonData.foo.bar.baz.quux.xyzzy;

    // bad
    $.ajax({ method: 'POST', url: 'https://airbnb.com/', data: { name: 'John' } }).done(() => console.log('Congratulations!')).fail(() => console.log('You have failed this city.'));

    // good
    const foo = jsonData
        && jsonData.foo
        && jsonData.foo.bar
        && jsonData.foo.bar.baz
        && jsonData.foo.bar.baz.quux
        && jsonData.foo.bar.baz.quux.xyzzy;

    // good
    $.ajax({
        method: 'POST',
        url: 'https://airbnb.com/',
        data: { name: 'John' },
    })
        .done(() => console.log('Congratulations!'))
        .fail(() => console.log('You have failed this city.'));
    ```


## 逗号(Commas)

* 20.1 不可以以逗号开头. eslint: comma-style
    ```
    // bad
    const story = [
        once
        , upon
        , aTime
    ];

    // good
    const story = [
        once,
        upon,
        aTime,
    ];

    // bad
    const hero = {
        firstName: 'Ada'
        , lastName: 'Lovelace'
        , birthYear: 1815
        , superPower: 'computers'
    };

    // good
    const hero = {
        firstName: 'Ada',
        lastName: 'Lovelace',
        birthYear: 1815,
        superPower: 'computers',
    };
    ```

* 20.2 添加尾逗号. eslint: comma-dangle
    > 为什么?(译者：个人不太稀饭) 这对于git diffs更加整洁,同时向Babel这个样的编译器会在编译的时候去掉尾逗号，这就意味着在合法的浏览器中你不用担心[尾逗号问题](https://github.com/airbnb/javascript/blob/es5-deprecated/es5/README.md#commas)
    ```
    // bad - git diff without trailing comma
    const hero = {
        firstName: 'Florence',
    -    lastName: 'Nightingale'
    +    lastName: 'Nightingale',
    +    inventorOf: ['coxcomb chart', 'modern nursing']
    };

    // good - git diff with trailing comma
    const hero = {
        firstName: 'Florence',
        lastName: 'Nightingale',
    +    inventorOf: ['coxcomb chart', 'modern nursing'],
    };
    ```

    ```
    // bad
    const hero = {
        firstName: 'Dana',
        lastName: 'Scully'
    };

    const heroes = [
        'Batman',
        'Superman'
    ];

    // good
    const hero = {
        firstName: 'Dana',
        lastName: 'Scully',
    };

    const heroes = [
        'Batman',
        'Superman',
    ];

    // bad
    function createHero(
        firstName,
        lastName,
        inventorOf
    ) {
    // does nothing
    }

    // good
    function createHero(
        firstName,
        lastName,
        inventorOf,
    ) {
        // does nothing
    }

    // good (注意逗号不能在"rest"元素后)
    function createHear(
        frstName,
        lastName.
        inventorOf,
        ...heroArs
    ) {
        // does nothing
    }

    // bad
    createHero(
        firstName,
        lastName,
        inventorOf
    );

    // good
    createHero(
        firstName,
        lastName,
        inventorOf,
    );

    // good (note that a comma must not appear after a "rest" element)
    createHero(
        firstName,
        lastName,
        inventorOf,
        ...heroArgs
    );
    ```

## 分号(Semicolons)

* 21.1 是滴 eslint: semi
    ```
    // bad
    (function () {
        const name = 'Skywalker'
        return name
    })()

    // good
    (function () {
        const name = 'Skywalker';
        return name;
    }());

    // good, but legacy (guards against the function becoming an argument when two files with IIFEs are concatenated)
    ;((() => {
        const name = 'Skywalker';
        return name;
    })());
    ```

## 类型转换和强制类型转换(Type Casting & Coercion)

* 22.1 在语句的开头执行强制类型转化

* 22.2 字符串：eslint: no-new-wrappers
    ```
     // => this.reviewScore = 9;

     // bad
     const totalScore = new String(this.reviewScore); // typeof totalScore 是"object"而不是"string"

     // bad
     const totalScore = this.reviewSore + ''; // 会触发this.reviewScore.valueOf()

     // bad 
     const totalScore = this.reviewScore.toStirng();// 不能保证是返回一个字符串,这个方法有可能被覆盖掉

     // good
     const totalScore = String(this.reviewScore)
    ```

* 22.3 数字：使用Number来类型转换，使用parseInt总是需要一个基数(radix) eslint: redix no-new-wrappers
    ```
    const inputValue = '4';

    // bad
    const val = new Number(inputValue);

    // bad
    const val = inputValue >> 0;

    // bad 
    const val = parseInt(inputValue);

    // good
    const val = Number(inputValue);

    // good
    const val = parseInt(inputValue, 10);
    ```

* 22.4 如果你因为某种原因而做一些疯狂事parseInt不能满足你的性能要求你需要使用位运算，写一些注释来解释为什么你这样做.
    ```
    // good
    /**
    * parseInt是我这段代码慢的原因
    * 位运算可以解决这个问题
    */

    const val = inputValue >> 0;
    ```

*  22.5 注意：使用位运算操作符时，运算前数字代表的是64位的值，但是位运算后返回的是一个32位的整数([source](https://es5.github.io/#x11.7)),位运算对于超过32位的整数可以引起不可预料的表现行为.最大的32位整数2,147,483,647:
    ```
    2147483647 >> 0; // => 2147483647
    2147483648 >> 0; // => -2147483648
    2147483649 >> 0; // => -2147483647
    ```

* 22.6 布尔： eslint: no-new-wrappers
    ```
    const age = 0;

    // bad
    const hasAge = new Boolean(age);

    // good
    const hasAge = Boolean(age);

    // best
    const hasAge = !!age;
    ```

## 命名习俗 (Naming Conventions)

* 23.1 避免单个字母命名，使用描述性命名. eslint: id-length
    ```
    // bad
    function q() {
        // ...
    }

    // good
    function query() {
        // ...
    } 
    ```

* 23.2 使用驼峰命名：对象,函数,实例. eslint: camelcase
    ```
    // bad
    const OBJEcttsssss = {};
    const this_is_my_object = {};
    function c() {}

    // good
    const thisIsMyObject = {};
    function thisIsMyFunction() {}
    ```

* 23.3 在命名构造函数或者类时使用 首字母大写  eslint: new-cap
    ```
    // bad
    function user(options) {
        this.name = options.name;
    }

    const bad = new user({
        name: 'nope',
    });

    // good
    class User {
        constructor(options) {
            this.name = options.name;
        }
    }

    const good = new User({
        name: 'yup',
    });
    ```

* 23.4 不要在头尾使用下划线. eslint: no-underscore-dangle
    > 为什么? js对于属性或者方法都没有私有的概念，虽然前置下划线按习俗来讲是一个私有的(private).事实上,这些带下划线的属性是完全公开的,并且存在于你的公共API接口中，这个习俗让开发者错误的认为一点改变不会是破坏，或者不需要测试，如果你想要某些东西真的私有("private"),你必须让他不可以观察的呈现.
    ```
    // bad
    this.__firstName__ = 'Panda';
    this.firstName_ = 'Panda';
    this._firstName = 'Panda';

    // good
    this.firstName = 'Panda';
    ```

* 23.5 希望不要保存this引用.与之代替的是使用箭头函数或者函数绑定
    ```
    // bad
    function foo() {
        const self = this;
        return function () {
            console.log(self);
        };
    }

    // bad
    function foo() {
        const that = this;
        return function () {
            console.log(that);
        };
    }

    // good
    function foo() {
        return () => {
            console.log(this);
        };
    }

    ```

* 23.6 一个基本文件名应该和默认导出(default export)相匹配
    ```
     // file 1 contents
    class CheckBox {
    // ...
    }
    export default CheckBox;

    // file 2 contents
    export default function fortyTwo() { return 42; }

    // file 3 contents
    export default function insideDirectory() {}

    // in some other file
    // bad
    import CheckBox from './checkBox'; // PascalCase import/export, camelCase filename
    import FortyTwo from './FortyTwo'; // PascalCase import/filename, camelCase export
    import InsideDirectory from './InsideDirectory'; // PascalCase import/filename, camelCase export

    // bad
    import CheckBox from './check_box'; // PascalCase import/export, snake_case filename
    import forty_two from './forty_two'; // snake_case import/filename, camelCase export
    import inside_directory from './inside_directory'; // snake_case import, camelCase export
    import index from './inside_directory/index'; // requiring the index file explicitly
    import insideDirectory from './insideDirectory/index'; // requiring the index file explicitly

    // good
    import CheckBox from './CheckBox'; // PascalCase export/import/filename
    import fortyTwo from './fortyTwo'; // camelCase export/import/filename
    import insideDirectory from './insideDirectory'; // camelCase export/import/directory name/implicit "index"
    // ^ supports both insideDirectory.js and insideDirectory/index.js   
    ```

* 23.7 当默认导出一个函数使用驼峰法命名,你文件名称应该和你函数名保存一致
    ```
    function makeStyleGuide() {

    }

    export default makeStyleGuide;
    ```

* 23.8 当你导出一个contructor/ class/函数库/单独的对象 使用首PascalCase
    ```
    const AirbnbStyleGuide = {
        es6: {
        },
    };

    export default AirbnbStyleGuide;
    ```
* 23.9 缩写是也应该首字母大写，其余全为小写
    ```
    // bad
    import SmsContainer from './containers/SmsContainer';

    // bad
    const HttpRequests = [
        // ...
    ];

    // good
    import SMSContainer from './containers/SMSContainer';

    // good
    const HTTPRequests = [
        // ...
    ];

    // also good
    const httpRequests = [
    // ...
    ];

    // best
    import TextMessageContainer from './containers/TextMessageContainer';

    // best
    const requests = [
        // ...
    ];
    ```


## 存取器(存取器)

* 24.1 对于属性存取器函数不是必要的

* 24.2 不要使用js的getters/setters因为它们会引起不可预期的副作用并且难于测试和维护，和推理. 如果你一定要使用存取函数使用getVal()和setVal('hello')
    ```
    // bad
    class Dragon {
        get age() {

        }

        set age(value) {
            // ...
        }
    }

    // good
    class Dragon {
        getAge() {

        }

        setAge(value){
            // ...
        }
    }
    ```

* 24.3 如果属性或方法是boolean,使用isVal()或者hasVal()
    ```
    // bad
    if(!dragon.age()) {
        return false;
    }

    // good
    if(!dragon.hasAge()){
        return false;
    }
    ```

* 24.4 可以创建get()和set()函数，但是得保持一致
    ```
    class Jedi {
        constructor(options = {}) {
            const lightsaber = options.lightsaber || 'blue';
            this.set('lightsaber', lightsaber);
        }

        set(key, val) {
            this[key] = val;
        }

        get(key) {
            return this[key];
        }
    }
    ```


## 事件(Events)

* 25.1 当添加数据加载到事件中(原生或者自定义),传入一个哈希值而不是原始数据,这样可以让后来的开发者添加更多数据到事件中时不要先找到或更新每个handler,也就是让传入参数更加灵活.如下：
    ```
    // bad
    $(this).trigger('listingUpdated', listing.id);

    // ...

    $(this).on('listingUpdated', (e, listingId) => {
        // do something with listingId
    });

    // good
    $(this).trigger('listingUpdated', { listingId: listing.id });

    // ...

    $(this).on('listingUpdated', (e, data) => {
        // do something with data.listingId
    });
    ```


## jQuery

* 26.1 对每个jQuery对象变量都加个前缀$  jscs: requireDollarBeforejQueryAssignment

    ```
    // bad
    const sidebar = $('.sidebar');

    // good
    const $sidebar = $('.sidebar');

    // good
    const $sidebarBtn = $('.sidebar-btn');
    ```

* 26.2 缓存jquery对象
    ```
    // bad
    function setSidebar() {
        $('.sidebar').hide();

        // ...

        $('.sidebar').css({
            'background-color': 'pink',
        });
    }

    // good
    function setSidebar() {
        const $sidebar = $('.sidebar');
        $sidebar.hide();

        // ...

        $sidebar.css({
            'background-color': 'pink',
        });
    }
    ```

* 26.3 对于DOM查询使用垂直查询$('.sidebar ul') 或者>  $('.sidebar > ul')
    ```
    // bad
    $('ul', '.sidebar').hide();

    // bad
    $('.sidebar').find('ul').hide();

    // good
    $('.sidebar ul').hide();

    // good
    $('.sidebar > ul').hide();

    // good
    $sidebar.find('ul').hide();
    ```

* 原文![Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript#arrow-functions)


    


