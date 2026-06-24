# 三端公开 API 最小稳定面实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 以 `udbx4spec` 为准，建立 Java、TypeScript、Go 三端 SDK 的最小公开 API 稳定面，并用自动化测试证明核心读写语义一致。

**Architecture:** `udbx4spec/docs/08-api-stable-surface.md` 定义统一语义，`06-language-mapping.md` 解释语言惯用映射。三端实现保留语言生态形态，但 `DataSource`、`Dataset`、`Feature`、`Geometry`、`count/list/getById/insert/update/delete` 的行为必须一致。

**Tech Stack:** Java 17 + Maven + JUnit，TypeScript + Vitest，Go + testing/testify，`udbx4spec` Markdown 规范。

---

## 范围

本计划只覆盖 SDK 发布前必须稳定的最小面：

- 数据源：打开、创建、关闭、列出数据集、按名称获取数据集。
- 数据集：`info`、`getFields`、`count`、`list`、`iterate/stream`、`getById`。
- 基础写入：`insert`、`insertMany`、`update`、`delete`。
- 模型：`DatasetKind`、`FieldType`、`Feature`、`TabularRecord`、2D/3D Geometry、Text、CAD。
- 错误：format、not found、unsupported、constraint、IO。

不纳入本轮：

- 投影转换。
- 空间索引高级查询。
- 样式编辑完整能力。
- SQL 查询 DSL。
- 完整 CAD 复杂对象编辑。

## 任务

### Task 1: 补齐稳定 API 规范

**Files:**
- Create: `udbx4spec/docs/08-api-stable-surface.md`
- Modify: `udbx4spec/docs/06-language-mapping.md`
- Modify: `docs/standards/api-design.md`

- [x] **Step 1: 新增稳定 API 规范**

写入 `udbx4spec/docs/08-api-stable-surface.md`，明确最小稳定面、返回语义、错误语义和语言映射表。

- [x] **Step 2: 更新语言映射**

在 `udbx4spec/docs/06-language-mapping.md` 中明确 Java/TypeScript/Go 的 API 名称可以不同，但 NotFound、list 排序、count 语义必须一致。

- [x] **Step 3: 更新工程 API 设计规范**

在 `docs/standards/api-design.md` 中索引 `08-api-stable-surface.md`，并声明公开 API 变更必须先更新稳定面规范。

### Task 2: 用失败测试锁定 NotFound 语义

**Files:**
- Modify: `udbx4j/src/test/java/com/supermap/udbx/spec/Dataset2DSpecTest.java`
- Modify: `udbx4j/src/test/java/com/supermap/udbx/spec/DatasetDeleteUpdateSpecTest.java`
- Modify: `udbx4ts/tests/unit/datasource-tabular.spec.ts`
- Modify: `udbx4ts/tests/unit/datasource-point.spec.ts`
- Modify: `udbx4go/internal/dataset/point_test.go`
- Modify: `udbx4go/internal/dataset/tabular_test.go`

- [x] **Step 1: Java 增加 getById 缺失记录抛 UdbxNotFoundError 的测试**

运行目标测试前应失败，因为当前 Java 多个 `getById` 返回 `null`。

- [x] **Step 2: TypeScript 增加 getById 缺失记录 reject UdbxNotFoundError 的测试**

运行目标测试前应失败，因为当前 TypeScript 多个 `getById` 返回 `null`。

- [x] **Step 3: Go 增加错误 id 上下文测试**

Go 当前已返回 NotFound，但错误消息中的 id 可能来自扫描辅助函数默认值。测试应锁定缺失 id 必须出现在错误上下文中。

### Task 3: 修正 NotFound 实现

**Files:**
- Modify: `udbx4j/src/main/java/com/supermap/udbx/dataset/*.java`
- Modify: `udbx4ts/src/core/dataset/*.ts`
- Modify: `udbx4go/internal/dataset/vector.go`
- Modify: `udbx4go/internal/dataset/tabular.go`

- [x] **Step 1: Java getById 未命中时抛 UdbxNotFoundError**

所有公开 `getById` 未命中必须抛出 `UdbxNotFoundError(datasetName, id)` 或等价上下文。

- [x] **Step 2: TypeScript getById 未命中时 reject UdbxNotFoundError**

所有公开 `getById` 返回类型从 `Promise<T | null>` 收敛为 `Promise<T>`。

- [x] **Step 3: Go NotFound 保留请求 id**

