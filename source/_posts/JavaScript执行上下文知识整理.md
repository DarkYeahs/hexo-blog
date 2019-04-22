---
title: JavaScript执行上下文知识整理
date: 2017-11-02 10:43:01
tags: [JavaScript, 笔记]
categories: 笔记
---
前几天在GitHub上看到了[mqyqingfeng](https://github.com/mqyqingfeng)的博客，在上面看了他的[JavaScript深入系列](https://github.com/mqyqingfeng/Blog/issues/17)中关于执行上下文部分的讲述，对于JavaScript的执行上下文有了进一步的了解,在这里总结下我新的认识。
### 执行上下文
首先说下什么是执行上下文，HTML规范上是这么说的
> When control is transferred to ECMAScript executable code, control is entering an execution context. Active execution contexts logically form a stack. The top execution context on this logical stack is the running execution context. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. The newly created execution context is pushed onto the stack and becomes the running execution context.

简单的来说就是当可执行代码被执行的时候，会创建一个执行上下文，这个上下文就是当前可执行代码的执行环境。并且多个执行上下文会构成一个执行上下文栈，栈底部始终存在着一个GlobalContext。其中可执行代码分为1. 全局代码;2. 函数代码;3. eval代码。

对于每个执行上下文，都有三个重要属性：
- 变量对象(Variable object，VO)
- 作用域链(Scope chain)
- this

### 变量对象
变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。其中，在函数上下文中，我们用活动对象(activation object, AO)来表示变量对象。

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的，不可在 JavaScript 环境中访问，只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 activation object，而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问。

活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

执行上下文的代码会分成两个阶段进行处理：分析和执行。
1. 分析
当进入执行上下文时，这时候只是进行代码的解析(变量命名的提升就在这一阶段)，
变量对象会包括：
  a) 函数的所有形参 (如果是函数上下文)
    + 由名称和对应值组成的一个变量对象的属性被创建
    + 没有实参，属性值设为 undefined

  b) 函数声明
    + 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
    + 如果变量对象已经存在相同名称的属性，则完全替换这个属性

  c) 变量声明
    + 由名称和对应值（undefined）组成一个变量对象的属性被创建；
    + 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

2. 执行
在执行阶段，浏览器会顺序执行代码，根据代码，修改变量对象的值

举个🌰：
```JavaScript
function foo (a) {
  var b = 1
  function bar () {
  }
  var d = function () {}
  b = 2
}
foo(1)
```
在分析阶段，这个时候的AO是：
```JavaScript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    bar: reference to function c(){},
    d: undefined
}
```
在执行阶段，这个时候的AO是：
```JavaScript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 2,
    bar: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```
### 作用域链
在上面说到每个上下文都有一个变量对象，而由多个上下文的变量对象构成的链表就叫做作用域链。
其中，函数的作用域链在函数定义的时候就决定了。在函数创建的时候，会把所有的父变量对象保存到函数的一个内部属性中。
举个🌰：
```JavaScript
function foo() {
    function bar() {
        ...
    }
}
```
函数创建时，各自的[[scope]]为：
```JavaScript
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,
    globalContext.VO
];
```
当函数激活时，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用链的前端。

这时候执行上下文的作用域链，我们命名为 Scope：

```JavaScript
Scope = [AO].concat([[Scope]]);
```
至此，作用域链创建完毕。

this再这里就先不展开了，单纯从ECMAScript的角度讲有点绕，也有点多。。。懒病又犯了。。。
