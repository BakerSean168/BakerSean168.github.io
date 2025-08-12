---
title: DailyUse的「Reminder」模块
categories:
  - null
description: DailyUse的「Reminder」模块
date: 2025-07-14 21:41:32
tags:
---

## 准备

### Reminder 模块的核心任务是：

1. 管理所有的 schedule 服务提供的提醒
2. 添加提醒任务
3. 类番茄时钟
4. 简单的绝对时间提醒和复杂的相对时间提醒

### 提醒任务组

提醒任务组，将多个同类型的提醒任务归为一组，能够管理多个提醒任务，也可以达到组合提醒的效果。

### 提醒任务

提醒周期：

能够选择一个时间段（或一直持续），在时间段内重复 提醒任务  
可以添加专注功能（专注周期），屏蔽其他普通提醒（但保留重要提醒）

任务周期：

完成一次定义的所有任务所需要的时间

任务时间：

应该支持复杂的时间设置：

- 经过 40m 后提醒，休息 10m；
- 经过随机 3-5m 后提醒，休息 30s；持续 90m 后，休息 10m；
  整个任务周期为 100m，但是任务周期里有多个小周期 3.5m-5.5m，默认的提醒周期应该为一个任务周期，但应该可以被用户设置

```ts
{
  name: "专注训练",
  description："专注90m，休息10m",
  duration: 100,
  type: 'relative',
  times: [
    {
      name: "专注周期",
      description: "多个小周期专注3-5m，休息30s",
      duration: 90,
      times: [
        {
            name: "专注小周期",
            duration: 3.5-5.5,
        },
        {
          name: "休息小周期",
          duration: 30,
        }
      ]
    }，
    {
      name: "休息周期",
      duration: 10,
    }
  ],
}
```

### 可视化功能：

- 添加一个任务周期
- 添加一个提醒周期

基础提醒任务组：

- 起床提醒：绝对时间 8:00
- 饭点提醒：多个绝对时间点
- 睡觉提醒：绝对时间 23:00

电脑前任务组：

- 休息提醒：每过 40m 提醒一次 休息 5m

## 具体实现

### aggregates

1. ReminderTemplateGroup

```ts
export type ReminderTemplateEnableMode = "group" | "individual";

export interface IReminderTemplateGroup {
  id: string;
  name: string;
  enabled: boolean;
  enableMode: ReminderTemplateEnableMode;
  templates: IReminderTemplate[];
}
```

2. ReminderTemplate

```ts
type ReminderTimeConfig = AbsoluteTimeConfig | RelativeTimeConfig;

interface RelativeTimeSchedule {
  name: string;
  description?: string;
  /** 时间长度，以秒为单位 */
  duration: number | DurationRange;
  times: RelativeTimeSchedule[];
}

interface DurationRange {
  min: number;
  max: number;
}

interface RelativeTimeConfig {
  name: string;
  description?: string;
  duration: number | DurationRange; // 时间长度，以秒为单位
  type: "relative";
  times: RelativeTimeSchedule[];
}

interface AbsoluteTimeConfig {
  name: string;
  type: "absolute";
  description?: string;
  /** 具体时间点 */
  times: RecurrenceRule[];
}

export interface IReminderTemplate {
  groupId: string;
  id: string;
  name: string;
  description?: string;
  importanceLevel: ImportanceLevel;
  selfEnabled: boolean;
  enabled: boolean;
  notificationSettings: {
    sound: boolean;
    vibration: boolean;
    popup: boolean;
  };
  timeConfig: ReminderTimeConfig;
}
```

为了实现灵活的提醒控制（组内统一开启、统一关闭、启用组内的部分提醒（可记忆，随时与两者之间切换））

20250718:为了实现组控制+自我控制结合，给 ReminderTemplate 添加 selfEnabled 属性

当前 group 有控制模式（组控制和模板自己控制）和 组是否启用 模板则只有 是否开启  
此时模板是否启用应该有两种情况：  
模板没有在组内，则模板启用 = 模板.enabled  
模板在组内，则模板启用 = （组.enabled）组启用了组控制时,(模板.enabled)组启用了模板控制时

我要实现 根据模板是否开启，来设置模板的样式

直接在 ReminderTemplate 中添加一个 selfEnabled 的属性，用 selfEnabled 来存储模板本身的启用状态。enabled 用来存储模板此时真正是否启用。如果一个 模板 在 组中，当组的控制模式变换时，组就可以根据当前的模式和模板的 selfEnabled 来计算出模板的 enabled。

1. 提醒模板定义
2. 提醒实例定义
3. 任务队列服务 + 弹窗提醒服务

### views

