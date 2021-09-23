---
title: JS的6种继承方式
categories:
- 前端
tags:
- JS
- 手撕
cover: https://tva2.sinaimg.cn/large/00897VxHly8gpqmvab8sxj30pv0htwgo.jpg
---
本文参考自[js继承的6种方式](https://www.cnblogs.com/ranyonsue/p/11201730.html)，感谢。

想要继承，就必须先有父类（继承谁，提供谁的属性）
```js
//父类
function Person(name) {
	//给构造函数添加了参数
	this.name = name;
	this.sum = function() {
		alert(this.name);
	}
} 
//给构造函数添加了原型属性
Person.prototype.age = 10;
```
# 原型链继承
```js
function Per() {
	this.name = "ker";
}
//新实例的原型 = 父类的实例
Per.prototype = new Person();
var per1 = new Per();
console.log(per1.age); //10

//instanceof 判断元素是否在另一个元素的原型链上
//per1 继承了Person的属性，返回true
console.log(per1 instanceof Person); //true
```
- 重点：让新实例的原型等于父类的实例。
- 特点：实例可继承的属性有：
    1. 实例的构造函数的属性；
    2. 父类构造函数的属性；
    3. 父类原型的属性（新实例不会继承父类实例的属性）。
- 缺点：
    1. 新实例无法向父类构造函数传参；
	2. 继承单一；
	3. 所有新实例都会共享父类实例的属性（原型上的属性是共享的，一个实例修改了原型属性，另一个实例的原型属性也会被修改）。
# 构造函数继承
```js
function Con() {
	Person.call(this, "jer");
	this.age = 12;
}
var con1 = new Con();
console.log(con1.name); //"jer"
console.log(con1.age); //12
console.log(con1 instanceof Person); //false
```
- 重点：用`.call()`和`.apply()`将父类构造函数引入子类函数（在子类函数中做了父类函数的自执行（复制））。
- 特点：
    1. 只继承了父类构造函数的属性，没有继承父类原型的属性；
	2. 解决了原型链继承缺点1、2、3；
	3. 可以继承多个构造函数属性（call多个）；
	4. 在子实例中可向父实例传参。
- 缺点：
    1. 只能继承父类构造函数的属性；
	2. 无法实现构造函数的复用（每次用每次都要重新调用）；
	3. 每个新实例都有父类构造函数的副本，臃肿。
# 组合继承
```js
function SubType(name) {
	//借用构造函数模式
	Person.call(this, name);
}
//原型链继承
SubType.prototype = new Person();
var sub = new SubType("gar");
console.log(sub.name); //"gar" （继承了构造函数属性）
console.log(sub.age); //10 （继承了父类原型的属性）
```
- 重点：结合了两种模式的优点，传参和复用。
- 特点：
    1. 可以继承父类原型上的属性，可以传参，可复用；
	2. 每个新实例引入的构造函数属性是私有的。
- 缺点：
    1. 调用了两次父类构造函数（耗内存）；
	2. 子类的构造函数会代替原型上的那个父类构造函数。

# 原型式继承
```js
function content(obj) { //先封装一个函数容器，用来输出对象和承载继承的原型
	function F(){}
	F.prototype = obj; //继承了传入的参数
	return new F(); //返回函数对象
}
var sup = new Person(); //拿到父类的实例
var sup1 = content(sup);
console.log(sup1.age); //10 （继承了父类函数的属性）
```
- 重点：用一个函数包装一个对象，然后返回这个函数的调用，这个函数就变成了个可以随意增添属性的实例或对象。`Object.create()`就是这个原理。
- 特点：类似于复制一个对象，用函数来包装。
- 缺点：
    1. 所有实例都会继承原型上的属性；
	2. 无法实现复用（新实例属性都是后面添加的）。
# 寄生式继承
```js
function content(obj) {
	function F(){};
	F.prototype = obj; //继承了传入的参数
	return new F(); //返回函数对象
}
var sup = new Person();
//以上是原型式继承，给原型式继承再套个壳子传递参数
function subobject(obj) {
	var sub = content(obj);
	sub.name = "gar";
	return sub;
}
var sup2 = subobject(sup);
//这个函数经过声明之后就成了可增添属性的对象
console.log(typeof subobject); //function
console.log(typeof sup2); //object
console.log(sup2.name); //"gar" （返回了一个sub对象，继承了sub的属性）
```
- 重点：就是给原型式继承外面套了个壳子。
- 特点：没有创建自定义类型，因为只是套了个壳子返回对象，这个函数顺理成章就成了创建的新对象。
- 缺点：没用到原型，无法复用。
# 寄生组合式继承
- 寄生：在函数内返回对象然后调用。
- 组合：
    1. 函数的原型等于另一个实例；
	2. 在函数中用`apply`或者`call`引入另一个构造函数，可传参。
```js
function content(obj) {
	function F(){};
	F.prototype = obj;
	return new F();
}
//content就是F实例的另一种表示法
var con = content(Person.prototype);
//con实例（F实例）的原型继承了父类函数的原型
//上述更像是原型链继承，只不过只继承了原型属性

//组合
function Sub() {
	Person.call(this); //这个继承了父类构造函数的属性
} //解决了组合式两次调用构造函数属性的缺点
//重点
Sub.prototype = con; //继承了con实例
con.constructor = Sub; //一定要修复实例
var sub1 = new Sub();
//Sub的实例就继承了构造函数属性，父类实例，con的函数属性
console.log(sub1.age); //10
```
- 重点：修复了组合继承的问题。
# 手动实现ES5 继承
以下内容来自[ConardLi/awesome-coding-js/JavaScript](https://github.com/ConardLi/awesome-coding-js/tree/master/JavaScript)，感谢。

有下面两个类，下面实现 `Man` 继承 `People`：
```js
function People() {
	this.type = 'prople'
}
People.prototype.eat = function () {
	console.log('吃东西啦');
}
function Man(name) {
	this.name = name;
	this.color = 'black';
}
```
### 原型继承
将父类指向子类的原型。
```js
Man.prototype = new People();
```
缺点：原型是所有子类实例共享的，改变一个其他也会改变。
### 构造继承
在子类构造函数中调用父类构造函数
```js
function Man(name) {
	People.call(this);
}
```
缺点：不能继承父类原型，函数在构造函数中，每个子类实例不能共享函数，浪费内存。
### 组合继承
使用构造继承继承父类参数，使用原型继承继承父类函数。
```js
function Man(name) {
	People.call(this);
}
Man.prototype = People.prototype;
```
缺点：父类原型和子类原型是同一个对象，无法区分子类真正是由谁构造。
### 寄生组合继承
在组合继承的基础上，子类继承一个由父类原型生成的空对象。
```js
function Man(name) {
	People.call(this);
}
Man.prototype = Object.create(People.prototype, {
	constructor: {
	value: Man
	}
})
```
### inherits函数
```js
function inherits(ctor, superCtor) {
  ctor.super_ = superCtor;
  ctor.prototype = Object.create(superCtor.prototype, {
    constructor: {
      value: ctor,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
}

//使用
function Man() {
  People.call(this);
  //...
}
inherits(Man, People);
Man.prototype.fun = ...
```