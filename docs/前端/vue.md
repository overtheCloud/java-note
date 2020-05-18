### Vue.js  是什么

Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

> 官网地址 https://cn.vuejs.org/v2/guide/#

##### 安装

开发版本

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

生产版本

```html
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

##### Hello World

```html
<!-- html -->
<div id="app">
  {{ message }}
</div>
<!-- js -->
<script type="text/javascript">
var app = new Vue({
  // 挂载点
  el: '#app',
  // 数据  
  data: {
    message: 'Hello Vue!'
  },
  // 方法  
  method: {
      
  }  
})
</script>
```

##### 基础-el挂载点

el 属性指定了 vue 的作用范围，支持 css 选择器，不适用于 <html> <body> 等标签，常用于 <div> 标签和 id 选择器。

##### 基础-数据绑定

数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

```html
<span>Message: {{ msg }}</span>
```

##### 基础-标签

- v-text 设置标签的文本值

- v-html 设置标签的 innerHtml，会解析 html 标签

  ```html
  <div v-text="message"></div> <!-- 显示 <a href="">Hello Vue!</a> -->
  <div v-html="message"></div> <!-- 显示 Hello Vue! -->
  
  <script type="text/javascript">
  var app = new Vue({
    el: '#app',
    data: {
      message: '<a href="">Hello Vue!</a>'
    }
  })
  </script>
  ```

- v-on 绑定事件

```
v-on:click="doSomething" 等同于
@click="doSomething" 
```

- v-show 是否显示，ture-显示，false-隐藏，切换标签 的 display 属性
- v-if 是否创建 dom 元素，true-创建，false-不创建
- v-bind 绑定属性值
- v-for 循环
- v-model 双向绑定 

对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

##### 网络请求 axios

```html
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
```

