---
title: 账户模块
categories:
  - null
description: 账户模块
date: 2025-07-08 12:54:12
tags:
---

把原本的账户模块拆分：  

### 一、账号模块（Account Context）
**核心职责**：  
- 管理用户的核心身份信息（如用户名、邮箱、手机号）  
- 维护账号生命周期（注册、注销、资料修改）  
- 处理账号状态（启用/禁用）和多设备绑定  

**领域对象设计**：  
1. **聚合根**：  
   - `Account`  
     - **唯一标识**：`AccountId`（UUID或自增ID）  
     - **关键属性**：  
       - `Email`（值对象）  
       - `PhoneNumber`（值对象）  
       - `AccountStatus`（枚举：ACTIVE/DISABLED）  
     - **行为**：  
       - 修改邮箱/手机号（需验证）  
       - 禁用账号（触发领域事件`AccountDisabledEvent`）  

2. **值对象**：  
   - `Email`：包含地址和验证状态，不可变  
   - `PhoneNumber`：包含号码和国际区号，不可变  
   - `Address`：用于实名认证，包含省、市、街道，整体替换  

3. **实体**：  
   - `UserProfile`（用户资料实体）  
     - **标识**：`ProfileId`  
     - **属性**：头像、昵称、性别等  
     - **行为**：更新个人资料  

**设计要点**：  
- 账号模块**不包含密码和认证凭证**，仅管理身份信息  
- 通过`AccountDisabledEvent`通知其他模块（如认证模块终止会话）  



---

### 二、认证模块（Authentication Context）
**核心职责**：  
- 验证用户身份（登录、OAuth、MFA）  
- 管理会话（Session）和凭证（Token、密码）  
- 实现“记住我”等快速登录功能  

**领域对象设计**：  
1. **聚合根**：  
   - `AuthCredential`  
     - **唯一标识**：`CredentialId`  
     - **关键属性**：  
       - `Password`（值对象，含加密哈希和盐值）  
       - `MFADevice`（实体列表）  
       - `Session`（实体列表）  
     - **行为**：  
       - 验证密码  
       - 绑定/解绑MFA设备  

2. **值对象**：  
   - `Password`：封装加密逻辑和强度校验  
   - `Token`：JWT或OAuth Token，含过期时间和签发者  

3. **实体**：  
   - `Session`  
     - **标识**：`SessionId`  
     - **属性**：创建时间、设备信息、IP地址  
     - **行为**：刷新过期时间  
   - `MFADevice`  
     - **标识**：`DeviceId`  
     - **属性**：设备类型（TOTP/短信）、绑定时间  

**设计要点**：  
- 认证模块通过`AccountId`关联账号模块，**不直接引用`Account`对象**  
- “记住我”功能通过长期有效的`Token`实现，存储于安全介质（如Keychain）  

---

### 三、会话记录模块（Session Logging Context）
**核心职责**：  
- 记录用户登录/登出行为  
- 审计异常登录（如异地登录）  
- 提供会话历史查询  

**领域对象设计**：  
1. **聚合根**：  
   - `SessionLog`  
     - **唯一标识**：`LogId`  
     - **关键属性**：  
       - `AccountId`（关联账号）  
       - `LoginTime`、`LogoutTime`  
       - `OperationType`（枚举：LOGIN/LOGOUT/EXPIRED）  
     - **行为**：  
       - 记录会话事件  
       - 标记异常会话  

2. **值对象**：  
   - `IPLocation`：解析IP所属地理信息，不可变  

3. **实体**：  
   - `AuditTrail`（审计轨迹实体）  
     - **标识**：`AuditId`  
     - **属性**：操作类型、时间戳、风险等级  
     - **行为**：触发风险告警  

**设计要点**：  
- 通过`AccountId`弱关联账号模块，避免直接依赖  
- 审计逻辑封装在聚合根内，如检测到同一账号短时间多设备登录  

---

### 四、模块间协作与边界
1. **账号与认证模块**：  
   - 账号模块发布`AccountDisabledEvent` → 认证模块终止所有活跃会话  
   - 认证模块通过`AccountId`查询账号状态，但**不修改账号信息**  

2. **认证与会话记录模块**：  
   - 认证模块的`Session`实体生成登录事件 → 会话记录模块创建`SessionLog`  

3. **设计原则**：  
   - **聚合根之间通过ID引用**，避免对象直接嵌套  
   - **值对象不可变**，确保线程安全和业务一致性  

---

### 总结对比表
| 模块           | 聚合根          | 核心实体                | 关键值对象          | 职责边界                  |
|----------------|-----------------|-------------------------|---------------------|---------------------------|
| **账号模块**   | `Account`       | `UserProfile`           | `Email`、`PhoneNumber` | 身份信息管理              |
| **认证模块**   | `AuthCredential`| `Session`、`MFADevice`  | `Password`、`Token`   | 身份验证与凭证管理        |
| **会话记录模块**| `SessionLog`    | `AuditTrail`            | `IPLocation`         | 会话行为审计与记录        |

## 业务实现

### 1. 注册账号

**废弃**
```

1. 在（渲染进程的 Account 模块中的）注册表单中填写信息（username等，不包括密码），调用主进程 Account 模块注册服务并验证。
2. Account 发送「为账号添加登录凭证的事件」事件，Authentication 模块监听事件。
3. Authentication 模块向渲染进程请求认证信息。
4. 渲染进程弹出表单让用户填写并返回给认证模块。
5. Authentication 模块验证后保存并发消息通知
```
1. 直接表单中填写信息（username、password），调用主进程 Authentication 模块认证服务。

#### 细节

注册表单应该先包含 Account 的信息（username、User相关）

### 2. 登录账号

1. 在（渲染进程的 Authentication 模块中的）登录表单组件 中填写信息（username、password），调用主进程 Authentication 模块认证服务。
2. 认证服务发送「验证账号状态事件」事件，Account 模块返回账号信息。
3. 认证服务收到信息后，验证账号状态，并根据返回的 account_uuid，找到存储的验证信息并验证。

#### account 数据库 findByUsername

1. `SELECT * FROM accounts WHERE username = ?`，然后将数据转换成账号对象。
2. 转换成对象函数中，通过 account_uuid 从 user_profiles 数据表中获取用户信息。

**废弃**
```
1. 在（渲染进程的 Authentication 模块中的）登录表单组件 中填写信息（username、password），调用主进程 Authentication 模块认证服务。
2. 认证服务发送「验证账号状态事件」事件，Account 模块返回状态。
3. Authentication 模块验证登录凭证（生成Token），发出消息通知。
4. SessionLogging 模块监听到后记录登录信息。  
```



### 3. 注销账号

1. 在（渲染进程的 Account 模块中的）账号功能组件 点击注销按钮，确认后调用主进程 Account 模块的账号注销服务。
2. 账号注销服务发送「验证账号注销事件」事件。
3. Authentication 模块监听后向渲染进程请求确认（再次输入认证信息，如密码）并返回。
4. Authentication 模块验证认证信息成功后，发送「确认注销账号事件」事件。并将账号认证信息删除。
5. Account 模块收到「确认注销账号事件」事件后，也将账号信息删除

### 4. 导出用户信息