# ct-biz-revenue 数据迁移方案

## 目标

新建 `ct-biz-revenue` 作为最终业务库，合并两套来源数据：

- `djwms_new`：新功能表结构、配置数据、菜单权限、业务扩展表；
- `djwms`：旧结构下的最新业务数据，作为用户、表具、账户、流水等主数据来源。

最终目标是：

1. `ct-biz-revenue` 能直接支撑当前系统运行；
2. 新功能表结构和配置完整保留；
3. `djwms` 的最新业务数据全部同步到新库；
4. 数据迁移脚本可重复执行，便于后续正式发版。

## 现有 SQL 升级方式

已扫描 `omsapi/db/migrations/`，当前升级脚本特点如下：

- 采用按日期命名的迭代 SQL；
- 以 `information_schema` 判断表、字段、索引是否存在；
- 结构变更以 `ALTER TABLE`、`CREATE TABLE IF NOT EXISTS` 为主；
- 配置类、菜单类、字典类改动直接写在迁移 SQL 中；
- 现有基线保存在 `db/baseline/djwms-lite.sql`；
- `db/patches/` 主要是阶段性补丁，不作为正式发版顺序依据。

## 迁移原则

### 1. 结构优先于数据

先把 `djwms_new` 的新表、新字段、新索引、新配置补到 `ct-biz-revenue`，再导数据。

### 2. 主数据以 `djwms` 为准

`djwms` 虽然是旧结构，但业务数据更新，用户、账户、表具、账单、流水、历史记录优先按它同步。

### 3. 新功能以 `djwms_new` 为准

新抄表、出账、结算、配置、菜单权限、定时任务等功能表结构和初始化数据，按 `djwms_new` 补齐。

### 4. 所有合并都按业务键，不按自增 ID 直接覆盖

优先使用：

- `profilesId`
- `ecuId`
- `book_code`
- `plan_code`
- `bill_no`
- `codeType + codeValue`
- `menu path + component`

## 建议迁移步骤

### 阶段一：只读比对

同时连接 `djwms`、`djwms_new`、`ct-biz-revenue`，输出：

- 表清单差异；
- 字段差异；
- 索引差异；
- 主键/唯一键差异；
- 行数和更新时间差异；
- 程序引用差异；
- 冲突清单。

建议生成：

- `ct_biz_revenue_schema_diff.md`
- `ct_biz_revenue_data_diff.md`
- `ct_biz_revenue_conflict_report.md`

### 阶段二：补结构

从 `djwms_new` 提取以下内容写入迁移脚本：

- 新增表；
- 新增字段；
- 新增索引；
- 新增字典、菜单、角色、站点配置；
- 新增定时任务配置。

结构脚本要求做成通用比对式，不按固定表清单写死：

- 扫描 `djwms_new` 的所有表、字段、索引；
- `ct-biz-revenue` 缺什么补什么；
- 后续新增表或字段后，直接重跑脚本即可；
- 不自动删除旧字段、旧表和旧索引。

### 阶段三：同步 `djwms` 最新业务数据

重点同步：

- 用户档案；
- 表具与表具最新状态；
- 账户和余额；
- 充值、退费、扣费流水；
- 抄表历史；
- 抄表册、抄表计划、抄表活动；
- 账单批次与账单明细；
- 其他业务唯一键可识别的数据。

### 阶段四：同步 `djwms_new` 新功能配置

重点同步：

- 菜单和权限；
- 角色、用户权限；
- 系统配置；
- 阶梯、价格、违约金配置；
- 自动结算、计划任务、账单查询、收费工作台相关配置；
- 新功能依赖表结构。

### 阶段五：校验

至少校验：

- 关键表总量；
- 主键唯一性；
- 用户-表具-账户关联；
- 充值和账单流水合计；
- 抄表册/计划/活动链路；
- 菜单权限是否齐全；
- 配置项是否能正常读取。

## 配置建议

建议在 `omsapi` 中增加 `ct-biz-revenue` 独立连接配置，至少支持：

- `CT_BIZ_REVENUE_DB_URL`
- `CT_BIZ_REVENUE_DB_USERNAME`
- `CT_BIZ_REVENUE_DB_PASSWORD`

如后续需要迁移工具独立跑批，再补：

- `DJWMS_DB_URL`
- `DJWMS_NEW_DB_URL`

## 落地方式

建议新增独立迁移脚本目录，例如：

```text
omsapi/db/migrations/ct-biz-revenue/
├── 00_precheck.sql
├── 01_schema_from_djwms_new.sql
├── 02_merge_djwms_base_data.sql
├── 03_merge_djwms_new_config_data.sql
├── 04_validation.sql
└── README.md
```

这样可以避免和现有生产迭代 SQL 混在一起，也方便后续只执行迁移脚本，不影响常规小版本升级。

## 具体执行步骤

1. 备份 `ct-biz-revenue` 目标库。
2. 执行 `00_precheck.sql`，确认三个库连接正常。
3. 比对 `djwms_new` 与 `ct-biz-revenue` 的结构差异，生成结构补齐清单。
4. 执行 `01_schema_from_djwms_new.sql`，补齐新表、新字段、新索引和基础配置。
5. 比对 `djwms` 与 `ct-biz-revenue` 的业务数据差异，生成主数据同步清单。
6. 执行 `02_merge_djwms_base_data.sql`，同步旧库最新业务数据。
7. 比对 `djwms_new` 与 `ct-biz-revenue` 的新功能配置差异。
8. 执行 `03_merge_djwms_new_config_data.sql`，补齐菜单、权限、系统配置和定时任务配置。
9. 执行 `04_validation.sql`，核对数据、权限和关联链路。
10. 校验通过后，切系统配置到 `ct-biz-revenue`。

## 注意事项

- 不允许直接覆盖自增 ID。
- 余额、账单、流水必须做幂等判断。
- 迁移期间暂停自动结算和计划任务。
- 菜单、角色、字典和配置优先按业务编码合并。
- 删除字段、删除表、清空数据先单独确认。
