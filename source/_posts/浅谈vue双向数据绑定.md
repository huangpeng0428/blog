---
title: 浅谈vue双向数据绑定
---
## 前沿
最近在研究vue的源码，再次记录一下
众所周知vue通过**数据劫持+发布订阅模式**实现双向绑定
那么我们就一步一步来实现一个简易版本的mvvm吧
#### 入口
```javascript
class Vue {
    constructor(options) {
        this.$el = options.el
        this.$data = options.data
        if (this.$el) {
            new Observer(this.$data)
            new Compiler(this.$el, this)
        }
    }
}

// 调用
new Vue({
    el: 'app'
    data: {
        someStr: 'hello ',
        school: {
            name: 'test',
            age: 10
        }
    },
})
```
这里我们定义了一个`Vue`的class,通过`new Vue`将我们的数据传入其中，执行数据观测方法`Observer`和模板解析方法`Compiler` 我们先看`Observer`
## Observer
```javascript
class Observer {
  constructor(data) {
    this.observe(data)
  }
  observe(data) {
    if (data && typeof data == 'object') {
      Object.keys(data).forEach(key => {
        this.defineReactive(data, key, data[key])
      })
    }
  }
  defineReactive(obj, key, value) {
      this.observe(value)
      let dep = new Dep()
      Object.defineProperty(obj, key, {
          get() {
              console.log(3333)

              console.log('Dep.target', Dep.target)
              Dep.target && dep.addSub(Dep.target)
              return value
          },
           set: (newVal) => {
            if (newVal !== value) {
                console.log(4444)
                this.observe(newVal)
                value = newVal
                dep.notify()
            }
           }
      })
  }
}
```
observe方法主要是遍历整个对象，通过`Object.defineProperty`给`data`的每个属性都加上对应的`get`和`set`，如果对象的属性依然是对象，会进行递归调用，给属性加上`get`和`set`
#### Compiler
```javascript
class Compiler {
    constructor(el, vm) {

        this.el = this.isElementNode(el) ? el : document.querySelector(el)

        // 把当前节点中的元素放到内存中
        let fragment = this.node2fragment(this.el)

        this.vm = vm

        // console.log(fragment)

        // 编译模板 用数据编译
        this.compile(fragment)

        // console.log(this.el)
        this.el.appendChild(fragment)
    }

    isDirective(attrName) {
        return attrName.indexOf('v-') !== -1
    }

    // 编译元素的方法
    compileElement(node) {
        let attributes = node.attributes; // 类数组
        [...attributes].forEach(attr => {
            let {name, value: expr} = attr
            if (this.isDirective(name)) {

                // console.log(name.split('-'))
                let [, directive] = name.split('-')
                CompileUtil[directive](node, expr, this.vm)
            }
        })
    }

    // 编译文本 找{{xxx}}
    compileText(node) {
        let content = node.textContent
        if (/\{\{(.+?)\}\}/g.test(content)) {

        CompileUtil['text'](node, content, this.vm)
        }
    }

    // 核心编译方法
    compile(node) {
        let childNodes = node.childNodes;

        // console.log([...childNodes]);
        [...childNodes].forEach(child => {
            if (this.isElementNode(child)) {

                // console.log('element', child)
                this.compileElement(child)

                // 如果是元素的话，需要除去自己，再去遍历子节点
                this.compile(child)
            } else {

                // console.log('text', child)
                this.compileText(child)
            }
        })
    }
    node2fragment(node) {
        let fragment = document.createDocumentFragment()
        let firstChild
        // eslint-disable-next-line no-cond-assign
        while (firstChild = node.firstChild) {

            // appendChild具有移动性
            // console.log(firstChild)
            fragment.appendChild(firstChild)
        }
        return fragment
    }
    isElementNode(node) {
        return node.nodeType === 1          // 元素
    }
}

//CompileUtil 包含编译表达式和指令的方法
const CompileUtil = {

  // 根据表达式取对应的数据
  getVal(vm, expr) {
    return expr.split('.').reduce((data, current) => {
      return data[current]
    }, vm.$data)
  },

  // 根据表达式设置对应的数据
  setVal(vm, expr, value) {
    expr.split('.').reduce((data, current, index, arr) => {
      if (index === arr.length - 1) {
        data[current] = value

        //  console.log(data[current])
        return data[current]
      }
      return data[current]
    }, vm.$data)
  },

  getContentVal(vm, expr) {
    return expr.replace(/\{\{(.+?)\}\}/g, (...args) => {
      return this.getVal(vm, args[1])
    })
  },

  // 解析v-model指令
  model(node, expr, vm) {
    let fn = this.updater['modelUpdater']
    new Watcher(vm, expr, newVal => {  // 给输入框加一个观察者，稍后数据更新，触发此方法，用新值给输入框赋值
      fn(node, newVal)
    })
    node.addEventListener('input', e => {
      let value = e.target.value
      this.setVal(vm, expr, value)
    })
    let value = this.getVal(vm, expr)
    console.log(value)
    fn(node, value)
  },

  html() {},

  text(node, expr, vm) {
    let fn = this.updater['textUpdater']
    console.log(expr)
    let content = expr.replace(/\{\{(.*?)\}\}/g, (...args) => {
      new Watcher(vm, args[1], () => {
        console.log(this.getContentVal(vm, expr))
        fn(node, this.getContentVal(vm, expr))
      })
      return this.getVal(vm, args[1])
    })
    fn(node, content)
  },

  updater: {
    modelUpdater(node, value) {
      node.value = value
    },
    textUpdater(node, value) {
      node.textContent = value
    },
    htmlUpdater(node, value) {
      node.innerHTML = value
    }
  }
}
```
`Compiler`这个类比较复杂，不过其主要作用是解析模板，包括对`{{xxx}}`和`v-model`，`v-text`等vue表达式的编译，我这里只是简易版，源码比这个要复杂的多

