# Text / GeoText 规范化实施计划

创建日期：2026-06-23

## 目标

本计划启动时要求先把 GeoText 的格式事实、概念模型、二进制布局、合规夹具和跨语言验收标准补齐，再进入 `TextDataset` 实现。该计划的目标不是尽快写出临时代码，而是避免 Java、TypeScript、Go 各自实现出互不兼容的私有 GeoText 格式。

## 背景

计划启动时项目已经完成 `point`、`line`、`region`、`pointZ`、`lineZ`、`regionZ`、`tabular` 与 CAD 最小 GeoHeader 的三端合规闭环，当时剩余主要数据集缺口是 `text`。当前最新状态是：`text` 的最小 GeoText 基线也已完成 Java、TypeScript、Go 三端读写闭环。

`TextDataset` 不能直接照搬 CAD 或 GAIA 的实现方式。当前已从白皮书和真实样本补齐 GeoText 的最小规范依据，并已建立 Golden Bytes 与 `test_text` 合规资产；后续实现仍必须以这些规范资产为准，不能写出私有 GeoText 格式。

## 范围

本计划覆盖：

- GeoText 白皮书依据整理。
- 真实样本中的 Text 数据集分析。
- `udbx4spec` TextGeometry / TextFeature 规范定义。
- TextDataset 的系统表、字段和 `geometry_columns` 规则。
- GeoText Golden Bytes 和 `compliance.udbx` 的 `test_text` 夹具。
- Java、TypeScript、Go 三端最小读写闭环。

本计划暂不覆盖：

- 高级排版能力。
- 富文本段落。
- 字体文件嵌入。
- 与外部标注格式的转换。
- UI 标注编辑器。

## 原则

- 先规范，后实现。
- 先读取真实样本，后写出新样本。
- 先最小闭环，后扩展样式。
- 不用占位字符串或私有 JSON BLOB 冒充 GeoText。
- 不为通过当前测试硬编码样本字节。
- 所有二进制字段必须能说明来源：白皮书、样本反推、或明确标注为待确认假设。

## 阶段拆分

### T0：证据收集

目标：确认 GeoText 的权威格式来源和真实样本范围。

任务：

- 整理白皮书中 Text / GeoText / 文本对象相关章节。
- 收集至少 2 个包含 Text 数据集的真实 `.udbx` 样本。
- 在 `docs/samples/README.md` 中记录样本来源、授权、脱敏状态和校验哈希。
- 提取样本中的 `SmRegister`、`SmFieldInfo`、`geometry_columns` 和 Text 数据表结构。
- 提取至少 3 条 Text `SmGeometry` BLOB，保留原始字节和可读字段推断。

退出标准：

- 能明确说明 Text 数据集是否注册 `geometry_columns`。
- 能明确说明 Text 数据集 `SmDatasetType`、几何列、SRID、对象数的元数据规则。
- 至少有一份 Text BLOB 的字段偏移推断表。

### T1：规范冻结

目标：把已确认的 GeoText 概念和二进制布局写入 `udbx4spec`。

任务：

- 在 `udbx4spec/docs/02-geometry-model.md` 增加 `TextGeometry` 最小结构。
- 在 `udbx4spec/docs/03-dataset-taxonomy.md` 明确 TextDataset 的系统表规则。
- 在 `udbx4spec/reference/typescript/udbx4spec.d.ts` 更新 `TextGeometry` / `TextFeature`。
- 增加 GeoText 二进制布局说明文档，至少包含字段名、类型、字节序、编码、可选性和未知字段处理规则。
- 标注未确认字段，不允许把推断写成确定事实。

当前已落入 `udbx4spec` 的最小 `TextGeometry` 结构：

```typescript
interface TextGeometry {
  readonly type: "Text";
  readonly text: string;
  readonly anchor: readonly [number, number];
  readonly rotation?: number;
  readonly srid?: number;
  readonly bbox?: [number, number, number, number];
  readonly style?: TextStyle;
  readonly subTexts?: readonly TextSubText[];
}
```

退出标准：

