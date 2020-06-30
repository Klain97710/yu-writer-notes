---
note: 基于vue + vuex + vue-router + axios + less + elementUI
---
# Vue开发中的小技巧及注意事项

## 父子组件的生命周期钩子函数执行先后顺序

组件的生命周期钩子函数是到了某个生命周期节点就会触发，而不是在这个钩子中进行生命周期，比如说DOM加载好了，就会触发`mounted`钩子函数，所以在`created`里面写一个延迟定时器，`mounted`钩子不会等定时器执行。

关于父子组件的生命周期：不同的钩子函数有不同的表现。父组件的虚拟 DOM 先初始化好了（`beforeMount`），才会去初始化子组件的虚拟 DOM （`beforeMount`），而 `mounted` 事件，等价于 `window.onload`，子组件 DOM 没加载好，父组件 DOM 永远不可能加载好。所以基本生命周期钩子函数执行顺序是：父beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted

父子组件的update和beforeUpdate执行先后顺序：数据修改+虚拟DOM准备好会触发beforeUpdate，换句话说beforeUpdate等价于beforeMount，而update等价于mounted。所以先后顺序是： 父beforeUpdate -> 子beforeUpdate -> 子update -> 父update。

同理`beforeDestory`和`destoryed`的先后顺序是：父beforeDestory -> 子beforeDestory -> 子destoryed -> 父destoryed。

生命周期钩子函数其实也可以写成数组的形式：`mounted: [mounted1, mounted2]`,同一个生命周期可以触发多个函数，这也是`mixin`（混入）的原理，`mixin`里面也可以写生命周期钩子，最终会和组件里面的生命周期钩子函数一起变成数组形式，`mixin`里面的钩子函数会先执行

## 父组件监听子组件的生命周期

可以写自定义事件，然后在子组件的生命周期函数中触发这个自定义事件，但是不优雅，我们可以使用`hook`：

```html
<child @hook:created="childCreated"></child>
```

从A页面切换到B页面，A页面中有一个定时器，到了B页面用不上，需要在离开A页面的时候清除掉，办法很简单，在A页面的生命周期钩子函数`beforeDestory`或者路由钩子函数`beforeRouteLeave`里面清除掉就行，但是问题来了，怎么拿到定时器呢？把定时器写到`data`里面，可行但是不优雅，我们有如下写法：

```javascript
// 在初始化定时器后
this.$once('hook:beforeDestory', ()=>{
  clearInterval(timer);
});
```

## is的作用和用法

众所周知，ul里面嵌套li的写法是html语法的固定写法（还有如table、select等）。

```html
<ul>
  <my-component></my-component>
  <my-component></my-component>
</ul>
```

因此，如上写法，是无效甚至是会报错的。
而使用is就可以解决这种问题。

```html
<ul>
  <li is="my-component"></li>
</ul>
```

当然，首先你需要注册my-component组件。

is还有一种用法，配合`component`一起使用时，实现动态匹配切换组件。

```html
<component :is="组件名变量"></component>
```

## 给elementUI下拉菜单的command事件传额外参数

原生DOM事件绑定的函数的第一个参数都会是事件对象event，但是有时候我们想给这个函数传其他的参数，直接传会覆盖掉event，我们可以这么写`<div @click="clickDiv(params, $event)"></div>`，变量`$event`就代表事件对象。

如果要传的变量不是事件对象呢？在使用elementUI的时候碰到这么一个情况，在表格中使用了下拉菜单组件，代码如下：

```html
<el-table-column label="日期" width="180">
  <template v-slot="{row}">
    <el-dropdown @command="handleCommand">
      <span>
        下拉菜单<i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <template slot="dropdown"> //<template slot="dropdown">
        <el-dropdown-menu>
          <el-dropdown-item command="a">黄金糕</el-dropdown-item>
          <el-dropdown-item command="b">狮子头</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </template>
</el-table-column>
```

