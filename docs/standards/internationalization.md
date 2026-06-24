# 国际化规范

本文档定义 `udbx4x` 的语言和国际化规则。

## 默认语言

本项目协作交流语言以简体中文为主。

根目录长期文档默认使用简体中文，包括：

- `README.md`
- `ROADMAP.md`
- `GOVERNANCE.md`
- `CONTRIBUTING.md`
- `RELEASE.md`
- `SECURITY.md`
- `AGENTS.md`

## 允许使用英文的情况

以下内容可以使用英文：

- 代码。
- API 名称。
- 类型名。
- 包名。
- 命令。
- 配置项。
- 错误码。
- 协议字段。
- npm、Maven、Go module 元数据。
- Git commit message。
- Git tag。
- 第三方库和上游标准专有名词。

## 英文文档

面向国际开源社区时，可以维护英文入口，例如：

- `README.en.md`
- 英文 API 文档。
- 英文 release notes。

英文文档应作为并列翻译入口，不应替代中文主文档。

## 术语翻译

长期术语必须进入 `docs/concepts/glossary.md`。

同一概念只能有一个主要中文译名。

示例：

| 英文 | 中文 |
|---|---|
| Dataset | 数据集 |
| DataSource | 数据源 |
| Feature | 要素 |
| FieldType | 字段类型 |
| DatasetKind | 数据集类型 |
| Geometry | 几何 |
| Compliance | 合规 |
| Fixture | 夹具 |

## API 命名

API 命名使用英文，并遵循语言生态惯例。

中文文档中第一次出现 API 名称时，应说明含义。

示例：

```text
`insertMany` 表示批量插入，所有实现都必须使用事务。
```

## 冲突处理

当中文说明和英文术语存在理解差异时，以中文解释和 `udbx4spec` 规范为准。

如果英文术语来自第三方标准，应保留英文原名，并提供中文解释。

