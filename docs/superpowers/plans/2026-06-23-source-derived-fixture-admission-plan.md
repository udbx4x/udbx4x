# Source-Derived Fixture Admission Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立 `udbx4spec` T3 Source-derived Fixture 的准入、生成、校验和三端验证路径，让真实样本行为可以安全沉淀为公开合规资产。

**Architecture:** T3 夹具必须先经过样本元数据、授权、脱敏和规范价值判定，再由 `udbx4spec/tools/` 脚本生成最小化资产。`udbx4spec/compliance/fixture-standard.md` 负责长期标准，`docs/guides/source-fixture-admission.md` 负责操作流程，`check-fixtures.mjs` 负责机器校验。

**Tech Stack:** Markdown 文档、Node.js `node:sqlite`、SHA-256 校验、`udbx4spec` manifest、Java/TypeScript/Go SDK 合规测试。

---

## 文件结构

- Modify: `docs/samples/README.md`，补齐 `SampleData.udbx` 来源、授权名称、生成工具版本后才能进入 T3。
- Modify: `docs/samples/compatibility-matrix.md`，记录每个 T3 候选行为的兼容状态。
- Modify: `docs/guides/source-fixture-admission.md`，维护 T3 准入操作指南。
- Modify: `udbx4spec/compliance/fixture-standard.md`，维护 T3 manifest 字段和校验规则。
- Create: `udbx4spec/tools/generate-source-derived-fixtures.mjs`，从已确认可公开样本生成 T3 资产。
- Modify: `udbx4spec/tools/check-fixtures.mjs`，校验 T3 manifest 和资产语义。
- Create: `udbx4spec/compliance/source-derived/manifest.json`，记录 T3 夹具。
- Create: `udbx4spec/compliance/source-derived/sampledata/...`，保存从 `SampleData.udbx` 派生的最小夹具。
- Modify: `udbx4spec/compliance/README.md`，增加 T3 资产入口。
- Modify: `udbx4spec/compliance/roundtrip-matrix.md`，增加 R6 资产和语言验证记录。

## Task 1: 补齐 SampleData 准入元数据

**Files:**
- Modify: `docs/samples/README.md`
- Modify: `docs/samples/compatibility-matrix.md`

- [ ] **Step 1: 确认来源和授权信息**

向样本提供方确认以下字段，并写入 `docs/samples/README.md`：

```markdown
- 来源：SuperMap 官方示例数据或维护者确认的公开样本来源
- 生成工具：SuperMap iDesktopX，具体版本按样本元数据或提供方确认填写
- 可公开分发：是
- 授权名称：按提供方确认填写，例如示例数据授权、开源仓库随附授权或单独授权说明
```

- [ ] **Step 2: 校验源样本 SHA-256**

Run:

```bash
shasum -a 256 data/SampleData.udbx
```

Expected:

```text
b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39  data/SampleData.udbx
```

- [ ] **Step 3: 更新兼容矩阵候选状态**

在 `docs/samples/compatibility-matrix.md` 的 `SampleData.udbx` 下一步中加入：

```markdown
- `County_T` 异常 Text BLOB 可作为首批 T3 `experimental` 候选，目标是验证真实 Text 中非 UTF-8 可读字节的保留和容错解码。
- `CADDT` 可作为第二批 T3 `experimental` 候选，目标是验证真实 CAD `SmGeoType` 与 `SmGeometry` 的最小解码行为。
```

## Task 2: 建立 T3 目录和 manifest

**Files:**
- Create: `udbx4spec/compliance/source-derived/manifest.json`
- Create: `udbx4spec/compliance/source-derived/README.md`
- Modify: `udbx4spec/compliance/README.md`

- [ ] **Step 1: 创建 manifest 初始结构**

Create `udbx4spec/compliance/source-derived/manifest.json`:

```json
{
  "schemaVersion": 1,
  "generatedAt": "2026-01-01T00:00:00.000Z",
  "status": "ready",
  "fixtures": []
}
```

- [ ] **Step 2: 创建目录说明**

Create `udbx4spec/compliance/source-derived/README.md`:

```markdown
# Source-derived Fixtures

本目录存放从已确认可公开真实样本派生的 T3 合规夹具。

T3 夹具必须由 `udbx4spec/tools/generate-source-derived-fixtures.mjs` 生成，不能手工编辑二进制文件。每个条目必须在 `manifest.json` 中记录源样本、源 SHA-256、抽取条件、授权状态、脱敏状态和规范价值。
```

- [ ] **Step 3: 更新 compliance 入口**

在 `udbx4spec/compliance/README.md` 的目录结构中加入：