下拉菜单事件`command`函数自带一个参数，为下拉选中的值，这个时候我们想把表格数据传过去，如果`@command="handleCommand(row)"`这样写，就会覆盖掉自带的参数，该怎么办呢？这时候我们可以借助箭头函数：`@command="command => handleCommand(row,command)"`，完美解决传参问题。

顺便说一下，elementUI的表格可以用变量`$index`代表当前的列数，和`$event`一样的使用：

```html
<el-table-column label="操作">
  <template v-slot="{ row, $index }">
    <el-button @click="handleEdit($index, row)">编辑</el-button>
  </template>
</el-table-column>
```

当默认参数有多个（或者未知个数）的时候，可以这样写：`@current-change="(...defaultArgs) => treeClick(otherArgs, ...defaultArgs)"`。

## 深入理解sync修饰符

我们知道，`v-model`实现的双向绑定其实只是Vue的语法糖而已，他是通过两步来实现的：父组件用`v-bind`将值传给子组件，子组件通过`change/input`事件触发修改父组件的值。

```html
<input v-model="inputValue"/>
<!-- 等价于 -->
<input :value="inputValue" @change="inputValue = $event.target.value"/>
```

双向绑定不仅仅能在`input`上用，在组件上也能使用。
vue组件间传递数据是单向的，即数据总是由父组件传递给子组件，子组件在其内部可以有自己维护的数据，但它无权修改父组件传递给它的数据，我们也可以参照`v-model`语法糖，使用自定义事件触发修改父组件的值：

```html
// 子组件Child.vue
<template>
  <div>
    <span>{{value}}</span>
    <button @click="changeVal">修改</button>
  </div>
</template>
<script>
export default {
  props: {
    value: {
      type: [String, Number],
      required: true
    }
  },
  methods: {
    changeVal() {
      this.$emit("changeVal", '修改后的值456');
    }
  }
}
</script>
```

```html
// 父组件
<template>
  <div>
    <child :value="value" @changeVal="e => value = e"></child>
  </div>
</template>
<script>
import Child from '@/components/Child'
export default {
  components: {
    child: Child
  },
  data() {
    return {
      value: '123'
    }
  }
}
```

Vue推荐以`update:myPropName`的模式取代自定义事件。

```javascript
// 子组件
this.$emit('update:value', newValue);
```

```html
// 父组件
<child :value="value" @update:value="value = $event"></child>
```

方便起见，Vue提供了一个缩写，即`.sync`修饰符：

```html
<child :value.sync="value"></child>
```

如果一个组件的多个prop都要实现双向绑定，只需要放到一个对象里绑定就行了。

```html
<child v-bind.sync="obj"></child>
```

## 父组件通过ref访问到子组件

虽然vue提供了`$parent`和`$children`来访问父子组件，但是组件的父组件/子组件存在很多不确定性，例如组件被复用，他的父组件有很多情况。这时我们可以通过ref访问到子组件的数据和方法。

```html
<child ref="myChild"></child>
<script>
export default {
  async mounted() {
    await this.$nextTick();
    console.dir(this.$refs.myChild);
  }
}
```

注意：

* `ref`必须等DOM加载好了才可以访问
* 虽然`mounted`生命周期DOM已经加载好了，但以防万一，可以使用`$nextTick`函数

## 背景图、css的@import使用路径别名

在用`webpack`处理打包时，可为某一目录配置一个别名，代码中就能使用此别名引用资源。

```javascript
import tool from '@/utils/test';
```

但是在`css`文件，使用`@import "@/style/theme"`的语法引用相对`@`的目录却会报错。解决方法是在`@`前加`~`。

* css module中：`@import "~@/style/theme.less"`
* css属性中：`background: url("~@/assets/xxx.jpg")`
* html标签中：`<img src="~@/assets/xxx.jpg">`

## Vuex持久化存储

Vuex中的数据，刷新页面之后就会丢失。要实现持久化存储需要借助本地存储（cookie和storage），一般是登录之后返回的数据（角色，权限，token等）需要存储到Vuex，所以我们可以在登录页将数据存储到本地，而在主页面（除了登录页，其他所有页面的入口）进入之前（`beforeCreate`或者路由钩子`beforeRouteEnter`）读取出来，并提交到Vuex就好了。这样即使刷新，也会触发主页面的进入钩子函数，会被提交到Vuex。

