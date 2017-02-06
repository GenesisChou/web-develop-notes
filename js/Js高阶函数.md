# 事件委托

# 作用域安全的构造函数

# 惰性载入函数

# 防篡改对象

## 1.不可扩展对象
在调用了Object.preventExtensions()方法后，就不能给person对象添加新属性和方法了。在非严格模式下，给对象添加新成员会导致静默失败。而在严格模式下，尝试给不可扩展的对象添加新成员会导致抛出错误。
虽然不能给对象添加新成员，但已有的成员则丝毫不受影响。你仍然还可以修改和删除已有的成员。另外，使用Object.isExtensible（）方法还可以确定对象是否可以扩展。
## 2.密封的对象
密封对象不可扩展，而且已有成员的[Configurable]特性将被设置为false。这就意味着不能删除属性和方法，因为不能使用Object.defineProperty()把数据属性修改为访问器属性，或者相反。属性值是可以修改的。
要密封对象，可以使用Object.seal()方法。
使用Object.isSealed()方法可以确定对象是否被密封了。因为被密封的对象不可扩展，所以用Object.isExtensible()检测密封的对象也会返回false。
## 3.冻结的对象
嘴严格的防篡改级别是冻结对象（frozen object）。
冻结的对象既不可扩展，又是密封的，而且对象数据属性的[Writable]特性会被设置为false.如果定义[Set]函数，访问器属性仍然是可写的。ECMAScript5定义的Object.freeze()方法可以用来冻结对象。
Object.isFrozen()方法用于检测冻结对象。因为冻结对象既是密封的又是不可扩展的，所以用Object.isExtensible()和Object.isSealed()检测冻结对象将分别返回false和true。
# 高级定时器
定时器对队列的工作方式是，当特定时间过去后将代码插入。注意，给队列添加代码并不意味着对它立刻执行，而只能表示它会尽快执行。设定一个150ms后执行的定时器不代表到了150ms代码就立刻执行，它表示代码会在150ms后呗加入到队列中。如果在这个时间点上，队列中没有其他东西，那么这段代码就会被执行，表面上看上去好像代码就在精确指定的时间点上执行了。其他情况下，代码可能明显地等待更长时间才执行。
## 1.重复的定时器
当使用setInterval()时，仅当没有该定时器的任何其他代码实例时，才将定时器代码添加到队列中。这确保了定时器代码加入到队列中的最小时间间隔为指定间隔。
这种重复定时器的规则有2点问题：（1）某些间隔会被跳过；（2）多个定时器的代码执行之间的间隔可能会比预期的小。假设，某个onclick事件处理程序使用setInterbal()设置了一个200ms间隔的重复定时器。如果事件处理程序花了300ms多一点的时间完成，同时定时器代码也花了差不多的时间，就会跳过一个间隔同时运行着一个定时器代码。为了避免setInterval()的重复定时器的这2个缺点，你可以用如下模式使用链式setTimeout()调用。
```javascript
setTimeout(function(){
    setTimeout(arguments.callee,interval);
},interval);
```
## 2.Yielding Processes
长时间运行的循环通常遵循以下模式：
```javascript
for(var i=0,len=data.length;i<len;i++){
    process(data[i]);
}
```
在处理不是必须同步完成，顺序完成的2个前提下。可以使用定时器分割这个循环。这是一种叫做数组分块（array chunking）的技术，小块小块地处理数组，通常每次一小块。基本的思路是为要处理的项目创建一个队列，然后使用定时器取出下一个要处理的项目进行处理，接着再设置另一个定时器。
```javascript
setTimeout(function(){
    //取出下一个条目并处理
    var item=array.shift();
    process(item);
    if(array.length>0){
        setTimeout(arguments.callee,100);
    }
})
```
# 函数节流
