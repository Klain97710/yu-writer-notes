---
flag: blue
note: Vue提升性用法
---
# Vue的高级技巧

## watch属性用法

在开发Vue项目时，我们经常性的使用到watch去监听数据的变化，然后在变化之后做一系列操作。

### 基础用法

比如一个列表页，我们希望用户在搜索框输入搜索关键字的时候，可以自动触发搜索，此时除了监听搜索框的`change`事件之外，我们也可以通过`watch`监听搜索关键字的变化。

```html
<template>
  <div>
    <span>搜索</span>
    <input v-model="searchValue" />
  </div>
</template>
<script>
export default {
  data() {
    return {
      searchValue: '
    }
  },
  watch: {
    // 在值发生变化之后，重新加载数据
    searchValue(newValue, oldValue) {
      // 判断搜索
      if (newValue != oldValue) {
        this.$_loadData();
      }
    }
  },
  methods: {
    $_loadData() {
      // 重新加载数据，此处需要通过函数防抖
    }
  }
}
</script>
```

### 立即触发

通过上面的代码，现在已经可以在值发生变化的时候触发加载数据了，但是如果要在页面初始化的时候加载数据，我们还需要在`created`或者`mounted`生命周期钩子里面再次调用`$_loadData`方法。不过，现在可以不用这样写了，通过配置`watch`的立即触发属性，就可以满足需求了。

```javascript
export default {
  watch: {
    // 在值发生变化之后，重新加载数据
    searchValue: {
      // 通过handler来监听属性变化，初次调用newValue为""，oldValue为undefined
      handler(newValue, oldValue) {
        if (newValue != newValue) {
          this.$_loadData();
        }
      },
      // 配置立即执行属性
      immediate: true
    }
  }
}
```

### 深度监听

一个表单页面，需求希望用户在修改表单的任意一项后，表单页面就需要变更为已修改状态。如果按照上例中`watch`的写法，那么我们就需要去监听表单每一个属性，太麻烦了，这时候就需要用到`watch`的深度监听`deep`。

```javascript
export default {
  data() {
    return {
      formData: {
        name: '',
        sex: '',
        age: 0,
        deptId: '',
      }
    }
  },
  watch: {
    formData: {
      // 需要注意，由于对象引用的原因，newValue和oldValue一直相等
      handler(newValue, oldValue) {
        // 在这里标记页面编辑状态
      },
      // 通过指定deep属性为true，watch会监听对象里面每一个值的变化
      deep: true
    }
  }
}
```

## 自定义vue指令

当我们需要直接操作DOM时，就需要用到自定义指令了。
自定义指令和组件一样存在全局注册和局部注册两种方式。

### 全局注册

```javascript
// 示例 使用Vue.directive方法注册
Vue.directive("focus", {
  inserted: function(el) {
    el.focus();
  }
});

// 实例 main.js中使用Vue.use注册
import inputFocus from '@/directives/inputFocus.js'
Vue.use(inputFocus);

// 实例 @/directives/inputFocus.js
export default (Vue) => {
  Vue.directive("focus", {
    inserted: function(el) {
      el.focus();
    }
  })
}
```

### 局部注册

```html
<template>
  <div class="main>
    <input type="text" v-focus />
  </div>
</template>
<script>
  export default {
    directives: {
      focus: {
        inserted: function(el) {
          el.focus();
        }
      }
    }
  }
</script>
```

### 钩子函数

一个指令定义对象可以提供如下几个钩子函数：

- `bind`：只调用一次，指令第一次绑定到元素时调用，在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）。
- `update`：所在组件的 `VNode` 更新时调用，但是可能发生在其子 `VNode` 更新之前。指令的值可发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新。
- `componentUpdated`：指令所在组件的 `VNode` 及其子 `VNode` 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

### 钩子函数参数

指令钩子函数会被传入以下参数：

- `el`：指令所绑定的元素，可以用来直接操作 DOM。
- `binding`：一个对象，包含以下 property：
  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
- `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-%E6%8E%A5%E5%8F%A3) 来了解更多详情。
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

## 使用`$attrs`与`$listeners`二次包装组件

项目中大部分弹框右下角都是确定和取消两个按钮，如果使用elementUI提供的`Dialog`，那么每一个弹框都要手动加按钮，不但代码量增多，而且后面如果按钮UI需求发生变化，改动量也比较大。

```html
<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%
  :before-close="handleClose"
>
  <span>这是一段信息</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确定</el-button>
  </span>
