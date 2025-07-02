---
title: 静态初始化引起的pinia报错
categories:
  - null
description: 静态初始化引起的pinia报错
date: 2025-07-02 08:20:46
tags:
---

### 报错信息：  

```
pinia.js?v=e344808b:5731 Uncaught Error: [🍍]: "getActivePinia()" was called but there was no active Pinia. Are you trying to use a store before calling "app.use(pinia)"?
See https://pinia.vuejs.org/core-concepts/outside-component-usage.html for help.
This will fail in production.
    at useStore (pinia.js?v=e344808b:5731:13)
    at new GoalApplicationService (goalApplicationService.ts:9:31)
    at <static_initializer> (goalEventHandlers.ts:6:32)
    at goalEventHandlers.ts:1:1
```

### 问题相关代码

```ts
export class GoalEventHandlers {
  private static goalService = new GoalApplicationService();
}
```

```ts
export class GoalApplicationService {
    private readonly goalRepository: ReturnType<typeof useGoalStore>;
    constructor() {
        // 初始化GoalStore
        this.goalRepository = useGoalStore();
    }
}
```

```ts

app.mount('#app')
  .$nextTick(() => {
    // 初始化所有插件
    window.shared.ipcRenderer.on('main-process-message', (_event: any, message: any) => {
      console.log(message)
    })
    EventSystem.initialize();
  })
```

### 报错原因

众所周知，ts 是静态编译的，是指在模块加载时就执行的代码，而不是在实例创建或方法调用时执行，这就会导致提前执行相关代码。

EventSystem 会初始化 GoalEventHandlers，而 GoalEventHandlers 会获取 GoalApplicationService 的实例；GoalApplicationService 则会调用 pinia  

静态编译顺序：  
```
1. 引擎启动
2. 解析 main.ts
3. 解析 import 语句链
   ├── import EventSystem
   ├── import GoalEventHandlers  ← 🎯 这里！
   └── import GoalApplicationService
4. 执行静态初始化代码
5. 创建 Vue 应用
6. 注册 Pinia

```

### 解决方法

延迟执行  

``` ts
export class GoalEventHandlers {
  private static goalService: GoalApplicationService | null = null;

  // 延迟获取服务实例
  private static getGoalService(): GoalApplicationService {
    if (!this.goalService) {
      this.goalService = new GoalApplicationService();
    }
    return this.goalService;
  }
}
```

在后面的函数中再调用 getGoalService 来生成 goalService 实例，避免在刚加载时生成。