---
title: redux初步认识以及与vuex的比较
date: 2019-05-07 16:01:53
tags:
categories:
---
整理下近期对redux的认识，
redux有四个概念
+ actions
+ actions创建函数
+ reducer
+ store

#### actions
actions的定义是
> Action 是把数据从应用（译者注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

因为先接触的是vuex的概念，redux的actions理解的是vuex是我acions的一部分，告诉redux的reducer要以何种方式去处理传递过去的数据。
#### actions创建函数
Action 创建函数 就是生成 action 的方法。在我看来actions创建函数与actions加起来等价于vuex中的actions。redux的actions创建函数可以被用于异步，与vuex中的异步类似，而redux的actions与vuex中的commit函数的参数类似。
#### reducer
关于reducer的解释文档是这么回答的
> Reducers 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 actions 只是描述了有事情发生了这一事实，并没有描述应用如何更新 state。

由传递的actions来决定如何去处理数据，处理完数据后交由store更新state。
vuex中更新数据直接由mutation完成，不需要返回新的state，直接在原来的state上进行修改。redux则是在新的state上修改，并且不修改原有的state。

#### store
个人理解就是一个集合的概念，与vuex的store类似。
