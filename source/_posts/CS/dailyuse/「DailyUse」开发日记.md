---
title: 「DailyUse」开发日记
categories:
  - null
description: 「DailyUse」开发日记
date: 2025-07-19 16:38:19
tags:
---

## Ai 使用语句

帮我调整代码位置和添加详细注释（包含输入输出的解释、返回对象的示例等），让代码更易读，并且在其他地方使用时，瞄上去就可以知道需要哪些参数、返回哪些参数、起什么作用

帮我参考 goal 的聚合根代码，改善 account 的代码，
1. 使用对象参数来构造
2. 修改构造函数的参数，去除掉 createAt 等应该自动生成的参数
3. 添加完善的辅助函数 
- `toDTO()`： 将对象实例转为对象接口数据
- `static fromDTO(dto: T)`： 将对象接口数据转为对象实例
- `clone()`： 创建对象的副本
- `forCreate()`： 创建对象实例，用于创建模式
- `isxxx()`： 判断对象实例是否为xxx对象实例
- `ensurexxx()`： 确保对象实例为xxx对象实例
- `ensurexxxNeverNull()`： 确保对象实例为xxx对象实例，并且不为null

在 fromDTO 函数中，先用 构造函数生成初步恢复对象，再用 set 方法恢复其他（createAt等）属性

# 整个项目的架构

- 采用了 Electron + Vue3 + TypeScript + Pinia + Vite  
- 分为主进程 和 渲染进程。  
- 目前主进程和渲染进程都采用 DDD 架构来管理代码模块。  

项目结构：  
```
DailyUse/
├── common/
│   ├── modules/
│   └── shared/
├── docs/
│   ├── common/
│   ├── electron/
│   └── src/
├── electron/
│   ├── modules/
│   ├── preload/
│   ├── shared/
│   ├── windows/
│   ├── electron-env.d.ts
│   ├── main.ts
│   └── preload.ts
├── src/
│   ├── assets/
│   ├── i18n/
│   ├── modules/
│   ├── plugins/
│   ├── shared/
│   ├── views/
│   ├── App.vue
│   ├── main.ts
│   ├── shims-vue.d.ts
│   ├── style.css
│   └── vite-env.d.ts
├── index.html
├── README.md
└── ...

common 文件夹用来存放一些公共文件，如：  
类型定义文件

docs文件夹用来存放一些文档文件。  

electron文件夹用来存放 主进程的代码。  

src 文件夹用来存放渲染进程的代码。  
```

每个主进程模块的架构大致如下：  
```
ModuleName/                  # 模块根目录
├── application/             # 应用层：编排业务用例，协调领域服务和基础设施，暴露给外部（如 IPC Handler）
│   ├── services/            # 应用服务：处理具体用例的服务（如 MainTaskApplicationService）
│   └── events/              # 应用层事件/事件处理器：如与前端或其他模块的集成事件
├── domain/                  # 领域层：核心业务逻辑和规则
│   ├── aggregates/          # 聚合根：领域对象的聚合（如 TaskTemplate、TaskInstance）
│   ├── events/              # 领域事件：领域内发生的重要业务事件
│   ├── repositories/        # 领域仓储接口：定义持久化接口（不关心实现）
│   ├── services/            # 领域服务：跨聚合/复杂业务逻辑
│   ├── types/               # 领域类型定义：如枚举、接口、类型别名
│   ├── utils/               # 领域工具类/函数：纯业务相关的工具
│   └── valueObjects/        # 值对象：不可变的业务对象（如时间段、优先级等）
├── infrastructure/          # 基础设施层：技术实现细节，外部系统适配
│   ├── di/                  # 依赖注入相关代码（如容器、注册器）
│   ├── ipc/                 # IPC Handler：主进程与渲染进程通信的适配器
│   └── repositories/        # 仓储实现：具体的数据库/存储实现
├── initialization/          # 初始化相关：模块启动、资源注册、生命周期管理等
└── index                    # 统一导出模块内部的类型方法等，方便其他地方引用
```

每个渲染进程模块：  
```
ModuleName/
├── application/
│   ├── services/
│   └── events/
├── domain/
│   ├── aggregates/
│   ├── entities/
│   ├── events/
│   ├── services/
│   ├── types/
│   ├── utils/
│   └── valueObjects/
├── infrastructure/
│   └── ipc/
├── initialization/
├── presentation
│   ├── components/
│   ├── composables/
│   ├── stores/
│   └── views/
└── index.ts

```

