# PROJECT_HISTORY

## 2026-08-12

- 为 `collect-data` 新增 `start.sh` 启动脚本，优先使用 Java 8，支持 `SPRING_PROFILES_ACTIVE`，并更新 `collect-data/README.md`。
- 根据 `product/LiteDJHarborDemo` 反编译结果新建 `collect-data` 独立采集项目：拆分平台适配、报文解析、公共入库、HTTP/RocketMQ 输入通道；保留华为平台与电信赛恩报文链路；新增 `COLLECT_DATA_PROJECT.md` 记录旧逻辑映射和扩展方式；`collect-data` 编译通过。
- 按 `/Users/sunday/Downloads/上东花苑二期.xlsx` 调整抄表导出样式：改用 POI 原生写出 Excel，表头为普通一行、默认列宽 9、默认行高 16.8，不再使用 EasyExcel 默认灰底粗体表头。
- 新增项目规则 `AGENTS.md`：明确每次数据库结构或初始化数据变更必须对照 `db/baseline/djwms-lite.sql`，并同步维护 `db/migrations/` 迭代 SQL，生产发版直接执行尚未执行的迭代 SQL。
- 修复首页用户头像破图：更正 `userFace` 字段判断，头像为空时显示默认圆形首字母头像。
- 调整抄表导出 Excel：季度表头按时间正序排列，先 2025 后 2026；文件名前缀改为 `抄表导出_`；表头不增加额外特殊样式。
- 修复刷新动态菜单页面白屏：首次进入动态路由时，菜单加载完成后重新匹配当前地址，避免 `/#/Doc/DocuserDocument` 这类页面刷新后路由未命中。
- 保存生产数据库结构基线：将 `/Users/sunday/Downloads/djwms-lite.sql` 复制到 `omsapi/db/baseline/djwms-lite.sql`；新增 `db/README.md` 约定基线、迭代 SQL、补丁目录用途；本次抄表导出数据库变更整理为生产迭代 SQL `db/migrations/20260812_meter_reading_export.sql`。
- 完成“用户档案管理-抄表导出”：前端新增按钮和小区多选弹窗，支持全选；单小区下载 Excel，多小区下载 zip；后端按导出时点倒推四个完整季度生成中文表头并通过预统计表查询；生产迭代 SQL 改为可重复执行。
- 分析“用户档案管理-抄表导出”SQL：当前写法在 `ecu_history` 约 123 万数据量下会全表扫描三次，单次测试 `上东花苑二期` 导出计数耗时约 4.4 秒；建议新增季度抄表历史预统计表，导出时只查预统计结果和当前档案/账户数据。
- 修复登录页回车无法登录问题：在登录表单级别绑定回车提交，并增加 `loading` 防重复提交保护。
- 测试库 `djwms` 已配置 `admin/admin` 用户：保留/更新现有 `admin` 账号，设置 BCrypt 密码、启用状态、组织为 `4`，并同步 `tjct` 的角色权限 `ROLE_admin(id=6)`；直测 `172.18.2.33:7077/login` 登录成功且菜单正常返回。
- 首页三个禁用快捷入口增加 hover 提示，鼠标移入显示“暂未开放功能”。
- 首页快捷菜单禁用态样式调整：`集中器状态`、`表具异常提醒`、`手动扣费管理` 三个不可点击入口的图标与文字置灰。
- 修正测试环境前端跨域：`omsvue/config/test.env.js` 的 `BASE_API` 改为同源 `/api`，由 nginx 反向代理到 `172.18.2.33:7077`。
- 前端 `omsvue/build.sh` 支持命令参数选择环境：`./build.sh test` 构建测试环境，`./build.sh prod` 构建正式环境，默认仍为测试环境。
- 前端 `omsvue` 增加环境配置：开发环境默认连接 `http://127.0.0.1:7077`，测试构建连接 `http://172.18.2.33:7077`，正式构建连接 `http://8.130.185.51:9900`；`build.sh` 默认测试环境，可通过 `BUILD_ENV=prod` 打正式包。
- 修正 `omsapi` Docker 构建兼容性：移除 Dockerfile 中需要 BuildKit 的 `RUN --mount=type=cache` 写法，确保服务器普通 `docker-compose build` 可直接执行。
- 优化 `omsapi` Docker 构建卡在 Maven 打包的问题：新增 Docker Maven settings 使用阿里云 Maven 镜像，并去掉 `-q` 方便观察依赖下载进度。
- 前端 `omsvue/build.sh` 压缩包名称固定为 `dist.zip`，每次构建覆盖旧包。
- 前端 `omsvue/build.sh` 构建完成后自动将 `dist` 目录打包为 zip。
- 为前端 `omsvue` 新增 `build.sh`，支持 nvm 自动切换 Node 20、Node 17+ legacy OpenSSL、缺少依赖时自动安装，并确保生产构建不注入开发默认账号密码。
- 为后端 `omsapi` 新增 Docker 构建配置：`Dockerfile` 使用 Maven JDK8 多阶段打包，运行镜像使用 Eclipse Temurin JRE8；新增 `docker-compose.yml` 暴露 `7077` 并挂载 `log` 目录；新增 `.dockerignore` 排除生成物。

## 2026-08-11

- 首页统计卡片补零值显示：`柜台退费（元）`、`柜台售水（吨）` 在无数据时默认展示 `0`。

- 首页快捷菜单调整：`集中器状态`、`表具异常提醒`、`手动扣费管理` 三个入口保留展示但移除点击跳转。

- 修复前端后端服务断开时刷新白屏：动态菜单加载改为完成登录态校验后再放行路由；后端不可用时进入登录页并每 3 秒自动重试，后端恢复且会话有效后返回原始页面。
- 增加运行中断线处理：接口请求检测到后端不可达时自动跳转登录页并保留当前地址，避免业务组件使用空响应导致白屏。
- 修复登录状态异常处理：网络故障和后端 5xx 不清理本地登录态，只有会话失效时退出；登录页支持恢复原页面，退出时同步清理 Vuex 用户状态。
- `omsvue` 执行 `npm run build` 通过。

