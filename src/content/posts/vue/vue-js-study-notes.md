---
title: VueJS Study Notes
published: 2021-07-25
description: ''
image: ''
tags: ['Vue', 'Study']
category: 'Develop'
draft: false
---

1.创建vue对象
```vuejs
var vm = new vue({
        el:'元素' ，    //id选择器或者类选择器
        data:{      //数据      键值对格式
            msg:'123'
        },
        methods:{       //函数
            show:function(){
               console.log('show f');
            }
        }
    })
```

2.插入表达式(绑定数据) 语法：{{字段名}}     相当于占位符
    v-text='字段'   没有闪烁（渲染网页会先执行html代码在执行vue），但会覆盖标签内容
    v-cloak    放在插入表达式所在标签内，防止闪烁，要先将其隐藏（[v-cloak]{display:none;}）
    v-html='字段'    渲染数据字段中的<标签>
    v-bind:title='字段'    绑定标签属性值   简写：“ : ”
    v-on:事件='函数名'     绑定事件 简写：“ @ ”    例如 v-on:clcik='show'
    事例：`<input type="button" value="按钮" v-on:click='show'>`

 3.this关键字获取对象实例
    3.1： 函数中获取data中的数据需要用`this`关键字，例如`this.msg`;调用VM中的函数也需要使用`this关键字.vm`实例会监听`data`中数据的改变，当数据发生变化会自动同步到页面上。
    3.2： 函数内部调用方法后this指向改变了，可以使用 `var that = this` 保存`this`的值，或者使用 `() => {代码块}`  的形式让`this`指向外部。

4.事件修饰符：  语法 ： `@事件名.修饰符:'函数名'`
    4.1：.stop 阻止冒泡触发事件(html默认会冒泡执行所有事件.stop可以阻止这一行为，只调用该方法)。
    4.2: .prevent 阻止元素默认行为 例如：住址链接的跳转、按钮的提交等等。
    4.3: .capture 捕获触发事件 会触发当前点击标签内所有标签的事件。
    4.4: .self 点击元素本身才会触发事件 只能阻止自身的冒泡行为，并不能阻止整个冒泡行为。
    4.5：.once 只触发一次。
    事例：`<button type="button" @click.once='btnClick'>按钮</button>`

5.数据双向绑定v-model：语法： `v-model='字段'`
    修改v-model 中的值 会同步到data中，只能运用于表单元素中。
    v-bind: 单向绑定，修改值不会同步到data中。
    事例：  `<p>{{msg}}</p>
            <input type="text" style="width: 50%;" v-model='msg'>`

6.vue样式绑定方式：
    6.1:绑定class
        6.1.1:使用数组绑定：`v-bind:class="['class1','class2']" `。
        6.1.2:数组中使用三元表达式： `:class="['class1','class2',true?'class3':'']"` 。
        6.1.3:数组中使用对象： `:class="['class1','class2',{'class3':true}]"` 。
        6.1.4:使用对象绑定： `:class="{'class1':true,'class2':false}" 或 :class="对象名"`。

6.2:内联样式绑定
        6.2.1:使用对象：`:style="{color='red','font-size':'33px'}"`或 `:style="对象名"`。
        6.2.2:绑定多个对象：`:style="['对象1','对象2']"`。

7.v-for循环遍历：
    7.1:循环数组：语法：` v-for='item in 数组名'` 实例： `<p v-for='(item , index) in arr'>值：{{item}}---索引：{{index}}</p> `（索引可省略）
    7.2:循环数组对象：语法： `v-for='item in 数组名'` 实例：`<p v-for='(user , index) in arr2'>name：{{user.name}}---age:{{user.age}}---索引：{{index}}</p>`
    7.3:循环对象：语法: `v-for='item in 对象名'` 实例：`<p v-for='(proitem , key, index) in pro'>{{proitem}}--key：{{key}}---索引：{{index}}</p>`
    7.4:迭代数字：语法: `v-for='count in int类型数据'` 实例：`<p v-for='count in 10'>{{count}}</p> `（从1开始）
    7.5:key的注意事项：key属性只能使用 `number` 或 `string` 类型作为唯一标识，在使用`v-for`时尽量带有key值。 key绑定语法：` v-bind:key='number/string'`

