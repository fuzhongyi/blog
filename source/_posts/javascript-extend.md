---
title: JavaScript 继承的几种方式
date: 2020-04-07 17:15:36
tags: javascript
categories: 技术水波文
---

在了解继承之前，我们先了解构造函数、原型和原型链的相关知识。

## 构造函数

构造函数和普通函数的区别仅在于调用它们的方式不同。只要通过 `new` 操作符来调用，那它就可以作为构造函数；如果不通过 `new` 操作符来调用，那么它就是一个普通函数。

实例拥有 `constructor` 属性，该属性返回创建实例对象的构造函数。

```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
}

var person = new Person('XYue', 20);
console.log(person.constructor === Person); // true
```

## 原型

我们创建的每个函数都有 `prototype` 属性，这个属性指向函数的原型对象。原型对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

在默认情况下，所有原型对象都会自动获得一个 `constructor` 属性，这个属性包含一个指向 `prototype` 属性所在函数的指针。

当调用构造函数创建一个新实例后，该实例的内部将包含一个指针，指向构造函数的原型对象（可以通过实例的 `__proto__` 来访问构造函数的原型对象）。

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayName = function () {
    console.log(this.name);
}

var person1 = new Person('XYue');
var person2 = new Person('歆月');
person1.sayName(); // XYue
person2.sayName(); // 歆月
console.log(person1.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); // true
```

## 原型链

每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个可以指向原型对象的内部指针（可以通过 `__proto__` 访问）。

假如我们让原型对象等于另一个类型的实例，那么此时原型对象包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系仍然成立，如此层层递进，就构成了实例与原型的链条，这就是原型链的基本概念。

## 继承

### 原型链继承

原型链继承的基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

```javascript
function SuperType(name) {
    this.name = name;
    this.hobbies = ['唱', '跳', 'rap'];
}
SuperType.prototype.getHobbies = function() {
    console.log(this.hobbies);
}

function SubType() {}
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;

var sub1 = new SubType();
var sub2 = new SubType();
sub1.hobbies.push('篮球');
sub1.getHobbies(); // ['唱', '跳', 'rap', '篮球']
sub2.getHobbies(); // ['唱', '跳', 'rap', '篮球']
```

+ 缺点
  1. 通过原型来实现继承时，原型会变成另一个类型的实例，原先的实例属性变成了现在的原型属性，该原型的引用类型属性会被所有的实例共享
  2. 在创建子类型的实例时，没有办法在不影响所有对象实例的情况下给超类型的构造函数中传递参数

### 构造函数继承

基本思路：在子类型的构造函数中调用超类型构造函数。

```javascript
function SuperType(name) {
    this.name = name;
    this.hobbies = ['唱', '跳', 'rap'];
}
SuperType.prototype.getHobbies = function() {
    console.log(this.hobbies);
}

function SubType(name) {
    SuperType.call(this, name);
}

var sub1 = new SubType('XYue-1');
var sub2 = new SubType('XYue-2');
sub1.hobbies.push('篮球');
console.log(sub1.hobbies); // ['唱', '跳', 'rap', '篮球'];
console.log(sub2.hobbies); // ['唱', '跳', 'rap'];
sub1.getHobbies(); // Uncaught TypeError: sub1.getHobbies is not a function
```

+ 优点
  1. 可以向超类传递参数
  2. 解决了原型中包含引用类型值被所有实例共享的问题
+ 缺点
  1. 方法都在构造函数中定义，函数复用无从谈起，另外超类型原型中定义的方法对于子类型而言都是不可见的。

### 组合继承

组合继承指的是将原型链和借用构造函数技术组合到一块，从而发挥二者之长的一种继承模式。

基本思路：使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对实例属性的继承，既通过在原型上定义方法来实现了函数复用，又保证了每个实例都有自己的属性。

```javascript
function SuperType(name) {
    this.name = name;
    this.hobbies = ['唱', '跳', 'rap'];
}
SuperType.prototype.getHobbies = function() {
    console.log(this.hobbies);
}

function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.getAge = function() {
    console.log(this.age);
}

