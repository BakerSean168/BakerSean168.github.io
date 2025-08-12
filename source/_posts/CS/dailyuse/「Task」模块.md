---
title: 任务模块开发过程
categories:
  - null
description: 
date: 2025-06-21 20:54:22
tags:
---


# 第一次具体实现

## 基础构建

现在 shared 文件夹下添加公共的基础文件

```
shared/
  domain/
    aggregateRoot.ts
    domain.ts
    domainEvent.ts
    entity.ts
    valueObject.ts
  events/
  infrastructure/


```

### 实体基类

- 实体是有唯一标识的对象
- 在 DDD 中，实体是具有连续性和唯一性的对象
- 即使属性发生变化，只要 ID 相同，就是同一个实体

```ts
// Entity.ts

export abstract class Entity {
  protected constructor(protected readonly _id: string) {}
  
  get id(): string {
    return this._id;
  }
  
  equals(other: Entity): boolean {
    if (!(other instanceof Entity)) {
      return false;
    }
    return this._id === other._id;
  }
  
  protected static generateId(): string {
    return Math.random().toString(36).substr(2, 9) + Date.now().toString(36);
  }
}

```

### ValueObject.ts-值对象基类

- 值对象 = 没有唯一标识，只关心值的对象
- 相等性基于值而不是标识
- 不可变性 - 一旦创建就不能修改

用于对象的值的封装

```ts
export abstract class ValueObject<T> {
  protected readonly _value: T;

  protected constructor(value: T) {
    this._value = value;
  }

  get value(): T {
    return this._value;
  }

  equals(other: ValueObject<T>): boolean {
    if (!(other instanceof ValueObject)) {
      return false;
    }
    return JSON.stringify(this._value) === JSON.stringify(other._value);
  }
}
```

### DomainEvent 领域事件

- 领域事件 = 业务中发生的重要事情
- 用于解耦不同模块之间的依赖
- 支持事件驱动架构

用来定义事件

```ts

export interface DomainEvent {
  aggregateId: string;
  eventType: string;
  occurredOn: Date;
  payload: any;
}

export abstract class BaseDomainEvent implements DomainEvent {
  public readonly occurredOn: Date;

  constructor(
    public readonly aggregateId: string,
    public readonly eventType: string,
    public readonly payload: any = {}
  ) {
    this.occurredOn = new Date();
  }
}

```

### AggregateRoot 聚合根

- 聚合根 = 聚合的入口点
- 负责维护业务不变性
- 管理领域事件

```ts

export abstract class AggregateRoot extends Entity {
  private _domainEvents: DomainEvent[] = [];

  protected constructor(id: string) {
    super(id);
  }

  get domainEvents(): ReadonlyArray<DomainEvent> {
    return [...this._domainEvents];
  }

  protected addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  clearDomainEvents(): void {
    this._domainEvents = [];
  }
}

```

## 领域层

### 具体的聚合根

- 独立的生命周期：可以独立创建、修改、删除
- 一致性边界：内部所有状态变更都通过业务方法控制
- 事务边界：一个 TaskInstance 的所有变更应该在一个事务中完成

聚合根内包含：  
- 聚合边界内的一致性规则
- 状态转换的业务规则
- 聚合内实体的协调
- 领域事件的发布

把 任务模板、任务实例、任务源模板可以作为聚合根  

任务模板中的 关联关键结果 好像可以作为实体

#### taskTemplate

<details>

<summary>taskTemplate 聚合根代码</summary>