- 对照 product 项目排查“表具最新状态”高级筛选级联逻辑：保留小区名称用于列表查询，新增页面级 `communityId` 供楼号、门号接口使用，修正小区选择后楼号和门号级联。
- 仅修改 `omsvue/src/components/meterReading/MetecuNewest.vue`，移除操作栏“关阀”按钮及页面层对应方法，后端接口未修改。
- `omsvue` 执行 `npm run build` 通过。
- 为“表具最新状态-导出数据”增加筛选条件支持：前端导出携带表号、用户、小区、楼号、门号、品牌、集中器号，后端导出查询按相同条件过滤。
- 前端构建通过；后端编译因当前环境仅有 JRE、缺少 JDK 编译器未完成。
- 新增 `API_SCHEDULED_TASKS.md`，梳理 API 项目定时任务；确认 Spring `@Scheduled` 任务存在但因 `@EnableScheduling` 注释当前不自动执行，`PinCodeManager` 的 Timer 清理任务独立生效。
- 修复“表具最新状态-导入数据”上传 `.xlsx` 报 `OfficeXmlFileException`：`PoiExcel.importsDocList` 改用 `WorkbookFactory` 自动识别 `.xls/.xlsx`，并增加空单元格保护，未改变导入字段映射和业务处理逻辑。
- 继续修正“表具最新状态-导入数据”：导入统一只跳过第一行表头；识别“表号/抄表止度”模板时走表具最新状态读数更新分支，更新 `ecu.reading/ecuStateDate/serverDate`；空文件返回明确错误。使用 Zulu JDK8 执行 `mvn -q -DskipTests compile` 通过。
- 完善 Excel 导入错误诊断：记录文件名、大小、解析条数、数据库更新条数和异常堆栈；接口分别返回文件为空、模板无有效数据、表号未匹配、业务返回码和具体异常信息。使用 Zulu JDK8 编译通过。
- 更新 `omsvue/start.sh`：增加后端 `OMS_API_URL` 健康监测，默认每 3 秒检测 `172.18.6.140:7077`，断开后自动重试，恢复后继续使用前端代理；可通过 `OMS_API_RETRY_INTERVAL` 调整间隔。
- 开发环境 `start.sh` 注入默认登录账号 `tjct / 18512270349`，仅通过 `dev.env.js` 传给开发登录页；生产 `dist` 构建不注入默认账号密码。
- 修复 Vue IP 访问时 `/login_p` 重定向循环：移除 `/login_p` 到 Java 后端的代理，webpack dev server 本地直接返回 `index.html`，避免后端 302 再次跳回 `/login_p`。
- 根据导入日志修正“上东湾aaa.xlsx”误判为用户档案模板的问题：放宽“表号/表具编号”和“抄表止度”表头识别，避免解析到 1 条后走错误业务分支。
- 分析 `omsapi/log/omsapi-kangpengdeMacBook-Pro.local-application.log`：确认 16:46、16:51、16:53 的导入失败均因 `上东湾aaa.xlsx` 被识别为用户档案模板，解析 1 条后档案导入业务返回 0；16:48 附近存在 devtools 热重启时 `lombok.Generated cannot be resolved to a type` 导致短暂启动失败，后续 16:52:49 已恢复启动到 7077。
- 继续修正 `上东湾aaa.xlsx` 导入模板识别：除“表号/表具编号/表记地址/表计地址/水表号”外，也识别“表底数/抄表时间”列，支持这类表具最新状态 Excel 直接更新 `ecu` 表的读数和抄表时间；`mvn -q -DskipTests compile` 通过。

## 2026-08-10

