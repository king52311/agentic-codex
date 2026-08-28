# omsapi 与 product 表具功能流程对比

## 1. 结论

`omsapi` 是在 `product` 旧实现基础上的整理版本，表具状态值和主要业务表基本兼容，核心差异在于：

- `product` 为旧版解包/反编译产物，流程分散在旧 Controller、Service、Mapper 中，部分品牌逻辑直接操作设备表。
- `omsapi` 保留旧业务兼容入口，并将状态变更、换表、炳华卡表处理集中到明确的 Service；状态操作补充操作员、IP、写入时间审计。
- 两套系统都不是单纯修改 `newsun_meter.ecuState`：状态变更还要写停用或落表/复装记录；换表还要写换表记录，部分品牌会同步账户、充值或写卡流水。
- `product` 中可看到旧逻辑的 SQL，但因缺少完整源码，部分精确执行顺序以反编译 class 和 Mapper 为依据。

## 2. 功能对照

| 功能 | product 旧流程 | omsapi 当前流程 | 主要数据变化 |
|---|---|---|---|
| 启用 | 按表号更新 `newsun_meter.ecuState=0`，写启用记录 | `PUT /meter/updateMeterState`，传 `unLoadType=0`；更新状态后写 `newsun_meter_stop`，`stopType=0` | `newsun_meter`、`newsun_meter_stop` |
| 停用 | 按表号更新状态，写停用记录 | 同一接口传 `unLoadType=3`；写 `newsun_meter_stop`，`stopType=0` | `newsun_meter`、`newsun_meter_stop` |
| 落表 | 更新状态为 `2`，记录落表 | 同一接口传 `unLoadType=2`；写 `newsun_meter_unload`，`unLoadType=0` | `newsun_meter`、`newsun_meter_unload` |
| 复装 | 更新状态为 `0`，写复装记录 | 同一接口传 `unLoadType=4`；写 `newsun_meter_unload`，`unLoadType=1` | `newsun_meter`、`newsun_meter_unload` |
| 普通换表 | 旧表/新表处理、换表记录和账户处理分散在旧 Service | `POST /newsunMeter/exchange/insertExchange`，由 `insertExchangeRecord2` 完成校验、换表记录、新表插入/旧表处理 | `newsun_meter_change`、`newsun_meter`，必要时账户/收费表 |
| 炳华换表 | 使用 `bh_smartmeter`、写卡记录等旧业务链路 | `POST /newsunMeter/exchange/insertExchangeBH`，`exchangeType=1` 进入炳华换表 | `bh_smartmeter`、`bh_writecard_record` 等 |
| 炳华补卡 | 先调用本地写卡服务，再记录补卡流水 | 前端调用本地 `findCard/makeUserCard`，成功后调用同一接口 `exchangeType=2` | 写卡服务、`bh_writecard_record`、换卡记录 |
| 卡表换 NB 表 | 旧逻辑涉及卡表、NB 表、补发量、账户和换表记录 | `POST /newsunMeter/exchange/exchangeMeterForSe`，按炳华/赛恩兼容流程处理 | `newsun_meter`、换表/账户/写卡相关表 |
| IC 卡转 NB | 通过新表号查询目标用户，再执行换表登记 | `POST /newsunMeter/exchange/exchangeMeterForIcToNb` 目前主要用于按新表号查询资料；登记记录使用 `insertExchangeForIcToNb` | `ic_nb_meterchange`，并视业务流程同步表具 |
| 取消 | 主要是前端关闭弹窗或取消选择，不是独立状态接口 | 前端 `取消` 同样只关闭操作，不改变数据库 | 无数据库变化 |

## 3. omsapi 实际请求链路

### 3.1 状态变更

1. 用户档案页面 `DocuserDocument.vue` 校验当前表具状态。
2. 页面组装表号、户 ID 和 `unLoadType`。
3. 调用 `PUT /meter/updateMeterState`。
4. `NewsunMeterController.updateMeterState` 获取当前登录操作员、IP、当前时间。
5. `NewsunMeterService.updateMeterStateByEcuId` 更新 `newsun_meter.ecuState`。
6. 根据操作类型写入历史记录：
   - 启用/停用：`NewsunMeterStopService` -> `newsun_meter_stop`。
   - 落表/复装：`NewsunMeterUnloadService` -> `newsun_meter_unload`。
7. 返回成功后前端刷新用户档案和表具列表。

状态映射：

```text
ecuState=0  正常/启用
ecuState=1  已换表（主要用于历史或换表场景）
ecuState=2  已落表
ecuState=3  已停用
```

