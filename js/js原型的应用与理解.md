# js原型
我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的那个对象实例的原型对象。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面的例子所示。

无论什么时候，只要创建了一个新函数，就会根据一组特点的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含了一个指向prototype属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor指向Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

尽管可以随时为原型添加属性和方法，并且修改能够立即在所有对象实例中反映出来，但如果是重写整个原型对象，那么情况就不一样了。我们知道，调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针(实例的_proto_属性)，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。请记住：实例中的指针仅仅指向原型，而不指向构造函数。看下面的例子。
```javascript
function Person(){
    
}
var friend=new Person();
Person.prototype={
    constructor:Person,
    name:'Nicholas',
    age:29,
    job:'Software Engineer',
    sayName:function(){
        alert(this.name);
    } 
};
friend.sayName(); //error
```
重写原型对象切断了现有原型与任何之前已经存在的对象实例之间的联系；它们引用的仍然是最初的原型。

# new的作用
- 创建一个新对象;
- 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
- 执行构造函数中的代码（为这个新对象添加属性）；
- 返回新对象。
```
function Person(name,age,job){
    this.name=name;
    this.age=age;
    this.job=job;
    this.friends=['Tom','Jack','Elly'];
}
function New(constructor){
    var temp={};
    var args=Array.prototype.slice.call(arguments,1);
    temp._proto_=constructor.prototype;
    constructor.apply(temp,args);
    return temp;
}
```