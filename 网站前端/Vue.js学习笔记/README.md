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
2. Vue实例会代理`data`、`methods`的属性，也有以`$`开头的实例属性（如`$el`）。
3. 模板插值，适用表达式，语句不会生效。只能访问部分全局变量。
2. Vue实例的属性：

    `new Vue(对象)`

    1. `el`字符串，选择器
    2. `data`对象，数据
    3. 钩子

        1. `created`方法
        2. `updated`方法
        3. `destroyed`方法
        4. 等
    4. `methods`对象
    5. `computed`对象，自动依赖而计算

        默认是getter，也可以设置getter+setter。
    6. `watch`对象，被watch的属性改变而执行
    7. `filters`对象，过滤器
3. 指令

    DOM的特殊属性。

    1. `v-bind`绑定DOM属性与Vue实例属性

        `v-bind:`、`:`

        1. 绑定`class`
        2. 绑定`style`
    2. `v-if`、`v-else`、`v-else-if`

        `key`属性决定是否公用。
    3. `v-for`

        >比`v-if`优先级更高。

        在组件中使用`v-for`时，必须添加`v-bind:key='值'`。

        `v-bind:key='值'`决定是否复用。
    3. `v-on`事件处理

        `v-on:`、`@`

        1. `$event`事件
    4. `v-model`表单
    5. `v-one`一次性插值，不再响应式
    6. `v-html`输入真正HTML
    7. 修饰符：`v-on`、`v-bind`、`v-module`
    8. 过滤器：`|`
    9. `v-show`
4. 组件

    原生HTML元素用`is`扩展。

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
    2. `props`数组，创造新属性，接受父级传递内容
    3. `data`方法，return数据对象，for循环的话每个实例都调用创建一份。

    - 父子组件间的通信

        1. 父->子，通过`props`向下传递初始化数据
        2. 子->父，通过`$emit`向上传递事件、参数