---
title: Account 模块
categories:
  - null
description: electron+vue3 项目怎么实现一个支持离线、在线的账号模块
date: 2025-05-24 10:42:00
tags:
---

# 准备，从功能来思考

1. 先想好功能
2. 按照功能来模块化管理分割代码  
3. 按照预定的功能主次来依次实现

账号模块应该有：  
- 用于认证 auth 相关，处理身份验证和授权相关逻辑
  登录、快速登录、自动登录、Token管理、权限验证
- 用于用户 user 相关，管理用户实体数据
  注册、获取当前用户信息、注销、个人信息管理、账户状态管理

快速登录应该是提供可快速登录的账户，点击后直接登录进入，不需要输入用户密码。  
要实现快速登录、自动登录的话，应该是要保存一些登录信息的，比如登录凭证、账号信息等。  

所以再引入 loginSession 相关功能：  
- 用于登录会话 loginsession 相关，在本地保存用户的登录信息，如账号、token、remember等信息
  查询记录获取需要快速登录的账号及凭证，为快速登录提供服务  

# 离线

离线就是本地账号服务，使用 Electron 的主进程来进行数据存储和相应的逻辑，使用 ipc 通信来暴露接口（类似后端），而 Vue3 则作为渲染进程（类似前端）。

整体结构：

```bash
electron/
  config/
    database.ts // 数据库配置
  modules/
    Account/
      ipcs/ // ipc 层， 用于主进程与渲染层通信
      models/ // 模型层，操控数据库
      services/ // 封装模型层，提供相关服务
src/
  modules/
    Account/
      components/
      composables/
      services/ // 渲染层相关服务，调用 ipc 暴露的接口
      stores/
      types/
      views/
```

## 主进程

### 数据库

选择使用 sqlite，使用的人数多，社区资源较丰富，使用难度较低

#### 用户表结构

先默认创建本地账户，只需要 username、password，将 username 作为键值。  
当用户需使用联网服务（同步功能），再升级账号，完善用户信息，服务端生成 onlineId 用于标识

```ts
db.exec(`
      CREATE TABLE IF NOT EXISTS users (
        username TEXT PRIMARY KEY,
        password TEXT NOT NULL,
        avatar TEXT,
        email TEXT,
        phone TEXT,
        accountType TEXT DEFAULT 'local',
        onlineId TEXT,
        createdAt INTEGER NOT NULL
      )
    `);
```

### 模型层

建立一个模型类,封装对配置的数据库进行的操作

```ts
/**
 * 用户数据模型类
 * 负责用户数据的持久化操作，包括增删改查
 */
export class UserModel {
  private db: Database | null = null;

  /**
   * 私有构造函数，防止直接实例化
   */
  private constructor() {}

  /**
   * 静态方法创建实例
   * @returns UserModel 实例
   */
  public static async create(): Promise<UserModel> {
    const instance = new UserModel();
    instance.db = await initializeDatabase();
    return instance;
  }

  /**
   * 确保数据库连接存在
   */
  private async ensureDatabase(): Promise<Database> {
    if (!this.db) {
      this.db = await initializeDatabase();
    }
    return this.db;
  }

  /**
   * 添加新用户
   * @param userData 用户数据对象
   * @returns 添加是否成功
   */
  async addUser(userData: TUser): Promise<boolean> {
    try {
      const db = await this.ensureDatabase();
      const stmt = db.prepare(`
        INSERT INTO users (
          username,
          password,
          avatar,
          email,
          phone,
          accountType,
          onlineId,
          createdAt
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?)`);

      const result = stmt.run(
        userData.username,
        userData.password,
        userData.avatar || null,
        userData.email || null,
        userData.phone || null,
        userData.accountType || "local",
        userData.onlineId || null,
        userData.createdAt
      );
      return result.changes > 0;
    } catch (error) {
      console.error("添加用户失败:", error);
      return false;
    }
  }
}
```

### 服务层

利用 model 层方法来提供响应的服务

