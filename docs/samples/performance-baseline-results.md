# 性能基线结果

本文档记录真实样本性能基线的实际运行结果。执行口径见 [性能基线方案](../guides/performance-baseline.md)。

当前结果是第一批本地基线，用于建立趋势观察起点，不作为跨机器绝对性能承诺。

## 运行环境

| 项 | 值 |
|---|---|
| 日期 | `2026-06-23` |
| 操作系统 | `Darwin 24.6.0 arm64` |
| CPU | `Apple M3 Pro` |
| 内存 | `36 GB` |
| 样本 | `data/henan.udbx` |
| 样本 SHA-256 | `e5f75d3b0d0334c778a2145f49836271664bd62cf354394fedb47c0ee7665356` |
| 数据集 | `weibo` |
| 物理表 | `henan_P` |
| 对象数 | `469308` |

运行时版本：

| SDK | commit | 运行时 | 命令 |
|---|---|---|---|
| `udbx4j` | `cae9277 + working tree` | Zulu JDK `17.0.11` | `make benchmark-real-samples JMH_ARGS='com.supermap.udbx.benchmark.RealSampleReadBenchmark -f 1'` |
| `udbx4ts` | `cae459f` | Node.js `v25.8.0`（脚本输出环境） | `npm run bench:real-samples` |
| `udbx4go` | `3496e8d + working tree` | Go `1.24.1` | `GOCACHE=/private/tmp/udbx4go-gocache go test . -run '^$' -bench 'BenchmarkRealHenan...' -benchmem -count 5` |

## 结果摘要

单位为毫秒。Java 使用 JMH `AverageTime`，本次以 fork 模式 `-f 1` 运行；表中 `median ms` 对 Java 记录为 JMH average。Go 结果由 `ns/op` 换算为毫秒。TypeScript 结果由脚本直接统计 5 轮 measured run 的最小值、中位数和最大值。

| SDK | 指标 | min ms | median ms | max ms | 备注 |
|---|---|---:|---:|---:|---|
| `udbx4j` | `open_data_source_ms` | 0.037 | 0.039 | 0.042 | JMH fork |
| `udbx4j` | `get_dataset_ms` | 0.007 | 0.008 | 0.008 | JMH fork |
| `udbx4j` | `count_ms` | 3.044 | 3.120 | 3.224 | JMH fork |
| `udbx4j` | `first_page_3_ms` | 0.039 | 0.041 | 0.042 | JMH fork |
| `udbx4j` | `second_page_3_ms` | 0.037 | 0.038 | 0.039 | JMH fork |
| `udbx4j` | `first_page_100_ms` | 0.088 | 0.091 | 0.097 | JMH fork |
| `udbx4j` | `deep_page_100_ms` | 0.999 | 1.081 | 1.198 | JMH fork |
| `udbx4j` | `ten_pages_100_ms` | 0.868 | 0.883 | 0.902 | JMH fork |
| `udbx4ts` | `open_data_source_ms` | 0.113 | 0.120 | 0.126 | Node benchmark |
| `udbx4ts` | `get_dataset_ms` | 1.383 | 1.522 | 1.816 | Node benchmark |
| `udbx4ts` | `count_ms` | 3.677 | 4.280 | 5.384 | Node benchmark |
| `udbx4ts` | `first_page_3_ms` | 0.072 | 0.094 | 0.117 | Node benchmark |
| `udbx4ts` | `second_page_3_ms` | 0.025 | 0.030 | 0.032 | Node benchmark |
| `udbx4ts` | `first_page_100_ms` | 0.232 | 0.240 | 0.831 | Node benchmark |
| `udbx4ts` | `deep_page_100_ms` | 1.612 | 1.784 | 2.321 | Node benchmark |
| `udbx4ts` | `ten_pages_100_ms` | 2.582 | 2.793 | 3.389 | Node benchmark |
| `udbx4go` | `open_data_source_ms` | 1.707483 | 1.930383 | 2.001908 | `go test -bench` |
| `udbx4go` | `get_dataset_ms` | 0.019660 | 0.020068 | 0.020468 | `go test -bench` |
| `udbx4go` | `count_ms` | 3.000050 | 3.067229 | 3.513158 | 已统一为物理表 `COUNT(*)` |
| `udbx4go` | `first_page_3_ms` | 0.008191 | 0.008369 | 0.008714 | `go test -bench` |
| `udbx4go` | `second_page_3_ms` | 0.008475 | 0.008723 | 0.009283 | `go test -bench` |
| `udbx4go` | `first_page_100_ms` | 0.101781 | 0.107374 | 0.111190 | `go test -bench` |
| `udbx4go` | `deep_page_100_ms` | 1.074619 | 1.148280 | 1.269838 | `go test -bench` |
| `udbx4go` | `ten_pages_100_ms` | 1.106522 | 1.135413 | 1.432221 | `go test -bench` |

## 解释约束

- 本结果只能与相同机器、相同样本、相同命令、相同运行时版本下的后续结果比较。
- `count_ms` 已统一为各 SDK 当前公开 `count()` API 对物理表实际行数的查询成本。
- Java 本次使用 JMH fork 模式 `-f 1`，可作为当前正式本地基线。
- TypeScript 当前使用 Node 侧驱动，不代表浏览器 Worker 运行时性能。
- Go 结果包含 `B/op` 和 `allocs/op`，后续优化分页内存分配时应继续跟踪。

## 下一步

- 手动触发三端 `Performance Baseline` workflow，确认 CI artifact 能稳定产出。
- 安装 `benchstat` 后，用 `make benchstat-real-samples BASELINE=... CURRENT=...` 建立 Go 性能回归对比记录。
- 后续如需保留快速元数据计数，应新增独立指标，避免与 `count_ms` 混用。
