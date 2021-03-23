---
title: javascript中的继承
date: 2018-04-06 20:55:18
desc:
tags: javascript
---

# 1.原型链
```js
function SuperType(){
    this.property = true;
}
SuperType.prototype.getSuperValue = function(){
    return this.property;
};

function SubType(){
    this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function (){
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());”

摘录来自: Nicholas C.Zakas. “JavaScript高级程序设计（第3版）。” iBooks. 

```
> 这个方式实现继承最大的问题就是在包含引用类型的原型会被所有实例共享，而导致子类型的实例对这个引用类型的修改可以被其他所有子类型看见。例如：

<!-- more -->
```js
function SuperType(){
    this.colors = ["red", "blue", "green"];
}

function SubType(){            
}

//继承了SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors);        //"red,blue,green,black"

var instance2 = new SubType();
console.log(instance2.colors);        //"red,blue,green,black”

摘录来自: Nicholas C.Zakas. “JavaScript高级程序设计（第3版）。” iBooks. 
```
子类型实例`instance2`修改应用类型属性`color`,导致其他子类型的实例的`color`属性也被修改了。(因为`color`是原型上的一个属性是被所有实例共享的)



# 2.借用构造函数
为了解决引用类型属性在原型上所导致的问题。我们可以在子类型的构造函数中使用父类型的构造函数。
```js
function SuperType(){
    this.colors = ["red", "blue", "green"];
}

function SubType(){ 
  // 借用父类型的构造函数
  SuperType.call(this);          
}

var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors);        //"red,blue,green,black"

var instance2 = new SubType();
console.log(instance2.colors);        //"red,blue,green"
```
> 借用构造函数虽然解决了原型链继承所带来的问题，但是由于方法都在构造函数中定义导致无法复用函数。而且父类型的方法对子类型也是不可见的。

# 3.组合继承
既然原型链继承和借用构造函数继承都有缺陷，那我们把它们结合起来互补，不就解决了问题了吗？
```js
// 父类型
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    alert(this.name);
};

// 子类型
function SubType(name, age){  
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}
// 指定原型
SubType.prototype = new SuperType();

SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors);      //"red,blue,green,black"
instance1.sayName();          //"Nicholas";
instance1.sayAge();           //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors);      //"red,blue,green"
instance2.sayName();          //"Greg";
instance2.sayAge();           //27”

摘录来自: Nicholas C.Zakas. “JavaScript高级程序设计（第3版）。” iBooks. 

```
> 这样我们既解决原型链的引用类型被共享的问题，也解决借用构造函数导致的无法复用函数和访问父类型的方法的问题。