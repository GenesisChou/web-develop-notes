# 注意作用域
## 1.避免全局查找
## 2.避免with语句
# 选择正确的方法
## 1.避免不必要的属性查找
一般来讲，只要能减少算法的复杂度，就要尽可能减少。尽可能多地使用局部变量将属性查找替换为值查找。进一步讲，如果既可以用数字化的数组位置进行访问，也可以使用命名属性，那么使用数字位置。
## 2.优化循环
- 减值迭代
- 简化终止条件
- 简化循环体
- 使用后测试循环（do-while）

## 3.展开循环
如果循环中的迭代次数不能事先确定，可以考虑使用一种叫做Duff装置的技术。这个技术是以其创建者Tom Duff命名的，他最早在C语言中使用这项技术。展开循环可以提升大数据集的处理速度。
Duff装置的基本概念是通过计算迭代的次数是否为8的倍数将一个循环展开为一系列语句。
- Jeff Greenberg的Duff
- Andrew B.King所著的Speed Up Your Site中提出了一个更快的Duff装置技术，将do-while循环分成2个单独的循环。
针对大数据集使用展开循环可以节省很多时间，小数据集不值得使用。

## 4.避免双重解释
尽量避免使用eval()
## 5.性能的其他注意事项
- **原生方法较快**——原生方法是用c/c++之类的编译型语言写出来的。
- **Switch语句较快**——如果有一系列复杂的if-else语句，可以转换成单个switch语句则可以得到更快的代码。
- **位运算符较快**——诸如取模，逻辑与和逻辑或都可以考虑用位运算来替换。
 
## 6.最小化语句数
- 多个变量声明
- 插入迭代值
- 使用数组和对象字面量

## 7.优化DOM交互
- 最小化现场更新,当要在循环体中添加列表项时，利用document.createDocumentFragment()创建文档碎片来构建DOM结构，接着将其添加到List元素中。这个方式避免了现场更新和页面闪烁问题。
- 使用innerHTML，对于大的DOM更改，使用innerHTML要比使用标准DOM方法创建同样的DOM结构快得多。构建好一个字符串后一次性调用innerHTML,要比调用innerHTML多次快得多。
- 使用事件代理，列表事件绑定在list上而不是每一个列表项上
- 注意HTMLCollection,发生以下情况时会返回HTMLCollection对象 
    - 进行了对 getElementsByTagName()的调用；
    - 获取了元素的childNodes属性；
    - 获取了元素的attributes属性；
    - 访问了特殊的集合，如document.forms,document.images等。