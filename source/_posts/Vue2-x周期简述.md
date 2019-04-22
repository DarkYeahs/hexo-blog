---
title: Vue2.x周期简述
date: 2017-10-28 23:06:05
tags: [笔记, Vue]
categories: 笔记
---
首先贴一张大图，跟vue1.x相比。vue2.0渲染周期发生了较大的变化。Vue的生命周期变为beforeCreate（Vue1.x为created），created（vue1.x为beforeCompile），beforeMount（vue1.x为compile），mounted（vue1.x为ready）以及beforeDestroy，destroyed.
![image](https://vuejs.org/images/lifecycle.png)

### beforeCreate周期
在beforeCreate周期前是获取不到任何数据的，这个周期只是进行事件的初始化以及生命周期的初始化，在完成事件的初始化以及生命周期的初始化后会触发beforeCreate的钩子函数，对于无关数据与渲染后的dom操作的可以在这个周期开始执行
### created周期
在created周期前，对数据通过vue本身改造Object.prototype.setter 和Object.prototype.getter两个函数对传入的数据进行监听，在这个周期可以对无关dom的操作进行处理，比如根据传入的数据获取后台数据等操作可以在这个周期进行
### beforeMount周期
在这个周期主要完成虚拟dom操作，Vue会去检测是否有没有el这个配置，如果没有，则会等待vm.$mounted(el)的调用。接着检测是否有没有template这个配置。如果有，这将这个传入render函数进行编译，如果没有，则将el元素内部的dom作为template参数传入render。对于这个周期如果不涉及到事件绑定方面的事件可以在这个周期进行操作
### mounted周期
这个周期是完成将构建的虚拟dom渲染到页面上的操作，这个时候可以对整个页面进行渲染完成后的各种动态操作
### beforeDestroy周期和destroyed周期
beforeDestroy周期和destroyed周期主要涉及到组件的!销毁,目前尚未涉及，等涉及了再补上
