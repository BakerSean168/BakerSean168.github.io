---
title: Cookie、Session、Token
categories:
  - HTTP
description: Cookie、Session、Token
date: 2025-01-20 09:30:43
tags:
---

HTTP 协议是无状态的，不保存任何信息。但是鉴权、获取对应用户数据等显然需要知道用户是谁等信息的。  
Cookie 就是最开始使用的用于保存信息的技术。

## Cookie

Cookie 是由服务器发送到用户浏览器并保存在本地的一小块数据，会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

### 特性

- 保存的是一段文本信息
- 大小受限，小于 4KB
- 由服务器设置，客户端存储
- 自动随 HTTP 请求发送
- 有域名和路径限制
- 支持过期时间设置

### Cookie 属性

| 属性              | 说明               | 示例                 |
| ----------------- | ------------------ | -------------------- |
| `Name=Value`      | Cookie 的名称和值  | `username=john`      |
| `Domain`          | 指定 Cookie 的域名 | `Domain=example.com` |
| `Path`            | 指定 Cookie 的路径 | `Path=/admin`        |
| `Expires/Max-Age` | 过期时间           | `Max-Age=3600`       |
| `HttpOnly`        | 防止 XSS 攻击      | `HttpOnly`           |
| `Secure`          | 只在 HTTPS 传输    | `Secure`             |
| `SameSite`        | 防止 CSRF 攻击     | `SameSite=Strict`    |

### 使用示例

<details>

<summary>实现代码</summary>

```javascript
// 设置 Cookie
document.cookie = "username=john; Max-Age=3600; Path=/; HttpOnly; Secure";

// 读取 Cookie
function getCookie(name) {
  const cookies = document.cookie.split(";");
  for (let cookie of cookies) {
    const [key, value] = cookie.trim().split("=");
    if (key === name) return value;
  }
  return null;
}

// 删除 Cookie
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/";
```

</details>

### 服务器端设置

```http
# HTTP 响应头
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
Set-Cookie: theme=dark; Max-Age=2592000; Path=/
```

## Session

Session 是一种服务器端的数据存储技术，用于存储用户会话信息。Session 数据存储在服务器端，通过 Session ID 来标识不同的会话。

### 工作机制

1. 用户首次访问，服务器创建 Session 并生成唯一 Session ID
2. Session ID 通过 Cookie 发送给客户端
3. 客户端后续请求携带 Session ID
4. 服务器根据 Session ID 查找对应的 Session 数据

### Session 生命周期

```mermaid
graph LR
    A[用户登录] --> B[服务器创建 Session]
    B --> C[返回 Session ID]
    C --> D[客户端存储 Session ID]
    D --> E[后续请求携带 Session ID]
    E --> F[服务器验证并获取 Session 数据]
    F --> G[Session 过期或用户登出]
    G --> H[销毁 Session]
```

### 服务器端实现（Node.js Express）

<details>

<summary>实现代码</summary>

```javascript
const session = require("express-session");

app.use(
  session({
    secret: "your-secret-key",
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: false, // HTTPS 环境设为 true
      httpOnly: true,
      maxAge: 1000 * 60 * 30, // 30 分钟
    },
  })
);

// 设置 Session
app.post("/login", (req, res) => {
  req.session.userId = user.id;
  req.session.username = user.username;
  res.json({ success: true });
});

// 获取 Session
app.get("/profile", (req, res) => {
  if (req.session.userId) {
    res.json({ userId: req.session.userId });
  } else {
    res.status(401).json({ error: "Not authenticated" });
  }
});

// 销毁 Session
app.post("/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      res.status(500).json({ error: "Could not log out" });
    } else {
      res.json({ success: true });
    }
  });
});
```

## Token

Token 是一种无状态的认证机制，通常是一个加密的字符串，包含了用户信息和过期时间等数据。

### JWT (JSON Web Token)

JWT 是目前最流行的 Token 实现方式，由三部分组成：

1. **Header**：包含算法和令牌类型
2. **Payload**：包含声明（用户信息）
3. **Signature**：用于验证令牌的签名

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Token 使用流程

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器

    C->>S: 1. 登录请求 (用户名/密码)
    S->>S: 2. 验证用户信息
    S->>C: 3. 返回 JWT Token
    C->>C: 4. 存储 Token
    C->>S: 5. 请求时携带 Token (Authorization Header)
    S->>S: 6. 验证 Token
    S->>C: 7. 返回数据
```

### 实现示例

#### 服务器端生成和验证 Token

<details>

<summary>code</summary>

```javascript
const jwt = require("jsonwebtoken");
const SECRET_KEY = "your-secret-key";

// 生成 Token
function generateToken(user) {
  const payload = {
    userId: user.id,
    username: user.username,
    exp: Math.floor(Date.now() / 1000) + 60 * 60, // 1小时过期
  };
  return jwt.sign(payload, SECRET_KEY);
}

// 验证 Token 中间件
function verifyToken(req, res, next) {
  const token = req.headers["authorization"]?.split(" ")[1]; // Bearer token

  if (!token) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// 登录接口
app.post("/login", async (req, res) => {
  // 验证用户信息...
  const token = generateToken(user);
  res.json({ token, user: { id: user.id, username: user.username } });
});

// 受保护的路由
app.get("/protected", verifyToken, (req, res) => {
  res.json({ message: "This is protected data", user: req.user });
});
```

</details>



#### 客户端使用 Token

<details>

<summary>code</summary>

```javascript
// 登录并获取 Token
async function login(username, password) {
  const response = await fetch("/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username, password }),
  });

  const data = await response.json();
  if (data.token) {
    localStorage.setItem("token", data.token);
  }
  return data;
}

// 携带 Token 的请求
async function fetchProtectedData() {
  const token = localStorage.getItem("token");

  const response = await fetch("/protected", {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  });

  return response.json();
}

// 登出（清除 Token）
function logout() {
  localStorage.removeItem("token");
}
```

</details>



## 对比

| 对比项         | Cookie           | Session                      | Token                                        |
| -------------- | ---------------- | ---------------------------- | -------------------------------------------- |
| **存储位置**   | 客户端（浏览器） | 服务器端                     | 客户端（通常在 localStorage/sessionStorage） |
| **存储大小**   | 4KB 限制         | 服务器内存/磁盘限制          | 无严格限制                                   |
| **安全性**     | 较低，可被篡改   | 较高，服务器端控制           | 中等，可加密但在客户端                       |
| **跨域支持**   | 受同源策略限制   | 受同源策略限制               | 支持跨域                                     |
| **服务器负载** | 无               | 有（存储会话数据）           | 无（无状态）                                 |
| **扩展性**     | 好               | 差（服务器集群需要会话同步） | 好（无状态）                                 |
| **过期控制**   | 支持             | 支持                         | 支持                                         |
