# Vue

领导让20几天搞一个前端出来，对于没学过前端的我来说，太难了~，没办法，这两天先后学习了**Html**、**Css**和**Js**的基础，并做了如下笔记。

## 一、标签

**v-once**

```vue
<span v-once>{{msg}}</span>
```

目的：只绑定一次，假设后续数据更新，

**V-html**

目的：输出HTML实现的效果，而非HTML代码。

*** v-bind**

```vue
<a v-bind:href="url">...</a>
```

目的：**Mustache语法**不能用在HTML特性上，如果特性动态绑定需要使用`v-bind`

缩写：

```vue
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

*** v-on**

```vue
<a v-on:click="doSomething">...</a>
```

目的：监听Dom事件

缩写：

```vue
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

事件修饰符：

- .stop：阻止单机事件继续传播
- .prevent：提交事件不再重载页面
- .capture：事件捕获，先自身处理，再交由内部元素处理
- .self：当前元素自身触发时处理
- .once
- .passive

```vue
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```

按键修饰符：

...

系统修饰符：

...

.exact修饰符：

鼠标按钮修饰符：

**v-if**

```vue
<p v-if="seen"></p>
```

目的：插入或删除元素

特点：惰性即懒加载，直到第一次使用。

**v-show**

同`v-if`，但与`v-if`不同的地方在于`v-show`总是会被加载，不会懒加载。

**v-for**

```vue
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

目的：对列表进行渲染，支持对数组和对象所有属性的访问。

建议：建议绑定key，防止数据错乱。

**key**

目的：管理重复元素，有的时候可能会用到缓存

**v-model**

目的：在表单`<input>`、`textarea`、`select`对数据进行双向绑定

修饰符：

- .lazy：每次`input`事件触发后输入框的值与数据进行同步。
- .number：将输入值转为数据类型
- .trim：自动过滤首尾的空白字符

组件使用：

- [ ] TODO

**插槽**：

像一个组件传递内容。

`:is`：

可以用来完成组件的动态切换

## 二、语法

**文本绑定**：称为**Mustache语法**

```vue
<span>{{msg}}</span>
```

**绑定Js表达式**

每个绑定智能包含单个表达式。

## 三、组件

#### # Prop大小写

#### # Prop类型

数组或者指定类型

#### # 传递静态或者动态的Prop

#### # 数据流

所有的prop都使得其父子prop之间形成了一个单向下行绑定

## 小技巧

#### # 如何在内部组件中调用外部的CSS?

在Vue组件内部的`style`标签内部使用`import *`导入，例如：

```javascript
@import "assets/css/tab.css";
```

#### # Object.freeze()

`Object.freeze()`会使数据的自动更新失效！

#### # 如何在组件中声明方法？

在组件的`methods`属性。