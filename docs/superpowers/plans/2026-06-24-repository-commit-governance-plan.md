# 代码提交治理 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将当前 `udbx4x` 工作区内已经形成的规范、SDK、文档、测试和样本治理成果，按仓库边界和语义边界安全提交到代码库，并建立后续持续提交流程。

**Architecture:** 当前工作区是多仓库结构，根目录负责跨项目治理但尚未初始化为 Git 仓库，`udbx4spec`、`udbx4j`、`udbx4ts`、`udbx4go` 是独立发布仓库。提交策略采用“先规范、后 SDK、再工具、最后根治理仓库”的依赖顺序，每个子仓库按可验证语义拆成小提交。

**Tech Stack:** Git、Maven、Java 17、JUnit、JMH、npm、TypeScript、Vitest、Playwright、Go、go test、Make、Markdown。

---

## 当前仓库事实

- `/Users/zhangyuting/github/udbx4x` 当前不是 Git 仓库。
- `/Users/zhangyuting/github/udbx4x/udbx4spec` 是独立 Git 仓库，远端为 `https://github.com/udbx4x/udbx4spec.git`。
- `/Users/zhangyuting/github/udbx4x/udbx4j` 是独立 Git 仓库，远端为 `https://github.com/udbx4x/udbx4j.git`。
- `/Users/zhangyuting/github/udbx4x/udbx4ts` 是独立 Git 仓库，存在 `origin=https://github.com/zhyt1985/udbx4ts` 和 `udbx4x=https://github.com/udbx4x/udbx4ts.git` 两个远端。
- `/Users/zhangyuting/github/udbx4x/udbx4go` 是独立 Git 仓库，远端为 `https://github.com/udbx4x/udbx4go.git`。
- `/Users/zhangyuting/github/udbx4x/udbx-copilot` 当前不是 Git 仓库。
- 根目录的 `AGENTS.md`、`README.md`、`ROADMAP.md`、`GOVERNANCE.md`、`CONTRIBUTING.md`、`RELEASE.md`、`SECURITY.md`、`docs/`、`data/` 尚无 Git 归属。

## 代码库设计

### 推荐结构

```text
udbx4x
├── AGENTS.md
├── README.md
├── ROADMAP.md
├── GOVERNANCE.md
├── CONTRIBUTING.md
├── RELEASE.md
├── SECURITY.md
├── docs/
├── data/
├── udbx4spec/
├── udbx4j/
├── udbx4ts/
├── udbx4go/
└── udbx-copilot/
```

### 仓库职责

- `udbx4x` 根仓库：项目总入口、治理规则、路线图、跨语言计划、样本授权、跨项目架构、统一工作原则。
- `udbx4spec`：UDBX 规范、跨语言 API 稳定面、合规夹具、golden bytes、roundtrip 矩阵、参考类型定义。
- `udbx4j`：Java SDK，独立 Maven 发布。
- `udbx4ts`：TypeScript SDK，独立 npm 发布。
- `udbx4go`：Go SDK，独立 Go module 与 Git tag 发布。
- `udbx-copilot`：工具生态仓库，后续单独初始化或并入工具集合仓库。

### 当前不迁移为 monorepo 的原因

- Maven、npm、Go module 的发布机制不同，独立仓库更符合生态习惯。
- `udbx4spec`、`udbx4j`、`udbx4ts`、`udbx4go` 已有独立历史和远端。
- Go module 位于子目录时，版本 tag、module path、pkg.go.dev 收录都会复杂化。
- 当前最重要目标是保存成果并恢复可控提交节奏，不应在提交治理阶段引入大规模仓库迁移。

## 提交总原则

- 不从根目录尝试一次性提交所有内容。
- 不使用 `git add .`。
- 不使用 `git reset --hard`、`git checkout --`、`git clean` 删除或回滚现有成果。
- 每次提交前必须先看 `git diff --stat` 和精确文件 diff。
- 每次提交只表达一个语义：规范、实现、测试、文档、治理、性能基线、CI 不能随意混合。
- 跨语言语义变更先提交 `udbx4spec`，再提交各 SDK。
- 已验证的小闭环优先提交，降低未提交成果继续堆积的风险。
- 涉及样本文件时必须确认授权、来源、生成工具、校验方式和是否可公开。
- 提交信息使用英文 Conventional Commits，提交说明和审查记录使用简体中文。

## 推荐提交顺序

1. `udbx4go` 小闭环提交：公开 `FeatureChanges` API 缺口修复。
2. `udbx4spec` 规范和夹具提交：稳定 API、T3 夹具、GeoText、golden bytes、roundtrip 矩阵。
3. `udbx4j` Java SDK 提交：规范对齐、Text/CAD/3D、T3 合规、性能基线文档。
4. `udbx4ts` TypeScript SDK 提交：规范对齐、Text/CAD/3D、T3 合规、浏览器与 Node 验证。
5. `udbx4go` Go SDK 大闭环提交：count 语义、Text/CAD、T3 合规、性能基线。
6. `udbx4x` 根治理仓库提交：根文档、跨项目计划、样本治理、架构文档。
7. `udbx-copilot` 仓库归属决策：初始化独立仓库或暂时保持工作区工具目录。

