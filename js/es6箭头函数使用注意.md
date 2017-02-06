# 箭头函数没有this
箭头函数没有 this,那下面的代码明显可以取到this啊：
```javascript
function foo(){
  this.a=1;
  let b=()=>console.log(this.a);
  b();
}
foot();//1
```
以上箭头函数中的this其实父级作用域中的this,即函数foo的this.箭头函数引用了父级的变量，构成了一个闭包。以上代码等价于：
```javascript
function foo(){
  return ()=>console.log(arguments[0]);
}
foo(1,2)(3,4);//1
```
上例中如果箭头函数有arguments,就应该输出的是3而不是1.
一个经常犯的错误就是使用箭头函数定义对象的方法，如：
```javascript
let a={
  foo:1,
  bar:()=>console.log(this.foo)
}
```
a.bar();
以上代码中，箭头函数中的this并不是指向a这个对象。对象a并不能构成一个作用域，所以再往上到达全局作用域，this就指向全局作用域。如果我们使用普通函数的定义方法，输出结果就符合预期，这是因为a.bar()函数执行时作用域绑定到了a对象。
```javascript
let a={
  foo:1,
  bar:function(){console.log(this.foo)}
}
```
a.bar();//1
另一个错误是在原型上使用箭头函数，如：
```javascript
function A(){
  this.foo=1;
}
A.prototype.bar=()=>console.log(this.foo);
let a=new A();
a.bar();//undefinded
```
同样，箭头函数中的this不是指向A,而是根据变量查找规则回溯到了全局作用域。同样，使用过普通函数就不存在问题。
通过以上说明，我们可以看出，箭头函数除了传入的参数之外，真的时什么都没有！如果你再箭头函数饮用了this,arguments或者参数之外的变量，那它们一定不是箭头函数本身包含的，而时从父级作用域继承的。

# 什么情况下该使用箭头函数
到这里，我们可以发现箭头函数并不是万金油，稍不留神就会踩坑。
至于什么情况该使用箭头函数，《You Don't Know JS》给出了一个决策图：

![image](https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/es6%20%26%20beyond/fig1.png)

以上决策图看起来有点复杂，我认为有三点比较重要：
 - 箭头函数适合于无发杂逻辑或者无副作用的纯函数场景下，例如用map,reduce,filter的回调函数定义中；
 - 不要再最外层定义箭头函数，因为再函数内部操作this会容易污染全局作用域。最起码在箭头函数外部包一层普通函数，将this控制在可见的范围内；
 - 如开头所述,箭头函数最吸引人的地方是简洁。在有多层嵌套的情况下，箭头函数的简洁性并没有很大的提升，反而影响了函数的作用范围的识别度，这种情况不建议使用箭头函数。

