# 类型安全最佳实践 (Type Safety Guidelines)

## 1. 契约优先：tRPC & Zod 集成
系统架构的核心是 **“类型即契约”**。

*   **定义端**: 使用 Zod 定义输入输出的 Schema。
*   **消费端**: 通过 tRPC 自动推导客户端 API 的入参和出参类型，实现 0 API 文档化开发。
*   **跨平台**: 同一套 Zod Schema 必须同时用于 Web 端的 tRPC 和 RN 端的本地逻辑校验。

## 2. 结果对象模式 (Result Pattern)
为了统一处理本地错误、网络错误和业务错误，API 层必须返回：
```typescript
type Result<T, E = AppError> = 
  | { ok: true; data: T; source: 'server' | 'cache' | 'local' } 
  | { ok: false; error: E };
```
**UI 强制处理**: 表现层通过判断 ok 状态进行渲染，杜绝未捕获异常。

## 3. 零 any 原则与类型守卫
*   **禁止 any**: 外部输入（如 URL 参数、存储读取）默认为 unknown。
*   **运行时检查**: 必须通过类型谓词 (Type Predicates) 或 Zod 解析后才能进入业务逻辑。

## 4. 存储层类型契约
```typescript
interface IStorageAdapter {
  // 使用泛型 T 确保从存储获取的对象符合领域模型定义
  query<T>(collection: string, params: QueryParams): Promise<Result<T[]>>;
  
  auth: {
    getSession(): Promise<Result<Session | null>>;
  };
}
```

## 5. 样式与 Token 类型化
```typescript
// 定义全局统一的设计 Token 类型
type ColorToken = keyof typeof DesignSystem.colors;
type SpacingToken = keyof typeof DesignSystem.spacing;

// UI 组件 Props 必须引用这些 Token 类型，而非 string
```

## 6. 类型安全检查清单
*   所有的 API 动作是否都有对应的 Zod 校验？
*   结果对象是否包含了 source 字段以支持移动端离线感知？
*   是否在 API 层之外的地方引用了数据库厂商的类型？（应使用自定义的 Entity 类型）
*   所有的枚举值是否都使用了 as const 或 enum 进行类型化？