</el-dialog>
```

如果可以将`Dialog`进行二次封装，将按钮封装到组件内部，就可以不用重复去写了。

定义基本弹框代码：

```html
<template>
  <el-dialog :visible.sync="dialogVisible">
    <!-- 内容区域的默认插槽 -->
    <slot></slot>
    <!-- 使用弹框的footer插槽添加按钮 -->
    <template #footer>
      <!-- 对外继续暴露footer插槽，有个别弹框按钮需要自定义 -->
      <slot name="footer">
        <!-- 将取消与确定按钮集成到内部 -->
        <span>
          <el-button @click="$_handleCancel">取消</el-button>
          <el-button type="primary" @click="$_handleConfirm">确定</el-button>
        </span>
      </slot>
    </template>
  </el-dialog>
</template>

<script>
export default {
  props: {
    // 对外暴露visible属性，用于显示隐藏弹框
    visible: {
      type: Boolean,
      default: false
    }
  }，
  computed: {
    // 通过计算属性，对.sync进行转换，外部也可以直接使用visible.sync
    dialogVisible: {
      get() {
        return this.visible;
      },
      set() {
        this.$emit("update:visible");
      }
    }
  },
  methods: {
    // 对外抛出cancel事件
    $_handleCancel() {
      this.$emit("cancel");
    },
    // 对外抛出confirm事件
    $_handleConfirm() {
      this.$emit("confirm");
    }
  },
};
</script>
```

通过上面的代码，我们已经将按钮封装到组价内部了，效果如下图所示：

```html
<!-- 外部使用方式 -->
<custom-dialog :visible.sync="dialogVisible">这是一段内容</custom-dialog>
```

但上面的代码存在一个问题，无法将`Dialog`自身的属性和事件暴露到外部（虽然可以通过`props`和`$emit`一个一个添加，但是很麻烦），这时候就可以使用`$attrs`与`$listeners`：

`$attrs`：当组件在调用时传入的属性没有在`props`里面定义时，传入的属性将被绑定到`$attrs`属性内（`class`和`style`除外，它们会挂载到组件最外层元素上）。并可通过`v-bind="$attrs"`传入到内部组件中。
`$listeners`：当组件被调用时，外部监听的这个组件的所有事件都可以通过`$listeners`获取到，并可通过`v-on="$listeners"`传入到内部组件中。

修改弹框代码：

```html
<template>
  <el-dialog
    :visible.sync="dialogVisible"
    v-bind="$attrs"
    v-on="listeners"
  >
    <!-- 其他代码不变 -->
  </el-dialog>
</template>

<script>
export default {
  //默认情况下父作用域的不被认作 props 的 attribute 绑定 (attribute bindings)
  //将会“回退”且作为普通的 HTML attribute 应用在子组件的根元素上。
  //通过设置 inheritAttrs 到 false，这些默认行为将会被去掉
  inheritAttrs: false
}
</script>

<!-- 外部使用方式 -->
<custom-dialog
  :visible.sync="dialogVisible"
  title="弹框标题"
  @opened="$_handleOpened"
>
  这是弹框内容
</custom-dialog>
```

## 通过`require.context`安装Vue组件

有关`require.context`具体内容在工程化初步接触中有说明。

```javascript
// directory=./ 扫描当前目录下面的所有文件
// useSubdirectories=false, 表示不需要扫描所有的子文件夹
// regExp=/\.vue$/ 所有以.vue结束的文件

const context = require.context('./', false, /\.vue$/);

//context.keys()返回所有匹配的文件的路径

context.keys().forEach(key => {
  // 通过context(key)可以获取到对应的文件 .default表示export default导出的内容
  component = context(key).default;
  // 安装vue组件
  Vue.component(component.name, component);
});

```

## 动态组件，灵活渲染页面

项目中遇到要根据用户权限的不同，页面展示不同的内容，直接写的话就通过`v-if`来判断了。

```html
<div class="info">
  <template v-if="role === 'admin'">
    <admin-info></admin-info>
  </template>
  <template v-else-if="role === 'bookkeeper'">
    <bookkeeper-info></bookkeeper-info>
  </template>
  <template v-else-if="role === 'hr'">
    <hr-info></hr-info>
  </template>
  <!-- 中间可能还有很多v-else-if -->
  <template v-else>
    <user-info></user-info>
  </template>
</div>
```

开始用动态组件优化。

```html
<template>
  <div class="info">
    <!-- 动态组件通过is来指定要渲染的组件 -->
    <component :is="roleComponent" v-if="roleComponent" />
  </div>
</template>

