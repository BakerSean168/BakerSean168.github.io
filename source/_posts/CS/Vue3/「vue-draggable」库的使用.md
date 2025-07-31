---
title: 「vue.draggable」库的使用
categories:
  - null
description: vue.draggable.next 是一款vue3的拖拽插件,是vue.draggable升级版本，同样是基于Sortable.js实现的，你可以用它来拖拽列表、菜单、工作台、选项卡等常见的工作场景。  
date: 2025-07-17 13:19:10
tags:
---

[vue.draggable中文文档](https://www.itxst.com/vue-draggable/tutorial.html)


draggable（通常指 vuedraggable）是一个基于 Sortable.js 的 Vue 组件，用于实现列表/网格的拖拽排序。它支持数组的拖拽排序、拖拽事件、动画、分组等功能，常用于实现拖拽式 UI。

### 基本用法

1. 安装
```bash
pnpm add vuedraggable
```

2. 引入并注册
```js
import draggable from 'vuedraggable';
```
在 `<script setup>` 里直接 import 并在模板中用 `<draggable />`。

3. 基本模板
```vue
<draggable v-model="list" item-key="id">
  <template #item="{ element }">
    <div>{{ element.name }}</div>
  </template>
</draggable>
```
- v-model 绑定你的数组（如 list）
- item-key 指定唯一 key（如 id）

### 常用属性

- v-model：绑定的数组，拖拽后自动更新顺序
- item-key：每个元素的唯一标识
- :animation="200"：拖拽动画时长（毫秒）
- :disabled="false"：是否禁用拖拽
- ghost-class / chosen-class / drag-class：拖拽时的样式类名
- group：分组拖拽（可实现跨列表拖拽）

### 常用事件

- @start="onDragStart"：拖拽开始
- @end="onDragEnd"：拖拽结束
- @change="onChange"：顺序变化时

### 示例

```vue
<template>
  <draggable
    v-model="items"
    item-key="id"
    :animation="200"
    ghost-class="ghost"
    @end="onDragEnd"
  >
    <template #item="{ element }">
      <div>{{ element.name }}</div>
    </template>
  </draggable>
</template>

<script setup>
import { ref } from 'vue';
import draggable from 'vuedraggable';

const items = ref([
  { id: 1, name: 'A' },
  { id: 2, name: 'B' },
  { id: 3, name: 'C' }
]);

const onDragEnd = (event) => {
  console.log('新顺序:', items.value);
};
</script>
```

### 注意事项

- v-model 绑定的数组会自动更新顺序
- item-key 必须唯一
- 你可以自定义每个 item 的渲染内容
- 支持嵌套、分组、拖拽限制等高级用法

## 一些问题

### draggable 内部元素的布局怎么控制

1. 可以在 #item 里添加布局样式，使用 绝对定位、transform 等方式控制布局
2. 给 draggable 添加 `tag='div'` 并添加类名，使用 grid、flex 布局

### draggable 实现“item 可以在所有空格内自由拖动”

vuedraggable（基于 Sortable.js）本身只支持“顺序拖拽”，即只能在已有 item 之间移动，不能拖到空白格子或任意 grid 空间。  

想法：  
默认使用类似手机的页面，直接给整个页面添加 item 占据所有空格，item 类型除了「文件夹」、「任务」外，还有「空白」。

