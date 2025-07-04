---
title: JSON-in-JS
categories:
  - 前端
description: JSON-in-JS
date: 2025-05-29 12:53:12
tags:
---

看到一篇文章[面对使我崩溃的字符串数组-笔试篇](https://juejin.cn/post/7143925756970663973)，发现作者将如下的字符串称为「字符串数组」

```js
let str1 = '["567",null,"u44","0",1,"eleven","ten","99"]';
console.log(typeof str1); // string
```

一个被 `''` 包裹的以字符串为主要数据的数组

我理解的字符串数组是长下面样子的：

```js
let str = "hello world";
```

仔细想想确实没问题，只不过 上面的字符串数组有点奇怪，`console.log(str[0]); // [`

其实 str1 是一个 JSON 格式的字符串

```js
let str_ori = ["567", null, "u44", "0", 1, "eleven", "ten", "99"];
let str_JSON = JSON.stringify(str_ori); // '["567",null,"u44","0",1,"eleven","ten","99"]';
// JSON.parse(str_JSON) // ["567",null,"u44","0",1,"eleven","ten","99"];
```

一般往往是见到 JSON 格式的对象：

```js
const obj = {
  name: "张三",
  age: 30,
  isStudent: false,
  hobbies: ["阅读", "游泳", "编程"],
};

// 将对象转换为JSON字符串
const jsonString = JSON.stringify(obj);

console.log(jsonString);
// 输出: '{"name":"张三","age":30,"isStudent":false,"hobbies":["阅读","游泳","编程"]}'
```

## JSON in JS

JSON 格式有什么用呢

定义：  
JSON (JavaScript Object Notation) 是 JavaScript 中处理数据交换的核心技术。

- 轻量级的数据交换格式，易于人阅读和编写，易于机器解析和生成。

所以 JSON 主要用于数据的传输，比如在前后端传输数据、本地数据存储时，往往使用 JSON 格式

### 核心 API

#### JSON.stringify()

JSON.stringify(value[, replacer[, space]])

- value：要转换的值
- replacer：  
  函数：转换每个属性  
  数组：指定包含的属性
- space：缩进格式（数字或字符串）

```js
const obj = {
  name: "李明",
  age: 28,
  skills: ["JavaScript", "Python"],
};

// 基本转换
console.log(JSON.stringify(obj));
// 输出: {"name":"李明","age":28,"skills":["JavaScript","Python"]}

// 带格式
console.log(JSON.stringify(obj, null, 2));
/*
{
  "name": "李明",
  "age": 28,
  "skills": [
    "JavaScript",
    "Python"
  ]
}
*/

// 使用replacer
console.log(JSON.stringify(obj, ["name", "skills"], 2));
/*
{
  "name": "李明",
  "skills": [
    "JavaScript",
    "Python"
  ]
}
*/
```

#### JSON.parse()

JSON.parse(text[, reviver])

- text: 要解析的 JSON 字符串
- reviver：转换函数，可修改解析结果

```js
const jsonStr = '{"name":"李明","age":28}';

// 基本解析
const parsed = JSON.parse(jsonStr);
console.log(parsed.name); // "李明"

// 使用reviver
const revived = JSON.parse(jsonStr, (key, value) => {
  return key === "age" ? value + 1 : value;
});
console.log(revived.age); // 29
```

### 特性

| JavaScript 值 | JSON 转换结果             |
| ------------- | ------------------------- |
| undefined     | 被忽略（数组中转为 null） |
| Function      | 被忽略                    |
| Symbol        | 被忽略                    |
| Date          | 转为 ISO 字符串           |
| Infinity/NaN  | 转为 null                 |
| RegExp        | 转为 {}                   |

### 高级用法

#### 自定义序列化

可定义 `toJSON()` 方法来控制  
```js
const customObj = {
  name: "自定义对象",
  secret: "不序列化",
  toJSON() {
    return { name: this.name };
  }
};

console.log(JSON.stringify(customObj)); // {"name":"自定义对象"}
```

### 实际应用

#### 前后端数据交互

```ts
// 发送数据到服务器
fetch('/api/user', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'user123',
    password: 'securePass'
  })
});

// 接收服务器响应
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

#### 本地存储

```js
// 存储到 localStorage
const settings = { theme: 'dark', fontSize: 14 };
localStorage.setItem('appSettings', JSON.stringify(settings));

// 从 localStorage 读取
const savedSettings = JSON.parse(localStorage.getItem('appSettings'));
```