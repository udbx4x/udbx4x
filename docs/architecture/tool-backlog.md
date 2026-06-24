# 工具生态待办

本文档记录 SDK 之上的工具生态待办事项。工具层必须依赖 SDK，不得复制 UDBX 底层解析逻辑。

## `udbx-copilot`

### 远程 planner 不可用时的降级模式

目标：当 planner 模型不可用时，提供 inspect-first 的明确降级路径。

背景：

- 当前 `ask` 流程依赖 `PlanGenerator -> plan JSON -> validatePlan() -> executor`。
- 当 API key 缺失、网络不可用或服务端错误时，用户仍应能检查客户文件。
- 降级模式不应伪装成成功问答，应明确失败原因并引导用户使用非 LLM 流程。

验收标准：

- `ask` 在缺失 API key 时返回清晰错误。
- 错误信息建议使用 `inspect <file.udbx>`。
- 远程规划失败时进程返回非零退出码。
- 本地 heuristic 模式不依赖网络。
- 文档说明 `inspect`、`list_fields`、bounded preview 等替代路径。

### 持久摘要缓存或轻量索引

目标：为频繁打开的大文件提供更稳定的重复检查体验。

背景：

- v1 可先使用会话级缓存。
- 大文件或重复打开场景可能需要持久摘要缓存。

验收标准：

- 缓存必须包含文件变更检测。
- 缓存失效规则清晰。
- 不缓存私有敏感字段值，除非用户明确允许。
- 性能收益通过真实样本验证。

## `udbx4go-viewer`

### 当前技术栈文档化

目标：以 `cmd/udbx4go-viewer` 中的 Wails v2 + React/TypeScript 实现为准，清理与当前实现冲突的历史方案文档。

状态：已完成。早期 viewer 长期方案文档已删除，当前 viewer 文档入口为 `udbx4go/cmd/udbx4go-viewer/README.md`。

验收标准：

- `udbx4go/cmd/udbx4go-viewer/README.md` 描述当前实现。
- 不再保留与当前实现冲突的长期方案文档。
- `udbx4go/AGENTS.md` 指向当前 viewer 文档。

### Viewer 功能稳定化

目标：把 viewer 从可运行原型推进到可验证工具。

验收标准：

- 能打开 `.udbx` 文件。
- 能列出数据集。
- 能分页展示记录。
- 能显示加载状态和错误。
- 能在关闭或切换文件时释放资源。