```ts
/**
 * 用户服务类
 * 处理与用户账户相关的业务逻辑
 */
export class UserService {
  private static instance: UserService;
  private userModel: UserModel;

  /**
   * 私有构造函数，初始化用户数据模型
   */
  private constructor(userModel: UserModel) {
    this.userModel = userModel;
  }

  /**
   * 获取UserService单例
   * @returns UserService实例
   */
  public static async getInstance(): Promise<UserService> {
    if (!UserService.instance) {
      const userModel = await UserModel.create();
      UserService.instance = new UserService(userModel);
    }
    return UserService.instance;
  }

  /**
   * 用户登录
   * @param data 登录数据
   * @returns 响应结果
   */
  public async login(data: TLoginData): Promise<TResponse> {
    try {
      // 验证输入数据
      if (!data.username || !data.password) {
        return {
          success: false,
          message: "用户名和密码不能为空",
        };
      }

      // 查找用户
      const user = await this.userModel.findUserByUsername(data.username);
      if (!user) {
        return {
          success: false,
          message: "用户不存在",
        };
      }

      // 验证密码
      const isPasswordValid = await this.verifyPassword(
        data.password,
        user.password
      );
      if (!isPasswordValid) {
        return {
          success: false,
          message: "密码错误",
        };
      }

      // 登录成功，返回用户信息（不包含密码）
      return {
        success: true,
        message: "登录成功",
        data: {
          username: user.username,
          avatar: user.avatar,
          email: user.email,
          phone: user.phone,
          accountType: user.accountType,
          onlineId: user.onlineId,
          createdAt: user.createdAt,
        },
      };
    } catch (error) {
      console.error("登录过程中发生错误:", error);
      return {
        success: false,
        message: "登录失败，服务器错误",
      };
    }
  }
}
```

### ipc

```ts
/**
 * 用户相关 IPC 处理器
 * 处理渲染进程与主进程之间的用户账户相关通信
 * 提供注册、登录、用户信息管理等功能的 IPC 接口
 */

/**
 * 设置用户相关的 IPC 处理器
 * 在应用启动时调用此函数注册所有用户相关的 IPC 处理器
 */
export async function setupUserHandlers(): Promise<void> {
  try {
    // 确保服务实例已初始化
    const service = await userService;

    /**
     * 用户注册
     * 处理新用户的注册请求
     *
     * 调用方式：
     * ipcRenderer.invoke('user:register', {
     *   username: 'testuser',
     *   email: 'test@example.com',
     *   password: 'password123',
     *   confirmPassword: 'password123'
     * })
     */
    ipcMain.handle("user:register", async (_event, form: TRegisterData): Promise<TResponse> => {
      console.log('IPC: 用户注册请求', { username: form.username, email: form.email });
      try {
        const response = await service.register(form);
        console.log('IPC: 注册结果', { success: response.success, username: form.username });
        return response;
      } catch (error) {
        console.error('IPC: 注册异常', error);
        return {
          success: false,
          message: error instanceof Error ? error.message : "注册失败，未知错误",
        };
      }
    });

    /**
     * 用户登录
     * 验证用户凭证并返回用户信息
     *
     * 调用方式：
     * ipcRenderer.invoke('user:login', {
     *   username: 'testuser',
     *   password: 'password123',
     *   remember: true
     * })
     */
    ipcMain.handle("user:login", async (_event, credentials: TLoginData): Promise<TResponse> => {
      console.log('IPC: 用户登录请求', { username: credentials.username });
      try {
        const response = await service.login(credentials);
        console.log('IPC: 登录结果', { success: response.success, username: credentials.username });
        return response;
      } catch (error) {
        console.error('IPC: 登录异常', error);
        return {
          success: false,
          message: error instanceof Error ? error.message : "登录失败，未知错误",
        };
      }
    });
  }
  }
```

## 渲染层

### 服务

```ts
/**
   * 用户注册
   * @param form - 注册表单数据
   * @returns 注册成功的用户信息
   * @throws 注册失败时抛出错误
   */
  async register(form: TRegisterData): Promise<TResponse> {
    try {
      // 创建一个可序列化的对象
      const registrationData = {
        username: form.username,
        password: form.password,
        confirmPassword: form.confirmPassword,
        email: form.email,
        // 只包含必要的基本类型数据
      };
      // 调用后端 API 进行注册
      const response = await window.shared.ipcRenderer.invoke(
        "user:register",
        registrationData
      );
      return response;
    } catch (error) {
      console.error("Registration error:", error);
      return {
        success: false,
        message: error instanceof Error ? error.message : "注册失败",
        data: null,
      };
    }
  }
```

# 在线

使用 express 作为服务端，渲染层调用的接口从 ipc 变为 express 提供的接口。  

## 注册功能

服务端  
1. 配置好 user 表
2. model 层实现插入、删除等需要操作
3. service 层利用model层方法实现注册服务
  传入 用户名、密码... 检查用户是否存在（用户名是否占用），没问题就插入数据库（注册成功）返回相应信息  
4. control 层，处理HTTP请求和响应
  从请求中获取数据（用户名、密码...），调用 service 层方法
5. route 层，配置 api 路由

客户端  
1. 调用 api 实现注册服务  
2. 生成相应的页面，在按钮中调用注册服务    

# 重构为 DDD

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