- 分析 `omsapi` 与 `omsvue` 两个项目目录结构。
- 新增根目录 `README.md`，记录后端/前端结构、技术栈、启动方式、业务模块和 Agentic 协作规则。
- 确认根目录当前不是 Git 仓库，未执行 commit/push。
- 为 `omsapi` 与 `omsvue` 新增 `start.sh` 开发环境启动脚本；后端默认 `server_dev` profile，前端默认 `npm run dev`。
- 修正 `omsapi/start.sh` 与 `omsvue/start.sh` 为 POSIX sh 兼容写法；前端启动脚本增加 nvm 自动加载与 `.nvmrc/default` 版本切换。
- 修正 API 启动脚本：检测 Maven Wrapper 不完整时回退系统 Maven，并自动优先使用 Java 8；修正前端 nvm 切换逻辑并为 Node 17+ 增加 legacy OpenSSL 参数。
- 进一步修正启动脚本：API 使用 `java_home -v 1.8.0` 避免命中 Java 插件目录；Vue 默认 `nvm use 20` 并刷新 shell 命令缓存。
- 修正 Vue 启动脚本：nvm 切换后显式将 `NVM_BIN` 放到 `PATH` 最前，避免 Homebrew `node@22` 优先。
- 修正 API 启动脚本强制优先使用 JDK8，解决老 Lombok 在 Java 9+ 下 `TypeTags` 编译失败；Vue 启动脚本默认代理到本机 Java dev 接口 `127.0.0.1:7077`，并让 webpack 代理读取 `OMS_API_URL/OMS_WS_URL`。
- 修正 Vue 启动脚本：启动前自动给 `node_modules/.bin/*` 和 `webpack-dev-server` 补执行权限，解决 `Permission denied`。
- 修正 Vue 编译失败：将 `DepMana.vue`、`SysOrganization.vue`、`SysAddress.vue` 中 JSX 形式的 `renderContent` 改为兼容当前构建链的 `h()` 渲染函数。
- 补充 `omsvue/.postcssrc.js`，解决生产构建处理 Element UI、Font Awesome、viewerjs CSS 时找不到 PostCSS 配置的问题。
- 修正 `DocTransferAdmin.vue` 中两处 `imgurl2` 列表渲染缺少 `key` 的 Vue warning。
- 修正 Vue 白屏：`store/index.js` 显式导入 `SockJS/Stomp`，避免 webpack 下全局变量未定义；开发启动强制 `BASE_API` 默认为空走 webpack 代理；后端 dev 数据库 URL 增加 `useSSL=false` 消除 MySQL SSL 警告。
- 继续修正 Vue 登录白屏/CORS：`omsvue/start.sh` 默认忽略外部 `BASE_API`，强制走 webpack 本机代理，仅在 `OMS_DIRECT_API=1` 时直连后端；`src/utils/api.js` 增加 `err.response` 空值保护，避免网络/CORS 错误触发二次异常。
- 修正 Vue dev WebSocket/SockJS 代理：`/ws` 代理提前到兜底 `/` 前，并使用后端 HTTP 地址加 `ws: true`，避免 SockJS 握手请求 504。
- 修正后端 302 跳转导致的 Vue dev CORS：webpack proxy 增加 `autoRewrite/protocolRewrite/cookieDomainRewrite`，把后端返回的 `Location: 127.0.0.1:7077` 改写回 `localhost:10001`。
- 修正未登录进入 Vue 时自动连接 SockJS 导致 `/ws/endpointChat/info` 被后端 302 到 `/login_p` 的问题：`store` 初始不再创建 WebSocket，登录成功后再连接，并为消息发送增加连接状态保护。
- 排查登录失败：直测 `POST /login` 返回 302 到 `/login_p`，确认后端自定义 `UrlFilterInvocationSecurityMetadataSource` 漏放行 `/login`，已补充放行登录提交地址。
- 继续修正登录失败：由于老 Spring Security 表单登录过滤器未接管 `/login`，新增显式 `LoginController` 复用 `AuthenticationManager` 完成认证、写入 `SecurityContext` 到 session，并保持原成功/失败 JSON 返回格式。
- 继续修正登录失败：将 `/login` 加入 `web.ignoring()`，确保登录请求绕过原安全链重定向，由显式 `LoginController` 处理并写入登录态。
- 为登录接口增加兜底异常 JSON 返回，避免登录内部异常被 `/error` 安全重定向掩盖成 302。
- 修正登录失败计数异常逃逸：`LoginController#setFailNum` 改为捕获所有异常，避免账号不存在或查询异常时跳转 `/error` 后变成 302。
- 修正 dev 库无 `sys_operator.fail_num` 导致认证 NPE/SQL 异常：`SysOperaterService` 对 `failNum` 做空值判断；显式登录接口不再更新缺失的 `fail_num` 字段。
- 按要求不改登录逻辑，撤回显式 `LoginController`、`/login` 绕过安全链和 `failNum` 空值兼容，改为补齐数据库字段。
- 已在 dev 库 `djwms.sys_operator` 增加 `fail_num int not null default 0 comment '登录失败次数'`；保持原 Spring Security 登录逻辑不变。使用账号 `tjct / 18512270349` 直测 `POST /login` 返回 `status=success`。
- 执行 git 提交前记录：本次仅提交开发启动、前端本机后端代理、构建兼容、白屏修复和 dev 数据库 SSL 参数；排除 `target/`、`dist/`、`log/` 等生成物。
- Git 已完成：`omsapi` 提交 `11ea28a 修复后端开发环境启动配置` 并推送 `origin/master`；`omsvue` 提交 `e85b1f0 修复前端开发启动和本机接口代理` 并推送 `origin/master`。
- 再次执行 git 检查：无新的业务代码提交；剩余仅为 `omsapi/target`、日志、`omsvue/dist` 等生成物及换行状态，未提交。
- 梳理项目整体运行逻辑，新增根目录 `PROJECT_RUNNING_LOGIC.md`，覆盖启动、前端路由、登录会话、权限菜单、后端分层、核心业务模块、接口请求、消息、部署和排查流程，并整理 Mermaid 流程图。
- 继续补充 `jxjcmiddleware` 与 `litejxdemo_juzhen`：梳理 IoT 平台订阅、回调接收、RocketMQ 缓冲、NB 报文解析入库、命令下发、示例工程与正式中间件关系，并补充完整业务闭环流程图。
- 更新 `omsapi/.gitignore`，将 `/log/`、`/target/` 目录整体忽略；已用 `git rm --cached` 解除 `target/` 已跟踪文件索引，保留本地文件不删除。
- 明确当前工程主项目为 `omsapi`，后续相关文档统一放在 `omsapi` 项目目录下。
- 根据 `/Users/sunday/Downloads/msnew.sql` 与 dev 库 `djwms` 比对，仅补齐程序实际引用的缺失表/字段，跳过备份、临时、人工留存表；已补齐 `invoice_info`、`nb_deviceorder_history`、`push_message_*`、`transfer_info`、`wechart_invoice_title`、`wt_*` 等表及 `nb_device`、`nb_deviceorder`、`newsun_payment`、`newsun_altermoney` 缺失字段。
- 将前端展示和后端短信注释中的 `天津市蓟州区` 统一改为 `天津市天津港`。
- Git 已完成：`omsapi` 提交 `6b49f73 补齐数据库结构并更新天津港文案` 并推送；`omsvue` 提交 `a504195 更新系统展示为天津港文案` 并推送。
- 修正 Vue dev 环境 `/login_p` 重定向循环：确认后端直接访问 `/login_p` 会 302 到自身；前端 dev proxy 对 `/login_p` 返回 `index.html`，Vue 路由将 `/login_p` 重定向到登录页 `/`。
- 排查首页快捷菜单 `/Rep/Repadd`、`/binghua/BhuserChange`、`/sys/SysSetting`、`/Abn/Abnmeter` 白屏：组件文件存在但登录账号 `tjct` 的动态菜单未返回这些路由；已给角色 `系统管理员(id=6)` 补齐菜单授权，并启用 `炳华业务/BhuserChange` 菜单。重新登录后 `/config/sysmenu` 已返回四个路由。
- 修正首页控制台 Vue warning：`Home.vue` 补充 `imgUrl`、`dialogTitle`、`showPassWord`，并对角色名称做空值保护，避免首页渲染和修改密码下拉菜单报未定义。
- 修正 `SysPaydetail.vue` 控制台 Vue warning：补充 `chargelist`、打印开关、表格选择回调和弹窗关闭回调声明，并将 `show-overflow-tooltip` 改为布尔绑定；`npm run build` 验证通过。
- 继续修正 `SysPaydetail.vue` 与 `Home.vue` 控制台报错：补充 `keywordsChange`，并将 `Home.vue` 通知列表读取改为数组判定，避免 `resp.data.forEach is not a function`；`npm run build` 再次通过。
- 抽取 `BhUserChangeDialog` 公共组件，将 `BhuserChange.vue` 的 IC 卡缴费弹窗改为公共组件调用；补齐 `dialogTitleExchange` 状态并修正关闭端口时 `_this` 未定义问题，消除 `meterExchange/dialogTitleExchange` 未声明 warning；`npm run build` 验证通过。
- 放开前后端 IP 访问：`omsvue` dev server 监听改为 `0.0.0.0`，默认公开地址 `172.18.6.140:10001`，接口代理默认指向 `172.18.6.140:7077`；`omsapi` dev profile 显式绑定 `0.0.0.0`，便于通过 `http://172.18.6.140:10001/` 访问。
- 继续追生产包 `product/msApi-0.0.1-SNAPSHOT`：确认 `/mprice/communityManual` 实际存在于 `ManualPricesController.class`，并且会调用 `AutoPricesService.autoPrices("manual", communityId)`；当前源码补齐同款接口与小区过滤参数，`NewsunAccountMapper.selectAutoAccount` 支持按 `communityId` 过滤。
- 前端补充小区结算页面 `omsvue/src/components/businessManage/BusiCommunityChange.vue`，提供小区下拉、结算确认和 `/mprice/communityManual` 调用。
- 验证结果：`omsvue` `npm run build` 通过；`omsapi` 使用 JDK8 执行 `mvn -q -DskipTests compile` 通过。
- 收紧当前账号菜单树：保留 `档案管理 / 缴费管理 / 表具管理 / 统计分析 / 系统设置`，将 `集中器管理 / NB控制器管理 / 炳华业务 / 环翔业务 / 异常提醒 / 查表科` 及其子菜单全部禁用；新增 `db/patches/patch_trim_menu_to_main_groups_20260811.sql` 作为补丁记录。
- 对照生产包 `product/msApi-0.0.1-SNAPSHOT/META-INF/mapper/EcuHistoryMapper.xml` 修正表具历史查询：生产版标准接口不限制 `ecuId` 长度，当前源码多出的 `length(eh.ecuId)=8` 已移除，兼容 dev 库中的 10/14/15 位表号。
- 将小区缴费管理页面改为表格样式：增加多选框、顶部批量结算、行内结算按钮；批量操作按小区逐个调用现有 `/mprice/communityManual` 接口并汇总成功/失败结果。
- `omsvue` `npm run build` 验证通过。
- 对照生产包 `ExcelImportController` 与 `ExcelImportService`，在 `MetecuNewest.vue` 补充“导入数据”按钮，调用 `/userprofiles/document/importDoc` 上传 Excel，成功后刷新表具列表。