## 文件结构与责任

### 根目录

- `AGENTS.md`：总工作规则和长期协作入口。
- `README.md`：项目对外入口。
- `ROADMAP.md`：路线图。
- `GOVERNANCE.md`：治理与权限。
- `CONTRIBUTING.md`：贡献流程。
- `RELEASE.md`：发布流程。
- `SECURITY.md`：安全策略。
- `docs/architecture/`：跨项目架构、能力矩阵、风险登记、工具待办。
- `docs/concepts/`：术语表和项目级概念。
- `docs/guides/`：本地开发、合规测试、样本管理、发布、性能基线。
- `docs/samples/`：样本授权、兼容性矩阵、性能基线结果。
- `docs/standards/`：开发、API、环境、国际化、模块、部署、安全规范。
- `docs/superpowers/plans/`：实施计划。

### `udbx4spec`

- `docs/`：跨语言规范。
- `reference/java/`：Java 参考接口。
- `reference/typescript/`：TypeScript 参考定义。
- `reference/json-schema/`：JSON Schema。
- `compliance/`：合规夹具、golden bytes、roundtrip、source-derived 样本说明。
- `tools/`：规范夹具相关工具。

### `udbx4j`

- `src/main/java/com/supermap/udbx/`：Java SDK 实现。
- `src/test/java/com/supermap/udbx/spec/`：规范级测试。
- `src/test/java/com/supermap/udbx/integration/`：样本和集成测试。
- `src/test/java/com/supermap/udbx/benchmark/`：JMH 性能基线。
- `docs/`：Java SDK 私有文档。

### `udbx4ts`

- `src/core/`：平台无关核心实现。
- `src/runtime-browser/`：浏览器运行时。
- `src/runtime-electron/`：Electron 运行时。
- `tests/unit/`：单元测试。
- `tests/integration/`：集成和合规测试。
- `scripts/`：测试、性能和工具脚本。

### `udbx4go`

- `udbx.go`、`datasource.go`：Go SDK 公开入口。
- `internal/codec/`：GAIA、CAD、GeoText 编解码。
- `internal/dataset/`：数据集实现。
- `pkg/types/`：公开类型。
- `cmd/`：示例、查看器、夹具生成工具。
- `*_test.go`：公开 API、真实样本、合规和性能测试。

## Task 1: 提交前冻结状态快照

**Files:**
- Read: `/Users/zhangyuting/github/udbx4x`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4spec`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4j`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4ts`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4go`

- [ ] **Step 1: 记录根目录和子仓库 Git 归属**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
pwd
find . -maxdepth 3 -name .git -type d -print | sort
```

Expected:

```text
/Users/zhangyuting/github/udbx4x
./udbx4go/.git
./udbx4j/.git
./udbx4spec/.git
./udbx4ts/.git
```

- [ ] **Step 2: 记录每个子仓库分支和远端**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  printf '\n[%s]\n' "$d"
  git -C "$d" branch --show-current
  git -C "$d" remote -v
done
```

Expected:

```text
[udbx4spec]
main
origin  https://github.com/udbx4x/udbx4spec.git (fetch)
origin  https://github.com/udbx4x/udbx4spec.git (push)

[udbx4j]
main
origin  https://github.com/udbx4x/udbx4j.git (fetch)
origin  https://github.com/udbx4x/udbx4j.git (push)

[udbx4ts]
gitbutler/workspace
origin  https://github.com/zhyt1985/udbx4ts (fetch)
origin  https://github.com/zhyt1985/udbx4ts (push)
udbx4x    https://github.com/udbx4x/udbx4ts.git (fetch)
udbx4x    https://github.com/udbx4x/udbx4ts.git (push)

[udbx4go]
gitbutler/workspace
origin  https://github.com/udbx4x/udbx4go.git (fetch)
origin  https://github.com/udbx4x/udbx4go.git (push)
```

- [ ] **Step 3: 记录完整工作区变更摘要**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  printf '\n[%s]\n' "$d"
  git -C "$d" status --short
done
```

Expected: 输出四个子仓库的修改、删除和新增文件清单，命令退出码为 `0`。

- [ ] **Step 4: 建立人工审查基线文件**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
mkdir -p /private/tmp/udbx4x-commit-snapshots
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  git -C "$d" status --short > "/private/tmp/udbx4x-commit-snapshots/$d.status.txt"
  git -C "$d" diff --stat > "/private/tmp/udbx4x-commit-snapshots/$d.diffstat.txt"
done
```

Expected: `/private/tmp/udbx4x-commit-snapshots/` 下生成 8 个文本文件，命令退出码为 `0`。

- [ ] **Step 5: 本任务不提交**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git -C udbx4spec status --short | sed -n '1,5p'
git -C udbx4j status --short | sed -n '1,5p'
git -C udbx4ts status --short | sed -n '1,5p'
git -C udbx4go status --short | sed -n '1,5p'
```

Expected: 仍然能看到未提交变更；本任务只做快照，不改变 Git 状态。

## Task 2: 优先提交 `udbx4go` FeatureChanges 小闭环

**Files:**
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/udbx.go`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4go/public_api_test.go`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/API.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/API.zh.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/README.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/README.zh.md`

