# list
```
classmates = ['Michael', 'Bob', 'Tracy']
```
**len**:
获取list元素个数
```
>>>len(classmates)
3
```
如果要取最后一个元素，除了计算索引位置外，还可以用-1做索引，直接获取最后一个元素
```
>>> classmates[-1]
'Tracy'
```
以此类推，可以获取倒数第2个、倒数第3个：
**append**:
可以往list中追加元素到末尾：
```
>>> classmates.append('Adam')
>>> classmates
['Michael', 'Bob', 'Tracy', 'Adam']
```
**insert**:
把元素插入到指定的位置，比如索引号为1的位置
```
>>> classmates.insert(1, 'Jack')
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy', 'Adam']
```
***pop**:
删除list末尾的元素
添加参数后可以删除指定索引位置的元素
```
>>> classmates.pop()
'Adam'
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy']
>>> classmates.pop(1)
'Jack'
>>> classmates
['Michael', 'Bob', 'Tracy']
```
# turple
另一种有序列表叫元组：tuple。tuple和list非常类似，但是tuple一旦初始化就不能修改，比如同样是列出同学的名字：
```
>>> classmates = ('Michael', 'Bob', 'Tracy')
```
现在，classmates这个tuple不能变了，它也没有append()，insert()这样的方法。其他获取元素的方法和list是一样的，你可以正常地使用classmates[0]，classmates[-1]，但不能赋值成另外的元素。
要定义一个只有1个元素的tuple，如果你这么定义：
```
>>> t = (1)
>>> t
1
```
定义的不是tuple，是1这个数！这是因为括号()既可以表示tuple，又可以表示数学公式中的小括号，这就产生了歧义，因此，Python规定，这种情况下，按小括号进行计算，计算结果自然是1。

所以，只有1个元素的tuple定义时必须加一个逗号,，来消除歧义：
```
>>> t = (1,)
>>> t
(1,)
```

