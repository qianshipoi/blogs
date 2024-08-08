---
title: Vue3 New Feature And Vite
published: 2022-06-13
description: ''
image: ''
tags: ['Vue', 'Vite']
category: 'Develop'
draft: false
---

## 升级vue-cli

```shell
npm i -g @vue/cli@next
```



## 使用vite初始化项目

```shell
1. npm init vite-app <project-name>
2. cd <project-name>
3. npm install
4. npm run dev
```

### 新特性

#### Composition API

```vue
setup函数 响应式数据创建声明
reactive({})方法 创建多个响应式对象
computed(() => {})方法 构建计算函数
onMounted(() => {}) onUnmounted(() => {})方法 生命周期钩子函数
ref()方法 1.创建一个响应式对象 2.初始化为null 如果文档存在元素同名ref引用则会赋值
toRefs()方法 处理reactive()方法生成的响应式对象 在接收时可以使用{key1, key2,...}展开
watch(属性,(val, oldVal) => {})方法 侦听器
```

> demo

```html
<template>
  <h1>{{ msg }}</h1>
  <p>{{ counter }}</p>
  <p>{{ doubleCounter }}</p>
  <p ref="desc"></p>
</template>

<script>
import { computed, reactive, onMounted, onUnmounted, ref, toRefs, watch } from 'vue'
export default {
  name: 'Composition',
  setup(){
    const { counter, doubleCounter } = useCounter()
    // 声明单个响应式数据
    const msg = ref("ref 单值声明响应式数据");
    // 初始化为空 如果文档存在同名ref引用则会设置为元素引用
    const desc = ref(null);

    // 侦听器
    watch(counter, (val, oldVal) => {
      desc.value.textContent = `${oldVal} to ${val}`;
    })
    return {counter, doubleCounter, msg, desc}
  },
}
// 封装成方法
function useCounter(){
  // 声明响应式数据
  const data = reactive({
      counter : 1,
      doubleCounter: computed(() => data.counter * 2)
    })

    let timer
    // 生命周期钩子
    onMounted(() => {
      timer = setInterval(() => {
        data.counter++
      },200)
    })
    onUnmounted(() =>{
     clearInterval(timer)
    });
    // toRefs 返回的数据可以使用{key1, key2,...}展开
    return toRefs(data);
}
</script>

```



### Teleport （传送门）

```vue
teleport 标签
to="body" 设置生成位置
```

demo

```html
<template>
    <div>
        <button @click="modalOpen = true">打开模态框</button>
        <teleport to="body">
            <div v-if="modalOpen" class="modal">
               <div>
                    生成在body的Modal弹框
                <button @click="modalOpen = false">关闭</button>
               </div>
            </div>
        </teleport>
    </div>
</template>

<script>
import { ref } from 'vue'
export default {
    name:"ModalButton",
    setup(){
        const modalOpen = ref(false)
        return {modalOpen}
    }
}
</script>

<style scoped>
.modal{
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, .2);
}
.modal > div{
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    width: 500px;
    height: 500px;
    padding: 20px;
    background-color: white;
}
</style>
```

#### custom render （自定义渲染器）

##### 导出自定义渲染器

```js
import { createApp, createRenderer } from 'vue' // 导入createRenderer模块
import CanvasApp from './CanvasApp.vue'  // 导入自定义组件
```

##### 实现自定义渲染器选项

```js
const nodeOps = {
    createElement(tag) {
        // 处理元素创建逻辑
        return { tag }
    },
    insert(child, parent, anchor) {
        // 处理元素插入逻辑
        // 如果是虚拟元素进入
        child.parent = parent;
        if (parent.childs) {
            parent.childs.push(child);
        } else {
            parent.childs = [child];
        }
        // 如果是真实dom元素，绘制canvas
        if (parent.nodeType == 1) {
            draw(child)
                // 事件处理
            if (child.onClick) {
                // canvas 上下文监听事件
                canvas.addEventListener('click', () => {
                    child.onClick()
                        // 重绘
                    setTimeout(() => {
                        draw(child)
                    }, 0)
                })
            }
        }
    },
    remove: child => {},
    createText: text => {},
    createComment: text => {},
    // 设置text
    setText: (node, text) => {},
    // 设置元素text
    setElementText: (el, text) => {},
    // 父节点
    parentNode: node => {},
    // 下一个兄弟节点
    nextSibling: node => {},
    // 获取元素
    querySelector: selector => {},
    setScopeId: (el, id) => {},
    cloneNode: (el) => {},
    insertStaticContent: (content, parent, anchor, isSVG) => {},
    // 属性更新 早于insert执行
    patchProp: (el, key, prevValue, nextValue) => {
        el[key] = nextValue
    }
}
```

##### 实现绘制方法