```ts
import { AggregateRoot } from "@/shared/domain/aggregateRoot";
import { DateTime } from '@/shared/types/myDateTime';
import type { 
  TaskTimeConfig, 
  TaskReminderConfig,
  KeyResultLink,
  ITaskTemplate,
  CreateTaskTemplateOptions
} from "@/modules/Task/domain/types/task";
import { TimeUtils } from "../../../../shared/utils/myDateTimeUtils";

export class TaskTemplate extends AggregateRoot implements ITaskTemplate {
  private _title: string;
  private _description?: string;
  private _timeConfig: TaskTimeConfig;
  private _reminderConfig: TaskReminderConfig;
  private _schedulingPolicy: {
    allowReschedule: boolean;
    maxDelayDays: number;
    skipWeekends: boolean;
    skipHolidays: boolean;
    workingHoursOnly: boolean;
  };
  private _metadata: {
    category: string;
    tags: string[];
    estimatedDuration?: number;
    priority?: 1 | 2 | 3 | 4 | 5;
    difficulty?: 1 | 2 | 3 | 4 | 5;
    location?: string;
  };
  private _lifecycle: {
    status: "draft" | "active" | "paused" | "archived";
    createdAt: DateTime;
    updatedAt: DateTime;
    activatedAt?: DateTime;
    pausedAt?: DateTime;
  };
  private _analytics: {
    totalInstances: number;
    completedInstances: number;
    averageCompletionTime?: number;
    successRate: number;
    lastInstanceDate?: DateTime;
  };
  private _keyResultLinks?: KeyResultLink[];
  private _version: number;

  constructor(
    id: string,
    title: string,
    timeConfig: TaskTimeConfig,
    reminderConfig: TaskReminderConfig,
    options?: CreateTaskTemplateOptions
  ) {
    super(id);
    const now = TimeUtils.now();

    this._title = title;
    this._description = options?.description;
    this._timeConfig = timeConfig;
    this._reminderConfig = reminderConfig;

    this._schedulingPolicy = {
      allowReschedule: true,
      maxDelayDays: 7,
      skipWeekends: false,
      skipHolidays: false,
      workingHoursOnly: false,
      ...options?.schedulingPolicy,
    };

    this._metadata = {
      category: options?.category || "general",
      tags: options?.tags || [],
      priority: options?.priority || 3,
      difficulty: options?.difficulty || 3,
      estimatedDuration: options?.estimatedDuration,
      location: options?.location,
    };

    this._lifecycle = {
      status: "draft",
      createdAt: now,
      updatedAt: now,
    };

    this._analytics = {
      totalInstances: 0,
      completedInstances: 0,
      successRate: 0,
    };

    this._keyResultLinks = options?.keyResultLinks;
    this._version = 1;
  }

  // Getters
  get title(): string {
    return this._title;
  }

  get description(): string | undefined {
    return this._description;
  }

  get timeConfig(): TaskTimeConfig {
    return this._timeConfig;
  }

  get reminderConfig(): TaskReminderConfig {
    return this._reminderConfig;
  }

  get schedulingPolicy() {
    return this._schedulingPolicy;
  }

  get metadata() {
    return this._metadata;
  }

  get lifecycle() {
    return this._lifecycle;
  }

  get analytics() {
    return this._analytics;
  }

  get keyResultLinks(): KeyResultLink[] | undefined {
    return this._keyResultLinks;
  }

  get version(): number {
    return this._version;
  }

  // Methods
  updateTitle(title: string): void {
    this._title = title;
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  updateDescription(description?: string): void {
    this._description = description;
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  updateTimeConfig(timeConfig: TaskTimeConfig): void {
    this._timeConfig = timeConfig;
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  updateReminderConfig(reminderConfig: TaskReminderConfig): void {
    this._reminderConfig = reminderConfig;
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  updateSchedulingPolicy(policy: Partial<typeof this._schedulingPolicy>): void {
    this._schedulingPolicy = { ...this._schedulingPolicy, ...policy };
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  updateMetadata(metadata: Partial<typeof this._metadata>): void {
    this._metadata = { ...this._metadata, ...metadata };
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  setPriority(priority?: 1 | 2 | 3 | 4 | 5): void {
    this._metadata.priority = priority;
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  addKeyResultLink(link: KeyResultLink): void {
    if (!this._keyResultLinks) {
      this._keyResultLinks = [];
    }
    this._keyResultLinks.push(link);
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  removeKeyResultLink(goalId: string, keyResultId: string): void {
    if (this._keyResultLinks) {
      this._keyResultLinks = this._keyResultLinks.filter(
        (link) => !(link.goalId === goalId && link.keyResultId === keyResultId)
      );
      this._lifecycle.updatedAt = TimeUtils.now();
    }
  }

  activate(): void {
    this._lifecycle.status = "active";
    this._lifecycle.activatedAt = TimeUtils.now();
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  pause(): void {
    this._lifecycle.status = "paused";
    this._lifecycle.pausedAt = TimeUtils.now();
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  archive(): void {
    this._lifecycle.status = "archived";
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  incrementAnalytics(completed: boolean, completionTime?: number): void {
    this._analytics.totalInstances++;
    if (completed) {
      this._analytics.completedInstances++;
      if (completionTime) {
        const currentAvg = this._analytics.averageCompletionTime || 0;
        const count = this._analytics.completedInstances;
        this._analytics.averageCompletionTime =
          (currentAvg * (count - 1) + completionTime) / count;
      }
    }
    this._analytics.successRate =
      this._analytics.totalInstances > 0
        ? this._analytics.completedInstances / this._analytics.totalInstances
        : 0;
    this._analytics.lastInstanceDate = TimeUtils.now();
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  addTag(tag: string): void {
    if (!this._metadata.tags.includes(tag)) {
      this._metadata.tags.push(tag);
      this._lifecycle.updatedAt = TimeUtils.now();
    }
  }

  removeTag(tag: string): void {
    this._metadata.tags = this._metadata.tags.filter((t) => t !== tag);
    this._lifecycle.updatedAt = TimeUtils.now();
  }

  isActive(): boolean {
    return this._lifecycle.status === "active";
  }

  isDraft(): boolean {
    return this._lifecycle.status === "draft";
  }

  isPaused(): boolean {
    return this._lifecycle.status === "paused";
  }

  isArchived(): boolean {
    return this._lifecycle.status === "archived";
  }

  /**
   * 从完整数据创建 TaskTemplate 实例（用于反序列化）
   * 保留所有原始状态信息
   */
  static fromCompleteData(data: any): TaskTemplate {
    // 创建基础实例
    const instance = new TaskTemplate(
      data.id || data._id,
      data.title || data._title,
      data.timeConfig || data._timeConfig,
      data.reminderConfig || data._reminderConfig,
      {
        description: data.description || data._description,
        keyResultLinks: data.keyResultLinks || data._keyResultLinks,
        category: data.metadata?.category || data._metadata?.category,
        tags: data.metadata?.tags || data._metadata?.tags,
        priority: data.metadata?.priority || data._metadata?.priority,
        difficulty: data.metadata?.difficulty || data._metadata?.difficulty,
        estimatedDuration: data.metadata?.estimatedDuration || data._metadata?.estimatedDuration,
        location: data.metadata?.location || data._metadata?.location,
        schedulingPolicy: data.schedulingPolicy || data._schedulingPolicy,
      }
    );

    // 恢复完整的生命周期状态
    if (data.lifecycle || data._lifecycle) {
      const lifecycle = data.lifecycle || data._lifecycle;
      instance._lifecycle = {
        status: lifecycle.status || "draft",
        createdAt: lifecycle.createdAt || instance._lifecycle.createdAt,
        updatedAt: lifecycle.updatedAt || instance._lifecycle.updatedAt,
        activatedAt: lifecycle.activatedAt || undefined,
        pausedAt: lifecycle.pausedAt || undefined,
      };
    }

    // 恢复分析数据
    if (data.analytics || data._analytics) {
      const analytics = data.analytics || data._analytics;
      instance._analytics = {
        totalInstances: analytics.totalInstances || 0,
        completedInstances: analytics.completedInstances || 0,
        averageCompletionTime: analytics.averageCompletionTime,
        successRate: analytics.successRate || 0,
        lastInstanceDate: analytics.lastInstanceDate || undefined,
      };
    }

    // 恢复版本号
    if (data.version !== undefined || data._version !== undefined) {
      instance._version = data.version || data._version || 1;
    }

    // 恢复完整的元数据（如果有额外信息）
    if (data.metadata || data._metadata) {
      const metadata = data.metadata || data._metadata;
      instance._metadata = {
        ...instance._metadata,
        ...metadata,
      };
    }

    // 恢复调度策略
    if (data.schedulingPolicy || data._schedulingPolicy) {
      const policy = data.schedulingPolicy || data._schedulingPolicy;
      instance._schedulingPolicy = {
        ...instance._schedulingPolicy,
        ...policy,
      };
    }

    return instance;
  }


  isTaskTemplate(): this is TaskTemplate {
    return this instanceof TaskTemplate;
  }
  /**
   * 克隆实例（用于创建副本）
   */
  clone(): TaskTemplate {
    return TaskTemplate.fromCompleteData(this.toJSON());
  }

  /**
   * 导出完整数据（用于序列化）
   */
  toJSON(): any {
    return {
      id: this.id,
      title: this._title,
      description: this._description,
      timeConfig: this._timeConfig,
      reminderConfig: this._reminderConfig,
      schedulingPolicy: this._schedulingPolicy,
      metadata: this._metadata,
      lifecycle: this._lifecycle,
      analytics: this._analytics,
      keyResultLinks: this._keyResultLinks,
      version: this._version,
    };
  }
}
```

</details>

### 领域层内的服务

我采用了聚合根服务 + 领域层服务的方式：  
- 把一些单个聚合根相关的复杂计算逻辑放在聚合根服务中
- 再把跨聚合的业务逻辑放在单个领域层服务文件中  

