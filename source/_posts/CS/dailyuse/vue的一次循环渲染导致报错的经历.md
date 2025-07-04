---
title: vue的一次循环渲染导致报错的经历
categories:
  - Vue
description: vue的一次循环渲染导致报错的经历
date: 2025-06-30 08:00:49
tags:
---

## `computed`套`computed`导致的循环报错

猜测原因如下：  

业务逻辑：  
1. 在 store 中有一个 editingTaskTemplate 变量  
2. 在 父组件中使用 computed（set + get）来响应式获取 store 中的 editingTaskTemplate  
3. 通过 props 传给多个子组件，子组件再通过 computed 来响应式修改 editingTaskTemplate

上面的业务逻辑导致了 子组件的 computed 在第一次渲染时触发 set ，修改 对象，然后又触发父组件的 set 修改，然后又触发子组件的 set，引起循环  

直接在每个子组件中使用 store 中的变量

# 一个高阶语法

这个 `updateTemplate` 使用了**高阶函数（Higher-Order Function）**语法，具体来说是：

## 语法分析

````typescript
const updateTemplate = (updater: (template: TaskTemplate) => void) => {
  const updatedTemplate = props.modelValue.clone();
  updater(updatedTemplate);
  emit('update:modelValue', updatedTemplate);
};
````

### 1. **函数作为参数**
- `updater: (template: TaskTemplate) => void` - 接收一个函数作为参数
- 这个函数接收 `TaskTemplate` 类型的参数，返回 `void`

### 2. **回调函数模式**
````typescript
// 使用时：
updateTemplate((template) => {
  template.updateTimeConfig({
    // 修改逻辑
  });
});
````

### 3. **函数式编程模式**
这是一种**函数式编程**的模式，类似于：
- JavaScript 的 `Array.map(callback)`
- `Array.forEach(callback)`
- React 的 `setState(updater)`

## 工作流程

````typescript
// 1. 调用时传入一个回调函数
recurrenceType.set = (value) => {
  updateTemplate((template) => {           // ← 传入回调函数
    template.updateTimeConfig({
      // ...修改逻辑
    });
  });
};

// 2. updateTemplate 内部执行：
const updateTemplate = (updater) => {
  const updatedTemplate = props.modelValue.clone();  // 克隆对象
  updater(updatedTemplate);                          // 执行回调，修改克隆的对象
  emit('update:modelValue', updatedTemplate);        // 发出更新事件
};
````

## 类似的模式

### React 的 setState 模式：
````javascript
// React
this.setState((prevState) => ({
  count: prevState.count + 1
}));
````

### JavaScript 数组方法：
````javascript
// Array.map
const newArray = oldArray.map((item) => item * 2);

// Array.forEach
array.forEach((item) => console.log(item));
````

## 优势

1. **不可变性**：先克隆，再修改，保证原对象不被改变
2. **复用性**：`updateTemplate` 可以处理各种不同的修改逻辑
3. **安全性**：统一的更新流程，确保每次都会发出事件
4. **清晰性**：修改逻辑和更新机制分离

## 等价的传统写法

如果不用高阶函数，需要这样写：

````typescript
// ❌ 传统写法 - 需要重复代码
const updateRecurrenceType = (value: string) => {
  const updatedTemplate = props.modelValue.clone();
  updatedTemplate.updateTimeConfig({
    // ...修改逻辑
  });
  emit('update:modelValue', updatedTemplate);
};

const updateRecurrenceInterval = (value: number) => {
  const updatedTemplate = props.modelValue.clone();  // 重复
  updatedTemplate.updateTimeConfig({
    // ...修改逻辑
  });
  emit('update:modelValue', updatedTemplate);        // 重复
};
````

所以 `updateTemplate` 是一个很优雅的**模板方法模式**，减少了重复代码。