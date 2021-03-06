# Vue.js学习笔记

## 目录
1. [vue](#vue)
1. [vue-router](#vue-router)
1. [vuex](#vuex)
1. [vue-cli](#vue-cli)
1. [nuxt.js](#nuxtjs)
1. [jQuery与Vue.js对比](#jquery与vuejs对比)

### [vue](https://github.com/vuejs/vue)
1. 单向数据流（实现双向绑定效果），响应式

    编写的代码不关注底层逻辑，只关注用**单向数据流**实现的**双向绑定**效果（Vue实例、组件间都是单向数据流）。

    1. 重用DOM
    
        1. 当没有`key`属性或`key`属性相同时：最大化DOM重用。
        2. 切换的DOM的`key`属性不同：不重用DOM。
    2. Vue实例代理`data`、`methods`、`computed`、`props`的属性内容，可以直接修改或调用；也有以`$`开头的实例属性（如`$el`、`$data`、`$watch`）。

        只有已经被代理的内容是响应的，值的改变（可能）会触发视图的重新渲染。

        1. 导致视图更新：

            1. 数组的变异方法：`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`。
            2. 数组赋值。
            3. 插值：`Vue.set(数组/对象, 索引/键, 新值)`：

                <details>
                <summary>e.g.</summary>

                ```javascript
                const vm = new Vue({
                  data: {
                    a: {
                      name: 'Anika'
                    }
                  }
                })

                Vue.set(vm.a, 'age', 27)
                ```
                </details>
            4. 改变数组长度：`数组.splice(新长度)`。
        2. 无法检测数组变动：

            1. 直接数组索引赋值。

                >视图更新的替代方法：`Vue.set(vm.items, index, value)`或`vm.items.splice(index, 1, value)`。
            2. 直接修改数组长度。

                >视图更新的替代方法：`vm.items.splice(newLength)`。
    3. 慎用~~箭头函数~~，`this`的指向无法按预期指向Vue实例。
    4. 因为HTML不区分大小写，所以大/小驼峰式命名的JS内容，在HTML使用时要转换为相应的短横线隔开式。

        不受限制、不需要转换：JS字符串模版、`.vue`组件。

        >JS字符串模版包括：`<script type="text/x-template">`、JS内联模板字符串。
2. 模板插值

    1. 支持JS表达式（单个），不支持~~语句~~、~~流控制~~。

        <details>
        <summary>e.g.</summary>
        
        ```html
        <!-- 可行 -->
        {{ number + 1 }}
        {{ ok ? 'YES' : 'NO' }}
        {{ Array.from(words).reverse().join('') }}
        <div v-bind:id="'list-' + id"></div>

        <!--
        是语句，不是表达式
        {{ var a = 1 }}

        流控制不会生效，请使用三元表达式
        {{ if ... }}

        不支持分号，无法添加多个表达式
        {{ message; 1 + 1 }}
        -->
        ```
        </details>
    2. 只能访问部分全局变量（白名单）；不允许访问自定义的全局变量（引入的其他库变量仅在JS代码中使用）。
    3. 作用域在所属Vue实例（组件也是Vue实例）中。
    4. `slot="字符串"`、`<slot name="字符串">`用于父级向子组件插入内容。
    5. 所有渲染结果不包含`<template>`
3. 指令

    指令（Directives）是带有`v-`前缀的DOM的特殊属性。

    >JS表达式的值改变时，将其产生的连带影响，响应式地作用于DOM。

    1. `:`参数

        用于添加指令后的参数。
    2. `v-if`、`v-else`、`v-else-if`
    3. `v-for="(值, 键, 索引) in/of JS表达式/整数"`

        >- 处于同一节点时，`v-for`比`v-if`优先级更高：这意味着`v-if`将分别重复运行于每个`v-for`循环中。
        >
        >    ```html
        >    <!-- e.g. -->
        >    <li v-for="todo in todos" v-if="!todo.isComplete">
        >      {{ todo }}
        >    </li>
        >    ```

        - 在组件中使用`v-for`时，必须添加`key`属性。
        - 尽可能在使用`v-for`时提供`key`属性。除非遍历输出的DOM内容非常简单，或是刻意依赖“就地复用”的默认行为以获取性能上的提升。

            ```html
            <!-- e.g. -->
            <div v-for="item in items" :key="item.id">
              <!-- 内容 -->
            </div>
            ```

            >没有提供`key`属性的高效的默认行为：如果数据项的顺序被改变，Vue将不会移动DOM元素来匹配数据项的顺序，而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。
    4. `v-bind`绑定DOM属性与JS表达式的结果。

        >此DOM属性随表达式最终值改变而改变，直接修改此DOM属性值不改变表达式的值。

        `v-bind:`缩写`:`。

        1. 绑定修饰符：
        
            1. `.sync`
        
                - 仅为语法糖：
                
                    `<my-component :foo.sync="bar"></my-component>`
                
                    等价于
                    
                    `<my-component :foo="bar" @update:foo="val => bar = val"></my-component>`
                    
                    - 若要达到效果（同步更新bar），还需要在组件中添加：
                    
                        ```javascript
                        Vue.component('myComponent', {
                          props: ['foo'],
                          template: '<p v-on:click="doIt">{{foo}}</p>',
                          methods: {
                            doIt () {
                              this.$emit('update:foo', 'new value') // 触发父级事件，父级事件改变值，再传入子组件
                            }
                          }
                        })
                        ```
        2. 特殊的DOM属性：
        
            1. 绑定`class`
    
                1. `v-bind:class="{class名: Vue属性[, class名: Vue属性]}"`
                2. `v-bind:class="Vue属性对象"`
                3. `v-bind:class="[Vue属性[, Vue属性]]"`
                4. `v-bind:class="[Vue属性 ? Vue属性 : ''[, Vue属性]]"`
                5. `v-bind:class="[{class名: Vue属性}[, Vue属性]]"`
            2. 绑定`style`
    
                >1. 自动添加样式前缀。
                >2. CSS属性名可以用驼峰式（camelCase）或短横线分隔（kebab-case，需用单引号包裹）命名。
    
                1. `v-bind:style="{css属性: Vue属性[, css属性: Vue属性]}"`
                2. `v-bind:style="Vue属性对象"`
                3. `v-bind:style="[Vue属性[, Vue属性]]"`
                4. 多重值
        3. 传递给子组件DOM属性的值类型

            <details>
            <summary>e.g.</summary>

            ```html
            <!-- 传递字符串 -->
            <my-component some-prop="1">传递字符串'1'</my-component>
            <my-component some-prop="a">传递字符串'a'</my-component>

            <!-- 传递表达式的值 -->
            <my-component v-bind:some-prop="1">传递表达式：1（Number）</my-component>
            <my-component v-bind:some-prop="'1'">传递表达式：'1'（String）</my-component>
            <my-component v-bind:some-prop="a">传递表达式：a（变量是什么类型就是什么类型）</my-component>
            ```
            </details>
        4. 若不带参数的`v-bind="表达式"`，则绑定表达式的所有属性到DOM。

            <details>
            <summary>e.g.</summary>

            ```html
            <div id="test">
              <div v-bind="objs">绑定了title和href属性</div>
              <div v-bind="{alt: 123, href:'https://asd.asd'}">绑定了alt和href属性</div>
            </div>

            <script>
              const vm = new Vue({
                el: '#test',
                data: {
                  objs: {
                    title: 'My title',
                    href: 'http://asd.asd'
                  }
                }
              })
            </script>
            ```
            </details>
    5. `v-on`事件监听

        `v-on:`缩写`@`。

        1. 事件修饰符：

            1. `.stop`（阻止冒泡）、`.prevent`（阻止默认行为）、`.capture`（捕获事件流）、`.self`（只当事件在该元素本身而不是子元素触发时才触发）、`.once`（事件将只触发一次）
            2. `.enter`、`.tab`、`.delete`、`.esc`、`.space`、`.up`、`.down`、`.left`、`.right`、`.数字键值`、[KeyboardEvent.key的段横线形式](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/key/Key_Values)

                键盘。
            3. `.left`、`.right`、`.middle`

                鼠标。
            4. `.native`

                监听组件根元素的原生事件，在父级引用子组件处添加。
            5. `.exact`

                精确匹配（有其他按键则失败）。

            >- 可同时使用，但改变顺序会产生不同效果。
            >
            >    <details>
            >    <summary>e.g.</summary>
            >
            >    ```html
            >    <!-- Alt + C -->
            >    <input @keyup.alt.67="clear">
            >
            >    <!-- Ctrl + Click -->
            >    <div @click.ctrl="doSomething">...</div>
            >
            >    <!-- 会阻止所有的点击 -->
            >    <div v-on:click.prevent.self="doThat">...</div>
            >
            >    <!-- 只会阻止元素上的点击 -->
            >    <div v-on:click.self.prevent="doThat">...</div>
            >    ```
            >    </details>
        2. `$event`原生DOM事件的变量，仅能由HTML传入

            <details>
            <summary>e.g.</summary>

            ```html
            <div id="test">
              <a href="#" v-on:click="a($event)">click</a>
            </div>

            <script>
              const vm = new Vue({
                el: '#test',
                methods: {
                  a: function (e) {
                    console.log(e)
                  }
                }
              })
            </script>
            ```
            </details>
        3. 自定义事件
        
            仅定义在子组件引用上，只能由子组件内部`$emit`触发，然后调用父级方法，再通过改变父级属性去改变子组件的`props`或置换组件。
    6. `v-model`表单

        >忽略表单元素上的`value`、`checked`、`selected`等初始值，而仅通过Vue实例赋值。

        1. 表单修饰符：
        
            `.lazy`（在`change`而不是`input`事件触发）、`.number`（输入值转为`Number`类型）、`.trim`（过滤首尾空格）
        2. 仅针对部分元素：`<input>`、`<textarea>`、`<select>`、组件
        3. 仅为语法糖：

            1. 普通DOM

                `<input v-model="bar">`

                等价于

                `<input v-bind:value="bar" v-on:input="bar = $event.target.value">`
            2. 组件

                `<my-input v-model="bar"></my-input>`

                等价于（默认：属性绑定为`value`、事件绑定为`input`）

                `<my-input v-bind:value="bar" v-on:input="bar = arguments[0]"></my-input>`

                - 若要达到效果（双向数据绑定），还需要在组件中添加：

                    ```javascript
                    Vue.component('myInput', {
                      props: ['value'],
                      template: '<input v-bind:value="value" v-on:input="updateValue($event)">',
                      methods: {
                        updateValue (e) {
                          this.$emit('input', e.target.value) // 触发父级事件，父级事件改变值，再传入子组件
                        }
                      }
                    })
                    ```
        4. 绑定的Vue实例的值：

            1. DOM的`value`属性的值；
            2. 若是`type="checkbox"`，则为`true/false`；
            3. 若要获得Vue实例的动态属性值：

                1. 用`v-bind:value="表达式"`；
                2. 若`type="checkbox"`，则用`v-bind:true-value="表达式" v-bind:false-value="表达式"`。
    7. `v-once`一次性插值，不再双向绑定数据
    8. `v-html`输入真正HTML

        >- 与其他插值（如模板插值）的区别：
        >
        >    1. `v-html`是`innerHTML`。
        >    2. 其他是`innerText`。
    9. `.`修饰符

        >用于指出一个指令应该以特殊方式绑定。

        使用在`v-on`、`v-bind`、`v-module`后添加。
    10. `|`过滤器，参数带入函数运行出结果（支持过滤器串联）

        <details>
        <summary>e.g.</summary>

        ```html
        <div id="test">
          <p>{{ 1 | a | b }}</p>    <!-- 3 -->
          <p>{{ num | a | b }}</p>  <!-- 5 -->
        </div>

        <script>
          const vm = new Vue({
            el: '#test',
            data: {
              num: 2
            },
            filters: {
              a: function (e) {
                return e * 2
              },
              b: function (e) {
                return e + 1
              }
            }
          })
        </script>
        ```
        </details>
    11. `v-show`

        总是渲染出DOM，根据值切换`display`值。

        >不支持`<template>`、不支持`v-else`。
4. Vue实例的属性：

    `new Vue(对象)`

    1. `el`（字符串）：选择器
    2. `data`（对象）：数据

        >要用到的数据必须添加初始值。
    3. `methods`（对象）：可调用方法
    4. `computed`（对象）：依赖的`data`属性改变而执行（不允许在`data`中出现同样的属性）

        默认是`getter`（初始化时会调用一次），也可以设置`getter`和`setter`（被赋值时执行）。

        ```javascript
        // e.g.
        const vm = new Vue({
          data: {
            firstName: 'firstname',
            lastName: 'lastname'
          },
          computed: {
            fullName: {
              // getter
              get: function () {            // 初始化和this.firstName或this.lastName改变时调用；this.fullName被赋值时不会直接调用
                return this.firstName + ' ' + this.lastName
              },
              // setter
              set: function (newValue) {    // this.fullName被赋值时调用
                const names = newValue.split(' ')
                this.firstName = names[0]
                this.lastName = names[names.length - 1]
              }
            }
          }
        })

        // 运行 vm.fullName = 'John Doe' 时，setter 会被调用，vm.firstName 和 vm.lastName 也会相应地被更新
        ```
    5. `watch`（对象）：被watch的`data`属性改变而执行（必须是在`data`中出现的属性）
    6. `filters`（对象）：过滤器方法
    7. `components`（对象）：局部注册组件
    8. 生命周期钩子

        1. `beforeCreate`
        2. `created`
        3. `beforeMount`
        4. `mounted`
        5. `beforeUpdate`
        6. `updated`
        7. `activated`
        8. `deactivated`
        9. `beforeDestroy`
        10. `destroyed`
        11. `errorCaptured`
    9. `mixins`（数组，单项为Vue属性对象）：混合

        1. 当组件和混合对象含有同名选项时，这些选项将以恰当的方式混合。
        2. 混合对象的**钩子**将在组件自身钩子之前调用。
        3. 值为对象的选项（例如`methods`、`components`、`directives`），将被混合为同一个对象；两个对象键名冲突时，取组件对象的键值对。

        - `Vue.mixin`全局注册混合对象，将会影响所有之后创建的（之前的不受影响）Vue实例，包括第三方模板。
5. 组件

    >所有组件都是被扩展的Vue实例：`new (Vue.extend({对象}))()`。

    1. 组件属性：

        1. `template`（字符串）：组件的字符串模板

            - 作用域：

                `template`的字符串模板内容为本组件作用域；父级添加的DOM（包括子组件引用）为父级作用域。
            - 内容分发

                使用`<slot>`可在子组件内部插入父级引入的内容（`slot="字符串"`）。

                1. 默认

                    1. 模板中没有`name`属性的`<slot>`，匹配父级中去除所有`slot="字符串"`引用的内容。
                    2. 模板中`<slot>`的DOM内容，仅当没有父级匹配时显示。
                2. 具名

                    1. 父级引用子组件，在元素内部添加的标签的DOM属性`slot="字符串"`；
                    2. 组件模板：`<slot name="字符串">`。
                3. 作用域插槽

                    1. 父级引用子组件元素内部的内容为：

                        `<template slot-scope="临时变量">{{临时变量.组件属性}}</template>`

                        >`<template>`使用`slot-scope`属性时，不要同时使用`v-if`。
                    2. 组件模板：

                        `<slot 组件属性="字符串">`或`<slot v-bind:组件属性="表达式">`
        2. `props`

            接受父级传递内容。

            1. （数组）：接受的DOM属性
            2. （对象）：`接受的DOM属性: 验证`

                - 验证：

                    >`props`会在组件实例创建之前进行校验。

                    1. 原生构造器（`String`、`Number`、`Boolean`、`Function`、`Object`、`Array`、`Symbol`）、`null`（允许任何类型）
                    2. 数组
                    3. 对象（type、required、default、validator）
        3. `data`（方法）：`return`数据对象

            ```javascript
            data: function () {
              return {
                a: 0,
                b: ''
              }
            }
            ```

            >`v-for`循环的每个实例都调用创建一份；仅执行一次，父组件传进来的props改变也不再触发执行。
        4. `model`（对象，包含`prop`、`event`）：修改`v-model`默认使用的属性和事件
    2. 注册组件方式：

        >要确保在初始化Vue实例之前注册了组件。

        1. 全局

            `Vue.component('组件元素名', 对象)`
        2. 局部

            ```javascript
            // Vue实例
            new Vue({
                components: {
                    '组件元素名': 对象 // 仅在此Vue实例中可用
                }
            })
            ```
    3. 组件命名：
    
        >W3C规定的组件命名要求：小写，且包含一个`-`。
        
        1. JS注册组件或`props`：
        
            短横线隔开式（kebab-case）、小驼峰式（camelCase）、大驼峰式（PascalCase）。
        2. HTML中：
        
            1. 仅能使用短横线隔开式（把大/小驼峰式用`-`隔开单词代替）。
            2. 在JS字符串模版、`.vue`组件，可以使用额外方式：

                <details>
                <summary>e.g.</summary>

                ```html
                <!-- HTML必须是短横线隔开式 -->
                1. <kebab-cased-component></kebab-cased-component>
                2. <camel-cased-component></camel-cased-component>
                3. <pascal-cased-component></pascal-cased-component>

                <!-- 在JS字符串模版、.vue组件额外可以使用 -->
                1. 无
                2. <camelCasedComponent></camelCasedComponent>
                3. <pascalCasedComponent></pascalCasedComponent>
                3. <PascalCasedComponent></PascalCasedComponent>
                
                 <script>
                 // 注册的组件
                 new Vue({
                   components: {
                     'kebab-cased-component': { /* ... */ },
                     camelCasedComponent: { /* ... */ },
                     PascalCasedComponent: { /* ... */ }
                   }
                 })
                 </script>
                ```
                </details>
    4. 使用组件：

        1. `<组件名></组件名>`

            >不会导致渲染出错的方式：JS字符串模版、`.vue`组件。
        2. `<标签 is="组件名"></标签>`
        3.  动态组件：`<component v-bind:is="表达式"></component>`
        
            ```html
            <div id="test">
              <component v-bind:is="current1"></component>
              <component v-bind:is="current2"></component>
            </div>

            <script>
              const vm = new Vue({
                el: '#test',
                data: {
                  current1: { template: '<p>000</p>' },     // 对象，则当作组件对象渲染
                  current2: 'component1'                    // 字符串，则搜索components
                },
                components: {
                  component1: { template: '<p>111</p>' },
                  component2: { template: '<p>222</p>' }
                }
              })
            </script>
            ```

            - `<keep-alive>`包裹，保留它的状态或避免重新渲染。
    5. 通信

        1. 父子组件间的通信

            >组件有自己独立的作用域：父子组件间、Vue实例间作用域独立，可以访问到互相依赖关系（`$parent`、`$children`），但是不建议（不允许）通过依赖获取、修改数据。

            1. 父->子：通过`props`向下传递初始化数据给子组件实例（不出现在DOM中）
            
                >添加在DOM上而不在`props`声明，则仅添加到子组件最外层的DOM属性，不传入子组件。其中`class`和`style`属性会合并，其他属性会覆盖。

                1. `props`是单向绑定的：当父级的属性变化时，将传导给子组件，不会反过来。
                
                    每次父组件更新时，子组件的所有prop都会更新为最新值。
                2. 不应该在子组件内部改变`props`，而应该定义一个局部变量（`data`），用`props`去初始化。

                    e.g.

                    ```javascript
                    // good
                    Vue.component(
                      'myComponent',
                      {
                        props: ['father'],
                        computed: {
                          son() {
                            return this.father
                          }
                        },
                        template: '<div>{{son}}</div>'
                      }
                    )

                    // bad
                    Vue.component(
                      'myComponent',
                      {
                        props: ['father'],
                        template: '<div>{{father}}</div>'
                      }
                    )
                    
                    ```

                >注意避免**引用数据类型**导致的子组件改变父级。
            2. 子->父：通过`$emit`向上传递事件、参数（去触发外部环境修改传进组件的`props`值，完成单向数据流的双向绑定）

                1. 在父级引用子组件处添加`v-on:自定义事件1="父方法"`监听；
                
                    >若`自定义事件1`是原生事件，可以添加`.native`修饰符，监听组件根元素的原生事件（不接收子组件的 ~~$emit~~）。
                2. 在子组件方法内添加`this.$emit('自定义事件1', 参数)`向上传递。
        2. 非父子组件通信

            1. 在简单的场景下，可以使用一个空的Vue实例作为中央事件总线。
            
                ```javascript
                const bus = new Vue()
 
                // 触发组件 A 中的事件
                bus.$emit('事件名', 传参)
 
                // 在组件 B 创建的钩子中监听事件
                bus.$on('事件名', function (para) {/* ... */})
                ```
            2. 或专门状态管理模式，如[vuex](https://github.com/vuejs/vuex)。
        - 组件的API来自三部分

            `<my-component :子属性="父属性" @事件="父方法"><标签 slot="名字">内容分发</标签></my-component>`

            1. `props`：允许外部环境传递数据给组件。
            2. `events`：允许从组件内触发外部环境的副作用。
            3. `slots`：允许外部环境将额外的内容组合在组件中。

    - 杂项
  
        1. 父级引用组件时添加属性`ref="字符串"`，可以在Vue实例的`$refs`中访问。
        2. 内联模板：
        
            引用组件时，添加`inline-template`DOM属性。组件的内容当作模板，而不是分发内容。
        3. `<script type=text/x-template id="id名">`
        
            ```javascript
            Vue.component('组件名', {
              template: '#id名'
            })
            ```
        4. 当组件中包含大量静态内容时，可以考虑使用`v-once`将渲染结果在组件注册的`template`字段里缓存起来。
        5. 异步组件。
        6. 高级异步组件。
        7. 递归组件。
        8. 循环组件。
6. 过渡&动画

### [vue-router](https://github.com/vuejs/vue-router)
>使用Charles代理到本地dev环境（map remote），要保证被代理和代理的路径相同，才能让路由正确。

### [vuex](https://github.com/vuejs/vuex)

### [vue-cli](https://github.com/vuejs/vue-cli)
快速构建Vue应用的脚手架，可以使用Vue官方或第三方模板来进行Vue应用的配置，一般包括webpack等工具的配置。

### [nuxt.js](https://github.com/nuxt/nuxt.js)
基于Vue的通用应用框架，把webpack、vue-loader、vuex、vue-router等工具整合在一起，并通过自带的`nuxt.config.js`统一配置，不需要对每个工具进行单独配置。内置了SSR解决方案。

>框架内的Vue组件都是以**Vue单文件组件**的形式，每一个`pages`目录下的组件都是一个页面。开发模式强制是SSR，打包可以选择是否SSR。

1. 目录结构

    1. `pages`：页面目录
        
        Vue单文件组件。目录中的`.vue`文件自动生成对应的路由配置。
    2. `assets`：待编译资源目录
    
        默认使用webpack的vue-loader、file-loader、url-loader加载器进行编译的资源，如`脚本（js、jsx、tsx、coffee）`、`样式（css、scss、sass、less）`、`模版（html、tpl）`、`JSON`、`图片`、`字体`文件。
        
        - 引用方式：组件中HTML引用、ES6引用`~/assets/`
        
        >对于不需要编译的静态资源可以放在`static`目录。
    3. `static`：静态资源目录
    
        不需要webpack编译的静态资源。该目录下的文件会映射至项目根路径。

        - 引用方式：组件中HTML引用`/`
    4. `components`：组件目录
    
        Vue单文件组件。提供给项目中所有组件使用。
        
        - 引用方式：组件中ES6引用`~/components/`

            <details>
            <summary>任意组件引用<code>components</code>目录下组件的方式</summary>
            
            ```html
            <template>
              <div>
                ...
                <other-component/>
                ...
              </div>
            </template>
            
            <script>
              import OtherComponent from '~/components/OtherComponent.vue';
            
              export default {
                components: {
                  OtherComponent
                },
                ...
              };
            </script>
            ```
            </details>
    5. `plugins`：Vue插件目录
    
        JS文件。在Vue根应用的实例化前需要运行的Vue插件（Vue添加全局功能）。
        
        - 引用方式：`nuxt.config.js`中加入`plugins`属性
        
            <details>
            <summary><code>nuxt.config.js</code>文件引用<code>plugins</code>目录下插件的方式</summary>
            
            ```javascript
            // nuxt.config.js
            module.exports = {
              plugins: ['~/plugins/插件文件名']   // 一般也会配置在vendor.bundle.js中
            }
            ```
            </details>
    6. `layouts`：布局目录
    
        Vue单文件组件。扩展默认布局（`default.vue`、`error.vue`）或新增自定义布局。在布局文件中添加`<nuxt/>`指定页面主体内容。
        
        - 引用方式：`pages`目录下组件中加入`layout`属性

            <details>
            <summary><code>pages</code>目录下组件引用<code>layouts</code>目录下布局的方式</summary>
            
            ```html
            <!-- layouts/布局文件名.vue -->
            <template>
              <div>
                ...
                <nuxt/>
                ...
              </div>
            </template>
            ...
            ```
            
            ```javascript
            // pages/页面名.vue
            export default {
              layout: '布局文件名',
              // 或
              layout (context) {
                return '布局文件名'
              },
              ...
            }
            ```
            </details>
    7. `store`：状态目录
    
        >若`store`目录存在，则：引入`vuex`->增加`vuex`至vendor配置->设置Vue根实例的`store`配置项。
    
        - 两种使用方式
        
            1. 普通方式：`store/index.js`返回`store`实例
            2. 模块方式：`store`目录下每个`.js`文件被转换为**指定命名的子模块**
    8. `middleware`：中间件目录

        JS文件。路由跳转之后，在页面渲染之前运行自定义函数。执行顺序：`nuxt.config.js`->`layouts`->`pages`。
        
        - 引用方式：`nuxt.config.js`文件或`layouts`或`pages`目录下组件中加入`middleware`属性

            <details>
            <summary><code>nuxt.config.js</code>文件或<code>layouts</code>或<code>pages</code>目录下组件引用<code>middleware</code>目录下中间件的方式</summary>
            
            ```javascript
            // middleware/中间件文件名.js
            export default function (context) {
              // 路由跳转之后，且在每页渲染前运行
              ...
            }
            ```
            
            ```javascript
            // nuxt.config.js
            module.exports = {
              router: {
                middleware: ['中间件文件名']
              },
              // 或
              middleware: ['中间件文件名'],
              ...
            }
  
  
            // layouts/布局文件名.vue
            module.exports = {
              middleware: ['中间件文件名'],
              ...
            }
  
  
            // pages/页面名.vue
            module.exports = {
              middleware: ['中间件文件名'],
              ...
            }
            ```
            </details>
    9. <details>
        
        <summary><code>nuxt.config.js</code>：配置文件</summary>
    
        1. `build`
        
            在自动生成的`vendor.bundle.js`文件中添加模块，以减少项目bundle的体积。
            
            1. `build.vendor`：配置公用插件`vendor.bundle.js`，避免每个页面都打包一次插件。
            
                <details>
                <summary>e.g.</summary>
                
                ```javascript
                module.exports = {
                  build: {
                    vendor: ['已安装插件名', '~/plugins/插件文件名']
                  },
                  ...
                };
                ```
                </details>
        2. `css`
        
            配置全局的（所有页面均需引用）样式，包括文件、模块、第三方库。
        3. `dev`
        
            配置开发/生产模式。
        4. `env`
        
            配置（客户端和服务端）环境变量。
        5. `generate`
        
            配置每个动态路由的参数，依据这些路由配置生成对应目录结构的HTML。
        6. `head`
        
            配置HTML的头部信息。
        7. `loading`
            
            配置加载组件。
        8. `modules`
        
            配置需要添加的nuxt模块。
        9. `plugins`
        
            配置在Vue根应用的实例化前需要运行的JS插件。
        10. `rootDir`
        
            配置根目录。
        11. `router`
        
            配置覆盖默认的`vue-router`配置。
        12. `srcDir`
            
            配置源码目录。
        13. `transition`
        
            配置过渡效果属性的默认值。
        </details>
    10. `.nuxt`：nuxt内部构建资源目录（不要修改）
    11. 根目录中创建自定义文件夹，放置JS模块，提供给其他文件引用
    
    ><details>
    >
    ><summary>别名</summary>
    >
    >1. `srcDir`：`~`或`@`
    >2. `rootDir`：`~~`或`@@`
    >
    >- 默认的`srcDir`等于`rootDir`
    ></details>
2. 视图

    1. 定制HTML模板
    
        <details>
        <summary>在根目录添加<code>app.html</code>，可在其中添加任意静态自定义内容</summary>
        
        ```html
        <!DOCTYPE html>
        <html {{ HTML_ATTRS }}>
          <head>
            {{ HEAD }}
          </head>
          <body {{ BODY_ATTRS }}>
            {{ APP }}
          </body>
        </html>
        ```
        </summary>
3. 路由

    依据`pages`目录结构自动生成`vue-router`模块的路由配置。
4. 命令

    `nuxt`、`nuxt build`、`nuxt start`、`nuxt generate`

### jQuery与Vue.js对比
1. 做的事情

    1. jQuery

        （旧时代到现在）相对于原生JS，更好的API，兼容性极好的DOM、AJAX操作。面向网页元素编程。
    2. Vue.js

        实现MVVM的数据双向绑定，实现自己的组件系统。面向数据编程。
2. 优劣势对比

    1. jQuery

        1. 兼容性好，兼容基本所有当今浏览器；出现早，学习、使用成本低。
        2. 程序员关注DOM，频繁操作DOM；代码量较多且不好维护、扩展，当页面需求变化之后代码改动难度大。
    2. Vue.js

        1. 程序员关注数据，DOM的操作交给框架；代码清晰、强制规范，利于维护；有自己的组件系统。
        2. 不兼容旧版本浏览器；需要一些学习成本。
