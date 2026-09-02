# PROJECT_HISTORY

## 2026-08-24

- 新增 `agent-docs/` 目录，作为 Agent 相关 Markdown 文档统一归档入口。
- 更新 `rules/project-script-standards.md`，补充业务项目忽略 `start.sh`、`stop.sh`、`build.sh`、`*.md` 的通用建议。
- 明确已跟踪文件需通过 `git rm --cached` 取消跟踪，需保留文档用 `.gitignore` 白名单放开。
- 更新 `AGENT.md` 与 `README.md` 索引，要求后续 Agent 相关 md 优先落到 `agentic-codex/agent-docs/`。

## 2026-08-11 08:34:39 CST

- 新增本机开发环境配置：`environment/local-dev-environment.md`。
- 记录本机 Python 有 conda、Node.js 有 nvm，并补充常用检查命令。
- 更新 `AGENT.md`，要求进入业务项目后读取该环境配置，避免误判系统环境。

## 2026-07-30

- 新增客户端应用创建通用规范：`rules/client-application-standards.md`。
- 规范独立英文应用目录、默认版本 `1.0.0`、应用展示命名、应用文档、帮助入口、Web 热更新、短参数脚本和公共组件库复用要求。
- 更新 `AGENT.md` 规则索引，便于后续项目创建客户端应用时复用。

## 2026-07-27 00:00:00 CST

- 新增微服务 Parent 工程参考架构：`architectures/microservice-parent-reference-architecture.md`。
- 方案沉淀 parent 职责边界、公共组件模块划分、不可上沉内容、依赖边界、新项目落地清单和验证方式。
- 更新 `architectures/README.md` 与 `AGENT.md` 索引。

## 2026-07-27 00:00:00 CST

- 从 drain 项目沉淀可复用规则和方法到 agentic-codex。
- 新增 Java 后端通用规范：`rules/java-backend-standards.md`。
- 新增项目脚本通用规范：`rules/project-script-standards.md`。
- 新增 Maven 多模块本地联调方法：`workflows/maven-multi-module-debug.md`。
- 新增集成自测脚本方法：`workflows/integration-self-test-scripts.md`。
- 新增快捷发版脚本推荐方案：`workflows/release-shortcut-scripts.md`。
- 更新 `AGENT.md` 与 `README.md` 索引。
## 2026-07-31

- 新增客户端应用生成框架参考架构：`architectures/client-app-scaffold-generation-architecture.md`。
- 沉淀独立客户端应用的完整生成流程、核心约束、目录脚手架、公共能力接入顺序和逻辑代码生成层次，方便后续项目快速搭建应用基础框架。
- 更新 `AGENT.md` 与 `architectures/README.md` 索引。
- 客户端应用生成框架去除案例专属品牌约束，将展示名规则改为使用当前组织或项目约定的统一前缀，保持框架通用性。

## 2026-08-27

