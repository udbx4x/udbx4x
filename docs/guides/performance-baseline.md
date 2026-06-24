# 性能基线方案

本文档定义 `udbx4x` 工作区的性能基线方法。它用于统一 `udbx4j`、`udbx4ts`、`udbx4go` 的性能观测口径，避免不同语言实现用不同样本、不同指标、不同命令得出不可比较的结论。

性能基线不是格式规范。格式语义仍以 UDBX 白皮书和 `udbx4spec` 为准；性能基线只回答“当前实现处理真实样本时是否稳定、是否退化、是否满足目标用户场景”。

## 目标

性能基线要达成以下目标：

- 建立可重复执行的本地性能观测流程。
- 以真实 `.udbx` 样本验证大对象读取、分页读取和计数能力。
- 用统一指标比较三端 SDK 的趋势，而不是只比较单次耗时。
- 为后续优化提供回归判断依据。
- 在发布前发现明显性能退化。

当前第一阶段只建立读路径基线，不建立写入基线。写入性能基线应在跨语言写入一致性、样本生成和授权问题稳定后再补。

## 基线样本

第一阶段固定使用 `data/henan.udbx` 中的 `weibo -> henan_P` 数据集：

| 项 | 值 |
|---|---|
| 文件 | `data/henan.udbx` |
| SHA-256 | `e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356` |
| 文件大小 | `200,163,328` 字节 |
| 数据集名 | `weibo` |
| 物理表名 | `henan_P` |
| DatasetKind | `point` |
| SRID | `4326` |
| 对象数 | `469308` |
| 字段特征 | `count1`、`count2` 等简单属性字段 |

选择该样本的原因：

- 数据量足够大，能暴露分页和对象解码成本。
- `SmDatasetName` 与 `SmTableName` 不一致，可同时覆盖真实兼容路径。
- 当前三端已经建立正确性测试，性能基线可以建立在已验证行为之上。

## 指标定义

第一阶段统一记录以下指标：

| 指标 | 说明 | 必须记录 |
|---|---|---|
| `open_data_source_ms` | 打开 `henan.udbx` 文件并完成数据源初始化耗时 | 是 |
| `get_dataset_ms` | 通过 `weibo` 获取数据集对象耗时 | 是 |
| `count_ms` | 执行 `count()` 并返回 `469308` 的耗时 | 是 |
| `first_page_3_ms` | `limit=3, offset=0` 读取耗时 | 是 |
| `second_page_3_ms` | `limit=3, offset=3` 读取耗时 | 是 |
| `first_page_100_ms` | `limit=100, offset=0` 读取耗时 | 是 |
| `deep_page_100_ms` | `limit=100, offset=100000` 读取耗时 | 是 |
| `ten_pages_100_ms` | 从 `offset=0` 开始连续读取 10 页，每页 100 条的总耗时 | 是 |
| `memory_peak_mb` | 进程峰值内存或可观测近似值 | 否 |
| `decode_features_per_second` | 对分页读取结果折算的要素解码吞吐 | 否 |

不得把单次运行结果作为稳定结论。正式记录时至少执行 5 轮，剔除第 1 轮冷启动结果，记录剩余轮次的最小值、中位数和最大值。

其中 `count_ms` 记录的是各 SDK 当前公开 `count()` API 对物理表实际行数的查询成本。`SmRegister.SmObjectCount` 或其他元数据计数若需保留，必须通过独立 API 或独立指标表达，不得继续与 `count_ms` 混用。

## 运行环境记录

每次提交性能基线结果时，必须记录以下环境信息：

| 项 | 示例 |
|---|---|
| 日期 | `2026-06-23` |
| 操作系统 | `macOS 15.x arm64` |
| CPU | `Apple M-series` |
| 内存 | `32 GB` |
| 磁盘 | 本地 SSD |
| Java | `17.x` |
| Node.js | `24.x` |
| Go | `1.22.x` |
| SQLite 驱动 | Java SQLite JDBC / TypeScript better-sqlite3 / Go go-sqlite3 |
| 样本 SHA-256 | `e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356` |
| Git commit | 各子项目当前 commit |

若样本、运行机器、SQLite 驱动或运行时版本变化，结果只能与同环境结果比较，不应直接作为性能退化结论。

## 执行口径

### 预热规则

