---
title: todo代办任务模块的实现
categories:
  - null
description: todo代办任务模块的实现
date: 2025-06-07 14:20:05
tags:
---

# 准备

目标是添加待办事项，高大上一点，任务管理系统！！  
重点在于实现复杂的任务创建、管理：  
- 任务的信息
- 任务的开始时间、结束时间、时间点或时间段任务（有些任务可能只关注开始时间，如8点开始抢票；有的则关注开始时间到结束时间，具体活动）
- 重复任务（开始时间、结束时间、重复类型（每日、每周））
- 任务提醒（提前多少分钟/具体时间）

*模板-实例模式（Template-Instance Pattern） 是一种将可复用的配置（模板）与具体执行（实例）分离的设计模式。在TODO应用中，这种模式特别适用于处理重复性任务。*
要实现重复任务，可以采用 模板-实例 模式  
- 模板负责定义任务的蓝图
- 实例负责生成具体的任务

我们需要的数据有很多  
先思考任务的数据结构  

<details>
<summary>时间结构</summary>

```ts
/**
 * 时间点类型 - 更精确的时间表示
 */
export type TimePoint = {
  /** 小时 (0-23) */
  hour: number;
  /** 分钟 (0-59) */
  minute: number;
  /** 时区 (可选，默认本地时区) */
  timezone?: string;
};

/**
 * 时间段类型
 */
export type TimeRange = {
  /** 开始时间 */
  start: TimePoint;
  /** 结束时间 */
  end: TimePoint;
  /** 是否跨天 */
  crossDay?: boolean;
};

/**
 * 日期类型 - 使用更标准的格式
 */
export type DateInfo = {
  /** 年份 */
  year: number;
  /** 月份 (1-12) */
  month: number;
  /** 日期 (1-31) */
  day: number;
};

/**
 * 完整的日期时间类型
 */
export type DateTime = {
  /** 日期信息 */
  date: DateInfo;
  /** 时间信息 */
  time?: TimePoint;
  /** UTC 时间戳 (毫秒) - 用于比较和排序 */
  timestamp: number;
  /** ISO 字符串 - 用于存储和传输 */
  isoString: string;
};

/**
 * 重复规则 - 更灵活的重复模式
 */
export type RecurrenceRule = {
  /** 重复类型 */
  type: "none" | "daily" | "weekly" | "monthly" | "yearly" | "custom";
  /** 重复间隔 (例如：每2天、每3周) */
  interval?: number;
  /** 结束条件 */
  endCondition: {
    /** 结束类型 */
    type: "never" | "date" | "count";
    /** 结束日期 (当 type 为 'date') */
    endDate?: DateTime;
    /** 重复次数 (当 type 为 'count') */
    count?: number;
  };
  /** 重复的具体配置 */
  config?: {
    /** 周重复：星期几 (0=周日, 1=周一, ..., 6=周六) */
    weekdays?: number[];
    /** 月重复：每月的第几天 */
    monthDays?: number[];
    /** 月重复：每月的第几个星期几 (如第二个周一) */
    monthWeekdays?: Array<{
      week: number; // 第几周 (1-5, -1表示最后一周)
      weekday: number; // 星期几 (0-6)
    }>;
    /** 年重复：月份 */
    months?: number[];
  };
};

/**
 * 提醒规则 - 更灵活的提醒设置
 */
export type ReminderRule = {
  /** 是否启用提醒 */
  enabled: boolean;
  /** 多个提醒时间点 */
  alerts: Array<{
    /** 提醒ID */
    id: string;
    /** 提前时间 (分钟) */
    minutesBefore: number;
    /** 提醒类型 */
    type: "notification" | "email" | "sound";
    /** 自定义消息 */
    message?: string;
    /** 是否已触发 */
    triggered?: boolean;
  }>;
  /** 提醒重复设置 */
  snooze?: {
    /** 是否允许稍后提醒 */
    enabled: boolean;
    /** 稍后提醒间隔 (分钟) */
    interval: number;
    /** 最大重复次数 */
    maxCount: number;
  };
};

/**
 * 任务时间配置 - 整合所有时间相关信息
 */
export type TaskTimeConfig = {
  /** 任务类型 */
  type: "allDay" | "timed" | "timeRange";
  /** 基础时间信息 */
  baseTime: {
    /** 开始时间 */
    start: DateTime;
    /** 结束时间 (可选) */
    end?: DateTime;
    /** 持续时间 (分钟，用于验证) */
    duration?: number;
  };
  /** 重复规则 */
  recurrence: RecurrenceRule;
  /** 提醒规则 */
  reminder: ReminderRule;
  /** 时区信息 */
  timezone: string;
  /** 夏令时处理 */
  dstHandling?: "auto" | "ignore";
};

```