渲染进程中一般不需要实现抽象仓储层，直接在 application 层中使用 store 进行数据操作。



### 模块的基本结构

### 20250719：  

1. 修改了渲染进程让主进程持久化存储时的账户验证逻辑

- 现有实现  
在仓库中设置 set username 方法，在账号登录时，初始化仓库，给所有仓库设置一个 username。  

- 新实现
在 ipc 层多传入 token 和 accountId，并进行验证，成功后将 accountId 和 数据传入服务层，调用仓库进行存储。  
仓库中不再使用 set username 方法，以 accountId 

1. 渲染进程的 IPC 层已经通过封装来全部都添加了 auth 参数
2. 修改主进程各个模块的 IPC 层代码，调用新的封装后的函数，实现 token 验证，成功后将 accountId 和 数据传入服务层。
3. 修改对应的应用服务层代码，增加传入的 accountId 参数
4. 还有对应的仓储层代码（accountId 作为用户标识，而非username），去除掉 username 相关的代码，直接使用传入的 accountId 和 数据源进行操作。
所有的修改都直接在原文件中直接修改，不需要新建文件；并且不需要考虑兼容之前的代码，因为所有的代码都是直接修改的。

2. 准备修复双窗口的状态不通用导致的问题

### 20250720

1. 准备把所有的 id 都改成 uuid

直接使用 vscode 的查找替换功能。  
比如：  
(\b\w+)\.id\b $1.uuid  

2. 修改过程中重构一下 Goal 模块的代码

### 20250721

1. 统一了所有的 id 为 uuid
2. 添加了一个 common 文件夹，用于存放一些同时用于渲染进程和主进程的通用代码

### 20250723

1. 让 Reminder 在添加 item 后，能立刻响应式渲染
2. 用 AI 添加了一些注释

帮我调整代码位置和添加详细注释（包含输入输出的解释、返回对象的示例等），让代码更易读，并且在其他地方使用时，瞄上去就可以知道需要哪些参数、返回哪些参数、起什么作用

3. 移动 reminderTemplate 表单中的 template 数据丢失 groupUuid 属性

查看打印信息可以发现是对象丢失了 groupUuid 属性

<detail>
<summary>具体对象信息</summary>

```json
[
    {
        "_uuid": "system-root",
        "_domainEvents": [],
        "_name": "系统分组",
        "_enabled": true,
        "_templates": [
            {
                "_uuid": "dca27478-ed6a-440e-92a5-e6aa27272856",
                "_name": "Test333",
                "_description": "这是测试33",
                "_enabled": 1
            }
        ],
        "_enableMode": "group"
    }
]

```

解决：  
发现是 Group 仓储层的转换函数出了问题，没有正确获取 template 的所有属性  

- 先修改为调用 template 仓库的 getByGroupUuid 方法直接获取 template 实例对象
- 并且给 主进程的 templateGroup 聚合根添加了 fromDTOForRepository 方法，用于转换 其他数据为 DTO 但 templates 为实例的数据为 templateGroup 实例的方法。

</detail>

### 20250724

1. 实现了 提醒模板 和 提醒组 的启用状态切换相关的 主进程和渲染进程的代码
2. 实现了在用户登录时初始化为启用状态的提醒

### 20250725

1. 修复了 repository 模块的简单功能和主进程与渲染进程数据同步
2. 改善了一些表单的样式，让按钮、标题区域不参与滚动，只有输入区域滚动
3. 重构 Goal 模块

使用 Date 以及 date-fns 来修改该函数，不再使用 自定义的 TimeUtils；使用参数对象给出完整代码

### 20250726

1. 重构 goal 模块

修改了主进程仓库层代码（数据表结构、数据库数据与对象数据的双向映射函数）、服务层、IPC层，渲染进程的 IPC 层、服务层、组件（goalDialog、ResultDialog）等  
创建 Goal 业务实现  

### 20250727

1. 继续重构 goal 模块

改善了 keyResult、goalReview 等相关的组件、业务逻辑



### 20250728

1. 修改了 Account 模块、实现点击头像显示账号信息和账号编辑框（类QQ）

