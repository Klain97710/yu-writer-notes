---
flag: blue
---
# Vue高级用法

## 自定义vue指令

```javascript
// ./src/directives/inputFilter/inputFilter.js
const addListener = function (el, type, fn) {
  el.addEventListener(type, fn, false)
}
const intFilter = function(el) {
  addListener(el, 'keyup', () => {
    el.value = el.value.replace(/\D/g, '')
  })
}

const priceFilter = function(el) {
  addListener(el, 'keyup', () => {
    el.value = el.value.replace(/[^\d.]*/g, '')
    if (isNaN(el.value)) {
      el.value = ''
    }
  })
}

const phoneFilter = function(el) {
  addListener(el, 'blur', () => {
    if (!el.value) {
      return
    }
    const phoneReg = new RegExp('^(13|14|15|16|17|18|19)[0-9]{9}$')
    if (!phoneReg.test(el.value)) {
      alert('手机号输入错误')
      el.value = ''
    }
  })
}

const urlFilter = function(el) {
  addListener(el, 'blur', () => {
    if (!el.value) {
      return
    }
    const urlReg = /(^#)|(^http(s*):\/\/[^\s]+\.[^\s]+)/
    if (!urlReg.test(el.value)) {
      alert('url输入错误')
      el.value = ''
    }
  })
}
const specialFilter = function (el) {
  addListener(el, 'input', () => {
    el.value = el.value.replace(/[\uD83C|\uD83D|\uD83E][\uDC00-\uDFFF][\u200D|\uFE0F]|[\uD83C|\uD83D|\uD83E][\uDC00-\uDFFF]|[0-9|*|#]\uFE0F\u20E3|[0-9|#]\u20E3|[\u203C-\u3299]\uFE0F\u200D|[\u203C-\u3299]\uFE0F|[\u2122-\u2B55]|\u303D|[\A9|\AE]\u3030|\uA9|\uAE|\u3030/ig, '')
  })
}
export default {
  bind(el, binding) {
    if (el.tagName.toLowerCase() !== 'input' || el.tagName.toLowerCase() !== 'textarea') {
      el = el.getElementsByTagName('input')[0] || el.getElementsByTagName('textarea')[0]
    }
    switch (binding.arg) {
      case 'int':
        intFilter(el)
        break
      case 'price':
        priceFilter(el)
        break
      case 'special':
        specialFilter(el)
        break
      case 'phone':
        phoneFilter(el)
        break
      case 'url':
        urlFilter(el)
        break
      default:
        break
    }
  }
}

```

```javascript
// ./src/directives/inputFilter/index.js
import inputFilter from './inputFilter'
const install = function (Vue) {
  Vue.directive('inputFilter', inputFilter)
}
if (window.Vue) {
  window.inputFilter = inputFilter
  Vue.use(install)
}
inputFilter.install = install
export default inputFilter
```

```javascript
// ./src/app.vue
import inputFilter from '@/directives/input-filter/index.js' // 引入
export default {
  directives: {
    inputFilter
  },
}
```

使用的时候：`v-input-filter:special`

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