</details>

<details>
<summary>任务的数据结构</summary>

```ts
import type { TaskTimeConfig, DateTime } from "./timeStructure";

/**
 * 关键结果关联
 */
export type KeyResultLink = {
  /** 目标ID */
  goalId: string;
  /** 关键结果ID */
  keyResultId: string;
  /** 完成任务时增加的值 */
  incrementValue: number;
};

export type ITaskTemplate = {
  /** 任务模板ID */
  id: string;
  /** 任务标题 */
  title: string;
  /** 任务描述 */
  description?: string;
  /** 时间配置 - 使用新的时间结构 */
  timeConfig: TaskTimeConfig;
  /** 调度策略 */
  schedulingPolicy?: {
    allowReschedule: boolean;
    maxDelayDays: number;
    skipWeekends: boolean;
    skipHolidays: boolean;
    workingHoursOnly: boolean;
  };
  /** 元素据 */
  metadata: {
    category: string;
    tags: string[];
    estimatedDuration?: number; // 分钟
    difficulty: 1 | 2 | 3 | 4 | 5;
    location?: string;
  };
  /** 生命周期 */
  lifecycle: {
    status: 'draft' | 'active' | 'paused' | 'archived';
    createdAt: DateTime;
    updatedAt: DateTime;
    activatedAt?: DateTime;
    pausedAt?: DateTime;
  };
  /** 统计数据 */
  analytics: {
    totalInstances: number;
    completedInstances: number;
    averageCompletionTime?: number;
    successRate: number;
    lastInstanceDate?: DateTime;
  };
  /** 关联的关键结果 */
  keyResultLinks?: KeyResultLink[];
  /** 任务优先级 */
  priority?: 1 | 2 | 3 | 4;
  /** 创建时间 */
  createdAt: DateTime;
  /** 更新时间 */
  updatedAt: DateTime;
  /** 版本号 - 用于数据迁移 */
  version: number;
};

export type ITaskInstance = {
  /** 任务实例ID */
  id: string;
  /** 关联的任务模板ID */
  templateId: string;
  /** 任务标题 */
  title: string;
  /** 任务描述 */
  description?: string;
  /** 计划执行时间 */
  scheduledTime: DateTime;
  /** 实际开始时间 */
  actualStartTime?: DateTime;
  /** 实际结束时间 */
  actualEndTime?: DateTime;
  /** 任务关联的关键结果 */
  keyResultLinks?: KeyResultLink[];
  /** 任务优先级 */
  priority: 1 | 2 | 3 | 4;
  /** 任务状态 */
  status: "pending" | "inProgress" | "completed" | "cancelled" | "overdue";
  /** 完成时间 */
  completedAt?: DateTime;
  /** 提醒状态 */
  reminderStatus: {
    /** 已触发的提醒 */
    triggeredAlerts: string[];
    /** 下一个提醒时间 */
    nextReminderTime?: DateTime;
    /** 稍后提醒次数 */
    snoozeCount: number;
  };
};

/**
 * 临时任务模板（用于表单）- 兼容旧格式
 */
export type ITempTaskTemplate = {
  id: string;
  title: string;
  description?: string;
  startTime?: string;
  endTime?: string;
  repeatPattern: {
    type: "none" | "daily" | "weekly" | "monthly";
    days?: number[];
    startDate: string;
    endDate?: string;
  };
  reminderPattern: {
    isReminder: boolean;
    timeBefore?: "5" | "10" | "15" | "30" | "60";
  };
  keyResultLinks?: KeyResultLink[];
  priority: 1 | 2 | 3 | 4;
  createdAt: string;
  updatedAt: string;
};

/**
 * 任务统计
 */
export interface ITaskStats {
  /** 总任务数 */
  total: number;
  /** 已完成数 */
  completed: number;
  /** 今日完成数 */
  todayCompleted: number;
  /** 逾期数 */
  overdue: number;
}

```

</details>


<details>
<summary>任务的数据结构</summary>

```ts

```

</details>