- 完成自动结算调度功能：支持每日、工作日、每周、每月、季度、年度，并记录执行批次与明细。
- 按现有出账模型调整自动结算：出账阶段已扣减余额并累计待收金额，自动结算仅核销待收金额，不重复扣减余额。
- 暂不接入余额信用/欠费开关，余额不足时按用户账期顺序停止后续账单处理。
- 增加调度日期边界测试 4 项，全部通过；后端配置接口已验证返回新增字段，服务正常启动。
- 营收首页快捷入口新增收费工作台、账单查询、抄表册管理、抄表计划管理、计费出账管理，统一改为配置驱动渲染并完成测试环境构建。
- 修复 omsapi Docker 构建对 `xjbw-security-access/` 空目录的无效 COPY 依赖；安全组件继续使用 `src/main/resources/lib/xjbw-security-access.jar`，日志目录取消跟踪后不再影响服务器构建。
- 修复营收首页饼图悬浮时被画布裁切的问题：统一缩小半径、下移圆心并限制悬浮偏移，已重新生成测试环境 `dist.zip`。
- 完成抄表周期与自动出账联动：抄表册新增独立账单日；系统按每个表册的周期、抄表月份和例日提前 3 天自动生成计划，同表册同账期防重复；自动出账取计划账单日排队执行，手动出账立即执行。新增待执行批次状态、计划执行时间及活动快照，测试库迁移、后端编译和前端测试构建均通过。
- 系统设置新增计划任务管理：统一展示自动生成抄表计划、自动出账、自动结算三类配置状态，并提供任务类型、执行状态、时间范围筛选及分页执行日志。
- 新增计划任务日志表并同步测试数据库；自动计划记录生成结果，自动出账和自动结算复用现有执行记录汇总展示。后端接口联调、JDK8 构建及前端测试构建通过，`dist.zip` 已更新。
- 本轮业务变更已提交并推送至 `new` 分支：`omsapi a81322d`（自动出账调度和计划任务管理），`omsvue ba283a5`（计划任务管理及抄表出账页面）。
- 系统设置目录下的同名子菜单改为“后台配置”，迁移 SQL 已新增并执行到测试数据库，目录名称保持“系统设置”。
- 新增系统设置下“违约金配置”：支持模板基础信息、用户性质、生效日期、基准日期、宽限偏移、日千分比、比例/固定金额封顶、详情、编辑和启停；新增 `penalty_rule` 表及测试库菜单。
- 违约金配置增加余额信用校验：余额信用允许负数时页面提示并禁止新增、编辑、启停，后端接口同步拦截；后端 JDK8 构建和前端测试环境构建通过，`dist.zip` 已更新。
- 核查违约金菜单缺失问题：测试库菜单 ID 79 已存在并授权系统管理员，`/config/sysmenu` 接口已验证返回“违约金配置”；需重启使用新构建的 API，并退出登录重新加载动态菜单。
- 违约金模板用户性质改为读取 `basic_price_manageNo` 真实字典数据，日利率与封顶比例单位调整为和数值同行显示；前端测试构建及 `dist.zip` 已更新。
- 本轮违约金配置代码已整理，待提交至业务项目 `new` 分支；包含 API 配置接口、数据库迁移和 Vue 页面构建产物。
# 2026-08-25

- 按 Agentic Codex 归档规则新增 `agent-docs/project-notes/cbys-omsapi/` 独立目录。
- 从 `omsapi` 根目录迁移说明类 Agent 回顾文档：`API_SCHEDULED_TASKS.md`、`COLLECT_DATA_PROJECT.md`、`PROJECT_RUNNING_LOGIC.md`、`DOCUSERDOCUMENT_PRODUCT_DIFF.md`。
- `omsapi` 继续保留 `AGENTS.md`、`PROJECT_HISTORY.md`、`README.md` 作为业务项目必要入口。
- 新增全局 Agent 文档落地规则：所有项目的 Agent 相关 Markdown 统一写入 `agentic-codex` 对应目录，不写入业务项目根目录；写完必须在 `agentic-codex` 中先 `git pull --rebase`，再 `git add`、中文提交并 `git push`。

## 2026-08-26

- 新增前端构建通用规则：项目存在 `build.sh` 时必须优先使用该脚本；仅在脚本不存在时，才执行 npm、pnpm 或 yarn 的系统构建命令。
- `omsvue` 阶梯设置页面增加管理类型、阶梯名称筛选，支持搜索和重置。
- `omsapi` 阶梯设置初始化接口 `/system/price/initData` 增加可选筛选参数 `manageNo`、`priceName`，保留旧的无参调用兼容。
- 验证通过：前端 `npm run build`，后端 JDK8 `mvn -q -DskipTests compile`。
- `omsvue` 调整系统列表公共宽度样式，`ws-table-wrap`、`el-table`、后台内容区统一支持自适应铺满，避免列表右侧空白。
- 验证通过：前端 `npm run build`。
- `omsapi` 充值缴费链路切换为按户 `profilesId` 聚合更新余额，保留抄表按表号逻辑不变。
- `newsun_account` 增加 `profilesId` 字段迁移，新增/导入账户时回填户ID，充值记录查询支持按户查询。
- 验证通过：后端 JDK8 `mvn -q -DskipTests compile`，前端 `npm run build`。

