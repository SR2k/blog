---
title: JavaScript中的面向对象
date: 2018-01-12 19:24:14
tags:
  - JavaScript
  - 读书笔记
  - JavaScript高级程序设计
---

# 对象简介

在JS中创建一个对象十分简单，可以使用`Object`构造方法，也可以直接使用对象字面量：

``` javascript
const person = {
  name: "Siyuan",
  sayName() {
    console.log(this.name);
  }
};
```

## 属性类型

JS引擎在内部使用的特性，描述了属性的各种特征。

### 数据类型

- `[[Configurable]]`：能否通过`delete`删除属性、能否修改属性的特性、或者能否把属性修改为访问器属性。默认值为`true`；
- `[[Enumerable]]`：能否通过`for-in`循环遍历到此属性。默认值为`true`；
- `[[Writable]]`：能否修改属性的值。默认值为`true`；
- `[[Value]]`：此属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。默认值为`undefined`。

ES5的`Object.defineProperty()`方法可以修改上述各值。这个方法接收三个参数：

- 属性所在的对象；
- 属性的名字；
- 描述符对象。

其中，描述符（descriptor）对象的属性必须是：`configurable`、`enumerable`、`writable`和`value`，如：

``` javascript
const person = {};
Object.defineProperty(person, "name", {
  writable: false,
  value: "Siyuan" 
});

console.log(person.name);    //"Siyuan"
person.name = "Not Siyuan";
console.log(person.name);    //"Siyuan"

// 如果是严格模式下会报错
```

如果把`configurable`设置为`false`，就不能再把它变回可配置了。此时，再调用`Object.defineProperty()`方法修改除`writable`之外的特性，都会导致错误。也就是说，可以多次调用`Object.defineProperty()`方法修改同一个属性，但在把`configurable`特性设置为`false`之后就会有限制了。

### 访问器属性

访问器属性包含一对可选的`getter`和`setter`函数。他们函数负责决定如何处理数据。访问器属性有如下4个特性：

- `[[Configurable]]`；
- `[[Enumerable]]`；
- `[[Get]]`：在读取属性时调用的函数。默认值为`undefined`；
- `[[Set]]`：在写入属性时调用的函数。默认值为`undefined`。

访问器属性不能直接定义，必须使用`Object.defineProperty()`来定义：

``` javascript
const sr2k = { 
  _absCount: 6 // 嗯谁不想要腹肌捏 ///w//
};

Object.defineProperty(sr2k, "absCount", {
  get() {
    return this._absCount;
  },
  set(newValue) {
  	// 正常人不会有超过8块的腹肌吧…
    this._absCount = newValue > 8 ? 8 : newValue; 
  }
});

sr2k.absCount = 12;
console.log(sr2k.absCount); // 8
```

## 定义多个属性

ES5又定义了一个`Object.defineProperties()`方法，可以一次定义多个属性。这个方法接收两个对象参数，如：

``` javascript
const sr2k = {};

Object.defineProperties(sr2k, {
  _hadBreakfast: {
    writable: true,
    value: false
  },

  hungry: {
    writable: true,
    value: true
  },

  hadBreakfast: {
    get() {
      return this._hadBreakfast;
    },
    set(newValue) {
      this._hadBreakfast = newValue;
      // 吃了早饭就不饿啦
      this.hungry = !newValue;
    }
  }
});
```

### 读取属性的特性

使用ES5的`Object.getOwnPropertyDescriptor()`方法，可以取得给定属性的描述符：

如上文吃早饭的例子：

``` javascript
let descriptor = Object.getOwnPropertyDescriptor(book, "_hadBreakfast");
console.log(descriptor.value);        // false
console.log(descriptor.configurable); // false
console.log(typeof descriptor.get);   // "undefined"

descriptor = Object.getOwnPropertyDescriptor(book, "hadBreakfast");
console.log(descriptor.value);        // undefined
console.log(descriptor.enumerable);   // false
console.log(typeof descriptor.get);   // "function"
```

# 创建对象

## 工厂模式

``` javascript
function createPerson(name, age) {
  let obj = new Object();
  obj.name = name;
  obj.age  = age;
  return obj;
}
```