接口参数中的 `unLoadType` 是操作类型，不完全等同于最终数据库状态：

```text
unLoadType=0  启用，最终 ecuState=0，写停用记录 stopType=1
unLoadType=2  落表，最终 ecuState=2，写落表记录 unLoadType=0
unLoadType=3  停用，最终 ecuState=3，写停用记录 stopType=0
unLoadType=4  复装，最终 ecuState=0，写落表记录 unLoadType=1
```

### 3.2 普通换表

1. 前端只允许当前表具为启用状态，并根据原表品牌加载新表类型。
2. 调用 `POST /newsunMeter/exchange/insertExchange`，传旧表号、新表号、旧止度、新表底数、品牌和换表说明。
3. `NewsunMeterExchangeService.insertExchangeRecord2` 执行旧表资料读取、新表资料校验、换表业务处理。
4. 通过 `NewsunMeterExchangeMapper.insertExchangeRecord` 写 `newsun_meter_change`。
5. 通过 `NewsunMeterMapper.exchangeInsertMeter` 插入或更新新表，必要时 `deleteOldMeter` 删除旧表。
6. 事务成功后前端刷新列表。

### 3.3 炳华换表/补卡

1. 前端先按表号、登记号读取 `bh_smartmeter` 和用户资料。
2. 换表调用 `exchangeMeterForSe` 或 `insertExchangeBH(exchangeType=1)`。
3. 补卡先通过本地写卡服务寻找卡并写卡；写卡成功后调用 `insertExchangeBH(exchangeType=2)` 记录业务流水。
4. Service 组装 `BhWritecardRecord`，写入写卡记录并执行插入后的补充更新。
5. 失败时前端仅提示失败，设备写卡服务和数据库记录需结合日志确认是否已经产生部分结果。

## 4. product 旧流程

`product/msApi-0.0.1-SNAPSHOT` 没有完整 Java 源码，主要依据解包后的 Mapper 和 class 追踪：

1. 旧 Controller 接收页面参数。
2. 旧 Service 直接读取 `newsun_meter`、`newsun_account` 或品牌专用表。
3. Mapper 执行状态更新、旧表删除、新表插入和历史记录写入。
4. 普通表具使用 `newsun_meter`；炳华使用 `bh_smartmeter`；IC 转 NB 使用 `ic_nb_meterchange`。
5. 换表记录使用 `newsun_meter_change`，停用和落表记录分别使用 `newsun_meter_stop`、`newsun_meter_unload`。

Product 中可确认的关键 SQL：

- `NewsunMeterMapper.xml`：`updateChangeByState`、`exchangeInsertMeter`、`deleteOldMeter`。
- `NewsunMeterExchangeMapper.xml`：插入 `newsun_meter_change`、插入 `ic_nb_meterchange`、换表查询。
- `NewsunMeterStopMapper.xml`：停用记录查询及列表分页。
- `NewsunMeterUnloadMapper.xml`：落表/复装记录查询及列表分页。

## 5. 关键差异与风险

1. **审计信息**：`omsapi` 状态变更明确补充 `operatorId`、`loginIp`、`dateWrite`；旧实现需以历史 class 行为为准。
2. **事务一致性**：`omsapi` 换表 Service 使用事务，但炳华补卡包含外部本地写卡服务，数据库事务无法回滚已经写入设备的数据。
3. **删除旧表**：普通换表流程存在 `deleteOldMeter`，如果旧表仍需要历史查询，依赖换表记录和历史表；迁移前应确认生产数据备份策略。
4. **多表用户**：当前查询和部分账户逻辑仍有按表号关联的旧代码；充值已按户改造，但本对比涉及的表具状态和换表仍以表号为主。
5. **设备品牌分支**：赛恩、炳华、浪花、环翔、水门、万泰等分支参数和专用表不同，不能只复用普通换表接口。
6. **取消按钮**：取消不是后端业务状态，不能据此判断某次操作是否回滚；真正提交后只能通过业务记录和当前状态核验。

## 6. 维护建议

- 新功能优先接入 `omsapi` Service，不直接在 Vue 中拼接数据库业务语义。
- 状态变更必须同时校验当前状态、写审计记录，并保证主表与历史表在同一事务内完成。
- 换表前保留旧表关键读数、账户余额、用户档案关系；换表后用 `newsun_meter_change` 作为追溯入口。
- 对炳华等外部写卡流程增加幂等号和结果查询，避免网络重试造成重复写卡或重复流水。
- 生产变更涉及表结构时，必须同步 `omsapi/db/migrations/` 迭代 SQL，不修改基线文件代替迁移。