- 每个 SDK 至少执行 1 轮预热。
- 正式统计至少执行 5 轮。
- Java 可通过 JMH 做预热；没有 JMH 包装的临时测试也必须显式区分预热与统计。
- TypeScript 和 Go 初期可使用脚本或 benchmark test 输出 JSON/Markdown，后续再固化为正式命令。

### 正确性前置条件

性能基线运行前必须先通过真实样本正确性测试：

```bash
cd udbx4ts
npx vitest run tests/integration/real-samples.integration.spec.ts
```

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test . -run 'RealSample|RealHenan' -v
```

```bash
cd udbx4j
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home \
mvn -Dit.test=HenanDatasetReadTest,TabularDatasetReadTest -Dgroups=integration failsafe:integration-test failsafe:verify
```

如果正确性测试失败，不允许记录性能结果。先修复正确性根因。

## 三端已落地命令

### udbx4j

`udbx4j` 已新增面向真实样本的 JMH benchmark：

```text
udbx4j/src/test/java/com/supermap/udbx/benchmark/RealSampleReadBenchmark.java
```

当前 benchmark case：

- `openHenanDataSource`
- `getWeiboDataset`
- `countWeibo`
- `readFirstPage3`
- `readSecondPage3`
- `readFirstPage100`
- `readDeepPage100`
- `readTenPages100`

当前命令：

```bash
cd udbx4j
make benchmark-real-samples
```

说明：

- `Makefile` 会自动构造 JMH 运行 classpath。
- 默认使用 `-f 0` 运行，便于在受限环境或本地快速验证。
- 正式基线记录建议带 fork 运行，可通过 `JMH_ARGS` 覆盖，例如：

```bash
cd udbx4j
make benchmark-real-samples JMH_ARGS='com.supermap.udbx.benchmark.RealSampleReadBenchmark -f 1'
```

### udbx4ts

`udbx4ts` 已新增 Node 侧基准脚本，避免浏览器和 Worker 变量干扰：

```text
udbx4ts/scripts/benchmark-real-samples.mjs
```

当前命令：

```bash
cd udbx4ts
npm run bench:real-samples
npm run bench:real-samples -- --json-out ../docs/samples/benchmark-artifacts/udbx4ts-real-samples.json
```

脚本输出应包含：

- human-readable Markdown 表格。
- machine-readable JSON，便于 CI artifact 收集和后续趋势比较。
- `--json-out <path>` 可把 machine-readable JSON 写入指定文件。路径相对 `udbx4ts` 当前工作目录解析。

### udbx4go

`udbx4go` 已新增 Go 标准 benchmark：

```text
udbx4go/real_samples_benchmark_test.go
```

当前 benchmark case：

- `BenchmarkRealHenanOpen`
- `BenchmarkRealHenanGetWeiboDataset`
- `BenchmarkRealHenanWeiboCount`
- `BenchmarkRealHenanWeiboFirstPage3`
- `BenchmarkRealHenanWeiboSecondPage3`
- `BenchmarkRealHenanWeiboFirstPage100`
- `BenchmarkRealHenanWeiboDeepPage100`
- `BenchmarkRealHenanWeiboTenPages100`

当前命令：

```bash
cd udbx4go
make benchmark-real-samples
make benchstat-real-samples BASELINE=bench-old.txt CURRENT=bench-new.txt
```

Go benchmark 中应避免把样本打开成本混入分页读取指标。除 `BenchmarkRealHenanOpen` 外，其他 case 应在计时前完成数据源和数据集准备，并使用 `b.ResetTimer()`。

`benchstat-real-samples` 不会自动安装 `benchstat`。如果本机缺少该命令，应显式执行：

```bash
go install golang.org/x/perf/cmd/benchstat@latest
```

## CI 手动触发

三个 SDK 仓库已各自提供手动触发的性能基线 workflow：

| SDK | workflow | 触发方式 | artifact |
|---|---|---|---|
| `udbx4j` | `udbx4j/.github/workflows/performance-baseline.yml` | GitHub Actions `workflow_dispatch` | `udbx4j-performance-baseline` |
| `udbx4ts` | `udbx4ts/.github/workflows/performance-baseline.yml` | GitHub Actions `workflow_dispatch` | `udbx4ts-performance-baseline` |
| `udbx4go` | `udbx4go/.github/workflows/performance-baseline.yml` | GitHub Actions `workflow_dispatch` | `udbx4go-performance-baseline` |

这些 workflow 不绑定 `push` 或 `pull_request`，不会在每次代码变更时自动运行。当前阶段应由维护者在以下场景手动触发：

- 发布前。
- 查询、分页、计数、几何解码、SQLite 访问、系统表读取发生变更后。
- 需要对比某次性能优化效果时。
- 需要生成可归档的 CI 侧性能证据时。

由于三个 SDK 是独立 Git 仓库，而 `data/henan.udbx` 当前位于工作区根目录，CI 手动触发时必须提供 `sample_url`，由 workflow 下载到仓库父目录的 `../data/henan.udbx`。同时必须保留 `sample_sha256` 校验，默认值为当前第一阶段样本 SHA-256：

```text
e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356
```

如果没有提供 `sample_url` 且 CI 环境中不存在 `../data/henan.udbx`，workflow 必须失败，不允许跳过真实样本或改用其他样本继续运行。

CI 结果属于运行证据，不直接覆盖本地基线。维护者应下载 artifact，核对样本校验值、运行时版本和结果波动后，再决定是否更新 [性能基线结果](../samples/performance-baseline-results.md)。

## 结果记录格式

正式结果建议记录在：

```text
docs/samples/performance-baseline-results.md
```

当前首批结果已记录在 [性能基线结果](../samples/performance-baseline-results.md)。后续每次正式基线复跑，都应更新该文档或追加新的日期章节。

建议 Markdown 表格：

| SDK | commit | 指标 | min ms | median ms | max ms | 样本 | 备注 |
|---|---|---:|---:|---:|---:|---|---|
| `udbx4j` | 待填 | `count_ms` | 待填 | 待填 | 待填 | `henan.udbx` | JMH |
| `udbx4ts` | 待填 | `count_ms` | 待填 | 待填 | 待填 | `henan.udbx` | Node |
| `udbx4go` | 待填 | `count_ms` | 待填 | 待填 | 待填 | `henan.udbx` | `go test -bench` |

建议 JSON 结构：

```json
{
  "date": "2026-06-23",
  "sample": {
    "path": "data/henan.udbx",
    "sha256": "e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356",
    "dataset": "weibo",
    "tableName": "henan_P",
    "objectCount": 469308
  },
  "environment": {
    "os": "",
    "cpu": "",
    "memory": "",
    "java": "",
    "node": "",
    "go": ""
  },
  "results": [
    {
      "sdk": "udbx4j",
      "commit": "",
      "metric": "count_ms",
      "runs": [],
      "minMs": 0,
      "medianMs": 0,
      "maxMs": 0
    }
  ]
}
```

## 回归判断

第一阶段只做人工判断，不立即引入 CI 强门禁。满足以下任一条件时，应视为需要调查：

- 同环境、同样本、同指标的中位数退化超过 20%。
- `count()` 或小分页读取从毫秒级退化到明显秒级。
- 深分页读取退化超过 30%，且不是 SQLite 查询计划、磁盘状态或运行环境变化导致。
- 峰值内存随分页大小之外的因素线性增长，疑似一次性加载全表。
- 正确性测试通过但性能测试出现超时、内存异常或资源未释放。

性能退化处理顺序：

1. 复跑确认是否稳定复现。
2. 确认样本 SHA-256、运行机器、运行时版本和 SQLite 驱动是否变化。
3. 用最小指标定位退化发生在打开、数据集获取、计数、分页查询还是几何解码。
4. 补回归 benchmark 或真实样本测试。
5. 修复根因后更新性能结果记录。

## 发布门禁关系

短期内性能基线作为发布前检查项，不作为阻断所有 PR 的硬门禁。

发布前至少应确认：

- 真实样本正确性测试通过。
- `weibo -> henan_P` 的 `count()`、第一页分页、深分页没有明显退化。
- 若本版本改动涉及查询、分页、几何解码、SQLite 访问、系统表读取，必须更新或复跑性能基线。

当三端 benchmark 命令稳定后，可逐步引入 CI 的非阻断性能报告；只有在结果波动范围明确后，才考虑强制门禁。

## 后续扩展

下一阶段可扩展：

- `SampleData.udbx` 的 CAD / Text / 3D 解码基线。
- 写入基线：创建数据源、批量写入、更新、删除。
- 跨语言 roundtrip 后的读取基线。
- 浏览器 Worker 运行时读取基线。
- CLI / viewer 工具层端到端基线。

扩展新指标前，必须先确认该指标对应真实用户场景，并在本文档中补充定义。