- 解决了创建多个相似对象的问题；
- 没有解决对象识别的问题（即怎样知道一个对象的类型）。

## 构造函数模式

与工厂模式相比，构造函数：

- 没有显式地创建对象；
- 直接将属性和方法赋给了`this`对象；
- 没有`return`语句。

``` javascript
function Person(name, age) {
	// this就指向了这个new出来的新对象
  this.name = name;
  this.age  = age;
  this.sayName = function() {
    alert(this.name);
  };    
}

const sr2k = new Person("Siyuan", 22);
console.log(sr2k instanceof Object);      // true
console.log(sr2k instanceof Person);      // true
console.log(sr2k.constructor === Person); // true
```

### 将构造函数当作函数

构造函数也是函数，不存在定义构造函数的特殊语法。任何函数，只要通过`new`操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过`new`操作符来调用，那它跟普通函数也不会有什么两样。

在使用`new`操作符时，函数内`this`就指向了这个`new`出来的新对象；而不使用`this`时，就依据原来的上下文判断`this`了：

``` javascript
// 当作构造函数使用，this指向new出来的person
const person = new Person("Siyuan", 22);
person.sayName(); //"Nicholas"

// 作为普通函数调用，this指向window
Person("Siyuan", 22);
window.sayName(); //"Greg"

// 在另一个对象的作用域中调用，this指向这个对象
const o = new Object();
Person.call(o, "Siyuan", 22);
o.sayName();     //"Siyuan"
```

### 构造函数的问题

每个方法都要在每个实例上重新创建一遍，无法实现继承。且每个对象都有自己的方法，虽然作用一样，却不是同一个`Function`对象：

``` javascript
new Person("", 0).sayName === new Person("", 0).sayName // false
```

## 原型模式

每个函数都有一个`prototype`（原型）属性，这个属性指向一个对象，而这个对象的用途是**包含可以由特定类型的所有实例共享的属性和方法**。

``` javascript
function Person() {}

Person.prototype.name = "Siyuan";
Person.prototype.age = 22;
Person.prototype.sayName = function() {
  console.log(this.name);
};

const person1 = new Person();
person1.sayName();   //"Siyuan"

const person2 = new Person();
person2.sayName();   //"Siyuan"

console.log(person1.sayName == person2.sayName); // true
```

### 理解原型对象

只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个`prototype`属性，这个属性指向函数的**原型对象**。

在默认情况下，所有原型对象都会自动获得一个`constructor`（构造函数）属性，这个属性是一个指向`prototype`属性所在函数的指针。

就拿前面的例子来说，`Person.prototype.constructor`指向`Person`。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

创建了自定义的构造函数之后，其原型对象默认只会取得`constructor`属性；至于其他方法，则都是从`Object`继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。ES5中称作针叫`[[Prototype]]`。虽然在脚本中没有标准的方式访问`[[Prototype]]`，但大部分浏览器在每个对象上都支持一个属性`__proto__`。

更加标准地获得`[[Prototype]]`的方法是使用ES5的`Object.getPrototypeOf()`，在所有支持的实现中，这个方法返回`[[Prototype]]`的值。

``` mermaid
graph TD
subgraph a
	func[function]
  prototype
end

subgraph b
	obj[object]
  proto[__proto__]
end

subgraph c
  protoObj[prototypeObject]
  con[constructor]
end

prototype --> protoObj
constructor --> func
proto --> protoObj
```

读取某个对象的某个属性时，查找这个属性的顺序如下：

- 如果在实例中找到了具有给定名字的属性，则返回该属性的值；
- 如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性；如果在原型对象中找到了这个属性，否则继续查找原型对象的原型对象；
- 直到查找到`Object`，还是没有找到这个属性，则返回`undefined`。

使用`hasOwnProperty`方法可以检测一个属性是存在于实例中，还是存在于原型中。这个方法是从`Object`继承来的），它只在给定属性存在于对象实例中时，才会返回`true`。来看下面这个例子。

### 原型与in操作符

有两种方式使用`in`操作符：单独使用和在`for-in`循环中使用。

在单独使用时，`in`操作符会在通过对象能够访问给定属性时返回`true`，**无论**该属性存在于实例中还是原型中。

