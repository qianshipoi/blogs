---
title: Vue Router4 New Feature
published: 2022-06-13
description: ''
image: ''
tags: ['Vue', 'VueRouter4']
category: 'Develop'
draft: false
---

#### 安装

```shell
npm i vue-router@next
```

#### 创建router实例

```js
// 导入vueRouter
import { createRouter, createWebHistory } from 'vue-router'

// 创建router实例
const router = createRouter({
    history: createWebHistory(),
    routes: [
        { path: '/', component: HelloWorld },
        { path: '/todos', component: Todos },
    ]
})
```

#### 挂载到vue实例

```js
createApp(App).use(router).mount('#app')
```

#### 新特性：动态添加单个路由

```js
// 添加根路由
router.addRoute({
        name: 'about',
        path: '/about',
        component: () =>
            import ('./components/About.vue')
    })
// 添加子路由
router.addRoute('about', {
    path: '/about/info',
    component: {
        render() {
            return h('div', "Info page")
        }
    }
})
```

#### Composition 使用router，route实例

```js
import { useRouter, useRoute } from "vue-router";

setup() {
    const state = reactive({
      timer: null,
    });
    const router = useRouter();
    const route = useRoute();
    onMounted(() => {
      state.timer = setInterval(() => {
        unref(router).push({
          path: "/about",
          query:{time: new Date().getTime() }
        });
      }, 1000);
    });
    onUnmounted(() => {
      clearInterval(state.timer);
    });
    // 监听路由参数变化
    watch(
      () => route.query,
      (query) => {
        console.log(query);
      }
    );
    // 路由守卫
    onBeforeRouteLeave((to,from) => {
        const answer = window.confirm("确认离开当前页面吗？");
        if(!answer){
            return false;
        }
    })
    return {
        router,
        route
    }
}
```

#### 自定义ViewLink

```js
<template>
  <div :class="{ active: isActive }" @click="navigate">{{ route.name }}</div>
</template>

<script>
import { RouterLink, useLink } from "vue-router";
export default {
  props: {
    ...RouterLink.props,
  },
  setup(props) {
    // 使用useLink获取路由信息
    const { route, href, isActive, isExactActive, navigate } = useLink(props);

    return { route, isActive, navigate };
  },
};
</script>

<style  scoped>
.active{
    background: chartreuse;
}
</style>

// 使用组件
<nav-link to="/"></nav-link>
<nav-link to="/todos"></nav-link>
<nav-link to="/about"></nav-link>
```

#### keep-alive标签 更换实现形式

```html
<router-view v-slot="{ Component }">
    <keep-alive>
      <component :is="Component" ></component>
    </keep-alive>
  </router-view>
```

#### router-link标签 tag/event 更换实现形式

> befor
>
> ```html
> <router-link to="/xx" tag="span" event="dbclick" ></router-link>
> ```
>
> now
>
> ```html
> <router-link to="/about" custom v-slot="{ navigate }">
> <span @dblclick="navigate">span => About</span>
> </router-link>
> ```

#### mixins中的路由守卫将被忽略

#### match方法移除，使用resolve替代

#### 移除router.getMatchedComponents()

```js
// 新方式获取
router.currentRoute.value
```

#### 包括首屏导航在内所有导航均为异步

```js
// 实现首屏前的动画
app.use(router)
router.isReady().then(() => app.mount('#app'))
```

#### route的parent属性被移除

```js
// 新方式获取parent
const parent = this.$route.metched[this.$route.metched.length - 2]
```

#### pathToRegexpOptions选项被移除

* pathToRegexpOptions => strict
* caseSensitive => sensitive

```js
createRouter({
	strict: Boolean,
    sensitive: Boolean
})
```

#### 使用history.state

```js
// befor
history.pushState(myState, '', url)
// now
router.push(url)
// or
history.replaceState({...history.state, ...myState}, '')
```

```js
// befor
history.replaceState({}, '', url)
// now
histoty.replaceState(history.state, '', url)
```

#### routes选项必填

```js
// 可以填空数组
createRouter({routes:[]})
```

#### 跳转不存在命名路由会抛出异常

```js
try{
    router.push({name:'abouts'})
}catch{

}
```

#### 缺少必填项参数会抛出异常

#### 命名子路由如果path为空的时候不再追加/

#### $route属性编码行为

> params/query/hash
>
> * path和fullpath 不再做解码
> * hash 会被解码
> * push、resolve和replace，字符串参数，或者对象参数path属性必须编码
> * params中"/"会被解码
> * query中"+"不处理，可以使用stringifyQuery处理