8.`v-if`和`v-show`的区别
    8.1:`v-if` 会创建或删除dom元素，`v-show`会显示和隐藏dom元素（添加`display:none`样式）。
    8.2:`v-if` 有较高的切换性能消耗，`v-show`有较高的初始渲染消耗。
    8.3:涉及到频繁切换最好不要使用`v-if`，如果元素从来不会被显示出来被用户看到这推荐使用`v-if`。
    事例：```<input type="button" value="toggle" @click='flag=!flag'>
            <p v-if='flag'>v-if渲染</p>
            <p v-show='flag'>v-show渲染</p>```

9.vue过滤器
    全局过滤器定义语法：`Vue.filter('过滤器名',function(需要过滤的数据,参数1,参数2){数据处理...}) `
    局部过滤器定义语法：`new Vue({el:'dom元素',filters:{过滤器名:function(数据,参数1,参数2...){处理数据...}}})`
    调用语法：`{{ 变量 | 过滤器1 | 过滤器2(参数1,参数2) ... }}`
    注意：过滤器调用顺序从前往后，遵循就近原则(有局部先调用局部过滤器)，全局过滤器与局部过滤器能重名。
    事例：  ```<p>{{msg | msgFormat('***','(口吐芬芳)') | test}}</p>
            Vue.filter('msgFormat',function(data,arg,arg2){
                return data.replace(/过滤器/g,arg+arg2)
            })```

10.按键修饰符
    语法：`@按键触发事件.修饰符='函数'。`
    vue自带按键修饰符：`enter tab delete space up down left right。`
    自定义全局按键修饰符：`Vue.config.keyCodes.按键名= 按键对应js码。`
    事例：  ```<input type="text" value="enter" @keyup.enter='enter()'>
            <input type="text" value="f2" @keyup.113='f2()'>
            <input type="text" value="f2自定义按键修饰符" @keyup.f2='f2()'>
            Vue.config.keyCodes.f2=113```

11.自定义指令
    自定义全局指令：语法：`Vue.directive('指令名',{触发指令相关函数})。`
    自定义局部指令：语法：`new Vue({el:'dom元素',directives:{'指令名':{触发相关函数}}}) `。
    指令相关函数：
        1.`bind` 当指令绑定到元素上时立即触发，只执行一次。
        2.`inserted` 当元素插入到DOM中触发，只执行一次。
        3.`update` 当VNode更新时触发，可执行多次。
        4.`componentUpdated` 指令所在组件的 VNode 及其子 VNode 全部更新后调用。
        5.`unbind` 当指令与元素解绑时触发，只执行一次。
    注意：定义指令时 指令名不需要加 v- 前缀，调用指令时要加 v- 前缀。
    事例： ```<input type="text" v-focus v-color="'red'" v-font-size="'30px'">
        Vue.directive('focus',{
            bind:function(el){  //当指令绑定到元素上是触发该函数
            },
            inserted:function(el){  //当元素插入到dom中时触发该函数
                el.focus();
            },
            updated:function(){ //当VNode更新时会触发当前函数，能触发多次
            }
        })
        //私有的自定义指令
        var vm = new Vue({
            el:'#app',
            directives:{
                color:{
                    bind(el,binding){
                        console.log(binding.name);  //获取该指令名称
                        console.log(binding.value);     //获取该指令值 计算后的
                        console.log(binding.expression);    //获取该指令的值 原样返回
                        el.style.color = binding.value;
                    }
                },
                'font-size':function(el, binding){    //函数简写 当只用到 bind 和 update 时可以使用简写
                    el.style.fontSize = parseInt(binding.value) +'px'
                }
            }
        })```

