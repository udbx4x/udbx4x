# 发布工作流

本文档是根目录 `RELEASE.md` 的操作化补充，说明发布前后建议执行的顺序。

发布前应先查看 [发布准备状态](./release-readiness.md)，确认目标仓库的版本号、changelog、tag、门禁命令和已知阻塞项。

## 总体原则

- 先验证，后发布。
- 先补文档，后打 tag。
- 先确认规范一致，再发布 SDK。
- 任何强制门禁失败都必须阻断发布。

## 建议顺序

### 1. 明确发布范围

先判断发布对象：

- `udbx4j`
- `udbx4ts`
- `udbx4go`
- 工具生态

### 2. 检查是否涉及规范变化

若本次发布包含以下内容，必须先检查 `udbx4spec`：

- 公开 API 变化
- 数据模型变化
- DatasetKind 变化
- FieldType 变化
- 几何编解码变化
- 错误分类变化

强制执行：

```bash
cd udbx4spec
node tools/generate-golden-gaia-bytes.mjs
node tools/generate-golden-text-bytes.mjs
node tools/generate-compliance-db.mjs
node tools/generate-roundtrip-fixtures.mjs
node tools/check-fixtures.mjs
```

这些命令必须串行执行。若生成 roundtrip 夹具后检查失败，应先判断是规范、SDK、夹具还是并发读写问题，不得绕过检查发布。

### 3. 更新用户文档

至少检查：

- 根 `README.md`
- 子项目 `README.md`
- `CHANGELOG.md`
- 已知缺口说明

### 4. 运行本地发布门禁

#### Java

```bash
cd udbx4j
make test
make test-all
make test-stable-t3
make package
```

若必须直接调用 Maven，必须显式使用 Java 17；否则 `mvn package -DskipTests` 在默认 JDK 低于 17 时会失败。

#### TypeScript

```bash
cd udbx4ts
npm run typecheck
npm test
npm run test:stable-t3
npx vitest run tests/integration/udbx4spec-compliance.integration.spec.ts
npm run build
npm pack --dry-run
```

若涉及浏览器运行时：

```bash
npm run test:browser
```

#### Go

```bash
cd udbx4go
make test
make test-stable-t3
go test ./...
GOCACHE=/private/tmp/udbx4go-gocache go test ./internal/codec ./internal/dataset . -run 'GeoText|Text|CAD|Cad|Udbx4Spec|RealSample' -v
go list ./...
```

### 5. 发布前人工核对

必须核对：

- 版本号、tag、changelog 和 release notes 一致。
- README、合规报告、能力矩阵和风险登记不含过期状态。
- Text / GeoText 与 CAD 的描述明确为当前最小合规基线，不夸大为完整真实世界兼容。
- 若声明真实样本兼容范围，真实样本专项测试必须通过，且 `docs/samples/compatibility-matrix.md` 已同步。
- T3 source-derived stable 夹具必须带有 `licenseName`、`licenseDocument`、`generator` 和脱敏结论，并且三端 `test-stable-t3` 门禁必须通过。
- 发布账号具备 Maven Central、npm 或 Git tag 权限。

### 6. 执行发布

实际发布命令以 `RELEASE.md` 和子项目发布脚本为准。

### 7. 发布后验证

验证：

- Maven Central / npm / pkg.go.dev 是否可见新版本
- 下游最小示例是否可消费
- release note 是否完整
- 安装或引用公开产物后能打开 `udbx4spec/compliance/compliance.udbx`

## 常见错误

- 未更新 changelog 就发布
- 规范变了但没更新 `udbx4spec`
- 工具发布早于依赖 SDK 稳定
- 版本号和 tag 不一致
- 只跑单语言测试，没有跑跨语言 roundtrip
