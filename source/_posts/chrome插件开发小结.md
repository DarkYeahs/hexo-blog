---
title: chrome插件开发小结
date: 2019-04-26 11:37:41
tags: chrome 插件开发
categories: 笔记
---
&nbsp;&nbsp;最近写了一个很简单的chrome插件todolist，一个很常见的任务列表插件，主要功能有
+ 任务列表展示
+ 任务完成操作
+ 任务新增
+ 桌面通知

&nbsp;&nbsp;项目地址是这个[todoList](https://github.com/DarkYeahs/todoList),整理下开发的流程以及开发过程中遇到的问题。

##### 开发历程
&nbsp;&nbsp;这里讲的是开发过程的目录结构，chrome会读取根目录下的`manifest.json`文件配置,这里项目的配置如下
```json
{
    "manifest_version": 2,

    "name": "todolist",
    "version": "1.0",
    "description": "todolist",
    "icons": {
        "16": "images/icon16.png",
        "48": "images/icon48.png",
        "128": "images/icon128.png"
    },

    "browser_action": {
        "default_icon": {
            "19": "images/icon19.png",
            "38": "images/icon38.png"
        },
        "default_title": "个人计划列表",
        "default_popup": "./dist/popup.html"
    },

    "permissions": [
        "*://*/",
        "notifications"
    ],

    "web_accessible_resources": [
        "images/*.png"
    ],

    "background": {
        "page": "./dist/background.html"
    },
    "content_security_policy": "script-src 'self' 'unsafe-eval' http://localhost:4000; object-src 'self';"
}
```
&nbsp;&nbsp;第一个字段`manifest_version`指的是使用的`manifest.json`的版本，目前1的版本已经被废弃了，所以使用的是2的版本。`name`、`version`、`description`以及`icon`字段分别对应插件名、插件版本、插件描述以及插件icon,这些配置会在插件安装的时候展示给用户。

&nbsp;&nbsp;`browser_action`字段则是声明会在浏览器的工具栏添加的操作，在这里声明了在工具栏添加一个icon以及对应的popup页面

&nbsp;&nbsp;`permissions`，`web_accessible_resources`则是声明插件需要的权限以及接收资源的格式，这里申请的是chrome桌面通知权限以及网络请求权限（PS:这里通配符匹配了任意请求地址），接收的资源格式是`images/*.png`格式

&nbsp;&nbsp;`background`则是声明后台常驻脚本/页面

&nbsp;&nbsp;项目主要由两部分构成，一个是popup页面以及后台运行页面，popup页面在用户打开工作栏上对应的插件按钮时会运行展示，后台运行页面则是在浏览器打开的时候就会在后台运行，直到浏览器关闭才会停止。

&nbsp;&nbsp;popup页面主要的功能有
+ 任务列表展示
+ 任务列表状态更改
+ 任务新增

&nbsp;&nbsp;后台页面主要的功能有
+ 数据请求
+ 任务通知

内容比较简单，具体可以参照代码查看。

##### 开发中遇到的问题
1. 跨域问题
    在popup页面上不能进行跨域请求，处理的方式，将请求发起放到`background.html`上，popup页面通过调用chrome的`chrome.runtime.sendMessage`向`background.html`发起请求，由`background.html`向服务端发起请求并将请求到的结果返回给popup页面。
2. 页面点击会导致以下报错，但不会导致页面本身执行失败
```
Refused to execute JavaScript URL because it violates the following Content Security Policy directive: "script-src 'self' chrome-extension://". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution.
```
    搜了下是chrome插件关于CSP的协议的控制，chrome最新的插件不允许内联js运行，搜索代码发现是因为`href="javascript:;"`导致的，将`href="javascript:;"`去除即可