<script>
import AdminInfo from './admin-info'
import BookkeeperInfo from './bookkeeper-info'
import HrInfo from './hr-info'
import UserInfo from './user-info'
export default {
  components: {
    AdminInfo,
    BookkeeperInfo,
    HrInfo,
    UserInfo
  },
  data() {
    return {
      // 定义一个角色组件映射对象
      roleComponents: {
        admin: AdminInfo,
        bookkeeper: BookkeeperInfo,
        hr: HrInfo,
        user:UserInfo
      },
      // 当前用户角色
      role: 'user',
      // 当前要显示的组件
      roleComponent: undefined
    }
  },
  created() {
    // 通过角色与角色映射表判断要显示的组件
    const { role, roleComponents } = this;
    this.roleComponent = roleComponents[role];
  }
}
</script>
```

## 使用mixins高效复用组件内容

类型：`Array<Object>`

我们知道，每个Vue组件本身是一个Vue实例对象，诸如`data`、生命周期钩子等都是它的属性，`mixins`的作用就是把一个对象中的属性，`extend`到一个组件中，这个过程类似于我们使用`Object.assign`合并对象，但`mixins`使用的是和 `Vue.extend()` 一样的选项合并逻辑。

Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。

```javascript
// mixin.js
export const mixin = {
  created: function () {
    console.log(1)
  }
}

// vue
import { mixin } from '../utils/mixin.js'
export default {
  created: function () {
    console.log(2)
  },
  mixins: [mixin]
}
// => 1
// => 2
```

混入规则：

  1. 对于`data`，在混入时会进行递归合并，如果两个属性发生冲突，则以组件自身为主。
  2. 对于生命周期钩子，混入时会将同名钩子函数加入到一个数组中，然后在调用时依次执行，混入对象里面的钩子函数会优先于组建的钩子函数执行。如果一个组件混入了多个对象，对于混入对象里面的同名钩子函数，将按照数组顺序依次执行。
  3. 对于值为对象的选项，如`methods`、`components`、`filter`、`directives`、`props`等等，将被合并为同一个对象。两个对象的键名冲突时，取组件对象的键值对。

## 使用`hookEvent`监听组件生命周期

监听组件生命周期的三种方法：

  1. 在Vue选项中添加。
  2. 外部监听：在组件上绑定`@hook:created="handleCreated"`。
  3. 内部监听：使用`vm.$on('hook:created', () => {})`或`vm.$once('hook:created', () => {})` 。

一般开发中我们使用最多的就是第一种方法。

第二种方法的适用场景：
有一个来自第三方的复杂表格组件，表格进行数据更新的时候渲染时间需要1s，由于渲染时间较长，为了更好的用户体验，希望在表格进行更新时显示一个loading动画，而这个表格组件又没有提供`change`事件。
这个时候，很明显我们无法轻易地采用第一种方法来解决问题，但用第二种就可以解决了。
`<Table @hook:updated="handleTableUpdated"></Table>`

第三种方法的适用场景：
在`mounted`中添加一个监听需要在`beforeDestory`中移除这个监听，考虑到某些原因不想再写个`beforeDestory`，这时候就可以使用内部监听了。

```javascript
// 使用前
mounted() {
  this.chart = echarts.init(this.$el);
  // 请求数据，赋值数据 等等一系列操作...
  // 监听窗口发生变化，resize事件
  window.addEventListener('resize', this.$_handleResizeChart);
},
beforeDestory() {
  // 组件销毁时，销毁监听事件
  window.removeEventListener('resize', this.$_handleResizeChart);
},
methods: {
  $_hanldeResizeChart() {
    this.chart.resize();
  }
}

// 使用后
mounted() {
  this.chart = echarts.init(this.$el);
  // 请求数据，赋值数据 等等一系列操作...
  // 监听窗口发生变化，resize事件
  window.addEventListener('resize', this.$_handleResizeChart);
  // 通过hook监听组件销毁钩子函数，并取消监听事件
  this.$once('hook:beforeDestory', () => {
    window.removeEventListener('resize', this.$_handleResizeChart);
  });
},
```

## transition组件完成过渡效果

写个简单点的用法

```html
<transition name="custom-fade">
  <div>...</div>
</transition>

<style>
.custom-fade-enter-active, .custom-fade-leave-active {
  transition: opacity .5s;
}
.custom-fade-enter, .custom-fade-leave-to {
  opacity: 0;
}
</style>
```

一图看懂
![transition]($resource/transition.png)

配合animate.css的用法：

1. 安装`npm install animate.css --save`
2. main.js引入`import 'animate.css'`
3. 使用

```html
<transition
  mode="out-in"
  :duration="{enter: 700, leave: 700}"
  enter-active-class="animate__animated animate__zoomInRight"
  leave-active-class="animate__animated animate__zoomOutRight">
  <router-view/>
</transition>
```

## Vue.extend开发全局组件

Vue.extend是一个全局Api，平时我们在开发中很少会用到它，但有时候我们希望可以开发一些全局组件比如`Loading`、`Notify`、`Message`等组件时，这时候就可以使用Vue.extend。

在使用`element-ui`的`loading`时，在代码中可能会这样写：

```javascript
// 显示loading
const loading = this.$loading();