- 增加两项目 `build.sh`：默认用于正式环境 jar 构建，支持 `test/prod` 参数切换；Docker / docker-compose 保持默认测试环境。
- `omsapi` 补充 `application-test.properties` 与 `application-prod.properties`，把数据库、发票、邮件等运行配置按环境拆分。
- `collect-data` 补充 `application-test.yml` 与 `application-prod.yml`，把数据库和 RocketMQ 配置按环境拆分。
- 两个后端项目统一增加 `build.sh`：默认构建正式环境 jar，支持 `test/prod` 参数；Docker 和 docker-compose 默认使用测试环境。
- `omsapi` 未进行高风险 Spring Boot 大版本升级，仅修正必要的 Maven 依赖和环境 profile 配置，启动默认 profile 统一为 `test`。

- collect-data README 增加数据采集流程图，补充 HTTP/RocketMQ 入口、平台适配、报文解析、公共入库和三类数据表节点说明。

- `collect-data` 新增 `docker-compose.yml`，默认以测试环境启动并映射 7078 端口，README 补充 compose 启动方式。
- `collect-data` 同步新增 Dockerfile，多阶段构建 jar，运行时默认 `SPRING_PROFILES_ACTIVE=test`。
- `collect-data` Docker 配置按 `omsapi` 风格对齐：新增 Maven settings 镜像源，日志挂载统一为 `./log:/app/log`，compose 校验通过。
- `collect-data` 增加 RabbitMQ 配置，测试和正式环境均先使用 `172.18.2.14:5672 / guest / guest`，README 同步说明。
- `collect-data` 继承 RocketMQ 消费能力并新增 RabbitMQ 消费能力，两种消息队列统一进入 `CollectDataService`；README 流程图和配置说明已更新。
- 新增 `DOCUSERDOCUMENT_PRODUCT_DIFF.md`，落地用户档案管理几个功能点与 `product` 的对照结果，覆盖停用、启用、落表、复装、换表、卡表换 NB 表和补卡流程。
- `omsapi` 打包名改为 `revenueApi`，同步更新 Maven artifact、构建脚本和 Docker 产物引用。
- 前端登录页参考 `drain-front` 登录风格改造，复用登录背景、标题栏、左侧图和输入框图标；平台文案统一改为 `天津港供水营收管理平台`，`npm run build` 验证通过。
- 继续微调 `omsvue` 登录页头部结构：按 `drain-front` 方式将 `head.png` 作为标题区独立背景图，不再铺满整条 header；确认旧标题 `天津市天津港自来水供水有限公司物联网管理云平台` 已替换为 `天津港供水营收管理平台`，`npm run build` 验证通过。
- 修复 `omsvue` 登录页顶部空白和滚动条：移除 `App.vue` 默认 `margin-top: 60px`，重置 `html/body` 边距和高度，登录页固定 `100vh` 且隐藏溢出；同时修正开发环境 `BASE_API` 默认值为空，避免通过 `172.18.6.140:10001` 访问时直连 `127.0.0.1:7077` 产生 CORS，接口改回同源代理。
- 抽取 `omsvue` 后台顶部为公共组件 `src/components/common/AppHeader.vue`，参考 `drain-front` 顶部栏样式更换导航、用户区和色调；`#/home` 首页卡片与图表容器改为浅色科技风并修复内容被撑开导致的大块空白；顶部 logo 与浏览器 favicon 均改用 `front` 图标，页面标题改为 `天津港供水营收管理平台`，`npm run build` 验证通过。
- 修复抽取顶部组件后后台布局被横向挤压的问题：`Home.vue` 主容器强制纵向 flex，主体区单独横向布局；`AppHeader` 改为普通 `div`，避免 Element `el-container` 未识别子组件导致顶部栏占据左侧整列；首页 ECharts 实例增加 resize/dispose 处理，`npm run build` 验证通过。
- 对照 `drain-front` 表格、输入框、下拉框样式，新增 `omsvue/src/assets/styles/front-ui.css` 作为 Vue2/Element UI 兼容公共样式，仅对带 `ws-page` 类的页面生效；首个页面替换 `MetecuNewest.vue`，将工具栏、高级筛选、表格容器、分页统一套用 `ws-toolbar/ws-filter-panel/ws-table-wrap/ws-pagination`，业务接口与查询逻辑未改，`npm run build` 验证通过。