```markdown
├── source-derived/                   # 从公开真实样本派生的 T3 合规夹具
```

在组成部分中加入：

```markdown
### Source-derived Fixtures

位于 `source-derived/` 目录。该目录只接收已完成来源、授权、脱敏和规范价值确认的 T3 夹具。
```

## Task 3: 实现 T3 生成脚本

**Files:**
- Create: `udbx4spec/tools/generate-source-derived-fixtures.mjs`

- [ ] **Step 1: 编写脚本骨架**

Create `udbx4spec/tools/generate-source-derived-fixtures.mjs`:

```javascript
#!/usr/bin/env node

import { createHash } from "node:crypto";
import { mkdir, readFile, writeFile } from "node:fs/promises";
import { DatabaseSync } from "node:sqlite";
import { dirname, resolve } from "node:path";
import { fileURLToPath } from "node:url";

const __dirname = dirname(fileURLToPath(import.meta.url));
const specRoot = resolve(__dirname, "..");
const workspaceRoot = resolve(specRoot, "..");
const sourcePath = resolve(workspaceRoot, "data/SampleData.udbx");
const expectedSourceSha256 = "b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39";
const outputRoot = resolve(specRoot, "compliance/source-derived");

function sha256(bytes) {
  return createHash("sha256").update(bytes).digest("hex");
}

async function assertSourceSample() {
  const bytes = await readFile(sourcePath);
  const actual = sha256(bytes);
  if (actual !== expectedSourceSha256) {
    throw new Error(`SampleData.udbx SHA-256 不匹配，expected=${expectedSourceSha256}, actual=${actual}`);
  }
}

await assertSourceSample();
await mkdir(resolve(outputRoot, "sampledata/county-t"), { recursive: true });

const db = new DatabaseSync(sourcePath, { readOnly: true });
try {
  const row = db.prepare('SELECT SmGeometry FROM "County_T" WHERE SmID = 1').get();
  if (!row?.SmGeometry) {
    throw new Error("County_T SmID=1 缺少 SmGeometry");
  }
  const blob = Buffer.from(row.SmGeometry);
  const relativePath = "sampledata/county-t/smid-1-smgeometry.bin";
  await writeFile(resolve(outputRoot, relativePath), blob);

  const manifest = {
    schemaVersion: 1,
    generatedAt: "2026-01-01T00:00:00.000Z",
    status: "ready",
    fixtures: [
      {
        id: "sampledata-county-t-smid-1-smgeometry",
        path: relativePath,
        tier: "T3",
        stability: "experimental",
        usage: ["decode"],
        byteSize: blob.byteLength,
        sha256: sha256(blob),
        source: {
          sample: "data/SampleData.udbx",
          sha256: expectedSourceSha256,
          dataset: "County_T",
          selection: "SmID=1 SmGeometry"
        },
        licenseStatus: "public-confirmed",
        deidentification: "not-required",
        specValue: "验证真实 Text 数据集中异常文本字节的保留和容错解码行为"
      }
    ]
  };
  await writeFile(resolve(outputRoot, "manifest.json"), JSON.stringify(manifest, null, 2) + "\n");
} finally {
  db.close();
}

console.log("Source-derived fixtures generated.");
```

- [ ] **Step 2: 运行生成脚本**

Run:

```bash
cd udbx4spec
node tools/generate-source-derived-fixtures.mjs
```

Expected:

```text
Source-derived fixtures generated.
```

## Task 4: 扩展 T3 校验

**Files:**
- Modify: `udbx4spec/tools/check-fixtures.mjs`

- [ ] **Step 1: 增加 source-derived 路径常量**

在 manifest 路径定义附近加入：

```javascript
const sourceDerivedRoot = resolve(specRoot, "compliance/source-derived");
const sourceDerivedManifestPath = resolve(sourceDerivedRoot, "manifest.json");
```

- [ ] **Step 2: 增加 T3 字段校验函数**

在 `validateCommonFixtureEntry` 后加入：

```javascript
function validateSourceDerivedEntry(entry) {
  if (entry.tier !== "T3") {
    fail(`${entry.id}: source-derived fixture 的 tier 必须为 T3`);
  }
  if (!entry.source?.sample || !entry.source?.sha256 || !entry.source?.dataset || !entry.source?.selection) {
    fail(`${entry.id}: T3 fixture 缺少 source.sample/source.sha256/source.dataset/source.selection`);
  }
  if (!["public-confirmed", "internal-only", "pending"].includes(entry.licenseStatus)) {
    fail(`${entry.id}: licenseStatus 必须是 public-confirmed/internal-only/pending`);
  }
  if (!["not-required", "applied", "required"].includes(entry.deidentification)) {
    fail(`${entry.id}: deidentification 必须是 not-required/applied/required`);
  }
  if (!entry.specValue) {
    fail(`${entry.id}: T3 fixture 缺少 specValue`);
  }
  if (entry.stability === "stable" && entry.licenseStatus !== "public-confirmed") {
    fail(`${entry.id}: stable T3 fixture 必须是 public-confirmed`);
  }
}
```