由于`in`操作符只要通过对象能够访问到属性就返回`true`，`hasOwnProperty()`只在属性存在于实例中时才返回`true`，因此只要`in`操作符返回`true`而`hasOwnProperty()`返回`false`，就可以确定属性是原型中的属性。

在使用`for-in`循环时，返回的是所有能够通过对象访问的、可枚举的（`enumerated`）属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。屏蔽了原型中不可枚举属性（即将`[[Enumerable]]`标记为`false`属性）的实例属性也会在`for-in`循环中返回。

要取得对象自身的所有可枚举的实例属性，可以使用ES5的`Object.keys()`方法。这个方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。

### 更简单的原型语法

每次都写一个`Func.prototype.xxx = xxx;`十分繁琐，因此更常见的做法是用一个对象字面量来重写整个原型对象：

``` javascript
function Person() {};

Person.prototype = {
  name : "siyuan",
  age : 22,
  sayName : function() {
    console.log(this.name);
  }
};
```

这样做与之前大体相同，不过有一点区别：`Person.prototype`不再带有指向`Person`的`constructor`属性了，因此只能通过`instanceof`操作符来判断对象的类型。

不过你也可以给`prototype`手动加上`constructor`：

``` javascript
function Person() {};

Person.prototype = {
  name : "siyuan",
  age : 22,
  sayName : function() {
    console.log(this.name);
  }
};

Object.defineProperty(Person.prototype, "constructor", {
	// 要保证constructor是不可枚举的
	enumerable: false,
  value: Person
});
```

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来，即使是先创建了实例后修改原型。

所有原生的引用类型（`Object`、`Array`、`String`，etc）都是采用这种原型模式创建的。

所有原生引用类型都在其构造函数的原型上定义了方法。例如在`Array.prototype`中可以找到`sort()`方法，而在`String.prototype`中可以找到`substring()`方法。

### 原型对象的问题

原型模式省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下**都将取得相同的属性值**。

原型中所有属性是被很多实例共享的，对于包含引用类型值的属性来说，这个问题很大很大：

``` javascript
function Person() {}

Person.prototype = {
  constructor: Person,
  name : "Siyuan",
  age : 22,
  friends : ["Eddy", "Jeffrey"],
  sayName : function () {
    alert(this.name);
  }
};

const person1 = new Person();
const person2 = new Person();

person1.friends.push("Amanda");

console.log(person1.friends);    // ["Eddy", "Jeffrey", "Amanda"]
console.log(person2.friends);    // ["Eddy", "Jeffrey", "Amanda"]
console.log(person1.friends === person2.friends);  //true
```

## 组合继承

组合继承就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。

这样每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数。

``` javascript
function Person(name, age){
	this.name = name;
  this.age = age;
  this.friends = ["Shelby", "Court"];
}

Person.prototype = {
	constructor : Person,
  sayName : function(){
    alert(this.name);
  }
}

const person1 = new Person("Siyuan", 22);
const person2 = new Person("Eddy", 23);

person1.friends.push("Van");
alert(person1.friends);    // "Shelby,Count,Van"
alert(person2.friends);    // "Shelby,Count"
alert(person1.friends === person2.friends);    //false
alert(person1.sayName === person2.sayName);    //true
```

## 动态原型模式

动态原型模式把所有信息都封装在了构造函数中，通过在构造函数中初始化原型（仅在必要的情况下），又保持了同时使用构造函数和原型的优点。

``` javascript
function Person(name, age) {
  // 属性
  this.name = name;
  this.age = age;

  // 方法
  if (typeof this.sayName != "function") {
    Person.prototype.sayName = function() {
      alert(this.name);
    };
  }
}
```

## 寄生构造函数模式

在前述的几种模式都不适用的情况下，可以使用寄生（parasitic）构造函数模式：创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数。假设你想创建一个具有额外方法的特殊数组，但又不能直接修改`Array`构造函数，因此可以使用这个模式：

``` javascript
function SpecialArray() {
  // 创建数组
  const values = new Array();
  // 添加值
  values.push.apply(values, arguments);
	// 添加方法
  values.toPipedString = function(){
    return this.join("|");
  };
  // 返回数组
  return values;
}

const colors = new SpecialArray("red", "blue", "green");
```