```javascript
beforeRouteEnter (to, from, next) {
  const token = localStorage.getItem('token');
  let right = localStorage.getItem('right');
  try{
    right = JSON.parse(right);
  }catch{
    next(vm => {
      //弹窗采用elementUI
      vm.$alert('获取权限失败').finally(() => {
        vm.$router.repalce({name:'login'})
      })
    })
  }
  if(!right || !token){
    next({name:'login',replace:true})
  }else{
    next(vm => {
      //这里面的事件会在mounted之后触发
      vm.$store.commit('setToken',token);
      vm.$store.commit('setRight',right);
    })
  }
}
```

`beforeRouteEnter`的回调会在`mounted`钩子之后触发，这就比较蛋疼了。而主页面的`mounted`会在所有子组件的`mounted`之后触发，所以我们可以这样写。

```javascript
import store from '^/store';//将实例化的store引入进来

beforeRouteEnter (to, from, next) {
  const token = localStorage.getItem('token');
  if(!token){
    next({name:'login',replace:true})
  }else{
    store.commit('setToken',token);
    next();
  }
}
```

要想实现数据修改之后仍能持久化存储，我们可以先把数据存到`localstorage`，然后监听`window.onstorage`事件，数据有修改提交到`Vuex`。

## mutations里面触发action

`mutations`是同步修改`state`的值，假如另一个值是异步获取（`action`）的，依赖于这个同步的值的修改，需要在`mutations`里面赋值之前触发`action`里面的事件，我们可以给实例化的`Vuex`命名，在`mutations`里面拿到`store`对象。

```javascript
const store = new Vuex.Store({
  state: {
    age: 18,
    name: 'zhangsan',
  },
  mutations: {
    setAge(state, val) {
      // 假如age变化了之后，name也要跟着变化
      // 需要在每次给age赋值的时候，同步触发action里面的getName
      state.age = val;
      store.dispatch("getName);
    },
    setName(state, val) {
      state.name = val;
    },
  },
  actions: {
    getName({ commit }) {
      const name = fetch("name"); // 从接口异步获取
      commit("setName", name);
    }
  }
});
```

## Vue.observable进行组件通信

如果项目很小，不需要用到`Vuex`，可以用`Vue.observable`来模拟一个：

```javascript
// store.js
import Vue from 'vue';

const store = Vue.observable({
  name: '张三',
  age: 20
});
const mutations = {
  setAge(age) {
    store.age = age;
  },
  setName(name) {
    store.name = name;
  }
};

export { store, mutations };
```

## axios的qs插件

get请求的数据放在url里面，类似于`http://www.baidu.com?a=1&b=2`，其中`a=1&b=2`就是get的参数，而对于post请求，参数放到body里面，常用的数据格式有表单数据和json数据，两者的差异就是数据格式不同，表单数据编码格式和get一样，只不过是放在body里面，而json数据则是json字符串。

qs基本使用：

```javascript
import qs from 'qs'; // qs是axios里面自带的，直接引入
const data = qs.stringify({
  username: this.formData.username,
  oldPassword: this.formData.oldPassword,
  newPassword: this.formData.newPassword
});
this.$http.post('/changePassword.php', data);
```

`qs.parse()`是将URL解析成对象的形式，`qs.stringify()`是将对象序列化成URL的形式，以&进行拼接。而对于不同的数据格式，axios会自动设置对应的`content-type`，不需要手动设置。

* 表单数据（不带文件）的content-type是`application/x-www-form-urlencoded`
* 表单数据（带文件）的content-type是`multipart/form-data`
* json数据的content-type是`application/json`

## elementUI的一些总结

1.表单验证同步写法，避免多层嵌套函数

```javascript
const valid = await new Promise(resolve => this.$refs.form.validate(resolve));
if(!valid) return
```