```
帮我参考 goal 的聚合根代码，改善 account 的代码，
1. 使用对象参数来构造
2. 修改构造函数的参数，去除掉 createAt 等应该自动生成的参数
3. 添加完善的辅助函数 
- `toDTO()`： 将对象实例转为对象接口数据
- `static fromDTO(dto: T)`： 将对象接口数据转为对象实例
- `clone()`： 创建对象的副本
- `forCreate()`： 创建对象实例，用于创建模式
- `isxxx()`： 判断对象实例是否为xxx对象实例
- `ensurexxx()`： 确保对象实例为xxx对象实例
- `ensurexxxNeverNull()`： 确保对象实例为xxx对象实例，并且不为null

在 fromDTO 函数中，先用 构造函数生成初步恢复对象，再用 set 方法恢复其他（createAt等）属性
```

### 20250729

1. 编辑 user_profile 数据和业务和页面

遇到了问题：  

我修改代码，触发 vite 的 HMR 热更新后，就会 console.log('Account:', localAccount); 打印这条语句，localAccount 也会有值；但我正常进入或刷新时，就获取不到数据，也不会打印上面的语句；这是什么原因，是初始化顺序的问题吗

..没有 用 computed 来获取 localAccount，以为 使用 pinia 的 get 获取值就能实现响应式，应该还是要用 computed 来获取值。

<details>
<summary>
具体代码
</summary>

```Vue
<template>
  <div class="profile-avatar-wrapper">
    <v-menu
      v-model="showInfo"
      :close-on-content-click="false"
      offset-x
      location="end"
      min-width="260"
      max-width="320"
      transition="slide-x-transition"
    >
      <template #activator="{ props }">
        <v-avatar
          v-bind="props"
          :size="size"
          class="mb-4"
          color="primary"
          style="cursor:pointer;"
        >
          <template v-if="avatarUrl">
            <img :src="avatarUrl" alt="用户头像" />
          </template>
          <template v-else>
            <span class="default-avatar-text">{{ '?' }}</span>
          </template>
        </v-avatar>
      </template>
      <div class="user-info-content pa-4">
        <div class="user-info-header mb-2">
          <v-avatar :size="48" color="primary">
            <template v-if="avatarUrl">
              <img :src="avatarUrl" alt="用户头像" />
            </template>
            <template v-else>
              <span class="default-avatar-text">{{ '?' }}</span>
            </template>
          </v-avatar>
          <div class="user-info-basic">
            <div class="user-name ml-3">{{ localAccount?.username || '未设置昵称' }}<v-icon>{{ localAccount?.user.sex === 'male' ? 'mdi-gender-male' : 'mdi-gender-female' }}</v-icon></div>
            <div class="user-uuid ml-3">{{ localAccount?.uuid || '未设置UUID' }}</div>
          </div>
          
        </div>
        <div class="user-detail mb-2">
          <div>邮箱：{{ localAccount?.email || '未绑定' }}</div>
          <div>手机号：{{ localAccount?.phoneNumber || '未绑定' }}</div>
        </div>
        <v-btn color="primary" variant="tonal" class="mt-3" @click="startEditProfile">
          编辑信息
        </v-btn>
      </div>
    </v-menu>
    <!-- 用户信息编辑框 -->
     <profile-dialog
      :model-value="profileDialog.show"
      :user="User.ensureUserNeverNull(user)"
      @update:model-value="profileDialog.show = $event"
      @handle-update-profile="handleUpdateUserProfile"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { Account } from '../../domain/aggregates/account';
import { User } from '../../domain/entities/user';
import { useAccountStore } from '../stores/accountStore';

// components
import ProfileDialog from './ProfileDialog.vue';
// composables
import { useAccountService } from '../composables/userAccountService';

const accountStore = useAccountStore();
const { handleUpdateUserProfile } = useAccountService();

const props = defineProps<{ avatarUrl?: string, size: string }>();


const localAccount = accountStore.currentAccount;

const user = computed(() => {
  if (localAccount && localAccount.user) {
    return localAccount.user;
  }
  return User.forCreate();
});

const profileDialog = ref<{
    show: boolean;
    account: Account | null;
}>({
    show: false,
    account: null
});

const startEditProfile = () => {
    profileDialog.value.show = true;
    profileDialog.value.account = Account.ensureAccount(localAccount);
};

const showInfo = ref(false);

onMounted(() => {
  // 初始化时获取当前账号信息
  22
  console.log('Account:', localAccount);
});

</script>

<style scoped>
.profile-avatar-wrapper {
  position: relative;
  display: inline-block;
}

.user-info-content {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  background-color: rgba(var(--v-theme-surface), 1);
}

.user-info-header {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-name {
    font-size: 1.1rem;
    font-weight: bold;
  color: rgb(var(--v-theme-font));
}

.user-uuid {
  font-size: 0.9rem;
  color: rgb(var(--v-theme-font));
}

.user-detail {
  font-size: 0.95rem;
  color: #666;
}

.default-avatar-text {
  font-size: 1.5rem;
  font-weight: bold;
}
</style>
```
</details>

