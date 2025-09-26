---
title: CSS
categories:
  - null
description: CSS
date: 2025-09-20 22:34:49
tags:
---
# CSS


## 动画

animation

### 常用属性

| 属性 | 说明 | 示例值 |
|------|------|--------|
| `animation-name` | 指定要运用的关键帧名称 | `slideIn` |
| `animation-duration` | 动画完成一个周期所需时间 | `1s` 或 `1000ms` |
| `animation-timing-function` | 动画的速度曲线 | `linear`, `ease`, `ease-in`, `ease-out`, `cubic-bezier(0.1, 0.7, 1.0, 0.1)` |
| `animation-delay` | 动画开始前的延迟时间 | `2s` |
| `animation-iteration-count` | 动画播放次数 | `3`, `infinite` |
| `animation-direction` | 动画是否反向播放 | `normal`, `reverse`, `alternate`, `alternate-reverse` |
| `animation-fill-mode` | 动画执行前后如何应用样式 | `none`, `forwards`, `backwards`, `both` |
| `animation-play-state` | 动画运行或暂停 | `running`, `paused` |

## transform

CSS transform 属性允许你对元素进行变换操作，包括旋转、缩放、倾斜和平移。

### 常用变换函数

| 函数 | 说明 | 示例值 |
|------|------|--------|
| `translate()` | 平移元素（X, Y方向） | `translate(50px, 20px)` |
| `translateX()` | 水平方向平移 | `translateX(30px)` |
| `translateY()` | 垂直方向平移 | `translateY(-10px)` |
| `translateZ()` | Z轴方向平移（3D） | `translateZ(100px)` |
| `translate3d()` | 3D平移 | `translate3d(10px, 20px, 30px)` |
| `scale()` | 缩放元素 | `scale(1.5)`, `scale(2, 0.5)` |
| `scaleX()` | 水平方向缩放 | `scaleX(1.2)` |
| `scaleY()` | 垂直方向缩放 | `scaleY(0.8)` |
| `scaleZ()` | Z轴方向缩放（3D） | `scaleZ(2)` |
| `rotate()` | 旋转元素 | `rotate(45deg)` |
| `rotateX()` | 绕X轴旋转（3D） | `rotateX(30deg)` |
| `rotateY()` | 绕Y轴旋转（3D） | `rotateY(60deg)` |
| `rotateZ()` | 绕Z轴旋转（3D） | `rotateZ(90deg)` |
| `skew()` | 倾斜元素 | `skew(30deg, 20deg)` |
| `skewX()` | 水平方向倾斜 | `skewX(15deg)` |
| `skewY()` | 垂直方向倾斜 | `skewY(10deg)` |
| `matrix()` | 2D矩阵变换 | `matrix(1, 0, 0, 1, 0, 0)` |
| `matrix3d()` | 3D矩阵变换 | `matrix3d(1,0,0,0,0,1,0,0,0,0,1,0,0,0,0,1)` |

### 相关属性

| 属性 | 说明 | 示例值 |
|------|------|--------|
| `transform-origin` | 变换的基点 | `center`, `top left`, `50% 50%` |
| `transform-style` | 3D变换样式 | `flat`, `preserve-3d` |
| `perspective` | 3D透视距离 | `1000px` |
| `perspective-origin` | 透视点位置 | `center`, `top left` |


## transition

CSS transition 属性允许你为元素的样式变化添加过渡效果，使变化更平滑。

### 常用属性

| 属性 | 说明 | 示例值 |
|------|------|--------|
| `transition-property` | 指定要过渡的CSS属性 | `all`, `width`, `color`, `transform` |
| `transition-duration` | 过渡动画持续时间 | `1s`, `500ms`, `0.3s` |
| `transition-timing-function` | 过渡的速度曲线 | `ease`, `linear`, `ease-in`, `ease-out`, `cubic-bezier(0.1, 0.7, 1.0, 0.1)` |
| `transition-delay` | 过渡开始前的延迟时间 | `0s`, `200ms`, `1s` |

### 简写语法

```css
transition: property duration timing-function delay;
```

## link 和 @import 引用 CSS 的区别

link  
- 在 `<head>` 部分使用
- 在网页加载时立即加载样式表
- 并行进行，性能好  
- 可通过 JavaScript 操作 DOM

@import
- 在 CSS 文件或`<style>`标签内使用
- 在加载包含它的 CSS 文件后加载  
- 加载顺序依赖，速度慢  
- 不易通过 JavaScript 操作  

## 面试

### BFC

Block Formatting Context，块级格式化上下文  
  它决定了元素如何对其内容进行定位和与其他元素的关系和相互作用。  
  BFC是一个独立的渲染区域，它规定了内部元素如何布局，并且与外部元素相互隔离。  

布局规则：   
- 垂直排列  
- 外边距计算  
- 左边缘对齐  
- 不与浮动重叠  
- 高度计算  
- 独立性  

触发BFC  
- float 值不为 none  
- display 值为 inline-box、flex、grid  
- overflow 值部位 visible  

作用：  
- 解决浮动导致的高度塌陷  
- 防止外边距合并  
- 实现自适应两栏布局  
- 防止元素被浮动元素覆盖  

### Canvas 和 SVG 有什么区别

又是用于在网页上绘制图形的技术  
Canvas 是基于像素的即时绘制技术，适合频繁更新和复杂动画  
SVG 是基于矢量的图形格式，适合需要无损缩放和高分辨率的静态图形  

### CSS3 新特性

布局：  
- 容器查询
  允许根据容器尺寸而非视口调整元素样式，实现组件级响应式设计。  
  `@container (width > 600px) { ... }`  
- 子网格与命名区域  
  子网格：嵌套网格可继承父级对齐方式，简化复杂布局代码。  
  命名区域：通过语义化名称定义网格区域，如`grid-template-areas: "header main sidebar"`，提升代码可读性。  
- 动态视口单位  
  新增`dvh`（动态视口高度）、`lvc`（逻辑视口单位）等，适配折叠屏、刘海屏等异形设备  

颜色与视觉效果：  
- 广色域与 HDR 支持  
  新增`oklch()`、`color(display-p3)`等函数，支持P3色域和HDR显示设备。  
  HWB颜色模型：hwb(0 0% 30%)直接定义色调、白度与黑度，更符合人类直觉。  
- 颜色混合与函数扩展  
  `color-mix()`:混合两种颜色并控制比例，如color-mix(in srgb, red 40%, blue)  
  `@property`：注册自定义颜色属性，支持动画与类型验证  
- 背景滤镜  
  对元素背景区域应用模糊、对比度等滤镜，增强视觉层次感。  

交互与动画：  
- 滚动驱动动画（Scroll-Driven Animations）  
  动画进度与滚动深度绑定，如animation-timeline: scroll()实现视差效果  
- 视图过渡  
  跨文档或单页应用切换时生成平滑动画，支持多页面无缝跳转  
    语法：`::view-transition-old`和`::view-transition-new`控制过渡状态   
- CSS Toggles
  无JavaScript实现状态切换，如暗黑模式：`toggle-root: theme [light dark]`  

响应式与逻辑控制：  
- 逻辑属性  
- `:has()` 选择器  
- 媒体查询扩展  

作用域与模块化  
- `@scope`规则  
- 原生嵌套语法  

