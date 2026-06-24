# 操作指南

本文档是 `udbx4x` 工作区的通用操作指南入口。

## 适用范围

本目录用于存放跨项目、跨语言的操作流程说明，例如：

- 本地开发与联调。
- 合规测试。
- 性能基线。
- 样本文件管理。
- 发布流程。
- 工具使用入口。

语言或子项目特有的操作说明，应优先放在对应子项目目录下；仅当流程对整个工作区普遍适用时，才放在本目录。

## 目录

- [本地开发与检查](./local-development.md)
- [合规测试指南](./compliance-testing.md)
- [性能基线方案](./performance-baseline.md)
- [样本文件管理](./sample-management.md)
- [真实来源夹具准入](./source-fixture-admission.md)
- [发布工作流](./release-workflow.md)

## 相关文档

- [样本文件说明](../samples/README.md)
- [真实样本兼容矩阵](../samples/compatibility-matrix.md)
- [性能基线结果](../samples/performance-baseline-results.md)

## 使用原则

- 先读根目录 `AGENTS.md`，再按本目录中的流程执行。
- 当指南与子项目 README、发布脚本或规范冲突时，先确认根目录文档和 `udbx4spec` 是否需要更新。
- 操作指南必须包含输入、命令、预期结果和失败时的下一步动作。
