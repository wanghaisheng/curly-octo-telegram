# 单体应用向 Monorepo 架构迁移指南 (Monorepo Migration Guide)

当项目从典型的单体（Monolithic）结构向更严谨的 Monorepo 结构演进时，本指南提供了标准化的迁移路径与最佳实践。

---

## 1. 迁移原则 (Principles)
*   **应用与库的分离**：`apps/` 存放可独立部署的程序；`packages/` 存放被依赖的逻辑库。
*   **功能归位**：服务端入口应属于具体的应用环境，核心业务逻辑应抽离为独立包。
*   **最小化路径风险**：通过代理入口保持兼容，逐步完成深度重构。

---

## 2. 迁移步骤 (Step-by-Step)

### 第一步：目录结构重构
1.  **创建包目录**：在 `/packages` 下创建标准化的子目录，如下：
    *   `ui/`：存放 React/Vue 通用组件。
    *   `core/`：核心模块 (db, payment, utils)。
    *   `api/`：站点感知的 Hono 业务 API。
    *   `auth/`：独立认证 Worker。
    *   `video-service/`：异步任务 Worker。
    *   `types/`：共享 TypeScript 类型。
    *   `config/`：共享构建配置 (ESLint, Prettier)。
2.  **规划边界**：确定哪些是独立的应用（部署单位），哪些是共享的逻辑依赖。

### 第二步：逻辑迁移 (Service Extraction)
将业务逻辑按职能分包：
1.  **提取逻辑**：将原根目录下的逻辑（如 `/server.ts`）抽离核心渲染逻辑。
2.  **移至包中**：将抽离的代码放入 `packages/api/src/` 中，通过 `export` 暴露初始化函数。
3.  **保持纯净**：包内的逻辑应尽量减少对外部硬依赖的引用，通过参数注入运行配置。

### 第三步：调整应用入口 (Application Refactoring)
1.  **代理入口**：在对应的应用下（如 `/apps/web/server.ts`）保留一个轻量级代理入口，仅负责调用 `packages/api` 导出的方法。
2.  **更新路径**：修复所有在被迁移文件中的相对导入路径（使用 `../../` 等方式精确回溯）。
3.  **配置更新**：
    *   更新 `package.json` 中的 `scripts` 执行路径（例如：从 `tsx server.ts` 更新为 `tsx apps/web/server.ts`）。
    *   确保构建工具和 IDE 路径映射正确。

### 第四步：基础设施适配
1.  **重启服务**：执行 `restart_dev_server` 清理旧缓存。
2.  **验证环境**：检查 `Vite` 中间件路径配置（`dist/client` 等静态路径）是否依然指向正确。

---

## 3. 注意事项 (Critical Directives)
*   **绝对路径与相对路径**：迁移后需要仔细排查 `import` 路径错误。
*   **依赖循环**：避免 `apps/web` 引用 `packages/api` 的同时，`packages/api` 又试图引用 `apps/web` 下的非共享组件。
*   **CI/CD 一致性**：如果在 CI 中存在自定义构建脚本，必须同步更新其入口路径。

---

## 4. 迁移检查清单 (Migration Checklist)
- [ ] 逻辑包是否已实现与运行环境（Browser/Node）的部分解耦？
- [ ] 应用入口代理脚本是否足够轻量（建议<10行）？
- [ ] `package.json` 中的 `scripts` 是否已更新为对应的应用子目录路径？
- [ ] 所有测试用例是否已指向新的文件位置？
