# 模块划分规范

本文档定义 `udbx4x` 的模块边界和依赖方向。

## 总体原则

依赖方向只能从规范到 SDK，再到工具：

```text
UDBX 白皮书 / 样本 -> udbx4spec -> SDK -> tools
```

禁止反向依赖：

- `udbx4spec` 不依赖任何 SDK 实现。
- SDK 不依赖工具层。
- 工具层不复制 SDK 的底层解析逻辑。

## `udbx4spec`

负责：

- 概念定义。
- API 语义。
- 数据模型。
- 枚举值。
- 错误分类。
- 参考类型。
- 合规夹具。

不负责：

- 具体语言运行时。
- 包发布。
- 工具业务流程。

## SDK

SDK 包括：

- `udbx4j`
- `udbx4ts`
- `udbx4go`

负责：

- UDBX 文件读写。
- SQLite 系统表访问。
- GAIA/CAD 编解码。
- Dataset API。
- 类型映射。
- 错误处理。

不负责：

- 交互式 UI。
- 自然语言问答。
- 批处理工具特有工作流。

## 工具层

工具包括：

- viewer。
- CLI。
- Copilot。
- validator。
- converter。

负责：

- 用户交互。
- 文件选择。
- 命令行参数。
- 查询计划展示。
- 结果格式化。
- 报告输出。

不负责：

- 直接解析 GAIA blob。
- 直接维护 `DatasetKind` 映射。
- 直接维护 `FieldType` 映射。
- 绕过 SDK 写系统表。

## `udbx4ts` 内部分层

`udbx4ts` 必须保持：

```text
src/core -> shared-runtime -> runtime-browser / runtime-electron
```

约束：

- `src/core/` 不依赖 DOM、Worker、Electron 或具体 SQLite 驱动。
- 浏览器数据库操作只在 Worker 内执行。
- Electron 驱动适配不得污染 core。

## `udbx4go` 内部分层

`udbx4go` 应保持：

```text
pkg -> public API
internal -> implementation
cmd -> applications
```

约束：

- `internal` 实现不得泄漏为公开 API。
- `cmd` 工具必须调用 SDK，不复制内部解析逻辑。

## `udbx4j` 内部分层

`udbx4j` 应保持：

```text
core -> system -> geometry -> dataset -> datasource
```

约束：

- 系统表 DAO 封装 SQLite 表访问。
- geometry 包封装二进制几何。
- dataset 包封装业务 API。
- datasource 是用户入口。

## 变更检查

新增模块前检查：

- 是否已有模块可承载该职责。
- 是否会造成双向依赖。
- 是否复制了已有 SDK 能力。
- 是否引入跨语言概念差异。
- 是否需要先更新 `udbx4spec`。

