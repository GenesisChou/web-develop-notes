 ```javascript
 //获取可视区域高度,宽度
function getViewPortWidth(){
    return window.innerWidth||Math.min(document.documentElement.clientWidth,document.body.clientWidth);
}
function getViewPortHeight(){
    return window.innerHeight||Math.min(document.documentElement.clientHeight,document.body.clientHeight);
}
//获取页面内容总高度
function getScrollHeight(){
    return Math.max(document.body.scrollHeight, document.documentElement.scrollHeight);
}
 ```
**clientHeight**
>  Returns the height of the visible area for an object, in pixels. The value contains the height with the padding, but it does not include the scrollBar, border, and the margin.

**offsetHeight**
> Returns the height of the visible area for an object, in pixels. The value contains the height with the padding, scrollBar, and the border, but does not include the margin.

![image](https://i.stack.imgur.com/zWca7.png)