- 前端公共样式继续收紧自适应宽度：统一 ws-page 下 el-container/header/main、toolbar、筛选面板、表格容器宽度，修复筛选区与表格区视觉宽度不一致；已替换表具历史数据 `#/Met/MetecuHistory`、表具异常提醒 `#/Abn/Abnmeter`、手动扣费管理 `#/busi/BusiCommunityChange`，并保持业务逻辑不变；`npm run build` 验证通过。

- 继续同步 `drain-front` 风格：公共样式补齐按钮高度、最小宽度、表格内按钮紧凑尺寸、日期控件样式；已替换用户档案管理 `#/Doc/DocuserDocument` 与缴费明细 `#/sys/SysPaydetail` 的工具栏、筛选区、表格容器和分页外观，保持业务逻辑不变；`npm run build` 验证通过。

- 用户档案管理页面调整操作按钮：顶部“添加用户档案”改为“添加”，表格操作栏“添加表具”改为“+表具”，与“编辑”并排显示，扩大操作列宽度；未修改业务逻辑。

- 继续统一前端页面样式：用量统计、抄表指令下发、当日异常扣费、预存缴费、余额/表号修改页面套用公共自适应布局和 front 风格按钮；新增公共操作面板与表单区域样式，未修改接口及业务逻辑；`npm run build` 验证通过。

- 前端继续统一页面样式：账户充值管理/机械表缴费的“用水分析”按钮改为蓝色主按钮；档案基础管理 `#/Doc/DocBasic` 及停用启用、落表复装、换表、卡表换 NB 表子页套用公共 `ws-page` 布局、表格、筛选和分页样式；业务逻辑未改。
- 前端品牌展示名调整：界面下拉、表格和单选展示将 `赛恩` 显示为 `晨天`，接口参数、业务判断和后端品牌值仍保留 `赛恩`；`npm run build` 验证通过。

- 小区缴费管理 `#/busi/BusiCommunityChange` 样式调整：批量结算按钮移动到工具栏右侧并保留右边距，按钮颜色由红色改为绿色；行内结算按钮同步为绿色；`npm run build` 验证通过。

- 前端公共分页样式同步 `front`：`ws-pagination` 改为居中白底阴影容器，页码/上一页/下一页增加边框，激活页使用蓝色渐变，跳页和每页条数输入统一高度；小区缴费管理批量结算按钮改为仅多选后显示；`npm run build` 验证通过。

- 综合信息统计 `#/Sta/StaAll` 页面展示优化：保持原 `countsum/cousumlist`、`paylist`、`operlist` 数据接口不变，将页面改为顶部日期筛选、三张汇总指标卡和下方两列统计表布局，提升展示效果；`npm run build` 验证通过。

