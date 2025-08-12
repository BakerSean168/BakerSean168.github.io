---
title: JS中的BOM对象
categories:
  - JavaScript
description: JS中的BOM对象
date: 2025-01-18 15:42:26
tags:
---

# BOM

## 是什么

BOM 是使用 JavaScript 开发 Web 应用程序的核心。  
BOM 提供了与网页无关的浏览器功能对象。

BOM 的核心是 window 对象，表示浏览器的实例。  
window 对象在浏览器中有两重身份，一个是 ECMAScript 中的 Global 对象，另一个就是浏览器窗口的 JavaScript 接口。  
这意味着网页中定义的所有对象、变量和函数都以 window 作为其 Global 对象，都可以访问其上定义的 parseInt()等全局方法。  

## 使用

### 相关属性

window 对象有以下相关属性/方法，通过这些就可以实现响应的操作。  

| 属性/方法类别  | 名称                   | 说明                                   |
| -------------- | ---------------------- | -------------------------------------- |
| **窗口大小**   | `innerHeight`          | 浏览器窗口中页面视口的高度             |
|                | `innerWidth`           | 浏览器窗口中页面视口的宽度             |
|                | `outerHeight`          | 浏览器窗口外部的高度                   |
|                | `outerWidth`           | 浏览器窗口外部的宽度                   |
| **位置信息**   | `screenLeft`/`screenX` | 窗口相对于屏幕左侧的位置               |
|                | `screenTop`/`screenY`  | 窗口相对于屏幕顶部的位置               |
|                | `scrollX`              | 页面水平方向滚动的像素值               |
|                | `scrollY`              | 页面垂直方向滚动的像素值               |
| **导航方法**   | `navigate(URL)`        | 导航到指定 URL                         |
|                | `back()`               | 后退一页                               |
|                | `forward()`            | 前进一页                               |
|                | `reload()`             | 重新加载当前页面                       |
| **系统对话框** | `alert()`              | 显示带有一条消息和确认按钮的警告框     |
|                | `confirm()`            | 显示带有消息以及确认和取消按钮的对话框 |
|                | `prompt()`             | 显示可提示用户输入的对话框             |
| **定时器**     | `setTimeout()`         | 在指定的毫秒数后执行函数               |
|                | `setInterval()`        | 每隔指定的毫秒数执行函数               |
|                | `clearTimeout()`       | 取消 setTimeout 设置的定时器           |
|                | `clearInterval()`      | 取消 setInterval 设置的定时器          |
| **全局对象**   | `document`             | 指向 Document 对象                     |
|                | `location`             | 指向 Location 对象                     |
|                | `history`              | 指向 History 对象                      |
|                | `navigator`            | 指向 Navigator 对象                    |
|                | `screen`               | 指向 Screen 对象                       |

### location 对象

location 是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。

它既是 window 的属性，也是 document 的属性。  
也就是说，window.location 和 document.location 指向同一个对象。

| 属性/方法     | 说明                                    | 示例                                  |
| ------------- | --------------------------------------- | ------------------------------------- |
| **href**      | 完整的 URL                              | `https://www.example.com:80/path?q=1` |
| **protocol**  | 使用的协议                              | `https:`                              |
| **host**      | 主机名和端口号                          | `www.example.com:80`                  |
| **hostname**  | 主机名                                  | `www.example.com`                     |
| **port**      | 端口号                                  | `80`                                  |
| **pathname**  | URL 中的路径部分                        | `/path`                               |
| **search**    | URL 的查询字符串                        | `?q=1`                                |
| **hash**      | URL 的锚部分（包括#号）                 | `#section1`                           |
| **origin**    | URL 的协议、主机名和端口                | `https://www.example.com:80`          |
| **assign()**  | 加载新的 URL，会生成历史记录            | `location.assign("newpage.html")`     |
| **replace()** | 用新 URL 替换当前 URL，不会生成历史记录 | `location.replace("newpage.html")`    |
| **reload()**  | 重新加载当前页面                        | `location.reload()`                   |

#### 查询字符串

老旧方法

```js
let getQueryStringArgs = function () {
  // 取得没有开头问号的查询字符串
  保存数据的对象;
  let qs = location.search.length > 0 ? location.search.substring(1) : "",
    //
    args = {};
  // 把每个参数添加到args对象
  for (let item of qs.split("&").map((kv) => kv.split("="))) {
    let name = decodeURIComponent(item[0]),
      value = decodeURIComponent(item[1]);
    if (name.length) {
      args[name] = value;
    }
  }
  return args;
};
```

URLSearchParams 提供了一组标准 API 方法，通过它们可以检查和修改查询字符串。  
给 URLSearchParams 构造函数传入一个查询字符串，就可以创建一个实例。这个实例上暴露了 get()、set()和 delete()等方法，可以对查询字符串执行相应操作。

#### 操作地址

可以通过修改 location 对象修改浏览器的地址。

1. location.assign("http://www.wrox.com");
2. window.location = "http://www.wrox.com";
3. location.href = "http://www.wrox.com";

_除了 hash 之外，只要修改 location 的一个属性，就会导致页面重新加载新 URL_

**replace()**
如果不希望增加历史记录，可以使用 replace()方法。  
这个方法接收一个 URL 参数，但重新加载后不会增加历史记录。  
调用 replace()之后，用户不能回到前一页。

**reload()**
能重新加载当前显示的页面。  
调用 reload()而不传参数，页面会以最有效的方式重新加载。也就是说，如果页面自上次请求以来没有修改过，浏览器可能会从缓存中加载页面。  
如果想强制从服务器重新加载，可以像下面这样给 reload()传个 true

```js
location.reload(); // 重新加载，可能是从缓存加载
location.reload(true); // 重新加载，从服务器加载
```

### navigator 对象

navigator 对象的属性通常用于确定浏览器的类型。

检测插件  
注册处理程序

### screen 对象

这个对象中保存的纯粹是客户端能力信息。  
也就是浏览器窗口外面的客户端显示器的信息，比如像素宽度和像素高度。  
每个浏览器都会在 screen 对象上暴露不同的属性。

### history 对象

history 对象表示当前窗口首次使用以来用户的导航历史记录。  
因为 history 是 window 的属性，所以每个 window 都有自己的 history 对象。  
出于安全考虑，这个对象不会暴露用户访问过的 URL，但可以通过它在不知道实际 URL 的情况下前进和后退。

#### 导航

go()方法可以在用户历史记录中沿任何方向导航，可以前进也可以后退。

history 对象还有一个 length 属性，表示历史记录中有多个条目。

#### 历史状态管理

hashchange 事件