var sub1 = new SubType('XYue1', 18);
var sub2 = new SubType('XYue2', 20);
sub1.hobbies.push('篮球');
console.log(sub1.hobbies); // ['唱', '跳', 'rap', '篮球'];
console.log(sub2.hobbies); // ['唱', '跳', 'rap'];
sub1.getHobbies(); // ['唱', '跳', 'rap', '篮球'];
sub1.getAge(); // 18
sub2.getAge(); // 20
```

+ 优点
  1. 可以向超类传递参数
  2. 每个实例都有自己的属性
  3. 实现了函数复用
+ 缺点
  1. 无论什么情况下，都会调用两次超类型构造函数。一次是在创建子类型原型的时候，另一次是在子类型构造函数内部

### 原型式继承

借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。

```javascript
function object(obj){
    function F(){};
    F.prototype = obj;
    return new F();
}
```

在 `object()` 函数内部，先添加一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回了这个临时类型的一个新实例，从本质上讲，`object()` 对传入的对象执行了一次浅拷贝。
`ECMAScript5` 通过新增 `Object.create()` 方法规范了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象（可以覆盖原型对象上的同名属性），在传入一个参数的情况下，`Object.create()` 和 `object()` 方法的行为相同。

```javascript
var person = {
    name: 'XYue',
    hobbies: ['唱', '跳', 'rap']
};
var person1 = Object.create(person);
var person2 = Object.create(person);
person1.hobbies.push('🏀');
person2.hobbies.push('⚽️');
console.log(person.hobbies); // ["唱", "跳", "rap", "🏀", "⚽️"]
console.log(person1.hobbies); // ["唱", "跳", "rap", "🏀", "⚽️"]
console.log(person2.hobbies); // ["唱", "跳", "rap", "🏀", "⚽️"]
```

+ 缺点
  1. 同原型链实现继承一样，包含引用类型值的属性会被所有实例共享

### 寄生式继承

`寄生继承` 是依托于一个对象而生的一种继承方式，因此称之为 `寄生`。

寄生式继承是与原型式继承紧密相关的一种思路。寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部已某种方式来增强对象，最后再像真的是它做了所有工作一样返回对象。

```javascript
function createObj(obj) {
    var clone = Object.create(obj);
    clone.sing = function() {
        console.log('大山的子孙呦~爱太阳啰~');
    }
    return clone;
}

var person = {
    name: 'XYue',
    hobbies: ['唱', '跳', 'rap']
};
var person1 = createObj(person);
person1.sing(); // 大山的子孙呦~爱太阳啰~
```

+ 缺点
  1. 使用寄生式继承来为对象添加函数，会由于不能做到函数复用而效率低下
  2. 同原型链实现继承一样，包含引用类型值的属性会被所有实例共享

### 寄生组合式继承

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链继承的形式来继承方法。

基本思路：不必为了指定子类型的原型而调用超类型的构造函数，我们需要的仅是超类型原型的一个副本，本质上就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。

```javascript
function inheritPrototype(subType, superType) {
    // 创建超类型原型的一个副本
    var prototype = Object.create(superType.prototype);
    // 为创建的副本添加 constructor 属性
    prototype.constructor = subType;
    // 将新创建的对象赋值给子类型的原型
    subType.prototype = prototype;
}
```

至此，我们就可以通过调用 `inheritPrototype` 为子类型原型赋值：

```javascript
function SuperType(name) {
    this.name = name;
    this.hobbies = ['唱', '跳', 'rap'];
}
function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}
inheritPrototype(SubType, SuperType);
```

+ 优点
  1. 只调用了一次超类构造函数，效率更高
  2. 避免在 `SubType.prototype` 上面创建不必要的、多余的属性，与其同时，原型链还能保持不变

因此寄生组合继承是引用类型最理性的继承范式。

### ES6 继承

`Class` 可以通过 extends 关键字实现继承，如:

```javascript
class SuperType {
    constructor(name) {
        this.name = name;
        this.hobbies = ['唱', '跳', 'rap'];
    }
    getName() {
        console.log(this.name);
    }
}

class SubType extends SuperType {
    constructor(name, age) {
        super(name);
        this.age = age;
    }
}

const sub = new SubType('XYue', 20);
sub.getName(); // XYue
```

使用 `extends` 关键字实现继承，有一点需要特别说明：

子类必须在 `constructor` 中调用 `super` 方法，否则新建实例时会报错。如果没有子类没有定义 `constructor` 方法，那么这个方法会被默认添加。在子类的构造函数中，只有调用 `super` 之后，才能使用 `this` 关键字，否则报错。这是因为子类实例的构建，基于父类实例，只有 `super` 方法才能调用父类实例。