# 代码实现

配置数据很多，如果每次都逐个配置的话，十分影响用户体验  
可以创建一个 模板工厂工具类 来生成常用模板（即模板的模板）  
可以是 **给定常见的任务，自己配置其他**， 或者 **给定常见的时间设置，配置任务内容**

1. 点击添加按钮，弹出模板选择
2. 选择模板，根据模板返回预配置数据的任务模板编辑对话框
3. 进入编辑框，编辑数据
4. 点击保存按钮，调用添加服务

## 1. 根据数据结构创建 任务模板工厂

作为工具类来生成模板  
即任务模板的模板，初始就先一个选项「自定义」，后续可根据用户需求调研来添加例如工作安排模板

<details>
<summary>模板工厂代码</summary>

```ts
 // src/modules/Task/utils/taskTemplateFactory.ts
import { TimeUtils } from './timeUtils';
import type { TaskTemplate, CreateTaskTemplateOptions } from '../types/task';
import { v4 as uuidv4 } from 'uuid';

export class TaskTemplateFactory {
  /**
   * 创建新的空白任务模板
   */
  static createEmpty(options: CreateTaskTemplateOptions = {}): TaskTemplate {
    const now = TimeUtils.now();
    const today = TimeUtils.createDateTime(
      now.date.year,
      now.date.month,
      now.date.day,
      {
        hour: 0,
        minute: 0
      }
    );

    return {
      id: uuidv4(),
      title: options.title || '',
      description: options.description || '',

      timeConfig: {
        type: 'timed', // 默认指定时间
        baseTime: {
          start: today,
          end: TimeUtils.addMinutes(today, 60) // 默认1小时
        },
        recurrence: {
          type: 'none', // 默认不重复
          interval: 1,
          endCondition: {
            type: 'never'
          }
        },
        reminder: {
          enabled: false,
          alerts: [],
          snooze: {
            enabled: false,
            interval: 5, // 默认5分钟
            maxCount: 1 // 默认最多1次
          }
        },
        timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
      },

      schedulingPolicy: {
        allowReschedule: true,
        maxDelayDays: 3,
        skipWeekends: false,
        skipHolidays: false,
        workingHoursOnly: false
      },

      metadata: {
        category: options.category || 'general',
        tags: [],
        estimatedDuration: 60, // 默认60分钟
        difficulty: options.priority || 2,
        location: ''
      },

      lifecycle: {
        status: 'draft',
        createdAt: now,
        updatedAt: now
      },

      analytics: {
        totalInstances: 0,
        completedInstances: 0,
        successRate: 0
      },

      keyResultLinks: options.keyResultLinks || [],
      priority: options.priority || 2,
      createdAt: now,
      updatedAt: now,
      version: 1
    };
  }

  /**
   * 从现有模板创建副本
   */
  static createFromTemplate(template: TaskTemplate): TaskTemplate {
    const now = TimeUtils.now();
    
    return {
      ...template,
      id: uuidv4(),
      title: `${template.title} - 副本`,
      lifecycle: {
        ...template.lifecycle,
        status: 'draft',
        createdAt: now,
        updatedAt: now,
        activatedAt: undefined,
        pausedAt: undefined
      },
      analytics: {
        totalInstances: 0,
        completedInstances: 0,
        averageCompletionTime: undefined,
        successRate: 0,
        lastInstanceDate: undefined
      },
      createdAt: now,
      updatedAt: now
    };
  }

  /**
   * 根据模板类型创建预配置的任务模板
   */
  static createByType(templateType: string, options: CreateTaskTemplateOptions = {}): TaskTemplate {
    switch (templateType) {
      case 'empty':
        return this.createEmpty(options);
      case 'habit':
        return this.createHabitTemplate(options);
      default:
        return this.createEmpty(options);
    }
  }

  /**
   * 习惯养成模板
   */
  static createHabitTemplate(options: CreateTaskTemplateOptions = {}): TaskTemplate {
    const template = this.createEmpty(options);
    
    return {
      ...template,
      title: options.title || '新习惯',
      timeConfig: {
        ...template.timeConfig,
        type: 'timed',
        recurrence: {
          type: 'daily',
          interval: 1,
          endCondition: {
            type: 'count',
            count: 21 // 21天习惯养成
          }
        },
        reminder: {
          enabled: true,
          alerts: [{
            id: uuidv4(),
            timing: {
              type: 'relative',
              minutesBefore: 15
            },
            type: 'notification',
            message: '是时候培养好习惯了！'
          }],
          snooze: {
            enabled: true,
            interval: 5,
            maxCount: 3
          }
        }
      },
      metadata: {
        ...template.metadata,
        category: 'habit',
        tags: ['习惯', '自我提升'],
        estimatedDuration: 30
      }
    };
  }
  

  /**
   * 验证模板数据
   */
  static validate(template: TaskTemplate): { valid: boolean; errors: string[] } {
    const errors: string[] = [];

    if (!template.title.trim()) {
      errors.push('任务标题不能为空');
    }

    if (template.timeConfig.type === 'timed' || template.timeConfig.type === 'timeRange') {
      if (!template.timeConfig.baseTime.start) {
        errors.push('必须设置开始时间');
      }
      
      if (template.timeConfig.type === 'timeRange' && !template.timeConfig.baseTime.end) {
        errors.push('时间段模式下必须设置结束时间');
      }

      if (template.timeConfig.baseTime.start && template.timeConfig.baseTime.end) {
        if (template.timeConfig.baseTime.end.timestamp <= template.timeConfig.baseTime.start.timestamp) {
          errors.push('结束时间必须晚于开始时间');
        }
      }
    }

    if (template.timeConfig.recurrence.type !== 'none') {
      if (template.timeConfig.recurrence.endCondition.type === 'date' && 
          !template.timeConfig.recurrence.endCondition.endDate) {
        errors.push('重复任务必须设置结束日期');
      }

      if (template.timeConfig.recurrence.endCondition.type === 'count' && 
          (!template.timeConfig.recurrence.endCondition.count || 
           template.timeConfig.recurrence.endCondition.count <= 0)) {
        errors.push('重复次数必须大于0');
      }
    }

    // 验证关键结果链接
    if (template.keyResultLinks) {
      template.keyResultLinks.forEach((link, index) => {
        if (!link.goalId) {
          errors.push(`第${index + 1}个关联缺少目标ID`);
        }
        if (!link.keyResultId) {
          errors.push(`第${index + 1}个关联缺少关键结果ID`);
        }
        if (link.incrementValue <= 0) {
          errors.push(`第${index + 1}个关联的增加值必须大于0`);
        }
      });
    }


    
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
}
```

