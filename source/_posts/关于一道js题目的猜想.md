---
title: 关于一道js题目的猜想
date: 2017-10-31 09:52:37
tags: JavaScript
categories: JavaScript
---
昨天在GitHub上无意逛到了[creeperyang的博客](https://github.com/creeperyang/blog),在他博客的[JavaScript问题集锦](https://github.com/creeperyang/blog/issues/2)，看到了这道题：
**问题：** 尝试解释下连等赋值的过程。下面的代码为什么是这样的输出？
```JavaScript
var a = {n: 1};  
var b = a;
a.x = a = {n: 2};  
console.log(a.x);// --> undefined  
console.log(b.x);// --> {n:2}
```
博主是这样解释的：
> 按照`es5`规范，题中连等赋值等价于
`a.x = (a = {n: 2});`，按优先获取左引用（`lref`），然后获取右引用（`rref`）的顺序，`a.x`和`a`中的`a`都指向了`{n: 1}`。至此，至关重要或者说最迷惑的一步明确。`(a = {n: 2})`执行完成后，变量`a`指向`{n: 2`}，并返回`{n: 2}`;接着执行`a.x = {n: 2}`，这里的`a`就是`b`（指向`{n: 1}`），所以`b.x`就指向了`{n: 2}`。

后面自己想了下有点解释不通，因为当`a`的指向更改的时候，`a.x`中的`a`指向也应该同时更改，那么`x`属性也应该是在新的对象上添加，为什么会说这里的`a`仍然就是`b`即指向的是`{n: 1}`,所以对原代码进行了改造：
```JavaScript
var a = {n: 1};  
var b = a;
function test () {
  return a = {n: 2};
}
a.x = test();
console.log(a.x);// --> undefined  
console.log(b.x);// --> {n:2}
```
运行结果与原结果相同。在使用`chrome`的`debugger`查看代码运行的时候，发现`chrome`在运行过程中，运行到`a.x = test()`这行代码的时候会停留在`a.x`再运行`test()`,联系到关于属性在原型链上的搜索，作出以下的解释：
~~`a.x = a = {n: 2}`在运行过程中浏览器会首先尝试读取属性x的值，即在`{n: 1}`上搜索x属性，在搜索不到的前提下又由于赋值的操作，会在`{n: 1}`上创建`x`属性并且`x`属性指向`{n: 2}`,由于`a`的指向发生变更,所以`a.x`为`undefined`,`b.x`为`{n: 2}`~~
js执行连等赋值语句之前，会取出变量的引用。`a.x = a = {n: 2}`在运行过程中，由于.运算符的优先级比=运算符优先级高，所以浏览器会先执行`a.x`的操作，也就是读取`a`的操作，此时a仍指向`{n: 1}`，执行结果为`undefined`，但是由于赋值操作，此时会给`{n: 1}`增加`x`属性这个值，所以`a.x`中的`a`仍然指向`{n: 1}`，且属性x的值为`a = {n: 2}`的执行结果。因为在执行`a = {n: 2}`的过程中改变了`a`的指向，所以`console.log(a.x)`其结果为`undefined`，b.x结果为`{n: 1}`。
