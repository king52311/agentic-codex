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
