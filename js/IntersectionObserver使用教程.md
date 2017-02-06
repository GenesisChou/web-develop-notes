当要观察某一个网页上某一个元素的可见性时，传统的做法是监听scroll事件，判断该元素与viewport左上角的距离来进行判断，但scroll事件容易造成性能问题。
而IntersectionObserver就能更加简单的解决这些问题。
创建IntersectionObserver对象
```js
var io=new IntersectionObserver(callback,option);
var element=document.querySelector('.example');
//观察
io.observe(element);
// 停止观察
io.unobserve(element);
// 关闭观察器
io.disconnect();
```
callback是一个回调函数，接收一个参数changes
```js
function callback(changes){
  console.log(changes);
}
```
changes是一个数组，根据io观察器所观察到的元素返回对应的变化
changes内每一个元素都是一个对象包含以下属性
```js
{
  time: 3893.92,
  rootBounds: ClientRect {
    bottom: 920,
    height: 1024,
    left: 0,
    right: 1024,
    top: 0,
    width: 920
  },
  boundingClientRect: ClientRect {
     // ...
  },
  intersectionRect: ClientRect {
    // ...
  },
  intersectionRatio: 0.54,
  target: element
}
```
每个元素的含义如下
```js
time：可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
target：被观察的目标元素，是一个 DOM 节点对象
rootBounds：根元素的矩形区域的信息，getBoundingClientRect()方法的返回值，如果没有根元素（即直接相对于视口滚动），则返回null
boundingClientRect：目标元素的矩形区域的信息
intersectionRect：目标元素与视口（或根元素）的交叉区域的信息
intersectionRatio：目标元素的可见比例，即intersectionRect占boundingClientRect的比例，完全可见时为1，完全不可见时小于等于0
```
IntersectionObserver构造函数的第二个参数是一个配置对象。它可以设置以下属性。
threshold属性决定了什么时候触发回调函数。它是一个数组，每个成员都是一个门槛值，默认为[0]，即交叉比例（intersectionRatio）达到0时触发回调函数。
```js
new IntersectionObserver(
  entries => {/* ... */}, 
  {
    threshold: [0, 0.25, 0.5, 0.75, 1]
  }
);
```
用户可以自定义这个数组。比如，[0, 0.25, 0.5, 0.75, 1]就表示当目标元素 0%、25%、50%、75%、100% 可见时，会触发回调函数。
