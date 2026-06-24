# 数据集类型支持边界决策

本文档记录当前 `udbx4spec` 最小合规范围之外的 `SmDatasetType` 处理决策。它用于避免把真实样本中观察到的类型误写成三端 SDK 已支持能力。

## 决策状态

| 类型值 | 白皮书名称或样本名称 | 白皮书依据 | `SampleData.udbx` 证据 | 当前决策 |
|---:|---|---|---|---|
| `4` | `Network` | 表 1 定义为二维网络数据集；3.1.7 说明网络数据集由主表和节点子表组成 | `Jingjin_Network`，对象数 `1622`，主表字段含 `SmFNode`、`SmTNode`、`SmEdgeID`、`SmResistanceA/B` | 暂不纳入当前最小 SDK 支持；先进入 `udbx4spec` 规范扩展设计 |
| `205` | `Network3D` | 表 1 定义为三维网络数据集；3.1.7 说明三维网络主表存储三维线，子表存储三维点 | `Jingjin_NetworkZ`，对象数 `570`，主表字段含网络拓扑字段和三维关联点字段 | 暂不纳入当前最小 SDK 支持；应与 `Network` 一并设计 |
| `203` | `Model` | 表 1 定义为三维模型数据集；3.1.8 和 4.5 说明 `GeoModel3D` 与模型实体结构 | `modeldt`，对象数 `2`，字段含 `SmIndexKey`、`SmGeometry`、`MODELNAME`、经纬高属性 | 暂不纳入当前最小 SDK 支持；需先建立 `GeoModel3D` 二进制规范 |
| `204` | 模型实体/纹理关联表 | 表 1 未列为顶层数据集类型；3.1.8 说明三维模型数据集有子表，4.5.3.3 定义 `EntityTexture` | `modeldt_Texture`，对象数 `47`，字段含 `SmHashCode`、`SmGeometry` | 不作为独立 `DatasetKind` 直接纳入；暂按模型数据集子表或模型资源表证据处理 |

## 决策原则

- 当前 `udbx4spec` 最小合规范围保持为 `tabular`、2D/3D 矢量、`text`、`cad`。
- `Network` / `Network3D` 不是普通 `line` / `lineZ` 的别名。它们包含拓扑边、节点子表和网络阻力字段，必须单独建模。
- `Model` 不是普通 CAD 或二进制字段。它依赖 `GeoModel3D`、`ModelNode`、`ModelEntity`、材质和纹理等复杂二进制结构，必须先补规范再实现。
- `SmDatasetType=204` 当前只在 `SampleData.udbx` 中作为 `modeldt_Texture` 出现。因白皮书表 1 未将 `204` 列为顶层数据集类型，暂不新增公开 `DatasetKind`。
- 三端 SDK 遇到这些类型时，可以在列表中暴露基础元信息，但不得宣称可完整读取、写入或 roundtrip。

## 后续准入条件

若后续决定支持这些类型，必须按以下顺序推进：

1. 在 `udbx4spec/docs/03-dataset-taxonomy.md` 中扩展数据集分类，明确 `Network`、`Network3D`、`Model` 和模型子表的概念边界。
2. 在 `udbx4spec` 增加 TypeScript、Java、JSON Schema 参考定义。
3. 设计 T0/T1/T3 合规夹具，避免直接使用完整真实样本作为规范资产。
4. 分别定义只读、写入、roundtrip 的发布门槛，不把只读观察误标为完整支持。
5. 三端 SDK 按同一规范实现，并更新能力矩阵、合规报告和发布说明。

## 当前样本处理结论

`SampleData.udbx` 中 `Jingjin_Network`、`Jingjin_NetworkZ`、`modeldt`、`modeldt_Texture` 保留为真实兼容性观察证据。它们不得进入当前 stable 合规门禁，也不得被用于证明三端 SDK 已支持网络或三维模型数据集。
