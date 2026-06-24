# 真实样本兼容矩阵

本文档记录当前工作区真实 `.udbx` 样本在 `udbx4j`、`udbx4ts`、`udbx4go` 中的兼容状态。它的作用是回答两个问题：

- 当前 SDK 已经对哪些真实样本特征建立了明确证据。
- 哪些样本行为仍只有结构观察，还没有建立专门自动化验证。

本矩阵是兼容性证据，不是格式规范。格式语义仍以 UDBX 白皮书和 `udbx4spec` 为准。

## 状态说明

- `✅ 已验证`：已有专门测试、实现验证或明确的自动化证据。
- `⚠️ 待补专门验证`：当前实现按规范应可支持，或已有间接证据，但尚未建立该真实样本的专门自动化验证。
- `❌ 当前不支持`：样本使用了当前 `udbx4spec` 和三端 SDK 未覆盖的类型或行为。
- `N/A`：不适用。

## 样本概览

| 样本 | 主要价值 | 当前主要风险 |
|---|---|---|
| `data/SampleData.udbx` | 基础集成测试样本；覆盖 2D/3D 矢量、tabular、Text、CAD，并暴露超出当前规范范围的旧类型 | 维护者已确认可以公开；授权确认文件、生成产品版本、T3 派生夹具和无需脱敏结论已记录；`County_T` 存在异常文本字节；包含 `4/203/204/205` 等非当前规范类型 |
| `data/henan.udbx` | 真实世界兼容样本；覆盖中文数据集名、Text、混合 SRID、数据集名与物理表名不一致；当前也是第一阶段读性能基线样本 | 公开状态待定；未确认前不得进入公开 release 或公开 CI 下载资产；性能基线仍需定期复跑和趋势维护 |

## SampleData.udbx

### 样本特征

- 文件：`data/SampleData.udbx`
- 公开状态：可以公开
- 授权状态：维护者已确认可公开分发；授权确认文件为 `docs/samples/licenses/sampledata-public-distribution-confirmation.md`。
- 生成工具：`SuperMap iDesktopX 2025 V12.0.1.0`。
- T3 脱敏结论：当前已抽取的 Text / CAD bytes 与 3D metadata 派生夹具仅包含最小几何、文本样式字节或系统表摘要，不包含业务属性表、用户标识字段或完整真实数据库，判定为无需脱敏。
- SHA-256：`b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39`
- 文件大小：`19,841,024` 字节
- `SmRegister` 共 15 个数据集条目
- 当前属于 `udbx4spec` 最小合规范围的数据集：
  - `tabular`：`TabularDT`
  - `point`：`BaseMap_P`、`Jingjin_Network_Node`
  - `line`：`BaseMap_L`
  - `region`：`BaseMap_R`
  - `text`：`County_T`
  - `pointZ`：`BaseMap_PZ`、`Jingjin_NetworkZ_Node`
  - `lineZ`：`BaseMap_LZ`
  - `regionZ`：`BaseMap_RZ`
  - `cad`：`CADDT`
- 超出当前 `udbx4spec` 最小合规范围的数据集：
  - `Jingjin_Network`：`SmDatasetType=4`，白皮书定义为二维网络数据集。
  - `modeldt`：`SmDatasetType=203`，白皮书定义为三维模型数据集。
  - `modeldt_Texture`：`SmDatasetType=204`，白皮书表 1 未列为顶层数据集类型，当前按模型资源/纹理子表证据处理。
  - `Jingjin_NetworkZ`：`SmDatasetType=205`，白皮书定义为三维网络数据集。

### 代表性数据集矩阵

