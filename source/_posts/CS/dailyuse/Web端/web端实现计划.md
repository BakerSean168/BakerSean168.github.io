---
title: web端实现计划
categories:
  - null
description: web端实现计划
date: 2025-08-09 09:28:30
tags:
---

下面是一份面向“从 Electron 离线桌面应用演进到 Web（浏览器端 SPA + Express 后端）”的可执行规划文档。目标是最大化复用你现有 Vue3/TypeScript/DDD 代码，构建清晰分层与可演进的架构，并支持在线服务与离线能力的融合。

一、目标与范围
- 目标
  - 基于现有离线 Electron 应用，构建 Web 版（Vue3 前端 + Express 后端）。
  - 复用既有领域模型、业务规则、DTO/校验、UI 组件与工具方法。
  - 强化架构分层与模块边界，支持长期演进、可测试、可观测、可扩展。
  - 打通在线账号与数据同步（后端为权威数据源，客户端支持离线读写与后台同步）。
- 范围
  - 新增 apps/web（浏览器端）与 apps/api（Express）。
  - 抽取 shared domain/utils/contracts/ui 等到独立可复用包。
  - 对既有 Electron（apps/desktop）仅做适配层改造，避免大改 UI/交互。

二、总体架构（Monorepo + DDD + Ports/Adapters）
- Monorepo：使用 pnpm workspaces 管理多项目与共享包。
- 分层
  - Domain（领域）：实体、值对象、领域服务、领域事件、用例（application service）。不依赖框架。
  - Ports（端口）：仓储接口、外部服务接口、事件总线接口（纯 TypeScript interface）。
  - Adapters（适配器）：
    - Web 前端：REST/WebSocket 客户端适配器、IndexedDB 本地存储适配器。
    - Electron：现有 SQLite/文件存储适配器、IPC 适配器。
    - API：数据库仓储适配器（Prisma/TypeORM）、消息/通知适配器。
  - Presentation（表现层）：
    - Web：Vue3 + Pinia + Vue Router + Vue Query + Vuetify。
    - Electron：保留现有 renderer，用共享 UI 包与领域用例。
  - Contracts（契约）：OpenAPI + zod schema/TypeScript DTO，前后端共享。
- 通信
  - REST（CRUD + 分页 + 增量同步）、WebSocket（实时变更通知/提醒/消息）。
- 运行时数据
  - 服务端数据库（权威）：PostgreSQL/MySQL 任一，建议 PostgreSQL。
  - 客户端缓存：Web 用 IndexedDB（Dexie/TanStack Query 缓存持久化），Electron 保持 SQLite/文件。
- 鉴权
  - 前端 SPA：短期内采用刷新令牌 + httpOnly 刷新 Cookie + 内存访问令牌；CSRF 防护。
  - Electron：本地安全存储（如 keytar），与服务端同一账号系统。

