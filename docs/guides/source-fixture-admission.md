# 真实来源夹具准入指南

本文档定义将真实 `.udbx` 样本中的稳定行为沉淀为 `udbx4spec` T3 Source-derived Fixture 的操作流程。T3 夹具用于补充真实兼容证据，但仍然是规范资产，不是真实样本归档。

## 适用范围

适用于以下场景：

- 从 `data/SampleData.udbx` 抽取可公开、可解释、可重复生成的小型合规夹具。
- 从未来确认可公开的 SuperMap 主流 UDBX 样本抽取规范行为。
- 将真实样本中稳定的 Text、CAD、三维、异常编码、系统表差异或数据集类型行为转化为跨语言验收资产。

不适用于以下场景：

- 直接发布完整真实样本。
- 把来源、授权或脱敏状态未确认的样本放入 `udbx4spec/compliance/`。
- 把性能基线样本作为合规夹具。
- 把真实样本中的偶然字段值、记录顺序或业务内容写成格式规则。

## 准入前置条件

候选样本必须先在 `docs/samples/README.md` 中记录：

- 文件路径。
- 来源。
- 授权名称或公开分发依据。
- 生成工具和版本，若已知。
- 是否允许公开分发。
- SHA-256。
- 文件大小。
- 覆盖的 `DatasetKind` 和 `FieldType`。
- 兼容性价值和已知限制。

若任一来源、授权、公开分发或脱敏结论缺失，该样本只能用于本地观察和内部验证，不得进入 `udbx4spec/compliance/`。

## T3 夹具选择原则

T3 夹具必须满足：

- 小型：只保留验证目标所需的最小数据集、字段和记录。
- 可公开：来源和授权允许公开分发。
- 可重复：由脚本从源样本抽取或重建，不能手工编辑 `.udbx`、`.bin`。
- 可解释：每个保留字段、系统表行为和二进制行为都有明确规范价值。
- 可验证：至少一个 SDK 有自动化测试，`udbx4spec/tools/check-fixtures.mjs` 能校验 manifest 和关键语义。

T3 夹具不要求完全复刻真实文件。优先做源样本派生的最小化资产，例如：

- 从真实 Text 中抽取一个异常编码或样式组合，生成最小 `.bin` golden bytes。
- 从真实 CAD 中抽取一个已确认的几何类型，生成最小数据库。
- 从真实样本中抽取一个数据集名与物理表名不一致的最小数据库。
- 从真实样本中抽取一个混合 SRID 或三维数据集的最小数据库。

## 当前样本准入状态

| 样本 | 公开状态 | T3 价值 | 准入结论 |
|---|---|---|---|
| `data/SampleData.udbx` | 可以公开 | 覆盖 2D/3D 矢量、tabular、Text、CAD，并包含 `SmDatasetType=4/203/204/205` 等当前未纳入规范的类型 | 已接入首批 T3 `stable` source-derived 夹具；维护者已确认公开分发状态，授权确认记录见 `docs/samples/licenses/sampledata-public-distribution-confirmation.md`；当前 T3 派生夹具无需脱敏；生成产品版本为 `SuperMap iDesktopX 2025 V12.0.1.0` |
| `data/henan.udbx` | 待定 | 覆盖中文数据集名、混合 SRID、数据集名与物理表名不一致、大对象分页和 GeoText UTF-8 样本 | 公开状态未确认前不得进入公开 T3 夹具；只能保留为本地样本和性能基线输入 |

`SampleData.udbx` 当前已确认的数据集覆盖如下：

| 数据集 | `DatasetKind` / 类型值 | 对象数 | 规范价值 |
|---|---:|---:|---|
| `BaseMap_P` | `point` / 1 | 15 | 2D 点真实样本 |
| `BaseMap_L` | `line` / 3 | 47 | 2D 线真实样本 |
| `BaseMap_R` | `region` / 5 | 15 | 2D 面真实样本 |
| `County_T` | `text` / 7 | 15 | Text 数据集，存在异常文本字节 |
| `BaseMap_PZ` | `pointZ` / 101 | 15 | 3D 点真实样本，`SmSRID=0` |
| `BaseMap_LZ` | `lineZ` / 103 | 47 | 3D 线真实样本，`SmSRID=0` |
| `BaseMap_RZ` | `regionZ` / 105 | 15 | 3D 面真实样本，`SmSRID=0` |
| `TabularDT` | `tabular` / 0 | 15 | 纯属性表，字段类型含 `int32`、`double`、`ntext` |
| `CADDT` | `cad` / 149 | 92 | CAD 真实样本，含 `SmGeoType` |
| `Jingjin_Network` | 4 | 1622 | 白皮书 `Network`，当前不纳入最小合规基线 |
| `modeldt` | 203 | 2 | 白皮书 `Model`，当前不纳入最小合规基线 |
| `modeldt_Texture` | 204 | 47 | 模型资源/纹理子表证据，暂不作为独立 `DatasetKind` |
| `Jingjin_NetworkZ` | 205 | 570 | 白皮书 `Network3D`，当前不纳入最小合规基线 |