以创建任务模板为例：  
创建任务模板的业务流程包含验证、保存、生成实例、将实例添加到提醒队列中（如果开启了reminder）  
在领域层中组合单独的业务  
```ts
 /**
   * 创建任务模板
   * @param {TaskTemplate} taskTemplate - 任务模板实体
   * @param {ITaskTemplateRepository} taskTemplateRepository - 任务模板仓储
   * @param {ITaskInstanceRepository} taskInstanceRepository - 任务实例仓储
   * @returns {Promise<TResponse<TaskTemplate>>} 创建响应
   */
  async createTaskTemplate(
    taskTemplate: TaskTemplate,
    taskTemplateRepository: ITaskTemplateRepository,
    taskInstanceRepository: ITaskInstanceRepository
  ): Promise<TResponse<TaskTemplate>> {
    const validation = TaskTemplateValidator.validate(taskTemplate);
    if (!validation.isValid) {
      return {
        success: false,
        message: `任务模板验证失败: ${validation.errors.join(", ")}`,
      };
    }

    taskTemplate.activate();
    const response = await taskTemplateRepository.save(taskTemplate);
    
    if (!response.success) {
      return {
        success: false,
        message: `任务模板保存失败: ${response.message}`,
      };
    }

    const initialInstances = taskInstanceService.generateInstancesFromTemplate(taskTemplate);

    const instanceResponses = await taskInstanceRepository.saveAll(initialInstances);
    if (!instanceResponses.success) {
      return {
        success: false,
        message: `初始任务实例保存失败: ${instanceResponses.message}`,
      };
    }

    for (const taskInstance of initialInstances) {
      const reminderResponse = await taskReminderService.createTaskReminders(taskInstance);
      if (!reminderResponse.success) {
        return {
          success: false,
          message: `任务实例 ${taskInstance.id} 的提醒创建失败: ${reminderResponse.message}`,
        };
      }
    }

    return {
      success: true,
      message: `任务模板 ${taskTemplate.title} 创建成功`,
      data: taskTemplate,
    };
  }
```

### 数据库接口

给每个聚合根定义相应的仓库接口，在基础设施层中实现

```ts
import type { TaskTemplate } from '../entities/taskTemplate';

export interface ITaskTemplateRepository {
  save(template: TaskTemplate): Promise<TResponse<TaskTemplate>>;
  saveAll(templates: TaskTemplate[]): Promise<TResponse<TaskTemplate[]>>;
  findById(id: string): Promise<TResponse<TaskTemplate>>;
  findAll(): Promise<TResponse<TaskTemplate[]>>;
  delete(id: string): Promise<TResponse<boolean>>;
  update(template: TaskTemplate): Promise<TResponse<TaskTemplate>>;
  findByKeyResult(goalId: string, keyResultId: string): Promise<TResponse<TaskTemplate[]>>;
}
```

## 基础设施层

基础设施层主要是数据的持久化、查找等数据库相关功能。 

### Di 容器

解决领域层对基础设施层的依赖问题：  
- 给应用层提供抽象的仓库接口，并在 taskContainer 容器中管理；可以通过 taskcontainer 来修改使用的仓库

```ts
export class TaskApplicationService {
  constructor(
    private repository: ITaskTemplateRepository // 依赖抽象接口
  ) {}
}
```

```ts
export class TaskContainer {
  private constructor() {
    // 集中配置所有依赖的具体实现
    this.taskTemplateRepository = new TaskTemplateStoreRepository();
    this.taskInstanceRepository = new TaskInstanceStoreRepository();
    this.taskMetaTemplateRepository = new TaskMetaTemplateStoreRepository();
  }
}
```

### 仓库实现
 
使用具体基础设施实现领域层中定义的仓库接口，给 taskContainer 提供可选方案  

```ts
export class TaskTemplateStoreRepository implements ITaskTemplateRepository {
  private store = useTaskStore();

  /**
   * 保存TaskTemplate到存储系统
   * 
   * @param template - 要保存的TaskTemplate实例
   * @returns 保存操作的响应结果
   */
  async save(template: TaskTemplate): Promise<TResponse<TaskTemplate>> {
    const response = await this.store.addTaskTemplate(template);
    if (response.success) {
      await this.store.saveTaskTemplates();
    }
    return response;
  }
}
```

我这里直接使用了 store 作为具体实现数据持久化的方法，因为之前已经实现了相关的数据初始化、加载、持久化存储的方案，直接调用store里面的方法就可以实现数据持久化存储、增删查改。

### 事件管理

监听对应的事件，解耦跨领域业务  

这里我期望当任务完成时，它所关联的关键结果也会增长一定的数值，即完成了部分目标。  
在任务的 application 层 中发送相应事件，并在 目标模块的 事件管理 中监听改事件，实现相应逻辑。  

```ts
import { EventBus } from '@/shared/events/eventBus';
import type { TaskCompletedEvent, TaskUndoCompletedEvent } from '@/shared/domain/domainEvent';
import { GoalApplicationService } from '../../application/services/goalApplicationService';

export class GoalEventHandlers {
  private static goalApplicationService: GoalApplicationService | null = null;

  private static getGoalService(): GoalApplicationService {
    if (!this.goalApplicationService) {
      this.goalApplicationService = new GoalApplicationService();
    }
    return this.goalApplicationService;
  }

  static registerHandlers(): void {
    const eventBus = EventBus.getInstance();

    eventBus.subscribe<TaskCompletedEvent>('TaskCompleted', async (event) => {
      console.log('Goal模块处理任务完成事件:', event);
      
      if (event.payload.keyResultLinks?.length) {
        const goalApplicationService = GoalEventHandlers.getGoalService();
        
        for (const link of event.payload.keyResultLinks) {
          await goalApplicationService.addRecordAboutTaskInstanceComplete(
            link.goalId, 
            link.keyResultId, 
            link.incrementValue
          );
        }
      }
    });

    eventBus.subscribe<TaskUndoCompletedEvent>('TaskUndoCompleted', async (event) => {
      console.log('Goal模块处理任务撤销完成事件:', event);
      
      if (event.payload.keyResultLinks?.length) {
        const goalApplicationService = GoalEventHandlers.getGoalService();
        
        for (const link of event.payload.keyResultLinks) {
          await goalApplicationService.addRecordAboutTaskInstanceComplete(
            link.goalId, 
            link.keyResultId, 
            -link.incrementValue
          );
        }
      }
    });
  }

  // 清理方法
  static cleanup(): void {
    this.goalApplicationService = null;
  }
}
```

## 应用层

- 协调跨领域的业务（使用事件总线）
- 作为用户界面层调用方法的接口（总合领域服务）

## 用户界面层

Vue 相关代码

# copilot 相关回答

