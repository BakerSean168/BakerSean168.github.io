---
title: Electron-Vue3项目数据的本地存储
categories:
  - 前端
description: Electron-Vue3项目数据的本地存储
date: 2025-06-02 12:13:54
tags:
---

# 准备

## 确定需要实现的功能和代码大致结构层次

项目会产生目标相关（目标、文件夹等）、仓库相关、设置相关、任务相关等的数据  
将这些数据存储到本地数据库中

- 存储
  基础服务，要实现 数据库 到 前端服务层  
  实现前端能调用服务来实现对数据的增删读写  
- 初始化
  基于存储服务  
  在登录成功时调用，加载该用户的数据
- 保存
  composable，可复用函数  
  基于存储服务的读取服务


表结构：
```ts
// 创建用户数据存储表
db.exec(`
  CREATE TABLE IF NOT EXISTS user_store_data (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL,
    store_name TEXT NOT NULL,
    data TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    UNIQUE(username, store_name),
    FOREIGN KEY (username) REFERENCES users(username) ON DELETE CASCADE
  )
`);
```

通过 username，store_name 来获取整个 store 保存的数据


## 怎么实现用户登录时，初始化用户数据的功能（利用 storeService，从数据表中读取每个 store 的数据，加载到相应的 store 中）

增加一个 userDataInitService  
用来给每个模块读取用户数据（登录时）、清空所有数据（登出时）、保存所有数据到数据库（内容修改时、退出时）

<details>
<summary>userDataInitService.ts 具体代码</summary>

```ts
import { UserStoreService } from "./userStoreService";
import { useGoalStore } from "@/modules/Goal/stores/goalStore";
import { useTaskStore } from "@/modules/Task/stores/taskStore";
import { useGoalDirStore } from "@/modules/Goal/stores/goalDirStore";
import { useGoalReviewStore } from "@/modules/Goal/stores/goalReviewStore";
import { useReminderStore } from "@/modules/Reminder/reminderStore";
import { useRepositoryStore } from "@/modules/Repository/stores/repositoryStore";
import { useSettingStore } from "@/modules/Setting/stores/settingStore";
import type { IGoal, IRecord, IGoalDir } from "@/modules/Goal/types/goal";
import type { ITaskInstance, ITaskTemplate } from "@/modules/Task/types/task";
import type { Review } from "@/modules/Goal/stores/goalReviewStore";
import type { Reminder } from "@/modules/Reminder/reminderStore";
import type { Repository } from "@/modules/Repository/stores/repositoryStore";
import type { AppSetting } from "@/modules/Setting/stores/settingStore";

/**
 * 用户数据初始化服务
 * 负责用户登录时从存储中加载各个模块的数据到对应的 store
 */
export class UserDataInitService {
  /**
   * 初始化用户所有数据
   * @param username 用户名（可选，如果不提供则使用当前登录用户）
   */
  static async initUserData(username?: string): Promise<void> {
    try {
      const targetUsername = username || UserStoreService.getCurrentUser();

      if (!targetUsername) {
        throw new Error("没有找到用户信息，无法初始化数据");
      }

      console.log(`开始初始化用户 ${targetUsername} 的数据...`);

      // 并行加载各个模块的数据
      await Promise.all([
        this.initGoalData(targetUsername),
        this.initTaskData(targetUsername),
        this.initGoalDirData(targetUsername),
        this.initGoalReviewData(targetUsername),
        this.initReminderData(targetUsername),
        this.initRepositoryData(targetUsername),
        this.initSettingData(targetUsername),
      ]);

      console.log(`用户 ${targetUsername} 数据初始化完成`);
    } catch (error) {
      console.error(`用户数据初始化失败:`, error);
      throw error;
    }
  }

  /**
   * 初始化目标模块数据
   */
  private static async initGoalData(username: string): Promise<void> {
    try {
      const goalStore = useGoalStore();

      const [goalsResponse, recordsResponse] = await Promise.all([
        UserStoreService.readWithUsername<IGoal[]>(username, "goals"),
        UserStoreService.readWithUsername<IRecord[]>(username, "records"),
      ]);

      goalStore.$patch({
        goals:
          goalsResponse.success && goalsResponse.data ? goalsResponse.data : [],
        records:
          recordsResponse.success && recordsResponse.data
            ? recordsResponse.data
            : [],
      });

      const goalsCount =
        goalsResponse.success && goalsResponse.data
          ? goalsResponse.data.length
          : 0;
      const recordsCount =
        recordsResponse.success && recordsResponse.data
          ? recordsResponse.data.length
          : 0;

      console.log(`加载了 ${goalsCount} 个目标和 ${recordsCount} 条记录`);
    } catch (error) {
      console.error("目标数据初始化失败:", error);
      const goalStore = useGoalStore();
      goalStore.$patch({
        goals: [],
        records: [],
      });
      throw error;
    }
  }

  /**
   * 清空所有 store 数据（用户退出登录时调用）
   */
  static clearAllStoreData(): void {
    try {
      const goalStore = useGoalStore();

      // 重置各个 store
      goalStore.goals = [];
      goalStore.records = [];
      goalStore.initTempGoal();
      goalStore.initTempKeyResult();

      console.log("所有 store 数据已清空");
    } catch (error) {
      console.error("清空 store 数据失败:", error);
    }
  }

  /**
   * 保存用户数据到存储
   */
  static async saveUserData(): Promise<void> {
    try {
      if (!UserStoreService.isUserLoggedIn()) {
        throw new Error("用户未登录，无法保存数据");
      }

      const goalStore = useGoalStore();

      // 并行保存所有数据
      const results = await Promise.all([
        UserStoreService.write("goals", goalStore.goals),
      ]);

      // 检查保存结果
      const failedSaves = results.filter((result) => !result.success);
      if (failedSaves.length > 0) {
        console.error("部分数据保存失败:", failedSaves);
        throw new Error("部分数据保存失败");
      }

      console.log(`用户数据保存成功`);
    } catch (error) {
      console.error(`用户数据保存失败:`, error);
      throw error;
    }
  }

  /**
   * 获取用户存储列表
   */
  static async getUserStoreList(): Promise<string[]> {
    try {
      const response = await UserStoreService.list();
      if (response.success && response.data) {
        return response.data;
      }
      return [];
    } catch (error) {
      console.error("获取用户存储列表失败:", error);
      return [];
    }
  }
}
```
</details>

