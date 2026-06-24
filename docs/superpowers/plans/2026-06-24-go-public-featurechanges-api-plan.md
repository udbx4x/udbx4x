# Go FeatureChanges 公开 API 缺口修复实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 修复 `udbx4go` 矢量数据集 `Update(id, changes)` 的公开参数类型缺口，让外部用户可以通过根包 `udbx4go.FeatureChanges` 调用更新 API。

**Architecture:** 保留当前内部数据集实现和方法签名，通过根包类型别名将 `internal/dataset.FeatureChanges` 暴露为 `udbx4go.FeatureChanges`。用外部包测试验证真实用户视角可编译、可调用、可更新。

**Tech Stack:** Go 1.21+、go-sqlite3、testify、Markdown。

---

## 任务

### Task 1: 用外部包测试锁定缺口

**Files:**

- Create: `udbx4go/public_api_test.go`

- [x] **Step 1: 新增 `udbx4go_test` 包测试**

测试从根包导入 `github.com/udbx4x/udbx4go`，创建 Point 数据集，使用 `&udbx4go.FeatureChanges{...}` 调用 `Update`。

- [x] **Step 2: 验证红灯**

命令：

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./ -run TestPublicFeatureChangesCanUpdatePointDataset
```

结果：编译失败，`undefined: udbx4go.FeatureChanges`。

### Task 2: 暴露公开类型

**Files:**

- Modify: `udbx4go/udbx.go`

- [x] **Step 1: 根包导入内部 dataset 包**
- [x] **Step 2: 增加类型别名**

```go
type FeatureChanges = dataset.FeatureChanges
```

- [x] **Step 3: 验证绿灯**

命令：

```bash
GOCACHE=/private/tmp/udbx4go-gocache go test ./ -run TestPublicFeatureChangesCanUpdatePointDataset
```

结果：通过。

### Task 3: 更新文档

**Files:**

- Modify: `udbx4go/README.md`
- Modify: `udbx4go/README.zh.md`
- Modify: `udbx4go/API.md`
- Modify: `udbx4go/API.zh.md`

- [x] **Step 1: 删除 FeatureChanges 公开 API 缺口说明**
- [x] **Step 2: 增加 `udbx4go.FeatureChanges` 更新示例**
- [x] **Step 3: 说明 `FeatureChanges` 是公开变更对象**

### Task 4: 验收

**Commands:**

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
GOCACHE=/private/tmp/udbx4go-gocache make test-stable-t3
rg -n "Public API gap|公开 API 缺口|not currently re-exported|尚未重导出|Do not use|不应使用" udbx4go docs README.md udbx4spec -S
```

- [x] **Step 1: 全量 Go 测试**

结果：`go test ./...` 通过。

- [x] **Step 2: stable T3 门禁**

结果：`make test-stable-t3` 通过。

- [x] **Step 3: 旧缺口文本扫描**

结果：旧缺口文本清零。