基于你的 Task 模块代码，我来详细讲解前端 DDD 模式的文件结构和职责分工：

## 1. 整体文件结构

````bash
src/
├── modules/                          # 业务模块（有界上下文）
│   ├── Task/                         # 任务模块
│   │   ├── domain/                   # 领域层
│   │   │   ├── entities/             # 实体
│   │   │   │   ├── taskTemplate.ts
│   │   │   │   ├── taskInstance.ts
│   │   │   │   └── taskMetaTemplate.ts
│   │   │   ├── valueObjects/         # 值对象
│   │   │   │   ├── SchedulingPolicy.ts
│   │   │   │   ├── TaskMetadata.ts
│   │   │   │   └── TaskLifecycle.ts
│   │   │   ├── services/             # 领域服务
│   │   │   │   └── TaskDomainService.ts
│   │   │   ├── events/               # 领域事件
│   │   │   │   └── TaskDomainEvents.ts
│   │   │   ├── factories/            # 工厂
│   │   │   │   └── TaskAggregateFactory.ts
│   │   │   ├── repositories/         # 存储库接口
│   │   │   │   └── ITaskRepository.ts
│   │   │   └── types/                # 领域类型
│   │   │       └── task.d.ts
│   │   ├── application/              # 应用层
│   │   │   ├── services/             # 应用服务
│   │   │   │   └── TaskApplicationService.ts
│   │   │   ├── useCases/             # 用例
│   │   │   │   ├── CreateTaskFlowUseCase.ts
│   │   │   │   ├── UpdateTemplateUseCase.ts
│   │   │   │   └── ScheduleTaskUseCase.ts
│   │   │   ├── dto/                  # 数据传输对象
│   │   │   │   ├── CreateTaskRequest.ts
│   │   │   │   └── TaskResponse.ts
│   │   │   └── queries/              # 查询服务
│   │   │       └── TaskQueryService.ts
│   │   ├── infrastructure/           # 基础设施层
│   │   │   ├── repositories/         # 存储库实现
│   │   │   │   └── TaskRepository.ts
│   │   │   ├── adapters/             # 适配器
│   │   │   │   ├── ReminderServiceAdapter.ts
│   │   │   │   └── CalendarAdapter.ts
│   │   │   └── persistence/          # 持久化
│   │   │       ├── TaskStore.ts
│   │   │       └── TaskMapper.ts
│   │   ├── presentation/             # 表现层
│   │   │   ├── components/           # Vue 组件
│   │   │   │   ├── TaskTemplate/
│   │   │   │   ├── TaskInstance/
│   │   │   │   └── TaskCalendar/
│   │   │   ├── composables/          # Vue 组合式函数
│   │   │   │   ├── useTaskTemplate.ts
│   │   │   │   ├── useTaskInstance.ts
│   │   │   │   └── useTaskFlow.ts
│   │   │   ├── stores/               # Pinia 状态管理
│   │   │   │   └── taskStore.ts
│   │   │   └── views/                # 页面视图
│   │   │       ├── TaskManagement.vue
│   │   │       └── TaskCalendar.vue
│   │   └── utils/                    # 工具函数
│   │       ├── taskTimeUtils.ts
│   │       └── taskValidation.ts
│   ├── Goal/                         # 目标模块
│   └── Calendar/                     # 日历模块
├── shared/                           # 共享代码
│   ├── domain/                       # 共享领域基础
│   │   ├── aggregateRoot.ts
│   │   ├── valueObject.ts
│   │   ├── entity.ts
│   │   └── events/
│   ├── infrastructure/               # 共享基础设施
│   │   ├── http/
│   │   ├── storage/
│   │   └── validation/
│   └── utils/                        # 共享工具
│       └── myDateTimeUtils.ts
└── app/                              # 应用入口
    ├── main.ts
    ├── router/
    └── plugins/
````

## 2. 各层职责详解

### 🎯 Domain Layer (领域层)
**职责**: 包含核心业务逻辑，不依赖任何外部技术

````typescript
/**
 * 实体 (Entities)
 * - 包含标识符的业务对象
 * - 封装核心业务规则
 * - 生命周期贯穿整个应用
 */
export class TaskTemplate extends AggregateRoot {
  // 业务行为方法
  activate(): void { /* 激活逻辑 */ }
  pause(): void { /* 暂停逻辑 */ }
  
  // 业务规则验证
  canDelete(): boolean { /* 删除规则 */ }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\domain\valueObjects\TaskMetadata.ts
/**
 * 值对象 (Value Objects)
 * - 无标识符的不可变对象
 * - 描述领域中的概念
 * - 包含相关验证逻辑
 */
export class TaskMetadata extends ValueObject<TaskMetadataProps> {
  addTag(tag: string): TaskMetadata { /* 返回新实例 */ }
  private validate(): void { /* 验证逻辑 */ }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\domain\services\TaskDomainService.ts
/**
 * 领域服务 (Domain Services)
 * - 处理跨实体的复杂业务逻辑
 * - 不属于任何特定实体的业务规则
 */
export class TaskDomainService {
  createTemplateFromMetaTemplate(): TaskTemplate { /* 跨实体逻辑 */ }
  validateTemplateDependencies(): ValidationResult { /* 依赖检查 */ }
}
````

### 🚀 Application Layer (应用层)
**职责**: 协调领域对象完成用例，不包含业务逻辑

````typescript
/**
 * 用例 (Use Cases)
 * - 实现具体的用户操作场景
 * - 协调多个领域对象
 * - 处理事务边界
 */
export class CreateTaskFlowUseCase {
  constructor(
    private taskRepository: ITaskRepository,
    private taskDomainService: TaskDomainService,
    private eventPublisher: IEventPublisher
  ) {}

  async execute(request: CreateTaskFlowRequest): Promise<CreateTaskFlowResponse> {
    // 1. 输入验证
    this.validateRequest(request);
    
    // 2. 调用领域服务
    const template = this.taskDomainService.createTemplateFromMetaTemplate(
      request.metaTemplate, 
      request.options
    );
    
    // 3. 生成实例
    const instances = this.taskDomainService.generateInstancesWithBusinessRules(
      template, 
      request.instanceOptions
    );
    
    // 4. 持久化
    await this.taskRepository.saveTemplate(template);
    await this.taskRepository.saveInstances(instances);
    
    // 5. 发布事件
    this.eventPublisher.publish(new TaskTemplateCreatedEvent(template.id));
    
    return { template, instances };
  }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\application\queries\TaskQueryService.ts
/**
 * 查询服务 (Query Services)
 * - 处理只读查询操作
 * - 可以直接访问数据存储
 * - 优化查询性能
 */
export class TaskQueryService {
  async getTasksForWeek(startDate: DateTime): Promise<WeeklyTaskView> {
    // 直接查询，不通过领域模型
    return this.taskRepository.getTasksInDateRange(startDate, endDate);
  }
  
