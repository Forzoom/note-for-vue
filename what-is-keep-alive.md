### 什么是\<keep-alive\>

`<keep-alive>`是一个抽象组件，为子元素提供缓存服务。至于在什么时候会用到它，可以根据官方文档中的说法，在你需要缓存组件内容的时候来使用，比如说，从一个路由组件跳转到另外一个路由组件，当跳转回去时，希望保持原本组件中的内容，目前看来主要是保持data数据。

官方文档中也说明，只有一个子元素会被处理，从源码中看来也是这样，不过为什么这样做，暂时不是很清楚，感觉应该也是可以实现多个组件的缓存。

#### 如何实现缓存

一般写组件时，为了实现`父元素`向`子元素`内传递HTML结构，会使用`slot`功能。但是因为`Vue`要求任何组件的根节点应该有且只有一个，所以一般会在组件最外层包裹一个`<div>`元素，再向其中写入`<slot>`。

例如：
```html
<!-- 子组件 -->
<div class="my-component">
    <slot></slot>
</div>
```

```html
<!-- 父组件 -->
<my-component>
    <div>test</div>
</my-component>
```

```html
<!-- 渲染结果 -->
<div class="my-component">
    <div>test</div>
</div>
```

上面例子中可以看到，除了我们写入的`<div>test</div>`之外，还多了一层`<my-component></my-component>`。
例子中代码是`SFC`样式的，也就是`.vue`文件中的模板`<template>`部分中的内容，除了`<template>`之外，为了实现更灵活的效果，`Vue`还提供了`render函数`这种方法来实现更加灵活的渲染方式。`<keep-alive>`即通过这种方式来实现自身的功能。

```javascript
export default {
    render(h) {
        return this.$slots.default;
    },
}
```

在上面例子中，`<keep-alive>`是一个抽象组件，本身不会渲染一个DOM元素，所以最终渲染的结果如下。相较于上面使用模板`<template>`的方式少了一层`<div>`元素。

```html
<div>test</div>
```

`<keep-alive>`即使用该方式进行渲染，并且为其缓存功能的实现提供了可能性。

```javascript
// <keep-alive>的render函数中的代码片段
if (cache[key]) {
    vnode.componentInstance = cache[key].componentInstance
    // make current key freshest
    remove(keys, key)
    keys.push(key)
} else {
    cache[key] = vnode
    keys.push(key)
    // prune oldest entry
    if (this.max && keys.length > parseInt(this.max)) {
        pruneCacheEntry(cache, keys[0], keys, this._vnode)
    }
}
```

具体缓存逻辑在render中处理，`<keep-alive>`在内部创建了一个`cache: object`和`keys: string[]`。其中`cache`是一个映射结构，用来存储需要缓存的组件实例，每次`render`函数执行时，都会检查`cache`中是否有可以复用的组件，当存在时，`render`就可以直接返回该组件。

#### 如何使用\<keep-alive\>

按照文档上的说法，在使用`动态组件`时，可以使用`<keep-alive>`来缓存组件实例。但是偶尔在代码中会看到一些错误的示范，例如

```html
<div v-if="enable">
    <keep-alive>
        <component :is="type"></component>
    </keep-alive>
</div>
<div v-else>
    <div>test</div>
</div>
```

当`enable`发生变化从true变为false，之后再变为true时，`<keep-alive>`并不能实现对于`<component>`组件的缓存效果。
