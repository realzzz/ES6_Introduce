> 原文地址 [ECMAScript 6 — New Features: Overview & Comparison](http://es6-features.org/)
> 本文介绍 ECMAScript 6中的新特性并通过具体代码演示与ECMAScript 5的差别。译者额外增加了一些例子和说明 .

> ECMAScript 6 是2015年推出的第6版本的ECMAScript标准(即Javascript标准)，同时也被成为ECMAScript 2015. 

> ECMAScript 6 中定了了许多新的Javascript特性，包括新的类和模块，类似python的generators和generators表达式, 箭头函数, 二进制数据, 类型数组, 集合类型(maps, set  & 弱maps), promises, reflection, proxy等。它也被称作 ES6 Harmony. (和谐ES6)

> 本文是该系列文章的第一篇

### 作用域

#### 变量的作用域
现在在代码块中以let声明的变量和常量不会被提升到顶部声明。

*ECMAScript 6的实现*
``` javascript
for (let i = 0; i < a.length; i++) {
    let x = a[i]  // 代码块中定义新的x变量
    …
}
for (let i = 0; i < b.length; i++) {
    let y = b[i] //  代码块中定义新的y变量
    …
}

let callbacks = []
for (let i = 0; i <= 2; i++) {
    callbacks[i] = function () { return i * 2 } // 代码块中变量赋值新函数
}
callbacks[0]() === 0  //  按照赋值函数运行的结果
callbacks[1]() === 2
callbacks[2]() === 4
```


*对比ECMAScript 5的实现*
```javascript
var i, x, y; // 会被提升到顶部，作为全局变量
for (i = 0; i < a.length; i++) {
    x = a[i];
    …
}
for (i = 0; i < b.length; i++) {
    y = b[i];
    …
}

var callbacks = [];
for (var i = 0; i <= 2; i++) {
    (function (i) {
        callbacks[i] = function() { return i * 2; }; // 必须通过模拟代码块环境来实现
    })(i);
}
callbacks[0]() === 0;
callbacks[1]() === 2;
callbacks[2]() === 4;
```


*ECMAScript 5中错误的写法*
```javascript

var callbacks = [];
for (var i = 0; i <= 2; i++) {
    callbacks[i] = function() { return i * 2; }; // 这里的i会被提升到顶部声明，导致三个callback中的i都是3
}
callbacks[0]() === 6;
callbacks[1]() === 6;
callbacks[2]() === 6;
```

#### 函数的作用域
函数在代码块中定义的方式

*ECMAScript 6的实现*
```javascript
{
    function foo () { return 1 }
    foo() === 1
    {
        function foo () { return 2 }
        foo() === 2
    }
    foo() === 1
}
```
*对比ECMAScript 5的实现*
```javascript
(function () {  // 必须通过模拟代码块环境来实现
    var foo = function () { return 1; }
    foo() === 1;
    (function () {
        var foo = function () { return 2; }
        foo() === 2;
    })();
    foo() === 1;
})();
```


### 箭头函数

####  表达体函数
更简单的表达函数包符号

*ECMAScript 6的实现*

``` javascript
odds  = evens.map(v => v + 1)  // 函数定义，输入，返回无明显的符号边界
pairs = evens.map(v => ({ even: v, odd: v + 1 }))  
nums  = evens.map((v, i) => v + i)
```

*对比ECMAScript 5的实现*

```javascript
odds  = evens.map(function (v) { return v + 1; });
pairs = evens.map(function (v) { return { even: v, odd: v + 1 }; });
nums  = evens.map(function (v, i) { return v + i; });
```

#### 条件体函数

*ECMAScript 6的实现*

``` javascript
nums.forEach(v => { //  明确函数体边界
   if (v % 5 === 0)
       fives.push(v)
})
```

*对比ECMAScript 5的实现*

```javascript
nums.forEach(function (v) {
   if (v % 5 === 0)
       fives.push(v);
});
```
#### this的指代
this更加直观的指代当前上下文

*ECMAScript 6的实现*

``` javascript
this.nums.forEach((v) => {
    if (v % 5 === 0)
        this.fives.push(v) // 这里的this 指的是运行for Each的上下文this.
})
```

*对比ECMAScript 5的实现*

```javascript
//  variant 1
var self = this;
this.nums.forEach(function (v) {
    if (v % 5 === 0)
        self.fives.push(v);  // 需要额外指定self来获取外部this
});

//  variant 2
this.nums.forEach(function (v) {
    if (v % 5 === 0)
        this.fives.push(v); // 需要将外部this当作参数传入
}, this);

//  variant 3 (since ECMAScript 5.1 only)
this.nums.forEach(function (v) {
    if (v % 5 === 0)
        this.fives.push(v); // 需要绑定外部this
}.bind(this)); 
```


### 扩展的函数参数处理

#### 默认参数
简单而且直观的默认参数方式

*ECMAScript 6的实现*

``` javascript
function f (x, y = 7, z = 42) {
    return x + y + z
}
f(1) === 50 // y取默认参数7, z取默认参数 42
```

*对比ECMAScript 5的实现*

```javascript
function f (x, y, z) {
    if (y === undefined)
        y = 7;
    if (z === undefined)
        z = 42;
    return x + y + z;
};
f(1) === 50;
```

#### 可变参数个数
对剩余参数可变个数的单个参数聚合

*ECMAScript 6的实现*

``` javascript
function f (x, y, ...a) {
    return (x + y) * a.length // 获取剩余参数a的参数个数 a.length
}
f(1, 2, "hello", true, 7) === 9
```

*对比ECMAScript 5的实现*

```javascript
function f (x, y) {
    var a = Array.prototype.slice.call(arguments, 2); // 通过协议获取参数个数并切割
    return (x + y) * a.length;
};
f(1, 2, "hello", true, 7) === 9;
```

#### 扩展操作符
针对可以遍历的对象集合(例如数组，字符串)，将其拆解成单个个体并填入其他参数/遍历对象。

*ECMAScript 6的实现*

``` javascript
var params = [ "hello", true, 7 ]
var other = [ 1, 2, ...params ] // [ 1, 2, "hello", true, 7 ] // 将数组params填入数组other

function f (x, y, ...a) {
    return (x + y) * a.length
}
f(1, 2, ...params) === 9 // 将params填入可变参数

var str = "foo"
var chars = [ ...str ] // [ "f", "o", "o" ] // 将str拆解为单个字符并填入数组
```

*对比ECMAScript 5的实现*

```javascript
var params = [ "hello", true, 7 ];
var other = [ 1, 2 ].concat(params); // [ 1, 2, "hello", true, 7 ] 

function f (x, y) {
    var a = Array.prototype.slice.call(arguments, 2);
    return (x + y) * a.length;
};
f.apply(undefined, [ 1, 2 ].concat(params)) === 9; 

var str = "foo";
var chars = str.split(""); // [ "f", "o", "o" ]
```


### 模版语法

#### 字符串的插入修改
对单行和多行的字符串进行方便直观的插入修改 

*ECMAScript 6的实现*

``` javascript
var customer = { name: "Foo" }
var card = { amount: 7, product: "Bar", unitprice: 42 }
// 可以看到 使用 ` ` 包含字符串模版，通过${} 插入内容
var message = `Hello ${customer.name},  
want to buy ${card.amount} ${card.product} for
a total of ${card.amount * card.unitprice} bucks?`
```

*对比ECMAScript 5的实现*

```javascript
var customer = { name: "Foo" };
var card = { amount: 7, product: "Bar", unitprice: 42 };
// 需要使用 + + 来转换连接内容
var message = "Hello " + customer.name + ",\n" +
"want to buy " + card.amount + " " + card.product + " for\n" +
"a total of " + (card.amount * card.unitprice) + " bucks?";
```


####  自定义添加修改
弹性的表达式针对各种方法都可以执行例如:

*ECMAScript 6的实现*

``` javascript
get`http://example.com/foo?bar=${bar + baz}&quux=${quux}`  
```

*对比ECMAScript 5的实现*

```javascript
get([ "http://example.com/foo?bar=", "&quux=", "" ],bar + baz, quux);
```

####  原始字符串的操作
String现在可以操作原始字符串了 (不支持反斜杠)

*ECMAScript 6的实现*

``` javascript
function quux (strings, ...values) {
    strings[0] === "foo\n"  
    strings[1] === "bar"
    strings.raw[0] === "foo\\n"
    strings.raw[1] === "bar"
    values[0] === 42
}
quux `foo\n${ 42 }bar`

String.raw `foo\n${ 42 }bar` === "foo\\n42bar"
```

*对比ECMAScript 5的实现*

```javascript
//无
```

### 文字扩展

#### 对二进制和八进制的扩展
现在可以直接操作二进制和八进制的字符了

*ECMAScript 6的实现*

``` javascript
0b111110111 === 503 // 二进制实现
0o767 === 503 // 八进制实现
```

*对比ECMAScript 5的实现*

```javascript
parseInt("111110111", 2) === 503;
parseInt("767", 8) === 503;
0767 === 503; // 只在 non-strict, 向后兼容模式
```

#### 对Unicode字符串和正则表达式的扩展
支持unicode的字符串和相对应的正则表达式

*ECMAScript 6的实现*

``` javascript
"𠮷".length === 2
"𠮷".match(/./u)[0].length === 2
"𠮷" === "\uD842\uDFB7"
"𠮷" === "\u{20BB7}"
"𠮷".codePointAt(0) == 0x20BB7
for (let codepoint of "𠮷") console.log(codepoint)
```

*对比ECMAScript 5的实现*

```javascript
"𠮷".length === 2;
"𠮷".match(/(?:[\0-\t\x0B\f\x0E-\u2027\u202A-\uD7FF\uE000-\uFFFF][\uD800-\uDBFF][\uDC00-\uDFFF][\uD800-\uDBFF](?![\uDC00-\uDFFF])(?:[^\uD800-\uDBFF]^)[\uDC00-\uDFFF])/)[0].length === 2;
"𠮷" === "\uD842\uDFB7";
//  ES5中无
//  ES5中无
//  ES5中无
```

### 正则表达式增强

#### 正则表达式的粘性匹配

通过保持匹配位置来更有效率的在不同的长字符串中匹配不同的正则表达式。

*ECMAScript 6的实现*

``` javascript
let parser = (input, match) => {
    for (let pos = 0, lastPos = input.length; pos < lastPos; ) {
        for (let i = 0; i < match.length; i++) {
            match[i].pattern.lastIndex = pos
            let found
            if ((found = match[i].pattern.exec(input)) !== null) {
                match[i].action(found)
                pos = match[i].pattern.lastIndex
                break
            }
        }
    }
}

let report = (match) => {
    console.log(JSON.stringify(match))
}
parser("Foo 1 Bar 7 Baz 42", [
    { pattern: /^Foo\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^Bar\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^Baz\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^\s*/y,         action: (match) => {}            }
])
```

*对比ECMAScript 5的实现*

```javascript
var parser = function (input, match) {
    for (var i, found, inputTmp = input; inputTmp !== ""; ) {
        for (i = 0; i < match.length; i++) {
            if ((found = match[i].pattern.exec(inputTmp)) !== null) {
                match[i].action(found);
                inputTmp = inputTmp.substr(found[0].length);
                break;
            }
        }
    }
}

var report = function (match) {
    console.log(JSON.stringify(match));
};
parser("Foo 1 Bar 7 Baz 42", [
    { pattern: /^Foo\s+(\d+)/, action: function (match) { report(match); } },
    { pattern: /^Bar\s+(\d+)/, action: function (match) { report(match); } },
    { pattern: /^Baz\s+(\d+)/, action: function (match) { report(match); } },
    { pattern: /^\s*/,         action: function (match) {}                 }
]);
```


> 原文地址 [ECMAScript 6 — New Features: Overview & Comparison](http://es6-features.org/)
> 本文介绍 ECMAScript 6中的新特性并通过具体代码演示与ECMAScript 5的差别。译者额外增加了一些例子和说明 .

> ECMAScript 6 是2015年推出的第6版本的ECMAScript标准(即Javascript标准)，同时也被成为ECMAScript 2015. 

> ECMAScript 6 中定了了许多新的Javascript特性，包括新的类和模块，类似python的generators和generators表达式, 箭头函数, 二进制数据, 类型数组, 集合类型(maps, set  & 弱maps), promises, reflection, proxy等。它也被称作 ES6 Harmony. (和谐ES6)

> 本文是该系列文章的第二篇


### 增强的对象属性

#### 变量速写

缩短了对于对象属性的定义习惯

*ECMAScript 6的实现*
```javascript
obj = { x, y }
```

*对比ECMAScript 5的实现*
```javascript
obj = { x: x, y: y };
```

#### 支持表达式计算属性的定义

支持表达式计算属性的定义

*ECMAScript 6的实现*
```javascript
let obj = {
    foo: "bar",
    [ "baz" + quux() ]: 42
}
```

*对比ECMAScript 5的实现*
```javascript
var obj = {
    foo: "bar"
};
obj[ "baz" + quux() ] = 42;
```


#### 函数属性

支持在对象中直接的函数符号定义, 普通函数和生产(generator)函数均可

*ECMAScript 6的实现*
```javascript
obj = {
    foo (a, b) {
        …
    },
    bar (x, y) {
        …
    },
    *quux (x, y) {
        …
    }
}
```

*对比ECMAScript 5的实现*
```javascript
obj = {
    foo: function (a, b) {
        …
    },
    bar: function (x, y) {
        …
    },
    //  quux: ES5中没有这样的生产(generator)函数
    …
};
```

### 释放赋值

#### 数组的值匹配

数组可以很直观的释放其中的值到不同的变量中去

*ECMAScript 6的实现*
```javascript
var list = [ 1, 2, 3 ]
var [ a, , b ] = list
[ b, a ] = [ a, b ]
```

*对比ECMAScript 5的实现*
```javascript
var list = [ 1, 2, 3 ];
var a = list[0], b = list[2];
var tmp = a; a = b; b = tmp;
```

#### 对象匹配的便捷符号
可以很直观的把对象中的属性释放到对应的变量中

*ECMAScript 6的实现*
```javascript
var { op, lhs, rhs } = getASTNode() // 返回obj中有op, lhs, rhs属性
```

*对比ECMAScript 5的实现*
```javascript
var tmp = getASTNode();
var op  = tmp.op;
var lhs = tmp.lhs;
var rhs = tmp.rhs;
```

#### 对象深层匹配

对于多层的对象属性，也能够通过符号直观的匹配

 *ECMAScript 6的实现*
```javascript
var { op: a, lhs: { op: b }, rhs: c } = getASTNode() //其中的lhs是一个有op属性的对象
```

*对比ECMAScript 5的实现*
```javascript
var tmp = getASTNode();
var a = tmp.op;
var b = tmp.lhs.op;
var c = tmp.rhs;
```

#### 参数上下文匹配
对函数传参的时候，也可以方便的把数组和对象释放成不同的参数

*ECMAScript 6的实现*
```javascript
function f ([ name, val ]) { // 数组拆出值 相当于传入 (name, val)
    console.log(name, val)
}
function g ({ name: n, val: v }) { // 对象拆出值, 相当于传入 (n, v)
    console.log(n, v)
}
function h ({ name, val }) { // 对象拆出值, 相当于传入 (name, val)
    console.log(name, val)
}
f([ "bar", 42 ])
g({ name: "foo", val:  7 })
h({ name: "bar", val: 42 })
```

*对比ECMAScript 5的实现*
```javascript
function f (arg) {
    var name = arg[0];
    var val  = arg[1];
    console.log(name, val);
};
function g (arg) {
    var n = arg.name;
    var v = arg.val;
    console.log(n, v);
};
function h (arg) {
    var name = arg.name;
    var val  = arg.val;
    console.log(name, val);
};
f([ "bar", 42 ]);
g({ name: "foo", val:  7 });
h({ name: "bar", val: 42 });
```



#### 释放的容错机制
当释放的参数无法匹配上时，会自动容错到默认值

*ECMAScript 6的实现*
```javascript
var list = [ 7, 42 ]
var [ a = 1, b = 2, c = 3, d ] = list
a === 7
b === 42
c === 3  // 匹配之前默认值3
d === undefined // 无匹配 成为默认undefined
```

*对比ECMAScript 5的实现*
```javascript
var list = [ 7, 42 ];
var a = typeof list[0] !== "undefined" ? list[0] : 1;
var b = typeof list[1] !== "undefined" ? list[1] : 2;
var c = typeof list[2] !== "undefined" ? list[2] : 3;
var d = typeof list[3] !== "undefined" ? list[3] : undefined;
a === 7;
b === 42;
c === 3;
d === undefined;
```

### 模块

#### 模块值的导入和导出 

支持从模块中导入/导出值，但又不污染全局名字空间

*ECMAScript 6的实现*
```javascript
//  lib/math.js
export function sum (x, y) { return x + y } // 导出函数
export var pi = 3.141593  // 导出变量 

//  someApp.js
import * as math from "lib/math"
console.log("2π = " + math.sum(math.pi, math.pi))

//  otherApp.js
import { sum, pi } from "lib/math"
console.log("2π = " + sum(pi, pi))

```

*对比ECMAScript 5的实现*
```javascript
LibMath = {};
LibMath.sum = function (x, y) { return x + y };
LibMath.pi = 3.141593;

//  someApp.js
var math = LibMath;
console.log("2π = " + math.sum(math.pi, math.pi));

//  otherApp.js
var sum = LibMath.sum, pi = LibMath.pi;
console.log("2π = " + sum(pi, pi));
```


#### 默认和通配符
可以通过通配符导出所有的符号， 通过标记default默认导出 (无需声明)
*ECMAScript 6的实现*
```javascript
//  lib/mathplusplus.js
export * from "lib/math"
export var e = 2.71828182846
export default (x) => Math.exp(x)  //默认导出exp函数

//  someApp.js
import exp, { pi, e } from "lib/mathplusplus"
console.log("e^{π} = " + exp(pi))
```

*对比ECMAScript 5的实现*
```javascript
//  lib/mathplusplus.js
LibMathPP = {};
for (symbol in LibMath)
    if (LibMath.hasOwnProperty(symbol))
        LibMathPP[symbol] = LibMath[symbol];
LibMathPP.e = 2.71828182846;
LibMathPP.exp = function (x) { return Math.exp(x) };

//  someApp.js
var exp = LibMathPP.exp, pi = LibMathPP.pi, e = LibMathPP.e;
console.log("e^{π} = " + exp(pi));
```


### 类

#### 类定义 
更加直接，面向对象风格的类

*ECMAScript 6的实现*
```javascript
class Shape {
    constructor (id, x, y) {   // 类的构造函数
        this.id = id
        this.move(x, y)
    }
    move (x, y) {   // 类函数
        this.x = x
        this.y = y
    }
}
```

*对比ECMAScript 5的实现*
```javascript
var Shape = function (id, x, y) {
    this.id = id;
    this.move(x, y);
};
Shape.prototype.move = function (x, y) {
    this.x = x;
    this.y = y;
};
```

#### 类的继承
面向对象风格的类继承方式
*ECMAScript 6的实现*
```javascript
class Rectangle extends Shape {
    constructor (id, x, y, width, height) {
        super(id, x, y)
        this.width  = width
        this.height = height
    }
}
class Circle extends Shape {  // Circle继承于Shape同时扩展了构造函数参数 
    constructor (id, x, y, radius) {
        super(id, x, y)
        this.radius = radius
    }
}
```

*对比ECMAScript 5的实现*
```javascript
var Rectangle = function (id, x, y, width, height) {
    Shape.call(this, id, x, y);
    this.width  = width;
    this.height = height;
};
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;
var Circle = function (id, x, y, radius) {
    Shape.call(this, id, x, y);
    this.radius = radius;
};
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;  // 需要通过prototype来替换构造函数
```


#### 类的多继承，聚合

支持对多个类的继承（即对应的类函数的聚合） 需要注意的是以下代码例子中的 聚合（aggregation）函数是有[现成的库](https://github.com/rse/aggregation)可以使用的

*ECMAScript 6的实现*
```javascript
var aggregation = (baseClass, ...mixins) => {
    let base = class _Combined extends baseClass {
        constructor (...args) {
            super(...args)
            mixins.forEach((mixin) => {
                mixin.prototype.initializer.call(this)
            })
        }
    }
    let copyProps = (target, source) => {
        Object.getOwnPropertyNames(source)
            .concat(Object.getOwnPropertySymbols(source))
            .forEach((prop) => {
            if (prop.match(/^(?:constructor|prototype|arguments|caller|name|bind|call|apply|toString|length)$/))
                return
            Object.defineProperty(target, prop, Object.getOwnPropertyDescriptor(source, prop))
        })
    }
    mixins.forEach((mixin) => {
        copyProps(base.prototype, mixin.prototype)
        copyProps(base, mixin)
    })
    return base
}

class Colored {
    initializer ()     { this._color = "white" }
    get color ()       { return this._color }
    set color (v)      { this._color = v }
}

class ZCoord {
    initializer ()     { this._z = 0 }
    get z ()           { return this._z }
    set z (v)          { this._z = v }
}

class Shape {
    constructor (x, y) { this._x = x; this._y = y }
    get x ()           { return this._x }
    set x (v)          { this._x = v }
    get y ()           { return this._y }
    set y (v)          { this._y = v }
}

class Rectangle extends aggregation(Shape, Colored, ZCoord) {}

var rect = new Rectangle(7, 42)
rect.z     = 1000
rect.color = "red"
console.log(rect.x, rect.y, rect.z, rect.color)

```

*对比ECMAScript 5的实现*
```javascript
var aggregation = function (baseClass, mixins) {
    var base = function () {
        baseClass.apply(this, arguments);
        mixins.forEach(function (mixin) {
            mixin.prototype.initializer.call(this);
        }.bind(this));
    };
    base.prototype = Object.create(baseClass.prototype);
    base.prototype.constructor = base;
    var copyProps = function (target, source) {
        Object.getOwnPropertyNames(source).forEach(function (prop) {
            if (prop.match(/^(?:constructor|prototype|arguments|caller|name|bind|call|apply|toString|length)$/))
                return
            Object.defineProperty(target, prop, Object.getOwnPropertyDescriptor(source, prop))
        })
    }
    mixins.forEach(function (mixin) {
        copyProps(base.prototype, mixin.prototype);
        copyProps(base, mixin);
    });
    return base;
};

var Colored = function () {};
Colored.prototype = {
    initializer: function ()  { this._color = "white"; },
    getColor:    function ()  { return this._color; },
    setColor:    function (v) { this._color = v; }
};

var ZCoord = function () {};
ZCoord.prototype = {
    initializer: function ()  { this._z = 0; },
    getZ:        function ()  { return this._z; },
    setZ:        function (v) { this._z = v; }
};

var Shape = function (x, y) {
    this._x = x; this._y = y;
};
Shape.prototype = {
    getX: function ()  { return this._x; },
    setX: function (v) { this._x = v; },
    getY: function ()  { return this._y; },
    setY: function (v) { this._y = v; }
}

var _Combined = aggregation(Shape, [ Colored, ZCoord ]);
var Rectangle = function (x, y) {
    _Combined.call(this, x, y);
};
Rectangle.prototype = Object.create(_Combined.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle(7, 42);
rect.setZ(1000);
rect.setColor("red");
console.log(rect.getX(), rect.getY(), rect.getZ(), rect.getColor());
```

#### 基类的访问

对基类的构造函数和方法的直观访问。

*ECMAScript 6的实现*
```javascript
class Shape {
    …
    toString () {
        return `Shape(${this.id})`
    }
}
class Rectangle extends Shape {
    constructor (id, x, y, width, height) {
        super(id, x, y)
        …
    }
    toString () {
        return "Rectangle > " + super.toString() // 通过super 访问父类
    }
}
class Circle extends Shape {
    constructor (id, x, y, radius) {
        super(id, x, y)
        …
    }
    toString () {
        return "Circle > " + super.toString()
    }
}
```

*对比ECMAScript 5的实现*
```javascript
var Shape = function (id, x, y) {
    …
};
Shape.prototype.toString = function (x, y) {
    return "Shape(" + this.id + ")"
};
var Rectangle = function (id, x, y, width, height) {
    Shape.call(this, id, x, y);
    …
};
Rectangle.prototype.toString = function () {
    return "Rectangle > " + Shape.prototype.toString.call(this);
};
var Circle = function (id, x, y, radius) {
    Shape.call(this, id, x, y);
    …
};
Circle.prototype.toString = function () {
    return "Circle > " + Shape.prototype.toString.call(this); //没有super只能通过对应的原型中函数调用 
};
```

####  静态成员
简单的支持类静态成员

*ECMAScript 6的实现*
```javascript
class Rectangle extends Shape {
    …
    static defaultRectangle () {
        return new Rectangle("default", 0, 0, 100, 100)
    }
}
class Circle extends Shape {
    …
    static defaultCircle () {
        return new Circle("default", 0, 0, 100)
    }
}
var defRectangle = Rectangle.defaultRectangle()
var defCircle    = Circle.defaultCircle()
```

*对比ECMAScript 5的实现*
```javascript
var Rectangle = function (id, x, y, width, height) {
    …
};
Rectangle.defaultRectangle = function () {
    return new Rectangle("default", 0, 0, 100, 100);
};
var Circle = function (id, x, y, width, height) {
    …
};
Circle.defaultCircle = function () {
    return new Circle("default", 0, 0, 100);
};
var defRectangle = Rectangle.defaultRectangle();
var defCircle    = Circle.defaultCircle();
```
#### Getter/Setter
Getter/Setter 支持直接在类中定义了 

*ECMAScript 6的实现*
```javascript
class Rectangle {
    constructor (width, height) {
        this._width  = width
        this._height = height
    }
    set width  (width)  { this._width = width               }
    get width  ()       { return this._width                }
    set height (height) { this._height = height             }
    get height ()       { return this._height               }
    get area   ()       { return this._width * this._height }
}
var r = new Rectangle(50, 20)
r.area === 1000
```

*对比ECMAScript 5的实现*
```javascript
var Rectangle = function (width, height) {
    this._width  = width;
    this._height = height;
};
Rectangle.prototype = {
    set width  (width)  { this._width = width;               },
    get width  ()       { return this._width;                },
    set height (height) { this._height = height;             },
    get height ()       { return this._height;               },
    get area   ()       { return this._width * this._height; }
};
var r = new Rectangle(50, 20);
r.area === 1000;
```



> 原文地址 [ECMAScript 6 — New Features: Overview & Comparison](http://es6-features.org/)
> 本文介绍 ECMAScript 6中的新特性并通过具体代码演示与ECMAScript 5的差别。译者额外增加了一些例子和说明 .

> ECMAScript 6 是2015年推出的第6版本的ECMAScript标准(即Javascript标准)，同时也被成为ECMAScript 2015. 

> ECMAScript 6 中定了了许多新的Javascript特性，包括新的类和模块，类似python的generators和generators表达式, 箭头函数, 二进制数据, 类型数组, 集合类型(maps, set  & 弱maps), promises, reflection, proxy等。它也被称作 ES6 Harmony. (和谐ES6)

> 本文是该系列文章的第三篇

### Symbol 类型

#### Symbol类型
作为对象的一个唯一而且不可修改的属性.  Symbol是个可选的描述，只是用在调试中。

*ECMAScript 6的实现*
```javascript
Symbol("foo") !== Symbol("foo") 
const foo = Symbol()
const bar = Symbol()
typeof foo === "symbol"
typeof bar === "symbol"
let obj = {}
obj[foo] = "foo"
obj[bar] = "bar"
JSON.stringify(obj) // {}
Object.keys(obj) // []
Object.getOwnPropertyNames(obj) // []
Object.getOwnPropertySymbols(obj) // [ foo, bar ]
```

*对比ECMAScript 5的实现*
```javascript
// ES5中无该实现
```

#### 全局Symbol

全局的Symbol是通过唯一的键来标志的

*ECMAScript 6的实现*
```javascript
Symbol.for("app.foo") === Symbol.for("app.foo")
const foo = Symbol.for("app.foo")
const bar = Symbol.for("app.bar")
Symbol.keyFor(foo) === "app.foo"
Symbol.keyFor(bar) === "app.bar"
typeof foo === "symbol"
typeof bar === "symbol"
let obj = {}
obj[foo] = "foo"
obj[bar] = "bar"
JSON.stringify(obj) // {}
Object.keys(obj) // []
Object.getOwnPropertyNames(obj) // []
Object.getOwnPropertySymbols(obj) // [ foo, bar ]
```

*对比ECMAScript 5的实现*
```javascript
// ES5中无该实现
```

###  迭代器 (Iterators)

#### 迭代器和For-Of 操作符

通过“迭代”协议支持对象自定义它们的迭代行为。
同时支持通过“迭代”协议来生成一些列的结果值 (无论有限还是无限的)
最后，提供了方便的操作符（For-of）遍历一个迭代对象的所有值. 

*ECMAScript 6的实现*
```javascript
let fibonacci = {
    [Symbol.iterator]() { // 声明迭代器
        let pre = 0, cur = 1
        return {
           next () { // 迭代器每次迭代的函数
               [ pre, cur ] = [ cur, pre + cur ]
               return { done: false, value: cur } // 每次迭代值返回
           }
        }
    }
}

for (let n of fibonacci) { // 这里n的值是在 二 中介绍的释放赋值
    if (n > 1000)
        break
    console.log(n)
}
```

*对比ECMAScript 5的实现*
```javascript
var fibonacci = {  // 译者注： 还是觉得ES5这种写法更直观，迭代器是ES6中才引入的，之后怎么变化还未知
    next: (function () {
        var pre = 0, cur = 1;
        return function () {
            tmp = pre;
            pre = cur;
            cur += tmp;
            return cur;
        };
    })()
};

var n;
for (;;) {
    n = fibonacci.next();
    if (n > 1000)
        break;
    console.log(n);
}
```


### 生产者

### 生产者模式，迭代协议

支持迭代形式的生产函数，通过控制流程的暂停和恢复来产生一些列的值 (无论有限还是无限)

*ECMAScript 6的实现*
```javascript
let fibonacci = {
    *[Symbol.iterator]() {  // * 来标示生产者
        let pre = 0, cur = 1
        for (;;) {
            [ pre, cur ] = [ cur, pre + cur ]
            yield cur
        }
    }
}

for (let n of fibonacci) {
    if (n > 1000)
        break
    console.log(n)
}
```

*对比ECMAScript 5的实现*
```javascript
var fibonacci = {
    next: (function () {
        var pre = 0, cur = 1;
        return function () {
            tmp = pre;
            pre = cur;
            cur += tmp;
            return cur;
        };
    })()
};

var n;
for (;;) {
    n = fibonacci.next();
    if (n > 1000)
        break;
    console.log(n);
}
```
#### 直接使用生产者函数

支持生产者函数，一种特别的函数， 通过控制流程的暂停和恢复来产生一些列的值 (无论有限还是无限)


*ECMAScript 6的实现*
```javascript
function* range (start, end, step) {  // * 标示这是一个生产者函数
    while (start < end) {
        yield start   // 通过yieldl来输出中间结果并暂停
        start += step  // 恢复生产者的执行
    }
}

for (let i of range(0, 10, 2)) { // 对生产者函数的使用 
    console.log(i) // 0, 2, 4, 6, 8
}
```

*对比ECMAScript 5的实现*
```javascript
 // ES5中无相对应的实现
```



#### 生产者匹配 
支持生产者函数产生的值分配到各种可能的情况中去

*ECMAScript 6的实现*
```javascript
let fibonacci = function* (numbers) { // 定义生产者函数
    let pre = 0, cur = 1
    while (numbers-- > 0) {
        [ pre, cur ] = [ cur, pre + cur ]
        yield cur
    }
}

for (let n of fibonacci(1000)) // 使用在数组遍历中
    console.log(n)

let numbers = [ ...fibonacci(1000) ] // 使用在数组赋值中

let [ n1, n2, n3, ...others ] = fibonacci(1000) // 使用在数组释放赋值中
```

*对比ECMAScript 5的实现*
```javascript
// 无对应的ES5实现
```

####  生产者流程控制 

支持生产者, 一种可以控制流程暂停/恢复的特殊迭代器, 以协作支持和Promises这样的异步编程. [请注意，以下的async函数通常使用已经存在的库，例如[co](https://github.com/tj/co) 或者[Bluebird](https://github.com/petkaantonov/bluebird) ]

*ECMAScript 6的实现*
```javascript
// 生成异步的流程控制驱动函数
function async (proc, ...params) {
    var iterator = proc(...params) // proc 是一个生产者函数 (带*)
    return new Promise((resolve, reject) => {
        let loop = (value) => {
            let result
            try {
                result = iterator.next(value)
            }
            catch (err) {
                reject(err)
            }
            if (result.done)
                resolve(result.value)
            else if (   typeof result.value      === "object"
                     && typeof result.value.then === "function")
                result.value.then((value) => {
                    loop(value)
                }, (err) => {
                    reject(err)
                })
            else
                loop(result.value)
        }
        loop()
    })
}

//  application-specific asynchronous builder
function makeAsync (text, after) {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve(text), after)
    })
}

