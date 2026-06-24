# SampleData.udbx 公开分发确认

本文档记录 `data/SampleData.udbx` 进入公开仓库、公开测试资产和 T3 source-derived stable 合规夹具的工程准入信息。

## 样本身份

- 样本路径：`data/SampleData.udbx`
- SHA-256：`b3cbac16f48e4e07fec726c781c42feeff8d5fd85dad4f2381fa538f3afbde39`
- 文件大小：`19,841,024` 字节
- 内部更新时间：`SmDataSourceInfo.SmLastUpdateTime=2020-12-11 03:04:03`

## 公开分发授权

- 授权名称：项目维护者确认的 `SampleData.udbx` 公开分发授权
- 授权依据：维护者已确认 `SampleData.udbx` 可以公开
- 授权范围：允许作为 `udbx4x` 公开仓库样本、公开 release 资产、公开测试资产和 T3 source-derived 夹具来源使用
- 授权限制：本文档不是第三方法律意见；如上游提供正式许可证或授权文本，应以正式文本更新本文档

## 生成环境证据

生成产品版本由项目维护者确认：

- 产品名称：`SuperMap iDesktopX 2025`
- 产品版本：`V12.0.1.0`

样本内部系统表还提供以下生成环境证据：

- `SmDataSourceInfo.SmVersion=0`
- `SmDataSourceInfo.SmDataFormat=1`
- `spatialite_history.ver_sqlite=3.24.0`
- `spatialite_history.ver_splite=4.3.0-RC1`
- `spatialite_history.timestamp` 范围始于 `2020-12-11T02:44:16.754Z`

## 脱敏结论

当前进入 `udbx4spec/compliance/source-derived/` 的 T3 派生资产仅包含：

- `County_T` / `SmID=1` 的最小 `SmGeometry` bytes。
- `CADDT` / `SmID=1/16/63` 的最小 `SmGeometry` bytes。
- `BaseMap_PZ`、`BaseMap_LZ`、`BaseMap_RZ` 的系统表摘要和首条 GAIA header 摘要。

这些派生资产不包含完整真实数据库、业务属性表、用户标识字段或可反推出业务语义的完整记录，脱敏状态为 `not-required`。

## stable T3 准入结论

基于维护者公开分发确认、样本 SHA-256 固定、派生资产最小化和无需脱敏结论，以下 T3 source-derived 夹具已提升为 `stable`：

- `sampledata-county-t-smid-1-smgeometry`
- `sampledata-caddt-smid-1-smgeometry`
- `sampledata-caddt-smid-16-smgeometry`
- `sampledata-caddt-smid-63-smgeometry`
- `sampledata-3d-srid-zero-metadata`

后续如果更换源样本、补充正式上游许可证、确认 SuperMap 产品精确版本或扩大派生资产范围，必须同步更新本文档、`docs/samples/README.md`、`docs/guides/source-fixture-admission.md` 和 `udbx4spec/compliance/source-derived/manifest.json`。
