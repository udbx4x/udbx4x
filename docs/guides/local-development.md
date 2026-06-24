# 本地开发与检查

本文档说明 `udbx4x` 工作区的本地开发基本流程。

## 目标

- 快速进入正确的子项目。
- 在本地完成最小可验证修改。
- 避免跨项目无边界修改。

## 开始前

先阅读：

1. 根目录 `AGENTS.md`
2. 目标子项目的 `AGENTS.md`
3. 目标子项目的 `README.md`

如果改动涉及公开 API、数据模型、格式行为、`DatasetKind` 或 `FieldType`，先检查 `udbx4spec`。

## 常用工作流

### 只改 `udbx4spec`

适用场景：

- 新增概念。
- 修改公开 API 语义。
- 修正格式解释。
- 补充合规规则。

建议步骤：

```bash
cd udbx4spec
```

完成后应同步检查：

- 是否需要新增合规夹具。
- 是否需要更新 SDK 文档。
- 是否需要更新能力矩阵或术语表。

### 只改单个 SDK

#### `udbx4j`

```bash
cd udbx4j
make test
```

必要时：

```bash
mvn verify
```

#### `udbx4ts`

```bash
cd udbx4ts
npm install
npm run typecheck
npm test
npm run build
```

浏览器运行时变更还需要：

```bash
npm run test:browser
```

#### `udbx4go`

```bash
cd udbx4go
make test
go test ./...
```

#### `udbx-copilot`

```bash
cd udbx-copilot
npm install
npm run typecheck
npm test
```

### 跨项目改动

适用场景：

- 规范变更影响多个 SDK。
- 跨语言一致性修复。
- 发布流程更新。

建议顺序：

1. 先改 `udbx4spec`
2. 再改目标 SDK
3. 再改工具层
4. 最后更新文档和能力矩阵

## 提交前最小检查

至少确认：

- 修改边界清晰。
- 对应子项目本地测试通过。
- 用户可见变更已更新 README 或 CHANGELOG。
- 若改动了概念或格式行为，已更新 `udbx4spec`。

## 常见错误

- 直接在工具层实现格式解析。
- 先改 SDK、后补规范。
- 只为样本通过而硬编码。
- 修改公开 API 但不更新文档。