12生命周期函数
    12.1: `beforeCreate` 在实例完全被创建出来之前会执行该函数，不能获取到 data 的值和 methods 的方法。
    12.2: `created` 在实例创建完毕后执行该函数，能获取到 data 的值和 methods 的方法（已经被初始化好了）。
    12.3: `beforeMount` 在渲染页面之前执行该函数，不能获取到页面最终的值，只能获取到之前写的模板字符串。
    12.4: `mounted` 在渲染页面完毕后执行该函数，能获取到页面最终值。
    12.5: `beforeUpdate` 在 data 的数据改变后、页面重新渲染之前触发(此时 data 值已经改变，但页面上的值还未发生改变)。
    12.6: `updated` 在 data 的数据改变后、页面重新渲染之后触发。
    12.7: `beforeDestroy` 在Vue实例销毁前执行该函数，此时Vue实例上的所有 data 、 methods、过滤器等都是可以使用的。
    12.8: `destroy` 在Vue实例销毁之后触发该函数，此时所有组件都无法使用。
    顺序：
        1.创建阶段：创建实例(new Vue({}))-->初始化一个空Vue实例(只有一些生命周期函数和默认事件)-->执行 beforeCreate 函数-->初始化数据和方法-->执行 created 函数(此时能获取到 data 和 methods 中的数据了)-->将el绑定的DOM元素编译成模板(存放在内存中)-->执行 beforeMount 函数-->将内存中的模板挂载到浏览器页面上-->执行 mounted 函数(此时Vue实例已经创建完毕)。
        2.运行阶段：data内的数据被修改时-->执行 beforeUpdate 函数(页面上的值还未被修改)-->将 data 中最新的数据在内存中重新渲染出新的DOM树，将新DOM树渲染到页面上(此时完成了data(model层)->view(视图层)的更新)-->执行 updated 函数(页面上的值已经被修改完成)。
        3.销毁阶段：执行 beforeDestroy 函数--> 解除绑定，销毁组件、监听器 --> 执行 destroy 函数(销毁完毕)。
    事例：
```
<div id="app">
            <input type="button" @click="msg='no'" value="修改msg">
            <h1 id="test">{{ msg }}</h1>
        </div>

        <script src="lib/vue.js"></script>
        <script>
            var vm = new Vue({
                el:'#app',
                data:{msg:'ok'},
                methods:{show(){console.log('执行了show方法')}},
                beforeCreate(){ console.log(this.msg) },
                created(){ console.log(this.msg);this.show()},
                beforeMount(){console.log(document.getElementById('test').innerText)},
                mounted(){console.log(document.getElementById('test').innerText)},
                beforeUpdate(){console.log('页面中的msg：'+document.getElementById('test').innerText+' / data中的msg:'+this.msg)},
                updated(){console.log('页面中的msg：'+document.getElementById('test').innerText+' / data中的msg:'+this.msg)}
            })
        </script>
```

13.`vue-resource`发送`http`请求
    13.1、请求种类：`get、post、jsonp`
    13.2、语法：```this.$http.get('url地址',{可选参数}).then(成功的回调,失败的回调);
                this.$http.post('url地址',{要传输的数据},{可选参数}).then(成功的回调,失败的回调);```
    13.3、定义全局请求根地址。实例：`Vue.http.options.root='http://localhost:2013'。`
    13.4、定义局部请求根目录。实例：`new Vue({el:'#app',http:{root:'url'}})。`
    13.5、全局启用 emulateJSON。实例：`Vue.http.options.emulateJSON = true。局部启用 emulateJSON。实例：new Vue({el:'#app',http:{emulateJSON:true}})。`
    注意：vue-resource需要引入包。地址：https://www.bootcdn.cn/vue-resource/
        参数地址：https://github.com/pagekit/vue-resource/blob/develop/docs/http.md
        使用根地址后 url 最前面不能带 " / "。
    事例：```new Vue({
            el:'#app',
            data:{msg:'ok'},
            http:{  //定义局部根地址
                root:'http://localhost:2013',
                emulateJSON:true    //启用 emulateJSON
            },
            methods:{
                getInfo(){this.$http.get('url').then(function(result){console.log(result.data)},function(erorrResult){console.log(erorrResult)})},
                postInfo(){this.$http.post('url',{},{emulateJSON : true}).then(function(result){console.log(result)})},
                jsonpInfo(){this.$http.post('url',{emulateJSON : true}).then(result => {console.log(result)})}
            }
        })```

