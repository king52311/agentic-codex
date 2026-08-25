# 用户档案管理功能对比

对比 `product/msApi-0.0.1-SNAPSHOT` 与当前 `omsapi/omsvue`、`omsapi` 源码，梳理用户档案管理页几个核心功能的差异。

## 结论

- 停用、启用、落表、复装的后端主流程基本一致。
- 换表、卡表换 NB 表、补卡的核心接口和入库链路也一致。
- 当前项目主要差异在前端交互集中化、少量兼容字段补齐、以及部分查询条件和文案调整。

## 对照表

| 功能 | product 逻辑 | 当前项目逻辑 | 差异 |
|---|---|---|---|
| 停用 | `UserProfilesController -> /meter/updateMeterState`，`unLoadType=3` | 同样走 `/meter/updateMeterState` | 基本一致 |
| 启用 | `unLoadType=0`，更新 `newsun_meter` 状态，并写 `newsun_meter_stop` | 同样逻辑 | 基本一致 |
| 落表 | `unLoadType=2`，更新状态，并写 `newsun_meter_unload` | 同样逻辑 | 基本一致 |
| 复装 | `unLoadType=4`，恢复状态，并写 `newsun_meter_unload` | 同样逻辑 | 基本一致 |
| 换表 | `/newsunMeter/exchange/insertExchange` | 同样有 `insertExchange` | 主流程一致，当前项目补过兼容字段 |
| 卡表换 NB 表 | `/newsunMeter/exchange/exchangeMeterForSe`、`/insertExchangeBH` | 同样有 | 当前项目前端拆得更清楚 |
| 补卡（炳华） | 先写卡，再回写换卡记录 | 同样有 | 核心一致，页面校验略有差异 |

## 关键流程

### 1. 停用 / 启用 / 落表 / 复装

入口在 `omsvue/src/components/documentmanage/DocuserDocument.vue` 的功能控制区。

后端入口在 `omsapi/src/main/java/com/newsun/controller/NewsunMeterController.java`：

- `updateMeterState(MeterUnLoad mu)`
- 根据 `mu.unLoadType` 分支处理
- 更新 `newsun_meter.ecuState`
- 写入 `newsun_meter_stop` 或 `newsun_meter_unload`

### 2. 换表

入口：

- 前端：`exchange(currentRow)`
- 后端：`/newsunMeter/exchange/insertExchange`

处理：

- 校验旧表状态
- 读取新表信息
- 写换表记录
- 更新旧表 / 新表状态

### 3. 卡表换 NB 表

入口：

- 前端：`exchangeMeterBH(currentRow)`
- 后端：`/newsunMeter/exchange/exchangeMeterForSe`

处理：

- 校验表具类型
- 读取新表信息
- 写换表记录
- 按新表品牌和补发量处理

### 4. 补卡（炳华）

入口：

- 前端：`exchangeCardBH(currentRow)`
- 后端：`/newsunMeter/exchange/insertExchangeBH`

处理：

- 先调用本地写卡接口
- 写卡成功后回写换卡记录

## 主要差异

- 当前项目把用户档案页的操作按钮集中到了一个控制区，交互更统一。
- 当前项目补了部分缺失字段和兼容查询，方便当前数据库直接跑通。
- `product` 中有些历史查询和旧逻辑更散，当前项目已经收敛过。

