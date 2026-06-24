# 能力矩阵

本文档记录 `udbx4spec`、`udbx4j`、`udbx4ts`、`udbx4go` 和主要工具的能力状态。

状态含义：

- ✅ 已实现或已定义。
- ⚠️ 部分实现、存在已知缺口或需要进一步验证。
- ❌ 尚未实现。
- N/A 不适用。

## 总体状态

| 能力 | `udbx4spec` | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---|---|---|---|---|
| 跨语言命名规范 | ✅ | ✅ | ✅ | ✅ | 以 `udbx4spec/docs/01-naming-conventions.md` 为准 |
| GeoJSON-like 几何模型 | ✅ | ✅ | ✅ | ✅ | Java/TS/Go 可使用语言内原生适配 |
| `DatasetKind` | ✅ | ✅ | ✅ | ✅ | `tabular`、2D/3D 矢量、`text`、`cad` 的最小规范值已三端对齐 |
| `FieldType` 14 种 | ✅ | ✅ | ✅ | ✅ | 需持续同步 |
| 错误分类 | ✅ | ✅ | ✅ | ✅ | Go 报告显示错误体系完整 |
| 合规夹具 | ✅ | N/A | N/A | N/A | 已包含 Golden GAIA Bytes、Golden GeoText Bytes、`compliance.udbx`、manifest、三端 roundtrip 夹具和首批 T3 source-derived 夹具 |
| 跨语言 roundtrip | ✅ | ✅ | ✅ | ✅ | Java、TypeScript、Go 三端已接入同一合规矩阵 |

## DatasetKind 支持

| DatasetKind | 值 | `udbx4spec` | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---:|---|---|---|---|---|
| `tabular` | 0 | ✅ | ✅ | ✅ | ✅ | 纯属性表 |
| `point` | 1 | ✅ | ✅ | ✅ | ✅ | 2D 点 |
| `line` | 3 | ✅ | ✅ | ✅ | ✅ | 2D 线 |
| `region` | 5 | ✅ | ✅ | ✅ | ✅ | 2D 面 |
| `text` | 7 | ✅ | ✅ | ✅ | ✅ | 三端已实现最小 GeoText 读写、`SmIndexKey` 写出与 `test_text` roundtrip；`SampleData.udbx` / `County_T` 异常字节已有 T3 夹具和三端真实样本验证；后续继续扩展真实样本文本样式兼容范围 |
| `pointZ` | 101 | ✅ | ✅ | ✅ | ✅ | 3D 点；`SampleData.udbx` 中 `SmSRID=0` 与 GAIA 3D header 共存已由 T3 metadata-json 和三端真实样本验证覆盖 |
| `lineZ` | 103 | ✅ | ✅ | ✅ | ✅ | 3D 线；`SampleData.udbx` 中 `SmSRID=0` 与 GAIA 3D header 共存已由 T3 metadata-json 和三端真实样本验证覆盖 |
| `regionZ` | 105 | ✅ | ✅ | ✅ | ✅ | 3D 面；`SampleData.udbx` 中 `SmSRID=0` 与 GAIA 3D header 共存已由 T3 metadata-json 和三端真实样本验证覆盖 |
| `cad` | 149 | ✅ | ✅ | ✅ | ✅ | 当前合规基线覆盖最小 GeoHeader：`GeoPoint`、`GeoLine`、`GeoRegion` |
| `network` | 4 | ⚠️ 已归档 | ❌ | ❌ | ❌ | 白皮书定义为二维网络数据集；当前仅作为真实样本观察证据，不属于最小合规基线 |
| `network3D` | 205 | ⚠️ 已归档 | ❌ | ❌ | ❌ | 白皮书定义为三维网络数据集；需与二维网络一并设计拓扑模型 |
| `model` | 203 | ⚠️ 已归档 | ❌ | ❌ | ❌ | 白皮书定义为三维模型数据集；需先补 `GeoModel3D` 二进制规范 |
| 模型资源表 | 204 | ⚠️ 已归档 | ❌ | ❌ | ❌ | 表 1 未列为顶层数据集类型；`SampleData.udbx` 中以 `modeldt_Texture` 出现，暂按模型子表证据处理 |

## FieldType 支持