| 数据集 | Kind | 对象数 | 字段数 | 关键特征 | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---|---:|---:|---|---|---|---|---|
| `TabularDT` | `tabular` | 15 | 15 | 纯属性表；字段类型含 `4/7/127` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 三端均已补真实样本测试，覆盖数据集元信息、计数、代表记录读取和无几何列约束 |
| `BaseMap_P` | `point` | 15 | 7 | `geometry_columns.smgeometry type=1, srid=4326` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java 有专门 `PointDatasetReadTest`；TS/Go 已补真实样本集成测试 |
| `BaseMap_L` | `line` | 47 | 5 | `geometry_columns.smgeometry type=5, srid=4326` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java 有专门 `LineDatasetReadTest`；TS/Go 已补真实样本集成测试 |
| `BaseMap_R` | `region` | 15 | 18 | `geometry_columns.smgeometry type=6, srid=4326` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java 有专门 `RegionDatasetReadTest`；TS/Go 已补真实样本集成测试 |
| `County_T` | `text` | 15 | 4 | `geometry_columns.smindexkey type=3, srid=4326`；GeoText 长度 `122`；存在异常文本字节 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 已接入 `sampledata-county-t-smid-1-smgeometry`；Java `SampleDataTextDatasetReadTest` 已覆盖异常字节容错解码；TS/Go 已补真实样本集成测试 |
| `BaseMap_PZ` | `pointZ` | 15 | 7 | `SmRegister.SmSRID=0`；`geometry_columns.smgeometry type=1001, srid=0, coord_dimension=3` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 已接入 `sampledata-3d-srid-zero-metadata`；Java `Vector3DRealSampleReadTest` 已覆盖 `srid=0`、`geometryType=1001` 和 Z 坐标读取；TS/Go 已补真实样本集成测试 |
| `BaseMap_LZ` | `lineZ` | 47 | 5 | `SmRegister.SmSRID=0`；`geometry_columns.smgeometry type=1005, srid=0, coord_dimension=3` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 已接入 `sampledata-3d-srid-zero-metadata`；Java `Vector3DRealSampleReadTest` 已覆盖 `srid=0`、`geometryType=1005` 和 Z 坐标读取；TS/Go 已补真实样本集成测试 |
| `BaseMap_RZ` | `regionZ` | 15 | 18 | `SmRegister.SmSRID=0`；`geometry_columns.smgeometry type=1006, srid=0, coord_dimension=3` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 已接入 `sampledata-3d-srid-zero-metadata`；Java `Vector3DRealSampleReadTest` 已覆盖 `srid=0`、`geometryType=1006` 和 Z 坐标读取；TS/Go 已补真实样本集成测试 |
| `CADDT` | `cad` | 92 | 5 | `SmIndexKey` 注册到 `geometry_columns`；含 `SmGeoType` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java 有专门 `CadDatasetReadTest`；TS/Go 已补真实样本集成测试，覆盖读取、计数和前 5 条 CAD 几何解码 |
| `Jingjin_Network` | `SmDatasetType=4` | 1622 | 12 | 白皮书 `Network`，二维网络数据集，含边、节点和阻力字段 | ❌ 当前不支持 | ❌ 当前不支持 | ❌ 当前不支持 | 已归档到 [数据集类型支持边界决策](../architecture/dataset-type-boundary-decisions.md)，需先进入 `udbx4spec` 扩展设计 |
| `modeldt` | `SmDatasetType=203` | 2 | 9 | 白皮书 `Model`，三维模型数据集，涉及 `GeoModel3D` | ❌ 当前不支持 | ❌ 当前不支持 | ❌ 当前不支持 | 已归档到支持边界决策；需先建立模型二进制规范 |
| `modeldt_Texture` | `SmDatasetType=204` | 47 | 3 | 模型纹理/资源子表证据；表 1 未列为顶层数据集类型 | ❌ 当前不支持 | ❌ 当前不支持 | ❌ 当前不支持 | 不作为独立 `DatasetKind` 直接纳入 |
| `Jingjin_NetworkZ` | `SmDatasetType=205` | 570 | 14 | 白皮书 `Network3D`，三维网络数据集 | ❌ 当前不支持 | ❌ 当前不支持 | ❌ 当前不支持 | 已归档到支持边界决策；应与 `Network` 一并设计 |

## henan.udbx

### 样本特征

- 文件：`data/henan.udbx`
- 公开状态：待定
- SHA-256：`e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356`
- 文件大小：`200,163,328` 字节
- `SmRegister` 当前可见 23 个数据集条目
- 当前属于 `udbx4spec` 最小合规范围的数据集主要覆盖：
  - `point`：`居民地地名`、`省会`、`县`、`乡`、`镇`、`村` 等
  - `line`：`公路`、`省道`、`县道`、`乡道`、`groad`
  - `region`：`省级行政区划`、`地级行政区划`、`县级行政区划`、`水库面`、`湖泊`、`自然河流` 等
  - `text`：`河南省标签`
- 该样本的重要真实世界特征：
  - 存在中文数据集名和中文 Text 内容。
  - `河南省标签` 使用 `SmSRID=4490`，而其他多数数据集使用 `4326`。
  - `水库面` 使用 `3857`。
  - 存在 `SmDatasetName` 与 `SmTableName` 不一致的数据集，如 `streetroad -> 街道`、`groad -> 国道`、`weibo -> henan_P`、`city -> 市级行政区划`。

### 代表性数据集矩阵

