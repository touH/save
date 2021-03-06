- [开始](#开始)
    - [使用Object.assign定义多个方法](#使用Object.assign定义多个方法)
    - [采用表达式写属性名](#采用表达式写属性名)
- [constructor](#constructor)
- [类的实例对象](#类的实例对象)
- [Class表达式](#class表达式)
- [不存在变量提升](#不存在变量提升)
- [私有方法和私有属性](#私有方法和私有属性)
- [this的指向](#this的指向)
- [name属性](#name属性)
- [getter和setter](#getter和setter)
- [Class的Generator方法](#class的generator方法)
- [Class的静态方法](#class的静态方法)
- [Class的静态属性和实例属性](#class的静态属性和实例属性)
- [new.target属性](#new_target)


类`class`不存在提升的问题， 函数和变量存在提升问题
### 开始
传统方法
```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return `(${this.x}, ${this.y})`;
};

var p = new Point(1, 2);
```
es6
```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
var p = new Point(1, 2);
```
- `constructor`就是构造方法，es6的类中称做构造函数，就是对应ES5中的构造函数。
- `toString()`方法对应ES5中`Point.prototype.toString`原型中定义的方法。在类上面定义的方法实际上就是在原型对象上定义。
- `typeof(Point) === 'function'`，类的数据类型就是函数。
- `Point === Point.prototype.constructor`，类本身就指向构造函数。构造函数的`prototype`属性在类上依旧存在。
- 类的内部所有定义的方法，都是不可枚举的。但在ES5中是可以被枚举的。
- 类必须使用new调用，否则会报错。

##### 使用Object.assign定义多个方法
由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。
```js
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```

##### 采用表达式写属性名
```js
let methodName = 'getArea';

class Square {
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

### constructor
`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。

`constructor`方法默认返回实例对象（即this），完全可以指定返回另外一个对象。
```js
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo   // false
```

### 类的实例对象
与 `ES5` 一样，实例的属性除非显式定义在其本身（即定义在`this`对象上），否则都是定义在原型上（即定义在`class`上）。类的所有实例共享一个原型对象。
```js
//定义类
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true

//类的所有实例共享一个原型对象
var p1 = new Point(2,3);
var p2 = new Point(3,2);
p1.__proto__ === p2.__proto__   //true

// 通过实例的__proto__属性为“类”添加方法
p1.__proto__.printName = function () { return 'Oops' };
```

### Class表达式
与函数一样，类也可以使用表达式的形式定义。
```js
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
let inst = new MyClass();
inst.getClassName()     // Me
Me.name                 // ReferenceError: Me is not defined
```
上面代码表示，`Me`只在 `Class` 内部有定义。

如果类的内部没用到的话，可以省略`Me`，也就是可以写成下面的形式。
```js
const MyClass = class { /* ... */ };
```

### 不存在变量提升
ES5存在函数声明提升，ES6类不存在变量提升
```js
new Foo(); // ReferenceError
class Foo {}
```

### 私有方法和私有属性
私有方法是常见需求，但 ES6 不提供，只能通过变通方法模拟实现，这里讲到3中方法。

一种做法是在命名上加以区别。
```js
class Widget {

  // 公有方法
  foo (baz) {
    this._bar(baz);
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz;
  }

  // ...
}
```
上面代码中，`_bar`方法前面的下划线，表示这是一个只限于内部使用的私有方法。但是，这种命名是不保险的，在类的外部，还是可以调用到这个方法。

另一种方法就是索性将私有方法移出模块，因为模块内部的所有方法都是对外可见的。
```js
class Widget {
  foo (baz) {
    bar.call(this, baz);
  }

  // ...
}

function bar(baz) {
  return this.snaf = baz;
}
```
上面代码中，`foo`是公有方法，内部调用了`bar.call(this, baz)`。这使得`bar`实际上成为了当前模块的私有方法。

还有一种方法是利用`Symbol`值的唯一性，将私有方法的名字命名为一个`Symbol`值。
```js
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

### this的指向
类的方法内部如果含有`this`，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。
```js
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```
如果将这个方法提取出来单独使用，`this`会指向该方法运行时所在的环境，因为找不到`print`方法而导致报错。

一个比较简单的解决方法是，在构造方法中绑定this，这样就不会找不到print方法了。
```js
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}
```
另一种解决方法是使用箭头函数。
```js
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }

  // ...
}
```
还有一种解决方法是使用Proxy，获取方法的时候，自动绑定this。
```js
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    get (target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}

const logger = selfish(new Logger());
```

### name属性
由于本质上，`ES6` 的类只是 `ES5` 的构造函数的一层包装，所以函数的许多特性都被`Class`继承，包括`name`属性。
```js
class Point {}
Point.name // "Point"
```

### getter和setter
对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
```js
class MyClass {
  constructor() {
    // ...
  }
  get prop() {          //prop属性
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;    // setter: 123

inst.prop       // 'getter'

inst.__proto__.hasOwnProperty('prop')   //true get set的属性时原型对象上的属性

//下面是得到MyClass.prototype原型对象中定义的prop这个属性的特性
var descriptor = Object.getOwnPropertyDescriptor(
  MyClass.prototype, "prop"
);
"get" in descriptor  // true
"set" in descriptor  // true
//还有configurable和enumerable

```
`prop`这个属性是在`MyClass.prototype`原型对象上被定义的，不在实例上。

### Class的Generator方法
如果某个方法之前加上星号`（*）`，就表示该方法是一个 `Generator` 函数。
```js
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```
上面代码中，`Foo`类的`Symbol.iterator`方法前有一个星号，表示该方法是一个 `Generator` 函数。`Symbol.iterator`方法返回一个`Foo`类的默认遍历器，`for...of`循环会自动调用这个遍历器。

### Class的静态方法
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。
```js
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()   // TypeError: foo.classMethod is not a function
```
注意，如果静态方法包含`this`关键字，这个`this`指的是类，而不是实例。
```js
class Foo {
  static bar () {
    this.baz();     //静态方法中this指的是Foo
  }
  static baz () {
    console.log('hello');
  }
  baz () {
    console.log('world');
  }
}

Foo.bar() // hello
```
父类的静态方法，可以被子类继承。
```js
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```
静态方法也是可以从super对象上调用的。
super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
```js
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod() // "hello, too"
```

### Class的静态属性和实例属性
静态属性指的是 `Class` 本身的属性，即`Class.propName`，而不是定义在实例对象`（this）`上的属性。
```js
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```
目前，只有这种写法可行，因为 `ES6` 明确规定，`Class` 内部只有静态方法，没有静态属性。
```js
// 以下两种写法都无效
class Foo {
  // 写法一
  prop: 2

  // 写法二
  static prop: 2
}

Foo.prop // undefined
```

### <span id='new_target'>new.target属性<span>
`new`是从构造函数生成实例对象的命令。`ES6` 为`new`命令引入了一个`new.target`属性，该属性一般用在构造函数之中，返回`new`命令作用于的那个构造函数。如果构造函数不是通过`new`命令调用的，`new.target`会返回`undefined`，如果是则返回类名。因此这个属性可以用来确定构造函数是怎么调用的。
```js
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
```
`Class` 内部调用`new.target`，返回当前 `Class`。

需要注意的是，子类继承父类时，`new.target`会返回子类。,指向正在执行的类
```js
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);  //此时为Square
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 输出 false
```
利用这个特点，可以写出不能独立使用、必须继承后才能使用的类。