## 首批 T3 资产与后续候选清单

| 候选 ID | 来源 | 目标行为 | 建议输出 | 建议稳定性 | 当前状态 | 阻塞项 |
|---|---|---|---|---|---|---|
| `sampledata-county-t-smid-1-smgeometry` | `SampleData.udbx` / `County_T` / `SmID=1 SmGeometry` | 真实 Text 中非 UTF-8 可读文本字节的保留和容错解码 | `.bin` | `stable` | 已接入 T3 生成与校验；Java / TypeScript / Go 已有真实样本读取验证；脱敏结论为无需脱敏；授权确认记录和生成产品版本已补齐 | 异常文本字节编码来源仍需确认 |
| `sampledata-caddt-smid-1-smgeometry` | `SampleData.udbx` / `CADDT` / `SmID=1 SmGeometry` | 真实 CAD `GeoPoint` 无样式 GeoHeader BLOB 解码 | `.bin` | `stable` | 已接入 T3 生成、校验与三端 decode 测试；脱敏结论为无需脱敏；授权确认记录和生成产品版本已补齐 | 无 |
| `sampledata-caddt-smid-16-smgeometry` | `SampleData.udbx` / `CADDT` / `SmID=16 SmGeometry` | 真实 CAD `GeoLine` 子对象和坐标解码 | `.bin` | `stable` | 已接入 T3 生成、校验与三端 decode 测试；脱敏结论为无需脱敏；授权确认记录和生成产品版本已补齐 | 无 |
| `sampledata-caddt-smid-63-smgeometry` | `SampleData.udbx` / `CADDT` / `SmID=63 SmGeometry` | 真实 CAD `GeoRegion` 子对象和坐标解码 | `.bin` | `stable` | 已接入 T3 生成、校验与三端 decode 测试；脱敏结论为无需脱敏；授权确认记录和生成产品版本已补齐 | 无 |
| `sampledata-3d-srid-zero-metadata` | `SampleData.udbx` / `BaseMap_PZ`、`BaseMap_LZ`、`BaseMap_RZ` | 三维数据集 `SmSRID=0` 与 `geometry_columns.srid=0`、`coord_dimension=3`、GAIA 3D `geoType` 共存 | `metadata-json` | `stable` | 已接入 T3 生成与校验；Java / TypeScript / Go 已有真实样本读取验证；脱敏结论为无需脱敏；授权确认记录和生成产品版本已补齐 | 无 |
| `sampledata-legacy-network-type-4` | `SampleData.udbx` / `Jingjin_Network` | `SmDatasetType=4` 的白皮书语义和 SDK 支持边界 | 决策记录，暂不生成夹具 | N/A | 已归档，暂缓实现 | 白皮书已确认为 `Network`；当前不纳入最小合规基线 |
| `sampledata-model-type-203-204-205` | `SampleData.udbx` / `modeldt`、`modeldt_Texture`、`Jingjin_NetworkZ` | `SmDatasetType=203/204/205` 的白皮书语义和 SDK 支持边界 | 决策记录，暂不生成夹具 | N/A | 已归档，暂缓实现 | `203` 为 `Model`，`205` 为 `Network3D`，`204` 暂按模型资源/纹理子表证据处理 |
| `henan-table-name-mapping` | `henan.udbx` / `streetroad`、`groad`、`city`、`weibo` | `SmDatasetName` 与 `SmTableName` 不一致的读取行为 | 最小 `.udbx` | `experimental` | 不准入 | `henan.udbx` 公开状态待定 |
| `henan-mixed-srid` | `henan.udbx` / `水库面`、`河南省标签` | 混合 SRID 真实样本行为 | 最小 `.udbx` | `experimental` | 不准入 | `henan.udbx` 公开状态待定 |

当前阶段只允许推进基于 `SampleData.udbx` 的后续公开 T3 扩展，以及补充异常文本字节编码来源等增强证据。任何基于 `henan.udbx` 的公开 T3 资产，必须等待公开状态确认后再进入准入流程。

`golden-text-bytes/geotext/simple-utf8.bin` 是基于 `henan.udbx` 结构最小化重建的 T0 Golden Bytes，用于验证白皮书和样本共同确认的 GeoText 最小布局；它不是 `henan.udbx` 原始记录发布，也不代表 `henan.udbx` 已通过 T3 公开准入。

## 准入流程

### 1. 建立候选记录

在 `docs/samples/README.md` 和 `docs/samples/compatibility-matrix.md` 中记录候选样本和目标行为。

记录必须说明：

- 目标行为属于格式事实、API 语义还是兼容性证据。
- 是否已在白皮书或 `udbx4spec` 中有对应概念。
- 是否需要先补规范文档或决策记录。

### 2. 做准入判定

按以下结论之一处理：

| 结论 | 处理方式 |
|---|---|
| 可直接准入 | 来源、授权、脱敏、规范价值均明确，可以进入抽取脚本设计 |
| 先补规范 | 行为有价值，但 `udbx4spec` 尚无对应概念或术语，先改规范再抽取 |
| 仅保留样本证据 | 样本有兼容价值，但不适合作为稳定合规夹具 |
| 拒绝准入 | 来源、授权、脱敏或规范价值不满足要求 |

