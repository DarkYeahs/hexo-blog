---
title: event loop学习回顾
date: 2017-10-27 10:43:01
tags: [JavaScript, 笔记]
categories: 笔记
---
以前事件队列在我的认知范围内是只有两种：同步事件队列与异步事件队列。在处理事件队列过程中，浏览器会定期检测同步队列并从同步事件队列获取事件，并将之执行，在这过程中，异步队列触发回调事件时，会将其从异步队列中出队，加入到同步事件队列中，一直重复上述过程。

但在这里有个疑问，当多个异步事件队列同时到同步队列过程中是以什么标准去实现的？

这个疑问直到前段时间看Vue有关的博客中无意间看到Chunk liu的[Vue源码详解之nextTick：MutationObserver只是浮云，microtask才是核心](https://chuckliu.me/#!/posts/58bd08a2b5187d2fb51c04f9)才了解到microTask的概念，才知道task的区分不是我原先理解的那么一回事。后面又读了Jake Archibald 写的[Tasks， microtasks， queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)才知道event loop到底是个怎么回事。

先说下task跟microTask的概念

#### task

task 又称 macrotask。HTML 规范中 task 的有关章节是这么说的：
>一个 eventloop 有一或多个 task 队列。每个 task 由一个确定的 task 源提供。从不同 task 源而来的 task 可能会放到不同的 task 队列中。例如，浏览器可能单独为鼠标键盘事件维护一个 task 队列，所有其他 task 都放到另一个 task 队列。通过区分 task 队列的优先级，使高优先级的 task 优先执行，保证更好的交互体验。

task源包括：
+ DOM 操作任务源：如元素以非阻塞方式插入文档
+ 用户交互任务源：如鼠标键盘事件。用户输入事件（如 click） 必须使用 task 队列
+ 网络任务源：如 XHR 回调
+ history 回溯任务源：使用 history.back() 或者类似 API
此外 setTimeout、setInterval、IndexDB 数据库操作等也是任务源。

#### microTask
microTask：规范上是这么说的：每个event loop都有一个microTask， microTask存在于microTask队列中。
microTask主要分以下几种：
+ Promise.then
+ MutationObserver
+ Object.observe
其中Promise.then由于不同浏览器的实现有所不同，有些运行触发的是microTask，有些触发的触发的是task，不过标准上是触发microTask的

#### event loop的循环过程
在说了task跟microTask的概念后再说下event loop的循环过程。先贴一下规范的说法(有点凑字数的嫌疑。。。)：
>An event loop must continually run through the following steps for as long as it exists:

>1. Select the oldest task on one of the event loop’s task queues, if any, ignoring, in the case of a browsing context event loop, tasks whose associated Documents are not fully active. The user agent may pick any task queue. If there is no task to select, then jump to the microtasks step below.
2. Set the event loop’s currently running task to the task selected in the previous step.
3. Run: Run the selected task.
4. Set the event loop’s currently running task back to null.
5. Remove the task that was run in the run step above from its task queue.
6. Microtasks: Perform a microtask checkpoint. //这里会执行所有的microtask
7. Update the rendering: If this event loop is a browsing context event loop (as opposed to a worker event loop), then run the following substeps.
7.1 Let now be the value that would be returned by the Performance object’s now() method.
7.2 Let docs be the list of Document objects associated with the event loop in question, sorted arbitrarily except that the following conditions must be met:
7.3 If there are top-level browsing contexts B that the user agent believes would not benefit from having their rendering updated at this time, then remove from docs all Document objects whose browsing context’s top-level browsing context is in B.
7.4 If there are a nested browsing contexts B that the user agent believes would not benefit from having their rendering updated at this time, then remove from docs all Document objects whose browsing context is in B.
7.5 For each fully active Document in docs, run the resize steps for that Document, passing in now as the timestamp. [CSSOMVIEW]
7.6 For each fully active Document in docs, run the scroll steps for that Document, passing in now as the timestamp. [CSSOMVIEW]
7.7 For each fully active Document in docs, evaluate media queries and report changes for that Document, passing in now as the timestamp. [CSSOMVIEW]
7.8 For each fully active Document in docs, run CSS animations and send events for that Document, passing in now as the timestamp. [CSSANIMATIONS]
7.9 For each fully active Document in docs, run the fullscreen rendering steps for that Document, passing in now as the timestamp. [FULLSCREEN]
7.10 For each fully active Document in docs, run the animation frame callbacks for that Document, passing in now as the timestamp.
7.11 For each fully active Document in docs, run the update intersection observations steps for that Document, passing in now as the timestamp. [INTERSECTIONOBSERVER]
7.12 For each fully active Document in docs, update the rendering or user interface of that Document and its browsing context to reflect the current state.
If this is a worker event loop (i.e. one running for a WorkerGlobalScope), but there are no tasks in the event loop’s task queues and the WorkerGlobalScope object’s closing flag is true, then destroy the event loop, aborting these steps, resuming the run a worker steps described in the Web workers section below.
Return to the first step of the event loop.

复述一下上面说的
>在前5的步骤是找出存在task队列中时间最长的task，出队并将之执行。
在第六步的时候将microTask队列中的microTask全部执行
第七步，更新渲染：
7.2到7.4，当前轮次的event loop中关联到的document对象会保持某些特定顺序，这些document对象都会执行需要执行UI render的，但是并不是所有关联到的document都需要更新UI(换个说法就是只有从UI Render收益的documnet才需要更新)，浏览器会判断这个document是否会从UI Render中获益，因为浏览器只需要保持60Hz的刷新率即可，而每轮event loop都是非常快的，所以没必要每个document都Render UI。
7.5和7.6 run the resize steps/run the scroll steps不是说去执行resize和scroll。每次我们scoll的时候视口或者dom就已经立即scroll了，并把document或者dom加入到 pending scroll event targets中，而run the scroll steps具体做的则是遍历这些target，在target上触发scroll事件。run the resize steps也是相似的，这个步骤是触发resize事件。
7.8和7.9 后续的media query, run CSS animations and send events等等也是相似的，都是触发事件，第10步和第11步则是执行我们熟悉的requestAnimationFrame回调和IntersectionObserver回调（第十步还是挺关键的,raf就是在这执行的！）。
7.12 渲染UI，关键就在这了。
第九步 继续执行event loop，又去执行task，microtasks和UI render。

个人理解是在event loop过程中，会循环检测各个task队列中的task，获取停留时间最久的task，出队并将之执行。在执行完后，会去检测microTask队列并执行在这个task过程中生成的microTask以及micorTask执行过程中生成microTask，在执行完成后进行页面的更新操作。
这里举个🌰：
```JavaScript
console.log('script start')

setTimeout(() => console.log('setTimeout'), 0)

Promise.resolve().then(() => console.log('Promise1')).then(() => console.log('Promise2'))

console.log('script end')
```
执行结果如下
```
script start
script end
Promise1
Promise2
setTimeout
```
浏览器首先执行的是`script解析`的`task`，所以最先打印的是在这个`task`中执行的`console.log`，然后在这个过程中，`Promise`会生成了一个`microTask`并将其加入`microTask`队列中，`setTimeout`会生成一个`task`并将其加入`task`队列中。
在执行完`script解析`这个`task`后，会去检测`microTask`队列，此时`microtask`队列中存在一个由`Promise`生成的`microTask`任务，将其执行后其返回结果又执行`then`，因此有生成一个`microTask`，将其加入到`microTask`队列中，执行完成后继续将`microTask`队列执行完成。
在`microTask`队列执行完成后继续检测`task`队列，此时存在一个`setTimeout`生成的`task`，将其完成后最终结果就如上所示。


#### 参考资料
1. [Vue源码详解之nextTick：MutationObserver只是浮云，microtask才是核心！](https://chuckliu.me/#!/posts/58bd08a2b5187d2fb51c04f9)
2. [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
3. [深入探究 eventloop 与浏览器渲染的时序问题 - 404Forest](https://www.404forest.com/2017/07/18/how-javascript-actually-works-eventloop-and-uirendering/)