//  application-specific asynchronous procedure
async(function* (greeting) {
    let foo = yield makeAsync("foo", 300)
    let bar = yield makeAsync("bar", 200)
    let baz = yield makeAsync("baz", 100)
    return `${greeting} ${foo} ${bar} ${baz}`
}, "Hello").then((msg) => {
    console.log("RESULT:", msg) // "Hello foo bar baz"
})

```

*对比ECMAScript 5的实现*
```javascript
 // ES5中无相对应的实现

```

#### 生产者方法

类或者对象中也支持生产者的方法


*ECMAScript 6的实现*
```javascript
class Clz {
    * bar () { // 类中的生产者方法
        …
    }
}
let Obj = {
    * foo () {  // 对象中的生产者方法
        …
    }
}
```

*对比ECMAScript 5的实现*
```javascript
 // ES5中无相对应的实现
```


### Map/Set 和WeakMap WeakSet

#### Set 数据格式

新增Set数据格式， 更清晰/方便的操作集合 

*ECMAScript 6的实现*
```javascript
let s = new Set()  
s.add("hello").add("goodbye").add("hello") // 添加成员
s.size === 2  // 成员数量
s.has("hello") === true  // 成员存在
for (let key of s.values()) // 遍历成员 -  通过添加的顺序(FIFO)
    console.log(key)