- [ ] **Step 3: 接入 source-derived manifest 校验**

在其他 manifest 校验块后加入：

```javascript
const sourceDerivedManifest = await readJson(sourceDerivedManifestPath);
if (sourceDerivedManifest) {
  validateCommonManifest(sourceDerivedManifest, "source-derived/manifest.json");
  if (Array.isArray(sourceDerivedManifest.fixtures)) {
    for (const entry of sourceDerivedManifest.fixtures) {
      if (!validateCommonFixtureEntry(entry, "source-derived/manifest.json")) {
        continue;
      }
      validateSourceDerivedEntry(entry);
      try {
        const bytes = await readFile(resolve(sourceDerivedRoot, entry.path));
        if (bytes.byteLength !== entry.byteSize) {
          fail(`${entry.id}: byteSize 不匹配，manifest=${entry.byteSize}, actual=${bytes.byteLength}`);
        }
        const actualSha = sha256(bytes);
        if (actualSha !== entry.sha256) {
          fail(`${entry.id}: sha256 不匹配，manifest=${entry.sha256}, actual=${actualSha}`);
        }
      } catch (error) {
        fail(`${entry.id}: 无法读取 source-derived fixture 文件 ${entry.path}：${error.message}`);
      }
    }
  }
}
```

- [ ] **Step 4: 运行校验**

Run:

```bash
cd udbx4spec
node tools/check-fixtures.mjs
```

Expected:

```text
合规资产检查通过。
```

## Task 5: 接入语言实现验证

**Files:**
- Modify: `udbx4ts/tests/...`
- Modify: `udbx4j/src/test/...`
- Modify: `udbx4go/...`
- Modify: `udbx4spec/compliance/roundtrip-matrix.md`
- Modify: `docs/samples/compatibility-matrix.md`

- [ ] **Step 1: TypeScript 增加 T3 读取测试**

测试应读取：

```text
udbx4spec/compliance/source-derived/sampledata/county-t/smid-1-smgeometry.bin
```

验收要求：

```text
能按 GeoText 布局解析；
异常文本字节不得导致崩溃；
原始字节长度与 manifest 一致。
```

- [ ] **Step 2: Java 增加 T3 读取测试**

验收要求同 TypeScript。若 Java 暂不支持异常文本字节容错，应在合规报告中标记 `experimental T3 not-run`，不能标记为通过。

- [ ] **Step 3: Go 增加 T3 读取测试**

验收要求同 TypeScript。Go 已支持异常文本字节容错读取时，应断言解码不会丢失原始字节。

- [ ] **Step 4: 更新矩阵**

在 `udbx4spec/compliance/roundtrip-matrix.md` 的 R6 中记录：

```markdown
| `sampledata-county-t-smid-1-smgeometry` | T3 | experimental | Text 异常字节 decode | TS/Java/Go 状态按合规报告记录 |
```

## Task 6: 稳定化与发布门禁

**Files:**
- Modify: `udbx4spec/compliance/source-derived/manifest.json`
- Modify: `RELEASE.md`
- Modify: `docs/guides/compliance-testing.md`

- [ ] **Step 1: 判断是否从 experimental 提升到 stable**

只有同时满足以下条件才允许提升：

```text
来源、授权、公开分发和脱敏状态明确；
Java、TypeScript、Go 三端均有自动化验证；
check-fixtures 通过；
合规矩阵和样本矩阵已更新；
没有把样本偶然值写成格式规则。
```

- [ ] **Step 2: 更新发布门禁**

若 T3 变为 `stable`，`RELEASE.md` 和 `docs/guides/compliance-testing.md` 必须要求发布前运行：

```bash
cd udbx4spec
node tools/check-fixtures.mjs
```

并要求三端合规测试读取 stable T3 夹具。

## Self-Review

- Spec coverage: 本计划覆盖 T3 准入元数据、目录和 manifest、生成脚本、校验脚本、三端验证、stable 提升和发布门禁。
- Placeholder scan: 本计划未使用 TBD、TODO 或“后续补充”作为执行步骤；授权字段必须由维护者确认后填写。
- Type consistency: `tier`、`stability`、`usage`、`source`、`licenseStatus`、`deidentification`、`specValue` 与 `docs/guides/source-fixture-admission.md` 保持一致。
