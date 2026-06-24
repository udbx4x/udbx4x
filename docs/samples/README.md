# 样本文件说明

本文档记录工作区中 `.udbx` 样本文件的用途、状态和维护规则。

配套兼容状态请同时查看：

- [真实样本兼容矩阵](./compatibility-matrix.md)
- [性能基线方案](../guides/performance-baseline.md)
- [性能基线结果](./performance-baseline-results.md)
- [真实来源夹具准入指南](../guides/source-fixture-admission.md)

## Benchmark Artifacts

`docs/samples/benchmark-artifacts/` 用于保存可机器读取的性能基线输出，例如 TypeScript benchmark 的 JSON 结果。该目录中的文件是性能趋势证据，不是格式规范。

## 样本文件原则

- 样本文件是兼容性证据，不是格式规范本身。
- UDBX 白皮书是最高格式依据。
- 样本观察到的行为必须经过分析，不能直接硬编码成通用规则。
- 可公开分发的样本必须记录来源、授权和校验值。
- 私有或来源不明的样本不得进入公开 release 资产。

## 当前样本

| 文件 | 用途 | 当前状态 |
|---|---|---|
| `data/SampleData.udbx` | 基础集成测试样本，用于验证常见数据集读取，并提供 `County_T` Text 与 `CADDT` CAD 证据 | 维护者已确认可以公开；已记录哈希、大小、公开分发确认、生成产品版本、T3 派生夹具和脱敏结论 |
| `data/henan.udbx` | 真实世界兼容性样本，用于覆盖更多数据集和中文字段场景，并提供 `河南省标签` Text 数据集证据；当前也是第一阶段性能基线样本 | 公开状态待定；已记录哈希、大小、Text 结构观察和性能基线目标数据集，未确认前不得进入公开 release 资产或公开 CI 下载资产 |

## 样本详情

### SampleData.udbx

- 路径：`data/SampleData.udbx`
- 来源：项目维护者确认的可公开 UDBX 示例样本；从文件内部系统表可见创建时间集中在 `2020-12-11`，但不能仅凭系统表反推出外部发布来源。
- 授权名称：项目维护者确认的 `SampleData.udbx` 公开分发授权。
- 授权文本路径：`docs/samples/licenses/sampledata-public-distribution-confirmation.md`
- 生成工具：`SuperMap iDesktopX 2025 V12.0.1.0`。样本内部可确认 SQLite `3.24.0`、SpatiaLite `4.3.0-RC1` 和 `SmDataSourceInfo.SmLastUpdateTime=2020-12-11 03:04:03`。
- 可公开分发：是
- 公开策略：可以进入公开仓库、公开测试资产和公开 release 资产；T3 派生资产已具备 stable 工程准入记录。若上游后续提供正式许可证或更正生成产品版本，应同步更新授权确认文档和 manifest。
- SHA-256：`b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39`
- 文件大小：`19,841,024` 字节
- 覆盖 DatasetKind：`tabular`、`point`、`line`、`region`、`text`、`pointZ`、`lineZ`、`regionZ`、`cad`；另含 `SmDatasetType=4/203/204/205` 等当前未纳入 `udbx4spec` 最小合规范围的类型。
- 覆盖 FieldType：已观察到 `int32`、`double`、`ntext`、`text`、`geometry` 等字段类型；完整字段覆盖以 `SmFieldInfo` 和后续机器化样本清单为准。
- 用途：基础集成测试样本；T3 Source-derived Fixture 来源样本；Text / CAD / 3D / tabular 真实兼容证据。
- T3 派生夹具：
  - `sampledata-county-t-smid-1-smgeometry`：从 `County_T` / `SmID=1` 抽取 `SmGeometry`，验证异常 Text 原始字节保留和容错解码。
  - `sampledata-caddt-smid-1-smgeometry`：从 `CADDT` / `SmID=1` 抽取 `SmGeometry`，验证 CAD `GeoPoint` 无样式 GeoHeader 解码。
  - `sampledata-caddt-smid-16-smgeometry`：从 `CADDT` / `SmID=16` 抽取 `SmGeometry`，验证 CAD `GeoLine` 子对象和坐标解码。
  - `sampledata-caddt-smid-63-smgeometry`：从 `CADDT` / `SmID=63` 抽取 `SmGeometry`，验证 CAD `GeoRegion` 子对象和坐标解码。
  - `sampledata-3d-srid-zero-metadata`：从 `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 抽取系统表摘要，验证真实 3D 数据集中 `SmRegister.SmSRID=0`、`geometry_columns.srid=0`、`coord_dimension=3` 和 GAIA 3D `geoType` 共存。
- 脱敏结论：当前已进入 `udbx4spec/compliance/source-derived/` 的 T3 派生夹具仅包含最小二进制几何、文本样式字节或系统表摘要，不包含业务属性表、用户标识字段或完整真实数据库；按当前公开夹具范围判定为无需脱敏。完整 `SampleData.udbx` 仍应作为公开示例样本单独维护授权说明。
- Text 证据：
  - `SmRegister` 中存在 `County_T`，`SmDatasetType=7`，`SmObjectCount=15`，`SmSRID=4326`
  - `geometry_columns` 注册 `county_t / smindexkey`，`geometry_type=3`
  - Text 表结构为 `SmID`、`SmUserID`、`SmGeometry`、`SmIndexKey`
  - 已观察到 `SmGeometry` BLOB 长度为 `122` 字节，可按白皮书 `GeoText` 结构解析
  - `faceName.length=4`、`subText.length=30`，文本区域存在尾部空格填充和非 UTF-8 可读字节，需要保留原始字节作为异常兼容样本
- 已知限制：部分文本字节编码显示异常，不能作为规范字符串样本；`SmDatasetType=4/203/204/205` 已按白皮书单独归档，不得直接标记为当前 SDK 已支持。

### henan.udbx

- 路径：`data/henan.udbx`
- 来源：待补充
- 生成工具：待补充
- 可公开分发：待定
- 公开策略：未确认前仅作为本地工作区样本和内部验证样本使用，不得进入公开 release 资产、公开 CI 下载资产或公开文档分发包。
- SHA-256：`e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356`
- 文件大小：`200,163,328` 字节
- 覆盖 DatasetKind：至少覆盖 `point`、`line`、`region`、`text`
- 用途：真实世界兼容性样本；中文数据集和 Text 证据样本
- 性能基线用途：第一阶段读性能基线样本；当前固定使用 `weibo -> henan_P`
- Text 证据：
  - `SmRegister` 中存在 `河南省标签`，`SmDatasetType=7`，`SmObjectCount=1`，`SmSRID=4490`
  - `geometry_columns` 注册 `河南省标签 / smindexkey`，`geometry_type=3`
  - Text 表结构为 `SmID`、`SmUserID`、`SmGeometry`、`SmIndexKey`
  - 已观察到 `SmGeometry` BLOB 长度为 `103` 字节，可按白皮书 `GeoText` 结构完整解析
  - `SmID=1` 解码结果包括字体 `宋体`、文本 `河南省`、锚点 `(113.165187569688, 33.875453985)`、旋转角 `0`
- 已知限制：公开状态待定；来源、授权名称和生成工具版本仍待补充

## 维护要求

新增样本时必须记录：

- 文件名。
- 来源。
- 生成工具或来源软件版本，若已知。
- 是否允许公开分发。
- SHA-256。
- 文件大小。
- 覆盖的数据集类型。
- 关键字段类型。
- 已知异常或兼容性价值。
- 与 `udbx4j`、`udbx4ts`、`udbx4go` 的兼容状态，或明确标记为待验证。

建议格式：

```markdown
## sample-name.udbx

