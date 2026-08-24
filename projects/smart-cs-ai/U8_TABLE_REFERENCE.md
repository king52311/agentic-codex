# U8 数据库表说明

用于整理当前仓库里已经实际接入或明确查询过的 U8 表，方便后续按“业务功能 -> 表 -> 关键字段”快速定位。

当前项目 U8 账套配置：

- 数据库：`UFDATA_001_2019`
- 架构：`dbo`

## 1. 供应商主数据

### `dbo.Vendor`

用途：

- 供应商主档
- 供应商注册时校验企业名称是否存在
- 采购部供应商管理列表主数据来源
- 供应商联系人、电话、采购负责人等信息维护

关键字段：

- `cVenCode`：供应商编码
- `cVenName`：供应商名称
- `cVenAbbName`：供应商简称
- `cVenPerson`：联系人
- `cVenPhone`：电话
- `cVenFax`：传真
- `cVenEmail`：邮箱
- `cCreatePerson`：建档人/采购负责人
- `dModifyDate`：最近修改时间
- `dVenCreateDatetime`：创建时间

当前代码用途：

- 供应商注册按 `cVenName` 校验企业是否存在
- 采购部 `供应商管理` 列表直接读取 `Vendor`
- 采购员下拉会读取 `cCreatePerson`
- 供应商编辑会回写联系人、电话、采购负责人

相关代码：

- `backend/app/api/endpoints/auth.py`
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`
- `backend/app/api/endpoints/supplier.py`
- `backend/tasks/sync_vendor_to_mysql.py`

## 2. 采购订单/采购台账

### `dbo.pu_RelPomain`

用途：

- 采购订单主表/采购台账主表
- 当前项目里判断供应商“活跃度”的核心表
- 查询最近一笔采购订单、最近下单日期、下单次数

关键字段：

- `id`：主键
- `ccode`：采购单号
- `dDate`：单据日期
- `cVerifier`：审核人
- `cptname`：采购类型
- `cbustype`：业务类型
- `cvenabbname`：供应商简称/名称
- `cvencode`：供应商编码
- `cdepname`：部门名称
- `cpersonname`：业务员
- `cwhname`：仓库名称

当前代码用途：

- 供应商活跃度判断：
  - 最近配置周期内是否有订单
  - 最近一次下单时间
  - 配置周期内下单次数
- 台账管理数据源
- 查询“今天最新一笔采购单号及供应商”

相关代码：

- `backend/app/services/sqlserver.py`
- `backend/tasks/sync_pu_relpomain_from_sqlserver.py`
- `backend/app/models/procurement_dept.py`

## 3. 采购订单明细

### `dbo.PO_Podetails`

用途：

- 采购订单明细表
- 台账管理里与采购主表联查
- 可用于统计金额、税率、到货日期等明细信息

关键字段：

- `POID`：关联 `pu_RelPomain.id`
- `cInvCode`：存货编码
- `iInvMoney`：金额
- `iPerTaxRate`：税率
- `dArriveDate`：到货日期
- `iReceivedMoney`：收货/收料金额

当前代码用途：

- 与 `pu_RelPomain`、`rdrecords32` 联查生成采购台账
- 用于展示金额、税率、到货日期等字段

相关代码：

- `backend/app/services/sqlserver.py`

## 4. 入库/物料关联辅助表

### `dbo.rdrecords32`

用途：

- 当前项目里主要作为台账联查辅助表
- 用于补充物料编码、物料名称

关键字段：

- `cInvCode`：存货编码
- `cItemCode`：项目编码
- `cName`：项目/物料名称
- `AutoID`：记录主键

当前代码用途：

- 与 `PO_Podetails.cInvCode` 联查
- 台账管理里展示项目编码、项目名称

相关代码：

- `backend/app/services/sqlserver.py`

## 5. 存货主数据

### `dbo.Inventory`

用途：

- U8 存货/物料主数据
- 当前项目主要用于采购部备件库

关键字段：

- `cInvCode`：存货编码
- `cInvName`：存货名称
- `cInvStd`：规格型号
- `cInvCCode`：存货分类编码
- `iTaxRate`：税率
- `iInvNCost`：无税参考成本
- `cCreatePerson`：建档人
- `cModifyPerson`：变更人

当前代码用途：

- 采购部备件库列表数据源
- 备件同步到本地 `procurement_spare_parts`

相关代码：

- `backend/app/services/sqlserver.py`
- `backend/tasks/sync_spare_parts_from_sqlserver.py`

### `dbo.InventoryClass`

用途：

- 存货分类表
- 给 `Inventory` 提供分类名称

关键字段：

- `cInvCCode`：存货分类编码
- `cInvCName`：存货分类名称

当前代码用途：

- 备件库分类筛选
- 与 `Inventory.cInvCCode` 联查

相关代码：

- `backend/app/services/sqlserver.py`

## 6. 项目目录

### `dbo.fitemss00`

用途：

- U8 项目目录/项目字典
- 项目相关下拉和映射数据源

关键字段：

- `I_id`：项目ID
- `citemcode`：项目编码
- `citemname`：项目名称
- `citemccode`：项目分类代码
- `bclose`：是否关闭

当前代码用途：

- 同步到本地 MySQL `fitemss00` 镜像表
- 供后续项目类数据选择和映射使用

相关代码：

- `backend/tasks/sync_fitemss00_from_sqlserver.py`
- `backend/app/models/procurement_dept.py`

## 7. 发票表

### `dbo.PurInVoice`

用途：

- 历史上用于按供应商统计采购发票数量、最近发票日期

关键字段：

- `cVenCode`：供应商编码
- `dPBVDate`：发票日期

当前情况说明：

- 仓库里仍保留过这张表的查询逻辑
- 但当前实际连接的 `UFDATA_001_2019` 账套里，这张表查询时报“对象名无效”
- 所以当前项目里供应商活跃度已经改为按 `pu_RelPomain` 采购订单判断，不再依赖这张表

相关代码：

- `backend/app/services/sqlserver.py`
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`

## 8. 本地镜像表对应关系

下面这些不是 U8 原表，但和 U8 表一一对应，排查时经常一起看：

| 本地 MySQL 表 | U8 源表 | 说明 |
| --- | --- | --- |
| `Vendor` | `dbo.Vendor` | 供应商主档镜像 |
| `pu_RelPomain` | `dbo.pu_RelPomain` | 采购台账镜像 |
| `fitemss00` | `dbo.fitemss00` | 项目目录镜像 |
| `procurement_spare_parts` | `dbo.Inventory` + `dbo.InventoryClass` | 备件库镜像 |

## 9. 以后查表建议

如果后面要继续查 U8，建议优先按下面顺序判断：

1. 查供应商信息：先看 `Vendor`
2. 查最近订单、活跃度、采购单：先看 `pu_RelPomain`
3. 查订单金额、税率、到货日期：再看 `PO_Podetails`
4. 查物料、项目编码名称：再联 `rdrecords32`
5. 查备件/存货：看 `Inventory`、`InventoryClass`
6. 查项目目录：看 `fitemss00`

## 10. 当前已确认的几个业务结论

- 当前账套业务数据已经更新到 `2026-06-26`
- “最新采购单”可以从 `pu_RelPomain` 查
- “最活跃供应商”目前建议按 `pu_RelPomain` 统计
- `PurInVoice` 在当前账套不可用，不建议继续作为主判断依据
