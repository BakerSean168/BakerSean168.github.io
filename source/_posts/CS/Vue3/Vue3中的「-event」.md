---
title: Vue3中的「$event」
categories:
  - null
description: Vue3中的「$event」
date: 2025-07-27 17:59:34
tags:
---

在 Vue 组件中，`$event` 是一个特殊变量，用于**接收事件传递的参数**。它的作用是把子组件通过 `$emit` 发送的参数，自动传递给父组件的事件处理函数。

---

## 1. 基本用法

假设你有一个子组件，里面这样触发事件：

```typescript
emit('create-key-result', keyResult);
```

在父组件模板里监听这个事件：

```vue
<KeyResultDialog @create-key-result="handleCreateKeyResult($event)" />
```

这里的 `$event` 就是子组件 emit 时传递的 `keyResult`。

---

## 2. 多参数情况

如果子组件 emit 多个参数：

```typescript
emit('edit-key-result', goal, keyResult);
```

父组件可以这样写：

```vue
<KeyResultDialog @edit-key-result="handleEditKeyResult($event)" />
```

此时 `$event` 是第一个参数（即 `goal`）。  
如果你想拿到所有参数，可以直接写方法：

```vue
<KeyResultDialog @edit-key-result="handleEditKeyResult" />
```

并在方法里：

```typescript
function handleEditKeyResult(goal, keyResult) {
  // 两个参数都能拿到
}
```

---

## 3. 结合其它参数

你可以在事件处理时，手动加其它参数：

```vue
<KeyResultDialog @create-key-result="handleCreateKeyResult(props.goal, $event)" />
```

这里 `props.goal` 是你自己传的，`$event` 是子组件 emit 的参数。

---

## 4. 总结

- `$event` 代表子组件 emit 的第一个参数
- 如果 emit 多个参数，建议直接用方法接收所有参数
- 可以和其它参数一起传递给事件处理函数

---

**实用例子：**

```vue
<!-- 子组件 emit -->
emit('save', formData);

<!-- 父组件监听 -->
<MyForm @save="onSave($event)" />

<!-- onSave 方法 -->
function onSave(formData) {
  // $event 就是 formData
}
```

---

**结论：**  
`$event` 是 Vue 模板里用于接收事件参数的快捷方式，通常用于只传一个参数的场景。如果有多个参数，直接用方法接收即可。