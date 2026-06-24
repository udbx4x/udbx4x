# 合规测试指南

本文档说明 `udbx4x` 的合规测试目标、执行顺序和当前工作重点。

## 合规测试目标

合规测试用于验证：

- SDK 是否遵循 `udbx4spec`。
- `DatasetKind`、`FieldType`、几何模型、错误分类是否一致。
- Java、TypeScript、Go 是否能读写相同语义的数据。
- 样本文件兼容行为是否稳定。

## 当前优先级

合规测试优先级从高到低：

1. `udbx4spec` 夹具和 golden bytes。
2. 跨语言 roundtrip。
3. 单语言单元测试。
4. 单语言集成测试。
5. 工具层端到端测试。

## 当前状态

根据现有文档和报告：

- `udbx4j`：已接入最小合规闭环，覆盖 2D/3D 矢量、tabular、Text / GeoText 最小基线和 CAD 最小 GeoHeader。
- `udbx4ts`：已接入最小合规闭环，覆盖 2D/3D 矢量、tabular、Text / GeoText 最小基线和 CAD 最小 GeoHeader。
- `udbx4go`：已接入最小合规闭环，覆盖 2D/3D 矢量、tabular、Text / GeoText 最小基线和 CAD 最小 GeoHeader。
- `udbx4spec`：已提供 point/line/region、三维、tabular、Text / GeoText 最小基线、CAD 最小基线、Golden Bytes、`compliance.udbx`、三实现 roundtrip 夹具和 `SampleData.udbx` 派生的 T3 stable source-derived 夹具；合规夹具已按 T0/T1/T2/T3 分层，并通过 manifest 的 `tier`、`stability`、`usage` 字段声明消费方式。

## 执行顺序

### 第一步：检查 `udbx4spec`

应确认：

- 命名规范存在。
- 几何模型存在。
- 数据集类型映射存在。
- 字段类型映射存在。
- 错误分类存在。
- 语言映射存在。
- 合规夹具标准存在，重点查看 `udbx4spec/compliance/fixture-standard.md`。
- `node tools/check-fixtures.mjs` 能校验 manifest 元字段和二进制资产。

当前需要继续补齐：

- 更多真实 SuperMap 主流 UDBX 样本
- 更复杂的 Text / CAD / 异常输入夹具
- 更严格的发布门禁和工具层端到端覆盖
- 基于真实样本的跨语言性能基线
- 按 `docs/guides/source-fixture-admission.md` 继续完善 T3 Source-derived Fixture 验收闭环；当前已建立 `SampleData.udbx` / `County_T` Text bytes、`SampleData.udbx` / `CADDT` CAD Point / Line / Region bytes，以及 `BaseMap_PZ` / `BaseMap_LZ` / `BaseMap_RZ` 的 3D `SmSRID=0` metadata-json 夹具，并接入 `check-fixtures` 校验。Java / TypeScript / Go 已有对应真实样本读取验证。

### 第二步：运行各 SDK 本地测试

#### Java

```bash
cd udbx4j
make test
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home mvn -Dtest=SmRegisterDaoSpecTest test
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home mvn -Djacoco.skip=true -Dit.test=SampleDataTextDatasetReadTest,Vector3DRealSampleReadTest test-compile failsafe:integration-test failsafe:verify
make test-stable-t3
```

#### TypeScript

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:stable-t3
npm run build
```

#### Go

```bash
cd udbx4go
make test
make test-stable-t3
```

### 第三步：核对合规报告

重点关注：

- `udbx4j/udbx4spec-compliance-report.md`
- `udbx4ts/udbx4spec-compliance-report.md`
- `udbx4go/udbx4spec-compliance-report.md`

检查：

- 哪些能力为 ✅
- 哪些能力为 ⚠️
- 哪些缺口仍然未关闭

### 第四步：核对真实样本兼容状态

重点关注：

- `docs/samples/README.md`
- `docs/samples/compatibility-matrix.md`
- `docs/guides/performance-baseline.md`

检查：

- 当前发布或验收范围是否只覆盖 `udbx4spec` 最小合规夹具，还是已经包含真实样本验证。
- 真实样本中的 Text / CAD / 中文数据集名 / 混合 SRID / 表名映射 / 非当前规范类型 等特征，是否已有明确验证结论。
- 若仅有结构观察而无专门自动化验证，文档中是否明确标记为 `待补专门验证`。
- 若真实样本正确性已验证，是否已进入统一性能基线计划，尤其是 `henan.udbx` 的 `weibo -> henan_P`。

## 跨语言 roundtrip

目标矩阵：

- Java 写 -> Java / TypeScript / Go 读
- TypeScript 写 -> Java / TypeScript / Go 读
- Go 写 -> Java / TypeScript / Go 读

当前要求：

- 不把“单语言测试通过”等同于“跨语言一致”。
- 任一跨语言差异都应先记录到规范或风险登记中。

## 失败时处理

若合规失败：

1. 判断失败来自规范缺失、SDK 实现错误、样本异常还是测试错误。
2. 补充回归测试或夹具。
3. 修复根因。
4. 更新合规报告和相关文档。
