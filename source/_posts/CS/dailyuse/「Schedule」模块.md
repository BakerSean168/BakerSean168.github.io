---
title: 「Schedule」模块
categories:
  - dailyuse
description: 「Schedule」模块是一个基础模块，为其他模块提供基础服务。
date: 2025-06-05 19:36:27
tags:
---

「Schedule」模块是一个基础模块，为其他模块提供基础服务。  
在 DDD 架构中

现在项目应该是还没有为 task、reminder 等模块集成 schedule 功能，我们现在先来讨论一下怎么集成，来实现 提醒到事件时，弹窗或者发声音来提醒用户 的功能。
已经有了初步的 schedule、task 等，要怎么连接起来。
我的想法：  
schedule 设置一个 eventhandler，来监听其他模块的请求生成队列的事件，生成一个对应 schedule 的实体对象，实体对象管理这个事件的生命周期。
对应的 task 提前完成，能取消掉已经生成的 schedule，让他销毁；或者重新设置时间之类的。
时间到后，schedule 对象能 触发 对应的事件（弹窗、发声等），暂时不考虑用户点击弹窗上的按钮，然后自我销毁
并且服务器启动时，要能重新初始化生成 schedule
请你帮我实现一下
