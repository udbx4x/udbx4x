# 部署规范

本文档定义 `udbx4x` 各项目的发布渠道和部署边界。具体发布门禁、dry run、tag 和发布后验证命令以根目录 `RELEASE.md` 为准；本文档不重复维护完整命令清单。

## 发布对象

| 项目 | 发布渠道 | 说明 |
|---|---|---|
| `udbx4j` | Maven Central | Java SDK |
| `udbx4ts` | npm | TypeScript SDK |
| `udbx4go` | Git tag / pkg.go.dev | Go SDK |
| viewer / CLI | 待定 | 工具生态，依赖 SDK 稳定性 |

## 发布前原则

- 本地测试必须通过。
- 版本号必须明确。
- changelog 必须更新。
- README 必须与发布能力一致。
- 已知限制必须写入 release notes。
- 跨语言行为变更必须更新 `udbx4spec`。
- 所有 SDK 发布前必须执行 `RELEASE.md` 中的全局 `udbx4spec` 合规资产校验和对应 SDK 门禁。

## Java 发布

`udbx4j` 发布到 Maven Central，包坐标为 `io.github.zhyt1985:udbx4j`。

发布检查、dry run、`mvn deploy`、tag 和 GitHub Release 操作以 `RELEASE.md` 的 Java 章节为准。

发布后验证：

- Maven Central 能检索到产物。
- 干净项目能引入依赖。
- README 中的版本号正确。

## TypeScript 发布

`udbx4ts` 发布到 npm，包名为 `udbx4ts`，对外入口包括 `udbx4ts`、`udbx4ts/web`、`udbx4ts/electron`。

发布检查、`npm pack --dry-run`、`npm publish` 和 tag 操作以 `RELEASE.md` 的 TypeScript 章节为准。

发布后验证：

- npm 页面显示新版本。
- `exports` 可用。
- 类型声明可用。
- 浏览器和 Electron 入口说明准确。

## Go 发布

`udbx4go` 通过 Git tag 发布到 pkg.go.dev，module path 为 `github.com/udbx4x/udbx4go`。

发布检查、dry run、tag 和 pkg.go.dev 验证以 `RELEASE.md` 的 Go 章节为准。

发布后验证：

- pkg.go.dev 能解析新版本。
- Go API 文档可读。
- README 与当前能力一致。

## 工具发布

工具发布必须说明：

- 依赖的 SDK 版本。
- 支持的操作。
- 是否只读。
- 是否会修改输入文件。
- 已知不支持的数据集类型。

工具不得早于依赖 SDK 的关键能力稳定发布。

## 回滚

包一旦公开发布，不应删除版本作为常规回滚手段。

如发布存在问题：

1. 记录问题。
2. 发布修复版本。
3. 在 changelog 和 release notes 中说明影响。
