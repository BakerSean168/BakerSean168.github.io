---
title: box-shadow详解
categories:
  - 前端
  - CSS
description: box-shadow详解
date: 2025-05-18 20:51:59
tags:
---

CSS 的 box-shadow 属性是一个功能强大的工具，可以为网页元素添加丰富的阴影效果，增强视觉层次感和立体感。  

## 基础语法

`box-shadow: [inset] <offset-x> <offset-y> [<blur-radius>] [<spread-radius>] [<color>];`

- inset  
  将默认的外部阴影改为内部阴影。  
  使用此关键字时，阴影会出现在元素边框内部、背景之上、内容之下  
- offset-x offset-y  
  设置阴影偏移量  
  正值为右下，负值为左上  
- blur-radius  
  设置阴影糊程度  
  值越大越模糊  
  模糊算法：以阴影边缘为中心，向内外各延伸模糊半径的一半，创建颜色过渡  
- spread-radius  
  控制阴影的扩展/收缩  
- color  
  设置阴影颜色  
  未指定时通常使用当前文本颜色  

*可以在一个元素上应用多个阴影，用逗号分隔*  

应用顺序：先根据偏移量定位 → 应用扩散半径 → 最后应用模糊效果  

## 使用技巧

### 单边阴影

调整偏移量和扩散半径，可以创建单边阴影效果：  

```css
/* 顶部阴影 */
.top { box-shadow: 0px -5px 10px -5px red; }

/* 右侧阴影 */
.right { box-shadow: 5px 0 10px -5px red; }

/* 底部阴影 */
.bottom { box-shadow: 0 5px 10px 10px -5px red; }

/* 左侧阴影 */
.left { box-shadow: -5px 0 10px -5px red; }
```

### 多边组合阴影

```css
/* 无顶部阴影 */
.no-top {
  box-shadow: 
    -5px 5px 10px -4px red, 
    5px 5px 10px -4px red;
}

/* 无右侧阴影 */
.no-right {
  box-shadow: 
    -5px -5px 10px -4px red, 
    -5px 5px 10px -4px red;
}
```

### 拟物化设计

利用内外阴影组合可以创建拟物化按钮效果：  

```css
.button {
  box-shadow: 
    inset 0 -3em 3em rgba(0,0,0,0.1),
    0 0 0 2px rgb(255,255,255),
    0.3em 0.3em 1em rgba(0,0,0,0.3);
}
```

[拟物化设计样式网站](https://neumorphism.io/)

### 高亮效果

通过超大扩散半径的阴影实现蒙版高亮效果：  
```css
.guide {
  box-shadow: 0 0 0 9999px rgba(0, 0, 0, .75);
  border-radius: 50%;
}
```

### 图片悬停效果

```css
img {
  transition: all 0.3s ease;
}
img:hover {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  transform: scale(1.1);
}
```