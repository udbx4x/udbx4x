# 发布准备状态

本文档记录 `udbx4x` 工作区在 2026-06-25 的发布准备状态，用于决定下一轮公开版本发布顺序、版本号和发布前门禁。

## 当前结论

当前代码和规范已经完成一轮提交治理，四个核心仓库均已建立独立仓库关系并同步到 `udbx4x` 组织远端。随后已启动发布准备：

- 根仓库 `udbx4x`：`main...origin/main`
- `udbx4spec`：本地已完成 `v1.1.0` 发布准备提交和 tag，待推送 `main` 与 `v1.1.0`
- `udbx4j`：`main...origin/main`
- `udbx4ts`：`main...origin/main`
- `udbx4go`：`main...origin/main`

下一步应先完成根目录发布准备文档提交，并推送 `udbx4spec v1.1.0`；之后再进入 `udbx4j v2.1.0` 发布准备。

## 建议发布顺序

1. `udbx4spec`
2. `udbx4j`
3. `udbx4ts`
4. `udbx4go`

原因：

- `udbx4spec` 是跨语言规范、合规夹具、golden bytes、roundtrip 和 stable T3 的事实入口。
- `udbx4j` 和 `udbx4ts` 是首批核心 SDK，应优先形成稳定对外认知。
- `udbx4go` 已具备发布基础，但仍是首次 tag，发布说明需要更克制地声明为早期 SDK。

## 仓库状态

| 仓库 | 当前版本来源 | 当前 tag | 建议下一版本 | 状态 |
|---|---|---|---|---|
| `udbx4spec` | `VERSION=1.1.0` | `v1.1.0` 本地已创建 | `v1.1.0` | 已完成本地准备，待推送 main 和 tag |
| `udbx4j` | `pom.xml=2.0.0` | `v2.0.0` | `v2.1.0` | 需要补 changelog、版本号、release notes |
| `udbx4ts` | `package.json=0.3.0` | `v0.3.0` | `v0.4.0` | 需要补 changelog、版本号、release notes |
| `udbx4go` | `go.mod` module path | 无 | `v0.1.0` | 需要形成首个 release notes 和 tag |

版本建议依据：

- `udbx4spec v1.1.0`：新增 GeoText 规范、稳定 API 面、合规数据库、roundtrip 夹具和工具，属于规范能力扩展。
- `udbx4j v2.1.0`：在 2.0.0 后新增 Text / GeoText、CAD 合规、stable T3、性能基线和公开 API 语义对齐，若未引入破坏性 API，可作为 minor 发布。
- `udbx4ts v0.4.0`：在 0.3.0 后新增稳定面、真实样本和合规门禁能力；0.x 阶段用 minor 表达公开能力扩展。
- `udbx4go v0.1.0`：首次公开 tag，应避免声明为稳定 1.0。

## 发布前阻塞项

### `udbx4spec`

- `VERSION` 已更新为 `1.1.0`。
- `CHANGELOG.md` 已新增 `1.1.0` 章节。
- 已重新生成并检查：
  - golden GAIA bytes。
  - golden GeoText bytes。
  - `compliance/compliance.udbx`。
  - 三端 roundtrip 夹具。
  - fixture manifest。
- 已创建本地 tag `v1.1.0`。
- 待完成：
  - 推送 `main`。
  - 推送 tag `v1.1.0`。

`1.1.0` 发布说明应覆盖：
  - GeoText 二进制布局规范。
  - 公开 API 最小稳定面。
  - `compliance/compliance.udbx`。
  - golden GAIA / golden GeoText bytes。
  - 三端 roundtrip 夹具。
  - source-derived stable T3 夹具。
  - compliance tools。

### `udbx4j`

- `pom.xml` 仍为 `2.0.0`。
- `CHANGELOG.md` 的 `Unreleased` 性能段落包含旧 API 名称和旧性能阶段信息，发布前需要整理为当前版本事实。
- 需要新增 `2.1.0` 章节，覆盖：
  - Text / GeoText 最小合规基线。
  - CAD 最小 GeoHeader 基线。
  - stable T3 和 compliance database 读取/roundtrip。
  - `update/delete/getById/count/list` 稳定语义对齐。
  - JMH 性能基线摘要。

### `udbx4ts`

- `package.json` 仍为 `0.3.0`。
- `CHANGELOG.md` 没有记录本轮 `0.4.0` 变更。
- 需要新增 `0.4.0` 章节，覆盖：
  - core SDK 稳定面。
  - 浏览器 runtime 稳定 API 对齐。
  - Text / GeoText、CAD、真实样本和 `udbx4spec` 合规测试。
  - 性能基线脚本和 workflow。

### `udbx4go`

- 尚无 release tag。
- `CHANGELOG.md` 仍为 `Unreleased`。
- 需要将当前内容整理为 `0.1.0` 首发章节，覆盖：
  - UDBX 基础读写。
  - 2D/3D、Tabular、Text、CAD 数据集。
  - 公开 API 类型 re-export。
  - `udbx4spec` 合规测试、真实样本测试和 roundtrip fixture generator。
  - 已知限制：复杂真实世界 Text / CAD / Network / Model 仍需扩展。

## 发布前强制门禁

### 规范门禁

```bash
cd udbx4spec
node tools/generate-golden-gaia-bytes.mjs
node tools/generate-golden-text-bytes.mjs
node tools/generate-compliance-db.mjs
node tools/generate-roundtrip-fixtures.mjs
node tools/check-fixtures.mjs
```

### Java 门禁

```bash
cd udbx4j
make test
make test-all
make test-stable-t3
mvn -DskipTests package
```

### TypeScript 门禁

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:stable-t3
npm run test:browser
npm run build
npm pack --dry-run
```

### Go 门禁

```bash
cd udbx4go
make test
make test-stable-t3
go test ./...
go test -race ./...
go vet ./...
go list ./...
```

## release notes 口径

发布说明必须使用克制表述：

- 可以声明：支持 `SampleData.udbx` 中首批 stable T3 source-derived 夹具。
- 可以声明：三端已对齐公开 API 最小稳定面。
- 可以声明：Text / GeoText 和 CAD 已达到最小合规基线。
- 不应声明：完整兼容所有 SuperMap 生成的主流 UDBX。
- 不应声明：Network、Network3D、Model 已支持。
- 不应声明：复杂 CAD/Text 样式完全兼容。

## 下一步执行建议

1. 先提交根目录发布准备文档。
2. 推送 `udbx4spec v1.1.0` 的 `main` 和 tag。
3. 再发布 `udbx4j v2.1.0` 和 `udbx4ts v0.4.0`。
4. 最后发布 `udbx4go v0.1.0`。
5. 每个仓库发布前先补 changelog、版本号和 release notes，再执行门禁命令。