| 数据集 | Kind | 对象数 | 字段数 | 关键特征 | `udbx4j` | `udbx4ts` | `udbx4go` | 说明 |
|---|---|---:|---:|---|---|---|---|---|
| `居民地地名` | `point` | 1863 | 10 | 中文数据集名；`geometry_columns.smgeometry type=1` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java `HenanDatasetReadTest.valid_point_dataset_returns_features()` 已覆盖；TS/Go 已补真实样本集成测试 |
| `公路` | `line` | 7805 | 13 | 中文字段；`geometry_columns.smgeometry type=5`；空间索引标记为 `1` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java `valid_line_dataset_returns_features()` 已覆盖；TS/Go 已补真实样本集成测试 |
| `省级行政区划` | `region` | 1 | 12 | 中文数据集名；`geometry_columns.smgeometry type=6, srid=4326` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java `valid_region_dataset_returns_features()` 已覆盖；TS/Go 已补真实样本集成测试 |
| `水库面` | `region` | 71 | 15 | `srid=3857`；字段类型含 `4/7/127/128` | ⚠️ 待补专门验证 | ✅ 已验证 | ✅ 已验证 | TS/Go 已补真实样本集成测试，覆盖混合 SRID 场景 |
| `weibo` / `henan_P` | `point` | 469308 | 5 | 大对象数；`SmDatasetName=weibo`，`SmTableName=henan_P` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 三端已补真实样本测试，覆盖表名映射、对象总数、分页顺序和代表字段值；已建立首批三端读性能基线 |
| `河南省标签` | `text` | 1 | 4 | `SmSRID=4490`；`geometry_columns.smindexkey type=3`；字体 `宋体`；文本 `河南省` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | Java `text_dataset_reads_geotext()` 已覆盖；TS/Go 已补真实样本集成测试，验证文本、锚点、字体和字号 |
| `streetroad` | `point` | 63 | 11 | `SmDatasetName=streetroad`，`SmTableName=街道` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 三端已补真实样本测试，验证按 `SmTableName` 读取物理表 |
| `groad` | `line` | 164 | 14 | `SmDatasetName=groad`，`SmTableName=国道` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 三端已补真实样本测试，验证按 `SmTableName` 读取物理表 |
| `city` | `region` | 18 | 6 | `SmDatasetName=city`，`SmTableName=市级行政区划` | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | 三端已补真实样本测试，验证按 `SmTableName` 读取物理表 |

## 当前结论

1. 三端已对两个真实样本建立了持续扩大的自动化证据，覆盖 `SampleData.udbx` 的 `tabular` / 点 / 线 / 面 / Text / 3D / CAD，以及 `henan.udbx` 的中文数据集名、Text、混合 SRID、数据集名与物理表名不一致、大对象分页读取。Java 侧已补 `SampleData.udbx` Text 异常字节和 3D `SmSRID=0` 专门测试。
2. `udbx4ts` 和 `udbx4go` 已补入真实样本集成测试，覆盖 `SampleData.udbx` 的 `BaseMap_P` / `BaseMap_L` / `BaseMap_R` / `County_T` / `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` / `CADDT` / `TabularDT`，以及 `henan.udbx` 的 `居民地地名` / `公路` / `省级行政区划` / `水库面` / `河南省标签` / `streetroad` / `groad` / `city` / `weibo`。
3. `udbx4j` 已补齐 `TabularDT`、`County_T`、`BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 和 `weibo` 的真实样本集成测试，同时修正了按 `SmTableName` 访问物理表的实现与验证路径。
4. `SampleData.udbx` 中的 `SmDatasetType=4/203/204/205` 已按白皮书和样本结构归档到 [数据集类型支持边界决策](../architecture/dataset-type-boundary-decisions.md)，这些类型不能被误写成“已支持”。
5. 下一阶段优先级应从“补分页正确性”转向“复跑和维护性能基线、`henan.udbx` 公开状态确认、样本来源与授权补齐、非当前规范类型确认”。性能基线口径见 [性能基线方案](../guides/performance-baseline.md)，首批结果见 [性能基线结果](./performance-baseline-results.md)。

## 下一步建议

- 按 [性能基线方案](../guides/performance-baseline.md) 定期复跑 `henan.udbx` 的 `weibo` / `henan_P` 三端 benchmark，并持续更新 [性能基线结果](./performance-baseline-results.md)。
- 按 [真实来源夹具准入指南](../guides/source-fixture-admission.md) 继续将真实样本中稳定、可公开、可脱敏的 Text / CAD / 异常输入沉淀到 `udbx4spec` T3 fixture；当前已从 `SampleData.udbx` / `County_T` 接入 `stable` T3 Text bytes 夹具，从 `SampleData.udbx` / `CADDT` 接入 `GeoPoint`、`GeoLine`、`GeoRegion` 三个 `stable` T3 CAD bytes 夹具，并接入 `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 的 3D `SmSRID=0` metadata-json 夹具。
- 按 [数据集类型支持边界决策](../architecture/dataset-type-boundary-decisions.md) 设计 `Network` / `Network3D` / `Model` 的后续规范扩展；当前不纳入最小合规基线。
- 继续确认 `SampleData.udbx` 的异常文本字节编码来源；确认 `henan.udbx` 是否可公开分发。
