---
title: express项目结构
categories:
  - 后端
  - express
  - nodejs
description:  express 项目的组织结构有哪些
date: 2025-05-18 22:57:37
tags:
---

## 基础分层结构（MVC 衍生）

适用于中小型项目，强调路由、控制器、模型的分层：  
```bash
project-root/
├── src/
│   ├── controllers/        # 控制器层：处理请求逻辑
│   ├── models/            # 模型层：数据库模型和业务逻辑
│   ├── views/             # 视图层：模板文件（如果有前端视图）
│   ├── routes/            # 路由定义
│   ├── middlewares/       # 中间件
│   └── app.js            # 应用入口文件
```

特点：  
- 路由层：仅定义 HTTP 端点，调用控制器 
- 控制器层：处理请求参数，调用服务或模型，返回响应 
- 模型层：封装数据库操作（如 Mongoose 模型）。

## 进阶分层结构（服务层解耦）

[csdn](https://blog.csdn.net/lj_1598/article/details/142848266)

适用于复杂业务逻辑的项目，引入服务层和中间件：
```bash
project-root/
├── src/
│   ├── api/             # API 层
│   │   ├── controllers/
│   │   ├── routes/
│   │   └── middlewares/
│   ├── services/        # 服务层
│   ├── repositories/    # 数据访问层
│   └── models/         # 数据模型层
```

关键改进：
服务层：将业务逻辑从控制器剥离，避免“面条代码”   

例如：  
```JavaScript
// services/userService.js
class UserService {
  async signup(userData) {
    const user = await UserModel.create(userData);
    eventEmitter.emit('user_signup', user); // 发布事件
    return user;
  }
}
```
中间件：集中处理认证（如 JWT）、请求日志等 
事件驱动：通过 eventEmitter 解耦后续操作（如发送邮件、记录日志） 

## 模块化结构（按功能划分）

适用于大型项目，每个功能模块自成体系：  
```bash
DailyUse-backend/
├── src/
│   ├── config/                    # 配置文件目录
│   │   ├── database.js           # 数据库配置
│   │   ├── redis.js             # Redis配置
│   │   ├── logger.js            # 日志配置
│   │   └── constants.js         # 常量配置
│   ├── modules/                  # 功能模块目录
│   │   ├── auth/                # 认证模块
│   │   │   ├── auth.controller.js
│   │   │   ├── auth.middleware.js
│   │   │   ├── auth.routes.js
│   │   │   └── auth.service.js
│   │   ├── users/               # 用户模块
│   │   │   ├── user.controller.js
│   │   │   ├── user.model.js
│   │   │   ├── user.routes.js
│   │   │   └── user.service.js
│   │   └── products/            # 产品模块
│   │       ├── product.controller.js
│   │       ├── product.model.js
│   │       ├── product.routes.js
│   │       └── product.service.js
│   ├── middlewares/             # 全局中间件
│   │   ├── error-handler.js
│   │   ├── request-logger.js
│   │   └── validator.js
│   ├── utils/                   # 工具函数
│   │   ├── helpers.js
│   │   ├── validators.js
│   │   └── formatters.js
│   ├── types/                   # TypeScript类型定义
│   │   └── index.d.ts
│   ├── docs/                    # API文档
│   │   └── swagger.json
│   └── app.js                   # 应用入口文件
├── tests/                       # 测试文件目录
│   ├── unit/                    # 单元测试
│   │   ├── auth.test.js
│   │   └── user.test.js
│   ├── integration/             # 集成测试
│   │   ├── auth.test.js
│   │   └── user.test.js
│   └── fixtures/                # 测试数据
│       └── users.json
├── scripts/                     # 脚本文件
│   ├── db/
│   │   ├── migrations/         # 数据库迁移
│   │   │   ├── 20240519_create_users.js
│   │   │   └── 20240519_create_products.js
│   │   └── seeds/             # 种子数据
│   │       └── init.js
│   └── deploy/                 # 部署脚本
│       ├── production.sh
│       └── staging.sh
├── logs/                       # 日志文件
│   ├── access.log
│   └── error.log
├── public/                     # 静态文件
│   ├── uploads/               # 上传文件
│   ├── images/
│   └── docs/
├── .env                       # 环境变量
├── .env.example              # 环境变量示例
├── .gitignore               # Git忽略文件
├── .eslintrc.js            # ESLint配置
├── .prettierrc             # Prettier配置
├── .dockerignore          # Docker忽略文件
├── Dockerfile             # Docker构建文件
├── docker-compose.yml    # Docker编排文件
├── jest.config.js        # Jest测试配置
├── tsconfig.json         # TypeScript配置
├── nodemon.json         # Nodemon配置
├── README.md            # 项目说明文档
├── CHANGELOG.md         # 变更日志
├── LICENSE             # 开源协议
└── package.json        # 项目配置文件
```

优势：
- 高内聚低耦合：功能模块独立，便于团队协作 
- 可复用性：模块可通过依赖注入（DI）灵活调用 

## 领域驱动设计（DDD）结构

```bash
project-root/
├── src/
│   ├── domain/           # 领域层
│   │   ├── entities/    # 实体
│   │   ├── value-objects/
│   │   └── services/    # 领域服务
│   ├── application/     # 应用层
│   │   └── services/   
│   ├── infrastructure/  # 基础设施层
│   │   ├── database/
│   │   └── external-services/
│   └── interfaces/      # 接口层
│       ├── http/
│       └── controllers/
```

## 静态文件与模板配置

静态资源：通过 express.static 指定目录，支持多目录和虚拟路径：  

```JavaScript
app.use('/static', express.static(path.join(__dirname, 'public'))); // 虚拟路径
```
模板引擎：如 EJS/Pug，通常放在 views/ 目录，通过 app.set('view engine', 'ejs') 配置  

## 测试与文档

测试目录：建议按类型划分（单元测试、集成测试）：
```bash
 tests/
├── controllers/
├── models/
└── integration/
```

文档：README.md 说明项目结构，swagger 生成 API 文档 

## 环境与工具配置

- 环境变量：使用 .env 文件管理敏感信息（如数据库连接字符串） 
- 脚本工具：package.json 定义启动命令（如 npm start 或 supervisor 热更新） 

# 选择建议

- 小型项目：基础分层结构足够 
- 中大型项目：采用服务层或模块化结构，结合事件驱动 
- 团队协作：严格遵循命名规范（如驼峰命名法）和目录注释 