## 实现数据更改时，将数据保存到数据库中

注意事项：  
ipc 通信要将数据序列化，所以调用 IPC 接口时要调用 `JSON.stringify()` 将数据序列化；  
如果后端还要处理数据，则在后端再转回 JS 对象；  
数据库中一般存储 JSON 数据。

先提示 有对象不能被克隆（未序列化）  
然后又在读取对象时报错（ipc 前序列化了，数据库里也序列化了，导致双重序列化，数据出问题）

对简单的 storeService 进行封装，实现防抖功能
<details>
<summary> useStoreSave.ts 具体代码</summary>

```ts
// useStoreSave.ts
import { UserStoreService } from "@/shared/services/userStoreService";
import { ref, type Ref } from "vue";

interface StoreSaveOptions {
  delay?: number; // 防抖延迟，默认 1000ms
  onSuccess?: (storeName: string) => void; // 保存成功回调
  onError?: (storeName: string, error: any) => void; // 保存失败回调
}

export function useStoreSave(options: StoreSaveOptions = {}) {
  const { delay = 1000, onSuccess, onError } = options;
  const timeouts = new Map<string, NodeJS.Timeout>();
  const saving = ref<Set<string>>(new Set());

  /**
   * 防抖保存数据
   */
  const debounceSave = async <T>(
    storeName: string,
    data: T
  ): Promise<boolean> => {
    // 清除之前的定时器
    if (timeouts.has(storeName)) {
      clearTimeout(timeouts.get(storeName)!);
    }

    return new Promise((resolve) => {
      const timeout = setTimeout(async () => {
        try {
          saving.value.add(storeName);
          let JSON_data = JSON.stringify(data);
          const response = await UserStoreService.write(storeName, JSON_data);

          if (response.success) {
            onSuccess?.(storeName);
            resolve(true);
          } else {
            onError?.(storeName, response.message);
            resolve(false);
          }
        } catch (error) {
          onError?.(storeName, error);
          resolve(false);
        } finally {
          saving.value.delete(storeName);
          timeouts.delete(storeName);
        }
      }, delay);

      timeouts.set(storeName, timeout);
    });
  };

  /**
   * 立即保存数据
   */
  const saveImmediately = async <T>(
    storeName: string,
    data: T
  ): Promise<boolean> => {
    try {
      saving.value.add(storeName);

      let JSON_data = JSON.stringify(data);
      const response = await UserStoreService.write(storeName, JSON_data);

      if (response.success) {
        onSuccess?.(storeName);
        return true;
      } else {
        onError?.(storeName, response.message);
        return false;
      }
    } catch (error) {
      onError?.(storeName, error);
      return false;
    } finally {
      saving.value.delete(storeName);
    }
  };

  /**
   * 检查是否正在保存
   */
  const isSaving = (storeName?: string): boolean => {
    if (storeName) {
      return saving.value.has(storeName);
    }
    return saving.value.size > 0;
  };

  /**
   * 清理定时器
   */
  const cleanup = () => {
    timeouts.forEach((timeout) => clearTimeout(timeout));
    timeouts.clear();
    saving.value.clear();
  };

  return {
    debounceSave,
    saveImmediately,
    isSaving,
    cleanup,
    saving: saving as Ref<ReadonlySet<string>>,
  };
}
```
<details>

