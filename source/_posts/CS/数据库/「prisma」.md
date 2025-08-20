---
title: 「prisma」
categories:
  - null
description: 「prisma」
date: 2025-08-09 18:25:33
tags:
---

[prisma官网](https://prisma.org.cn/docs)

# 跟着 demo

## 创建 prisma 模型

简历一个 prisma 文件夹，里面创建一个 schema.prisma 文件。  

这个文件指定了：  
- 一个数据源（PostgreSQL 或 MongoDB）
- 一个生成器（Prisma Client）
- 一个包含两个模型（带有一个关系）和一个 enum 的数据模型定义
- 几个原生数据类型属性（@db.VarChar(255), @db.ObjectId）

```Prisma Schema Language
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  published Boolean  @default(false)
  title     String   @db.VarChar(255)
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  Int?
}

enum Role {
  USER
  ADMIN
}
```

## 将数据模型映射到数据库 schema

`npx prisma migrate dev --name init`

此命令执行了两项操作

- 它为此迁移生成了一个新的 SQL 迁移文件
- 它针对数据库运行了 SQL 迁移文件

新创建的 prisma/migrations 目录中检查生成的 SQL 迁移文件。  

## 安装并生成 Prisma Client

安装 @prisma/client 包:  
`npm install @prisma/client`  

运行 prisma generate，读取 Prisma schema 并生成 Prisma Client:  
`npx prisma generate`

由于 Prisma Postgres 提供了连接池和（可选）带有 Prisma Accelerate 的缓存层，您还需要在项目中安装 Accelerate Client 扩展：  
`npm install @prisma/extension-accelerate`

## 使用 Prisma Client 来进行数据库操作

```ts
// 1
import { PrismaClient } from './generated/prisma'
import { withAccelerate } from '@prisma/extension-accelerate'

// 2
const prisma = new PrismaClient()
  .$extends(withAccelerate())

// 3
async function main() {
  // ... you will write your Prisma Client queries here
  const allUsers = await prisma.user.findMany()
  console.log(allUsers) // npx tsx queries.ts => [] 还没有 User 记录
}

// 4
main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    // 5
    await prisma.$disconnect()
    process.exit(1)
  })
```

# 实际使用

## 更新数据表

```bash
# 开发：生成并应用迁移（会在 prisma/migrations 生成 SQL）
pnpm exec prisma migrate dev --name describe_change
pnpm exec prisma generate

# 快速同步（不生成迁移文件）
pnpm exec prisma db push
pnpm exec prisma generate

# 生产：应用已有迁移（无交互）
pnpm exec prisma migrate deploy
pnpm exec prisma generate

# 如果你没有把 prisma 安装到项目依赖，可用 pnpm dlx（一次性下载并运行）
pnpm dlx prisma migrate dev --name describe_change
pnpm dlx prisma generate
```