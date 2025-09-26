---
title: 把Electron项目迁移到monorepo中
categories:
  - null
description: 把Electron项目迁移到monorepo中
date: 2025-09-07 15:09:55
tags:
  - dailyuse
  - monorepo
  - Nx
---

原本项目是简单的 Electron + Vue3 的项目，现在改为 monorepo 项目，怎么把原本的 Electron+Vue3 融入进去呢  

- 结构上，将 项目按照规范放入 apps 文件夹中
- 配置上，让 项目能够直接被 Nx 管理

## 1. 文件迁移

DailyUse\electron -> DailyUse\apps\desktop\src\main  
DailyUse\src -> DailyUse\apps\desktop\src\renderer
DailyUse\vite.config.ts -> DailyUse\apps\desktop\vite.
config.ts    

### 文件提取

直接的 实体的DTO 是不是最好和 数据库存储的DTO 统一比较方便？

以 Goal 模块提取为例：  
1. 将 Goal 模块的定义确定好放到 contracts 中
  包括 核心的实体定义（IGoal、IGoalDir 等）、DTO数据定义、事件定义
  dto 类型与实体定义的区别在，使用 number 替代 Date、字符串数据或对象来替换 Map、Set 等
2. 将 Goal 模块的实体类提取到 domain-core、domain-server、domain-client 中，要 implement 上面的实体定义
3. 在 api 中完成基础的接口和迁移已有的接口、方法
4. 在 web 中也迁移已有的页面、服务



## 2. 配置修改

本身