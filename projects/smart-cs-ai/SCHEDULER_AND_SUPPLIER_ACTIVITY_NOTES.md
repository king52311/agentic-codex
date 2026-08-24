# 计划任务与供应商活跃度说明

用于整理当前系统里的计划任务列表，并说明供应商“活跃度”刷新、排序和数据库字段现状，方便后续排查和继续演进。

## 1. 当前计划任务列表

当前 APScheduler 注册的任务如下：

| 任务ID | 中文名称 | 执行周期 | 作用说明 |
| --- | --- | --- | --- |
| `knowledge_sync` | 知识库同步 | 每 `KNOWLEDGE_SYNC_INTERVAL_MINUTES` 分钟 | 从 Coze 同步知识库及文档状态，更新后台知识库数据 |
| `u8_vendor_sync` | U8供应商同步 | 每天 `01:30` | 将 U8 `dbo.Vendor` 同步到本地 MySQL `Vendor` 表 |
| `procurement_supplier_risk_refresh` | 供应商风险刷新 | 每天 `02:00` | 扫描采购供应商风险并刷新风险缓存 |
| `procurement_supplier_activity_refresh` | 供应商活跃度刷新 | 每天 `02:30` | 按 U8 采购订单刷新供应商活跃状态、最近下单时间、周期内下单次数 |
| `procurement_contract_sync` | 采购合同同步 | 每天 `04:00` | 从 Ecology 合同视图同步到本地合同同步表 |
| `software_work_record_reminder` | 软件部日报提醒 | 每天 `09:00` | 检查昨天未填写工作记录的员工，并推送提醒 |

代码位置：

- `backend/app/tasks/scheduler.py`
- `backend/app/api/endpoints/admin/scheduler.py`

## 2. 是否已有“更新活跃状态”功能

有，已经存在，现已拆分为独立计划任务：

- 任务ID：`procurement_supplier_activity_refresh`
- 执行时间：每天 `02:30`
- 实际流程：
  1. 遍历 `Vendor`
  2. 按供应商编码读取 U8 采购订单统计
  3. 更新本地 `procurement_suppliers` 的活跃缓存字段

也就是说：

- 现在已经有单独的“活跃度刷新任务”
- 风险刷新和活跃度刷新已分开执行

关键代码：

- `backend/app/tasks/scheduler.py`
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`

## 3. 活跃度最新规则

当前最新规则已经匹配为：

- 最近配置周期内有采购订单：`活跃`
- 最近配置周期内没有采购订单：`不活跃`

当前配置项：

- `ACTIVE_SUPPLIER_ORDER_DAYS`
- 默认值：`720`

当前数据来源：

- U8：`dbo.pu_RelPomain`

当前缓存写入字段：

- `procurement_suppliers.is_active`
- `procurement_suppliers.active_invoice_count`
- `procurement_suppliers.last_invoice_at`
- `procurement_suppliers.active_updated_at`

说明：

- 虽然字段名还叫 `active_invoice_count`、`last_invoice_at`
- 但现在存进去的已经是：
  - 配置周期内订单次数
  - 最近下单时间

也就是**字段名还是旧名字，字段值已经按最新“订单活跃度”规则在用了**

## 4. 当前排序是否已匹配最新规则

已匹配。

当前供应商管理列表排序规则是：

1. 先按 `is_active` 排序
2. 活跃供应商内部按 `last_invoice_at DESC`
3. 若最近时间相同，再按 `active_invoice_count DESC`
4. 最后按供应商名称

对应到最新规则，实际含义就是：

1. 先分活跃 / 不活跃
2. 活跃供应商里，最近下单时间越近越靠前
3. 最近下单时间一样时，配置周期内下单次数越多越靠前

所以从“功能是否正确”来说，**现在已经能满足最新规则，不需要额外增加排序字段才能实现排序**。

## 5. 是否需要新增“活跃度排序字段”

结论：

- **功能上不需要**
- **语义上可以考虑**

原因：

- 现有字段已经足够支撑排序：
  - `is_active`
  - `active_invoice_count`
  - `last_invoice_at`
- 只是字段命名仍然是“发票”口径，和现在实际“订单活跃度”含义不完全一致

所以分两种看法：

### 方案A：不加字段，继续沿用现有字段

适合现在：

- 不改表结构
- 不补 migration
- 当前功能已经能跑
- 风险最小

### 方案B：增加语义更准确的新字段

如果后面你想把数据含义彻底理顺，建议新增：

- `active_order_count`
- `last_order_at`

这样以后看表、写 SQL、排查逻辑都更直观。

## 6. 如果要补新字段，这里是 SQL

如果你决定把“订单活跃度”字段单独落库，可以执行下面 SQL：

```sql
ALTER TABLE procurement_suppliers
  ADD COLUMN active_order_count INT NULL DEFAULT 0 COMMENT '配置周期内采购订单数' AFTER active_invoice_count,
  ADD COLUMN last_order_at DATETIME NULL COMMENT '最近一次采购订单日期' AFTER last_invoice_at;
```

如果要顺手给排序查询准备索引，可以加：

```sql
ALTER TABLE procurement_suppliers
  ADD INDEX idx_procurement_suppliers_activity_sort (
    is_active,
    last_order_at,
    active_order_count
  );
```

说明：

- 这两条 SQL **不是当前必须执行**
- 当前代码逻辑不依赖它们也能工作
- 只有当你决定把字段语义从“发票”彻底改成“订单”时，才建议执行

## 7. 当前建议

当前最稳妥建议是：

1. 先继续沿用现有字段，不急着改库结构
2. 风险与活跃度计划任务分开执行：
   - `procurement_supplier_risk_refresh`：每天 `02:00`
   - `procurement_supplier_activity_refresh`：每天 `02:30`
3. 活跃度规则按 `ACTIVE_SUPPLIER_ORDER_DAYS` 配置走
4. 如果后面确认这套规则长期稳定，再补 migration，把字段名正式迁移成订单口径
