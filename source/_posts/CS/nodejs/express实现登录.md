---
title: express实现登录
categories:
  - nodejs
description: express实现登录
date: 2025-05-13 15:31:53
tags:
---

## 1. 内存中存储数据

需要的库  
`npm install jsonwebtoken bcryptjs`
- nodemon  
  检测代码变化，自动重启服务（类似热更新）  

package.json 中添加运行脚本`"dev": "nodemon app.js",` 使用 nodemon 运行项目来使用热更新。  

使用 AI 写一个简单的在内存中存储数据的 demo，测试运行：  
```js
const express = require('express')
const jwt = require('jsonwebtoken')
const bcrypt = require('bcryptjs')
const app = express()
const port = 3000

// 添加 JSON 解析中间件
app.use(express.json())

// 模拟用户数据库
const users = []
const JWT_SECRET = 'your-secret-key'

// 注册路由
app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.post('/register', async (req, res) => {
  try {
    const { username, password } = req.body
    
    // 检查用户是否已存在
    if (users.find(u => u.username === username)) {
      return res.status(400).json({ message: '用户已存在' })
    }

    // 加密密码
    const hashedPassword = await bcrypt.hash(password, 10)
    
    // 保存用户
    users.push({
      username,
      password: hashedPassword
    })

    res.status(201).json({ message: '注册成功' })
  } catch (error) {
    res.status(500).json({ message: '服务器错误' })
  }
})

// 登录路由
app.post('/login', async (req, res) => {
  try {
    const { username, password } = req.body
    
    // 查找用户
    const user = users.find(u => u.username === username)
    if (!user) {
      return res.status(400).json({ message: '用户不存在' })
    }

    // 验证密码
    const validPassword = await bcrypt.compare(password, user.password)
    if (!validPassword) {
      return res.status(400).json({ message: '密码错误' })
    }

    // 生成 JWT token
    const token = jwt.sign({ username: user.username }, JWT_SECRET, {
      expiresIn: '1h'
    })

    res.json({ token })
  } catch (error) {
    res.status(500).json({ message: '服务器错误' })
  }
})

// 获取所有用户
app.get('/users', (req, res) => {
  res.json(users)
})

// 验证中间件
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization']
  const token = authHeader && authHeader.split(' ')[1]

  if (!token) {
    return res.status(401).json({ message: '未提供认证令牌' })
  }

  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ message: '令牌无效' })
    }
    req.user = user
    next()
  })
}

// 受保护的路由示例
app.get('/protected', authenticateToken, (req, res) => {
  res.json({ message: '这是受保护的内容', user: req.user })
})

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`)
})

```

使用 postman 测试一下 API，可以正常返回  

## 2. 加入数据库

需要的库  
- mysql2
  使用 Mysql8 需要  
  支持 Promise  

### 数据库配置

#### 创建数据库和表

**创建一个相应的文件，使用脚本来生成**  
```js
// script/create_database.js
// 该脚本用于创建数据库和表
const mysql = require('mysql2/promise');

async function initDatabase() {
  try {
    // 首先创建与MySQL的连接（不指定数据库）
    const connection = await mysql.createConnection({
      host: 'localhost',
      user: 'root',
      password: 'Sichuan168@'  // 替换为你的MySQL密码
    });

    // 创建数据库
    await connection.query('CREATE DATABASE IF NOT EXISTS dailyuse');
    
    // 切换到新创建的数据库
    await connection.query('USE dailyuse');
    
    // 创建用户表
    await connection.query(`
      CREATE TABLE IF NOT EXISTS users (
        id INT PRIMARY KEY AUTO_INCREMENT,
        username VARCHAR(255) NOT NULL UNIQUE,
        password VARCHAR(255) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);

    console.log('Database and tables created successfully!');
    await connection.end();
  } catch (error) {
    console.error('Error initializing database:', error);
    process.exit(1);
  }
}

initDatabase();
```

**添加脚本"init_db": "node scripts/create_database.js"**


npm run init_db  // 使用脚本来生成  

Database and tables created successfully! // 控制台打印表示成功  

#### 连接数据库

```js
// config/database.js
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: 'localhost',
  user: 'user',
  password: 'password', 
  database: 'name',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

module.exports = pool;

```

新代码：  

```js
// app.js
const express = require('express')
const jwt = require('jsonwebtoken')
const bcrypt = require('bcryptjs')
const db = require('./config/database') // 引入数据库配置
const app = express()
const user = express.Router()
const port = 3000

// 添加 JSON 解析中间件
app.use(express.json())

const JWT_SECRET = 'your-secret-key'

// 测试路由
user.get('/', (req, res) => {
  res.send('Hello World!')
})

// 注册路由
user.post('/register', async (req, res) => {
  try {
    const { username, password } = req.body
    
    // 检查用户是否已存在
    const [users] = await db.execute(
      'SELECT * FROM users WHERE username = ?',
      [username]
    )
    
    if (users.length > 0) {
      return res.status(400).json({ message: '用户已存在' })
    }

    // 加密密码
    const hashedPassword = await bcrypt.hash(password, 10)
    
    // 保存用户
    await db.execute(
      'INSERT INTO users (username, password) VALUES (?, ?)',
      [username, hashedPassword]
    )

    res.status(201).json({ message: '注册成功' })
  } catch (error) {
    console.error(error)
    res.status(500).json({ message: '服务器错误' })
  }
})

// 登录路由
user.post('/login', async (req, res) => {
  try {
    const { username, password } = req.body
    
    // 查找用户
    const [users] = await db.execute(
      'SELECT * FROM users WHERE username = ?',
      [username]
    )
    
    if (users.length === 0) {
      return res.status(400).json({ message: '用户不存在' })
    }

    const user = users[0]

    // 验证密码
    const validPassword = await bcrypt.compare(password, user.password)
    if (!validPassword) {
      return res.status(400).json({ message: '密码错误' })
    }

    // 生成 JWT token
    const token = jwt.sign(
      { userId: user.id, username: user.username },
      JWT_SECRET,
      { expiresIn: '1h' }
    )

    res.json({ token })
  } catch (error) {
    console.error(error)
    res.status(500).json({ message: '服务器错误' })
  }
})

// 获取所有用户
user.get('/users', async (req, res) => {
  try {
    const [users] = await db.execute(
      'SELECT id, username, created_at FROM users'
    )
    res.json(users)
  } catch (error) {
    console.error(error)
    res.status(500).json({ message: '服务器错误' })
  }
})

// 验证中间件
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization']
  const token = authHeader && authHeader.split(' ')[1]

  if (!token) {
    return res.status(401).json({ message: '未提供认证令牌' })
  }

  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ message: '令牌无效' })
    }
    req.user = user
    next()
  })
}

// 受保护的路由示例
user.get('/protected', authenticateToken, (req, res) => {
  res.json({ message: '这是受保护的内容', user: req.user })
})

app.use('/api', user)

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`)
})
```

- 创建了数据库连接池而不是单个连接
- 将用户数据存储从内存数组改为数据库表
- 使用参数化查询防止 SQL 注入
- 添加了错误处理和日志记录
- 使用了 router，使 API 更有组织性
  现在 API 都以 /api 开头  

使用 postman 测试，成功