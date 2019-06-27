---
title: gitlab CI初步学习
date: 2019-06-26 10:07:48
tags: git
categories: 笔记
---
最近看了下关于devOps方面的内容，初步的了解了下自动化部署方面关于`gitlab`的`CI`处理流程，这里大概记录一下相关的东西。

`gitlab CI`是通过`Gitlab Runner`执行的一个脚本配置，当gitlab上的项目配置了runner并且在项目的根路径下添加了`.gitlab-ci.yml`的时候，每次的git提交或者推送都会触发`gitlab ci`（触发方式是可以在`gitlab-ci.yml`中配置的）。

`Gitlab Runner`将会根据`.gitlab-ci.yml`配置项执行相应的脚本，默认是有三个阶段:`build`, `test`, `deploy`。每个阶段下可以有多个任务，如果阶段下没有配置任务，则该阶段会被忽略。
`Gitlab CI`是使用`YAML`的语法进行配置文件编写的，语法请查看[官方文档](https://yaml.org/)，以下是关于配置项的具体说明。

## 配置项
### 任务
任务配置项是`.gitlab-ci.yml`中的最基本元素，是具有任意名称并且包含script配置项的顶级元素，没有数量限制，定义了当前任务的执行约束。

任务名字的保留关键字
+ image
+ services
+ stages
+ types
+ before_script
+ after_script
+ variables
+ caches

每个任务下有以下配置项

|字段名|描述|
|------|------|
|script|Runner执行的shell脚本|
|image |任务执行的docker镜像|
|services|任务执行的docker services镜像|
|before_script|重写一组在作业前执行的命令|
|after_script|重写一组在作业后执行的命令|
|stages|定义被job调用的stages|
|stage|定义当前job的stage|
|only|定义任务创建的情况|
|except|定义任务不需要创建的情况|
|tags|定义执行的runner的tags列表|
|allow_failure|允许任务失败，任务失败不影响commit状态|
|when|定义什么时候执行job|
|environment|定义deploy job的执行环境|
|cache|定义后续运行之间应缓存的文件列表|
|artifacts|用于指定成功后应附加到job的文件和目录的列表|
|dependencies|定义任务间的依赖关系，有依赖关系的任务之前可以进行artifacts传递|
|coverage|提取该任务下的代码覆盖率|
|retry|定义任务失败自动重试的时间以及重试次数|
|parallel|定义并行运行的作业实例数量|
|trigger|可用于强制使用API调用重建特定分支，tag或commits|
|include|允许当前任务由额外的yaml文件配置|
|extends|配置当前任务继承的配置项入口|
|pages|上传作业结果以用于gitlab pages|
|variables|在job级别定义job变量|

#### stage

#### image & services
指定任务执行的`Docker`镜像以及一系列服务，并应用于整个`job`周期
#### before_script & after_script
`before_script`定义了在所有任务（`script`配置项）执行前需要执行的命令，包括`deploy jobs`，在`artifacts`修复后，这个配置项是一个数组或者多行字符串。
`after_script`定义了在所有任务（`script`配置项）完成后需要执行的命令，包括失败项。这个配置项可以是数组或者多行字符串。
#### stages
在全局定义执行的各个阶段
```YAML
stages:
  - build
  - test
  - deploy
```
每个stage允许有多个，按定义的顺序依次执行，只有当前的阶段全部执行成功后才会执行下一个阶段。
#### stage
`stage`是按工作定义的，依赖于全局定义的阶段。 它允许将作业分组到不同的阶段，并且同一阶段的作业并行执行（受特定条件限制）。