- [ ] **Step 1: 查看本提交候选文件 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git diff -- udbx.go public_api_test.go API.md API.zh.md README.md README.zh.md
```

Expected: diff 只包含公开 `FeatureChanges` 类型、公开 API 测试、README/API 文档对齐内容。

- [ ] **Step 2: 运行公开 API 定向测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./ -run TestPublicFeatureChangesCanUpdatePointDataset
```

Expected:

```text
ok  	github.com/udbx4x/udbx4go	...
```

- [ ] **Step 3: 运行 Go 全量测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
```

Expected: 所有包测试通过，命令退出码为 `0`。

- [ ] **Step 4: 运行 stable T3 合规测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache make test-stable-t3
```

Expected: stable T3 相关测试通过，命令退出码为 `0`。

- [ ] **Step 5: 精确暂存候选文件**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git add udbx.go public_api_test.go API.md API.zh.md README.md README.zh.md
git diff --cached --stat
```

Expected: 暂存区只包含 6 个候选文件。

- [ ] **Step 6: 提交小闭环**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git commit -m "fix: expose FeatureChanges in public API"
```

Expected: 生成一个新提交，提交信息为 `fix: expose FeatureChanges in public API`。

- [ ] **Step 7: 确认剩余变更未被误提交**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git status --short
```

Expected: `datasource.go`、`internal/codec/`、`internal/dataset/`、性能测试、合规测试、viewer 文档删除等其他变更仍在工作区。

## Task 3: 提交 `udbx4spec` 规范和合规资产

**Files:**
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4spec/README.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/01-naming-conventions.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/02-geometry-model.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/03-dataset-taxonomy.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/06-language-mapping.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/07-geotext-binary-layout.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4spec/docs/08-api-stable-surface.md`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4spec/compliance/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4spec/reference/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4spec/tools/`

- [ ] **Step 1: 查看规范文档 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git diff -- README.md docs/01-naming-conventions.md docs/02-geometry-model.md docs/03-dataset-taxonomy.md docs/06-language-mapping.md docs/07-geotext-binary-layout.md docs/08-api-stable-surface.md
```

Expected: diff 聚焦命名、几何模型、数据集分类、语言映射、GeoText 布局、公开 API 稳定面。

- [ ] **Step 2: 暂存并提交规范文档**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git add README.md docs/01-naming-conventions.md docs/02-geometry-model.md docs/03-dataset-taxonomy.md docs/06-language-mapping.md docs/07-geotext-binary-layout.md docs/08-api-stable-surface.md
git diff --cached --stat
git commit -m "docs: define geotext and stable API surface"
```

Expected: 生成一个规范文档提交。

- [ ] **Step 3: 查看合规夹具资产 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git status --short compliance
git diff -- compliance/README.md compliance/golden-gaia-bytes/README.md compliance/java-compliance-checklist.md compliance/fixture-standard.md compliance/roundtrip-matrix.md
```

Expected: 输出合规夹具说明、golden bytes、T3 准入、roundtrip 矩阵等变更。

- [ ] **Step 4: 暂存并提交合规夹具资产**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git add compliance/README.md compliance/java-compliance-checklist.md compliance/fixture-standard.md compliance/roundtrip-matrix.md compliance/compliance.udbx compliance/fixtures compliance/golden-gaia-bytes compliance/golden-text-bytes compliance/roundtrip compliance/source-derived
git diff --cached --stat
git commit -m "test: add stable compliance fixtures"
```

Expected: 生成一个合规夹具提交，暂存区包含 `.udbx` 夹具、manifest、golden bytes、roundtrip 资产和说明文档。

- [ ] **Step 5: 查看参考定义 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git status --short reference
git diff -- reference/java reference/json-schema reference/typescript
```

Expected: 输出 Java 参考接口、JSON Schema、TypeScript 定义对公开 API、GeoText、TextFeature 的对齐变更。

- [ ] **Step 6: 暂存并提交参考定义**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git add reference/java reference/json-schema reference/typescript
git diff --cached --stat
git commit -m "refactor: align reference APIs with stable surface"
```

Expected: 生成一个参考定义提交。

- [ ] **Step 7: 查看并提交工具目录**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git status --short tools
git add tools
git diff --cached --stat
git commit -m "chore: add compliance fixture tools"
```

Expected: 生成一个工具提交；如果 `tools/` 无可提交文件，`git diff --cached --stat` 为空，此步骤停止并不创建空提交。

- [ ] **Step 8: 确认 `udbx4spec` 剩余状态**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4spec
git status --short
```

Expected: 只剩 `.DS_Store` 等不应提交的忽略文件，或输出为空。

## Task 4: 提交 `udbx4j` Java SDK 成果

**Files:**
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4j/src/main/java/com/supermap/udbx/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4j/src/test/java/com/supermap/udbx/`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4j/README.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4j/CHANGELOG.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4j/CONTRIBUTING.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4j/Makefile`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4j/AGENTS.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4j/CLAUDE.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4j/docs/performance-summary.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4j/.github/workflows/`
- Delete: `/Users/zhangyuting/github/udbx4x/udbx4j/docs/juejin-announcement-v1.md`
- Delete: `/Users/zhangyuting/github/udbx4x/udbx4j/docs/v2ex-announcement-v1.md`