```

*对比ECMAScript 5的实现*
```javascript 
// set的出现的确让之前这类的操作便利性增加不少！
var s = {}; 
s["hello"] = true; s["goodbye"] = true; s["hello"] = true;
Object.keys(s).length === 2;
s["hello"] === true;
for (var key in s) // arbitrary order
    if (s.hasOwnProperty(key))
        console.log(s[key]);
```

#### Map数据格式 

新增Map数据格式， 更清晰/方便的操作数据映射

*ECMAScript 6的实现*
```javascript
// 增加了几个方案的map操作，但总体上和之前的对象属性并无太大差别

let m = new Map()
let s = Symbol()
m.set("hello", 42) 
m.set(s, 34)
m.get(s) === 34
m.size === 2
for (let [ key, val ] of m.entries()) 
    console.log(key + " = " + val)

```

*对比ECMAScript 5的实现*
```javascript
var m = {};
// ES5中无
m["hello"] = 42;
// ES5中无
// ES5中无
Object.keys(m).length === 2;
for (key in m) {
    if (m.hasOwnProperty(key)) {
        var val = m[key];
        console.log(key + " = " + val);
    }
}
```

#### 弱指向的数据格式

对 Set和Map而言可以在前面增加关键字Weak, 这样能保证其中的对象只是引用关系，对象在WeakSet和WeakMap外发送的变化都会体现在其中。 能够保证无内存泄漏问题。

*ECMAScript 6的实现*
```javascript
let isMarked     = new WeakSet()
let attachedData = new WeakMap()