扫描辅助函数不得把缺失 id 写成 `0`。

### Task 4: 收敛 update/delete 字段与记录缺失语义

**Files:**
- Modify: `udbx4j/src/test/java/com/supermap/udbx/spec/DatasetDeleteUpdateSpecTest.java`
- Modify: `udbx4ts/tests/unit/datasource-tabular.spec.ts`
- Modify: `udbx4ts/tests/unit/datasource-point.spec.ts`
- Modify: `udbx4j/src/main/java/com/supermap/udbx/dataset/*.java`
- Modify: `udbx4ts/src/core/dataset/*.ts`

- [x] **Step 1: 写失败测试**

缺失记录的 `update/delete` 必须返回 NotFound；未知字段的 `update` 必须返回 FieldNotFound/NotFound 类错误，不得静默忽略。

- [x] **Step 2: 修实现**

执行 SQL 后检查 affected rows；字段变更先校验字段集合。

### Task 5: 验证 list/count 稳定语义

**Files:**
- Modify: `udbx4spec/docs/08-api-stable-surface.md`
- Modify: 三端现有 dataset 测试文件

- [x] **Step 1: 测试 list 默认按 SmID 升序**

三端 `list()` 和带 `ids` 的 `list(options)` 都必须按 `SmID` 升序返回。

- [x] **Step 2: 测试 count 来源物理表**

三端 `count()` 必须读取物理表真实行数，不以 `SmRegister.SmObjectCount` 缓存值为准。

### Task 6: 更新文档和合规报告

**Files:**
- Modify: `udbx4spec/compliance/roundtrip-matrix.md`
- Modify: `udbx4j/udbx4spec-compliance-report.md`
- Modify: `udbx4ts/udbx4spec-compliance-report.md`
- Modify: `udbx4go/udbx4spec-compliance-report.md`
- Modify: 三端 README/API 文档中公开 API 章节

- [x] **Step 1: 更新稳定面覆盖状态**

说明三端均已覆盖最小稳定面。

- [x] **Step 2: 更新语言差异说明**

说明 Java 使用异常，TypeScript 使用 rejected Promise，Go 使用 `(value, error)`。

### Task 7: 发布级验收

**Commands:**

```bash
cd udbx4j
make test
make test-stable-t3
```

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:stable-t3
npm run build
```

```bash
cd udbx4go
GOCACHE=/private/tmp/udbx4go-gocache go test ./...
make test-stable-t3
```

```bash
cd udbx4spec
node tools/check-fixtures.mjs
```

- [x] **Step 1: 运行验收命令**
- [x] **Step 2: 修复失败项**
- [x] **Step 3: 完成要求逐项审计**

## 2026-06-24 补充缺口修复记录

后续审计发现 Java 侧 `update/delete` 在目标对象不存在时仍存在普通 `RuntimeException` 包装路径，不满足 `udbx4spec/docs/08-api-stable-surface.md` 中 not found 错误语义。

已完成修复：

- Java `Dataset` 基类新增写操作目标记录缺失错误构造入口 `recordNotFound(id)`。
- Java 九类公开数据集的 `update/delete` 在 affected rows 为 0 时统一抛出 `UdbxNotFoundError`。
- Java 对应 spec 测试已将缺失对象断言从普通 `RuntimeException` 收敛为 `UdbxNotFoundError`。
- 清理机械替换引入的读路径多余回滚逻辑；读路径继续只包装 `SQLException`，写事务路径保留 `UdbxNotFoundError` 回滚后透传。

本次重新验证：

```bash
cd udbx4j
sh -c 'export JAVA_HOME="/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home" && export PATH="$JAVA_HOME/bin:$PATH" && mvn -Dtest=DatasetDeleteUpdateSpecTest,Dataset2DSpecTest,Vector3DDatasetSpecTest test'
sh -c 'export JAVA_HOME="/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home" && export PATH="$JAVA_HOME/bin:$PATH" && make test'
sh -c 'export JAVA_HOME="/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home" && export PATH="$JAVA_HOME/bin:$PATH" && make test-stable-t3'
```

验证结果：

- 目标稳定面测试：68 tests，0 failures，0 errors。
- Java 常规测试：330 tests，0 failures，0 errors，1 skipped。
- Java stable T3 门禁：第一段 19 tests，0 failures，0 errors；第二段 5 tests，0 failures，0 errors。