三、代码复用策略（从当前仓库出发）
- 强复用
  - common/modules 与 src/shared 下的 domain、types、utils、i18n、事件模型与业务规则。
  - src/modules/* 中的纯前端业务逻辑（和 UI 解耦后）可迁移到应用服务层或 hooks。
- 适配重写
  - 访问存储/IPC/系统 API 的代码：抽象成 Ports，分别实现 web/electron 适配器。
  - 网络层：统一封装 axios 实例、拦截器、错误映射、重试和幂等。
- UI
  - 抽取可复用基础组件与样式到 @dailyuse/ui（Vuetify 基础上封装）。
- 契约
  - 使用 zod 定义 DTO 与校验，zod-to-openapi 生成 OpenAPI；前端通过 openapi-typescript 或 orval 生成客户端类型与调用器。

四、目标目录结构（示例）
- apps/
  - desktop/（现有 Electron 应用：electron/、src/ 等迁入）
  - web/（新建 Vue3 SPA）
  - api/（Express 服务）
- packages/
  - domain/（领域模型、用例、领域事件；无框架依赖）
  - contracts/（zod schema、OpenAPI、生成的 TS 类型）
  - utils/（通用工具：日志、时间、ID、错误类型、重试）
  - ui/（可复用 Vue 组件、主题、图标、布局）
  - adapters/
    - repo-indexeddb/（Web 客户端仓储实现）
    - repo-sqlite/（Electron 仓储实现，若已有可迁移）
    - repo-prisma/（API 仓储实现）
    - messaging/（WebSocket 客户端/服务端）
- tooling/
  - eslint、prettier、tsconfig 基线
  - scripts：生成契约、代码检查、构建、发布

五、后端（Express）设计
- 技术
  - Express + zod（请求/响应校验）+ Prisma（PostgreSQL）+ pino（日志）+ helmet/cors/rate-limit + bullmq（可选异步任务）+ socket.io/ws（实时）。
- 模块化（与当前领域一致）
  - Authentication、Account、Task、Goal、Reminder、Repository、SessionLogging、Notification...
- API 设计
  - REST 风格、/v1 前缀；资源化 + 分页 + 排序 + 过滤；一致错误结构（code、message、details、traceId）。
  - 增量同步端点：/sync/pull、/sync/push（按模块细分亦可）。
- 鉴权
  - 登录颁发：短期 token（内存）+ 刷新 token（httpOnly、SameSite=strict Cookie），刷新端点滚动刷新；附加 CSRF token。
- 合规与安全
  - 输入验证、输出过滤、速率限制、CORS 白名单、TLS、审计日志。
- 契约与文档
  - zod-to-openapi 生成 openapi.json，Swagger UI 本地查看；作为 @dailyuse/contracts 来源。
- 可观测性
  - 请求日志、领域事件日志、Prometheus 指标（可选）与健康检查。

六、前端（Web）设计
- 技术
  - Vue3 + Vite + TypeScript + Pinia + Vue Router + Vue Query（server state）+ Axios + Vuetify + Vue I18n + Vee-Validate + zod。
  - PWA：workbox，预缓存 + 运行时缓存；后台同步（Background Sync）用于重试推送。
- 状态管理
  - UI/视图状态：Pinia。
  - 远程数据：Vue Query（缓存、请求去重、重试、失效策略、持久化到 IndexedDB）。
- 存储
  - IndexedDB（Dexie）存离线草稿与同步队列；LocalStorage 仅存轻量非敏感偏好。
- 路由与页面
  - Auth、Dashboard、Tasks、Goals、Reminder、Repository、Settings、Profile、Audit/Logs。
- 网络层
  - 统一 axios 客户端，处理 401/403、重试、错误提示、traceId 透传。
- 离线/同步
  - 写入：落本地队列（操作日志），后台推送到 /sync/push。
  - 拉取：按 updatedAt/version 增量 /sync/pull，合并至 IndexedDB 与内存缓存。
  - 冲突：按资源定义策略（建议：带 version 的乐观并发控制 + 人工介入界面）。
- 实时
  - 登录后建立 WebSocket；接收变更事件以失效/更新对应 Query，驱动实时 UI。
- 体验与可访问性
  - 骨架屏/占位符、错误边界、A11y 对比度与键盘导航、主题与 i18n 复用。

七、同步与一致性策略（关键）
- 服务端为权威，客户端可离线操作，最终一致。
- 模型字段建议
  - id、updatedAt、version（乐观锁）、deletedAt（软删）、clientChangeId（幂等）。
- Push：客户端发送操作日志（含 clientChangeId），服务端按版本校验并持久化，返回新的版本/时间戳。
- Pull：按 updatedAt > lastPulledAt 增量拉取，含软删记录。
- 冲突解决
  - 默认：并发冲突时 409，返回服务器版本与差异；客户端提示合并；简单场景可 LWW（最后写入优先）或字段级合并策略。
- 幂等与重放
  - 服务端记录 clientChangeId 以避免重复提交。
- 审计
  - SessionLogging 作为变更流水，支撑问题定位与回溯。

八、迭代计划与交付物
- 里程碑 1：Monorepo 化与包抽取（1-2 周）
  - 建立 pnpm workspaces。
  - 抽取 packages/domain、packages/utils、packages/ui、packages/contracts 雏形。
  - Electron 适配 imports，确保桌面版不受影响。
- 里程碑 2：API 骨架与鉴权（1-2 周）
  - apps/api：基础中间件、账户/鉴权模块、OpenAPI 文档。
  - DB 迁移与本地环境，接入日志与健康检查。
- 里程碑 3：Web 应用骨架（1-2 周）
  - apps/web：路由、布局、主题、i18n、鉴权流程打通（登录/刷新/登出）。
  - Axios/Vue Query 基础与错误处理。
- 里程碑 4：核心业务模块迁移（2-4 周）
  - Task、Goal、Reminder、Repository 等页面与用例。
  - 将领域用例从 Electron 复用到 web（通过 adapters）。
- 里程碑 5：离线与同步（2-3 周）
  - IndexedDB、本地操作队列、/sync/push & /sync/pull。
  - 冲突处理 UI 与策略落地；WebSocket 实时更新。
- 里程碑 6：打磨与稳定（1-2 周）
  - 单元/集成/E2E 测试、性能优化、监控告警、错误收集（Sentry 可选）。
- 里程碑 7：部署与发布（1 周）
  - API 上线（容器化 + 数据库），Web 前端静态托管，域名与 TLS，CI/CD 全流程。

九、测试与质量
- 单元测试：domain（纯函数/用例）、adapters（使用内存/仿真）。
- API 测试：Supertest + 测试数据库（迁移 + 隔离）。
- 前端组件与页面：Vitest + Vue Testing Library。
- E2E：Playwright（登录、增删改查、离线/在线切换、冲突场景）。
- 静态检查：ESLint、TypeScript 严格模式、prettier、一致 commit 规范。
- 合约测试：OpenAPI 校验、前后端类型生成一致性校验。

十、CI/CD 与运维
- CI
  - 安装依赖、类型检查、lint、测试、构建、OpenAPI 生成与校验。
- CD
  - API：容器镜像（多环境），数据库迁移，蓝绿/滚动发布。
  - Web：静态构建制品上传 + 缓存策略；Sourcemap 上报（Sentry）。
- 配置与密钥
  - 多环境变量管理（.env.* + 密钥管理服务），明确配置矩阵（dev/staging/prod）。
- 监控
  - API：日志聚合、请求指标、错误率、可用性探针。
  - 前端：前端错误收集、性能指标（FCP、LCP、CLS）。

十一、模块映射与落地建议
- 可直接迁移/复用
  - common/modules/**/types、领域规则与校验、src/shared/utils、i18n、axios 封装（调整为跨端）。