  async getTaskStatistics(): Promise<TaskStatistics> {
    return this.taskRepository.getStatistics();
  }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\application\dto\CreateTaskRequest.ts
/**
 * DTO (Data Transfer Objects)
 * - 跨层数据传输
 * - 输入验证
 * - API 契约定义
 */
export interface CreateTaskFlowRequest {
  metaTemplateId?: string;
  template: {
    title: string;
    timeConfig: TaskTimeConfig;
    reminderConfig: TaskReminderConfig;
  };
  instanceOptions?: {
    maxInstances?: number;
    dateRange?: DateRange;
  };
}
````

### 🔧 Infrastructure Layer (基础设施层)
**职责**: 提供技术实现，支撑上层业务逻辑

````typescript
/**
 * 存储库实现 (Repository Implementation)
 * - 实现领域层定义的存储库接口
 * - 处理数据持久化细节
 * - 数据映射和转换
 */
export class TaskRepository implements ITaskRepository {
  constructor(
    private storage: IStorage,
    private mapper: TaskMapper
  ) {}

  async saveTemplate(template: TaskTemplate): Promise<void> {
    const dto = this.mapper.templateToDto(template);
    await this.storage.save('task_templates', dto);
  }

  async getTemplateById(id: string): Promise<TaskTemplate | null> {
    const dto = await this.storage.findById('task_templates', id);
    return dto ? this.mapper.dtoToTemplate(dto) : null;
  }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\infrastructure\adapters\ReminderServiceAdapter.ts
/**
 * 适配器 (Adapters)
 * - 适配外部服务接口
 * - 协议转换
 * - 第三方集成
 */
export class ReminderServiceAdapter {
  constructor(private notificationService: INotificationService) {}

  async createTaskReminder(instance: TaskInstance): Promise<void> {
    const notifications = instance.reminderStatus.alerts.map(alert => ({
      id: alert.id,
      scheduledTime: alert.scheduledTime,
      message: `任务提醒: ${instance.title}`,
      type: 'task_reminder'
    }));

    await this.notificationService.scheduleNotifications(notifications);
  }
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\infrastructure\persistence\TaskStore.ts
/**
 * 数据存储 (Data Store)
 * - 内存存储实现
 * - 缓存管理
 * - 数据同步
 */
export class TaskStore {
  private templates = new Map<string, TaskTemplate>();
  private instances = new Map<string, TaskInstance>();

  addTaskTemplate(template: TaskTemplate): void {
    this.templates.set(template.id, template);
  }

  getTaskTemplateById(id: string): TaskTemplate | undefined {
    return this.templates.get(id);
  }
}
````

### 🎨 Presentation Layer (表现层)
**职责**: 用户界面和用户交互

````typescript
/**
 * 组合式函数 (Composables)
 * - 封装业务逻辑的 Vue 组合式 API
 * - 状态管理
 * - 副作用处理
 */
export function useTaskFlow() {
  const { taskTemplates, addTemplate } = useTaskStore();
  const createTaskFlowUseCase = inject<CreateTaskFlowUseCase>('createTaskFlowUseCase');

  const createTaskFlow = async (request: CreateTaskFlowRequest) => {
    try {
      loading.value = true;
      const result = await createTaskFlowUseCase.execute(request);
      
      // 更新本地状态
      addTemplate(result.template);
      
      // 显示成功消息
      showSuccess(`成功创建任务流: ${result.template.title}`);
      
      return result;
    } catch (error) {
      showError(`创建失败: ${error.message}`);
      throw error;
    } finally {
      loading.value = false;
    }
  };

  return {
    createTaskFlow,
    loading: readonly(loading)
  };
}

// filepath: d:\myPrograms\DailyUse\src\modules\Task\presentation\stores\taskStore.ts
/**
 * 状态存储 (State Store)
 * - 响应式状态管理
 * - 计算属性
 * - 操作方法
 */
export const useTaskStore = defineStore('task', () => {
  const templates = ref<TaskTemplate[]>([]);
  const instances = ref<TaskInstance[]>([]);

  // 计算属性
  const activeTemplates = computed(() => 
    templates.value.filter(t => t.lifecycle.status === 'active')
  );

  const todayTasks = computed(() =>
    instances.value.filter(instance => 
      isSameDay(instance.timeConfig.scheduledTime, new Date())
    )
  );

  // 操作方法
  const addTemplate = (template: TaskTemplate) => {
    templates.value.push(template);
  };

  const updateTemplate = (id: string, updates: Partial<TaskTemplate>) => {
    const index = templates.value.findIndex(t => t.id === id);
    if (index !== -1) {
      templates.value[index] = { ...templates.value[index], ...updates };
    }
  };

  return {
    templates: readonly(templates),
    instances: readonly(instances),
    activeTemplates,
    todayTasks,
    addTemplate,
    updateTemplate
  };
});
````

## 3. 层间依赖规则

````typescript
/**
 * 依赖方向 (从外到内):
 * Presentation → Application → Domain ← Infrastructure
 * 
 * 核心原则:
 * 1. 内层不依赖外层
 * 2. 领域层纯净，不依赖任何框架
 * 3. 基础设施层实现领域层定义的接口
 * 4. 应用层协调，不包含业务逻辑
 */

// ✅ 正确：表现层依赖应用层
import { CreateTaskFlowUseCase } from '@/modules/Task/application/useCases/CreateTaskFlowUseCase';

// ✅ 正确：应用层依赖领域层
import { TaskDomainService } from '@/modules/Task/domain/services/TaskDomainService';

// ❌ 错误：领域层不应依赖基础设施层
// import { TaskRepository } from '@/modules/Task/infrastructure/repositories/TaskRepository';

// ✅ 正确：领域层定义接口，基础设施层实现
import { ITaskRepository } from '@/modules/Task/domain/repositories/ITaskRepository';
````

## 4. 模块间通信

````typescript
/**
 * 领域事件总线 - 处理模块间通信
 */
export class DomainEventBus {
  // Task 模块发布事件
  publish(event: TaskTemplateCreatedEvent): void {
    // Goal 模块可以监听并更新目标进度
    // Calendar 模块可以监听并创建日历事件
  }
}

// 跨模块用例协调
// filepath: d:\myPrograms\DailyUse\src\modules\Task\application\useCases\CreateTaskWithGoalUseCase.ts
export class CreateTaskWithGoalUseCase {
  constructor(
    private taskDomainService: TaskDomainService,
    private goalQueryService: GoalQueryService  // 依赖其他模块的查询服务
  ) {}
}
````

这种 DDD 架构的优势：

1. **业务聚焦**: 领域层专注业务逻辑
2. **技术隔离**: 基础设施层封装技术细节  
3. **测试友好**: 各层可以独立测试
4. **易于维护**: 职责清晰，修改影响范围小
5. **可扩展性**: 新功能符合现有架构模式

# 修改过程中的其他问题

## 任务元模板生成任务模板的业务流程

{
    "id": "3e43a990-7e6b-4202-95d8-027c35e4f01f",
    "name": "空白模板",
    "description": "从零开始创建自定义任务模板",
    "category": "general",
    "defaultTimeConfig": {
        "type": "timed",
        "recurrence": {
            "type": "none",
            "interval": 1,
            "endCondition": {
                "type": "never"
            }
        }
    },
    "defaultReminderConfig": {
        "enabled": false,
        "alerts": [],
        "snooze": {
            "enabled": false,
            "interval": 5,
            "maxCount": 1
        }
    },
    "defaultMetadata": {
        "category": "general",
        "tags": [],
        "priority": 3,
        "estimatedDuration": 60
    },
    "lifecycle": {
        "createdAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 29
            },
            "time": {
                "hour": 15,
                "minute": 47
            },
            "timestamp": 1751183261938,
            "isoString": "2025-06-29T07:47:41.938Z"
        },
        "updatedAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 29
            },
            "time": {
                "hour": 15,
                "minute": 47
            },
            "timestamp": 1751183261938,
            "isoString": "2025-06-29T07:47:41.938Z"
        },
        "status": "active"
    }
}


生成的任务模板：  
{
    "id": "d8a866cf-e3bd-41f6-9c51-e8cb8677a931",
    "title": "",
    "timeConfig": {
        "type": "timed",
        "recurrence": {
            "type": "none",
            "interval": 1,
            "endCondition": {
                "type": "never"
            }
        },
        "timezone": "Asia/Shanghai"
    },
    "reminderConfig": {
        "enabled": false,
        "alerts": [],
        "snooze": {
            "enabled": false,
            "interval": 5,
            "maxCount": 1
        }
    },
    "schedulingPolicy": {
        "allowReschedule": true,
        "maxDelayDays": 7,
        "skipWeekends": false,
        "skipHolidays": false,
        "workingHoursOnly": false
    },
    "metadata": {
        "category": "general",
        "tags": [],
        "priority": 3,
        "difficulty": 3,
        "estimatedDuration": 60
    },
    "lifecycle": {
        "status": "draft",
        "createdAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 29
            },
            "time": {
                "hour": 15,
                "minute": 47
            },
            "timestamp": 1751183265432,
            "isoString": "2025-06-29T07:47:45.432Z"
        },
        "updatedAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 29
            },
            "time": {
                "hour": 15,
                "minute": 47
            },
            "timestamp": 1751183265432,
            "isoString": "2025-06-29T07:47:45.432Z"
        }
    },
    "analytics": {
        "totalInstances": 0,
        "completedInstances": 0,
        "successRate": 0
    },
    "version": 1
}

