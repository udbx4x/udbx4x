# 开源 GIS 成熟度路线实施计划

本文档是 2026-06-22 形成的历史实施计划摘要，用于保留当时的路线判断和阶段拆分。当前长期规则以根目录 `AGENTS.md`、`ROADMAP.md`、`GOVERNANCE.md`、`CONTRIBUTING.md`、`RELEASE.md`、`SECURITY.md` 以及 `docs/` 下的规范文档为准。

## 目标

短期目标是把 `udbx4x` 建设为成熟、高质量、可发布的开源 UDBX SDK 体系：

- `udbx4spec` 作为跨语言规范和合规测试入口。
- `udbx4j` 面向 Maven Central 发布。
- `udbx4ts` 面向 npm 发布。
- `udbx4go` 面向 pkg.go.dev 发布。

中长期目标是在 SDK 之上建设完整工具生态，包括 viewer、CLI、validator、converter、copilot 等工具。工具层必须依赖 SDK，不得复制底层格式解析逻辑。

## 核心原则

1. 规范优先：公开 API、格式语义和跨语言行为先进入 `udbx4spec`，再进入语言实现。
2. 证据优先：白皮书是最高依据，真实 `.udbx` 样本是兼容性证据。
3. SDK 优先：先稳定 SDK，再建设工具生态。
4. 一致性优先：Java、TypeScript、Go 必须围绕同一概念模型和同一合规夹具演进。
5. 本地可复现：构建、测试、发布前检查必须能在本地执行。
6. 开源治理优先：路线图、治理、贡献、发布、安全和样本来源都是项目成熟度的一部分。

## 成熟度阶段

| 阶段 | 目标 | 退出标准 |
|---|---|---|
| L0 盘点基线 | 理清项目结构、能力状态和主要风险 | 能力矩阵和风险登记存在 |
| L1 可执行规范 | `udbx4spec` 不只是文档，还包含夹具和检查工具 | fixture manifest、golden bytes、合规报告规则存在 |
| L2 SDK 对齐 | Java、TypeScript、Go 通过同一合规语料 | 跨语言读写矩阵通过 |
| L3 公开发布 | Maven、npm、pkg.go.dev 发布流程可重复 | 发布清单、版本号、变更记录和本地门禁齐全 |
| L4 开发者采用 | GIS 开发者能快速接入 SDK | README、示例、样本说明和排障文档齐全 |
| L5 工具生态 | viewer、CLI、copilot、validator 等工具稳定依赖 SDK | 工具有独立测试、发布边界和文档 |

## 已落实的文档建设

以下根目录和工作区文档已经建立或改造：

- `AGENTS.md`：全工作区总规则。
- `README.md`：项目入口。
- `ROADMAP.md`：公开路线图。
- `GOVERNANCE.md`：治理和权限。
- `CONTRIBUTING.md`：贡献流程。
- `RELEASE.md`：发布流程。
- `SECURITY.md`：安全策略。
- `docs/architecture/overview.md`：整体架构。
- `docs/architecture/compatibility-matrix.md`：能力矩阵。
- `docs/architecture/risk-register.md`：风险登记。
- `docs/architecture/tool-backlog.md`：工具生态待办。
- `docs/concepts/glossary.md`：术语表。
- `docs/samples/README.md`：样本文件规则。
- `docs/standards/`：开发、API、环境、国际化、模块、部署、安全规范。
- `docs/guides/`：本地开发、合规测试、样本管理、发布指南。
- `docs/troubleshooting/README.md`：排障入口。

## 当前优先级

1. 补齐 `udbx4spec` 可执行合规资产，包括真实 fixture、golden bytes、manifest 和报告格式。
2. 建立跨语言 roundtrip 测试矩阵，验证 Java、TypeScript、Go 的读写一致性。
3. 加固 `udbx4j` 与 `udbx4ts` 的发布门禁和公开文档。
4. 在 Text / GeoText 与 CAD 最小基线三端闭环的基础上，继续扩大真实 SuperMap 主流样本兼容覆盖。
5. 稳定 `udbx4go-viewer`、`udbx-copilot` 等工具，确保工具层依赖 SDK。

## 历史清理结论

早期与当前实现冲突的 viewer 方案文档已经清理。`udbx4go-viewer` 当前以 `udbx4go/cmd/udbx4go-viewer/README.md` 中记录的 Wails v2 + React + TypeScript 实现为准。

根目录长期文档不再保留分散的临时 TODO、项目总结或宣传稿。仍有价值的信息应迁移到：

- 架构和风险：`docs/architecture/`
- 概念和术语：`docs/concepts/`
- 开发规范：`docs/standards/`
- 操作指南：`docs/guides/`
- 故障排查：`docs/troubleshooting/`
- 模块文档：对应模块目录下

## 后续维护规则

- 本文档只作为历史计划摘要保留，不作为最新执行清单。
- 新实施计划应放入 `docs/superpowers/plans/`，并明确创建日期、目标、范围、状态和失效条件。
- 如果计划内容已经沉淀为长期规则，应迁移到 `AGENTS.md` 或 `docs/` 对应目录。
- 如果计划内容已经完成且没有长期参考价值，应删除，不长期堆积。