14.动画
    14.1、通过修改过渡类名实现动画效果:
        `Vue` 官方提供一个元素 `transition` 使用该元素将需要执行动画的元素包裹。
        动画列表过滤：使用 `transition-group` 元素进行包裹。
        默认类名以 v- 开头，定义多个不同动画需要给 `transition` 加 `name` 属性修改前缀。
        动画类名：
            1、`v-enter` 动画入场前状态。
            2、`v-enter-to` 动画入场完毕状态。
            3、`v-leave` 动画离场前状态。
            4、`v-leave-to` 动画离场后状态。
            5、`v-enter-active` 动画入场时间段。
            6、`v-leave-active` 动画离场时间段。
            7、`v-move` 元素移除时动画。
        注意：使用 `v-move` 时需要对 `v-leave-active` 设置 `position : absolute` ，并对要移除的元素设置宽度。
            添加 `tag='元素名'`，可以设置 `transition` 或 `transition-group` 元素 渲染后的元素(默认渲染为`span`)。
            给 `transition-group` 添加 `appear` 属性，可以使页面渲染刚渲染出来时有入场效果。
        事例：
```
.v-enter,
                .v-leave-to{
                    opacity: 0;
                    transform: translateX(200px);
                }
                .v-enter-active,
                .v-leave-active{
                    transition: all 1s ease;
                }
                .my-enter,
                .my-leave-to{
                    opacity: 0;
                    transform: translatey(200px);
                }
                .my-enter-active,
                .my-leave-active{
                    transition: all 1s ease;
                }
                <input type="button" value="toggle" @click='flag=!flag'>
                <transition>
                    <h1 v-show='flag' v-cloak>H1H1H1H1</h1>
                </transition>
                 <hr style="height: 5px; background-color: aqua;">
                <input type="button" value="toggle2" @click='flag2=!flag2'>
                <transition name='my'>
                    <h1 v-show='flag2' v-cloak>H1H1H1H1</h1>
                </transition>
                列表过渡动画
                <ul>
                    <transition-group>
                        <li v-for='(item ,index) in list' :key='item.id'>{{item.id}}---{{item.name}}</li>
                    </transition-group>
                </ul>
```

14.2、使用动画函数实现动画效果：
        1、`before-enter` 入场动画开始前的状态（设置动画起始状态）。
        2、`enter` 入场动画结束后的状态（设置状态结束状态）。
        3、`after-enter` 入场动画结束后。
        4、`enter-cancelled`
        5、`before-leave` 离场动画开始前的状态（设置动画起始状态）。
        6、`leave` 离场动画结束后的状态（设置状态结束状态）。
        7、`after-leave` 离场动画结束后。
        8、`leave-cancelled`
        事例：   ```<input type="button" value="toggle" @click='flag=!flag'>
                <transition @before-enter='beforeEnter' @enter = 'enter' @after-enter='afterEnter'>
                <div v-show='flag' style="width: 15px; height: 15px; border-radius: 50%; background-color: red;"></div>
                </transition>
                 methods:{
                    beforeEnter(el){el.style.transform='translate(0 , 0)'},
                    enter(el,done){
                        el.offsetWidth
                        el.style.transform='translate( 150px, 400px)';
                        el.style.transition = 'all 1s ease'
                        done()
                    },
                    afterEnter(el){this.flag = !this.flag}
                }```
        注意：事件参数1 为当前 DOM 对象 参数2 为回调函数。enter 必须添加  el.offsetWidth 否则无法实现动画效果。
            enter leave 必须加上 参数2 表示引用 after-enter、after-leave。

