---
title: try-catch捕获错误
categories:
  - null
description: try-catch捕获错误
date: 2025-08-19 14:44:45
tags:
---

简短结论  
- 不加 try/catch 时，错误会向上传播——在不同上下文会有不同后果：可能被事件总线/框架捕获并记录、可能导致请求失败返回 500、也可能产生未处理的 Promise 拒绝 (unhandledRejection) 导致进程不稳定或崩溃。建议在边界层（应用服务/事件处理器/Controller）做有意义的捕获与错误映射。

要点（按场景）
- 在事件处理器内被 eventBus 调用：
  - 你的 eventBus.publish 已经在调用 handler 时做了 try/catch 并记录错误（并用 Promise.allSettled）。因此 handler 抛出异常通常不会让 publish 抛出，但你会丢失业务级别的上下文日志／转换；也无法实现自定义重试/补偿逻辑。
- 在 Express 路由（Controller）内：
  - 如果异步函数抛出并且没有被捕获且没有把错误传给 next(err)，请求会变成未处理的 Promise，Express 可能不会正确返回错误，会导致请求挂起或默认 500。建议在 Controller 层捕获并转换为合适的 HTTP 响应。
- 在应用服务层（AuthenticationApplicationService）：
  - 不捕获：错误会传给调用者（Controller 或事件处理器）。这是可以接受的，只要调用方负责捕获并做正确处理。优点是不上层重复捕获；缺点是丢失局部上下文日志或无法把底层错误转换为友好业务错误。
- 事务与资源清理：
  - 若流程涉及 DB 事务或临时资源，必须确保在异常情况下做回滚/清理——这通常需要在调用链某处（应用层或更上层）捕获并做补偿。

建议实践（简短）
- 在应用服务内：只捕获你能处理或需要转换的错误（如密码校验、唯一约束），并抛出自定义的业务异常；不捕获则允许上层统一处理。
- 在 Controller 层：统一捕获并把业务异常映射为 HTTP 状态码（400/409/500）。
- 在事件处理器：仍然做必要的 try/catch 以记录上下文、实现幂等/重试或将失败入死信队列；若使用 eventBus 已捕获，也应在业务层记录与告警。
- 全局：使用 process.on('unhandledRejection') / process.on('uncaughtException') 做最后的日志/告警与优雅重启（但不要依赖其做正常业务处理）。

示例：Express async 包装器（推荐用法）
````typescript
export const asyncHandler = (fn: (req, res, next) => Promise<any>) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};
````

示例：Controller 使用（映射错误为响应）
````typescript
router.post('/register', asyncHandler(async (req, res) => {
  try {
    const account = await accountService.create(req.body);
    res.status(201).json(account);
  } catch (err) {
    if (err instanceof ValidationError) return res.status(400).json({ message: err.message });
    if (err instanceof UniqueConstraintError) return res.status(409).json({ message: err.message });
    throw err; // 交给全局错误处理器
  }
}));
````

总结一句话：可以让错误向上传播，但必须在边界处（Controller / 事件处理器 / 全局）有明确的捕获与映射策略，且对涉及事务的流程在捕获时做回滚/补偿。