</details>

## 2. 创建 时间工具类

任务数据中，时间毫无疑问是最重要的数据之一，上面的数据结构中也给出了一个独特的时间类型
```ts
export type DateTime = {
  /** 日期信息 */
  date: DateInfo;
  /** 时间信息 */
  time?: TimePoint;
  /** UTC 时间戳 (毫秒) - 用于比较和排序 */
  timestamp: number;
  /** ISO 字符串 - 用于存储和传输 */
  isoString: string;
};
```

对于这中自定义的时间类型，需要创建一个时间工具类来提供相关的帮助，比如将 iso 时间转为该类型等  

<details>
<summary>自定义的时间的工具类</summary>

```ts
// src/modules/Task/utils/timeUtils.ts
import type { TimePoint, DateTime, DateInfo, TaskTimeConfig, ReminderRule, RecurrenceRule } from '../types/timeStructure';

export class TimeUtils {
  /**
   * 创建时间点
   */
  static createTimePoint(hour: number, minute: number, timezone?: string): TimePoint {
    return {
      hour: Math.max(0, Math.min(23, hour)),
      minute: Math.max(0, Math.min(59, minute)),
      timezone
    };
  }

  /**
   * 创建日期时间
   */
  static createDateTime(year: number, month: number, day: number, time?: TimePoint): DateTime {
    const date: DateInfo = { year, month, day };
    const jsDate = new Date(year, month - 1, day, time?.hour || 0, time?.minute || 0);
    
    return {
      date,
      time,
      timestamp: jsDate.getTime(),
      isoString: jsDate.toISOString()
    };
  }

  /**
   * 检查是否为 DateTime 对象
   */
  static isDateTime(obj: any): obj is DateTime {
    return obj && 
           typeof obj === 'object' && 
           obj.timestamp && 
           obj.isoString && 
           obj.date;
  }

  /**
   * 从ISO字符串创建DateTime
   */
  static fromISOString(isoString: string): DateTime {
    const jsDate = new Date(isoString);
    return {
      date: {
        year: jsDate.getFullYear(),
        month: jsDate.getMonth() + 1,
        day: jsDate.getDate()
      },
      time: {
        hour: jsDate.getHours(),
        minute: jsDate.getMinutes()
      },
      timestamp: jsDate.getTime(),
      isoString
    };
  }

  /**
   * 获取当前时间
   */
  static now(): DateTime {
    return this.fromTimestamp(Date.now());
  }

  /**
   * 从时间戳创建DateTime
   */
  static fromTimestamp(timestamp: number): DateTime {
    const jsDate = new Date(timestamp);
    return {
      date: {
        year: jsDate.getFullYear(),
        month: jsDate.getMonth() + 1,
        day: jsDate.getDate()
      },
      time: {
        hour: jsDate.getHours(),
        minute: jsDate.getMinutes()
      },
      timestamp,
      isoString: jsDate.toISOString()
    };
  }

   /**
   * 从各种格式创建 DateTime
   */
   static toDateTime(input: Date | string | number | DateTime): DateTime {
    if (this.isDateTime(input)) {
      return input as DateTime;
    }
    
    if (typeof input === 'number') {
      return this.fromTimestamp(input);
    }
    
    if (typeof input === 'string') {
      return this.fromISOString(input);
    }
    
    if (input instanceof Date) {
      return this.fromTimestamp(input.getTime());
    }
    
    throw new Error('Invalid input type for DateTime conversion');
  }

  /**
   * 比较两个DateTime
   */
  static compare(dt1: DateTime, dt2: DateTime): number {
    return dt1.timestamp - dt2.timestamp;
  }

  /**
   * 批量比较多个时间
   */
  static compareMultiple(times: DateTime[]): DateTime[] {
    return [...times].sort((a, b) => this.compare(a, b));
  }

  /**
   * 时间比较的便捷方法
   */
  static isAfter(dt1: DateTime, dt2: DateTime): boolean {
    return this.compare(dt1, dt2) > 0;
  }

  static isBefore(dt1: DateTime, dt2: DateTime): boolean {
    return this.compare(dt1, dt2) < 0;
  }

  static isEqual(dt1: DateTime, dt2: DateTime): boolean {
    return this.compare(dt1, dt2) === 0;
  }

  /**
   * 检查时间是否在指定范围内
   */
  static isInRange(target: DateTime, start: DateTime, end: DateTime): boolean {
    return target.timestamp >= start.timestamp && target.timestamp <= end.timestamp;
  }

  /**
   * 计算下一个重复时间
   */
  static getNextOccurrence(config: TaskTimeConfig, from?: DateTime): DateTime | null {
    const fromTime = from || this.now();
    const { recurrence, baseTime } = config;

    if (recurrence.type === 'none') {
      return baseTime.start.timestamp > fromTime.timestamp ? baseTime.start : null;
    }

    // 根据重复规则计算下一次时间
    return this.calculateNextByRule(baseTime.start, recurrence, fromTime);
  }

  /**
   * 计算所有提醒时间
   */
  static calculateReminderTimes(taskTime: DateTime, reminderRule: ReminderRule): DateTime[] {
    if (!reminderRule.enabled) return [];

    return reminderRule.alerts
      .filter(alert => !alert.triggered)
      .map(alert => {
        if (alert.timing.type === 'absolute' && alert.timing.absoluteTime) {
          return alert.timing.absoluteTime;
        } else if (alert.timing.type === 'relative' && alert.timing.minutesBefore) {
          return this.addMinutes(taskTime, -alert.timing.minutesBefore);
        } else {
          throw new Error('Invalid reminder timing configuration');
        }
      })
      .filter(reminderTime => reminderTime.timestamp > Date.now())
      .sort((a, b) => a.timestamp - b.timestamp);
  }

  

  /**
   * 添加一段时间（以分钟为单位）
   */
  static addMinutes(dateTime: DateTime, minutes: number): DateTime {
    const newTimestamp = dateTime.timestamp + (minutes * 60 * 1000);
    return this.fromTimestamp(newTimestamp);
  }
  
  /**
   * 根据重复规则计算下一次时间
   */
  private static calculateNextByRule(
    baseTime: DateTime, 
    rule: RecurrenceRule, 
    fromTime: DateTime
  ): DateTime | null {
    const { type, interval = 1, endCondition, config } = rule;
    
    // 检查是否已达到结束条件
    if (endCondition.type === 'date' && endCondition.endDate && 
        fromTime.timestamp > endCondition.endDate.timestamp) {
      return null;
    }

    let nextTime: DateTime;
    const baseDate = new Date(baseTime.timestamp);
    const fromDate = new Date(fromTime.timestamp);

    switch (type) {
      case 'daily':
        nextTime = this.getNextDaily(baseDate, fromDate, interval);
        break;
      case 'weekly':
        nextTime = this.getNextWeekly(baseDate, fromDate, interval, config?.weekdays);
        break;
      case 'monthly':
        nextTime = this.getNextMonthly(baseDate, fromDate, interval, config);
        break;
      default:
        return null;
    }

    return nextTime;
  }

  private static getNextDaily(baseDate: Date, fromDate: Date, interval: number): DateTime {
    const nextDate = new Date(Math.max(baseDate.getTime(), fromDate.getTime()));
    if (nextDate.getTime() === fromDate.getTime()) {
      nextDate.setDate(nextDate.getDate() + interval);
    }
    return this.fromTimestamp(nextDate.getTime());
  }

  private static getNextWeekly(
    baseDate: Date, 
    fromDate: Date, 
    interval: number, 
    weekdays?: number[]
  ): DateTime {
    if (!weekdays || weekdays.length === 0) {
      const nextDate = new Date(Math.max(baseDate.getTime(), fromDate.getTime()));
      nextDate.setDate(nextDate.getDate() + (7 * interval));
      return this.fromTimestamp(nextDate.getTime());
    }

    // 找到下一个匹配的星期几
    const currentWeekday = fromDate.getDay();
    const sortedWeekdays = [...weekdays].sort((a, b) => a - b);
    
    for (const weekday of sortedWeekdays) {
      if (weekday > currentWeekday) {
        const daysToAdd = weekday - currentWeekday;
        const nextDate = new Date(fromDate.getTime());
        nextDate.setDate(nextDate.getDate() + daysToAdd);
        nextDate.setHours(baseDate.getHours(), baseDate.getMinutes(), 0, 0);
        return this.fromTimestamp(nextDate.getTime());
      }
    }

    // 如果没有找到本周的，找下周的第一个
    const daysToAdd = (7 * interval) + sortedWeekdays[0] - currentWeekday;
    const nextDate = new Date(fromDate.getTime());
    nextDate.setDate(nextDate.getDate() + daysToAdd);
    nextDate.setHours(baseDate.getHours(), baseDate.getMinutes(), 0, 0);
    return this.fromTimestamp(nextDate.getTime());
  }

  private static getNextMonthly(
    baseDate: Date, 
    fromDate: Date, 
    interval: number, 
    config?: any
  ): DateTime {
    const nextDate = new Date(Math.max(baseDate.getTime(), fromDate.getTime()));
    nextDate.setMonth(nextDate.getMonth() + interval);
    
    // 处理月末日期 (如31号在2月不存在)
    if (nextDate.getDate() !== baseDate.getDate()) {
      nextDate.setDate(0); // 设置为上个月的最后一天
    }
    
    return this.fromTimestamp(nextDate.getTime());
  }

}
```

