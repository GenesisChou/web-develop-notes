# dict
义同dictory，按索引查找
```
>>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
>>> d['Michael']
95
```

要避免key不存在的错误，有两种办法，一是通过in判断key是否存在：
```
>>> 'Thomas' in d
False
```
二是通过dict提供的get方法，如果key不存在，可以返回None，或者自己指定的value：
```
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1
```
要删除一个key，用pop(key)方法，对应的value也会从dict中删除：
```
>>> d.pop('Bob')
75
>>> d
{'Michael': 95, 'Tracy': 85}
```
和list比较，dict有以下几个特点：
- 查找和插入的速度极快，不会随着key的增加而变慢；
- 需要占用大量的内存，内存浪费多。

而list相反：
- 查找和插入的时间随着元素的增加而增加；
- 占用空间小，浪费内存很少。

# set

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。重复元素在set中自动被过滤。
通过add(key)方法可以添加元素到set中，可以重复添加，但不会有效果：
```
>>> s.add(4)
>>> s
{1, 2, 3, 4}
>>> s.add(4)
>>> s
{1, 2, 3, 4}
```
通过remove(key)方法可以删除元素：
```
>>> s.remove(4)
>>> s
{1, 2, 3}
```