export class Node {
    constructor (id)   { this.id = id                  }
    mark        ()     { isMarked.add(this)            }
    unmark      ()     { isMarked.delete(this)         }
    marked      ()     { return isMarked.has(this)     }
    set data    (data) { attachedData.set(this, data)  }
    get data    ()     { return attachedData.get(this) }
}

let foo = new Node("foo")

JSON.stringify(foo) === '{"id":"foo"}'
foo.mark()
foo.data = "bar"
foo.data === "bar"
JSON.stringify(foo) === '{"id":"foo"}'

isMarked.has(foo)     === true
attachedData.has(foo) === true
foo = null  /* 释放foo 同时也释放了WeakSet和WeakMap中的foo */
attachedData.has(foo) === false
isMarked.has(foo)     === false
```

*对比ECMAScript 5的实现*
```javascript
// ES5中无
```

### 类型数组
#### 类型数组

类型数组是一种从二进制的数组空间里构建的数组，这可以方便的用来实现各种网络协议，加密算法和文件格式的操作。

*ECMAScript 6的实现*
```javascript
//  ES6 下面例子中的Example类 同等于 这个c 的结构:
//  struct Example { unsigned long id; char username[16]; float amountDue }
class Example {
    constructor (buffer = new ArrayBuffer(24)) {
        this.buffer = buffer
    }
    set buffer (buffer) {
        this._buffer    = buffer
        this._id        = new Uint32Array (this._buffer,  0,  1) // 取该buffer从第0字节开始，1个uint32长作为id
        this._username  = new Uint8Array  (this._buffer,  4, 16) // 取该buffer的第4字节开始，开始共16个uint8长作为 username (字符数组)的空间
        this._amountDue = new Float32Array(this._buffer, 20,  1) // 取该buffer中的第20字节开始，共1个float32长作为amountDue的空间。
    }
    get buffer ()     { return this._buffer       }
    set id (v)        { this._id[0] = v           } // 为什么不是._id = v? 因为_id指向的是数组，但数组真正存储的起点是在第一个元素处， 即 ._id[0]  下面其他的元素同理
    get id ()         { return this._id[0]        }
    set username (v)  { this._username[0] = v     }  
    get username ()   { return this._username[0]  }
    set amountDue (v) { this._amountDue[0] = v    }
    get amountDue ()  { return this._amountDue[0] }
}