- 路径：
- 来源：
- 生成工具：
- 可公开分发：
- SHA-256：
- 文件大小：
- 覆盖 DatasetKind：
- 覆盖 FieldType：
- 用途：
- 已知限制：
```

新增样本后，还应同步更新：

- [真实样本兼容矩阵](./compatibility-matrix.md)
- `docs/guides/sample-management.md`
- 如有必要，`docs/architecture/risk-register.md`

## 合规夹具关系

样本文件和合规夹具不是同一概念：

- 样本文件用于观察真实兼容性。
- 合规夹具用于固定规范行为。
- `compliance.udbx` 应是小型、可控、可重复生成的规范样本。
- `golden-gaia-bytes` 应用于字节级编解码验证。

`udbx4spec/compliance/fixture-standard.md` 已定义合规夹具的 T0/T1/T2/T3 分层、manifest 必填字段、稳定性语义和准入规则。当前 `udbx4spec/compliance/fixtures/manifest.json`、`golden-*-bytes/manifest.json` 和 `roundtrip/manifest.json` 已维护机器可读清单，并由 `node tools/check-fixtures.mjs` 校验。

真实样本进入 `udbx4spec/compliance/` 前，必须先完成来源、授权、脱敏和规范价值确认。可公开的 `SampleData.udbx` 已接入首批 T3 `stable` Source-derived Fixture；`henan.udbx` 公开状态待定，不能作为公开合规夹具直接发布。

当前已从 `SampleData.udbx` 接入五个 T3 `stable` 派生夹具：`County_T` Text 异常字节、`CADDT` CAD Point / Line / Region 无样式 GeoHeader，以及 `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 的 3D `SmSRID=0` metadata-json。它们均由 `udbx4spec/tools/generate-source-derived-fixtures.mjs` 生成，并由 `udbx4spec/tools/check-fixtures.mjs` 校验。按当前派生资产范围，脱敏结论为无需脱敏；公开分发授权记录见 `docs/samples/licenses/sampledata-public-distribution-confirmation.md`。

## 当前待补信息

| 文件 | 待补内容 |
|---|---|
| `data/SampleData.udbx` | 异常文本字节的编码来源 |
| `data/henan.udbx` | 公开状态、来源、授权名称、生成工具版本、覆盖 FieldType |

## 使用规则

- 测试中使用样本文件时，应尽量通过相对路径引用。
- 不要在测试里依赖样本中的偶然记录顺序，除非该顺序是测试目标。
- 不要把样本中的特定字段名、数据集名作为通用格式规则。
- 若样本暴露格式差异，先记录到规范或风险文档，再修改 SDK。