### 3. 设计最小夹具

设计时必须明确：

- 输出类型：`.bin`、`.udbx` 或 manifest-only 记录。
- 输出目录：建议新增 `udbx4spec/compliance/source-derived/`，或在现有 `golden-*-bytes/` 下新增明确来源的条目。
- `tier`：必须为 `T3`。
- `stability`：首次接入真实来源行为时建议为 `experimental`；进入发布门禁前再提升为 `stable`。
- `usage`：按实际用途选择 `decode`、`read`、`roundtrip` 或 `performance`。
- 源样本引用：必须包含源文件名、源 SHA-256、源数据集、源记录或抽取条件。
- 规范价值：说明该夹具验证的格式事实或兼容性边界。

### 4. 编写生成脚本

生成脚本必须放在 `udbx4spec/tools/`，命名建议：

```text
generate-source-derived-fixtures.mjs
```

脚本职责：

- 校验源样本 SHA-256。
- 从源样本读取目标系统表、字段、记录或 BLOB。
- 生成最小化 T3 夹具。
- 写出 manifest，包含 `tier`、`stability`、`usage`、`source`、`licenseStatus`、`deidentification`、`specValue`。
- 保持输出稳定，避免当前时间、随机 ID、环境路径进入资产。

### 5. 扩展校验脚本

更新 `udbx4spec/tools/check-fixtures.mjs`：

- 校验 T3 manifest 的基础字段。
- 校验源样本引用字段存在。
- 校验输出文件大小和 SHA-256。
- 校验目标行为的关键系统表或二进制标记。

### 6. 接入三端验证

至少完成：

- 一个语言实现的专门自动化测试。
- `docs/samples/compatibility-matrix.md` 的状态更新。
- `udbx4spec/compliance/roundtrip-matrix.md` 的 R6 记录更新。

若 T3 夹具提升为 `stable`，必须补齐 Java、TypeScript、Go 三端验证。

## Manifest 字段建议

T3 条目除通用字段外，应增加：

```json
{
  "id": "sampledata-county-t-geotext-anomalous-bytes",
  "path": "source-derived/sampledata/county-t-geotext-anomalous-bytes.bin",
  "tier": "T3",
  "stability": "experimental",
  "usage": ["decode"],
  "source": {
    "sample": "data/SampleData.udbx",
    "sha256": "b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39",
    "dataset": "County_T",
    "selection": "SmID=1 SmGeometry"
  },
  "licenseStatus": "public-confirmed",
  "deidentification": "not-required",
  "specValue": "验证真实 Text 数据集中非 UTF-8 可读文本字节的保留和容错解码行为"
}
```

字段说明：

- `source.sample`：源样本路径。
- `source.sha256`：源样本 SHA-256。
- `source.dataset`：源数据集名称。
- `source.selection`：抽取记录、字段或筛选条件。
- `licenseStatus`：`public-confirmed`、`internal-only`、`pending`。
- `deidentification`：`not-required`、`applied`、`pending`。
- `specValue`：该夹具沉淀的规范或兼容价值。

`licenseStatus` 不是法律意见，只是项目内工程准入状态。发布前仍应以维护者确认的授权信息为准。

## 首批建议路线

1. 已为 `County_T` / `SmID=1` / `SmGeometry` 建立 T3 `stable` bytes 夹具，用于验证异常 Text 原始字节保留；Java / TypeScript / Go 已有真实样本读取验证。
2. 已为 `CADDT` / `SmID=1/16/63` / `SmGeometry` 建立 T3 `stable` CAD bytes 夹具，用于验证真实 CAD Point / Line / Region 无样式 GeoHeader 解码。
3. 已为 `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 建立 T3 `stable` metadata-json 夹具，用于验证真实 3D 数据集 `SmSRID=0` 与 `geometry_columns.srid=0` 的系统表组合；Java / TypeScript / Go 已有真实样本读取验证。
4. 继续尝试确认 `SampleData.udbx` 的异常文本字节编码来源；当前已接入的 `sampledata-county-t-smid-1-smgeometry`、`sampledata-caddt-*` 与 `sampledata-3d-srid-zero-metadata` 已具备 stable 工程准入记录。
5. `SmDatasetType=4/203/204/205` 已完成白皮书核对和支持边界归档，后续只在完成 `udbx4spec` 扩展设计后再考虑夹具。
6. 若 `henan.udbx` 公开状态确认，再考虑将中文数据集名、混合 SRID、表名映射行为转为 T3 候选。

## 验收命令

每次新增或修改 T3 夹具后，至少执行：

```bash
cd udbx4spec
node tools/check-fixtures.mjs
```

若新增脚本依赖 `SampleData.udbx`，还应执行：

```bash
shasum -a 256 ../data/SampleData.udbx
```

预期 SHA-256：

```text
b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39
```

若 SHA-256 不一致，必须停止生成，先确认样本版本。