- 系统设置 `#/sys/SysSetting` 页面布局优化：主页面改为左侧设置导航、右侧渐变标题与内容面板；组织机构子页改为组织树卡片和顶部说明工具栏；数据字典子页套用统一 `ws-page/ws-toolbar/ws-table-wrap/ws-pagination` 样式；原接口和业务逻辑未改，`npm run build` 验证通过。
- 地址管理 `#/sys/SysAddr` 页面布局优化：地址管理拆为城市、区县、街道维护，小区维护独立为 `#/sys/SysCommunityManage`；四个地址维护页统一工具栏、表格、分页样式并铺满宽度，操作按钮保持单行；新增菜单迁移 SQL `db/migrations/20260813_add_sys_community_manage_menu.sql` 并已同步测试库；缴费充值查询 `#/sys/SysPaydetail` 列表上方增加汇总统计卡片；业务逻辑未改，`npm run build` 验证通过。
- 修复后端菜单加载报错：`SysMenuMapper` 会映射 `sys_menu.keepAlive`，但 `SysMenu` 实体缺少 `keepAlive` setter，导致 MyBatis 反射异常；已补齐实体字段并隐藏 JSON 输出。同步去掉缴费充值查询 `#/sys/SysPaydetail` 中按 `printNum` 给整行染绿的旧逻辑，避免误认为默认选中行。前端 `npm run build` 通过；后端使用 JDK8 `mvn -q -DskipTests compile` 通过。
- 修复小区管理菜单未显示：`/sys/SysCommunityManage` 菜单已存在但缺少角色授权，已按拥有地址管理的角色同步授权并更新迁移 SQL；顶部菜单统计分析/系统设置图标从重复 `fa-windows` 调整为 `fa-line-chart/fa-cog`，新增迁移 SQL `db/migrations/20260813_update_top_menu_icons.sql` 并同步测试库；综合信息统计 `#/Sta/StaAll` 日期筛选增加“清除”按钮；`npm run build` 验证通过。
- 缴费充值查询 `#/sys/SysPaydetail` 表格调整：去掉“开票状态”列，操作列固定在右侧并设置固定宽度，表格容器保持浏览器宽度自适应；`npm run build` 验证通过。
- 月度用水分析改为新增独立路由：保留旧 `/Met/MetecuAmountService` 页面和旧接口不变，新增 `/Met/MetWaterAnalysis`、组件 `MetWaterAnalysis` 和后端接口 `/meter/ecu/month-analysis`；新页面使用 `water_usage_period_stat` 周期统计表，提供年度汇总、月度趋势、小区排行、用户类型结构和异常推理清单；新增迁移脚本 `20260813_create_water_usage_period_stat.sql`、`20260813_add_water_analysis_menu.sql`，已同步测试库并生成 154616 条统计数据；前后端构建均通过。
- 首页 `#/home` 重构为营收运营仪表盘：增加大屏标题区、核心营收指标、快捷入口、渠道概览和今日销售统计图区；隐藏未开放快捷入口，保留可用业务入口；新增迁移 `20260814_disable_old_month_usage_menu.sql` 隐藏旧“月度用水分析”菜单，保留旧组件和接口，新菜单使用“用水智能分析”。
- 首页 `#/home` 的“今日销售统计”图表样式优化：保留 `/chartsum/list` 原接口，重绘 ECharts 配色和布局，将金额、售水量、笔数分布与下方售水趋势、用户类型金额条形图分区展示，减少拥挤标签，提升运营看板观感；`npm run build` 验证通过。
- 前端继续细化页面样式：修正首页“今日销售统计”三个圆环中心文字居中；档案基础管理 `#/Doc/DocBasic` 隐藏停用启用、落表复装、卡表换 NB 表页签，仅保留换表列表；公共筛选面板增加内边距；小区缴费管理 `#/busi/BusiCommunityChange` 的批量结算按钮移入表格卡片内，仅勾选后显示且不再占用空白区域；`npm run build` 验证通过。
- 登录页输入框样式修复：增大输入框左侧 padding，固定前缀图标宽度与居中方式，避免账号和密码 placeholder 被图标遮挡；`npm run build` 验证通过。
- 登录页构建后输入框遮挡继续修复：移除 Element Input 的 `prefix` 插槽依赖，改为自定义绝对定位图标容器，并用更高优先级固定输入框左侧间距；已确认打包后的 CSS 包含新样式，`npm run build` 验证通过。
- 根据《晨天营业收费系统需求说明文档》分析“新增抄表管理”模块：需求覆盖抄表册、抄表计划、抄表活动明细、未抄导出、读数导入、计划出账；现有 `newsun_check*` 更偏旧查表科任务，不建议直接复用为新业务主表，建议新增标准化抄表业务表并复用用户档案、表具、历史读数、计费结算能力。
- 新增抄表管理一期功能：后端新增 `reading_book`、`reading_book_user`、`reading_plan`、`reading_activity`、`reading_import_log`、`reading_bill_log` 及 `/reading/**` 接口，支持抄表册关联用户、手动生成计划和活动、未抄明细导出、抄表读数导入、勾选部分实时出账；计费先走 `ReadingPriceService` 固定单价 4.00 元并预留后续阶梯水价扩展。前端新增 `ReadingBook`、`ReadingPlan`、`ReadingActivity` 页面并接入动态路由。迁移 SQL `db/migrations/20260825_create_reading_management.sql` 已同步测试库，前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
# 2026-08-25