- `TextGeometry` 的最小字段有规范来源。
- 二进制布局足以支持一个真实样本读取。
- 写入范围被明确限制为最小可验证字段。

当前状态：

- 已完成 `udbx4spec/docs/02-geometry-model.md` 的 `TextGeometry` 定义。
- 已完成 `udbx4spec/docs/03-dataset-taxonomy.md` 的 Text 系统表与 `geometry_columns` 规则说明。
- 已完成 `udbx4spec/docs/07-geotext-binary-layout.md`。
- 已完成 `udbx4spec/reference/typescript/udbx4spec.d.ts` 的 `TextGeometry` / `TextFeature` / `GeoTextCodecContract`。
- 已完成 `udbx4spec/reference/json-schema/geometry/text.json` 与 `feature/text-feature.json`。

### T2：合规资产

目标：让 `udbx4spec` 提供可执行 Text 合规资产。

任务：

- 新增 `compliance/golden-text-bytes/` 或等价目录，保存 GeoText Golden Bytes。
- 更新 `tools/check-fixtures.mjs`，校验 Text fixture 的文件大小、哈希、关键标记和 manifest。
- 更新 `tools/generate-compliance-db.mjs`，增加 `test_text`。
- 更新 `tools/generate-roundtrip-fixtures.mjs`，在实现可用后纳入 `test_text`。
- 更新 `compliance/roundtrip-matrix.md`，增加 Text 的 R3/R4/R5 验收项。

退出标准：

- `node tools/check-fixtures.mjs` 能验证 Text 资产。
- `compliance.udbx` 中的 `test_text` 可由资产校验脚本验证；SDK 读取能力留到 T3 验收。

当前状态：

- 已完成 `compliance/golden-text-bytes/`，包含 `geotext/simple-utf8.bin` 与机器可读 manifest。
- 已完成 `tools/generate-golden-text-bytes.mjs`，可重复生成 GeoText Golden Bytes。
- 已完成 `tools/check-fixtures.mjs` 的 GeoText 校验，覆盖 `geoType`、`styleSize`、`subCount`、TextStyle、ABGR 颜色、UTF-8 字符串长度、GeoSubText 和 `subAngle`。
- 已完成 `tools/generate-compliance-db.mjs` 的 `test_text` 数据集写入，`SmGeometry` 为 GeoText BLOB，`SmIndexKey` 注册到 `geometry_columns`。
- 当前 `compliance.udbx` 已包含 `test_text`；Text roundtrip 已纳入 R4/R5，并已接入 Java、TypeScript、Go 三端语义 roundtrip。

### T3：三端最小实现

目标：Java、TypeScript、Go 都能读写最小 TextDataset。

计划启动时建议顺序：

1. `udbx4j`：优先利用现有 `TextDataset` 框架，接入真实 GeoText 读取和写入。
2. `udbx4ts`：实现 `TextDataset` codec 和 CRUD，保持 `src/core/` 平台无关。
3. `udbx4go`：新增 `TextDataset`、`GeoTextCodec`、DataSource API 和公开类型导出。

最小能力：

- `getById`
- `list`
- `count`
- `insert`
- `insertMany`
- `update`
- `delete`
- `TextGeometry` decode / encode

退出标准：

- 三端均通过 `test_text` 读取。
- 三端均能写出 `test_text` roundtrip 夹具。
- 三端互读 Text roundtrip 结果语义一致。

### T4：文档和发布门禁

目标：把 Text 支持纳入公开状态和发布检查。

任务：

- 更新三端 `udbx4spec-compliance-report.md`。
- 更新 `docs/architecture/compatibility-matrix.md`。
- 更新 `docs/architecture/risk-register.md`。
- 更新各子项目 README、CHANGELOG、AGENTS.md。
- 在发布指南中加入 Text 合规测试命令。

退出标准：

- Text 相关风险从“格式不明”降级为“样本覆盖仍需扩展”。
- 发布门禁能阻止 Text 回归。

## 验收命令

```bash
cd udbx4spec
node tools/generate-golden-text-bytes.mjs
node tools/generate-compliance-db.mjs
node tools/generate-roundtrip-fixtures.mjs
node tools/check-fixtures.mjs
```