### 生成的任务模板是有时间的，为什么表单中没有正确显示出时间

查看创建的模板信息发现没有生成默认的具体时间，只生成了时间类型

{
    "id": "4704d4a7-1227-4bbc-bfce-f3c9bec95f07",
    "title": "",
    "timeConfig": {
        "type": "timed",
        "recurrence": {
            "type": "none"
        },
        "timezone": "Asia/Shanghai"
    },
    "reminderConfig": {
        "enabled": false,
        "alerts": [],
        "snooze": {
            "enabled": false,
            "interval": 5,
            "maxCount": 1
        }
    },
    "schedulingPolicy": {
        "allowReschedule": true,
        "maxDelayDays": 7,
        "skipWeekends": false,
        "skipHolidays": false,
        "workingHoursOnly": false
    },
    "metadata": {
        "category": "general",
        "tags": [],
        "priority": 3,
        "difficulty": 3,
        "estimatedDuration": 60
    },
    "lifecycle": {
        "status": "draft",
        "createdAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 30
            },
            "time": {
                "hour": 11,
                "minute": 52
            },
            "timestamp": 1751255575938,
            "isoString": "2025-06-30T03:52:55.938Z"
        },
        "updatedAt": {
            "date": {
                "year": 2025,
                "month": 6,
                "day": 30
            },
            "time": {
                "hour": 11,
                "minute": 52
            },
            "timestamp": 1751255575938,
            "isoString": "2025-06-30T03:52:55.938Z"
        }
    },
    "analytics": {
        "totalInstances": 0,
        "completedInstances": 0,
        "successRate": 0
    },
    "version": 1
}

在任务元模板创建任务模板时正确初始化时间（任务元模板的时间、生成时的时间）

### 设置时间会报错

` Cannot read properties of undefined (reading 'start')`

没有正确配置相关代码

### 修改表单没反应

没有正确配置相关代码

### 任务模板生命周期中的 DateTime 嵌套问题

原因貌似是 AI 生成的将Task Template 反序列化的逻辑有问题，修改相关代码就没问题了

