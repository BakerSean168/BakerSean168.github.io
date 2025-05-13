---
title: express体验
categories:
  - nodejs
description: express体验
date: 2025-05-12 22:23:40
tags: 后端框架
---

# 1. 安装配置

初始化一个 npm 项目 然后 install express  
```bash
npm init

npm install express
```

# 2. Hello World

创建 app.js
```js
// app.js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```

`$ node app.js` 运行

然后，在浏览器中加载`http://127.0.0.1:3000/`以查看输出。

# 基本路由

路由用于确定应用程序如何响应对特定端点的客户机请求，包含一个 URI（或路径）和一个特定的 HTTP 请求方法（GET、POST 等）。  
每个路由可以具有一个或多个处理程序函数，这些函数在路由匹配时执行。  
`app.METHOD(PATH, HANDLER)`
- app 是 express 的实例。
- METHOD 是 HTTP 请求方法。
- PATH 是服务器上的路径。
- HANDLER 是在路由匹配时执行的函数。

```js
app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.post('/', (req, res) => {
  res.send('Got a POST request')
})

app.put('/user', (req, res) => {
  res.send('Got a PUT request at /user')
})

app.delete('/user', (req, res) => {
  res.send('Got a DELETE request at /user')
})
```