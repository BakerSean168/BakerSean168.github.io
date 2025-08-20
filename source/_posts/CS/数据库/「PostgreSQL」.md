---
title: 「PostgreSQL」
categories:
  - null
description: 「PostgreSQL」
date: 2025-08-09 19:53:48
tags:
---

[PostgreSQL中文文档](https://postgresql.uihtm.com/)

Stack Builder 可以用来为 PostgreSQL 安装模块  

windows 安装默认包含 pgAdmin 工具

# pgAdmin 工具

## 查看表信息

1. 连接到数据库
   - 启动pgAdmin 4
   - 在登录界面输入您的凭据（如果设置了密码）
   - 在左侧导航树中，双击您的服务器名称或点击旁边的展开箭头

2. 浏览表结构
   - 展开数据库 → 展开"Schemas" → 展开"public"（或其他模式）
   - 点击"Tables"查看所有表列表
   - 您可以右键点击表名选择"Properties"查看表结构

3. 查看表数据
   - 找到您想查看的表，右键点击
   - 选择"View/Edit Data" → 然后选择：
   - "All Rows"：查看所有数据
   - "First 100 Rows"：查看前100行
   - "Last 100 Rows"：查看最后100行
   - "Filtered Rows"：按条件筛选数据

4. 也可以使用SQL查询来查看表内容：
   1. 右键点击数据库名称
   2. 选择"Query Tool"
   3. 输入SQL语句，如：`SELECT * FROM 表名;`
   4. 点击执行按钮（或按F5）