| FieldType | 值 | SQLite 类型 | `udbx4j` | `udbx4ts` | `udbx4go` |
|---|---:|---|---|---|---|
| `boolean` | 1 | INTEGER | ✅ | ✅ | ✅ |
| `byte` | 2 | INTEGER | ✅ | ✅ | ✅ |
| `int16` | 3 | INTEGER | ✅ | ✅ | ✅ |
| `int32` | 4 | INTEGER | ✅ | ✅ | ✅ |
| `int64` | 5 | INTEGER | ✅ | ✅ | ✅ |
| `single` | 6 | REAL | ✅ | ✅ | ✅ |
| `double` | 7 | REAL | ✅ | ✅ | ✅ |
| `date` | 8 | TEXT | ✅ | ✅ | ✅ |
| `binary` | 9 | BLOB | ✅ | ✅ | ✅ |
| `geometry` | 10 | BLOB | ✅ | ✅ | ✅ |
| `char` | 11 | TEXT | ✅ | ✅ | ✅ |
| `time` | 16 | TEXT | ✅ | ✅ | ✅ |
| `ntext` | 127 | TEXT | ✅ | ✅ | ✅ |
| `text` | 128 | TEXT | ✅ | ✅ | ✅ |

## API 能力

| 能力 | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---|---|---|---|
| 打开现有 UDBX | ✅ | ✅ | ✅ | `open` / `Open` |
| 创建 UDBX | ✅ | ✅ | ✅ | `create` / `Create` |
| 列出数据集 | ✅ | ✅ | ✅ | `listDatasets` / `ListDatasets` |
| 获取数据集 | ✅ | ✅ | ✅ | `getDataset` / `GetDataset` |
| 分页查询 | ✅ | ✅ | ✅ | 通过 QueryOptions 或等价结构 |
| 流式读取 | ✅ | ✅ | ⚠️ | Go 可考虑 iterator/channel 风格 |
| `count()` | ✅ | ✅ | ✅ | Go 由 BaseDataset 提供 |
| `insert()` | ✅ | ✅ | ✅ | 2D/3D 矢量、tabular、Text / GeoText 最小基线和 CAD 最小 GeoHeader 均已接入 |
| `insertMany()` | ✅ | ✅ | ✅ | 三端已覆盖批量写入；批量写必须保持事务语义 |
| `update()` | ✅ | ✅ | ✅ | 三端已支持最小合规基线更新；后续继续扩展复杂 CAD/Text 对象变更覆盖 |
| `delete()` | ✅ | ✅ | ✅ | 三端已支持最小合规基线删除，并同步对象计数 |

## 运行时能力

| 能力 | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---|---|---|---|
| JVM 服务端 | ✅ | N/A | N/A | Java SDK |
| 浏览器 | N/A | ✅ | N/A | SQLite WASM + Worker |
| OPFS | N/A | ✅ | N/A | 浏览器持久化主路线 |
| 无 OPFS fallback | N/A | ✅ | N/A | 需持续测试 |
| Electron | N/A | ✅ | N/A | `better-sqlite3` |
| Node.js | N/A | ✅ | N/A | 依赖运行时入口 |
| Go CLI | N/A | N/A | ✅ | Go SDK |
| 桌面 viewer 后端 | N/A | N/A | ✅ | Wails viewer |

## 工具生态能力

| 工具 | 依赖 SDK | 状态 | 说明 |
|---|---|---|---|
| `udbx-copilot` | `udbx4ts` | ⚠️ | 本地 inspect/ask/explain，仍需 fallback 和缓存策略 |
| `udbx4go-viewer` | `udbx4go` | ⚠️ | Wails + React 实现存在，仍需功能稳定化和真实样本验证 |
| validator | 待定 | ❌ | 应基于 SDK 和 `udbx4spec` 合规资产 |
| converter | 待定 | ❌ | 应基于 SDK，不复制解析逻辑 |

## 当前关键缺口

1. Text / GeoText 三端最小合规闭环已完成，下一阶段重点是扩大真实 SuperMap 主流 UDBX 样本覆盖。
2. `udbx4spec` 合规夹具分层和 manifest 校验已标准化，首批 T3 `stable` source-derived 夹具已接入；仍需按 T3 准入规则扩展更多真实样本、异常样本与复杂文本/CAD 几何夹具。
3. `SmDatasetType=4/203/204/205` 已按白皮书和 `SampleData.udbx` 形成支持边界决策，见 [数据集类型支持边界决策](./dataset-type-boundary-decisions.md)。
4. `udbx4go-viewer` 需要补齐真实样本打开、分页展示和错误状态验证。
5. `udbx-copilot` 需要明确无 LLM 或远程 planner 不可用时的降级行为。
6. 发布门禁需要继续收敛为稳定、可重复、面向外部贡献者的标准流程。