> **注意**：返回的对象与构造函数或者与构造函数的原型属性之间没有关系；也就是说，构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此，不能依赖`instanceof`操作符来确定对象类型。由于存在上述问题，建议在可以使用其他模式的情况下，不要使用这种模式。

## 稳妥构造函数模式

所谓稳妥对象，指的是没有公共属性，而且其方法也不引用`this`的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用`this`和`new`），或者在防止数据被其他应用程序（如Mashup程序）改动时使用。

稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：

- 新创建对象的实例方法不引用`this`；
- 不使用`new`操作符调用构造函数。

按照稳妥构造函数的要求，可以将前面的`Person`构造函数重写如下：

``` javascript
function Person(name, age) {
  // 创建要返回的对象
  const o = new Object();

  // ...定义私有变量和函数

	//添加方法
  o.sayName = function() {
  	alert(name);
  };    
	// 返回对象
  return o;
}
```

这样，变量`friend`中保存的是一个稳妥对象，而除了调用`sayName()`方法外，没有别的方式可以访问其数据成员。即使有其他代码会给这个对象添加方法或数据成员，但也不可能有别的办法访问传入到构造函数中的原始数据。

# 继承

## 原型链

让原型对象等于另一个类型的实例，此时原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。

``` javascript
function SuperType() {
  this.property = true;
}

SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}

// 继承了SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
  return this.subproperty;
};

var instance = new SubType();
console.log(instance.getSuperValue()); // true
```

## 借用构造函数

原型中包含引用类型值存在问题，我们可以使用一种叫做借用构造函数（constructor stealing）的技术（有时候也叫做伪造对象或经典继承）：在子类型构造函数的内部调用超类型构造函数。

函数只不过是在特定环境中执行代码的对象，因此通过使用`apply()`和`call()`方法也可以在（将来）新创建的对象上执行构造函数，如：

``` javascript
function SuperType() {
  this.colors = ["red", "blue", "green"];
}

function SubType() {  
  // 继承了SuperType
  SuperType.call(this);
}

const instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors);    // ["red", "blue", "green", "black"]

var instance2 = new SubType();
console.log(instance2.colors);    // ["red", "blue", "green"]
```

## 组合继承

组合继承（combination inheritance）指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。

其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性：

``` javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {  
  // 继承属性
  SuperType.call(this, name);

  this.age = age;
}

// 继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType; 
SubType.prototype.sayAge = function() {
  alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors);      // ["red", "blue", "green", "black"]
instance1.sayName();          // "Nicholas"
instance1.sayAge();           // 29

var instance2 = new SubType("Greg", 27);
console.log(instance2.colors);      // ["red", "blue", "green"]
instance2.sayName();          // "Greg"
instance2.sayAge();           // 27
```

## 原型式继承

ES5新增了`Object.create()`方法，接收两个参数：

- 用作新对象原型的对象；
- （可选的）一个为新对象定义额外属性的对象。

在传入一个参数的情况下，`Object.create()`与`object()`方法的行为相同。

`Object.create()`方法的第二个参数与`Object.defineProperties()`方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。

``` javascript
const person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

const anotherPerson = Object.create(person, {
  name: {
    value: "Greg"
  }
});

console.log(anotherPerson.name); // "Greg"
```

## 寄生式继承

寄生式（parasitic）继承是与原型式继承紧密相关的一种思路，思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。以下代码示范了寄生式继承模式。

``` javascript
function createAnother(original) {
	// 通过调用函数创建一个新对象
  const clone = Object(original); 
  // 以某种方式来增强这个对象
  clone.sayHi = function() {
    alert("hi");
  };
  // 返回这个对象
  return clone;
}

const person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

const anotherPerson = createAnother(person);
anotherPerson.sayHi(); // "hi"
```

## 寄生组合式继承

寄生组合式继承是通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。基本思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。

本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。寄生组合式继承的基本模式如下所示。

``` javascript
function inheritPrototype(subType, superType){
    var prototype = Object(superType.prototype); // 创建对象
    prototype.constructor = subType;             // 增强对象
    subType.prototype = prototype;               // 指定对象
}
```