</details>

## 3. 生成实例

<details>
<summary>任务实例的服务</summary>

```ts
import { v4 as uuidv4 } from 'uuid';
import type { TaskTemplate, ITaskInstance } from '../types/task';
import type { DateTime } from '../types/timeStructure';
import { TimeUtils } from '../utils/timeUtils';
import { useTaskStore } from '../stores/taskStore';

export class TaskInstanceService {
  private taskStore = useTaskStore();

  /**
   * 根据任务模板生成任务实例
   */
  generateInstancesFromTemplate(template: TaskTemplate, maxInstances: number = 100): ITaskInstance[] {
    const instances: ITaskInstance[] = [];
    const { timeConfig } = template;
    
    if (timeConfig.recurrence.type === 'none') {
      return [this.createTaskInstance(template, timeConfig.baseTime.start)];
    }

    // 重复任务生成逻辑
    let currentTime = timeConfig.baseTime.start;
    let count = 0;
    const now = TimeUtils.now();

    while (count < maxInstances) {
      const nextTime = TimeUtils.getNextOccurrence(timeConfig, currentTime);
      
      if (!nextTime) break;

      if (this.shouldStopGeneration(timeConfig, nextTime, count)) {
        break;
      }

      if (nextTime.timestamp >= now.timestamp) {
        instances.push(this.createTaskInstance(template, nextTime));
      }

      currentTime = nextTime;
      count++;

      if (count > 1000) {
        console.warn('任务实例生成达到最大限制');
        break;
      }
    }

    return instances;
  }

  /**
   * 生成指定时间范围内的任务实例
   */
  generateInstancesInRange(
    template: TaskTemplate, 
    startDate: DateTime, 
    endDate: DateTime
  ): ITaskInstance[] {
    const instances: ITaskInstance[] = [];
    const { timeConfig } = template;

    if (timeConfig.recurrence.type === 'none') {
      if (TimeUtils.isInRange(timeConfig.baseTime.start, startDate, endDate)) {
        instances.push(this.createTaskInstance(template, timeConfig.baseTime.start));
      }
      return instances;
    }

    let currentTime = timeConfig.baseTime.start;
    if (currentTime.timestamp < startDate.timestamp) {
      currentTime = startDate;
    }

    let count = 0;
    const maxInstances = 1000;

    while (currentTime.timestamp <= endDate.timestamp && count < maxInstances) {
      const nextTime = TimeUtils.getNextOccurrence(timeConfig, currentTime);
      
      if (!nextTime || nextTime.timestamp > endDate.timestamp) break;

      if (this.shouldStopGeneration(timeConfig, nextTime, count)) {
        break;
      }

      instances.push(this.createTaskInstance(template, nextTime));
      currentTime = nextTime;
      count++;
    }

    return instances;
  }

  /**
   * 创建单个任务实例
   */
  private createTaskInstance(template: TaskTemplate, scheduledTime: DateTime): ITaskInstance {
    return {
      id: uuidv4(),
      templateId: template.id,
      title: template.title,
      description: template.description,
      scheduledTime,
      keyResultLinks: template.keyResultLinks ? [...template.keyResultLinks] : undefined,
      priority: template.priority || 3,
      status: 'pending',
      reminderStatus: {
        triggeredAlerts: [],
        snoozeCount: 0,
        nextReminderTime: template.timeConfig.reminder.enabled ? 
          TimeUtils.calculateReminderTimes(scheduledTime, template.timeConfig.reminder)[0] : undefined
      }
    };
  }

  /**
   * 批量添加任务实例到存储
   */
  async addInstancesToStore(instances: ITaskInstance[]): Promise<void> {
    instances.forEach(instance => {
      this.taskStore.taskInstances.push(instance);
    });
    await this.taskStore.saveTaskInstances();
  }

  /**
   * 为模板生成并保存未来的任务实例
   */
  async generateAndSaveInstances(template: TaskTemplate, daysAhead: number = 30): Promise<ITaskInstance[]> {
    const now = TimeUtils.now();
    const endDate = TimeUtils.addDays(now, daysAhead);
    
    const instances = this.generateInstancesInRange(template, now, endDate);
    await this.addInstancesToStore(instances);
    
    return instances;
  }

  /**
   * 检查是否应该停止生成
   */
  private shouldStopGeneration(timeConfig: any, nextTime: DateTime, count: number): boolean {
    if (timeConfig.recurrence.endCondition.type === 'date' && 
        timeConfig.recurrence.endCondition.endDate &&
        nextTime.timestamp > timeConfig.recurrence.endCondition.endDate.timestamp) {
      return true;
    }

    if (timeConfig.recurrence.endCondition.type === 'count' &&
        timeConfig.recurrence.endCondition.count &&
        count >= timeConfig.recurrence.endCondition.count) {
      return true;
    }

    return false;
  }
}
```

</details>

## 4. 初始化任务实例的提醒服务

1. 当任务新添加时，创建实例的同时，如果开启提醒，调用任务队列服务将实例添加到队列中
2. 重新启动应用时，需要重新将开启提醒的任务实例添加到队列中  

## 5. 制作页面组件

```结构
TaskManagementView.vue 页面框架，切换任务管理 和 实例
  TaskTemplateManagement.vue 任务模板的管理页面  
    TaskTemplateCard.vue 展示单个任务模板
    TemplateSelectionDialog.vue 任务模板的模板选择
    TaskTemplateDialog.vue 创建任务模板  
      TaskTemplateForm.vue 任务模板编辑框
  TaskInstanceManagement.vue 任务实例的管理页面  
```