- 按 Agentic Codex 规则清理项目根目录说明类 Markdown：`API_SCHEDULED_TASKS.md`、`COLLECT_DATA_PROJECT.md`、`PROJECT_RUNNING_LOGIC.md`、`DOCUSERDOCUMENT_PRODUCT_DIFF.md` 已迁移到 `agentic-codex/agent-docs/project-notes/cbys-omsapi/`。
- `omsapi` 根目录仅保留必要入口文档：`AGENTS.md`、`PROJECT_HISTORY.md`、`README.md`。
- 排查“添加抄表册-客户列表没数据”：测试库 `newsun_userprofiles` 有 6335 条有效客户，`/reading/user/candidates` 登录态返回正常；前端改为打开添加/编辑抄表册弹窗时自动加载候选客户，并对搜索关键字做 URL 编码。`omsvue` 执行 `npm run build` 通过。
- 系统设置新增独立菜单“用户管理”和“角色权限管理”，分别复用现有操作员管理与角色权限组件；新增常规角色：抄表员、营收管理员、客服人员、财务审核、运营查询，并按业务常规初始化菜单权限。迁移 SQL `db/migrations/20260825_add_system_user_role_permissions.sql` 已对照基线结构编写并同步测试库；`/config/sysmenu` 已返回新菜单，前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 权限管理-用户管理增加批量删除：用户表格新增多选列，选择用户后显示“批量删除”按钮，复用现有 `/system/user/deleteoperId/{ids}` 批量删除接口；后端删除用户时同步清理 `sys_operator_role` 用户角色关系，避免残留权限数据。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 系统设置下“阶梯设置”“价格设置”菜单补齐管理员和营收管理员授权：更新 `db/migrations/20260825_add_system_user_role_permissions.sql`，并已同步测试库。账号 `tjct` 重新登录后 `/config/sysmenu` 已返回 `/sys/SysStaircasePrice` 与 `/sys/Pricedetail`。
- 抄表管理出账计费改为优先复用系统已有阶梯/价格模型：`ReadingPriceService` 根据抄表活动的档案和表号查询 `newsun_account.manageNo`、用户人口、年度累计量，调用 `AutoPricesService#getChargeSumResult` 计算应收金额；抄表出账仍落一条汇总扣费记录，`price` 保存本次均价。若价格配置缺失或计算异常，自动回退固定 4.00 元兜底。后端 JDK8 `mvn -q -DskipTests compile` 通过。
- 权限管理-权限组页面宽度修复：`MenuRole.vue` 移除固定 500px 折叠面板宽度，新增自适应容器样式；`SysRolePermission.vue` 外层改为 100% 宽度，解决页面只占一半的问题。前端 `npm run build` 通过。
- 添加抄表册弹窗的“抄表员”改为远程搜索下拉，仅查询角色为“收费员”的启用用户；保存抄表册时同步保存 `readerId`，并保留 `readerName` 用于列表展示。新增后端接口 `/system/user/readers`，前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 表具历史列表 `#/Met/MetecuHistory` 查询优化：`/meter/ecu/historydata` 空表号查询去掉 `ecuId like '%'`，分页 `limit` 不再依赖排序字段；空条件总数改读 `information_schema.tables.table_rows` 近似值，避免每次扫 120 万+ `ecu_history`；新增迁移 SQL `db/migrations/20260825_optimize_ecu_history_query.sql`，给 `ecu_history` 增加 `idx_ecu_history_ecuid_state(ecuId, ecuStateDate, id)`，已同步测试库。后端 JDK8 `mvn -q -DskipTests compile` 通过。
- 前端修复扣费记录查询 `#/busi/Busicharge`、账户充值管理 `#/busi/BusiprePay` 的 loading 位置：去掉搜索输入框上的 `v-loading`，改为表格容器 loading，旋转等待图标只覆盖列表内容区。同步修复顶部菜单点击后左侧菜单不展开/不选中问题：左侧菜单绑定当前路由 `default-active/default-openeds`，路由变化时主动打开对应目录；顶部模块高亮改为递归匹配当前路由。`npm run build` 通过。
- 抄表册管理 `#/reading/ReadingBook`、抄表计划管理 `#/reading/ReadingPlan` 的列表日期列改为 `yyyy-MM-dd HH:mm:ss` 展示。系统类页面规则更新：按钮、搜索框、标题和正文不得贴边，有单条删除的列表默认增加多选批量删除。权限管理、角色权限管理、用户管理、阶梯设置、价格设置开始统一为系统配置页视觉：新增渐变标题区、白底卡片、加大容器内边距、统一表格/工具栏/弹窗色调。`npm run build` 通过。
- 一户多表补充一期支持：后端新增 `/meter/profileMeters?profilesId=` 查询用户档案名下全部表具，`newsun_meter` 增加 `idx_newsun_meter_profiles_ecu(profilesId, ecuId)` 索引并已同步测试库；用户档案管理 `#/Doc/DocuserDocument` 操作栏增加“表具列表”，可查看同一用户下多块表并继续使用现有“+表具”新增流程。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 价格设置页面补充阶梯关联展示：后端 `basic_price_detail` 查询/新增/编辑补齐 `levelNum` 字段映射，前端 `#/sys/Pricedetail` 增加“所属阶梯”列和新增/编辑下拉框，明确价格明细通过 `manageNo + levelNum` 关联阶梯设置。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 价格设置 `#/sys/Pricedetail` 增加筛选和批量删除：支持按用户类别下拉、价格名称、所属阶梯下拉过滤；表格增加多选列，选中后显示“批量删除”，复用现有批量删除接口。前端 `npm run build` 通过。
- 价格设置补充“所属阶梯”字段展示和编辑：后端 `basic_price_detail` 查询映射、添加、更新均支持 `levelNum`，前端 `#/sys/Pricedetail` 表格和弹窗增加第一/第二/第三阶梯选择，明确通过 `manageNo + levelNum` 与阶梯设置关联。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- api 服务构建加速：`build.sh` 默认改为增量快速打包，使用 `-T 1C -Dmaven.test.skip=true package`，需要全量构建时执行 `./build.sh test clean` 或设置 `OMSAPI_CLEAN_BUILD=1`；Dockerfile 增加 Maven 依赖预热层并启用多线程/跳过测试构建；`.dockerignore` 排除文档、SQL、历史和数据库目录，减少 Docker build context。已执行 `./build.sh test` 验证通过，产物为 `target/revenueApi-0.0.1-SNAPSHOT.jar`。
- api `start.sh` 已同步优化：默认 `test` 环境，dev 模式使用 Maven 多线程并跳过测试启动；新增 `OMSAPI_START_MODE=jar ./start.sh` 快速启动模式，存在 jar 时直接运行 `target/revenueApi-0.0.1-SNAPSHOT.jar`，不存在时自动先调用 `build.sh` 构建。已验证 dev 模式参数正常，jar 模式可进入 Spring Boot 启动流程。
- `omsapi/AGENTS.md` 与 `omsapi/PROJECT_HISTORY.md` 已迁移到 `agentic-codex/agent-docs/project-notes/cbys-omsapi/`，业务项目根目录不再保留这两个 Agent 专用上下文文件；目录索引 README 已同步更新。
- 修复登录页点击登录后 loading 出现白色方块问题：移除登录表单容器上的 `v-loading` 白色遮罩，改为登录按钮自身 `:loading`，并补充请求异常时恢复 loading 状态。前端 `npm run build` 验证通过。
- api 启动完成日志补齐：`MsApiApplication` 增加 `ApplicationReadyEvent` 监听，Spring Boot 完全就绪后在命令行输出 `API服务已启动成功`。后端 JDK8 `mvn -q -DskipTests compile` 验证通过。
- 修复页面白屏报错 `this.loadAll is not a function`：移除机械表缴费 `BusiMechanicalMeter.vue` 和组织机构用户管理 `Organization.vue` 中残留的 `this.restaurants = this.loadAll()` 调用，相关 `loadAll` 方法已不存在且该数据未参与当前页面逻辑。前端 `npm run build` 验证通过。生产/测试 nginx 若使用 `BASE_API=/api`，必须配置 `/api/` 反代并剥离前缀，否则后端会把 `/api/login` 当成未授权路径重定向到 `/login_p`，出现 `/api/login_p` 循环跳转。
- 首页“今日销售统计”图表重叠修复：隐藏三个环图外部标签和引导线，数据改由 tooltip 展示；图例上移到环图与下方图表之间，折线/柱状图区域下移；图表容器高度从 420px 调整为 500px，避免文字互相遮挡。前端 `npm run build` 验证通过。
- 首页“今日销售统计”图例继续优化：将环图半径由百分比改为固定像素，避免宽屏下环图过大压住图例；图表高度调整为 560px，图例放到 54%，下方折线/柱状图从 66% 开始，减少文字覆盖统计图。前端 `npm run build` 验证通过。
- 左侧一级菜单交互调整：移除 `Home.vue` 左侧一级目录标题上的 `goMenu(item)` 跳转事件，点击一级菜单只触发展开/收起，不再更改当前页面，也不再自动进入第一个二级菜单；顶部菜单默认进入子页逻辑保持不变。前端 `npm run build` 验证通过。
- 新增 Agent 文档落地规则：后续所有与 Agent 上下文相关的 Markdown 文档统一写入 `agentic-codex` 项目对应目录，不再写到业务项目根目录；写完自动在 `agentic-codex` 中先 `git pull --rebase`，再 `git add`、中文提交、`git push`。
- Agent 文档落地规则已升为 `agentic-codex` 全局通用规则，不再局限于 `cbys-omsapi`：所有项目都适用。
- 继续优化前后端基础能力：前端新增公共分页组件 `WsPagination.vue`，先替换表具最新状态 `MetecuNewest.vue` 和缴费充值查询 `SysPaydetail.vue` 的分页区域，并显式补充 `size` 响应式字段；后端新增 `GlobalExceptionHandler`，统一记录未捕获接口异常的 method、uri、query 和堆栈，返回标准 `RespBean("error", "系统异常，请联系管理员")`。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均验证通过。
- 继续推进前后端基础优化：前端公共分页组件 `WsPagination.vue` 继续替换抄表册管理 `ReadingBook.vue`、抄表计划管理 `ReadingPlan.vue`、抄表活动明细 `ReadingActivity.vue` 三个新增页面；后端新增 `SlowRequestLogFilter`，自动记录耗时超过 3 秒的接口 method、uri、query、status 和 cost，便于后续排查慢接口。前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均验证通过。
- 继续替换高频列表页公共分页：用户档案管理 `DocuserDocument.vue`、表具历史 `MetecuHistory.vue`、表具基础管理 `DocMeter.vue` 改用 `WsPagination`，保持原分页事件、页码和 pageSize 逻辑不变；后端无业务改动，本轮复验前端 `npm run build`、后端 JDK8 `mvn -q -DskipTests compile` 均通过。
- 前端打包输出降噪：`omsvue/build.sh` 增加 `NODE_OPTIONS=--no-warnings` 与 `BROWSERSLIST_IGNORE_OLD_DATA=1`，并使用 `npm run --silent build`；`package.json` 的 `build` 脚本同步设置 `--no-warnings` 和 Browserslist 静默变量。已执行 `./build.sh test` 验证，Node circular dependency warning 与 Browserslist outdated 提示不再输出，dist 与 dist.zip 正常生成。