// 关闭loading
loading.close();
```

这样写可能没什么特别的，但是如果你这样写：

```javascript
const loading1 = this.$loading();
const loading2 = this.$loading();
setTimeout(() => {
  loading1.close();
}, 3000)
```

这时候你会发现，我调用了两次loading，但是只出现了一个，而且我只关闭了`loading1`，但是`loading2`也被关闭了。这是怎么实现的呢？

`Vue.extend` + 单例模式实现一个loading组件

### 开发loading组件

```html
<template>
  <transition name="custom-loading-fade">
    <!-- loading蒙板 -->
    <div v-show="visible" class="custom-loading-mask" @click="handleClickMask">
      <!-- loading中间的图标 -->
      <div class="custom-loading-spinner">
        <slot name="icon">
          <span class="custom-spinner-icon iconfont icon-dengdai"></span>
        </slot>
        <!-- loading上面显示的文字 -->
        <div class="custom-loading-text">{{text}}</div>
      </div>
    </div>
  </transition>
</template>
<script>
export default {
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    text: {
      type: String,
      default: ''
    }
  },
  mounted(){
    // document.body.style.position = 'fixed'
    // document.body.style.top = 0
  },
  methods: {
    handleClickMask() {
      this.$emit('update:visible', false);
    }
  }
}
</script>
<style>
@keyframes Rotate {
  0% {
    transform: rotateZ(0deg);
  }
  100% {
    transform: rotateZ(360deg);
  }
}
.custom-loading-mask {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  background: rgba(0, 0, 0, 0.7);
}
.custom-loading-spinner {
  width: 55%;
  /* height: 55%; */
  /* background: #fff; */
  position: fixed;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
  border-radius: 6px;
  color: #fff;
}
.custom-spinner-icon {
  display: inline-block;
  font-size: 30px !important;
  animation: Rotate 1.5s linear infinite;
}
</style>
```

使用的时候：

```html
<template>
  <div class="component-code">
    <!--其他一堆代码-->
    <custom-loading :visible="visible" text="加载中" />
  </div>
</template>
<script>
import CustomLoading from '@/components/CustomLoading.vue'
export default {
  components: {
    CustomLoading: CustomLoading
  },
  data() {
    return {
      visible: false
    }
  }
} </script>
```

但这样使用并不能满足我们的需求

1. 可以通过js直接调用方法来显示关闭。
2. `loading`可以将整个页面全部遮罩起来。

### 通过Vue.extend将组件转换为全局组件

第一步：改造loading组件，将组件的`props`改为`data`

```javascript
export default {
  data() {
    return {
      text: '',
      visible: false  
    }
  }
}
```

第二步：通过Vue.extend改造组件

```javascript
// loading/index.js
import Vue from 'vue'
import LoadingComponent from './CustomLoading.vue'

// 通过Vue.extend将组件包装成一个Vue子类
const LoadingConstructor = Vue.extend(LoadingComponent);

// 组件实例载体初始化
let loading = undefined;

// 定义loading组件实例创建方法
const Loading = (options = {}) => {
  //如果组件已经渲染，则返回当前实例
  if (loading) {
    return loading
  }
  // 要挂载的元素
  const parent = document.body;
  // 组件属性
  const opts = {
    text: '',
    ...options
  }
  // 通过构造函数初始化组件 相当于new Vue()
  const instance = new LoadingConstructor({
    el: document.createElement('div'),
    data: opts
  });
  // 将loading元素挂载到parent上面
  parent.appendChild(instance.$el);
  // 显示loading
  Vue.nextTick(() => {
    instance.visible = true;
  });
  // 将组件实例赋值给loading
  loading = instance;
  return instance
}

// 定义loading关闭方法
LoadingConstructor.prototype.close = function() {
  // 如果loading已经存在了，则去掉
  if (loading) {
    loading = undefined;
  }
  // 先将组件隐藏
  this.visible = false;
  // 延迟300ms，等待loading关闭动画结束再销毁组件
  setTimeout(() => {
    // 移除挂载的dom元素
    if (this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el);
    }
    // 调用组件的$destroy方法进行组件销毁
    this.$destroy();
  }, 300);
}

export default Loading
```

### 在页面使用loading

```javascript
import Loading from '@/components/loading/index.js'
export default {
  created() {
    const loading = Loading({ text: '正在加载...' });
    setTimeout(() => {
      loading.close();
    }, 3000);
  }
}
```

通过上面的改造，loading已经可以在全局使用了，如果需要像`element-ui`一样挂载到`Vue.prototype`上面，通过`this.$loading`调用，还需要改造一下。

### 将组件挂载到Vue.prototype上面

```javascript
// loading/index.js
Vue.prototype.$loading = Loading;
// 在export之前将Loading方法进行绑定
export default Loading

// 在组件内使用
this.$loading({ text: '正在查询...' });
```
