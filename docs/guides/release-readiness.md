# 发布准备状态

本文档记录 `udbx4x` 工作区在 2026-06-26 的发布准备状态，用于决定下一轮公开版本发布顺序、版本号和发布前门禁。

## 当前结论

当前代码和规范已经完成一轮提交治理，四个核心仓库均已建立独立仓库关系并同步到 `udbx4x` 组织远端。随后已启动发布准备：

- 根仓库 `udbx4x`：`main...origin/main`
- `udbx4spec`：本地已完成 `v1.1.0` 发布准备提交和 tag，待推送 `main` 与 `v1.1.0`
- `udbx4j`：`main...origin/main`
- `udbx4ts`：本地已完成 `v0.4.0` 版本、changelog 和发布门禁准备，待提交、tag 与 npm 发布
- `udbx4go`：`main...origin/main`

下一步应提交 `udbx4ts v0.4.0` 发布准备结果；之后进入 `udbx4go v0.1.0` 首发准备。远端推送、tag 推送和正式发布仍受网络、仓库权限和生态发布凭据约束。

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
| `udbx4j` | `pom.xml=2.1.0` | `v2.0.0` | `v2.1.0` | 已完成本地版本、changelog 和发布门禁准备，待 tag / 发布 |
| `udbx4ts` | `package.json=0.4.0` | `v0.3.0` | `v0.4.0` | 已完成本地版本、changelog 和发布门禁准备，待提交 / tag / npm 发布 |
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

- `pom.xml` 已更新为 `2.1.0`。
- `README.md` 已更新为 `2.1.0` 发布准备口径，不提前声明 Maven Central 已发布。
- `CHANGELOG.md` 已新增 `2.1.0` 章节，并清理旧 API 名称对应的过期性能段落。
- JaCoCo 已合并单元测试与集成测试执行数据后再执行覆盖率报告和检查。
- Swing viewer 已从 SDK 核心覆盖率门禁中排除，覆盖率报告仍保留 viewer 包可见。
- GPG 签名默认在本地验证中跳过；发布 Maven Central 时必须显式使用 `-Dgpg.skip=false`。
- 已通过：
  - `make test`：345 个测试通过，1 个性能测试跳过。
  - `make test-all`：345 个单元/spec 测试通过，35 个集成测试通过，SDK 核心 JaCoCo 行覆盖率 `2869/3267 = 87.8%`，超过 80% 门禁。
  - `make test-stable-t3`：golden/spec 门禁 19 个测试通过，stable T3 真实样本门禁 5 个集成测试通过。
  - `make package`：生成 `udbx4j-2.1.0` 包成功。
- 当前阻塞：
  - 远端推送和 Maven Central 正式发布仍需具备网络、Maven Central 凭据和 GPG 签名环境。

`2.1.0` 发布说明应覆盖：
  - Text / GeoText 最小合规基线。
  - CAD 最小 GeoHeader 基线。
  - stable T3 和 compliance database 读取/roundtrip。
  - `update/delete/getById/count/list` 稳定语义对齐。
  - JMH 性能基线摘要。

### `udbx4ts`

- `package.json` 和 `package-lock.json` 已更新为 `0.4.0`。
- `CHANGELOG.md` 已新增 `0.4.0` 章节，覆盖公开 API 最小稳定面、合规数据库、三端 roundtrip、stable T3、真实样本和浏览器运行时稳定 API 对齐。
- 已通过：
  - `npm run typecheck`：TypeScript 类型检查通过。
  - `npm test`：26 个测试文件、129 个用例通过。
  - `npm run test:stable-t3`：2 个集成测试文件、18 个用例通过。
  - `npx vitest run tests/integration/udbx4spec-compliance.integration.spec.ts`：合规集成入口 7 个用例通过。
  - `npm run test:browser`：Playwright 浏览器 smoke 2 个用例通过。
  - `npm run build`：CJS、ESM 和 DTS 产物生成成功。
  - `npm pack --dry-run --cache /private/tmp/udbx4ts-npm-cache`：发布包 dry-run 成功，包版本 `0.4.0`，23 个文件，约 312.1 kB。
- 当前阻塞：
  - 默认 `npm pack --dry-run` 在本机因 `~/.npm` cache 权限问题失败；本次验证使用临时 cache 目录规避，不影响项目发布包内容。
  - 远端推送和 npm 正式发布仍需具备网络、npm token 和 `udbx4x` 组织发布权限。

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

1. 提交 `udbx4ts v0.4.0` 发布准备改动和根目录发布状态更新。
2. 进入 `udbx4go v0.1.0` 首发准备，补齐版本章节、release notes 和发布门禁验证。
3. 在具备网络和凭据后，推送各仓库 `main` 与 release tag。
4. 按 `udbx4spec -> udbx4j -> udbx4ts -> udbx4go` 顺序完成正式发布。
5. 每个仓库正式发布前复跑对应门禁命令，避免只依赖历史本地结果。
