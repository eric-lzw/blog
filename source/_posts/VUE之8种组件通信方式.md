---
title: VUE之8种组件通信方式
date: 2019-03-19 12:45:29
type: "js-base"
tag: vue
description:
keywords:
top_img:
mathjax:
katex:
aside:
---

### Props传递数据

```javascript
components
   ├── Grandson1.vue // 孙子1
   ├── Grandson2.vue // 孙子2
   ├── Parent.vue   // 父亲
   ├── Son1.vue     // 儿子1
   └── Son2.vue     // 儿子2
```

在父组件中使用儿子组件

```vue
<template>
 <div>
  父组件:{{num}}
  <Son1 :num="num"></Son1>
 </div>
</template>
<script>
import Son1 from "./Son1";
export default {
 components: {
  Son1
 },
 data() {
  return { num: 100 };
 }
};
</script>
```

子组件：
```vue
<template>
 <div>
  子组件1: {{num}}
 </div>
</template>
<script>
export default {
 props: {
  num: {
   type: Number
  }
 }
};
</script>
```

### $emit使用

子组件触发父组件方法,通过回调的方式将修改的内容传递给父组件

```vue
<template>
 <div>
  父组件:{{num}}
  <Son1 :num="num" @input="change"></Son1>
 </div>
</template>
<script>
import Son1 from "./Son1";
export default {
 methods: {
  change(num) {
   this.num = num;
  }
 },
 components: {
  Son1
 },
 data() {
  return { num: 100 };
 }
};
</script>
```

子组件触发绑定自己身上的方法

```vue
<template>
 <div>
  子组件1: {{num}}
  <button @click="$emit('input',200)">更改</button>
 </div>
</template>
<script>
export default {
 props: {
  num: {
   type: Number
  }
 }
};
</script>
```

> 这里的主要目的就是同步父子组件的数据,->语法糖的写法

#### .sync

```html
<Son1 :num.sync="num"></Son1>
<!-- 触发的事件名 update:(绑定.sync属性的名字) -->
<button @click="$emit('update:num',200)">更改</button>
```

#### v-model

```html
<Son1 v-model="num"></Son1>
<template>
 <div>
  子组件1: {{value}} // 触发的事件只能是input
  <button @click="$emit('input',200)">更改</button>
 </div>
</template>
<script>
export default {
 props: {
  value: { // 接收到的属性名只能叫value
   type: Number
  }
 }
};
</script>
```

### $parent、$children

继续将属性传递

```html
<Grandson1 :value="value"></Grandson1>
<template>
 <div>
  孙子:{{value}}
  <!-- 调用父组件的input事件 -->
  <button @click="$parent.$emit('input',200)">更改</button>
 </div>
</template>
<script>
export default {
 props: {
  value: {
   type: Number
  }
 }
};
</script>
```
> 如果层级很深那么就会出现$parent.$parent.....我们可以封装一个$dispatch方法向上进行派发


vue1.0里有$dispatch和$broadcast，vue2.0移除了此方法，我们来利用$emit分别实现下

#### $dispatch

```javascript
Vue.prototype.$dispatch = function $dispatch(eventName, data) {
  let parent = this.$parent;
  while (parent) {
    parent.$emit(eventName, data);
    parent = parent.$parent;
  }
};
```

既然能向上派发那同样可以向下进行派发

#### $broadcast

```javascript
Vue.prototype.$broadcast = function $broadcast(eventName, data) {
  const broadcast = function () {
    this.$children.forEach((child) => {
      child.$emit(eventName, data);
      if (child.$children) {
        $broadcast.call(child, eventName, data);
      }
    });
  };
  broadcast.call(this, eventName, data);
};
```

### $attrs、$listeners

#### attrs

批量向下传入属性

```javascript
<Son2 name="张三" age="10"></Son2>

<!-- 可以在son2组件中使用$attrs属性,可以将属性继续向下传递 -->
<div>
  儿子2: {{$attrs.name}}
  <Grandson2 v-bind="$attrs"></Grandson2>
</div>


<template>
 <div>孙子:{{$attrs}}</div>
</template>
```

#### $listeners

批量向下传入方法

```html
<Son2 name="张三" age="10" @click="()=>{this.num = 500}"></Son2>
<!-- 可以在son2组件中使用listeners属性,可以将方法继续向下传递 -->
<Grandson2 v-bind="$attrs" v-on="$listeners"></Grandson2>

<button @click="$listeners.click()">更改</button>
```

### Provide & Inject

#### Provide

在父级中注入数据

```javascript
provide() {
  return { parentMsg: "父亲" };
}
```

#### Inject

在任意子组件中可以注入父级数据

```javascript
inject: ["parentMsg"] // 会将数据挂载在当前实例上
```

### Ref

```html
<Grandson2 v-bind="$attrs" v-on="$listeners" ref="grand2"></Grandson2>
mounted() { // 获取组件定义的属性
  console.log(this.$refs.grand2.name);
}
```

### EventBus

用于跨组件通知(不复杂的项目可以使用这种方式)

```javascript
Vue.prototype.$bus = new Vue();
```

Son2组件和Grandson1相互通信

```javascript
mounted() {
  this.$bus.$on("my", data => {
   console.log(data);
  });
 },
 ```

 ```javascript
 mounted() {
  this.$nextTick(() => {
   this.$bus.$emit("my", "我是Grandson1");
  });
 },
 ```

 ### Vuex通信

 ![img.png](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfd4lhzpu0j30jh0fbwef.jpg)

 