# 环境配置规范

本文档定义 `udbx4x` 工作区的本地开发环境约定。

## 总体原则

- 以本地构建和本地测试为准。
- 每个子项目应能独立运行本地测试。
- 发布前必须运行目标项目的本地发布门禁。
- 不依赖维护者个人机器上的隐式状态。

## 基础工具

建议安装：

- Git
- Java 17
- Maven 3.6+
- Node.js 24+
- npm
- Go 1.22+
- C 编译器，供 `go-sqlite3` 使用

## Java 环境

`udbx4j` 要求 Java 17。

日常开发推荐命令：

```bash
cd udbx4j
make test
make test-all
```

若系统默认 JDK 不是 Java 17，应显式设置 `JAVA_HOME`。本项目已有 Makefile 处理本机 Java 17 路径时，应优先使用 Makefile。

## TypeScript 环境

`udbx4ts` 和 `udbx-copilot` 使用 Node.js 和 npm。

日常开发推荐命令：

```bash
cd udbx4ts
npm install
npm run typecheck
npm test
npm run build
```

浏览器运行时变更需要额外运行：

```bash
npm run test:browser
```

## Go 环境

`udbx4go` 使用 Go 1.22 和 `github.com/mattn/go-sqlite3`，因此需要 CGO 和 C 编译器。

日常开发推荐命令：

```bash
cd udbx4go
make test
go test ./...
```

## 样本文件

当前工作区样本：

- `data/SampleData.udbx`
- `data/henan.udbx`

样本文件用于本地兼容性验证。公开分发或进入 CI 前，必须补齐来源、授权和 SHA-256。

## 环境变量

默认测试不应依赖私有环境变量。

仅在使用远程 planner 或发布渠道时允许使用凭据类环境变量，例如：

- `OPENAI_API_KEY`
- `ZHIPU_API_KEY`
- npm token
- Maven Central credentials

凭据不得写入仓库。

## 本地检查顺序

改动多个子项目时建议按顺序运行：

```bash
cd udbx4j && make test
cd ../udbx4ts && npm run typecheck && npm test && npm run build
cd ../udbx4go && make test
```

若修改 `udbx4spec`，还应运行或补齐对应合规检查。公开发布前必须改按根目录 `RELEASE.md` 执行完整发布门禁，不能只依赖本文件中的日常开发命令。