15.Vue组件
    15.1、创建组件：语法：`var com1 = Vue.extend({template:'组件要展示的内容'}) ; Vue.component('组件名',组件模板对象)。`
    15.2、调用组件：<组件名称></组件名称>。
    定义全局组件：`Vue.component() `：第一个参数：组件的名称，将来使用组件时，以标签形式来引入。
                      第二个参数：`Vue.extend({template:''}) `创建的组件，其中 template 就是组件要展示的内容。
    定义局部组件： `var vm2 = new Vue({el:'#app2',components:{'mycom4':{template:'<h1>私有组件</h1>'}})。`
    注意：组件名使用驼峰命名法时，调用时必须用小写且在原大写字符前加" - "连接，不使用驼峰则直接使用组件名。template 中有且只有一个跟元素。
    事例： ``` <my-com1></my-com1>
            //创建全局组件(返回组件模板对象)
            var com1 = Vue.extend({
                template:'<h1>这是一个使用 Vue.extend 创建的组件</h1>'
            })
            //  Vue.component('组件名称', 组件模板对象)
            Vue.component('myCom1',com1)
            var vm = new Vue({
                el:'#app',
                data:{},
                methods:{}
            })```
    15.3、组件中的 `data` 和 `methods`：
        组件可以有data数据，组件中的data是一个方法，还必须返回对象，组件中data的使用方式根实例中data的使用方式相同。
        组件中 methods 用法与实例中相同。
        事例：```<div id="app">
                <mycom1></mycom1>
                <hr>
                <mycom1></mycom1>
            </div>
            <template id='tmp1'>
                <div>
                    <h1>{{count}}</h1>
                    <input type="button" value="+1" @click='addone()'>
                </div>
            </template>
                var vm = new  Vue({
                    el:'#app',
                    components:{
                        'mycom1':{
                            template:'#tmp1',
                            data:function(){return {msg:'这是组件中的data',count:0}},
                            methods:{addone(){this.count++}}
                        }
                    }
                })```
    15.4、组件切换：
        1、`v-if v-else` 实现切换：`<com1 v-if='flag'></com1><com2 v-else='flag'></com2>` flag为变量。
        2、使用 component 元素 实现切换：`<component :is='comName'></component>  `comName为组件名称。
        3、组件切换动画：` <transition mode='out-in'><component :is='comName'></component></transition>。`
        注意：`component` 元素是Vue提供的，相当于占位符，`:is `属性对应组件名称。 `transition` 中` mode='out-in' `熟悉表示动画先出后入。
    15.5、父子组件传值：
        默认子组件是无法访问到父组件上 `data` 的值和 `methods` 中的方法。
        父组件向子组件传值：给自定义组件添加属性-->在子组件中想要调用必须先定义在 props 数组中。
            事例： ```<com1 v-bind:prmsg='msg'></com1>
                var vm = new Vue({
                    el:'#app',
                    data:{msg:'父组件上的msg'},
                    components:{
                        com1:{
                            template:'<h1>子组件---{{prmsg}}</h1>',
                            props:['prmsg']
                        }
                    }
                })```
        子组件调用父组件方法：给自定义组件添加事件-->在子组件中调用使用 this.$emit 方法。
            事例：```<com1 v-bind:prmsg='msg' v-on:func='show'></com1>
                    methods:{
                        show(data,data2){console.log("调用了父组件的show方法"+data+data2)}
                    },
                    components:{
                        com1:{
                            template:'#tmp1',
                            props:['prmsg'],
                            methods:{
                                myClick(){this.$emit('func',123,456)}
                            }
                        }
                    }```
            注意：$emit方法 第一个参数为组件上所绑定的方法名，第二个及之后的参数是需要传递的参数值。传递参数就相当于子组件向父组件传值，父组件能够获取到子组件的值。
    15.6、ref的使用：
        Vue不提倡直接操作DOM元素，那么他提供了一个 `ref` 来获取元素。
        用法：在元素上绑定 `ref='名称'` 就能使用` this.$refs.名称 `来获取这个 DOM 对象。ref还能使用在组件上，能获取整个组件对象，可获取到组件所有数据及函数。

