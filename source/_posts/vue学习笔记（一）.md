---
title: vue学习笔记（一）
date: 2019-01-23 18:41:34
tags:
- 学习笔记
- Vue
categories: 
- 前端
---

## 从构造函数说起

构造vue对象是一切的开始。
```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

由此我们可以知道，Vue其实是个构造函数，它接受一个对象作为参数。构造函数拿到参数对象后进一步对它做出处理，并把最终处理得到的结果挂到本实例的$option属性下(即this.$option)。在完成了对配置参数的处理后，将会初始化出生命周期（lifecycle），事件系统(events)，初始状态等功能(state)，在initstate初始化状态之前，生命周期钩子beforeCreate会被回调执行，在初始化状态完成后，created会被调用，这也解释了为什么created的时候无法访问和操作DOM，因为此时仅仅是数据的初始化完成了，还没有真实的DOM元素。这么些初始化的过程实际上就是不断调用函数给vue实例的prototype上添加属性和方法，并在合适的时机按顺序调用。

在initstate中还会调用其他的init方法，这些其他的init方法包括对props、method、data、computed、watch等实例属性的初始化。要注意的是，在created之后，会初始化渲染过程，如果vm.$option.el存在，会调用$mount方法挂载DOM节点，如果不传el参数那就需要手动mount。

## 怎样渲染出一个DOM

在上面的例子里，option中给出了el配置项，因此会直接调用monut方法。mount方法是存在在vm.prototype上的一个函数，它把获得的el参数转换成相应的DOM节点，在vm上添加$el属性（vm.$el）指向挂载点DOM，触发beforeMount生命周期。如果option里不包含render选项，monut函数将会尝试将template或者el的outerHTML当成模版，转换成render函数，并把编译好的render函数挂到vm.$option中，调用render返回的是一个虚拟DOM，这个生成render函数的过程相当复杂，包括了生成抽象语法树再将抽象语法树转换成render函数等。如果是第一次挂载DOM将会触发mounted生命周期。

monut函数执行过程中还会添加一个watcher监视器，监视器会对render求值，render又会收集依赖变量，并在依赖变量发生变化时重新求值，从而触发re-render得到一个新的虚拟DOM，beforeUpdate和updated生命周期就发生在这个过程始末。

这个时候再看官网上的生命周期图解，会明了很多。

![lifecycle](https://cn.vuejs.org/images/lifecycle.png)

## 所谓双向数据绑定



前面说到，在初始化vue实例的时候，会进行initstate，在initstate中又会进行initdata，对数据进行初始化就发生在initdata过程中。

这个初始化的过程会先为data属性中的数据做一层代理，这样data.message就变成了this.message，然后创建observer观察数据变化，这个过程是递归的，否则只有直接子属性会被观察器观察到。通过Object.defineProperty，将data的属性转换为访问器属性，当data被中的属性被访问时触发get方法，在属性被更改时则触发set方法，藉此就可以让数据的订阅者得到通知。

vue的思路是在get中收集依赖，在set中触发依赖。每个data中的属性都有一个唯一的Dep对象，这个对象在调用get方法时收集当前属性的依赖，在调用set方法时触发收集到的所有依赖，这样就完成了一个发布订阅模式，最终的结果就是，在对数据做访问时会触发get方法，get中会给Dep新增一个订阅者（即收集了一个依赖），data中的属性发生变化时会触发set方法，循环挨个调用保存在Dep中的回调函数，通知所有订阅者做出更新。

由此可以看出，

- 只有保存在data中的对象才会被双向数据绑定，其他地方定义的变量没有机会被vue绑定。
- 绑定是递归的，和对象的层级无关，每个属性都会被绑定到。
- 绑定是针对对象设计的，数组不能得到很好的支持。

关于对数组的双向绑定，vue拦截了数组的splice,push,pop,shift,unshift等方法，使用这些方法将触发视图更新，而某个数组元素的变更不能引起视图自动更新，需要使用vm.$set方法通知vue实例做出更新。