2. 整理一下代码，删除了部分无用代码

### 20250731

1. 恢复了误删除的代码
2. 修改了仓储层更新 Goal 的逻辑

之前直接删除 KeyResult 再重新生成新的 KeyResult，来更新 Goal 的关键结果。  
但这由于 Record 表的 `FOREIGN KEY (key_result_uuid) REFERENCES key_results(uuid) ON DELETE SET NULL` 的限制，导致 Record 的关键结果字段被置空。  
现在代码如下：  

```ts
private async updateKeyResultsForGoal(accountUuid: string, goalUuid: string, keyResults: KeyResult[]) {
  // 1. 查询数据库中现有的 keyResult uuid
  const stmt: any = this.db.prepare(`SELECT uuid FROM key_results WHERE goal_uuid = ? AND account_uuid = ?`);
  const dbKRs: { uuid: string }[] = stmt.all(goalUuid, accountUuid);
  const dbKRUuidSet = new Set(dbKRs.map(kr => kr.uuid));
  const inputKRUuidSet = new Set(keyResults.map(kr => kr.uuid));

  // 2. 更新和新增
  for (const kr of keyResults) {
    if (dbKRUuidSet.has(kr.uuid)) {
      // 已存在，执行 UPDATE
      await this.updateKeyResult(accountUuid, goalUuid, kr);
    } else {
      // 不存在，执行 INSERT
      await this.createKeyResult(accountUuid, goalUuid, kr);
    }
  }

  // 3. 删除数据库中有但前端没有的
  for (const dbUuid of dbKRUuidSet) {
    if (!inputKRUuidSet.has(dbUuid)) {
      await this.deleteKeyResult(accountUuid, goalUuid, dbUuid);
    }
  }
}
```

3. 修复了 goalDialog 表单在修改 KeyResult 数据时无法响应式更新的问题
4. 在 goalReview 中添加了图表帮助用户复盘

### 20250801

1. 改善了 goalReview 的页面样式（布局、图标）
2. 重构 sessionLog 模块的代码（改善聚合根代码，修复仓库代码）

仿照 goal仓库（实现 mapRowToxx 、mapxxToRow。。。） 和 sessionLog 表的构造字段，来修改该文件，给出完整代码  

### 20250802

1. 实现 logout 功能
2. 优化 authentication 模块的代码结构

### 20250803

1. 实现 快速登录 功能

帮我把基础的通用的代码放到 common 文件夹中，比如 src/shared/domain 中的基础类（在electron/shared/domain中也有），在主进程和渲染进程中都会使用；
还有 src/shared/events 的事件总线基础类；
现在主进程和渲染进程中同时有相同的基础类，在将这些基础类放到 common 文件夹中后，修改所有使用这些文件的引用路径，使用 `@common/xxx` 来引用；

还有有 `export * from "@electron/shared/domain/domainEvent"` 类似这中引用的类型，都放到 common文件夹中。
一次性完成，直接移动文件，不需要经我确认，正确修改路径引用！！！

当前很多模块的代码的主进程和渲染进程都有相同或者类似的 接口定义文件，重复了，希望你能帮我把他们合并并移动到 common 文件中，比如把这两个 task.ts 移动到 common/modules/task/types/task.ts ， 以 authentication.ts 为例，同时帮我实现基础接口和 主进程、渲染进程下的接口，并区分 接口 和 DTO数据接口，数据接口 应该把时间、boolean转为 number，Map等也修改为可传输类型。
上面是一个例子，请你帮我修改所有还没有修改好的模块

2. 去除掉 DateTime 定义，直接使用 Date;大改 Task 模块相关代码（task 的时间主要采用了 DateTime），修改了 聚合根的 toDTO 相关方法，消除了报错，但是应该仍有 BUG

### 20250804

1. 开始重构 Task 模块
- 修改存储数据表结构，数据库层代码
2. 修复了一个注册账号的BUG，之前意外把带生成认证信息的事件的注册方法删除了，现在重新添加了。

### 20250805

1. 修复了渲染进程获取不到 taskInstance 的 bug
2. 优化了 TaskInstanceManagement.vue 的页面
3. 删除了 goal 表 的 目录id 要求
