# 发布流程

本文档定义 UDBX4X 各包的本地发布门禁。任何公开版本发布前都必须先通过本文档列出的强制检查；任一强制门禁失败时，不得发布。

公开发布渠道：

| 项目 | 渠道 | 产物 |
|---|---|---|
| `udbx4j` | Maven Central | `io.github.zhyt1985:udbx4j` |
| `udbx4ts` | npm | `udbx4ts` |
| `udbx4go` | pkg.go.dev | `github.com/udbx4x/udbx4go` 的 Git tag |

## 发布原则

- 发布必须可在本地复现。
- 发布前必须写好 changelog。
- 公开 API 变更必须写文档。
- 跨语言行为变更必须体现在 `udbx4spec` 中。
- 已知缺口必须明确列出。
- release tag 必须与包版本一致。
- 发布说明必须区分“当前最小合规基线已支持”和“真实世界完整兼容范围仍需扩展”。

## 全局发布前检查

- `README.md` 是最新的。
- `ROADMAP.md` 与本次发布不矛盾。
- 目标项目的 `CHANGELOG.md` 已更新。
- 行为变化已更新兼容性或合规报告。
- 目标项目本地测试通过。
- 版本号一致。
- Release notes 提到已知限制。
- 若 release notes 声明真实样本兼容范围，必须同步运行对应真实样本专项测试，并更新 `docs/samples/compatibility-matrix.md`。

## 全局强制门禁

所有 SDK 发布前都必须先验证 `udbx4spec` 合规资产。生成和检查命令必须串行执行，避免 roundtrip 夹具读写竞争。

```bash
cd udbx4spec
node tools/generate-golden-gaia-bytes.mjs
node tools/generate-golden-text-bytes.mjs
node tools/generate-compliance-db.mjs
node tools/generate-roundtrip-fixtures.mjs
node tools/check-fixtures.mjs
```

发布前必须确认：

- `compliance/compliance.udbx` 覆盖 `point/line/region/pointZ/lineZ/regionZ/tabular/cad/text`。
- `compliance/roundtrip/manifest.json` 中三端 roundtrip 夹具都包含 `test_text`。
- `compliance/source-derived/manifest.json` 中所有 `stability=stable` 的 T3 夹具都带有 `licenseName`、`licenseDocument`、`generator`、`deidentification`，并参与三端发布门禁。
- `expectedGeometryColumns` 与当前规范一致。
- 三端合规报告已同步最新状态。
- 若本次修改了格式、字段、DatasetKind、FieldType、GeoText、CAD 或系统表规则，必须先更新 `udbx4spec` 文档和夹具，再发布 SDK。

## Java：`udbx4j`

必要检查：

```bash
cd udbx4j
make test
make test-all
make test-stable-t3
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Dgroups=integration -Dit.test=Udbx4SpecComplianceDatabaseReadTest failsafe:integration-test failsafe:verify
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Dgroups=integration -Dit.test=Udbx4SpecComplianceDatabaseRoundtripTest failsafe:integration-test failsafe:verify
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Dtest=SmRegisterDaoSpecTest test
env JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home PATH=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin:/Users/zhangyuting/apache-maven-3.9.9/bin:/usr/bin:/bin:/usr/sbin:/sbin mvn -Djacoco.skip=true -Dit.test=SampleDataTextDatasetReadTest,Vector3DRealSampleReadTest test-compile failsafe:integration-test failsafe:verify
```

发布前 dry run：

```bash
cd udbx4j
mvn -DskipTests package
```

包元数据：

- Group ID：`io.github.zhyt1985`
- Artifact ID：`udbx4j`
- 发布渠道：Maven Central

发布命令：

```bash
cd udbx4j
mvn clean deploy
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
gh release create vX.Y.Z
```

发布前确认：

- `pom.xml` 版本正确。
- `CHANGELOG.md` 有带日期的发布章节。
- `udbx4spec-compliance-report.md` 已更新。
- release maintainer 具备 Maven Central 凭据。
- GPG 签名、sources jar、javadoc jar 可正常生成。
- 干净的下游 Maven 项目可以消费该产物。

## TypeScript：`udbx4ts`

必要检查：

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:stable-t3
npx vitest run tests/integration/udbx4spec-compliance.integration.spec.ts
npm run build
```

涉及浏览器运行时变更时：

```bash
cd udbx4ts
npm run test:browser
```

发布前 dry run：

```bash
cd udbx4ts
npm pack --dry-run
```

发布前确认：

- `package.json` 版本正确。
- `CHANGELOG.md` 有带日期的发布章节。
- `README.md` 与 `README.en.md` 已同步。
- `udbx4spec-compliance-report.md` 已更新。
- `udbx4ts`、`udbx4ts/web`、`udbx4ts/electron` 的 `exports` 正确。
- 包内容只包含预期文件。
- `npm pack --dry-run` 输出中必须包含 `dist/` 和类型声明，不应包含测试临时产物。

发布：

```bash
cd udbx4ts
npm publish
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
```

## Go：`udbx4go`

必要检查：

```bash
cd udbx4go
make test
make test-stable-t3
go test ./...
GOCACHE=/private/tmp/udbx4go-gocache go test ./internal/codec ./internal/dataset . -run 'GeoText|Text|CAD|Cad|Udbx4Spec|RealSample' -v
```

可选检查：

```bash
cd udbx4go
go test -race ./...
go vet ./...
```

发布前 dry run：

```bash
cd udbx4go
go list ./...
go test ./...
```

打 tag 前确认：

- `CHANGELOG.md` 有带日期的发布章节。
- 公开 API 文档清晰。
- `README.md` 与 `README.zh.md` 已同步。
- `udbx4spec-compliance-report.md` 已更新。
- 已知缺口已记录。
- `go.mod` module path 正确。
- tag 格式使用 `vX.Y.Z`，并与对外发布说明一致。

发布：

```bash
cd udbx4go
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
```

打 tag 后确认 pkg.go.dev 能解析新版本。

如果 pkg.go.dev 未自动刷新，可访问：

```text
https://pkg.go.dev/github.com/udbx4x/udbx4go@vX.Y.Z
```

## 工具发布

`udbx-copilot`、viewer 等工具在依赖的 SDK 能力稳定前，不应声明为稳定版本。

工具发布必须说明：

- SDK 依赖版本。
- 支持的 `.udbx` 操作。
- 只读或写入行为。
- 已知不支持的问题或数据集。