16.路由：
    16.1、创建路由： 引入 `vue-router.js `文件 --> `var routerObj = new VueRouter({routers:[{路由规则1},{路由规则2}]})`
    16.2、添加路由对象规则：`{path:'/login',component:login},{path:'/reg',component:reg}。`路由对象规则必须保函两个属性，属性1：path 路由地址。属性2：component 组件模板对象。component 的值必须是一个"组件模板对象"，不能是引用名。
    16.3、元素占位符 `<router-view></router-view> `VueRouter 提供的元素，用来展示根据路由规则匹配到的 component 组件。
    16.4、路由的使用： `<a href="#/login">登录</a><a href="#/reg">注册</a> `地址前面加 " # "。
    16.5、路由的执行顺序：点击链接url地址改变-->vue实例监听到改变-->根据规则匹配对应 component 组件。
    16.6、元素`<router-link></router-link>` VueRouter 提供的元素。
        1、to 属性为路由地址不需要加" # "，默认渲染为 a 标签， 可以通过 tag 属性渲染指定标签。
        2、链接选中高亮：a)修改 .router-link-active 类样式。 b)在创建实例时添加 linkActiveClass:'myactive' 属性修改类名，再修改类样式。
    16.7、设置根路由规则 {path:'/',redirect:'/login'} 让他重定向到指定路由规则，使页面渲染完成自动显示对应组件。
    16.8、路由传参：
        1、 `<router-link to='/login?id=10'>登录</router-link> `'?' 传参。使用 `this.$route.query` 获取到所有传入的值。
        2、 定义路由规则传参： `<router-link to='/reg/20/lisi'>注册</router-link> {path:'/reg/:id/:name',component:reg}`, 使用 `this.$route.params` 获取所传入的值。
    16.9、路由嵌套： `{path:'/account',component:account,children:[{path:'login',component:login},{path:'reg',component:reg}]} `使用路由规则中 children 属性。在组件定义时要给地址加上父组件的地址：` <router-link to='/account/login'>登录</router-link>`，并在组件中创建新`<router-view></router-view>。`
    注意：在子路由 path 前面不要带 '/'，否则永远以根路径去请求。
    事例：``` <a href="#/login">登录</a><router-link to="/reg" tag='span'>注册</router-link><router-view></router-view>
            var login = {template:'<h1>登录组件</h1>'}
            var reg = {template:'<h1>注册组件</h1>'}
            var routerObj = new VueRouter({
                routes:[
                    {path:'/login',component:login},
                    {path:'/reg',component:reg}
                ]
            })
            var vm = new Vue({ el:'#app',router: routerObj})```
   ` <router-view></router-view>` 元素:name属性指定哪个组件。
    `components` 路由规则中的属性：可以自定多个组件。默认指定default对应组件，当 outer-view 有name属性时 根据属性值指定组件。
    事例：``` {path:'/',components:{ //components 指定多个组件
                    'default':header,
                    'left':leftbox,
                    'main':mainbox
                }
          <router-view></router-view>
        <router-view name='left'></router-view>
        <router-view name='main'></router-view>```

17.watch监视属性 语法: 要监听的数据 : `function('新值','旧值'){}`
    1、监听 `data` 中的数据:
        用法：``` var vm = new Vue({
                el:'#app',
                data:{
                    fname:'',
                    lname:'',
                    allname:''
                },
                watch:{ //watch 属性会监听 data 中数据的改变
                    'fname':function(newVal, oldVal){ //当 fname 数据发生改变是会触发该方法
                        console.log(newVal+'------'+oldVal)
                        this.allname = newVal +'-' + this.lname
                    },
                    'lname':function(newVal, oldVal){ //当 fname 数据发生改变是会触发该方法
                        console.log(newVal+'------'+oldVal)
                        this.allname = this.fname+'-' +newVal
                    }
                }
            })```
    2、监听路由变化：`'$route.path':function(newVal,oldVal){console.log(newVal+'---'+oldVal)}`

18.computed计算属性
    创建： ```computed:{
                'allname':function(){
                    return this.fname + '-' + this.lname
                }
            } //在data 中取消 allname 属性```
    计算属性 监听需要计算的数据 如果数据改变则 触发该事件，在调用的当做属性 不要在最后加 '()'，计算属性会对求职的结果进行缓存，在之后调用时如果要计算的值未发生改变就不会调用这个方法。
    `methods`、`watch`、`computed`的比较：
        `methods`：方法表示一个具体的操作，主要书写业务逻辑。
        `watch`：一个对象，键是需要观察的表达式，值是对应的函数。主要用于监听某些特定数据的变化，从而进行某些具体的业务逻辑处理。
        `computed`：属性的结果会被缓存，除非依赖的值发生改变才会重新计算，主要当做属性来使用。

19.render函数
    语法：`var vm = new Vue({el:'#app',render:function(createElements){return createElements(模板对象名称)}})`
    `createElements` 是一个方法，调用它能够将指定 '模板对象' 渲染到 `el` 绑定的容器中，会清空 `el` 容器内所有元素。
