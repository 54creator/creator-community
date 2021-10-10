---
title: Serverless 的运行原理与组件架构
description: 本文重点探讨下开发者使用 Serverless 时经常遇到的一些问题，以及如何解决
keywords: Serverless 运行原理,Serverless 组件架构
date: 2019-08-21
thumbnail: https://img.serverlesscloud.cn/20191227/1577409288454-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg
categories:
  - guides-and-tutorials
authors:
  - Alfred Huang
authorslink:
  - https://zhuanlan.zhihu.com/ServerlessGo
tags:
  - 运行原理
  - serverless
---

过去一年，我们和大量 Serverless 用户进行了线上和线下的交流，了解大家的业务场景、对 Serverless 的看法和使用体验。

大部分用户认为 Serverless 会是云计算下一阶段的必然趋势，**但不是现在。** 为什么呢？因为构成 Serverless 架构的云函数尽管有引以为傲的自动扩缩能力，但是糟糕的开发体验、让人畏惧的冷启动、原有业务的改造难题等等，均降低了使用者的信心。

因此，尽管不少用户认可 Serverless 的价值，但依然认为其很难承载核心业务。

针对这些关键问题，腾讯云在今年 6 月份发布了 Serverless 2.0，全面升级了产品形态、系统调度以及开发者工具。为了便于大家理解，我们就从云函数的运行原理作为切入点，以解释问题产生的原因以及云函数的应对方法。

* * *

### 首先，我们看一下云函数，或者说 FaaS 和 IaaS、PaaS 的区别。

如下图所示，FaaS 不仅给用户提供了标准的 Runtime，同时在应用层也帮用户管理了请求的调度。开发者只需要聚焦在核心业务逻辑开发，按照函数的粒度去编写代码。而与底层硬件相关的资源维护，则交给更加专业的云厂商来搞定。

因此，对于用户来讲，可以把更多的精力和时间放在业务上。而 IaaS 和 PasS，则均需要用户去运维云主机或者容器集群、搭建业务所需的运行环境。

![FaaS 所处的地位](https://img.serverlesscloud.cn/20191227/1577409288439-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg)

### 其次，我们来看下云函数如何构成 Serverless 架构，以及云函数如何帮助用户做资源管理和请求调度。

在这里我们也将解答云函数的冷启、降低核心业务迁移复杂度等问题。

![Serverless 促进架构演进](https://img.serverlesscloud.cn/20191227/1577409288438-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg)

如下图所示，开发者在实际使用时，可以借助 Web IDE 或者本地 IDE 完成代码开发，然后通过插件、工具等方式把代码及其相关依赖，一起打包部署到云函数平台，用户可以自行选择部署为函数形态或者服务形态。

![Serverless 2.0 运行原理](https://img.serverlesscloud.cn/20191227/1577409288436-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg)

在代码里，用户需要自己实现业务逻辑，比如访问数据库、对象存储、消息队列、第三方服务接口等。计算逻辑和后端服务共同构成了所谓的 Serverless 应用架构。而终端用户根据平台提供的请求方式，去触发部署在云函数平台上的业务代码，比如发送 http 请求，平台会根据用户的请求量去拉起相应的计算资源运行用户代码。

### 这里需要重点关注函数形态和服务形态的差异，因为服务的形态可以大大降低复杂业务迁移的成本。

*   服务形态支持直接部署基于框架开发的核心业务，如 Node.js 的 express、koa 等框架，不用为了应用 Serverless 而拆分成函数。平台会帮用户启动服务进程、端口监听，同时服务形态不会限制业务的实际运行时长。
*   函数形态和服务形态在收到用户请求的时候，均能实现自动扩缩。
       * 函数形态会针对用户的每个请求都分配一个运行实例，因此所有请求的执行体验是一样的。当没有请求的时候，平台是没有实例在运行的，所以可以做到按需请求，但是这也会造成所谓的冷启动 —— 即当用户的首次请求进入平台的时候，平台会临时拉起资源，而这个过程会消耗一定的时间。为了消除冷启，云函数平台会预先初始化一批不同规格的实例放在资源池中，当用户有请求进入时，可以快速从资源池申请一个实例，直接挂载用户的代码运行，从而降低了资源申请时间。同时，针对函数形态，平台会根据历史并发数据进行预测，帮用户预留一定量的实例，这些实例会预先分配到用户的账号下并且加载好了用户的代码，从而不仅直接消除了冷启，也增加了实例复用几率。
       * 而服务形态可以至少帮用户预留一个常驻实例，并且把用户的所有请求都投递到首个实例，根据实例的使用情况，自动的动态扩缩。

![示意图](https://img.serverlesscloud.cn/20191227/1577409288399-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg)


* 函数形态更适合新建项目，可以敏捷迭代，业务按照函数的粒度开发，不仅可以轻松实现云上多产品的联动，也可以享受函数的高并发及性能一致体验。服务形态更适合已有项目的迁移、重度复杂业务、需要长时运行的业务。

### 最后讲讲 Serverless 2.0 的组件架构。

如下图所示，用户虽然只需要关注绿色部分和业务相关的代码实现，但是平台也需要提供强大的开发者工具来保障开发和使用体验。如云函数推出的 Serverless 本地开发工具、VS Code 插件，与 CODING 联合推出的 Web IDE、DevOps 平台等，均能很大程度上提升开发、部署效率，实现本次开发、本地调试、联动云端调试、本地部署、版本发布等能力。

![组件架构](https://img.serverlesscloud.cn/20191227/1577409288636-v2-577c2b21d600e3ea07f156f3e9d2d6b8_1200x500.jpg)

同时，云函数也完善了配套的监控和告警机制，提供如调用次数、内存使用、并发使用、超时、代码错误等多维度的监控和告警能力。这些基础设施、资源管理、安全、容灾等能力，是云函数平台必备的基础能力，也是开发者关心的核心能力。

Serverless 不仅仅是计算，还需要不断完善周边生态。

随着用户量的增加，Serverless 必然会面临更多的挑战 —— 怎么帮助用户组织管理代码，怎么解决带状态的业务诉求，怎么实现数据库连接数管理，怎么实现应用级部署等等。我们也在不断探索和优化用户的使用体验，计划提供诸如 Serverless DB、性能监控、日志分析、Serverless 框架、函数编排、高性能调用等功能。

后续的专栏文章也将陆续解读更多核心能力，帮助开发者更好地理解和使用 Serverless。

**Serverless is more!**

> **传送门：**
> - GitHub: [github.com/serverless](https://github.com/serverless/serverless/blob/master/README_CN.md) 
> - 官网：[serverless.com](https://serverless.com/)

欢迎访问：[Serverless 中文网](https://serverlesscloud.cn/)，您可以在 [最佳实践](https://serverlesscloud.cn/best-practice) 里体验更多关于 Serverless 应用的开发！