#ref
类型： string
ref 被用来给元素或子组件注册引用信息。引用信息会根据父组件的 $refs 对象进行注册。如果在普通的DOM元素上使用，引用信息就是元素; 如果用在子组件上，引用信息就是组件实例:
```html
<!-- vm.$refs.p will be the DOM node -->
<p ref="p">hello</p>
<!-- vm.$refs.child will be the child comp instance -->
<child-comp ref="child"></child-comp>
```
当 v-for 用于元素或组件的时候，引用信息将是包含DOM节点或组件实例数组。
关于ref注册时间的重要说明: 因为ref本身是作为渲染结果被创建的，在初始渲染的时候你不能访问它们 - 它们还不存在！$refs 也不是响应式的，因此你不应该试图用它在模版中做数据绑定。

#使用自定义事件的表单输入组件

自定义事件也可以用来创建自定义的表单输入组件，使用 v-model 来进行数据双向绑定。牢记：
```html
<input v-model="something">
```
仅仅是一个语法糖：
```html
<input v-bind:value="something" v-on:input="something = $event.target.value">
```
所以在组件中使用时，它相当于下面的简写：
```html
<custom-input v-bind:value="something" v-on:input="something = arguments[0]"></custom-input>
```
所以要让组件的 v-model 生效，它必须：
- 接受一个 value 属性
- 在有新的 value 时触发 input 事件