let example = new Example()
example.id = 7  // 
example.username = "John Doe"
example.amountDue = 42.0

// 这是buffer内存的状况:
// 以16进制标示 每两位标示一个byte 
// 0000 0007 (id)  
// 4a6f 686e 2044 6f65 0000 0000 0000 0000 (John Doe)
//  4228 0000 (42.0)
```



*对比ECMAScript 5的实现*
```javascript
// ES5中无
```




> 原文地址 [ECMAScript 6 — New Features: Overview & Comparison](http://es6-features.org/)
> 本文介绍 ECMAScript 6中的新特性并通过具体代码演示与ECMAScript 5的差别。译者额外增加了一些例子和说明 .

> ECMAScript 6 是2015年推出的第6版本的ECMAScript标准(即Javascript标准)，同时也被成为ECMAScript 2015. 

> ECMAScript 6 中定了了许多新的Javascript特性，包括新的类和模块，类似python的generators和generators表达式, 箭头函数, 二进制数据, 类型数组, 集合类型(maps, set  & 弱maps), promises, reflection, proxy等。它也被称作 ES6 Harmony. (和谐ES6)

> 本文是该系列文章的第四篇

### 新内建方法

####  对象属性的赋值

新的assign函数可以枚举属性并指定一个或者多个对象映射到目标对象中

*ECMAScript 6的实现*
```javascript
var dest = { quux: 0 }
var src1 = { foo: 1, bar: 2 }
var src2 = { foo: 3, baz: 4 }
Object.assign(dest, src1, src2) // assign src1, src2的属性到 dest对象中
dest.quux === 0
dest.foo  === 3
dest.bar  === 2
dest.baz  === 4
```

*对比ECMAScript 5的实现*
```javascript
var dest = { quux: 0 };
var src1 = { foo: 1, bar: 2 };
var src2 = { foo: 3, baz: 4 };
Object.keys(src1).forEach(function(k) { // 需要遍历key并取值赋值
    dest[k] = src1[k];
});
Object.keys(src2).forEach(function(k) {
    dest[k] = src2[k];
});
dest.quux === 0;
dest.foo  === 3;
dest.bar  === 2;
dest.baz  === 4;
```

####  数组元素的寻找
新的方法用来寻找数组中的元素
*ECMAScript 6的实现*
```javascript
// 这个find/findIndex函数只寻找返回满足条件的第一个元素
[ 1, 3, 4, 2 ].find(x => x > 3) // 4
[ 1, 3, 4, 2 ].findIndex(x => x > 3) // 2 
```

*对比ECMAScript 5的实现*
```javascript
[ 1, 3, 4, 2 ].filter(function (x) { return x > 3; })[0]; // 4
//  ES5中无findIndex
```


#### 字符串重复

新的repeat方法来重复字符串

*ECMAScript 6的实现*
```javascript
" ".repeat(4 * depth)
"foo".repeat(3)
```

*对比ECMAScript 5的实现*
```javascript
Array((4 * depth) + 1).join(" ");
Array(3 + 1).join("foo");
```

#### 字符串查找
新的statsWith endsWith和includes 方法用来查找字符串
*ECMAScript 6的实现*
```javascript
"hello".startsWith("ello", 1) // true
"hello".endsWith("hell", 4)   // true
"hello".includes("ell")       // true
"hello".includes("ell", 1)    // true
"hello".includes("ell", 2)    // false
```

*对比ECMAScript 5的实现*
```javascript
"hello".indexOf("ello") === 1;    // true
"hello".indexOf("hell") === (4 - "hell".length); // true
"hello".indexOf("ell") !== -1;    // true
"hello".indexOf("ell", 1) !== -1; // true
"hello".indexOf("ell", 2) !== -1; // false
```

#### 数值类型检查

检查是否为NaN和无穷数值
*ECMAScript 6的实现*
```javascript
Number.isNaN(42) === false
Number.isNaN(NaN) === true