- [ ] **Step 1: 运行 Java 核心测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
make test
```

Expected: Maven/JUnit 测试通过，命令退出码为 `0`。

- [ ] **Step 2: 运行 Java stable T3 合规测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
make test-stable-t3
```

Expected: stable T3 合规测试通过，命令退出码为 `0`。

- [ ] **Step 3: 查看 SDK 实现 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git diff -- src/main/java/com/supermap/udbx
```

Expected: diff 聚焦 Dataset 语义、Text/CAD/3D、系统表读取、公开 API 行为。

- [ ] **Step 4: 暂存并提交 SDK 实现**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git add src/main/java/com/supermap/udbx
git diff --cached --stat
git commit -m "feat: align SDK implementation with stable compliance surface"
```

Expected: 生成 Java SDK 实现提交。

- [ ] **Step 5: 查看测试 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git status --short src/test/java/com/supermap/udbx
git diff -- src/test/java/com/supermap/udbx
```

Expected: diff 聚焦规范测试、合规数据库读写、真实样本读取、JMH 基线。

- [ ] **Step 6: 暂存并提交测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git add src/test/java/com/supermap/udbx
git diff --cached --stat
git commit -m "test: add stable compliance and real sample coverage"
```

Expected: 生成 Java 测试提交。

- [ ] **Step 7: 查看文档和治理 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git diff -- README.md CHANGELOG.md CONTRIBUTING.md Makefile AGENTS.md CLAUDE.md docs/performance-summary.md
git status --short .github docs
```

Expected: diff 聚焦文档入口、Agent 规则、性能摘要、CI 配置和无用宣传文档删除。

- [ ] **Step 8: 暂存并提交文档和治理**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
git add README.md CHANGELOG.md CONTRIBUTING.md Makefile AGENTS.md CLAUDE.md docs/performance-summary.md .github/workflows docs/juejin-announcement-v1.md docs/v2ex-announcement-v1.md
git diff --cached --stat
git commit -m "docs: align Java project governance and performance notes"
```

Expected: 生成 Java 文档治理提交，两个已删除宣传文档进入提交。

- [ ] **Step 9: 提交后复跑 Java 测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4j
make test
make test-stable-t3
```

Expected: 两个命令均退出码为 `0`。

## Task 5: 提交 `udbx4ts` TypeScript SDK 成果

**Files:**
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4ts/src/core/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4ts/src/runtime-browser/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4ts/tests/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4ts/scripts/`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/package.json`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/README.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/README.en.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/CHANGELOG.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/AGENTS.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4ts/CLAUDE.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4ts/.github/workflows/performance-baseline.yml`

- [ ] **Step 1: 运行 TypeScript 类型检查**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
npm run typecheck
```

Expected: TypeScript 类型检查通过，命令退出码为 `0`。

- [ ] **Step 2: 运行 TypeScript 单元测试和集成测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
npm test
```

Expected: Vitest 测试通过，命令退出码为 `0`。

- [ ] **Step 3: 运行浏览器测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
npm run test:browser
```

Expected: 浏览器运行时测试通过，命令退出码为 `0`。

- [ ] **Step 4: 查看核心实现 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git diff -- src/core src/index.ts
```

Expected: diff 聚焦核心 Dataset、DataSource、GAIA、CAD、GeoText、公开导出和二进制工具。

- [ ] **Step 5: 暂存并提交核心实现**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git add src/core src/index.ts
git diff --cached --stat
git commit -m "feat: align core SDK with stable compliance surface"
```

Expected: 生成 TypeScript 核心实现提交。

- [ ] **Step 6: 查看浏览器运行时 diff**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git diff -- src/runtime-browser
```

Expected: diff 聚焦浏览器 DatasetClient、UdbxClient、Worker runtime 对公开 API 的对齐。

- [ ] **Step 7: 暂存并提交浏览器运行时**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git add src/runtime-browser
git diff --cached --stat
git commit -m "feat: align browser runtime with stable dataset API"
```

Expected: 生成浏览器运行时提交。

- [ ] **Step 8: 查看并提交测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git status --short tests
git diff -- tests
git add tests
git diff --cached --stat
git commit -m "test: add stable compliance and real sample coverage"
```

Expected: 生成 TypeScript 测试提交。

- [ ] **Step 9: 查看并提交脚本和性能 CI 配置**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git status --short scripts .github
git diff -- scripts .github
git add scripts .github/workflows/performance-baseline.yml
git diff --cached --stat
git commit -m "chore: add TypeScript performance baseline tooling"
```

Expected: 生成脚本和性能基线配置提交。

- [ ] **Step 10: 查看并提交文档治理**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git diff -- README.md README.en.md CHANGELOG.md AGENTS.md CLAUDE.md package.json udbx4spec-compliance-report.md
git add README.md README.en.md CHANGELOG.md AGENTS.md CLAUDE.md package.json udbx4spec-compliance-report.md
git diff --cached --stat
git commit -m "docs: align TypeScript public API and compliance docs"
```

Expected: 生成 TypeScript 文档治理提交。

