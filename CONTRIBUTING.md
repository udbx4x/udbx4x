# 贡献指南

感谢参与 UDBX4X。

本工作区遵循一条基本规则：规范优先，SDK 其次，工具第三。

## 开始之前

请先阅读：

1. [AGENTS.md](./AGENTS.md)
2. [GOVERNANCE.md](./GOVERNANCE.md)
3. 目标子项目的 `AGENTS.md`
4. 目标子项目的 `README.md`

## 贡献范围

请先确认修改属于哪个边界：

- `udbx4spec`：跨语言概念、API 模型、夹具、合规规则。
- `udbx4j`：Java SDK 实现。
- `udbx4ts`：TypeScript SDK 实现。
- `udbx4go`：Go SDK 实现。
- `udbx-copilot`：基于 SDK 的 CLI 工具。

当能力属于 SDK 时，不要在工具中实现格式解析。

## 开发规则

- 公开 API 变更先修改 `udbx4spec`。
- 格式行为变更先修改 `udbx4spec`。
- 修复 bug 时应尽量增加回归测试。
- 新增 SDK 能力必须包含测试。
- 跨语言行为变更必须更新合规资产。
- 避免无关重构。
- 避免为已知错误概念增加兼容层。
- 不把样本文件中的偶然行为硬编码为通用规则。

## 本地检查

根据修改的子项目运行对应检查。

Java：

```bash
cd udbx4j
make test
```

TypeScript：

```bash
cd udbx4ts
npm run typecheck
npm test
npm run build
```

Go：

```bash
cd udbx4go
make test
```

Copilot：

```bash
cd udbx-copilot
npm run typecheck
npm test
```

## Pull Request 检查清单

- 变更理由清晰。
- 修改范围限定在相关项目内。
- 需要规范变更时，已先修改规范。
- 已增加测试，或明确说明无法补测的原因。
- 用户可见行为已更新文档。
- 包可见行为已更新 CHANGELOG。
- 未提交无关生成产物，除非它是必要夹具或发布资产。

## Commit Message

建议使用简洁的 conventional-style 前缀：

- `feat:`
- `fix:`
- `test:`
- `docs:`
- `refactor:`
- `chore:`

示例：

```bash
git commit -m "feat: add regionZ GAIA roundtrip fixture"
```

## 提交 Issue

提交 bug 时请包含：

- SDK 或工具名称。
- 版本或 commit。
- 操作系统。
- 最小复现步骤。
- 样本文件来源，若可以提供。
- 期望行为。
- 实际行为。

不要在公开 issue 中附加私有或不可再分发的 `.udbx` 文件。

