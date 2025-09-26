---
title: 「DailyUse」使用的事件方法总结
categories:
  - dailyuse
description: 「DailyUse」使用的事件方法总结
date: 2025-07-30 19:38:54
tags:
---

### 1. DDD 架构中的领域事件处理系统

```ts
import type { DomainEvent } from '../domain/domainEvent';

/**
 * 领域事件处理器类型
 */
type EventHandler<T extends DomainEvent = DomainEvent> = (event: T) => Promise<void>;

/**
 * 简单的内存事件总线
 * 用于在主进程中实现模块间的事件通信
 */
export class EventBus {
  private static instance: EventBus;
  private handlers: Map<string, EventHandler[]> = new Map();

  private constructor() {}

  static getInstance(): EventBus {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus();
    }
    return EventBus.instance;
  }

  /**
   * 订阅事件
   */
  subscribe<T extends DomainEvent>(eventType: string, handler: EventHandler<T>): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler as EventHandler);
    console.log(`📝 [EventBus] 订阅事件: ${eventType}`);
  }

  /**
   * 发布事件
   */
  async publish(event: DomainEvent): Promise<void> {
    const handlers = this.handlers.get(event.eventType) || [];
    
    if (handlers.length === 0) {
      console.log(`⚠️ [EventBus] 没有找到事件处理器: ${event.eventType}`);
      return;
    }

    console.log(`📢 [EventBus] 发布事件: ${event.eventType}, 处理器数量: ${handlers.length}`);

    // 并行执行所有处理器
    const promises = handlers.map(async (handler) => {
      try {
        await handler(event);
      } catch (error) {
        console.error(`❌ [EventBus] 事件处理器执行失败 (${event.eventType}):`, error);
        // 这里可以添加重试逻辑或死信队列
      }
    });

    await Promise.allSettled(promises);
  }

  /**
   * 发布多个事件
   */
  async publishMany(events: DomainEvent[]): Promise<void> {
    for (const event of events) {
      await this.publish(event);
    }
  }

  /**
   * 取消订阅
   */
  unsubscribe(eventType: string, handler: EventHandler): void {
    const handlers = this.handlers.get(eventType);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
        console.log(`🗑️ [EventBus] 取消订阅事件: ${eventType}`);
      }
    }
  }

  /**
   * 清除所有订阅
   */
  clear(): void {
    this.handlers.clear();
    console.log(`🧹 [EventBus] 清除所有事件订阅`);
  }

  /**
   * 获取已订阅的事件类型
   */
  getSubscribedEventTypes(): string[] {
    return Array.from(this.handlers.keys());
  }
}

/**
 * 便捷的全局事件总线实例
 */
export const eventBus = EventBus.getInstance();

```

### 2. `EventEmitter`基础类 / nodeSchedule

**`EventEmitter`**

`EventEmitter` 是 Node.js（以及 Electron）内置的事件发布/订阅（发布-订阅）机制的基础类。它允许你在对象间通过事件进行通信，非常适合模块解耦和异步事件处理。  

在 Electron（和 Node.js）中，很多核心模块（如 ipcMain、ipcRenderer、BrowserWindow、net 等）都继承自 EventEmitter，可以用它来监听和触发事件。

nodeSchedule 就是基于 EventEmitter 事件的一个工具类。
