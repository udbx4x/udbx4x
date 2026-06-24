# 三端公开 API 文档入口统一实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将三端 SDK README、Go API 文档和 Java 参考伪接口统一到 `udbx4spec/docs/08-api-stable-surface.md` 定义的公开 API 最小稳定面。

**Architecture:** 根 README 作为统一入口说明跨语言稳定语义；三端 README 说明各语言 API 映射和错误处理方式；`udbx4spec/reference/java` 作为 Java 规范参考入口，不得保留与稳定面冲突的旧 nullable 或缓存计数语义。

**Tech Stack:** Markdown、Java pseudo-interface、ripgrep、`udbx4spec/tools/check-fixtures.mjs`。

---

## 任务

### Task 1: 根 README 增加稳定面入口

**Files:**

- Modify: `README.md`

- [x] **Step 1: 增加“公开 API 最小稳定面”章节**

说明三端 API 映射、not found、`list` 排序、`count` 物理表计数、`update/delete` 缺失对象错误语义。

### Task 2: 三端 README 对齐稳定语义

**Files:**

- Modify: `udbx4j/README.md`
- Modify: `udbx4ts/README.md`
- Modify: `udbx4ts/README.en.md`
- Modify: `udbx4go/README.md`
- Modify: `udbx4go/README.zh.md`

- [x] **Step 1: Java README 增加稳定语义和 `UdbxNotFoundError` 示例**
- [x] **Step 2: TypeScript 中英文 README 增加稳定语义和 rejected `UdbxNotFoundError` 示例**
- [x] **Step 3: Go 中英文 README 增加稳定语义和 `IsNotFound(err)` 示例**

### Task 3: Go API 文档对齐稳定语义

**Files:**

- Modify: `udbx4go/API.md`
- Modify: `udbx4go/API.zh.md`

- [x] **Step 1: 增加数据集稳定语义章节**
- [x] **Step 2: 补充 `GetByID`、`List`、`Update`、`Delete` 的 not found 和排序说明**

### Task 4: Java 参考伪接口修正

**Files:**

- Modify: `udbx4spec/reference/java/README.md`
- Modify: `udbx4spec/reference/java/dataset/Dataset.java`
- Modify: `udbx4spec/reference/java/dataset/VectorDataset.java`
- Modify: `udbx4spec/reference/java/dataset/TabularDataset.java`

- [x] **Step 1: 移除 `@Nullable getById` 旧语义说明**
- [x] **Step 2: 将 `count()` 从默认 `getObjectCount()` 改为必须由实现提供物理表真实计数**
- [x] **Step 3: 补充 `update/delete` 的 `UdbxNotFoundError` Javadoc**
- [x] **Step 4: 修复 `VectorDataset.java` 缺失的接口闭合括号**

### Task 5: 验证

**Commands:**

```bash
rg -n '@Nullable\s+[^\n]*getById|Nullable.*getById|getById\(id\).*@Nullable|return getInfo\(\)\.getObjectCount\(\)|count\(\).*getObjectCount' README.md udbx4j/README.md udbx4ts/README.md udbx4ts/README.en.md udbx4go/README.md udbx4go/README.zh.md udbx4go/API.md udbx4go/API.zh.md udbx4spec/reference/java udbx4spec/docs -S
node -e "const fs=require('fs'); for (const f of ['udbx4spec/reference/java/dataset/VectorDataset.java','udbx4spec/reference/java/dataset/TabularDataset.java','udbx4spec/reference/java/dataset/Dataset.java']) { const s=fs.readFileSync(f,'utf8'); let n=0; for (const c of s) { if (c==='{') n++; if (c==='}') n--; } console.log(f, n); if(n!==0) process.exitCode=1; }"
cd udbx4spec && node tools/check-fixtures.mjs
```

- [x] **Step 1: 旧冲突语义扫描**

结果只剩“不得返回 null / 不 resolve null”的正向约束文本，无旧 nullable 成功返回语义。

- [x] **Step 2: Java 参考伪接口括号平衡检查**

结果：`Dataset.java`、`VectorDataset.java`、`TabularDataset.java` 均为 0。

- [x] **Step 3: udbx4spec 合规资产检查**

结果：`合规资产检查通过。`
