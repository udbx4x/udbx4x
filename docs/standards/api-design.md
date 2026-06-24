# API 设计规范

本文档定义 `udbx4x` 多语言 SDK 的公开 API 设计原则。

## 设计目标

API 应同时满足：

- 跨语言语义一致。
- 符合语言生态习惯。
- 易读、易测、易维护。
- 能反映 UDBX 格式本质。

## 规范来源

公开 API 必须以 `udbx4spec` 为准。

相关文档：

- `udbx4spec/docs/01-naming-conventions.md`
- `udbx4spec/docs/02-geometry-model.md`
- `udbx4spec/docs/03-dataset-taxonomy.md`
- `udbx4spec/docs/04-field-taxonomy.md`
- `udbx4spec/docs/05-error-taxonomy.md`
- `udbx4spec/docs/06-language-mapping.md`
- `udbx4spec/docs/08-api-stable-surface.md`

## 命名原则

核心概念使用统一名称：

| 概念 | 规范名 |
|---|---|
| 数据源 | `DataSource` / `UdbxDataSource` |
| 数据集类型 | `DatasetKind` |
| 字段类型 | `FieldType` |
| 数据集信息 | `DatasetInfo` |
| 字段信息 | `FieldInfo` |
| 查询选项 | `QueryOptions` |
| 要素 | `Feature` |
| 纯属性记录 | `TabularRecord` |

禁止为同一概念新增第二套长期名称。

## 数据集 API

所有数据集应尽量提供统一语义：

| 语义 | 规范方法 |
|---|---|
| 按 ID 查询 | `getById` |
| 列表查询 | `list` |
| 流式读取 | `stream` / `iterate` |
| 计数 | `count` |
| 插入 | `insert` |
| 批量插入 | `insertMany` |
| 更新 | `update` |
| 删除 | `delete` |

语言可以使用惯用大小写：

- Java：`getById`
- TypeScript：`getById`
- Go：`GetByID`

语义不得改变。

`getById` / `GetByID` 找不到对象时必须进入 not found 错误分支，不得在某端返回空值、某端抛错。`count` 必须读取物理表真实行数；`list` 默认按 `SmID` 升序返回。

## 异步与同步

TypeScript 公开 API 默认异步，因为浏览器和 Electron 运行时需要统一抽象。

Java 和 Go 可以使用同步 API，但资源释放必须明确。

禁止为了表面一致强行破坏语言生态习惯。

## 几何模型

跨语言交换使用 GeoJSON-like 模型：

- Point 使用 `Point`。
- Line 使用 `MultiLineString`。
- Region 使用 `MultiPolygon`。
- 扩展字段允许 `srid`、`hasZ`、`bbox`。

语言内部可以使用原生几何库：

- Java 可以使用 JTS。
- TypeScript 可以使用 JSTS。
- Go 可以使用自定义结构或后续适配库。

对外交换语义必须一致。

## 错误设计

错误必须能区分：

- 格式错误。
- 不存在。
- 不支持。
- 约束冲突。
- IO 错误。

错误消息应包含上下文，但不得泄露敏感路径、凭据或私有数据。

## 兼容策略

不为旧错误 API 长期保留兼容分支。

如需破坏性变更：

1. 修改 `udbx4spec`。
2. 更新 SDK。
3. 更新 README 和 changelog。
4. 在 release notes 中说明迁移方式。

## API 审查清单

新增或修改公开 API 前检查：

- 是否已在 `udbx4spec` 中定义。
- 是否已有等价 API。
- 是否符合语言生态惯例。
- 是否会引入跨语言语义差异。
- 是否需要合规夹具。
- 是否需要示例。
- 是否需要迁移说明。