在 store 中使用：

<details>
<summary>在 settingStore 中的使用案例</summary>

```ts
import { defineStore } from "pinia";
import { useThemeStore } from "@/modules/Theme/themeStroe";
import { useStoreSave } from "@/shared/composables/useStoreSave";
import { setLanguage } from "@/i18n";

// ...existing interfaces...

// 在模块级别创建 autoSave 实例
let autoSaveInstance: ReturnType<typeof useStoreSave> | null = null;

function getAutoSave() {
  if (!autoSaveInstance) {
    autoSaveInstance = useStoreSave({
      onSuccess: (storeName) => console.log(`✓ ${storeName} 数据保存成功`),
      onError: (storeName, error) =>
        console.error(`✗ ${storeName} 数据保存失败:`, error),
    });
  }
  return autoSaveInstance;
}

export const useSettingStore = defineStore("setting", {
  actions: {
    // 自动保存方法
    async saveSettings(): Promise<boolean> {
      const autoSave = getAutoSave();
      return autoSave.debounceSave("settings", this.$state);
    },

    async saveSettingsImmediately(): Promise<boolean> {
      const autoSave = getAutoSave();
      return autoSave.saveImmediately("settings", this.$state);
    },

    // 检查保存状态
    isSavingSettings(): boolean {
      const autoSave = getAutoSave();
      return autoSave.isSaving("settings");
    },

    //在对应的增删改 服务中调用保存方法
    // 修改现有方法，添加自动保存
    async setTheme(themeMode: string) {
      this.themeMode = themeMode;
      const themeStore = useThemeStore();
      themeStore.setCurrentTheme(themeMode);

      // 自动保存
      const saveSuccess = await this.saveSettings();
      if (!saveSuccess) {
        console.error("主题设置保存失败");
      }
    },
  },
});
```

另一种调用方法：

```ts
actions: {
    // 初始化自动保存（在需要时调用）
    _initAutoSave() {
        if (!this._autoSave) {
          this._autoSave = useStoreSave({
            onSuccess: (storeName) => console.log(`✓ ${storeName} 数据保存成功`),
            onError: (storeName, error) => console.error(`✗ ${storeName} 数据保存失败:`, error),
          });
        }
        return this._autoSave;
      },

      // 自动保存方法
      async saveGoals() {
        const autoSave = this._initAutoSave();
        return autoSave.debounceSave('goals', this.goals);
      },

      async saveRecords() {
        const autoSave = this._initAutoSave();
        return autoSave.debounceSave('records', this.records);
      },

      async saveGoalsImmediately() {
        const autoSave = this._initAutoSave();
        return autoSave.saveImmediately('goals', this.goals);
      },

      async saveRecordsImmediately() {
        const autoSave = this._initAutoSave();
        return autoSave.saveImmediately('records', this.records);
      },

      // 检查保存状态
      isSavingGoals() {
        const autoSave = this._initAutoSave();
        return autoSave.isSaving('goals');
      },

      isSavingRecords() {
        const autoSave = this._initAutoSave();
        return autoSave.isSaving('records');
      },

      isSavingAny() {
        const autoSave = this._initAutoSave();
        return autoSave.isSaving();
      },

}
```

</details>