我想让 ReminderView 有一下特征：  
采用手机桌面的样式，用 grid 布局，将显示区域分为多个方格，每个 reminderTemplate 以「图标+名称」的样式占一格；而 reminderTemplateGroup 以「文件夹」的样式占四格（正方形），右键显示区域出现一个菜单，可以添加 reminderTemplate、reminderTemplateGroup。右键对应对象，出现一个菜单，可以删除、修改、添加 reminderTemplate、reminderTemplateGroup。
有拖动功能。能拖动调整对象位置、将 reminderTemplate 添加到 reminderTemplateGroup 中。  
左键点击 reminderTemplate 显示 reminderTemplate 的详情以及 enable 开关，左键点击 reminderTemplateGroup 的空白区域显示 reminderTemplateGroup 的详情（所有包含的 reminderTemplate 以及 文件夹的统一 enable 按钮（是否由文件夹控制所有 reminder、enbale or disable））点击 reminderTemplateGroup 的文件夹中的 reminderTemplate 则视为左键点击 reminderTemplate。  
总之提供类似手机 app 与 文件夹的功能。  
将整体代码较好地分割为多个 components、composables 文件，并在 ReminderView 中进行组合。缺失的功能可以对已有代码进程修改

让 claude4 生成了一下，用不了，慢慢修改一下。

1. 创建「屏幕区域」
   屏幕区域背景暂时透明，应当支持自定义背景，以 Grid 布局，items 能按顺序排列。

2. 实现点击 ReminderTemplate 显示详情框

在点击 RemidnerTemplate 时显示，展示 reminder 属性，还有按钮控制是否启用；有编辑按钮，点击后属性进入可编辑状态；总之是一个集成展示、控制、编辑的组件。

# 第一次重构 Reminder 模块

20250722

1. 之前把 Reminder 和 ReminderGroup 都作为聚合根对象，在 reminder 中有 GroupId 属性，在 reminderGroup 中有 Reminders 属性。这有点耦合复杂了。  
   新想法：

- 只把 ReminderGroup 作为聚合根对象，reminder 作为实体。直接通过 Group 来管理 reminder。
- 设置一个系统 Group 作为初始的根 Group，所有 reminder、group 都会添加到这个 Group 中。
- 除了根 Group，其他 Group 应该不允许有子 Group，防止无限嵌套。

2. 也没有必要给 reminder 添加 enabled 属性，它应该是根据 selfEnablebled 和 父 Group 的属性实时计算得出的

3. 初始化根 Group

先添加了一个用户登录时执行的初始化任务，给数据库中添加根 Group。

数据库存储：  
Group 和 reminder 分开存储

主进程存储：  
根 Group、其他 Group、reminder。  
reminder 直接包含在 Group 中。

```ts
export interface IReminderTemplateGroup {
  uuid: string;
  name: string;

  enabled: boolean;
  enableMode: ReminderTemplateEnableMode;
  templates: IReminderTemplate[];
}
```

渲染进程:  
只要获取所有 Group 就行了。  
先给 GRID 组件传入根 Group 的（子元素，template 和 Group，简化处理） templates 和 其他 Group。  
根 Group 的 enabled 属性直接控制所有的启用状态。

_有点类似原型链的关系_

## views

渲染进程只需要接收 所有的 templateGroup 数据，将 系统 Group（根 Group）中的 Reminder 和 其他 Group 作为 Grid 的子元素。

Reminderview.vue 作为 Reminder 模块的主界面  

### 怎么实现修改 item 后，响应式渲染？

现在采用修改数据后，立刻重新从主进程获取所有数据并更新 pinia store 中的数据，然后重新渲染的方式。  

### 各种事件系统

现在我的 ReminderGrid 布局里有 GroupItem 和 TemplateItem， GroupItem 里又使用 RemidnerGrid 布局来展示包含的 TemplateItem，我的 Grid 组件是不是使用 inject 来处理事件比较好？ 否则用 emit 的化 GroupItem里的 Grid 布局里的 TemplateItem 的事件是跨多个组件是处理不了的

## 主进程业务

现在只有 聚合根 Group 和 实体 Template，实体的 TimeConfig 属性存储提醒的时间，每个 template 只有一个 schedule，类似闹钟。  

主要业务：  
- 启用 Reminder，将 Remidner 添加到 弹窗提醒的任务队列中。当前只实现弹窗提醒。
- 给渲染进程提供修改 启用状态的方法（group 的 enableMode、enabled，template 的 selfEnabled）， 主进程根据 Group 的 enableMode 和 enabled、Template 的 selfEnabled，来判断某一个 Template 是否启用，并根据变化来操作 Reminder 的提醒队列。

创建一个初始化一个 Group 的所有 Reminder 的方法，在一个 enable 相关的变化后调用该方法。


# 渲染进程

## presentation

### 实现细节

#### reminderGroup 的模式更改时页面响应式变化的问题

传入 reminderGroupCard 的对象是通过下面的响应式对象传入的，传入后在 Card 中调用了方法，通过ipc修改对象的 enableMode，并在 store 中同步，但是 Card 中还没有响应式更新 enableMode的状态。该怎么做  
```ts
const reminderTemplateGroupCard = ref({
  show: false,
  templateGroup: null as ReminderTemplateGroup | null
});
```

有两种方案：  
1. 传入 templateGroupUuid 而非完整对象，通过传入的 uuid 获取 store 中的对象，绕过了 Card 传入的对象不是响应式的问题。
2. 传入 完整对象，但是页面中渲染的属性（如 enbaled）还是通过 computed 和 传入对象的 uuid 从 store 中获取。

好像其实是一个方案