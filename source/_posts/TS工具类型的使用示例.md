---
title: TS工具类型的使用示例
categories:
  - null
description: TS工具类型的使用示例
date: 2025-06-12 15:31:55
tags:
---

```ts
// TypeScript 工具类型使用示例
// 展示如何从复杂类型中提取子类型

import type { ReminderRule, ReminderAlert, ReminderTiming } from '../../types/timeStructure';

// ===== 方案1: 使用 TypeScript 工具类型 (推荐) =====

/**
 * 使用工具类型的优势:
 * 1. 自动同步：ReminderRule 变化时，ReminderAlert 自动更新
 * 2. 类型安全：确保类型一致性
 * 3. 维护简单：只需要维护一个主类型
 */

// 提取单个提醒项类型
type MyReminderAlert = ReminderRule['alerts'][number];

// 提取时机配置类型
type MyReminderTiming = ReminderAlert['timing'];

// 提取提醒类型枚举
type ReminderType = ReminderAlert['type']; // "notification" | "email" | "sound"

// 提取时机类型枚举
type TimingType = ReminderTiming['type']; // "relative" | "absolute"

// ===== 其他有用的工具类型示例 =====

// 1. 使用 Pick 选择特定字段
type ReminderAlertMinimal = Pick<ReminderAlert, 'id' | 'type'>;

// 2. 使用 Omit 排除特定字段
type ReminderAlertWithoutTrigger = Omit<ReminderAlert, 'triggered'>;

// 3. 使用 Partial 使所有字段可选
type PartialReminderAlert = Partial<ReminderAlert>;

// 4. 使用 Required 使所有字段必选
type RequiredReminderAlert = Required<ReminderAlert>;

// ===== 实际应用示例 =====

/**
 * 创建新提醒项的工厂函数
 */
export function createReminderAlert(
  type: ReminderAlert['type'],
  minutesBefore: number,
  message?: string
): ReminderAlert {
  return {
    id: crypto.randomUUID(),
    timing: {
      type: 'relative',
      minutesBefore
    },
    type,
    message,
    triggered: false
  };
}

/**
 * 验证提醒项的函数
 */
export function validateReminderAlert(alert: ReminderAlert): boolean {
  if (!alert.id || !alert.type) {
    return false;
  }
  
  if (alert.timing.type === 'relative') {
    return !!alert.timing.minutesBefore && alert.timing.minutesBefore > 0;
  }
  
  if (alert.timing.type === 'absolute') {
    return !!alert.timing.absoluteTime;
  }
  
  return false;
}

/**
 * 更新提醒时机的函数
 */
export function updateReminderTiming(
  alert: ReminderAlert,
  timing: ReminderTiming
): ReminderAlert {
  return {
    ...alert,
    timing
  };
}

// ===== 对比：如果要手动定义类型 =====

/**
 * 方案2: 手动定义 (不推荐，但有时必要)
 * 缺点：需要手动保持同步，容易出现不一致
 */

// export type ReminderAlert = {
//   id: string;
//   timing: {
//     type: "relative" | "absolute";
//     minutesBefore?: number;
//     absoluteTime?: DateTime;
//   };
//   type: "notification" | "email" | "sound";
//   message?: string;
//   triggered?: boolean;
// };

export {};
```