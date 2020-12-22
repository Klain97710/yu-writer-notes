---
note: 总结VueRouter中的知识点
---

# Vue-Router

## 起步

`npm install vue-router`

```javascript
// src/router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)

import Home from '@/views/Home.vue'

const routes = [
  { path: '/home', component: Home }
]
const router = new VueRouter({
  routes
})
export default router

// src/main.js
// ...
import router from './router'

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```

## 动态路由匹配

### 响应路由参数的变化

当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar` ，原来的组件实例会被复用。这意味着组件的生命周期钩子不会再被调用。
复用组件时，想对路由参数的变化作出响应的时候，可以采取以下两种方法：

```javascript
export default {
  // 1.监测变化
  watch: {
    $route(to, from) {
      // 对路由变化作出响应
    }
  },
  // 2.组件内守卫
  beforeRouteUpdate(to, from, next) {
    // ...
    next();
  }
}
```

### 捕获所有路由——通配符

```javascript
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 '/user-' 开头的任意路径
  path: '/user-*'
}
```

含有通配符的路由应该放在最后。
当使用一个通配符时， `$route.params` 内会自动添加一个名为 `pathMatch` 参数：

```javascript
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

## 路由守卫

### 全局前置守卫

```javascript
const router = new VueRouter({ ... });

router.beforeEach((to, from, next) => {
  document.title = to.meta.title || '默认';
  if (to.meta.needLogin && !$store.state.isLogin) {
    next({
      path: '/login'
    });
  } else {
    next();
  }
})
```

参数解析：

- `to: Route`：即将要进入的目标路由对象
- `from: Route`：当前正要离开的路由对象
- `next: Function`：一定要调用该方法来resolve这个钩子

`next`方法接收不同的参数，会产生不同的效果：

- `next()`：进行管道中的下一个钩子，如果全部钩子执行完了，则导航的状态就是confirmed
- `next(false)`：中断当前的导航，如果浏览器的URL改变了（可能是用户手动或者浏览器后退按钮），那么URL地址会重置到`from`路由对应的地址
- `next('/')`或者`next({ path: '/' })`：跳转到一个不同的地址，当前的导航被中断，然后进行一个新的导航，可以向`next`传递任意位置对象，且允许设置诸如`replace:true`、`name:'home'`之类的选项以及任何用在`router-link`的`to`prop或`router.push`中的选项
- `next(error)`：如果传入的参数是一个Error实例，则导航会被终止且该错误会被传递给`router.onError()`注册过的回调

### 全局解析守卫

```javascript
router.beforeResolve((to, from, next) => {
  // ...
  next();
})
```

### 全局后置钩子

```javascript
router.afterEach((to, from) => {
  // ...
})
```

### 路由独享守卫

可以在路由配置上直接定义`beforeEnter`守卫：

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        //...
        next();
      }
    }
  ]
});
```

### 组件内的守卫

```javascript
export default {
  data() {
    return {
      msg: 'hello'
    }
  },
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不能获取组件实例this
    // 因为当守卫执行时，组件实例还没被创建
    // 不过，可以通过传一个回调给next来访问组件实例
    // 在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数
    next(vm => {
      // 通过 vm 访问组件实例
      alert( vm.msg );
    })
  },
  beforeRouteUpdate(to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id
    // 在 /foo/1 和 /foo/2 之间跳转的时候
    // 由于会渲染同样的 foo 组件，因此组件会被复用
    // 这个钩子就会在这个情况下被调用
    // 可以访问组件实例 this
    this.name = to.params.name;
    next();
  },
  beforeRouteLeave(to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问 this
    // 通常用来禁止用户在还未保存修改前突然离开
    // 可以通过 `next(false)` 来取消导航
    if (confirm("你确定要离开吗？") == true) {
      next();
    } else {
      next(false);
    }
  }
}
```

## 嵌套路由

```html
// app.vue
<div id="app">
  <router-view></router-view>
</div>
```

```javascript
// router.js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User}
  ]
});
```

`app.vue` 中的 `<router-view>` 是最顶层的出口，渲染最高级路由匹配到的组件。同样地，一个被渲染组件里面也可以包含自己的 `<router-view>`：

```javascript
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

