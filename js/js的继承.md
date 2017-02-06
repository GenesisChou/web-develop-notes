# 原型链
每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。那么，加入我们让原型对象等于另一个类型的shi li实例，结果会怎么样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，相应de地，另一个原型中也包含着一个指向另一个构造函数的指针。加入另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型链条。这就是所谓原型链的基本概念。
实现原型链有一种基本模式，其代码大致如下。
```javascript
function SuperType(){
    this.property=true;
}
SuperType.prototype.getSuperValue=function(){
    return this.property;
}
function SubType(){
    this.subproperty=false;
}
//继承了SuperType
SubType.prototype=new SuperType();
SubType.prototype.getValue()=function(){
    return this.subproperty;
};
var instance=new SubType();
alert(instance.getSuperValue());//true
```
我们知道，所有引用类型默认都继承了Object,而这个继承也是通过原型链实现的。要记住，所有函数的默认原型都是Object的实例，因此默认原型都会包含一个内部指针，指向Object.prototype。这也正是所有自定义类型都会继承toString()、valueOf()等默认方法的根本原因。所以，我们说上面例子展示的原型链中还应该包括另外一个继承层次。
一句话，SubType继承了SuperType，而SuperType继承了Object。当调用instance.toString()时，实际上调用的是保存在Object.prototype中的哪个方法。
实际上，不是SubType的原型的constructor属性被重写了，而是SubType的原型指向了另一个对象——SubType的原型，而这个原型对象的constructor属性指向的是SuperType.

缺陷：1.因为子类的prototype指向超类实例，所有子类实例都会共享超类实例的实例属性。2.在创建子实例时，不能向超类的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类的构造函数传递参数。有鉴于此，再加上前面刚刚讨论锅的由于原型中包含饮用类型值所带来的问题。实践中很少会单独使用原型链。
# 借用构造函数
```javascript
function SuperType(){
    this.colors=['red','blue','green'];
}
function SubType(){
    SuperType.call(this);
}
var instance1=new SubType();
instance1.colors.push('black'); //red,blue,green,black
var instance2=new SubType();
instance2.colors //red,blue,green
```
缺陷：方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类的原型中定义的方法，对子类而言也是不可见的，结果所有类型都只能使用构造函数模式。考虑到这些问题，借用构造函数的技术也很少单独使用的。
# 组合继承
组合继承（combination inheritance),有时候也叫做经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。下面来砍一个例子。
```javascript
function SuperType(name){
    this.name=name;
    this.colors=['red','blue','green'];
}
SuperType.prototype.sayName=function(){
    alert(this.name);
}
function SubType(name,age){
    SuperType.call(this,name);
    this.age=age;
}
SubType.prototype=new SuperType();
SubType.prototype.sayAge=function(){
    alert(this.age);
};
var instance1=new SubType('Nicholas',29);
instance1.colors.push('black');
alert(instance1.colors); //red,blue,green,black
instance1.sayName();//Nicholas
instance1.sayAge();//29
var instance2=new SubType('Greg',27);
alert(instance2.colors);//red,blue,green
instance2.sayName();//Greg
instance2.sayAge();//27
```
在这个例子中,SuperType构造函数定义了两个属性：name和colors。SuperType的原型定义了一个方法sayName().SubType构造函数在调用SuperType构造函数时传入了name参数，紧接着又定义了它自己的属性age。然后，将要SuperType的实例赋值给SubType的原型，然后又在该新原型上定义了方法sayAge()。这样以来，就可以让两个不同的SubType实例既分别拥有自己属性——包括colors属性，又可以使用相同的方法了。
组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为JavaScript中最常用的继承模式。而且，instanceof和isPrototypeOf()也能够用于识别基于组合继承创建的对象。