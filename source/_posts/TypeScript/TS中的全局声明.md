---
title: TS中的全局声明
categories:
  - null
description: TS中的全局声明
date: 2025-07-20 12:22:50
tags:
---

# TypeScript 中 `declare interface Window` 的详细解析

在 TypeScript 中，`declare interface Window` 是一种扩展浏览器全局 `window` 对象类型定义的方式，它允许开发者向 TypeScript 类型系统描述在 `window` 对象上添加的自定义属性或方法。这在集成第三方库或添加全局变量时非常有用。

## 基本概念与作用

`declare interface Window` 是 TypeScript 的声明合并(Declaration Merging)特性的一部分，它允许我们扩展已有的 `Window` 接口定义。

当你在 JavaScript 中向 `window` 对象添加属性时：
```javascript
window.myCustomProperty = "some value";
```

TypeScript 默认不知道这个新属性，会报类型错误。通过 `declare interface Window`，我们可以告诉 TypeScript 这个新属性的存在。

## 使用方法

### 1. 在 `.d.ts` 文件中声明（推荐）

创建一个全局声明文件（如 `globals.d.ts` 或 `window.d.ts`），然后添加：

```typescript
interface Window {
myCustomProperty: string;
myCustomMethod: () => void;
}
```

这种方法最简洁，因为 `.d.ts` 文件中的内容会自动进入全局作用域。

### 2. 在模块中使用 `declare global`

如果需要在模块文件（包含 `import`/`export` 的文件）中扩展 `Window`，必须使用 `declare global`：

```typescript
export {}; // 使文件成为模块

declare global {
interface Window {
myCustomProperty: string;
}
}
```

`declare global` 允许在模块内部修改全局类型。

### 3. 单文件覆盖（局部扩展）

如果只想在特定文件中覆盖 `window` 类型：

```typescript
declare const window: {
myCustomProperty: string;
} & Window;
```

这种方法只影响当前文件。

## 实际应用场景

### 1. 添加全局配置

```typescript
interface Window {
appConfig: {
apiUrl: string;
env: 'dev' | 'prod';
};
}
```

### 2. 集成第三方库

许多库（如 Google Analytics）会向 `window` 添加属性：

```typescript
interface Window {
dataLayer: any[];
gtag: (...args: any[]) => void;
}
```

### 3. 服务器渲染数据传递

从服务器向客户端传递初始数据：

```typescript
interface Window {
__INITIAL_STATE__: {
user: User;
preferences: Preferences;
};
}
```

## 类型安全与最佳实践

1. **避免使用 `any`**：尽量为自定义属性定义具体类型。

不推荐：
```typescript
interface Window {
myProp: any;
}
```

推荐：
```typescript
interface Window {
myProp: {
id: number;
name: string;
};
}
```

2. **组织声明文件**：将全局类型定义放在专门的 `.d.ts` 文件中。

3. **模块化扩展**：对于大型项目，可以按功能拆分 `Window` 扩展：
```typescript
// window-api.d.ts
interface Window {
api: {
getUsers: () => Promise<User[]>;
};
}
```

4. **兼容性检查**：确保运行时属性确实存在，类型声明只是编译时检查。

## 常见问题解决

### 1. 属性访问报错

如果遇到 `Property 'X' does not exist on type 'Window & typeof globalThis'` 错误，说明需要扩展 `Window` 接口。

### 2. 模块中的全局扩展

在模块中使用 `window.customProp` 会报错，必须使用 `declare global`。

### 3. 多个声明合并

TypeScript 会自动合并同名的 `interface` 声明，所以可以在不同文件中扩展 `Window`。

## 与其他方法的比较

| 方法 | 作用域 | 适用场景 | 缺点 |
|------|--------|----------|------|
| `declare interface Window` | 全局 | 大多数情况 | 需要创建声明文件 |
| `declare global` | 模块内全局 | 需要在模块中扩展 | 语法稍复杂 |
| `window as any` | 单文件 | 快速修复 | 失去类型安全 |
| `declare const window` | 单文件 | 局部覆盖 | 不适用于多文件 |

## 总结

`declare interface Window` 是 TypeScript 中扩展浏览器全局对象的标准方式，它提供了类型安全的同时保持了 JavaScript 的灵活性。通过合理组织类型声明，可以大大提升代码的可维护性和开发体验。

在实际项目中，推荐将全局类型定义集中放在 `.d.ts` 文件中，并为每个属性或方法提供精确的类型定义，避免过度使用 `any` 类型。