```js
let ctx, canvas
const draw = (el, noClear) => {
    if (!noClear) {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
    }
    if (el.tag == 'piechart') {
        // 获取元素上的属性和数据
        let { data, r, x, y } = el;
        let total = data.reduce((memo, current) => memo + current.count, 0)
        let start = 0,
            end = 0;
        data.forEach(item => {
            end += item.count / total * 360
            drawPieChart(start, end, item.color, x, y, r)
            drawPieChartText(item.name, (start + end) / 2, x, y, r)
            start = end
        });
    }
    el.childs && el.childs.forEach(child => draw(child, true))
}

const d2a = (n) => {
    return n * Math.PI / 180
}

const drawPieChart = (start, end, color, cx, cy, r) => {
    let x = cx + Math.cos(d2a(start)) * r
    let y = cy + Math.sin(d2a(start)) * r
    ctx.beginPath()
    ctx.moveTo(cx, cy)
    ctx.lineTo(x, y)
    ctx.arc(cx, cy, r, d2a(start), d2a(end), false)
    ctx.fillStyle = color
    ctx.fill()
    ctx.stroke()
    ctx.closePath()
}

const drawPieChartText = (val, position, cx, cy, r) => {
    ctx.beginPath()
    let x = cx + Math.cos(d2a(position)) * r / 1.25 - 20
    let y = cy + Math.sin(d2a(position)) * r / 1.25
    ctx.fillStyle = "#000"
    ctx.font = "20px 微软雅黑"
    ctx.fillText(val, x, y)
    ctx.closePath()
}

```

##### 创建渲染器

```js
const renderer = createRenderer(nodeOps)
```



##### 扩展mount挂载方法

```js
function createCanvasApp(App) {
    const app = renderer.createApp(App) // 创建app
    const mount = app.mount // 保存原始mount方法

    app.mount = function(selector) {
        // 创建画布
        canvas = document.createElement('canvas')
        ctx = canvas.getContext('2d')

        // 设置画布属性
        canvas.width = 500
        canvas.height = 500
        document.querySelector(selector).appendChild(canvas)

        // 执行默认mount
        mount(canvas)
    }
    return app
}
```

##### 调用方法渲染元素

```js
createCanvasApp(CanvasApp).mount("#renderer")
```

> CanvasApp.vue
>
> ```vue
> <template>
> <piechart @click="handleClick" :data="state.data" :x="200" :y="200" :r="200"></piechart>
> </template>
>
> <script>
> import { reactive } from 'vue'
> export default {
> name:'CanvasApp',
> setup(){
>   const state = reactive({
>       data:[
>           {name: "张三", count: 500, color:'skyblue'},
>           {name: "张三", count: 500, color:'pink'},
>           {name: "张三", count: 500, color:'brown'},
>       ]
>   })
>   return { state }
> },
> methods:{
>   handleClick(){
>       this.state.data.push({name:'其他',count: 200, color: 'green'})
>   }
> }
> }
> </script>
> ```

#### v-model 与 sync 合并

```vue
<VmodelTest v-model="state.counter" ></VmodelTest>

<template>
	<!--原insert改为update v-model传入的值为modelValue-->
    <div @click="$emit('update:modelValue', modelValue + 1)">
        counter: {{ modelValue }}
    </div>
</template>

<script>
    export default {
        name:'VmodelTest',
        props:{
            modelValue:{
                type:Number,
                default:0
            }
        }
    }
</script>

```

#### Render函数

1. 导入h函数
2. 创建组件ReanderTest，设置传入参数和所需方法
3. render()函数 返回h()方法 。h()方法 参1：标签元素，参2： 元素属性，参3 ：子元素
4. 获取组件插槽数据：this.$slots.插槽名() 获取插槽内容

```vue
  <RenderTest v-model:counter="state.counter">
    <template v-slot:default >title</template>
  </RenderTest>

import { reactive, h } from 'vue'
 RenderTest:{
      props:{
        counter:{
          type: Number,
          default:0
        }
      },
      render(){
        return h('div',[
          h('div',{ onClick: this.clickHandle }, `RenderTest--counter:${ this.counter }`),
          h('div',`插槽内容：${this.$slots.default()[0].children}`)
        ])
      },
      methods: {
        clickHandle() {
          this.$emit('update:counter', this.counter + 1)
        }
      }
```

#### 函数式组件

> 1. APP.vue
>
> ```vue
>   <Functional :level="2">动态组件</Functional>
>   <Functional :level="1">动态组件</Functional>
>   <Functional :level="4">动态组件</Functional>
> ```
>
> 2. Functional.vue
>
> ```vue
> <script>
> import { h } from "vue"
>
> function Heading(props, context){
>     console.log(props);
>     console.log(context);
>     return h(`h${props.level}`, context.attrs, context.slots)
> }
> // 声明属性
> Heading.props = ['level']
>
> export default Heading
> </script>
>
> ```

#### 异步组件

1. 定义异步组件

   ```js
   import { defineAsyncComponent } from 'vue'
   ```

2. 导入不带配置的组件

   ```js
   const asyncComp = defineAsyncComponent(() => import('path'))
   ```

3. 加载带配置的组件

   ```js
   const asyncComp = defineAsyncComponent({
   	loader: () => import('compPath'),
   	delay: 200,
   	timeout:3000,
   	errorComponent:"错误组件",
   	loadingComponent:"加载组件"
   })
   ```

#### 跳过自定义组件警告信息

1. vite

> vite.config.js
>
> ```js
> module.exports = {
> 	vueCompilerOptions: {
> 		isCustomElement: tag => tag === '组件元素名'
> 	}
> }
> ```

2. vue-cli

> vue.config.js
>
> ```js
> rules: [
> 	{
> 		test: /.vue$/,
> 		use: 'vue-loader',
> 		options:{
> 			compilerOptions: {
> 				isCustomElement: tag => tag === '组件元素名'
> 			}
> 		}
> 	}
> ]
> ```
