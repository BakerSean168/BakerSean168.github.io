---
title: 软件测试基础与实践
date: 2025-07-29
description: 系统讲解软件测试的基本概念、分类、流程与常用方法，适合工程化知识体系梳理。
---

## 测试

**目的**：

- 提高代码质量
- 确保系统稳定性

**分类**：

- 静态测试：代码审查、静态分析
- 动态测试：单元测试、集成测试、端到端测试

## 方案选择与设计

1. 单测（Unit Test）

   - 验证单一功能模块，sayHello
   - 单测用力可以用来主导开发

2. 集成测试（Integration Test）

   - 验证模块交互逻辑
   - 单测之后实施

3. 端到端（e2e/ui test）
   - 验证最终用户体验
   - 最后阶段，优先测试一些**关键路径**
     1. 注册登录流程
     2. 核心业务流程

_测试金字塔模型_

顶部层：端到端测试
中间层：集成测试
底部层：单元测试

## 单元测试

**目的**

- 方法、组件作为测试单元
- 测试逻辑正确性

**工具框架**

- Vitest
- Jest
- Mocha

## 集成测试

**目的**  
- 测试模块之间的交互（登录模块+用返回的凭证请求）
- 涉及到数据库、API 外部依赖

**工具框架**
- Supertest（Nestjs）
- Cypress
- JUnit

## 端到端测试

**目的**  
站在用户角度，测试关键流程。下单流程（购买、选择地址、支付方式、完成支付、查看订单、查看物流）

**工具框架**
- Playwright
- Cypress
- Selenium

端到端测试就类似爬虫，模拟用户操作，之前用 playwright 来尝试实现自动登录网站。  
RPA（robotic process automation）机器人流程自动化
- 抢茅台
- 影刀
- Headless Browser
- 使用一大堆 iPad 和 Phone，来模拟用户行为

## 9. 测试常见工具

- 单元测试：Jest、Mocha、JUnit、pytest
- UI 自动化：Selenium、Cypress、Playwright
- 性能测试：JMeter、Locust、LoadRunner
- 静态分析：SonarQube、ESLint、Pylint

# 问题

## 你有做过测试吗，单侧测、集成测试、端到端测试的方案与实践

## 了解过 Vitest 和 Playwright 吗，详细说说单测、集成测试体系搭建与测试用例设计编写的最佳实践

## 假如你是 Vitest 和 Playwright 作者，详细说明单测、集成测试库的架构设计与实现
