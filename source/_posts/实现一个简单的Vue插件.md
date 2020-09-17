---
title: 实现一个简单Vue插件
---
## 概念
最近看了篇关于Vue插件的文章，为了避免忘记，打算写下来，以遍日后查阅。

我们引入全局变量的时候，可能需要一个一个的引入，而且在一个vue文件中引用的组件多了，会显得代码臃肿，所以才有了封装vue插件的需求。


## 插件组件模板
```javascript 
<template>
    <transition name="fade">
        <div class="toast" v-show="show">
            {{message}}
        </div>

    </transition>
</template>

<script>
export default {
  data () {
    return {
      show: false,
      message: ''
    };
  }
}
</script>

<style lang="scss" scoped>
.toast {
  position: fixed;
  top: 40%;
  left: 50%;
  margin-left: -15vw;
  padding: 2vw;
  width: 30vw;
  font-size: 4vw;
  color: #fff;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.8);
  border-radius: 5vw;
  z-index: 999;
}

.fade-enter-active,
.fade-leave-active {
  transition: 0.3s ease-out;
}
.fade-enter {
  opacity: 0;
  transform: scale(1.2);
}
.fade-leave-to {
  opacity: 0;
  transform: scale(0.8);
}
</style>
```
这里面主要的内容只有两个，决定是否显示的show和显示什么内容的message，关键在于逻辑实现。
## 插件逻辑实现

 ```javascript 
import ToastComponent from './toast.vue'
const Toast = {}

// 注册toast
Toast.install = function (Vue) {
  const ToastContstructor = Vue.extend(ToastComponent)
  const instance = new ToastContstructor()
  instance.$mount(document.createElement('div'))
  document.body.appendChild(instance.$el)

  Vue.prototype.$toast = (msg, duration = 2000) => {
    instance.message = msg
    instance.show = true
    setTimeout(() => {
      instance.show = false
    }, duration)
  }
}
export default Toast
```

这里的逻辑大致可以分成这么几步：

创建一个空对象，这个对象就是日后要使用到的插件的名字。此外，这个对象中要有一个install的函数。
使用vue的extend方法创建一个插件的构造函数（可以看做创建了一个vue的子类），实例化该子类，之后的所有操作都可以通过这个子类完成。
之后再Vue的原型上添加一个共用的方法。


## 使用
之后再main.js中像我们之前一样使用普通的组件就行了
 ```javascript 
import Vue from 'vue'
import App from './App'
import router from './router'
import Toast from './components/toast'

Vue.use(Toast)

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

## 全局调用
 ```javascript 
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button @click="toast">显示toast弹出框</button>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods: {
    toast () {
      this.$toast('测试')
    }
  }
}
</script>
```
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
