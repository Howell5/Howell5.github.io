---
title: JavaScript之原型
date: 2016-08-01
---
> 在 JavaScript 中，每个对象都有一个指向它的原型（prototype）对象的内部链接。这个原型对象又有自己的原型，直到某个对象的原型为 null 为止（也就是不再有原型指向），组成这条链的最后一环。这种一级一级的链结构就称为原型链（prototype chain）。          --- MDN

<!--more-->
### 原型链
```
//创建构造函数
function Person(name){
    this.name = name
}

//每个构造函数JS引擎都会自动添加一个prototype属性，我们称之为原型，这是一个对象
//每个由构造函数创建的对象都会共享prototype上面的属性与方法
console.log(typeof Person.prototype) // 'object'


//我们为Person.prototype添加sayName方法
Person.prototype.sayName = function(){
    console.log(this.name)
}

//创建实例
var person1 = new Person('Messi')
var person2 = new Person('Suarez')

person1.sayName() // 'Messi'
person2.sayName() // 'Suarez'

person1.sayName === person2.sayName //true
```
借助这个例子，我们理清几个概念
 - 原型指针`__proto__`，向上游找原型对象

```
person1.__proto__ === Person.prototype; //true
Person.__proto__ === Object.prototype; //true
```
- 对象都有一个`constructor`属性，指向构造它的函数

```
console.log(person1.constructor = Person); //true
console.log(Person.constructor = Function); //true
var o = {};
console.log(o.constructor === Object); //true
```

![图1](http://upload-images.jianshu.io/upload_images/2575359-9e891bdd0eb3485f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图2](http://upload-images.jianshu.io/upload_images/2575359-28c82cb4080e2919.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注：Op是Object.prototype的简写。Fp是Function.prototype的简写