- 需抽象的适配层
  - repository、notification、schedule、sessionLog：定义 ports，提供 web/electron/api 三端适配器。
- Electron 专属能力
  - 文件系统、系统通知、窗口管理等保留在 desktop 适配器，不进入 web。

十二、关键技术选型清单
- 前端：Vue3、Pinia、Vue Router、Vuetify、Vue Query、Axios、Dexie、Vee-Validate+zod、Workbox、mitt（事件总线）。
- 后端：Express、Prisma+PostgreSQL、zod、pino、helmet、cors、rate-limiter-flexible、socket.io/ws、bullmq（可选）。
- 合约：zod-to-openapi、openapi-typescript/orval。
- 工具：pnpm workspaces、ESLint、Prettier、Husky（pre-push hooks 可选）、Vitest、Playwright、Sentry（可选）。

十三、风险与对策
- 同步与冲突复杂度高：先实现单向在线模式（仅在线 CRUD），再引入离线与增量同步；从一个模块试点（如 Task）。
- 复用代码耦合 UI/存储：优先解耦为用例 + 端口 + 适配器，逐步替换引用。
- 鉴权跨端差异：Web 使用 Cookie 刷新 + 内存访问令牌；Electron 使用本地安全存储；统一后端令牌模型与过期策略。
- 渐进式迁移：保证桌面应用可持续运行，避免大爆炸式重构。

十四、近期行动清单（本周可做）
- 建 Monorepo 与 packages/domain、packages/utils、packages/contracts 雏形。
- 在 domain 中定义 Repository 等 ports 接口，梳理 Task/Goal 的核心实体与用例。
- apps/api 建骨架，接入鉴权与 OpenAPI。
- apps/web 建骨架，完成登录与基础路由，打通鉴权与 OpenAPI 生成的客户端类型。
- 选择一个模块（Task）打通“后端 REST + 前端页面 + 增量拉取”的最小闭环。

如需，我可以基于你当前仓库直接初始化 monorepo 结构、抽取共享包，并搭建 apps/web 与 apps/api 的最小可运行骨架。


## 第一步

请你参考 domain-server、domain-core 的 account 模块，来帮我实现 domain-server、domain-core 的 authentication模块。
重点：
1. 在 domain-core 的 authentication 模块的 types 文件夹 下创建每个domain对象的接口定义，并在每个domain对象中 implement。
2. 在 domain-server 中的 types 文件夹下 继承 接口，可以先用 isClient isServer 占位，或者你直接给出需要的接口，并在 domain 对象中 implement