Number.isFinite(Infinity) === false
Number.isFinite(-Infinity) === false
Number.isFinite(NaN) === false
Number.isFinite(123) === true
```

*对比ECMAScript 5的实现*
```javascript
var isNaN = function (n) {
    return n !== n;
};
var isFinite = function (v) {
    return (typeof v === "number" && !isNaN(v) && v !== Infinity && v !== -Infinity);
};
isNaN(42) === false;
isNaN(NaN) === true;

isFinite(Infinity) === false;
isFinite(-Infinity) === false;
isFinite(NaN) === false;
isFinite(123) === true;
```
#### 数值的安全检查
检查一个数字的值是否在安全的范围内， 例如能够正确的以javascript展示(其实说有的数值，包括integer数值，都是浮点数值)

*ECMAScript 6的实现*
```javascript
Number.isSafeInteger(42) === true
Number.isSafeInteger(9007199254740992) === false
```

*对比ECMAScript 5的实现*
```javascript
function isSafeInteger (n) {
    return (
           typeof n === 'number'
        && Math.round(n) === n
        && -(Math.pow(2, 53) - 1) <= n
        && n <= (Math.pow(2, 53) - 1)
    );
}
isSafeInteger(42) === true;
isSafeInteger(9007199254740992) === false;
```

#### 数值的比较
提供了标准的 Epsilon (临界值)，用来作为浮点数直接的比较依据。

*ECMAScript 6的实现*
```javascript
console.log(0.1 + 0.2 === 0.3) // false
console.log(Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON) // true
```

*对比ECMAScript 5的实现*
```javascript
console.log(0.1 + 0.2 === 0.3); // false
console.log(Math.abs((0.1 + 0.2) - 0.3) < 2.220446049250313e-16); // true
```

#### 数值截断
讲浮点(float)数截断为整型(integer)数值, 这里的截断是完全去掉小数点后的值
*ECMAScript 6的实现*
```javascript
console.log(Math.trunc(42.7)) // 42
console.log(Math.trunc( 0.1)) // 0
console.log(Math.trunc(-0.1)) // -0
```

*对比ECMAScript 5的实现*
```javascript
function mathTrunc (x) {
    return (x < 0 ? Math.ceil(x) : Math.floor(x));
}
console.log(mathTrunc(42.7)) // 42
console.log(mathTrunc( 0.1)) // 0
console.log(mathTrunc(-0.1)) // -0
```


#### 数值符号选定

确定一个数值的符号， 包括带符号0和非数值.

*ECMAScript 6的实现*
```javascript
console.log(Math.sign(7))   // 1
console.log(Math.sign(0))   // 0
console.log(Math.sign(-0))  // -0
console.log(Math.sign(-7))  // -1
console.log(Math.sign(NaN)) // NaN
```

*对比ECMAScript 5的实现*
```javascript
function mathSign (x) {
    return ((x === 0 || isNaN(x)) ? x : (x > 0 ? 1 : -1));
}
console.log(mathSign(7))   // 1
console.log(mathSign(0))   // 0
console.log(mathSign(-0))  // -0
console.log(mathSign(-7))  // -1
console.log(mathSign(NaN)) // NaN
```


### Promise

#### Promise的使用

首先是关于如果表述一个异步在未来可以获取到的值的表达式. 更多[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)的信息.
*ECMAScript 6的实现*
```javascript
function msgAfterTimeout (msg, who, timeout) {
    return new Promise((resolve, reject) => { // 返回的promise, 会异步，在未来的某个时间调用resolve来完成
        setTimeout(() => resolve(`${msg} Hello ${who}!`), timeout)
    })
}
msgAfterTimeout("", "Foo", 100).then((msg) => //  对于完成的promise需要通过then来继续实现
    msgAfterTimeout(msg, "Bar", 200)
).then((msg) => {
    console.log(`done after 300ms:${msg}`)
})
```

*对比ECMAScript 5的实现*
```javascript
function msgAfterTimeout (msg, who, timeout, onDone) {
    setTimeout(function () {
        onDone(msg + " Hello " + who + "!");
    }, timeout);
}
msgAfterTimeout("", "Foo", 100, function (msg) {
    msgAfterTimeout(msg, "Bar", 200, function (msg) {
        console.log("done after 300ms:" + msg);
    });
});
```

#### Promise的联合
同时也可以把多个promise联合到一个新的promise, 等待他们的完成，无需关心他们的中间执行顺序。

 *ECMAScript 6的实现*
```javascript
function fetchAsync (url, timeout, onData, onError) {
    …
}
let fetchPromised = (url, timeout) => {
    return new Promise((resolve, reject) => {
        fetchAsync(url, timeout, resolve, reject)
    })
}
Promise.all([
    fetchPromised("http://backend/foo.txt", 500),
    fetchPromised("http://backend/bar.txt", 500),
    fetchPromised("http://backend/baz.txt", 500)
]).then((data) => {
    let [ foo, bar, baz ] = data // 返回的数值是按照promise排列的顺序执行
    console.log(`success: foo=${foo} bar=${bar} baz=${baz}`)
}, (err) => {
    console.log(`error: ${err}`)
})
```

*对比ECMAScript 5的实现*
```javascript
function fetchAsync (url, timeout, onData, onError) {
    …
}
function fetchAll (request, onData, onError) {
    var result = [], results = 0;
    for (var i = 0; i < request.length; i++) {
        result[i] = null;
        (function (i) {
            fetchAsync(request[i].url, request[i].timeout, function (data) {
                result[i] = data;
                if (++results === request.length)
                    onData(result);
            }, onError);
        })(i);
    }
}
fetchAll([
    { url: "http://backend/foo.txt", timeout: 500 },
    { url: "http://backend/bar.txt", timeout: 500 },
    { url: "http://backend/baz.txt", timeout: 500 }
], function (data) {
    var foo = data[0], bar = data[1], baz = data[2];
    console.log("success: foo=" + foo + " bar=" + bar + " baz=" + baz);
}, function (err) {
    console.log("error: " + err);
});
```



### 元程序

#### Proxying
Proxy能够嵌入运行时对象的元操作.

*ECMAScript 6的实现*
```javascript
let target = {
    foo: "Welcome, foo"
}
let proxy = new Proxy(target, { // 定义了一个针对对象的proxy, 重定义了get元操作.
    get (receiver, name) {
        return name in receiver ? receiver[name] : `Hello, ${name}`
    }
})
proxy.foo   === "Welcome, foo"
proxy.world === "Hello, world"
```

*对比ECMAScript 5的实现*
```javascript
// ES5中无
```

#### Reflection

针对Object的一系列操作，和之前的Object.方法不同的是，reflection的返回值是有意义的。
Reflection提供了一系列的操作方法，这里的例子中只有一个，更详细的请看[这里](https://h3manth.com/new/blog/2015/es6-reflect-api/) - 译者会跟进翻译这篇文章。

*ECMAScript 6的实现*
```javascript
let obj = { a: 1 }
Object.defineProperty(obj, "b", { value: 2 })
obj[Symbol("c")] = 3
Reflect.ownKeys(obj) // [ "a", "b", Symbol(c) ] // 这里只介绍了ownKeys这个方法 
```

*对比ECMAScript 5的实现*
```javascript
var obj = { a: 1 };
Object.defineProperty(obj, "b", { value: 2 });
// no equivalent in ES5
Object.getOwnPropertyNames(obj); // [ "a", "b" ]
```