编译`文本{{xxx}}`和`v-model`等指令的时候，会进行一个`new Watcher`的操作，这里设计的非常巧妙，完美的运用了`js`单线程的特点。也是这里，将我们的数据和视图做到关联，实现双向数据绑定
```javascript
class Watcher {
    constructor(vm, expr, cb) {
        this.vm = vm
        this.expr = expr
        this.cb = cb

        this.oldValue = this.get()
    }
    get() {
        Dep.target = this
        console.log(2222)
        let value = CompileUtil.getVal(this.vm, this.expr)
        Dep.target = null
        return value
    }
    update() {
        let newVal = CompileUtil.getVal(this.vm, this.expr)
        if (newVal !== this.oldValue) {
            this.cb(newVal)
        }
    }
}
```
可以看到wacher中会执行一个`get`方法去获取数据，在获取数据之前会将`实例watcher`赋值给`Dep.target`，取完值之后又将`Dep.target`置为null


这里还要看一下之前的数据劫持：
```javascript
defineReactive(obj, key, value) {
    this.observe(value)
    let dep = new Dep()
    Object.defineProperty(obj, key, {
        get() {
            Dep.target && dep.addSub(Dep.target)
            return value
        },
        set: (newVal) => {
            if (newVal !== value) {
                console.log(4444)
                this.observe(newVal)
                value = newVal
                dep.notify()
            }
        }
    })
}
```
可以看到每条数据都有一个对应的dep，get数据时，如果有`Dep.target`就将`Dep.target` push进dep数组，也就是将watcher添加进dep。set数据时，会执行`dep.notify()`执行对应的更新操作

#### Dep
```javascript
    constructor() {
        this.subs = []      // 存储watcher
    }

    // 订阅 将属性的watcher添加进订阅器
    addSub(watcher) {
        this.subs.push(watcher)
    }

    // 发布 执行watcher中的update方法，更新视图
    notify() {
        this.subs.forEach(watcher => watcher.update())
    }
}
```

#### 总结

1. 首先对数据进行observer数据劫持，利用Object.defineProperty给每个属性加上get(),set()
2. 之后会进行数据的解析，解析数据时(比如v-model时)给每条数据加上watcher，并将watcher对象赋值给Dep.target（发布订阅器target属性）数据解析渲染时触发get()，将watcher添加订阅
3. 当修改数据时，会触发set(), 此时会触发我们在set()中定义的dep.notify方法，循环更新watcher updata方法，形成数据驱动视图，mvvm模式
