# Vue.js学习笔记

### jQuery与Vue.js对比
1. 做的事情

    1. jQuery

        （旧时代到现在）相对于原生JS，更好的API，兼容性极好的DOM、AJAX操作。
    1. Vue.js

        实现MVVM的数据双向绑定，实现自己的组件系统。
2. 优劣势对比

    1. jQuery

        1. 兼容性好，兼容基本所有当今浏览器；出现早，学习、使用成本低。
        2. 程序员关注DOM，频繁操作DOM；代码量较多且不好维护，当页面需求变化之后代码改动难度大。
    2. Vue.js

        1. 程序员关注数据，DOM的操作交给框架；代码清晰，利于维护；有自己的组件系统。
        2. 不兼容旧版本浏览器；需要一些学习成本。

### [Vue官方教材](https://cn.vuejs.org/v2/guide/)
1. 双向绑定，响应式。

    编写的代码不需要关注底层逻辑，只需要关注数据双向绑定。

    2. Vue实例会代理`data`、`methods`的属性，也有以`$`开头的实例属性（如`$el`、`$data`、`$watch`）。

        只有被代理的`data`是响应的，也就是说值的任何改变都是触发视图的重新渲染。

        1. 导致试图更新：

            1. 数组的变异方法：`push`、`pop`、`shift`、`UNshift`、`splice`、`sort`、`reverse`。
            2. 数组赋值。
            3. `Vue.set(数组, 索引, 新值)`插值。
            4. `数组.splice(新长度)`
        2. 无法检测数组变动：

            1. 数组索引赋值。
            2. 修改数组长度。
    Vue 实现了一些智能启发式方法来最大化 DOM 元素重用。
3. 模板插值，适用表达式，语句、流控制不会生效。只能访问部分全局变量。会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。

    对于所有的数据绑定， Vue.js 都提供了完全的 JavaScript 表达式支持。

    e.g.
    ```html
    {{ number + 1 }}
    {{ ok ? 'YES' : 'NO' }}
    {{ message.split('').reverse().join('') }}
    <div v-bind:id="'list-' + id"></div>
    ```
4. 慎用箭头函数，`this`的指向无法按预期指向Vue实例。
5. 生命周期。
4. 指令

    >指令（Directives）是带有 v- 前缀的特殊属性。

    DOM的特殊属性。


    1. `:`参数

        指令后的参数用`:`指明。
    1. `v-bind`绑定DOM属性与Vue实例属性

        `v-bind:`、`:`

        1. 绑定`class`

            1. `v-bind:class="{class名: Vue属性[, class名: Vue属性]}"`
            2. `v-bind:class="Vue属性"`
            3. `v-bind:class="[Vue属性[, Vue属性]]"`
            4. `v-bind:class="[Vue属性 ? Vue属性 : ''[, Vue属性]]"`
            5. `v-bind:class="[{class名: Vue属性}[, Vue属性]]"`
        2. 绑定`style`

            >自动添加样式前缀。

            1. `v-bind:style="{css属性: Vue属性[, css属性: Vue属性]}"`
            2. `v-bind:style="Vue属性"`
            3. `v-bind:style="[Vue属性[, Vue属性]]"`
            4. 多重值
    2. `v-if`、`v-else`、`v-else-if`

        `key`属性使切换的DOM不公用。

        `<template>`渲染结果不包括。
    3. `v-for="(value, key, index) in/of Vue属性/整数"`

        >比`v-if`优先级更高。

        `v-bind:key="唯一的值"`使切换的DOM不复用。

        在组件中使用`v-for`时，必须添加`v-bind:key='值'`。

        `<template>`渲染结果不包括。
    3. `v-on`事件处理

        `v-on:`、`@`

        1. `$event`事件
    4. `v-model`表单
    5. `v-once`一次性插值，不再响应式
    6. `v-html`输入真正HTML
    7. 修饰符：`v-on`、`v-bind`、`v-module`
    8. 过滤器：`|`
    9. `v-show`

        仅切换`display`属性。
    10. `<slot>`

        用于父向子组件插入内容。
    11. 渲染结果不包含`<template>`：`v-for`、`v-if`
5. Vue实例的属性：

    `new Vue(对象)`

    1. `el`字符串，选择器
    2. `data`对象，数据
    3. 生命周期钩子

        1. `created`方法
        2. `updated`方法
        3. `destroyed`方法
        4. 等
    4. `methods`对象
    5. `computed`对象，自动依赖而计算

        默认是getter，也可以设置getter+setter。
    6. `watch`对象，被watch的属性改变而执行
    7. `filters`对象，过滤器
6. 组件

    >所有的 Vue.js 组件其实都是被扩展的 Vue 实例。

    原生HTML元素用`is`扩展。

    组件有自己独立的作用域，只能通过`props`传递数据。

    - 声明方式：

        1. 全局

            `Vue.component('组件元素名', 对象)`
        2. 局部

            ```
            new Vue({
                components: {
                    '组件元素名':对象
                }
            })
            ```

    1. `template`字符串，结构

        >`slot`使用
    2. `props`数组，创造新属性，接受父级传递内容
    3. `data`方法，return数据对象，for循环的话每个实例都调用创建一份。

    - 父子组件间的通信

        1. 父->子，通过`props`向下传递初始化数据
        2. 子->父，通过`$emit`向上传递事件、参数