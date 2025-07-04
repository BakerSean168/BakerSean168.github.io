---
title: RESTful-API
categories:
  - null
description: RESTful-API
date: 2025-07-03 14:11:27
tags:
---

# RESTful API 设计规范

## 1. 核心概念

REST (Representational State Transfer) 是一种软件架构风格，不是标准也不是协议，而是一组设计原则和约束条件。RESTful API 是基于 REST 原则设计的 Web API。

## 2. 主要设计规范

### (1) 资源导向

- 所有事物都抽象为资源，使用名词而非动词
- 使用复数形式表示资源集合，如`/users`而非`/user`
- 资源使用 URI 唯一标识，如`/users/123`

### (2) HTTP 方法语义化

- **GET**：获取资源（安全且幂等）
- **POST**：创建资源（非幂等）
- **PUT**：完整更新资源（幂等）
- **PATCH**：部分更新资源
- **DELETE**：删除资源（幂等）

### (3) 状态码规范

- 2xx 成功：
  - 200 OK - 通用成功
  - 201 Created - 资源创建成功
  - 204 No Content - 成功但无返回体
- 3xx 重定向
- 4xx 客户端错误：
  - 400 Bad Request - 请求错误
  - 401 Unauthorized - 未认证
  - 403 Forbidden - 无权限
  - 404 Not Found - 资源不存在
- 5xx 服务端错误：
  - 500 Internal Server Error - 通用服务端错误
  - 503 Service Unavailable - 服务不可用

### (4) 版本控制

- URI 中包含版本：`/api/v1/users`
- 或通过 Accept 头指定：`Accept: application/vnd.myapi.v1+json`

### (5) 过滤、排序、分页

- 过滤：`/users?role=admin`
- 排序：`/users?sort=-created_at,username`
- 分页：`/users?page=2&limit=20`

### (6) 响应格式

- 统一使用 JSON 格式
- 包含资源本身和可能的元数据
- 错误响应标准化：
  ```json
  {
    "error": {
      "code": "invalid_param",
      "message": "Invalid email format",
      "details": {
        "email": "Should be a valid email address"
      }
    }
  }
  ```

### (7) 超媒体(HATEOAS)

- 响应中包含相关资源的链接
- 示例：
  ```json
  {
    "id": 123,
    "name": "John",
    "links": [
      {
        "rel": "self",
        "href": "/users/123"
      },
      {
        "rel": "friends",
        "href": "/users/123/friends"
      }
    ]
  }
  ```

## 3. 其他最佳实践

1. **安全性**：

   - 使用 HTTPS
   - 敏感数据不在 URL 中传递
   - 适当的认证机制（如 JWT、OAuth）

2. **性能**：

   - 支持缓存（ETag、Last-Modified）
   - 考虑使用 gzip 压缩
   - 避免过度获取数据（支持字段选择）

3. **文档**：

   - 提供清晰、完整的 API 文档
   - 使用 Swagger/OpenAPI 等工具

4. **命名规范**：
   - 使用小写字母和连字符`-`（而非下划线`_`）
   - 保持一致性

## 4. RESTful 与非 RESTful 对比

**RESTful**:

```
GET /users/123
PUT /users/123
DELETE /users/123
```

**非 RESTful**:

```
GET /getUser?id=123
POST /updateUser
POST /deleteUser
```

作为开发者，我会遵循这些规范来设计易于理解、维护和扩展的 API，同时也会根据实际业务需求进行适当调整，在规范性和实用性之间取得平衡。
