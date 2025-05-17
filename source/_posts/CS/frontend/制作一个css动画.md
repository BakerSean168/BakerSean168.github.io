---
title: 制作一个css动画
categories:
  - 前端
  - CSS
description: 制作一个简单的 火车从左跑到右 的css动画
date: 2025-05-17 18:21:39
tags:
---

# 1. 生成容器

```html
<!-- 添加火车动画容器 -->
<div class="train-container">
  <div class="train">
    <!-- 车厢形状 -->
    <div class="locomotive">
      <div class="square1" data-letter="Daily"></div>
      <div class="square2" data-letter="Use"></div>
      <div class="engine">
        <div class="engine-light"></div>
      </div>
    </div>

    <!-- 8个轮子 -->
    <div class="wheels">
      <div class="wheel" data-letter="d"></div>
      <div class="wheel" data-letter="a"></div>
      <div class="wheel" data-letter="i"></div>
      <div class="wheel" data-letter="l"></div>
      <div class="wheel" data-letter="y"></div>
      <div class="wheel" data-letter="u"></div>
      <div class="wheel" data-letter="s"></div>
      <div class="wheel" data-letter="e"></div>
    </div>
  </div>
</div>
```

# 2. 完善样式

## 生成零部件

```css
.square1 {
  width: 160px;
  height: 60px;
  background: #4CAF50;
  border-radius: 8px;
}

/* 车头样式 */
.engine {
  position: relative;
  width: 80px;
  height: 60px;
  background: #2196F3;
  border-radius: 8px;
  clip-path: polygon(0 0, 100% 20%, 100% 100%, 0 100%);
}
```

### clip-path

`clip-path: polygon(x1 y1, x2 y2, ..., xn yn);`  
- 其中每一对 x y 值代表多边形的一个顶点坐标，坐标点之间用逗号分隔。  
- 左上角为（0，0）  


## 布局、组装

通过布局样式，将图形组装成火车  

# 3. 添加动画

```css
.train {
  position: absolute;
  bottom: 50px;
  left: -200px; 
  display: flex;
  flex-direction: column;
  animation: moveTrain 15s linear infinite;
}

@keyframes moveTrain {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(calc(100vw + 200px));
  }
}
```

# 整体代码

```css
/* 火车动画相关样式 */
.train-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

/* 火车 */
.train {
  position: absolute;
  bottom: 50px;
  left: -350px;
  /* 初始位置在屏幕外 */
  display: flex;
  flex-direction: column;
  flex-wrap: nowrap;

  /* animation: moveTrain 15s linear infinite; */
  animation: moveTrain 10s linear infinite;
}

/* 车厢 */
.locomotive {
    display: inline-flex;
    align-items: center;
    gap: 5px;
  }

.square1 {
  position: relative;
  width: 160px;
  height: 60px;
  background: #4CAF50;
  border-radius: 8px;
}

.square1::before {
  content: attr(data-letter);
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: bold;
  font-size: 24px;
}

.square2 {
  position: relative;
  width: 100px;
  height: 60px;
  background: #FF9800;
  border-radius: 8px;
}

.square2::before {
  content: attr(data-letter);
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: bold;
  font-size: 24px;
}

.circle {
  width: 60px;
  height: 60px;
  background: #2196F3;
  border-radius: 50%;
}

.triangle {
  width: 0;
  height: 0;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  border-bottom: 60px solid #F44336;
}

/* 车头样式 */
.engine {
  position: relative;
  width: 80px;
  height: 60px;
  background: #2196F3;
  border-radius: 8px;
  clip-path: polygon(0 0, 50% 0, 100% 60%, 100% 100%, 0 100%);
}

/* 车头灯 */
.engine-light {
  position: absolute;
  width: 15px;
  height: 15px;
  background: #FFD700;
  border-radius: 50%;
  right: 0;
  top: 70%;
  transform: translateY(-50%);
  box-shadow: 0 0 10px #FFD700;
  animation: glowLight 1s ease-in-out infinite alternate;
}

.wheels {
  display: flex;
  gap: 15px;
  margin-top: 10px;
}

.wheel {
  width: 30px;
  height: 30px;
  background: #9C27B0;
  border-radius: 50%;
  position: relative;
  animation: rotateWheel 2s linear infinite;
}

.wheel::before {
  content: attr(data-letter);
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: bold;
  font-size: 16px;
}

@keyframes moveTrain {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(calc(100vw + 350px));
  }
}

@keyframes rotateWheel {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

```