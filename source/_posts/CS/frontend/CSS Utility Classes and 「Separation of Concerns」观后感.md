---
title: CSS Utility Classes and 「Separation of Concerns」观后感
categories:
  - null
description: new post CSS Utility Classes and 「Separation of Concerns」观后感
date: 2025-05-28 16:11:05
tags:
---

[原文链接](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)

文章讲述了作者 CSS 写法的变化及其原因

# 语义 CSS

**第一层**

有说法，CSS 的最好实践是关注点分离：  
HTML、CSS、JavaScript 各司其职，HTML 只包含内容，CSS 决定样式，JavaScript 决定行为

而语义化 CSS 是关注点分离的一个实现方法

```html
<!-- text-center 类名是设计决策，不符合语义化CSS -->
<p class="text-center">Hello there!</p>

<!-- greeting 类名是元素内容,符合语义化CSS -->
<p class="greeting">Hello there!</p>
```

优势：  
说是可以在重新设计新网站时，只需要更换一个样式表  
[一个应用不同样式来重新设计网站的网站](https://www.csszengarden.com/)

## 例子

在最外层使用语义化的类名，内部按照 HTML 的结构来组织 CSS 样式

```HTML
.author-bio {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
  > img {
    display: block;
    width: 100%;
    height: auto;
  }
  > div {
    padding: 1rem;
    > h2 {
      font-size: 1.25rem;
      color: rgba(0,0,0,0.8);
    }
    > p {
      font-size: 1rem;
      color: rgba(0,0,0,0.75);
      line-height: 1.5;
    }
  }
}



body {
  background-color: #eee;
}

.container {
  max-width: 350px;
  margin: 2rem auto;
}
```

```SCSS
.author-bio {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
  > img {
    display: block;
    width: 100%;
    height: auto;
  }
  > div {
    padding: 1rem;
    > h2 {
      font-size: 1.25rem;
      color: rgba(0,0,0,0.8);
    }
    > p {
      font-size: 1rem;
      color: rgba(0,0,0,0.75);
      line-height: 1.5;
    }
  }
}



body {
  background-color: #eee;
}

.container {
  max-width: 350px;
  margin: 2rem auto;
}
```

# 将风格与结构分离

原作者认为上面的 CSS 写法会增加选择器的特异性（不断嵌套选择器），对 DOM 结构的依赖增加

**第二层**

又找到了新的写法：  
添加更多的类（给大量标签都添加一个类名）  
这样的话就减少了子元素选择器的使用

eg：

```css
.author-bio {
  background-color: white;
  border: 1px solid hsl(0, 0%, 85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}
.author-bio__image {
  display: block;
  width: 100%;
  height: auto;
}
.author-bio__content {
  padding: 1rem;
}
.author-bio__name {
  font-size: 1.25rem;
  color: rgba(0, 0, 0, 0.8);
}
.author-bio__body {
  font-size: 1rem;
  color: rgba(0, 0, 0, 0.75);
  line-height: 1.5;
}
```

主张改思想的最著名的方法是：  
块元素修改器（Block Element Modifer）

**第三层**

如上面的例子，固定了 author 的语义，有相同布局的语义应该怎么办？

1. 为 article 的例子逐个更改前缀
2. 使用 `@extend` 扩展

```CSS
.article-preview {
@extend .author-bio;
}
```

3. 创建内容留空的组件，将类名抽象

```CSS
.media-card {
 background-color: white;
 border: 1px solid hsl(0,0%,85%);
 border-radius: 4px;
 box-shadow: 0 2px 4px rgba(0,0,0,0.1);
 overflow: hidden;
}
.media-card__image {
 display: block;
 width: 100%;
 height: auto;
}
.media-card__content {
 padding: 1rem;
}
.media-card__title {
 font-size: 1.25rem;
 color: rgba(0,0,0,0.8);
}
.media-card__body {
 font-size: 1rem;
 color: rgba(0,0,0,0.75);
 line-height: 1.5;
}
```

第一种方法太烦

第二种方法，似乎[通常不推荐使用 @extend](https://csswizardry.com/2014/11/when-to-use-extend-when-to-use-a-mixin/)

第三种方法：  
似乎能很好地解决问题
但是当我们想改变单一（作者）页面的样式，不改变文章页面样式（重新编写HTML）  
或者新增一种语义不被 media 包括但样式相同的样式时（为新内容创建新的CSS组件）  

相反如果不专注与分离「关注点分离」，似乎以上两者都不需要编辑 CSS 文件。  
由是观之，关注点分离不一定能更高效，让代码的改动更简洁。  

原作者认为「关注点分离」这个问题并不是非黑即白的，应该考虑「依赖方向」：  
- CSS 依赖于 HTML
  HTML 是独立的，他暴露了可被控制的钩子`.author-bio`  
  CSS 是不独立的，他要知道 HTML 暴露了哪些钩子  
- HTML 依赖于 CSS
  CSS 是独立的，提供一组标记集合  
  HTML 是不独立的，他要知道 CSS 提供哪些样式并组合  

# 与内容无关的 CSS 组件

**第四层**

类似 `.btn`、`.card` 等  

一个组件做的越多，或者一个组件越具体，就越难重用。


```html
<form class="stacked-form" action="#">
  <div class="stacked-form__section">
    <!-- ... -->
  </div>
  <div class="stacked-form__section">
    <!-- ... -->
  </div>
  <div class="stacked-form__section">
    <button class="stacked-form__button">Submit</button>
  </div>
</form>
```

总结：  
针对某一场景使用一系列的样式  
去除掉特定的类名如上面的 stack-form，使用抽象类  
用子组件组合成组件  

# 内容无关组件+实用工具类

**第五层**

针对某些简单的样例，为什么不直接使用更简单的方式

比如对于某一个左对齐的组件，使用了 `.actions-list--left`  
当我们需要一个相同的组件，但是右对齐时，我们需要新创建一个 `.actions-list--right`的组件  
**We prefer composition to duplication**  
解决方式就是引入如下实用工具类，然后组装 `class="actions-list align-left"`  
```css
.align-left {
  text-align: left;
}
.align-right {
  text-align: right;
}
```

# 实用工具类优先

**第六层**

直接使用实用工具类来制作UI（不需要编写CSS）  

```html
<div class="card rounded shadow">
    <a href="..." class="block">
        <img class="block fit" src="...">
    </a>
    <div class="py-3 px-4 border-b border-dark-soft flex-spaced flex-y-center">
        <div class="text-ellipsis mr-4">
            <a href="..." class="text-lg text-medium">
                Test-Driven Laravel
            </a>
        </div>
        <a href="..." class="link-softer">
            @icon('link')
        </a>
    </div>
    <div class="flex text-lg text-dark">
        <div class="py-2 px-4 border-r border-dark-soft">
            @icon('currency-dollar', 'icon-sm text-dark-softest mr-4')
            <span>$3,475</span>
        </div>
        <div class="py-2 px-4">
            @icon('user', 'icon-sm text-dark-softest mr-4')
            <span>25</span>
        </div>
    </div>
</div>
```

当有需要多次重复使用由多个实用工具类组成的样式组时，可以将其提取为CSS组件  

# 总结

原作者针对「关注点分离」的主流思想，结合自己的实际开发经历提出新的看法：  
「关注点分离」原则与「实用工具类」两者本质是「依赖方向」的选择，而非绝对的对错。  

关注点分离特点是：  
CSS 需要根据 HTML 的结构和语义化类名来编写  
切换设计风格只需要切换CSS文件

实用工具类方法特点则是：  
HTML 调用原子化的、预设的 CSS 样式来设计 UI  

前者设计的网站应该是偏向长期使用、设计多样的网站  
后者则往往偏向样式类似，快速开发的网站  

现在往往倾向于快速开发，所以使用后者的人应该较多（猜测），组件库也是基于后者  
