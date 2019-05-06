---
title: bfc,盒子模型小结
date: 2019-05-03 00:48:54
tags: CSS
categories: 笔记
---
### 盒子模型

CSS 基础框盒模型（CSS basic box model），是浏览器渲染中比较常用到的概念。主要有两种，一种是ie标准盒模型，一种是w3c标准盒模型。
```CSS
/* ie标准盒模型 */
.container {
  box-sizing: border-box;
}

/* w3c标准盒模型，大多数浏览器默认w3c标准盒模型 */
.container {
  box-sizing: content-box;
}
```

#### 构成
盒模型由4部分组成，内容（content），内边距（padding），边框（border）以及外边距（margin）。ie盒模式盒子的宽度和高度即为指定的宽度和高度，由内容，内边距以及边框构成，w3c标准盒模型盒子的宽度和高度指的是内容占据的空间，元素实际占据的空间要计算内边距以及边框的大小。

内容区域 content area ，由内容边界限制，容纳着元素的“真实”内容，例如文本、图像，或是一个视频播放器。它的尺寸为内容宽度（或称 content-box 宽度）和内容高度（或称 content-box 高度）。它通常含有一个背景颜色（默认颜色为透明）或背景图像。

内边距区域 padding area 由内边距边界限制，扩展自内容区域，负责延伸内容区域的背景，填充元素中内容与边框的间距。它的尺寸是 padding-box 宽度 和 padding-box 高度。

内边距的粗细可以由 padding-top、padding-right、padding-bottom、padding-left，和简写属性 padding 控制。

边框区域 border area 由边框边界限制，扩展自内边距区域，是容纳边框的区域。其尺寸为 border-box  宽度 和 border-box 高度。

### bfc
bfc全称是块格式化上下文（block formatting context），是页面可视化css渲染的一部分。其主要表现为内部子元素布局不会干扰到外部元素。

创建bfc的方式有以下几种：
+ 根元素或包含根元素的元素（html或者iframe）
+ 浮动元素（元素的 float 不是 none）
+ 绝对定位元素（元素的 position 为 absolute 或 fixed）
+ 行内块元素（元素的 display 为 inline-block）
+ 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
+ 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
+ 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
+ overflow 值不为 visible 的块元素
+ display 值为 flow-root 的元素
+ contain 值为 layout、content或 strict 的元素
+ 弹性元素（display为 flex 或 inline-flex元素的直接子元素）
+ 网格元素（display为 grid 或 inline-grid 元素的直接子元素）
多列容器（元素的 column-count 或 column-width 不为 auto，包括 column-count 为 1）
+ column-span 为 all 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（标准变更，Chrome bug）。

bfc主要的应用有以下几种：
+ 内边距折叠问题
+ 双列布局
+ 浮动清除
