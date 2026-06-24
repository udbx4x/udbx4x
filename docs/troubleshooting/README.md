# 常见故障排查

本文档是 `udbx4x` 工作区排障文档入口。

## 适用范围

本目录用于记录跨项目、跨语言的常见故障和排查方法。

典型问题包括：

- 构建失败
- 测试失败
- 样本文件路径错误
- 合规测试不一致
- 版本号或发布产物不一致
- SDK 与工具行为不一致

## 排障原则

- 先确认问题属于规范、SDK、工具还是环境。
- 先复现，再修复。
- 先定位根因，再动代码。
- 修复后补回归测试或补文档。

## 建议排查顺序

1. 根目录 `AGENTS.md`
2. 目标子项目 `AGENTS.md`
3. `docs/guides/` 对应操作指南
4. `udbx4spec` 规范文档
5. 子项目测试输出
6. 合规报告

## 建议后续新增的排障文档

- `docs/troubleshooting/java-build.md`
- `docs/troubleshooting/ts-browser-runtime.md`
- `docs/troubleshooting/go-cgo.md`
- `docs/troubleshooting/compliance-drift.md`
- `docs/troubleshooting/sample-file-issues.md`

## 单篇排障文档格式

每篇排障文档应包含：

- 现象
- 影响范围
- 根因
- 诊断命令
- 修复步骤
- 预防措施