- [ ] **Step 11: 提交后复跑 TypeScript 验证**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
npm run typecheck
npm test
npm run test:browser
```

Expected: 三个命令均退出码为 `0`。

## Task 6: 提交 `udbx4go` 大闭环成果

**Files:**
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/datasource.go`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/internal/codec/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/internal/dataset/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/pkg/types/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/cmd/`
- Modify/Create: `/Users/zhangyuting/github/udbx4x/udbx4go/*_test.go`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/Makefile`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/CHANGELOG.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/cmd/udbx4go-viewer/README.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4go/AGENTS.md`
- Modify: `/Users/zhangyuting/github/udbx4x/udbx4go/CLAUDE.md`
- Create: `/Users/zhangyuting/github/udbx4x/udbx4go/.github/workflows/performance-baseline.yml`
- Delete: `/Users/zhangyuting/github/udbx4x/udbx4go/viewer/IMPLEMENTATION_PLAN.md`
- Delete: `/Users/zhangyuting/github/udbx4x/udbx4go/viewer/README.md`
- Delete: `/Users/zhangyuting/github/udbx4x/udbx4go/viewer/TechSpec.md`

- [ ] **Step 1: 运行 Go 全量测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
```

Expected: 所有 Go 包测试通过，命令退出码为 `0`。

- [ ] **Step 2: 运行 race 测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test -race ./...
```

Expected: race 测试通过，命令退出码为 `0`。

- [ ] **Step 3: 运行 stable T3 合规测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache make test-stable-t3
```

Expected: stable T3 合规测试通过，命令退出码为 `0`。

- [ ] **Step 4: 查看并提交 codec 实现**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git diff -- internal/codec pkg/types/geometry.go
git add internal/codec pkg/types/geometry.go
git diff --cached --stat
git commit -m "feat: add CAD and GeoText codecs"
```

Expected: 生成 codec 和几何类型提交。

- [ ] **Step 5: 查看并提交 dataset 实现**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git diff -- datasource.go internal/dataset
git add datasource.go internal/dataset
git diff --cached --stat
git commit -m "feat: align dataset semantics with stable API"
```

Expected: 生成 dataset 语义提交。

- [ ] **Step 6: 查看并提交合规、真实样本和性能测试**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git status --short '*_test.go' internal/codec internal/dataset
git diff -- '*_test.go' internal/codec internal/dataset
git add '*_test.go' internal/codec/*_test.go internal/dataset/*_test.go
git diff --cached --stat
git commit -m "test: add stable compliance and real sample coverage"
```

Expected: 生成 Go 测试提交。

- [ ] **Step 7: 查看并提交 roundtrip 工具**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git status --short cmd/udbx4go-roundtrip-fixture
git add cmd/udbx4go-roundtrip-fixture
git diff --cached --stat
git commit -m "chore: add Go roundtrip fixture generator"
```

Expected: 生成 roundtrip 夹具生成工具提交。

- [ ] **Step 8: 查看并提交文档治理和 viewer 文档迁移**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
git diff -- CHANGELOG.md Makefile cmd/udbx4go-viewer/README.md AGENTS.md CLAUDE.md udbx4spec-compliance-report.md .github/workflows/performance-baseline.yml viewer/IMPLEMENTATION_PLAN.md viewer/README.md viewer/TechSpec.md
git add CHANGELOG.md Makefile cmd/udbx4go-viewer/README.md AGENTS.md CLAUDE.md udbx4spec-compliance-report.md .github/workflows/performance-baseline.yml viewer/IMPLEMENTATION_PLAN.md viewer/README.md viewer/TechSpec.md
git diff --cached --stat
git commit -m "docs: align Go governance and performance notes"
```

Expected: 生成 Go 文档治理提交，旧 `viewer/` 文档删除进入提交。

- [ ] **Step 9: 提交后复跑 Go 验证**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
GOCACHE=/private/tmp/udbx4go-gocache make test-stable-t3
```

Expected: 两个命令均退出码为 `0`。

## Task 7: 初始化根 `udbx4x` meta 仓库

**Files:**
- Create: `/Users/zhangyuting/github/udbx4x/.gitignore`
- Track: `/Users/zhangyuting/github/udbx4x/AGENTS.md`
- Track: `/Users/zhangyuting/github/udbx4x/CLAUDE.md`
- Track: `/Users/zhangyuting/github/udbx4x/README.md`
- Track: `/Users/zhangyuting/github/udbx4x/ROADMAP.md`
- Track: `/Users/zhangyuting/github/udbx4x/GOVERNANCE.md`
- Track: `/Users/zhangyuting/github/udbx4x/CONTRIBUTING.md`
- Track: `/Users/zhangyuting/github/udbx4x/RELEASE.md`
- Track: `/Users/zhangyuting/github/udbx4x/SECURITY.md`
- Track: `/Users/zhangyuting/github/udbx4x/docs/`
- Track: `/Users/zhangyuting/github/udbx4x/data/SampleData.udbx`
- Track: `/Users/zhangyuting/github/udbx4x/UDBX开放数据格式白皮书(V1.0).pdf`
- Exclude: `/Users/zhangyuting/github/udbx4x/data/henan.udbx`
- Exclude: `/Users/zhangyuting/github/udbx4x/.claude/`
- Exclude: `/Users/zhangyuting/github/udbx4x/.codex/`
- Exclude: `/Users/zhangyuting/github/udbx4x/.gstack/`
- Exclude: `/Users/zhangyuting/github/udbx4x/labels/`
- Exclude: `/Users/zhangyuting/github/udbx4x/sessions/`
- Exclude: `/Users/zhangyuting/github/udbx4x/statuses/`
- Exclude: `/Users/zhangyuting/github/udbx4x/views.json`
- Exclude: `/Users/zhangyuting/github/udbx4x/events.jsonl`
- Exclude: `/Users/zhangyuting/github/udbx4x/config.json`
- Exclude: `/Users/zhangyuting/github/udbx4x/jmh结果.txt`

- [ ] **Step 1: 创建根 `.gitignore`**

Create `/Users/zhangyuting/github/udbx4x/.gitignore` with:

```gitignore
.DS_Store

# Local agent/runtime state
.claude/
.claude-plugin/
.codex/
.gstack/
labels/
sessions/
statuses/
views.json
events.jsonl
config.json

# Local benchmark scratch files
jmh结果.txt

# Private or pending sample files
data/henan.udbx

# Nested repositories are managed independently
udbx4spec/
udbx4j/
udbx4ts/
udbx4go/

# Local dependency/build outputs in tool experiments
udbx-copilot/node_modules/
udbx-copilot/dist/
```

- [ ] **Step 2: 初始化根仓库**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git init
git status --short
```

Expected: 根目录生成 `.git/`，`git status --short` 显示根文档、`docs/`、`data/SampleData.udbx`、白皮书和 `.gitignore` 为未跟踪文件，不显示四个子仓库内部文件。

- [ ] **Step 3: 提交根治理入口**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git add .gitignore AGENTS.md CLAUDE.md README.md ROADMAP.md GOVERNANCE.md CONTRIBUTING.md RELEASE.md SECURITY.md
git diff --cached --stat
git commit -m "docs: add udbx4x governance entrypoints"
```

Expected: 生成根治理入口提交。

- [ ] **Step 4: 提交跨项目文档**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git add docs
git diff --cached --stat
git commit -m "docs: add cross-project architecture and standards"
```

Expected: 生成跨项目文档提交。

- [ ] **Step 5: 提交可公开样本和白皮书**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git add data/SampleData.udbx "UDBX开放数据格式白皮书(V1.0).pdf"
git diff --cached --stat
git commit -m "docs: add public sample and whitepaper assets"
```

Expected: 生成样本和白皮书资产提交，不包含 `data/henan.udbx`。

- [ ] **Step 6: 设置根仓库远端**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git remote add origin https://github.com/udbx4x/udbx4x.git
git remote -v
```

Expected:

```text
origin  https://github.com/udbx4x/udbx4x.git (fetch)
origin  https://github.com/udbx4x/udbx4x.git (push)
```

- [ ] **Step 7: 推送根仓库**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git push -u origin main
```

Expected: GitHub 上出现 `udbx4x/udbx4x` 根治理仓库，`main` 分支跟踪 `origin/main`。

## Task 8: 处理 `udbx-copilot` 仓库归属

**Files:**
- Read: `/Users/zhangyuting/github/udbx4x/udbx-copilot/README.md`
- Read: `/Users/zhangyuting/github/udbx4x/udbx-copilot/package.json`
- Read: `/Users/zhangyuting/github/udbx4x/udbx-copilot/src/`
- Read: `/Users/zhangyuting/github/udbx4x/udbx-copilot/tests/`

- [ ] **Step 1: 检查工具目录状态**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
test -d udbx-copilot/.git && echo "git repo" || echo "not a git repo"
find udbx-copilot -maxdepth 2 -type f \( -name README.md -o -name package.json -o -name "*.ts" -o -name "*.js" \) | sort | sed -n '1,80p'
```

Expected:

```text
not a git repo
```

and a list of CLI source, test, and package files.

- [ ] **Step 2: 决策工具仓库路径**

Decision:

```text
短期不把 udbx-copilot 混入根 meta 仓库。
推荐后续新建独立仓库 https://github.com/udbx4x/udbx-copilot.git。
原因是工具层存在独立 npm 依赖、dist、fixtures、测试和发布节奏。
```

- [ ] **Step 3: 在根 `.gitignore` 中保持工具构建产物排除**

Verify `/Users/zhangyuting/github/udbx4x/.gitignore` contains:

```gitignore
udbx-copilot/node_modules/
udbx-copilot/dist/
```

Expected: 根仓库不会误提交工具层依赖和构建产物。

## Task 9: 推送与发布前远端整理

**Files:**
- Read: `/Users/zhangyuting/github/udbx4x/udbx4spec/.git/config`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4j/.git/config`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4ts/.git/config`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4go/.git/config`

- [ ] **Step 1: 检查所有远端**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  printf '\n[%s]\n' "$d"
  git -C "$d" remote -v
done
```

Expected: `udbx4spec`、`udbx4j`、`udbx4go` 使用 `github.com/udbx4x/*`；`udbx4ts` 同时存在个人远端和组织远端。

- [ ] **Step 2: 将 `udbx4ts` 默认 push 远端对齐到组织仓库**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x/udbx4ts
git remote set-url origin https://github.com/udbx4x/udbx4ts.git
git remote remove udbx4x
git remote -v
```

Expected:

```text
origin  https://github.com/udbx4x/udbx4ts.git (fetch)
origin  https://github.com/udbx4x/udbx4ts.git (push)
```

- [ ] **Step 3: 推送各子仓库当前分支**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git -C udbx4spec push origin main
git -C udbx4j push origin main
git -C udbx4ts push origin HEAD:main
git -C udbx4go push origin HEAD:main
```

Expected: 四个远端仓库 `main` 分支包含本轮提交。

## Task 10: 建立后续提交流程

**Files:**
- Modify: `/Users/zhangyuting/github/udbx4x/AGENTS.md`
- Modify: `/Users/zhangyuting/github/udbx4x/docs/guides/release-workflow.md`
- Modify: `/Users/zhangyuting/github/udbx4x/docs/guides/local-development.md`

- [ ] **Step 1: 在 `AGENTS.md` 增加提交治理摘要**

Add this section to `/Users/zhangyuting/github/udbx4x/AGENTS.md` under `开发基础约定`:

```markdown
### 提交治理约定

- 当前工作区采用多仓库治理：根 `udbx4x` 仓库负责跨项目文档和治理，`udbx4spec`、`udbx4j`、`udbx4ts`、`udbx4go` 独立提交和发布。
- 不使用 `git add .` 提交跨仓库成果。
- 跨语言语义变更必须先提交 `udbx4spec`，再提交语言 SDK。
- 每次提交前必须运行对应子项目的最小验证命令，并在提交说明或实施记录中记录验证结果。
- 样本文件提交前必须确认来源、授权、公开范围和校验方式。
```

- [ ] **Step 2: 更新发布流程文档**

Add this section to `/Users/zhangyuting/github/udbx4x/docs/guides/release-workflow.md`:

```markdown
## 多仓库提交顺序

涉及跨语言行为的发布，按以下顺序提交和验证：

1. `udbx4spec`：规范、夹具、reference、合规矩阵。
2. `udbx4j`：Java SDK 实现、测试、文档。
3. `udbx4ts`：TypeScript SDK 实现、测试、文档。
4. `udbx4go`：Go SDK 实现、测试、文档。
5. `udbx4x`：总入口文档、路线图、样本治理、跨项目计划。

任一 SDK 未通过本地验证时，不推进对应语言发布。
```

- [ ] **Step 3: 更新本地开发文档**

Add this section to `/Users/zhangyuting/github/udbx4x/docs/guides/local-development.md`:

````markdown
## 提交前最小验证

### udbx4spec

```bash
cd udbx4spec
git status --short
```

### udbx4j

```bash
cd udbx4j
make test
make test-stable-t3
```

### udbx4ts

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:browser
```

### udbx4go

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
GOCACHE=/private/tmp/udbx4go-gocache make test-stable-t3
```
````

- [ ] **Step 4: 提交根流程文档更新**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git add AGENTS.md docs/guides/release-workflow.md docs/guides/local-development.md
git diff --cached --stat
git commit -m "docs: define multi-repository commit workflow"
```

Expected: 根仓库生成提交流程文档更新提交。

## Task 11: 最终验证和状态清理

**Files:**
- Read: `/Users/zhangyuting/github/udbx4x`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4spec`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4j`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4ts`
- Read: `/Users/zhangyuting/github/udbx4x/udbx4go`

- [ ] **Step 1: 检查所有仓库状态**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
printf '\n[root]\n'
git status --short
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  printf '\n[%s]\n' "$d"
  git -C "$d" status --short
done
```

Expected: 根仓库和四个子仓库没有未提交的业务变更；可能出现 `.DS_Store`、本地缓存、构建产物等已忽略文件。

- [ ] **Step 2: 检查最近提交**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
printf '\n[root]\n'
git log --oneline -5
for d in udbx4spec udbx4j udbx4ts udbx4go; do
  printf '\n[%s]\n' "$d"
  git -C "$d" log --oneline -5
done
```

Expected: 能看到本计划中定义的语义化提交。

- [ ] **Step 3: 记录最终验证结果**

Create a final note in `/Users/zhangyuting/github/udbx4x/docs/superpowers/plans/2026-06-24-repository-commit-governance-plan.md` under this heading:

```markdown
## 执行结果记录

- 根仓库提交状态：
- `udbx4spec` 提交状态：
- `udbx4j` 提交状态：
- `udbx4ts` 提交状态：
- `udbx4go` 提交状态：
- 未提交但保留的本地文件：
- 未执行或失败的验证命令：
```

Expected: 文档保留本次提交治理执行结果，便于后续审计。

- [ ] **Step 4: 提交计划执行记录**

Run:

```bash
cd /Users/zhangyuting/github/udbx4x
git add docs/superpowers/plans/2026-06-24-repository-commit-governance-plan.md
git diff --cached --stat
git commit -m "docs: record repository commit governance execution"
```

Expected: 根仓库生成计划执行记录提交。

## 风险和控制措施

- 风险：不同阶段成果混入同一提交。
  控制：禁止 `git add .`，每次使用精确路径暂存。

- 风险：根目录文档没有 Git 归属。
  控制：初始化 `udbx4x` meta 仓库，只提交治理和跨项目文档，忽略嵌套 SDK 仓库。

- 风险：`henan.udbx` 授权待定。
  控制：根 `.gitignore` 明确排除 `data/henan.udbx`。

- 风险：`udbx4ts` 推送到个人仓库而非组织仓库。
  控制：推送前将 `origin` 对齐到 `https://github.com/udbx4x/udbx4ts.git`。

- 风险：提交 CI 配置但暂时不投入 CI 验证。
  控制：CI 文件可提交为后续能力，但发布准入仍以本地验证命令为准。

- 风险：子仓库处于 `gitbutler/workspace` 分支。
  控制：推送使用 `git push origin HEAD:main`，并在推送前确认维护者接受将当前工作区结果进入 `main`。

## 验收标准

- 当前已有成果被拆分为语义清晰、可审查、可回滚的小提交。
- `udbx4spec`、`udbx4j`、`udbx4ts`、`udbx4go` 的提交顺序符合“规范先行、SDK 跟进”的依赖方向。
- 根目录治理文档进入 `udbx4x` meta 仓库，四个 SDK 仓库仍保持独立发布。
- `SampleData.udbx` 可提交，`henan.udbx` 不提交。
- 每个 SDK 仓库提交前后均有本地验证命令。
- 后续贡献者可以根据 `AGENTS.md` 和 `docs/guides/` 理解多仓库提交流程。

## 自检结果

- Spec coverage: 本计划覆盖代码库设计、提交顺序、操作步骤、验证命令、根仓库治理、样本公开边界、远端整理和后续流程固化。
- Placeholder scan: 未发现计划占位项，所有任务均包含具体路径、命令和预期结果。
- Type consistency: 本计划不定义代码类型；仓库名、路径名、远端名与当前工作区检查结果一致。

## 执行结果记录

- 根仓库提交状态：未完成。已创建 `/Users/zhangyuting/github/udbx4x/.gitignore`，但根目录 `git init` 在 sandbox 下返回 `Operation not permitted`；提升权限重试被外部审批服务拒绝，错误码为 `SETTLEMENT_PRICING_NOT_FOUND`。
- `udbx4spec` 提交状态：已完成本地提交，当前 `main` 分支相对 `origin/main` ahead 4。新增提交为 `2c284a6 docs: define geotext and stable API surface`、`11ea7dd test: add stable compliance fixtures`、`3f0f271 refactor: align reference APIs with stable surface`、`1087ea2 chore: add compliance fixture tools`。
- `udbx4j` 提交状态：已完成本地提交，当前 `main` 分支相对 `origin/main` ahead 3。新增提交为 `a90493c feat: align SDK implementation with stable compliance surface`、`9b3a7e2 test: add stable compliance and real sample coverage`、`3ca9183 docs: align Java project governance and performance notes`。
- `udbx4ts` 提交状态：已完成本地提交，并从 `gitbutler/workspace` 切到普通分支 `commit-governance/udbx4ts-stable-surface`。已将默认 `origin` 调整为 `https://github.com/udbx4x/udbx4ts.git`。新增提交为 `6686590 feat: align core SDK with stable compliance surface`、`ffcec04 feat: align browser runtime with stable dataset API`、`6d85741 test: add stable compliance and real sample coverage`、`b0a410f chore: add TypeScript performance baseline tooling`、`cf754da docs: align TypeScript public API and compliance docs`。
- `udbx4go` 提交状态：已完成本地提交，并从 `gitbutler/workspace` 切到普通分支 `commit-governance/udbx4go-public-api`。新增提交为 `3ec20b7 fix: expose stable public API types`、`76b5fec feat: add CAD and GeoText codecs`、`e419f8d feat: align dataset semantics with stable API`、`4eb14b8 test: add stable compliance and real sample coverage`、`1e7f6a0 chore: add Go roundtrip fixture generator`、`52188ed docs: align Go governance and performance notes`。
- 未提交但保留的本地文件：根目录 `.gitignore` 和根目录治理文档仍未进入根 Git 仓库，因为根仓库尚未初始化；`udbx-copilot` 仍不是 Git 仓库；`data/henan.udbx` 仍按待定授权处理，不纳入提交计划。
- 验证命令结果：`udbx4j make test` 通过，`udbx4j make test-stable-t3` 通过；`udbx4ts npm run typecheck`、`npm test`、`npm run test:browser` 均通过；`udbx4go go test ./...`、`go test -race ./...`、`make test-stable-t3` 均通过。`go test -race` 出现 macOS linker warning，但命令退出码为 0。
- 未执行或失败的外部操作：根目录 `git init` 需要提升权限但审批服务不可用；`git push origin main` 推送 `udbx4spec` 时同样被审批服务拒绝，错误码为 `SETTLEMENT_PRICING_NOT_FOUND`。因此远端推送和根 meta 仓库提交尚未完成。
