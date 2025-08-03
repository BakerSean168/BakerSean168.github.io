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

[配置项文档](https://echarts.apache.org/zh/option.html#grid)

通过提供的参数来修改 X轴、Y轴、tooltip 等的样式。  
我是用了 vuetify 的主题变量来作为颜色参数，可以根据用户选择的主题来变化样式。  

```Vue
<template>
  <v-chart class="chart" :option="krBarOption" autoresize />
</template>

<script setup lang="ts">
import { computed } from 'vue'
import VChart from 'vue-echarts'
import type { Goal } from '@/modules/Goal/domain/aggregates/goal'
import { useTheme } from 'vuetify'

const props = defineProps<{
  goal: Goal | null
}>()
const theme = useTheme()
const surfaceColor = theme.current.value.colors.surface 
const fontColor = theme.current.value.colors.font // 获取主题主色

const keyResults = computed(() => props.goal?.keyResults || [])

const krNames = computed(() => props.goal?.keyResults?.map(kr => kr.name) ?? [])
const krProgress = computed(() => props.goal?.keyResults?.map(kr => kr.progress) ?? [])

const krBarOption = computed(() => ({
  backgroundColor: surfaceColor,
  title: { text: '关键结果进度', left: 'center', top: 10, textStyle: { fontSize: 16 } },
  grid: { left: 60, right: 60, top: 50, bottom: 30 },
  tooltip: {
    show: true,
    backgroundColor: 'transparent',
    borderColor: 'transparent',
    textStyle: {
      color: fontColor,
      fontSize: 14
    },
    formatter: (params: any) => {
      // params.dataIndex 对应 keyResult 的索引
      const kr = keyResults.value[params.dataIndex]
      if (!kr) return ''
      return `
        <div>
          <strong>${kr.name}</strong><br/>
          起始值: ${kr.startValue}<br/>
          目标值: ${kr.targetValue}<br/>
          当前值: ${kr.currentValue}
        </div>
      `
    },

  },
  xAxis: {
    max: 100,
    splitLine: { show: false },
    axisLabel: { show: false },
    axisLine: { show: false },
    axisTick: { show: false }
  },
  yAxis: {
    type: 'category',
    data: krNames.value,
    axisTick: { show: false },
    axisLine: { show: false },
    axisLabel: { fontSize: 14 }
  },
  series: [{
    type: 'bar',
    data: krProgress.value,
    label: {
      show: true,
      position: 'right',
      formatter: '{c}%',
      fontSize: 14,
      color: fontColor
    },
    itemStyle: {
      color: '#1890ff',
      borderRadius: [8, 8, 8, 8]
    },
    barWidth: 18
  }]
}))
</script>

<style scoped>
.chart {
  width: 100%;
  height: 220px;
  min-height: 180px;
  margin-bottom: 24px;

  border-radius: 16px;
  overflow: hidden;
}
</style>

```

# 参考

https://segmentfault.com/a/1190000021898188#item-4-3