```javascript
const router = new VueRouter({
  routes: [
   {
     path: '/user/:id',
     component: User,
     children: [
       {
         // 当 /user/:id/profile 匹配成功
         // UserProfile 会被渲染在 User 的 <router-view> 中
         path: 'profile',
         component: UserProfile
       },
       {
         // 当 /user/:id/posts 匹配成功
         // UserPosts 会被渲染在 User 的 <router-view> 中
         path: 'posts',
         component: UserPosts
       }
     ]
   }
  ]
});
```

**注意**
以 `/` 开头的嵌套路径会被当作根路径

## 编程式的导航

| 声明式 | 编程式 |
| --- | --- |
| `<router-link :to="...">` | `router.push(...)` |
| `<router-link :to="..." replace>` | `router.replace(...)` |

```javascript
// 不加'/'相对路径，加'/'根路径
// path
router.push('home');
router.push('home?id=1');
router.push({ path: 'home' });
router.push({
  path: 'home',
  query: {
    id: '1'
  }
});
// path 与 params 一起使用时，params 失效

// name
router.push({
  name: 'Home',
  params: {
    id: '1'
  }
});
router.push({
  name: 'Home',
  query: {
    id: '1'
  }
});

// history.forward();
router.go(1);
// history.back();
router.go(-1);
```

## 命名视图

有时候相同时(同级)展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` 和 `main` 两个视图，这个时候命名视图就排上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确地使用 `components` 配置：

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
});
```

### 嵌套命名视图

我们也有可能使用命名视图创建嵌套视图的复杂布局：

```html
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
```

- `UserSettings` 是一个视图组件
- `Nav` 是一个常规组件
- `UserEmailsSubscriptions`、`UserProfile`、`UserProfilePreview`是嵌套的视图组件

`UserSettings` 组件的 `<template>` 部分应该是类似下面的这段代码：

```html
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>
```

然后你可以用这个路由配置完成该布局：

```javascript
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [
    {
      path: 'emails',
      component: UserEmailsSubscriptions
    },
    {
      path: 'profile',
      components: {
        default: UserProfile,
        helper: UserProfilePreview
      }
    }
  ]
}
```

## 重定向和别名

### 重定向

用户访问 `/a` 时，URL将会被替换成 `/b`，然后匹配路由为 `/b`。

```javascript
const router = new VueRouter({
  routes: [
    // 以路径作目标
    {
      path: '/a',
      redirect: '/b'
    },
    // 以命名的路由作目标
    {
      path: '/c',
      redirect: {
        name: 'foo'
      }
    },
    // 以一个方法作目标
    {
      path: '/d',
      redirect: to => {
        // 目标路由 作为参数
        // return 重定向的 字符串路径/路径对象
      }
    }
  ]
});
```

### 别名

`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/a',
      component: A,
      alias: '/b'
    }
  ]
});
```

## 路由组件传参

在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的URL上使用，限制了其灵活性。

```javascript
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
});

// 通过 props 解耦
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },
    // 对于包含命名视图的路由，必须分别为每个命名视图添加 props 选项
    {
      path: '/user/:id',
      components: {
        default: User,
        sidebar: Sidebar
      },
      props: {
        default: true,
        sidebar: false
      }
    }
  ]
});
```

## 滚动行为

使用前端路由，当切换到新路由时，想要页面滚动到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。

**注意：**
这个功能只在支持 `history.pushState` 的浏览器中可用。

```javascript
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到的位置
  }
});
```

这个方法返回的滚动位置的对象信息：

- `{ x: number, y: number }`
- `{ selector: string, offset? : { x: number, y: number } }`

如果返回一个falsy的值或者是一个空对象，那么不会发生滚动。

## 路由懒加载

```javascript
const Foo = () => import('./Foo.vue');

const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo }
  ]
});
```

### 把组件按块分组

想把某个路由下的所有组件都打包在同个异步块(chunk)中，可以通过一个特殊的注释语法来提供chunk name。

```javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar= () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz= () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

Webpack会将任何一个异步模块与相同的块名称组合到相同的异步块中。