## 2026-08-31

- 统一前端表具品牌显示：将数据库品牌值 `赛恩` 的错误展示名称“晨天”改为“赛恩”，保留后端接口、数据库值及业务逻辑不变。
- 更新品牌筛选、用户档案品牌选项、表具更换品牌选项及公共品牌格式化方法。
- 前端测试环境构建成功，并重新生成 `omsvue/dist.zip`。

## 2026-08-31

- 新增系统设置下“品牌管理”：基于 `basic_code` 的 `metercompany` 类型维护表具品牌，支持搜索、分页、新增和修改，保留历史品牌数据，不提供删除操作。
- 后端新增品牌专用分页、新增和修改接口；新增菜单及角色权限迁移 SQL：`omsapi/db/migrations/20260831_add_meter_company_manage_menu.sql`。
- 前端新增 `SysBrandManage.vue`，系统设置增加品牌管理入口；前端测试构建、后端 Maven 编译均通过。
- 品牌管理迁移 SQL 已执行到测试数据库 `djwms`，新增菜单 ID `80`，已授权 3 个管理角色；现有品牌数据为赛恩、浪花、炳华NB、环翔NB、水门NB。
- 修正品牌管理菜单层级：保留一级菜单“系统设置”，将“品牌管理”从“后台配置”下移为“系统设置”的直属子菜单，与“后台配置”同级；测试数据库已同步更新，前端测试构建通过。
- 修复用户档案品牌选项与品牌管理不一致：新增档案、编辑档案、添加表具及档案筛选均改为读取 `metercompany` 品牌字典，当前统一展示赛恩、浪花、炳华NB、环翔NB、水门NB；前端测试构建通过。
- 抄表册新增/编辑弹窗候选水表优化：增加小区下拉筛选，候选列表改为以 `newsun_meter` 为主表查询全部正常状态水表，并带出小区列；已选列表同步展示小区。后端编译和前端测试构建通过，`dist.zip` 已更新。

## 2026-09-01

- 抄表计划管理新增批量结算：勾选计划后显示“结算”按钮，调用 `/reading/plan/billing-settle`。
- 后端新增计划结算模式 `PLAN_SETTLE`：仅余额信用开关允许负数时可用；同一计划复用同一个出账批次，重复点击会回滚旧扣款与旧流水后重算，不重复生成批次。
- 新增迁移 SQL `omsapi/db/migrations/20260901_add_plan_settle_batch_index.sql`，用于加速按计划复用批次查询。
- 验证通过：`omsapi mvn -q -DskipTests compile`，`omsvue ./build.sh test`，并更新 `dist.zip`。

- 测试库 `djwms` 已执行抄表计划批量结算索引变更：`idx_reading_billing_batch_plan_type(plan_id, billing_type)`。

- 已将 `20260901_add_plan_settle_batch_index.sql` 调整为可重复执行脚本，并重新执行到测试库 `djwms`，确认索引存在。

## 2026-09-01

- 抄表册管理新增“小区表册生成”：支持多选小区、选择抄表员、设置周期/抄表月份/抄表例日/账单日，一次性按小区生成表册。
- 后端新增 `/reading/book/generate-by-community`，仅余额信用允许负数时可用；每个小区生成一个表册，表册成员为该小区全部正常状态水表对应用户，支持一户多表后续生成抄表活动。
- 验证通过：`omsapi mvn -q -DskipTests compile`，`omsvue ./build.sh test`，并更新 `dist.zip`。

