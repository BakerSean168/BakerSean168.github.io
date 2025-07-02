---
title: vue中props和emit的定义方法总结
categories:
  - null
description: vue中props和emit的定义方法总结
date: 2025-06-24 16:00:26
tags:
---

## Composition API + defineProps

```ts
// 基础形式
const props = defineProps<{
  visible: boolean;
  task: TaskInstance;
}>();

// 带默认值
const props = withDefaults(defineProps<{
  visible?: boolean;
  title?: string;
  count?: number;
}>(), {
  visible: false,
  title: '默认标题',
  count: 0
});

// 运行时声明
const props = defineProps({
  visible: Boolean,
  task: Object as PropType<TaskInstance>
});

// 基础形式
const emit = defineEmits(['close', 'save']);

// TypeScript 类型定义
const emit = defineEmits<{
  (e: 'close'): void;
  (e: 'save', data: TaskData): void;
  (e: 'update', id: string, value: any): void;
}>();

// 运行时声明带验证
const emit = defineEmits({
  close: null,
  save: (payload: TaskData) => {
    return payload && typeof payload.id === 'string'
  }
});

```