- 继续统一前端系统设置分页：`SysCity`、`SysArea`、`SysStreet`、`SysCommunity`、`SysCodeTable`、`Organization` 改用公共 `WsPagination`，并通过 `npm run build` 验证。

- 前端提交 `402e492 统一系统页面分页组件` 已推送到 `omsvue/new`：统一系统设置与用户管理相关页面分页组件。

- 确认前端构建产物 `dist.zip` 已纳入 `omsvue/new` 并推送，提交号 `673865e 更新dist.zip文件`。

- 统一缴费管理分页样式：`Busicharge`、`BusimoneyLoad`、`BusiEcuLoad`、`BusiprePay`、`BusiMechanicalMeter` 改用公共 `WsPagination` 和 `ws-page` 白底样式，`npm run build` 验证通过。

- 新增抄表员手机端：前端短地址 `/m` 登录、`/mr` 待抄列表与逐条提交读数；后端新增 `/reading/mobile/tasks`、`/reading/mobile/submit`，按当前登录抄表员过滤任务；API 编译和前端构建通过。

- 修正抄表计划抄表员同步：新建/编辑计划选择抄表册后自动带出周期、例日、抄表员ID和抄表员名称；后端保存计划时以抄表册抄表员为准，API 编译和前端构建通过。

- 明确抄表任务按抄表员ID绑定：手机端待抄列表和提交接口均使用当前登录用户ID匹配 `reading_plan.reader_id`；后端保存计划同步抄表册 `readerId/readerName`，避免姓名手填或不同步。

- 手机抄表端 `/mr` 增加已抄列表：支持待抄/已抄切换，已上传记录可更新读数，已结算记录只读并提示原因；读数前后端均限制为非负整数；API 编译和前端构建通过。