```bash
cd udbx4ts
npx vitest run tests/integration/udbx4spec-compliance.integration.spec.ts
npm run typecheck
```

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./internal/codec ./internal/dataset . -run 'Text|Udbx4Spec' -v
```

```bash
cd udbx4j
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Dgroups=integration -Dit.test=Udbx4SpecComplianceDatabaseReadTest failsafe:integration-test failsafe:verify
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Dgroups=integration -Dit.test=Udbx4SpecComplianceDatabaseRoundtripTest failsafe:integration-test failsafe:verify
```

## 当前状态

- CAD 最小 GeoHeader 基线已完成。
- T0 证据收集已确认 `data/SampleData.udbx` 与 `data/henan.udbx` 均包含真实 Text 数据集，并已用白皮书结构解析 `SmGeometry`。
- T1 规范冻结已完成最小基线：`udbx4spec` 已补入 `TextGeometry` / `TextFeature`、JSON Schema 和 `docs/07-geotext-binary-layout.md`。
- T2 合规资产已完成最小基线：`golden-text-bytes/geotext/simple-utf8.bin`、`golden-text-bytes/manifest.json`、`test_text` 和 `check-fixtures` 校验均已接入。
- T3 三端最小实现已完成：Java、TypeScript、Go 已实现最小 GeoText 编解码、`TextDataset` CRUD、`SmIndexKey` 写出，以及 `test_text` 单语言与跨语言 roundtrip。
- T4 文档和发布门禁已部分完成：三端合规报告、README 和能力矩阵已同步；后续重点转向真实样本兼容范围和更严格的发布门禁。

### 白皮书依据

权威来源为根目录 `UDBX开放数据格式白皮书(V1.0).pdf`。

- 3.1.5 文本数据集：Text 数据集一张表，系统字段为 `SmID`、`SmUserID`、`SmGeometry`、`SmIndexKey`。
- 3.1.5 文本数据集：`SmGeometry` 为 `GeoText` 二进制流，`SmIndexKey` 为对象范围，存储为 `GAIAPolygon`。
- 4.1 基本类型：对象二进制字节序为 Little-Endian。
- 4.1.2 String：字符串为 `int32 length + UTF-8 bytes`。
- 4.1.10 Color：颜色字节顺序为 `a`、`b`、`g`、`r`。
- 4.4 文本对象：Text 数据集中的 `GeoHeader.styleSize` 为 `0`。
- 4.4 文本对象：`GeoText` 由 `GeoHeader`、`subCount`、`TextStyle`、`GeoSubText[]` 组成。

### T0 初步证据

当前工作区中已确认两个 Text 样本：

| 文件 | Text 数据集 | SmDatasetType | 表名 | 对象数 | SRID | geometry_columns |
|---|---|---:|---|---:|---:|---|
| `data/SampleData.udbx` | `County_T` | 7 | `County_T` | 15 | 4326 | 注册 `smindexkey`，`geometry_type=3` |
| `data/henan.udbx` | `河南省标签` | 7 | `河南省标签` | 1 | 4490 | 注册 `smindexkey`，`geometry_type=3` |

两个样本的 Text 表结构一致：

| 字段 | SQLite 类型 | 说明 |
|---|---|---|
| `SmID` | `INTEGER` | 主键 |
| `SmUserID` | `INTEGER` | 系统用户 ID |
| `SmGeometry` | `BLOB` | GeoText 二进制几何 |
| `SmIndexKey` | `POLYGON` | 空间索引 envelope |

已抽取的 `SmGeometry` BLOB 特征：

| 文件 | SmID | 字节长度 | 头部特征 |
|---|---:|---:|---|
| `data/SampleData.udbx` | 1 | 122 | `geoType=7`，`styleSize=0`，`subCount=1`，文本长度 `30` |
| `data/SampleData.udbx` | 2 | 122 | 与 SmID=1 布局相同，锚点、旋转和文本不同 |
| `data/SampleData.udbx` | 3 | 122 | 与 SmID=1 布局相同，锚点、旋转和文本不同 |
| `data/henan.udbx` | 1 | 103 | `geoType=7`，`styleSize=0`，`subCount=1`，字体 `宋体`，文本 `河南省` |

`data/henan.udbx` / `河南省标签` / `SmID=1` 可作为第一份完整字段偏移证据：

| 偏移 | 长度 | 字段 | 值 | 来源 |
|---:|---:|---|---|---|
| 0 | 4 | `GeoHeader.geoType` | `7` | 白皮书 4.4，GeoText |
| 4 | 4 | `GeoHeader.styleSize` | `0` | 白皮书 4.4，Text 数据集固定为 0 |
| 8 | 4 | `subCount` | `1` | 白皮书 4.4 |
| 12 | 4 | `TextStyle.color` | `ABGR=(0,0,0,255)` | 白皮书 4.1.10 / 4.4 |
| 16 | 4 | `TextStyle.textStyleBit` | `fixedSize=10, weight=64, styleFlag=6, alignFlag=0` | 白皮书 4.4 |
| 20 | 4 | `TextStyle.bgColor` | `ABGR=(255,255,255,255)` | 白皮书 4.1.10 / 4.4 |
| 24 | 8 | `TextStyle.fontWidth` | `0` | 白皮书 4.4 |
| 32 | 8 | `TextStyle.fontHeight` | `0.406494140625` | 白皮书 4.4 |
| 40 | 8 | `TextStyle.pntAnchor.x` | `113.165187569688` | 白皮书 4.1.3 / 4.4 |
| 48 | 8 | `TextStyle.pntAnchor.y` | `33.875453985` | 白皮书 4.1.3 / 4.4 |
| 56 | 4 | `TextStyle.faceName.length` | `6` | 白皮书 4.1.2 |
| 60 | 6 | `TextStyle.faceName.bytes` | `宋体` | UTF-8 |
| 66 | 8 | `GeoSubText[0].pntAnchor.x` | `113.165187569688` | 白皮书 4.1.3 / 4.4 |
| 74 | 8 | `GeoSubText[0].pntAnchor.y` | `33.875453985` | 白皮书 4.1.3 / 4.4 |
| 82 | 4 | `GeoSubText[0].subAngle` | `0` | 白皮书 4.4，实际角度乘 10 |
| 86 | 4 | `GeoSubText[0].reserved` | `0` | 白皮书 4.4 |
| 90 | 4 | `GeoSubText[0].subText.length` | `9` | 白皮书 4.1.2 / 4.4 |
| 94 | 9 | `GeoSubText[0].subText.bytes` | `河南省` | UTF-8 |

`data/SampleData.udbx` / `County_T` 的样本显示同一布局，但 `faceName` 和 `subText` 中存在非 UTF-8 可读字节或编码异常显示，需要在实现阶段按原始字节保留并建立异常样本测试，不能仅按可读中文字符串判断兼容性。

初步判断：

- Text 数据集使用 `SmDatasetType=7`。
- Text 数据表包含 `SmGeometry` BLOB，而不是普通字符串字段。
- Text 数据集在真实样本中会注册 `geometry_columns`，几何列名为大小写不敏感的 `smindexkey`。
- `SmIndexKey` 的 `geometry_type=3`，用于对象范围和空间索引，不等同于 Text 主几何类型。
- `SmGeometry` 可按白皮书 `GeoText` 结构解析；最小读取实现至少应支持 `geoType=7`、`styleSize=0`、单个或多个 `GeoSubText`、`String` UTF-8 解码、ABGR 颜色和 `subAngle / 10` 的旋转角语义。
- 写入实现不得省略 `SmIndexKey`；最小写入应能根据 Text 锚点和字体高度生成可被真实软件接受的范围 polygon，具体范围规则仍需进一步样本验证。

## 失效条件

如果后续白皮书或真实样本证明本文档中的最小 `TextGeometry` 字段不足以表达真实 Text 对象，应先修订本计划和 `udbx4spec` 规范，再继续实现。
