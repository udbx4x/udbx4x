# UDBX4X 路线图

## 使命

UDBX4X 提供用于读取、写入、校验和检查 UDBX 空间数据库文件的开源 SDK 与工具。

## 产品方向

短期方向：建设成熟的 Java、TypeScript、Go 开源 SDK。

长期方向：在 SDK 之上建设完整的 UDBX 开源工具生态，包括查看器、校验器、转换器、CLI 和开发者工具。

## 优先级顺序

1. 将 `udbx4spec` 建设为可执行规范，补齐夹具、golden bytes 和合规报告。
2. 强化 `udbx4j` 和 `udbx4ts` 作为首批核心 SDK 的发布质量。
3. 推动 `udbx4go` 达到完整规范合规。
4. 标准化文档、示例和发布流程。
5. 发展基于 SDK 的 viewer、CLI、validator、converter、Copilot 等工具。

## 成熟度等级

| 等级 | 目标 | 退出条件 |
|---|---|---|
| L0 盘点 | 理清当前工作区状态 | 能力矩阵和风险登记存在 |
| L1 规范契约 | 让 `udbx4spec` 可执行 | golden bytes、夹具清单、合规数据库存在 |
| L2 SDK 一致性 | 对齐 Java、TypeScript、Go 行为 | 跨语言读写矩阵通过 |
| L3 公开发布 | 发布流程可重复 | Maven、npm、Go 发布检查可本地通过 |
| L4 开发者采用 | 让开发者容易使用 | Quickstart、样本、排障文档存在 |
| L5 工具生态 | 建设稳定 SDK 工具 | 工具有独立测试、文档和发布节奏 |

## 近期里程碑

### M1：P0 文档

必要文档：

- `README.md`
- `ROADMAP.md`
- `GOVERNANCE.md`
- `CONTRIBUTING.md`
- `RELEASE.md`
- `SECURITY.md`

退出条件：贡献者可以理解项目是什么、如何决策、如何贡献、如何发布。

### M2：可执行规范

必要工作：

- 增加夹具 manifest。
- 增加 GAIA golden bytes。
- 增加合规报告格式。
- 定义跨语言 roundtrip 矩阵。
- 建立首批来自 `SampleData.udbx` 的 T3 `stable` source-derived 夹具。

退出条件：每个 SDK 都能基于同一套规范资产运行本地合规检查。

### M3：SDK 发布强化

必要工作：

- 明确 `udbx4j` 发布状态。
- 明确 `udbx4ts` 发布状态。
- 在 `udbx4go` 已完成 Text 能力和三端 Text 闭环、首批 T3 `stable` 夹具已建立的基础上，继续扩大真实样本兼容范围并收紧发布门禁。
- 统一 SDK quickstart。

退出条件：Java、TypeScript、Go SDK 都有清晰发布门禁和兼容状态。

### M4：跨语言一致性

必要工作：

- Java 写，TypeScript 读。
- Java 写，Go 读。
- TypeScript 写，Java 读。
- TypeScript 写，Go 读。
- Go 写，Java 读。
- Go 写，TypeScript 读。

退出条件：读写差异被测试、记录，并被修复或列为已知缺口。

### M5：工具生态

必要工作：

- 对齐 `udbx4go-viewer` 文档与实际 Wails 实现。
- 稳定 `udbx-copilot` 作为 SDK 消费者的定位。
- 定义 validator 和 converter 的边界。

退出条件：工具依赖 SDK，不重复实现二进制解析逻辑。

## 兼容策略

UDBX 白皮书是最高格式依据。真实 SuperMap 生成的 `.udbx` 文件是兼容性证据。`udbx4spec` 将这些依据转化为工程化的 API、夹具和测试。

当行为冲突时，先更新规范并记录决策，再修改 SDK 代码。
