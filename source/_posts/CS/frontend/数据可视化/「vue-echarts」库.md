---
title: 「vue-echarts」库
categories:
  - null
description: 「vue-echarts」库
date: 2025-07-31 11:19:47
tags:
---

[echarts 官方文档](https://echarts.apache.org/zh/index.html)
[vue-echarts 官方文档](https://github.com/ecomfe/vue-echarts)


## 安装

使用 pnpm：  
```bash
pnpm add echarts vue-echarts
```

Vue3 中全局注册 vue-charts 组件：  
```ts
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import VueECharts from 'vue-echarts'
import 'echarts' // 确保引入 ECharts

const app = createApp(App)
app.component('v-chart', VueECharts)
app.mount('#app')
```

## 问题

安装时出现了报错：  
```bash
WARN  Issues with peer dependencies found
.
└─┬ vue-echarts 7.0.3
  └── ✕ unmet peer echarts@^5.5.1: found 6.0.0
```

上面的安装代码默认安装最新版本，但 vue-echarts 还没有适配最新版本 echarts，所以需要降级 `pnpm add echarts@^5.5.1`

## 使用

### 导入

使用需要导入对应的 echarts 模块 和 vue-charts 组件：  
可以全量导入，也可以按需导入。 [vue-echarts 提供了检测所需导入的 echarts 模块的工具](https://vue-echarts.dev/#codegen)，只需要提供 option 数据，工具会自动生成所需导入的 echarts 模块。

```ts
// 示例
import { use } from 'echarts/core'
import { BarChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  GridComponent
} from 'echarts/components'
import { CanvasRenderer } from 'echarts/renderers'
import type { ComposeOption } from 'echarts/core'
import type { BarSeriesOption } from 'echarts/charts'
import type {
  TitleComponentOption,
  TooltipComponentOption,
  GridComponentOption
} from 'echarts/components'
import VChart, { THEME_KEY } from 'vue-echarts';
use([TitleComponent, TooltipComponent, GridComponent, BarChart, CanvasRenderer])
provide(THEME_KEY, 'light');

```

### 示例

使用 computed 来响应式地获取要渲染图表的参数。  
不过这样就不能使用官方工具来按需导入了（会报参数识别不到的错误），可以先使用 mock 数据。  

```ts
// template
<section>
    <h2>高效时间段分析</h2>
    <h5>当前目标: {{ selectedGoal?.name }}</h5>
    <div>
      <v-chart class="chart" :option="periodBarOption" autoresize />
    </div>
</section>

// script
const periodBarOption = computed(() => {
  // 获取当前目标的记录
  const records = (selectedGoal.value?.records as GoalRecord[]) || [];
  const stat = classifyGoalRecordsByPeriod(records);

  return {
    title: {
      text: '不同时间段的任务完成数',
      left: 'center',
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
    },
    xAxis: {
      type: 'category',
      data: timePeriods,
    },
    yAxis: {
      type: 'value',
      minInterval: 1,
    },
    series: [
      {
        name: '完成数',
        type: 'bar',
        data: timePeriods.map(period => stat[period]),
        itemStyle: {
          color: '#5470C6',
        },
      },
    ],
  };
});
```

### 优化细节