- 自动生成抄表计划规则调整：由抄表日前 3 天生成，改为抄表日前一天或当天扫描生成；计划任务管理文案同步为“抄表日前一天生成”。
- 抄表活动上期读数仍保持计划生成时固定，出账时使用 `reading_activity.last_reading`，不实时重取。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 修复表具最新状态列表搜索已建档但无实时 `ecu` 记录的表具时表号为空的问题：列表 SQL 改为 `ifnull(e.ecuId, nm.ecuId)`，排序表号时使用 `newsun_meter.ecuId`。测试表号 `8787687654` 已验证可返回表号。
- 验证通过：`omsapi mvn -q -DskipTests compile`，`EcuMapper.xml` 结构校验通过。

- 修复用户档案管理抄表导出导致页面白屏：前端由 `window.location.href` 跳转下载改为 `axios` blob 下载，支持单小区 Excel 和多小区 zip，并解析后端文件名。
- 优化内容区顶部多标签栏：标签超出时横向滚动、标题省略，避免标签过多导致错位。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 内容区多标签栏改为左右箭头翻动模式：隐藏原生横向滚动条，左右按钮控制标签滚动，当前激活标签自动滚入可视区；样式参考用户提供截图调整为紧凑按钮态。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 抄表导出季度口径调整为包含当前日期所在季度，并向前推三个季度；2026 年 9 月导出最后一列季度为 2026 第三季度抄表止度。
- 抄表导出“用户名”列增加格式化：匹配 `楼栋#房号` 时导出为 `楼栋-1-房号`，如 `上东湾5#101` 导出为 `上东湾5-1-101`，只影响导出文件不改数据库。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 抄表导出文件名调整：单小区格式为 `yyyyMMdd统计小区-小区名的季度抄表情况.xlsx`，多小区压缩包格式为 `yyyyMMdd统计小区的季度抄表情况.zip`。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 修复后端重启后后台登录态丢失问题：Spring Security remember-me 增加 `alwaysRemember(true)`，登录成功后默认写入 30 天持久 Cookie，后端重启后可自动恢复认证；原 JSESSIONID 仍会随 JVM 重启失效。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 修复抄表导出前端下载兜底文件名：浏览器取不到后端 `Content-Disposition` 或后端未重启时，也按 `yyyyMMdd统计小区-小区名的季度抄表情况.xlsx` / `yyyyMMdd统计小区的季度抄表情况.zip` 命名。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 重做内容区标签栏样式为浅灰卡片风格：白底、细边框、顶部强调线，左右箭头和更多菜单保持独立按钮区，整体更接近用户提供的参考图。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 收紧内容区标签栏箭头展示：去掉外层重复的左右箭头按钮，仅保留 tabs 本身的滚动表现，避免双层箭头干扰点击。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 修复自动生成抄表计划任务扫描空指针：`ReadingService.generateDuePlans` 先判空再计算日期差，`daysBetween` 增加空值兜底，避免定时任务因空日期中断。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 修复缴费记录接口 `/newsun/payment/getpayments` 查询用户 `profilesId=7503` 时的零除异常：缴费量为空或为 0 时，污水处理单价返回 `0`，不再执行除法。
- 验证通过：`omsapi mvn -q -DskipTests compile`。

- 抄表活动明细查询时自动同步符合计划抄表日期的表具实时读数；已有本期读数但状态为待抄的记录自动改为“已抄”，并同步抄表时间、用水量和采集来源。
- 前端抄表活动状态文案由“已上传”调整为“已抄”。
- 验证通过：`omsapi mvn -q -DskipTests compile`，`omsvue ./build.sh test`，并更新 `dist.zip`。

- 收费工作台新增“充值记录”“用水分析”入口，分别复用原账户充值管理的充值记录接口和用水分析接口。
- 优化收费工作台预存弹窗：当前用户和可用余额改为分行展示，避免长文本挤压换行；补充入口及分析弹窗响应式样式。
- 验证通过：`omsvue ./build.sh test`，并更新 `dist.zip`。

- 修复 `ReadingMapper.xml` 中 SQL 比较符未转义导致的 MyBatis XML 解析失败，`<>` 改为 `&lt;&gt;`。
- 验证通过：`omsapi mvn -q -DskipTests compile`。
