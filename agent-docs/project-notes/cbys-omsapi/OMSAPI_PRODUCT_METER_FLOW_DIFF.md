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

## 3.0 代码执行顺序对比

本次核对后，普通换表的 `omsapi` 与 `product` 反编译 class 实际调用顺序基本相同；差异主要不是“业务步骤改变”，而是源码可维护性、接口保留情况和后续改造内容不同。

### 状态操作的实际执行

```text
Vue DocuserDocument.disable/enabled/fallmeter/reinstall
  -> putRequest("/meter/updateMeterState", meterUnLoad)
  -> NewsunMeterController.updateMeterState
  -> SecurityContextHolder 获取登录用户
  -> SysOperaterService.loadUserByUsername 获取 operatorId
  -> CommonUtils.getIpAddr 获取 loginIp
  -> NewsunMeterService.updateMeterStateByEcuId
  -> NewsunMeterMapper.updateMeterStateByEcuId
  -> 根据 unLoadType 写 NewsunMeterStopMapper 或 NewsunMeterUnloadMapper
  -> 返回 RespBean
  -> Vue 刷新列表
```

`product` 的 Controller class 同样按 `unLoadType` 分支调用状态 Service 和停用/落表 Service；`omsapi` 已有对应 Java 源码，便于继续维护。两边都没有在“取消”时调用后端，取消只是关闭弹窗。

### 普通换表的实际执行

```text
Vue exchangeData
  -> 校验 exchangeForm
  -> POST /newsunMeter/exchange/insertExchange
  -> NewsunMeterExchangeController.insertExchangeRecord
  -> NewsunMeterExchangeService.insertExchangeRecord2
  -> getAccountByecuId(旧表号)
  -> getMeterByEcuId(旧表号)
  -> 获取当前操作员、IP、时间和用户档案
  -> sumPDetail() 查用户类型合计水价
  -> 计算 (页面旧止度 - 账户上次结算读数) * 水价
  -> exchangeInsertMeter() 插入新表，状态设为 0
  -> deleteOldMeter() 删除旧表
  -> updateEcuIdAccount() 把账户表号换成新表号并更新余额、结算读数
  -> insertexCharge() 写换表扣费流水
  -> insertExchangeRecord() 写 newsun_meter_change
  -> 返回“换表成功”
```

`product` 反编译的 `insertExchangeRecord2` 依次调用的 Mapper 与上述一致，因此普通换表当前不是“将旧表改成已换表”，而是**新增新表、删除旧表、迁移账户、写扣费流水和换表记录**。源码中 `updateChangeByState` 代码被注释，旧表不会保留在 `newsun_meter` 中。

### 阶梯换表的实际执行

代码中另有 `insertExchangeRecord3`，但当前 Controller 的 `/insertExchange` 实际调用的是 `insertExchangeRecord2`，所以页面普通换表目前走的是“合计水价”算法，不会自动走 `insertExchangeRecord3` 的阶梯计算。

如果业务代码显式调用 `insertExchangeRecord3`，执行顺序才是：读取账户累计用量 -> 调用 `ChargeService.getChargeSumResult` -> 按阶梯拆分流水 -> 新表插入/旧表删除 -> 账户余额更新 -> 写多条扣费流水 -> 写换表记录。

### 炳华补卡的实际执行

```text
Vue exchangeDataBH(exchangeCardFormBH)
  -> 校验表具类型和补卡原因
  -> GET http://localhost:5000/omspaymentapi/findCard
  -> 组装卡类型、表号、表具类型和 0 购水量
  -> POST http://localhost:5000/omspaymentapi/makeUserCard
  -> 写卡成功后 POST /newsunMeter/exchange/insertExchangeBH
  -> Controller 根据 exchangeType=2 调 insertExchangeCardRecordBH
  -> 查询炳华用户、表具和账户
  -> 组装 BhWritecardRecord
  -> BhWritecardRecordMapper.insert
  -> BhWritecardRecordMapper.updateAfterInsert
  -> 返回结果，关闭串口并刷新页面
```

这里存在两个系统边界：本地写卡服务成功不代表 API 数据库写流水一定成功；反过来也一样。因此 `product` 与 `omsapi` 都有“设备已写卡但数据库记录失败”的部分成功风险。

### 炳华换表与卡表换 NB

- 炳华换表：Vue 先校验新表号、补水量为非负整数，再调用 `exchangeMeterForSe`；Service 查询旧卡/登记号对应用户和表具，按补水量计算写卡/账户数据，最后记录换表和写卡流水。
- 卡表换 NB：Vue 先调用 `exchangeMeterForIcToNb` 按新表号查询资料，返回客户、地址、价格类型后才允许继续；真正的登记数据写入由 `insertExchangeForIcToNb` 写入 `ic_nb_meterchange`，不是仅靠查询接口完成。
- `product` 中上述入口、参数和 Mapper 表名均存在；`omsapi` 的代码分支更容易追踪，但设备服务地址仍是前端写死的本机 `5000` 端口。

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

## 4.1 数据库查询与写入差异

### 结论

表具状态和换表的核心 SQL 基本没有变化。对 `NewsunMeterMapper.xml`、`NewsunMeterExchangeMapper.xml`、`NewsunMeterStopMapper.xml`、`NewsunMeterUnloadMapper.xml` 做对照后：

