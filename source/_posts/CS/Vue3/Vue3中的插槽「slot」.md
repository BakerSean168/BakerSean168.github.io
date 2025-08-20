---
title: Vue3中的插槽「slot」
categories:
  - null
description: Vue3中的插槽「slot」
date: 2025-08-14 13:43:14
tags:
---

## slot

### slot 是什么有什么用

插槽是 Vue 实现内容分发的 API，通过 `<slot>` 元素作为承载分发内容的出口。  
插槽的核心作用是：父组件在组件标签内插入任意内容，子组件内通过插槽控制这些内容的摆放位置。  

可以类比为 JavaScript 函数：  

```js
// 父元素传入插槽内容
FancyButton('Click me!')

// FancyButton 在自己的模板中渲染插槽内容
function FancyButton(slotContent) {
  return `<button class="fancy-btn">
      ${slotContent}
    </button>`
}
```

### 插槽的类型

- **默认插槽**  
  只有一个插槽，父组件传递的内容会被渲染到 `<slot></slot>` 位置。

- **具名插槽**  
  一个组件可以有多个插槽，通过 `name` 属性区分。父组件通过 `v-slot:插槽名` 指定内容插入到对应插槽。
  ```vue
  <!-- 子组件 -->
  <slot name="header"></slot>
  <slot></slot>
  <slot name="footer"></slot>

  <!-- 父组件 -->
  <template #header>头部内容</template>
  <template #default>主体内容</template>
  <template #footer>底部内容</template>
  ```

- **作用域插槽**  
  子组件通过 `<slot :prop="value">` 向父组件暴露数据，父组件通过 `v-slot` 获取并使用这些数据。
  ```vue
  <!-- 子组件 -->
  <slot :user="user"></slot>

  <!-- 父组件 -->
  <template #default="{ user }">
    <div>{{ user.name }}</div>
  </template>
  ```

- **条件插槽**  
  可以根据条件渲染不同的插槽内容。

- **动态插槽**  
  插槽名可以动态绑定，适用于插槽名不确定的场景。

### 插槽的使用场景

- 组件库开发：如弹窗、对话框、表格等，允许用户自定义内容。
- 复用组件：如卡片、列表项等，插槽让组件更灵活。
- 复杂布局：通过具名插槽实现灵活的页面结构。

### 注意事项

- 插槽内容默认在父组件作用域内编译（即作用域插槽除外）。
- 具名插槽和默认插槽可以混用。
- 作用域插槽常用于列表、表格等需要自定义渲染内容的场景。

### Vue3 新特性

- `v-slot` 统一了插槽语法，推荐使用 `#插槽名` 简写。
- 支持在 `<script setup>` 中使用插槽类型推断（TypeScript 支持更好）。

---

**参考文档