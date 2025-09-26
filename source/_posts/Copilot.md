---
title: GitHub Copilot 使用技巧与最佳实践
categories:
description: 全面解析 GitHub Copilot 的使用技巧、最佳实践和高效工作流程
date: 2025-09-01 13:30:00
tags:
  - copilot
  - ai
  - 编程助手
  - vscode
  - 开发效率
---

## 基础概念

GitHub Copilot 是由 GitHub 和 OpenAI 合作开发的 AI 代码助手，基于大型语言模型训练，能够根据上下文自动生成代码建议、完成函数实现、提供代码解释等功能。

特殊文件（通过聊天窗口右上角设置添加，或者手动添加）
**prompt** 提示词文件
instruction 说明文件

## 工作流

以下面的需求为例：  

> SDN 网络软件开发  
要求基于软件定义网络技术，包括编程与测试平台（Nexus 9000v）、开发接口
NX-SDK（NXAPI），利用 Python 等通用编程语言开发网络运维或应用软件，
同时可集成 Chef/Puppet/RESTCONF/NETCONF 等，解决网络部署、运维、管理、
排错等任意应用场景下的自动化或智能化需求,例如：
（1）实现一个网络软件，完成接收事件通知、故障告警，及时发现和快速定位
有线或者无线网络的问题等功能，让网管人员可以随时随地掌握校园网的健康状
况，预先发现网络故障隐患，及时排除网络故障；
（2）实现一个网络软件，可灵活、方便地解决校园网中关键应用（如选课系统）
与非关键应用竞争网络资源的问题，通过创建和修改网络策略，下发策略，网络
设备根据推送策略修改 QoS 配置，从而确保关键应用在校园网具有高优先级。

### 生成需求文档

```md

## 需求

**效果**:  
实现一个网络软件，完成接收事件通知、故障告警，及时发现和快速定位有线或者无线网络的问题等功能，让网管人员可以随时随地掌握校园网的健康状况，预先发现网络故障隐患，及时排除网络故障；

**技术限定**：  
- 基于软件定义网络技术，包括编程与测试平台（Nexus 9000v）、开发接口NX-SDK（NXAPI）
- 基于 python

## 技术方案

请你帮我生成一份可行的技术方案（直接创建文档），能够在三个星期内实现
```

### 使用 prompt 文件

1. 创建 prompt
2. 把 生成的技术方案 直接复制进去
3. 后续可以添加其他提示词，比如：
  **角色定位**：  
  你是一个网络工程师（偏向软件），帮我实现我的网络软件

后续直接：  
```输入框
/[创建的 prompt 的名字] 用来引用 prompt
请你直接帮我按照这个方案生成项目代码
```
然后慢慢调，慢慢问

可以直接 默认允许那些 命令，就不用手动点了

## 核心使用技巧

### 编写高质量注释
Copilot 依赖上下文理解，详细的注释能显著提升代码建议质量。

**好的注释示例：**
```typescript
/**
 * 计算两个日期之间的工作日天数（不包含周末和节假日）
 * @param startDate 开始日期
 * @param endDate 结束日期
 * @param holidays 节假日列表，格式：['2023-01-01', '2023-10-01']
 * @returns 工作日天数
 */
function calculateWorkdays(startDate: Date, endDate: Date, holidays: string[]): number {
  // Copilot 会基于这个详细注释生成准确的实现
}
```

### 使用描述性函数名
```typescript
// 好的函数名，Copilot 能准确推断实现
function validateEmailAndSendWelcome(email: string) {
  // Copilot 会生成邮箱验证和发送欢迎邮件的代码
}

// 模糊的函数名
function process(data: any) {
  // Copilot 难以推断具体需要做什么
}
```

### 提供类型信息（TypeScript）
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

// 有了类型信息，Copilot 能生成更准确的代码
function createUserProfile(user: User) {
  // Copilot 知道 user 的结构，生成的代码会更准确
}
```

### 使用示例驱动开发
```typescript
// 先写测试用例，让 Copilot 理解预期行为
describe('formatCurrency', () => {
  it('should format number to currency string', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
    expect(formatCurrency(0)).toBe('$0.00');
  });
});

// 然后实现函数，Copilot 会根据测试用例生成实现
function formatCurrency(amount: number): string {
  // Copilot 会生成符合测试预期的代码
}
```

---

## 高级使用技巧

### 1. 利用 Copilot Chat 进行代码审查
```
/explain 这段代码的时间复杂度是什么？
/fix 这段代码有什么潜在的性能问题？
/optimize 如何优化这个算法？
/generate 为这个函数生成单元测试
```

### 2. 上下文工程（Context Engineering）
```typescript
// 在文件顶部提供上下文信息
/**
 * 用户管理模块
 * - 使用 JWT 进行身份验证
 * - 密码使用 bcrypt 加密
 * - 支持邮箱验证
 * - 集成 Redis 缓存
 */
```

### prompt 文件

[官网的 prompt 文件配置页](https://code.visualstudio.com/docs/copilot/customization/prompt-files?originUrl=%2Fdocs%2Fcopilot%2Fchat%2Fprompt-crafting)

把之前放在 prompt.md 文件中的内容放到了改配置文件中，确实好用。  
把项目介绍、模块、指令都放进去  

## 常见场景与技巧

### 1. API 开发
```typescript
/**
 * RESTful API for user management
 * - GET /users - list users with pagination
 * - POST /users - create new user
 * - PUT /users/:id - update user
 * - DELETE /users/:id - delete user
 */
app.get('/users', async (req: Request, res: Response) => {
  // Copilot 会生成包含分页、错误处理的代码
});
```

### 2. 数据处理
```typescript
// 描述数据转换逻辑
const rawData = [
  { name: 'John', age: 25, department: 'Engineering' },
  { name: 'Jane', age: 30, department: 'Marketing' }
];

// Transform raw data to chart-friendly format with age groups
const chartData = rawData
  // Copilot 会生成数据分组和转换逻辑
```

### 3. 测试代码生成
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Given: valid user data
      // When: creating user
      // Then: user should be created successfully
      // Copilot 会生成完整的测试代码
    });
  });
});
```

---

## 性能优化技巧

### 1. 减少不必要的建议
```json
{
  "github.copilot.enable": {
    "markdown": false,    // 在 markdown 中禁用
    "txt": false,         // 在纯文本中禁用
    "json": false         // 在配置文件中禁用
  }
}
```

### 2. 快捷键设置
```json
{
  "key": "ctrl+enter",
  "command": "github.copilot.generate"
},
{
  "key": "alt+]",
  "command": "editor.action.inlineSuggest.showNext"
}
```

---


**参考资源**
- [GitHub Copilot 官方文档](https://docs.github.com/en/copilot)
- [VS Code Copilot 扩展](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
-

