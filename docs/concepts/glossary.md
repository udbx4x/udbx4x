# 术语表

本文档维护 `udbx4x` 工作区的核心术语。新增长期概念前，应优先检查本文档，避免同一概念多种叫法。

## 术语原则

- 中文说明为主。
- 英文术语用于 API、代码、包名、生态检索和跨语言一致性。
- 当中文和英文同时出现时，以中文解释为准。
- 不为旧称呼保留兼容概念；旧称呼应迁移到规范名称。

## 核心项目术语

| 术语 | 英文 / 代码名 | 说明 |
|---|---|---|
| UDBX | Universal Spatial Database Extension | SuperMap 使用的基于 SQLite 的空间数据库扩展格式 |
| 工作区 | workspace | 当前 `udbx4x` 根目录，包含规范、SDK、工具和样本 |
| 规范工程 | `udbx4spec` | 跨语言规范、参考类型、合规夹具和测试矩阵 |
| Java SDK | `udbx4j` | UDBX Java 读写库 |
| TypeScript SDK | `udbx4ts` | UDBX TypeScript 读写库，支持浏览器和 Electron |
| Go SDK | `udbx4go` | UDBX Go 读写库 |
| 工具生态 | tools ecosystem | 建立在 SDK 之上的 viewer、CLI、Copilot、validator、converter 等 |

## 数据源和数据集

| 术语 | 英文 / 代码名 | 说明 |
|---|---|---|
| 数据源 | `DataSource` / `UdbxDataSource` | 一个 UDBX 文件对应的数据源入口 |
| 数据集 | `Dataset` | UDBX 文件中的一个数据表或空间图层 |
| 数据集信息 | `DatasetInfo` | 数据集名称、类型、SRID、对象数等元信息 |
| 字段信息 | `FieldInfo` | 字段名称、类型、别名、是否可空、默认值等元信息 |
| 查询选项 | `QueryOptions` | 分页、ID 过滤、限制数量等查询参数 |
| 要素 | `Feature` | 带几何和属性的空间记录 |
| 纯属性记录 | `TabularRecord` | 无几何的属性表记录 |

## 数据集类型

| 术语 | 规范名 | 值 | 说明 |
|---|---|---:|---|
| 纯属性表 | `tabular` | 0 | 无空间几何 |
| 点数据集 | `point` | 1 | 2D 点 |
| 线数据集 | `line` | 3 | 2D 多线 |
| 面数据集 | `region` | 5 | 2D 多面 |
| 文本数据集 | `text` | 7 | 文本标注数据集 |
| 三维点数据集 | `pointZ` | 101 | 带 Z 坐标的点 |
| 三维线数据集 | `lineZ` | 103 | 带 Z 坐标的多线 |
| 三维面数据集 | `regionZ` | 105 | 带 Z 坐标的多面 |
| CAD 数据集 | `cad` | 149 | SuperMap CAD 复合几何数据集 |

## 字段类型

| 术语 | 规范名 | 值 | SQLite 类型 |
|---|---|---:|---|
| 布尔 | `boolean` | 1 | INTEGER |
| 字节 | `byte` | 2 | INTEGER |
| 16 位整数 | `int16` | 3 | INTEGER |
| 32 位整数 | `int32` | 4 | INTEGER |
| 64 位整数 | `int64` | 5 | INTEGER |
| 单精度浮点 | `single` | 6 | REAL |
| 双精度浮点 | `double` | 7 | REAL |
| 日期 | `date` | 8 | TEXT |
| 二进制 | `binary` | 9 | BLOB |
| 几何字段 | `geometry` | 10 | BLOB |
| 定长字符 | `char` | 11 | TEXT |
| 时间 | `time` | 16 | TEXT |
| Unicode 长文本 | `ntext` | 127 | TEXT |
| 长文本 | `text` | 128 | TEXT |

## 几何术语

| 术语 | 英文 / 代码名 | 说明 |
|---|---|---|
| GeoJSON-like | GeoJSON-like geometry | 跨语言几何交换模型，允许扩展 `srid`、`hasZ`、`bbox` |
| 点 | `Point` | 坐标为 `[x, y]` 或 `[x, y, z]` |
| 多线 | `MultiLineString` | 线数据集的规范几何模型 |
| 多面 | `MultiPolygon` | 面数据集的规范几何模型 |
| 坐标系 ID | `srid` | 空间参考 ID |
| 三维标记 | `hasZ` | 是否包含 Z 坐标 |
| 外包框 | `bbox` | 几何范围 |
| GAIA Geometry | GAIA | SpatiaLite 风格二进制几何编码 |
| CAD Geometry | CAD / GeoHeader | SuperMap CAD 自定义二进制几何编码 |

## 系统表

| 表名 | 说明 |
|---|---|
| `SmRegister` | 数据集注册信息 |
| `SmFieldInfo` | 字段元信息 |
| `geometry_columns` | 空间列注册信息 |
| `SmDataSourceInfo` | 数据源级元信息 |

## 旧称呼迁移

| 旧称呼 | 规范称呼 | 说明 |
|---|---|---|
| `DatasetType` | `DatasetKind` | Java v2 统一为规范名称 |
| `smId` | `id` | Feature / Record 的规范主键名称 |
| `getFeatures()` | `list()` | 查询列表 |
| `getFeature()` | `getById()` | 按 ID 查询 |
| `addFeature()` | `insert()` | 插入单条 |
| `addFeaturesBatch()` | `insertMany()` | 批量插入 |
| `getCount()` | `count()` | 统计数量 |

