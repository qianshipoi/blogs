---
title: Vuex4 New Feature
published: 2022-06-13
description: ''
image: ''
tags: ['Vue', 'Vuex4']
category: 'Develop'
draft: false
---

#### 安装

```shell
npm i vuex@next
```

#### 创建实例

```js
// 导入实例创建方法
import { createStore } from 'vuex'

// 创建store实例
const store = createStore({
    state() {
        return {
            count: 1
        }
    },
    mutations: {
        addCount(state) {
            state.count++
        }
    }
})
// 挂载实例
createApp(App).use(router).use(store).component('EditTodo', EditTodo).mount('#app')
```

#### 页面使用

```html
<!-- 传统写法 -->
<span @click="$store.commit('addCount')">{{ $store.state.count }} </span>
<!-- composition 写法 -->
<span @click="addCount">{{ state.count }} </span>
```

```js
// 导入useStore
import { useStore } from "vuex";
// setup函数
setup(){
    const store = useStore()
    return {
        state: store.state,
        addCount(){
            store.commit("addCount")
        },
        // ...toRefs(store.state)
    }
}
```