{
    "status": "active",
    "createdAt": {
        "date": {
            "year": null,
            "month": null,
            "day": null
        },
        "time": {
            "hour": null,
            "minute": null
        },
        "timestamp": null,
        "isoString": {
            "date": {
                "year": null,
                "month": null,
                "day": null
            },
            "time": {
                "hour": null,
                "minute": null
            },
            "timestamp": null,
            "isoString": {
                "date": {
                    "year": null,
                    "month": null,
                    "day": null
                },
                "time": {
                    "hour": null,
                    "minute": null
                },
                "timestamp": null,
                "isoString": {
                    "date": {
                        "year": null,
                        "month": null,
                        "day": null
                    },
                    "time": {
                        "hour": null,
                        "minute": null
                    },
                    "timestamp": null,
                    "isoString": {
                        "date": {
                            "year": null,
                            "month": null,
                            "day": null
                        },
                        "time": {
                            "hour": null,
                            "minute": null
                        },
                        "timestamp": null,
                        "isoString": {
                            "date": {
                                "year": null,
                                "month": null,
                                "day": null
                            },
                            "time": {
                                "hour": null,
                                "minute": null
                            },
                            "timestamp": null,
                            "isoString": {
                                "date": {
                                    "year": null,
                                    "month": null,
                                    "day": null
                                },
                                "time": {
                                    "hour": null,
                                    "minute": null
                                },
                                "timestamp": null,
                                "isoString": {
                                    "date": {
                                        "year": null,
                                        "month": null,
                                        "day": null
                                    },
                                    "time": {
                                        "hour": null,
                                        "minute": null
                                    },
                                    "timestamp": null,
                                    "isoString": {
                                        "date": {
                                            "year": null,
                                            "month": null,
                                            "day": null
                                        },
                                        "time": {
                                            "hour": null,
                                            "minute": null
                                        },
                                        "timestamp": null,
                                        "isoString": {
                                            "date": {
                                                "year": null,
                                                "month": null,
                                                "day": null
                                            },
                                            "time": {
                                                "hour": null,
                                                "minute": null
                                            },
                                            "timestamp": null,
                                            "isoString": {
                                                "date": {
                                                    "year": null,
                                                    "month": null,
                                                    "day": null
                                                },
                                                "time": {
                                                    "hour": null,
                                                    "minute": null
                                                },
                                                "timestamp": null,
                                                "isoString": {
                                                    "date": {
                                                        "year": null,
                                                        "month": null,
                                                        "day": null
                                                    },
                                                    "time": {
                                                        "hour": null,
                                                        "minute": null
                                                    },
                                                    "timestamp": null,
                                                    "isoString": {
                                                        "date": {
                                                            "year": null,
                                                            "month": null,
                                                            "day": null
                                                        },
                                                        "time": {
                                                            "hour": null,
                                                            "minute": null
                                                        },
                                                        "timestamp": null,
                                                        "isoString": {
                                                            "date": {
                                                                "year": null,
                                                                "month": null,
                                                                "day": null
                                                            },
                                                            "time": {
                                                                "hour": null,
                                                                "minute": null
                                                            },
                                                            "timestamp": null,
                                                            "isoString": {
                                                                "date": {
                                                                    "year": null,
                                                                    "month": null,
                                                                    "day": null
                                                                },
                                                                "time": {
                                                                    "hour": null,
                                                                    "minute": null
                                                                },
                                                                "timestamp": null,
                                                                "isoString": {
                                                                    "date": {
                                                                        "year": null,
                                                                        "month": null,
                                                                        "day": null
                                                                    },
                                                                    "time": {
                                                                        "hour": null,
                                                                        "minute": null
                                                                    },
                                                                    "timestamp": null,
                                                                    "isoString": {
                                                                        "date": {
                                                                            "year": 2025,
                                                                            "month": 6,
                                                                            "day": 30
                                                                        },
                                                                        "time": {
                                                                            "hour": 19,
                                                                            "minute": 42
                                                                        },
                                                                        "timestamp": 1751283754689,
                                                                        "isoString": "2025-06-30T11:42:34.689Z"
                                                                    }
                                                                }
                                                            }
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    },
    "updatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 19,
            "minute": 43
        },
        "timestamp": 1751283792492,
        "isoString": "2025-06-30T11:43:12.492Z"
    },
    "activatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 19,
            "minute": 43
        },
        "timestamp": 1751283792492,
        "isoString": "2025-06-30T11:43:12.492Z"
    }
}

{
    "status": "active",
    "createdAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751284988419,
        "isoString": "2025-06-30T12:03:08.419Z"
    },
    "updatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751285009132,
        "isoString": "2025-06-30T12:03:29.132Z"
    },
    "activatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751285009132,
        "isoString": "2025-06-30T12:03:29.132Z"
    }
}

{
    "status": "active",
    "createdAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751284988419,
        "isoString": "2025-06-30T12:03:08.419Z"
    },
    "updatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751285009132,
        "isoString": "2025-06-30T12:03:29.132Z"
    },
    "activatedAt": {
        "date": {
            "year": 2025,
            "month": 6,
            "day": 30
        },
        "time": {
            "hour": 20,
            "minute": 3
        },
        "timestamp": 1751285009132,
        "isoString": "2025-06-30T12:03:29.132Z"
    }
}


### 调用 taskInstance 实体的方法没有生成事件


<details>
<summary>方法相关代码</summary>

```ts
complete(): void {
    if (this._status !== "inProgress" && this._status !== "pending") {
      throw new Error(`Cannot complete task in status: ${this._status}`);
    }

    const now = TimeUtils.now();
    this._status = "completed";
    this._completedAt = now;
    this._actualEndTime = now;
    this._lifecycle.completedAt = now;
    this._lifecycle.updatedAt = now;

    if (this._keyResultLinks?.length) {
      const event: TaskCompletedEvent = {
        eventType: "TaskCompleted",
        aggregateId: this._id,
        occurredOn: new Date(),
        payload: {
          taskId: this._id,
          keyResultLinks: this._keyResultLinks,
          completedAt: new Date(),
        },
      };
      this.addDomainEvent(event);
      console.log(`发布事件: ${event.eventType}`);
    }
    
    

    // Calculate actual duration if start time exists
    if (this._actualStartTime) {
      this._metadata.actualDuration = Math.round(
        (now.timestamp - this._actualStartTime.timestamp) / (1000 * 60)
      );
    }

    this._lifecycle.events.push({
      type: "task_completed",
      timestamp: now,
      details: {
        actualDuration: this._metadata.actualDuration,
      },
    });
  }
```

</details>

<details>
<summary>打印出来的实例对象</summary>

```ts
{
    "id": "f386a023-1620-49ba-a732-a482853f7fe1",
    "templateId": "70d64e57-49f3-4e84-88e2-c20b856f9e19",
    "title": "任务",
    "timeConfig": {
        "type": "timed",
        "scheduledTime": {
            "date": {
                "year": 2025,
                "month": 7,
                "day": 2
            },
            "time": {
                "hour": 23,
                "minute": 34
            },
            "timestamp": 1751470478412,
            "isoString": "2025-07-02T15:34:38.412Z"
        },
        "endTime": {
            "date": {
                "year": 2025,
                "month": 7,
                "day": 3
            },
            "time": {
                "hour": 0,
                "minute": 34
            },
            "timestamp": 1751474078412,
            "isoString": "2025-07-02T16:34:38.412Z"
        },
        "estimatedDuration": 60,
        "timezone": "Asia/Shanghai",
        "allowReschedule": true,
        "maxDelayDays": 7
    },
    "actualEndTime": {
        "date": {
            "year": 2025,
            "month": 7,
            "day": 2
        },
        "time": {
            "hour": 8,
            "minute": 57
        },
        "timestamp": 1751417838176,
        "isoString": "2025-07-02T00:57:18.176Z"
    },
    "priority": 3,
    "status": "completed",
    "completedAt": {
        "date": {
            "year": 2025,
            "month": 7,
            "day": 2
        },
        "time": {
            "hour": 8,
            "minute": 57
        },
        "timestamp": 1751417838176,
        "isoString": "2025-07-02T00:57:18.176Z"
    },
    "reminderStatus": {
        "enabled": true,
        "alerts": [],
        "globalSnoozeCount": 0
    },
    "lifecycle": {
        "createdAt": {
            "date": {
                "year": 2025,
                "month": 7,
                "day": 1
            },
            "time": {
                "hour": 17,
                "minute": 52
            },
            "timestamp": 1751363555439,
            "isoString": "2025-07-01T09:52:35.439Z"
        },
        "updatedAt": {
            "date": {
                "year": 2025,
                "month": 7,
                "day": 2
            },
            "time": {
                "hour": 8,
                "minute": 57
            },
            "timestamp": 1751417838176,
            "isoString": "2025-07-02T00:57:18.176Z"
        },
        "completedAt": {
            "date": {
                "year": 2025,
                "month": 7,
                "day": 2
            },
            "time": {
                "hour": 8,
                "minute": 57
            },
            "timestamp": 1751417838176,
            "isoString": "2025-07-02T00:57:18.176Z"
        },
        "events": [
            {
                "type": "task_completed",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 0,
                        "minute": 4
                    },
                    "timestamp": 1751385872010,
                    "isoString": "2025-07-01T16:04:32.010Z"
                },
                "details": {}
            },
            {
                "type": "task_undo",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 54
                    },
                    "timestamp": 1751417677126,
                    "isoString": "2025-07-02T00:54:37.126Z"
                },
                "details": {
                    "previousStatus": "completed",
                    "newStatus": "inProgress"
                }
            },
            {
                "type": "task_completed",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 54
                    },
                    "timestamp": 1751417684675,
                    "isoString": "2025-07-02T00:54:44.675Z"
                },
                "details": {}
            },
            {
                "type": "task_undo",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 56
                    },
                    "timestamp": 1751417796277,
                    "isoString": "2025-07-02T00:56:36.277Z"
                },
                "details": {
                    "previousStatus": "completed",
                    "newStatus": "inProgress"
                }
            },
            {
                "type": "task_completed",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 56
                    },
                    "timestamp": 1751417800182,
                    "isoString": "2025-07-02T00:56:40.182Z"
                },
                "details": {}
            },
            {
                "type": "task_undo",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 57
                    },
                    "timestamp": 1751417836532,
                    "isoString": "2025-07-02T00:57:16.532Z"
                },
                "details": {
                    "previousStatus": "completed",
                    "newStatus": "inProgress"
                }
            },
            {
                "type": "task_completed",
                "timestamp": {
                    "date": {
                        "year": 2025,
                        "month": 7,
                        "day": 2
                    },
                    "time": {
                        "hour": 8,
                        "minute": 57
                    },
                    "timestamp": 1751417838176,
                    "isoString": "2025-07-02T00:57:18.176Z"
                },
                "details": {}
            }
        ]
    },
    "metadata": {
        "category": "general",
        "tags": [],
        "estimatedDuration": 60,
        "difficulty": 3
    },
    "version": 1
}
```
</details>

发现实例对象没有 关键结果链接属性。  
原来是测试时使用 更新方法 来添加的链接，但 更新方法 逻辑有问题，没有正确添加链接属性。  


<details>
<summary>相关代码</summary>
</details>
<details>
<summary>相关代码</summary>
</details>


# 个人理解

DDD 模式把代码分为四部分

- application
- domain
- infrastructure
- presentation

我认为 DDD 模式是面向对象类的  
首先是 domain 层，这里用来定义具体的实体（对象，如任务实例、任务模板）、值对象（如邮箱）、领域事件、实体服务、领域服务。  

infrastructure 层，用来实现存储、查找实体之类的方法，用来持久化数据。

application 层，用来在 presentation 中调用用领域层的方法，相当于一个统一的接口，同时也协调跨领域的业务逻辑

presentation 层，实现页面相关的代码，主要是 Vue 相关的代码，组件、composables、stores、views 等

## 服务相关代码

DDD 模式最重要的应该就是服务代码的管理，它应该分三层：  
1. 实体服务
2. 领域服务
3. 应用服务

通过服务代码的分层，通过统一的 应用服务 访问

# 第二次重构

意识到之前的前后端职责区分有一定的问题：  
将整个实体对象的定义、复杂的业务逻辑等都放在了渲染进程，应该将其移动到主进程中。  

所以第二次重构的重点就是将部分代码转移到主进程中。  

## 移动应用层、领域层、基础设施层

这三层的作用是 复杂的业务逻辑、数据持久化等相关，应该放到主进程中。  

这时候，渲染层获取数据就通过 IPC 通信来向主进程获取数据。  
所以需要考虑传输的数据的格式问题（Dto）：  
1. 数据传输要避免暴露某些不安全的属性
2. 要注意序列化时可能丢失的属性

本次选择直接复用 对应接口的格式 来作用为传输给前端的数据格式，比较简单。

## 修改基础设施层的具体仓库实现

现在改为直接操作数据库，而非渲染层的 store。  

## 渲染进程的修改

之前的业务逻辑（以添加任务模板为例）：  
1. 应用服务层传入仓库和任务模板
2. 任务模板验证
3. 调用领域服务
4. 一系列聚合根服务后进行数据持久化

现在的业务逻辑：  
1. 渲染层调用服务

### 基础设施层

### 初始化



#### 1.

❓  
我当前设计的是离线模式，直接通过主进程来进行处理，只不过 ipc 是主进程进程数据处理， store 是状态管理工具；我的存储应该就是通过 ipc 来存储。只不过我在进行相应操作时，也需要维护状态管理中的数据，正常流程因该是。所以我还需要通过 storerepo 层来管理状态吗。  

不需要抽象 IPC 仓库了，这应该是主进程需要操作的（数据持久化），渲染进程只需要调用 IPC 以及管理状态仓库，所以可以抽象 storeRepo。

#### 2.

❓  
是不是直接通过调用主进程提供的 API，直接传入 任务模板，主进程就可以实现持久化，返回数据，渲染进程的抽象仓库是不是没有存在的必要，或者用来抽象状态管理

同 1 问

#### 3

❓  
渲染进程中是不是一般都是直接使用结构化的数据，使用面向过程的修改对象方式，还是说我应该在渲染进程中也定义和主进程一样的聚合根对象和实体对象等

在渲染进程中使用一个 没有业务方法，但新增 UI 方法的对象

#### 5

❓  
IPC 通信报了 无法克隆的错误时，仍然会继续执行主进程的创建任务模板的服务


# 第三次重构

1. 使用 Date 时间类型，而非 DateTime
2. 优化 toDTO 相关传输方法，严格区分 接口 和 数据传输接口
3. 修改了 数据库 的字段
template_old-> template 是我给你的修改范例 还有新的 taskTemplateDatabaseRepository 数据库代码,我也更新了 taskInstance 的类型接口，请你帮我重构 另外的 taskInstance 和 taskMetaTemplate

createTaskTemplateByMetaTemplate 方法可以直接放在渲染进程的 metaTemplate 聚合根中


