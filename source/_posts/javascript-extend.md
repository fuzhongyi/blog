---
title: javascript 继承
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
console.log( Person.prototype.__proto__ === Object.prototype); // true
console.log( Object.prototype.__proto__ === null); // true
```

## 原型链

每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个可以指向原型对象的内部指针（可以通过 `__proto__` 访问）。

假如我们让原型对象等于另一个类型的实例，那么此时原型对象包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系仍然成立，如此层层递进，就构成了实例与原型的链条，这就是原型链的基本概念。

## 继承

### 原型链继承

原型链继承的基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

```javascript
function Animal() {
    this.type = "动物";
}
Animal.prototype.getType = function() {
    console.log(this.type);
}

function Dog() {}
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog;
Dog.prototype.sound = function() {
    console.log("汪~汪~");
}

var dog = new Dog();
dog.sound(); // 汪~汪~
dog.getType(); // 动物
```

缺点：

1. 通过原型来实现继承时，原型会变成另一个类型的实例，原先的实例属性变成了现在的原型属性，该原型的引用类型属性会被所有的实例共享
2. 在创建子类型的实例时，没有办法在不影响所有对象实例的情况下给超类型的构造函数中传递参数