# udbx4x

`udbx4x` 是围绕 UDBX（Universal Spatial Database Extension）开放格式建设的开源 GIS SDK 与工具生态工作区。

UDBX 是 SuperMap 使用的一种基于 SQLite 的空间数据库扩展格式。本项目短期目标是提供成熟、稳定、高质量的多语言开源 SDK；中长期目标是在 SDK 之上建设完整的开源 UDBX 工具生态。

## 项目目标

- 提供可用于生产环境的 UDBX 读写、校验、检查和分析 SDK。
- 通过统一规范保持 Java、TypeScript、Go 实现的一致性。
- 以 UDBX 白皮书作为最高格式依据，以真实 `.udbx` 样本作为兼容性证据。
- 工具层必须建立在 SDK 之上，不重复实现底层二进制解析逻辑。

## 子项目

| 项目 | 作用 | 状态 |
|---|---|---|
| `udbx4spec` | 跨语言规范、类型模型、合规夹具、语言映射 | 最高优先级 |
| `udbx4j` | Java SDK，发布到 Maven Central | 核心 SDK |
| `udbx4ts` | TypeScript SDK，支持浏览器、Node.js、Electron，发布到 npm | 核心 SDK |
| `udbx4go` | Go SDK，通过 Go module tag 发布到 pkg.go.dev | 核心 SDK，已接入最小合规闭环 |
| `udbx-copilot` | 基于 SDK 的本地优先 UDBX 检查和问答 CLI | 工具生态 |

## 目录结构

```text
.
├── AGENTS.md
├── README.md
├── ROADMAP.md
├── GOVERNANCE.md
├── CONTRIBUTING.md
├── RELEASE.md
├── SECURITY.md
├── data
├── docs
├── udbx4spec
├── udbx4j
├── udbx4ts
├── udbx4go
└── udbx-copilot
```

## 核心概念

- UDBX 文件本质上是 SQLite 数据库文件。
- 系统表包括 `SmRegister`、`SmFieldInfo`、`geometry_columns`、`SmDataSourceInfo`。
- 空间几何主要使用 GAIA Geometry 和 SuperMap CAD 二进制格式。
- 跨语言几何交换使用 GeoJSON-like 模型。
- `DatasetKind` 和 `FieldType` 必须在所有 SDK 中保持一致。

## 快速入口

按语言查看 SDK 安装和使用示例：

- Java：[udbx4j/README.md](./udbx4j/README.md)
- TypeScript：[udbx4ts/README.md](./udbx4ts/README.md)
- Go：[udbx4go/README.md](./udbx4go/README.md)
- 规范：[udbx4spec/README.md](./udbx4spec/README.md)

## 公开 API 最小稳定面

三端 SDK 的公开 API 以 [`udbx4spec/docs/08-api-stable-surface.md`](./udbx4spec/docs/08-api-stable-surface.md) 为准。语言实现可以采用各自生态惯用写法，但以下语义必须一致：

| 语义 | Java | TypeScript | Go |
|---|---|---|---|
| 打开数据源 | `UdbxDataSource.open` | `UdbxDataSource.open` / 运行时工厂 | `Open` |
| 创建数据源 | `UdbxDataSource.create` | `UdbxDataSource.create` / 运行时工厂 | `Create` |
| 列出数据集 | `listDatasets` | `listDatasets` | `ListDatasets` |
| 按名称获取数据集 | `getDataset` | `getDataset` | `GetDataset` |
| 按 ID 获取对象 | `getById` | `getById` | `GetByID` |
| 列表读取 | `list` | `list` | `List` |
| 计数 | `count` | `count` | `Count` |
| 批量插入 | `insertMany` | `insertMany` | `InsertMany` |
| 更新 | `update` | `update` | `Update` |
| 删除 | `delete` | `delete` | `Delete` |

稳定语义：

- `getById` / `GetByID` 不存在时必须进入 not found 错误分支，不返回 `null` / `nil` 成功值。
- `list` / `List` 默认按 `SmID` 升序返回；`ids` 过滤不改变升序排序。
- `count` / `Count` 读取物理表真实行数，不以 `SmRegister.SmObjectCount` 缓存值为准。
- `update` / `delete` 目标对象不存在时必须返回 not found；未知字段必须返回 not found 或约束类错误，不得静默忽略。

## 开发入口

开始开发前先阅读：

- [AGENTS.md](./AGENTS.md)：项目规则、架构、文档约定。
- [CONTRIBUTING.md](./CONTRIBUTING.md)：贡献流程。
- [RELEASE.md](./RELEASE.md)：发布门禁和发布流程。
- [ROADMAP.md](./ROADMAP.md)：路线图和里程碑。

常用本地检查：

```bash
cd udbx4j && make test
cd ../udbx4ts && npm run typecheck && npm test && npm run build
cd ../udbx4go && make test
```

## 规范优先

任何公开 API、数据模型、二进制格式、数据集类型、字段类型或跨语言行为变更，都必须先进入 `udbx4spec`。

`udbx4spec` 当前包含 T0/T1/T2/T3 合规资产，首批 T3 `stable` source-derived 夹具来自已确认可公开的 `SampleData.udbx`。

当白皮书、样本文件、规范和 SDK 行为冲突时，先记录冲突和解释，再修改代码。

## 许可证

各子项目的包元数据和许可证状态以对应子项目为准。本工作区目标是建设开放、可协作、可复用的 UDBX SDK 与工具生态。