- 换表记录插入 SQL 两边一致：插入 `newsun_meter_change`，字段包括旧/新表号、读数、金额、余额、用户类型、操作员、时间和 IP。
- IC 转 NB 记录插入 SQL 两边一致：插入 `ic_nb_meterchange`，保存旧表/新表品牌、表号、用户、地址、表类型、底数、补发量、价格和审计信息。
- 状态列表查询两边一致：停用列表从 `newsun_meter_stop` 左连接表具、用户档案、地址视图、操作员；落表列表从 `newsun_meter_unload` 使用相同方式关联。
- 普通换表两边都执行 `newsun_meter` 新表插入和旧表按 `ecuId` 删除；Mapper 中的 `updateChangeByState` 虽然存在，但当前普通换表代码没有调用。

### omsapi 新增或变化的查询

当前 `omsapi` 的 `NewsunMeterMapper.xml` 相比 `product` 增加或保留了以下扩展查询：

```sql
-- 一户多表：按户 ID 查询全部表具
select nm.*, e.lhecuId, na.balanceAmount
from newsun_meter nm
left join ecu e on e.ecuId = nm.ecuId
left join newsun_account na on na.ecuId = nm.ecuId
where nm.profilesId = ?
order by nm.ecuState asc, nm.id desc
```

这部分是 `omsapi` 的业务扩展，旧 `product` 没有对应的 `getMetersByProfilesId` 查询。它不会改变原有按表号查询状态和换表的结果，但为一户多表提供了户 ID 查询入口。

另外，当前 Mapper 增加了万泰表具关联 `wt_valve_list` 的查询，以及按时间查询 `newsun_payment` 的开阀相关查询；这些属于新增/扩展功能，不是状态变更主流程。旧 `product` 中存在的 `findEcuIdList`、赛恩 IMEI 查询则不在当前 Mapper 的同一位置保留，属于功能整理或迁移差异。

### 查询条件上的实际影响

- 状态更新使用精确匹配：`where ecuId = ?`，两套逻辑都按表号更新，不按户 ID 更新。
- 停用/落表历史列表使用模糊匹配：`ecuId like concat('%', ?, '%')`，两套逻辑一致。
- 普通换表账户查询使用旧表号：`getAccountByecuId(oldEcuId)`；用户档案和表具也使用旧表号查出 `profilesId`，之后才把账户表号改为新表号。
- 换表历史查询使用新表号、旧表号、联系人、换表日期模糊过滤；两套 `NewsunMeterExchangeMapper.xml` 查询结构一致。
- 账户、表具、历史流水没有统一改为按 `profilesId` 聚合处理；当前只有一户多表资料查询明确按户 ID，状态/换表仍按表 ID。

## 4.2 对外接口差异

### 旧 product 与新 omsapi 仍兼容的接口

两套 Controller 的公开方法和路径基本一致，前端原有调用无需改地址：

```text
PUT  /meter/updateMeterState
GET  /meter/meterStopList
GET  /meter/meterUnLoadList

POST /newsunMeter/exchange/insertExchange
POST /newsunMeter/exchange/insertExchangeBH
POST /newsunMeter/exchange/exchangeMeterForSe
POST /newsunMeter/exchange/exchangeMeterForIcToNb
GET  /newsunMeter/exchange/list
GET  /newsunMeter/exchange/listIcToNb
GET  /newsunMeter/exchange/loadAllDropdownList
GET  /newsunMeter/exchange/export
GET  /newsunMeter/exchange/export2
```

### omsapi 新增接口

`omsapi` 在表具 Controller 增加了：

```text
GET /meter/profileMeters?profilesId={户ID}
```

该接口返回一个用户档案下的全部表具，服务层调用 `getMetersByProfilesId`。这是当前一户多表页面使用的扩展接口，`product` 的旧 Controller 公共方法中没有该接口。

### 接口内部行为差异

- `product` 是已编译的旧 Controller，外部接口主要负责把参数转给旧 Service；只能从 class 和 XML 看到调用关系。
- `omsapi` 的状态接口在 Controller 内显式补充当前操作员 ID、客户端 IP、写入时间，再进入 Service；因此同一个请求写入的审计字段更明确。
- `omsapi` 普通换表接口固定调用 `insertExchangeRecord2`；该方法执行合计水价计算。代码中的 `insertExchangeRecord3` 阶梯计算方法没有被该公开接口调用。
- `insertExchangeBH` 通过 `exchangeType=1/2` 在同一个接口内分流换表和换卡；接口名没有拆分，属于旧接口兼容设计。
- `exchangeMeterForIcToNb` 当前对外表现是“按新表号查询目标资料”，返回 `exdata`；实际写入 `ic_nb_meterchange` 的 Mapper 方法是内部换表流程调用的 `insertExchangeForIcToNb`，不是这个查询接口本身。

### 外部设备接口差异

普通状态和普通换表只访问 API 数据库；炳华补卡/换卡还依赖前端直接访问：

```text
GET  http://localhost:5000/omspaymentapi/findCard
POST http://localhost:5000/omspaymentapi/makeUserCard
```

这两个设备接口在 `omsapi` Controller 中没有代理，仍由 Vue 页面直接调用。`product` 的旧页面也采用同类本地写卡调用方式，因此这里没有形成新的后端统一接口；服务器部署时尤其要保证操作员电脑上的 `5000` 端口可访问。

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
