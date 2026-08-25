# 项目历史记录（压缩版）

用于在会话恢复、插件重载、上下文压缩后，快速恢复当前项目状态。

## 读取顺序

1. 先读 `CURRENT_CONTEXT.md`
2. 再读本文件
3. 需要更早细节时，再查 `PROJECT_HISTORY.archive.md`

## 当前活跃主题

- 2026-08-24 根目录文档迁移补充：`SCHEDULER_AND_SUPPLIER_ACTIVITY_NOTES.md`、`U8_TABLE_REFERENCE.md` 已从主仓根目录移动到 `agentic-codex/projects/smart-cs-ai/`，根目录本地副本已清理。

- 2026-08-24 财务报销开票主体清理：正式库 `finance_reimbursement_invoice_subjects` 已删除“天津宏捷安装工程有限公司”“晨天润泓（天津）科技服务有限公司”2 条，删除后剩 11 条；本地 `backend/sql/create_finance_reimbursement_invoice_subjects.sql` 和 `backend/migrations/versions/fa20260821_create_finance_reimbursement_assistant.py` 也移除了这两条种子数据。验证：全仓 `rg` 无残留、迁移文件 `py_compile`、`git diff --check` 通过。

- 2026-08-24 项目 Agent 文档迁移：建立 `agentic-codex/projects/smart-cs-ai/` 项目目录，迁移 `AGENTS.md`、`CURRENT_CONTEXT.md`、`PROJECT_HISTORY.md`、`PROJECT_HISTORY.archive.md`、`GIT_CHANGELOG.md`；根目录同名文件改为轻量入口指针。用户确认财务报销开票主体中“天津宏捷安装工程有限公司”“晨天润泓（天津）科技服务有限公司”已删除，不再按停用主体处理。

- 2026-08-24 `oa_e9_browser_session_guard` 已改成仅正式环境启动：`APP_ENV=prod/production` 才注册每分钟守护和启动即拉起；本地/测试环境启动 scheduler 时只打印 skipped，不再启动浏览器守护进程。后端 `py_compile app/tasks/scheduler.py` 通过。

- 2026-08-24 已停止测试环境 `oa_e9_browser_session_guard`：测试库 `scheduler_job_meta.is_paused=1`，当前守护进程 `pid=12235` 已结束。

- 2026-08-24 OA E9 独立凭证保活复测：正式环境 `zhidao-cron` 已吃到 `backend/runtime/oa_e9/credentials.json`，守护日志出现 `OA E9 独立凭证文件已刷新，当前会话保持可用。`；当前默认 10 分钟一轮保活。

- 2026-08-24 财务报销助手“更新凭证”已改成当前页内弹窗，不再跳转到后台爬虫页；支持直接在弹窗里保存 cookie、curl，OA E9 还可顺手手动登录和千问识别验证码，保存后仅关闭弹窗并留在原页。

- 2026-08-24 财务报销助手新增导入导出：提供 zip 模板下载，Excel 与 PDF/图片附件放在同一目录；导入时自动上传附件并生成 OSS 地址，支持导出 Excel。另清理 Git 误跟踪的 OA E9 浏览器 profile、缓存、锁文件、运行日志和凭证文件，改为 `.gitignore` 忽略，保留运行时数据。

- 2026-08-24 OKCIS 招投标每日推送日期口径修正：`backend/app/tasks/scheduler.py` 的 `push_okcis_daily_summary` 改为统计昨天的发布日期数据，详情链接日期参数和模板变量 `today_date` 均使用昨天日期；无数据日志文案同步为“昨日”。后端 `py_compile`、`git diff --check` 通过。

- 2026-08-24 OA E9 浏览器会话守护已接入 cron：新增 `backend/app/services/oa_e9_browser_guard.py`，用 Playwright 持久化 profile + 后台巡检常驻维护会话；`backend/app/tasks/scheduler.py` 已加入 `oa_e9_browser_session_guard` 内建调度任务，启动即拉起、每分钟巡检，服务重启后会自动重新执行。另补了 `backend/sql/upsert_oa_e9_browser_session_guard_scheduler_meta.sql` 方便写入调度元数据。

- 2026-08-24 财务报销助手 `1211398` 重新同步完成：新 cookie 已刷新进 `crawler_site_credentials` 和站点文件，`check_oa_e9_login_status` 已通过，`is_nologin=false`。再次跑 `1211398` 9 个附件后，8 个发票号都已识别出来，`41.pdf` 也补齐；附件名和 OSS 备份链接仍完整保留。
- 2026-08-24 财务报销助手前端 OSS 预览错位修复：OSS 预览标签现在只按 pdf/jpg/png 参与配对，`xlsx` 不再混进 OSS 预览行，避免第一条附件名称和第一条 OSS 预览错位。

- 2026-08-24 招标信息公示日期组件增加已有数据标识：公共 `DatePickerInput` 新增 `markedDates`，`/okcis/notices` 返回当前非日期筛选条件下的已有发布日期，列表“发布日期开始/截止”日期组件显示标记点。后端 `py_compile`、前端 `./build.sh` 均通过。

- 2026-08-24 OKCIS 昨日数据补跑完成：正式环境 `crawler_task_3`（run_id `2248`）和 `crawler_task_4`（run_id `2249`）均成功，失败请求均为 `0`。历史详情采集间隔固定至少 5 秒，未取消间隔控制。

- 2026-08-24 OKCIS 周日任务未执行排查：`crawler_task_3/4` 在 2026-08-23 没有运行记录，原因是 cron 写成 `sun-thu`，APScheduler 不支持跨周反向范围，导致任务注册失败。已把代码和正式库 cron 改为 `sun,mon,tue,wed,thu`，并在 `_normalize_standard_cron_day_of_week` 里兼容展开 `sun-thu`；已向正式调度器发送 refresh 和补跑命令，正式库当前 `crawler_task_3` 已在 2026-08-24 08:41:50 开始 running，`crawler_task_4` 等队列继续执行。

- 2026-08-22 财务报销二维码识别继续收口：不再假定二维码在固定位置，OCR 现对 PDF/图片做全页、旋转、半区和角区多轮扫描；只要票面带二维码，就能按二维码拿到票号。已实测 `1211398` 里 9 个附件识别出 7 个发票号，`63.pdf`、`6元.pdf`、`45.pdf`、`47.jpg`、`10元.jpg`、`74元.jpg` 可正常取号。

- 2026-08-22 财务报销票号识别再补一层：阿里云发票 OCR 已过期，`recognize_invoice_file` 现会自动走 PDF 文本层兜底；之前购买方字段的局部正则在空匹配上会炸，已修成安全提取。已用正式附件实测，PDF 兜底能读出票号、抬头和税号，`1170074` 可识别出 `26137000000339213793`。

- 2026-08-22 财务报销附件下载链路修复：确认 OA 附件直链放行依赖浏览器式导航头，原先 `httpx` 请求少了 `Accept-Language / Sec-Fetch-* / DNT` 等头会被 302 到无权限页；已把财务报销附件下载、回填、备份请求改成更接近浏览器的请求头。正式库 `finance_reimbursement_records` 当前 2 条记录都已有 `invoice_no` 和 `invoice_oss_url`，回填脚本小批次运行无待补数据。

- 2026-08-22 OA E9 验证码展示修复：新增后端验证码代理 `/admin/crawler/sites/oa_e9/captcha`，前端携带登录令牌请求并用 Blob URL 展示，绕过 OA 自签名证书/跨域导致的图片加载失败。后端 `py_compile`、前端 `./build.sh` 均通过。

- 2026-08-22 OA E9 手动登录页已补齐：爬虫站点凭证弹窗新增 `username/password/captcha` 手动登录，登录后直接保存 cookie 到系统；后端新增 `POST /admin/crawler/sites/oa_e9/credentials/manual-login`。前端 `./build.sh` 已通过，Node `v20.20.2`；后端 `py_compile` 也已通过。

- 2026-08-22 正式环境 OKCIS 任务调整：crawler_task_3（售后）详情采集开启，周日到周四 21:00；crawler_task_4（售前）详情采集开启，周日到周四 23:00；两条招投标汇总推送改为工作日 09:00。正式库已执行 SQL 并复查生效。

- 2026-08-22 财务报销助手历史修复现状：已给 OCR 增加 PDF 文本层兜底，能补抓清晰电子票里的票号；但历史回填脚本在正式库仍受 OA E9 Cookie 失效影响，当前无法重新下载附件，因此 `oa_request_id=1211399` 这类旧单仍未补出 OSS 和票号。后端 `py_compile` 通过，后续更新一版有效 OA Cookie 后可以继续回填。
- 2026-08-22 财务报销助手继续落地：已直接执行 `finance_reimbursement_invoice_subjects` 和 `finance_reimbursement_records.attachment_names / invoice_oss_url` 到 test/prod 两套库，复查两边各有 13 条开票主体且新增列均已存在；`load_active_invoice_subjects` 已兼容表不存在场景，旧库不会直接 500。`backend` 语法检查通过。
- 2026-08-22 财务报销助手新增校验：OA 报销表单的付款主体与任一发票抬头不一致时会直接拦截整单并发消息，理由使用真实值：`发票抬头（实际识别抬头）与付款主体（OA付款主体）不一致`；当前调试期临时只推送康鹏。历史数据回填仍受数据库连接权限限制未能执行，但代码已兼容旧库和历史展示。
- 2026-08-22 财务报销助手附件名对齐 OA：列表和弹窗现在优先显示 OA 原始文件名，不再只显示占位名；后端会从 `raw_json.file_items` 兜底补历史附件名，并在同步写库时兼容 `attachment_names` 列是否存在。新增回填脚本 `backend/scripts/backfill_finance_reimbursement_attachment_names.py`，但当前数据库连接权限不足，脚本未能实际落库，历史展示先靠接口兜底修正。前端 `./build.sh` 已通过，Node `v20.20.2`。
- 2026-08-22 财务报销助手附件展示优化：列表保留前 4 个附件快捷入口，超出的 `+N` 变成可点击汇总入口，弹窗内列出全部附件链接；继续优先使用 OSS 链接。前端 `./build.sh` 已通过，Node `v20.20.2`。
- 2026-08-22 财务报销助手增强继续收口：新增开票主体管理 tab 和独立权限 `app:finance_reimbursement:admin`；报销列表去掉“当前节点”列，`流程ID` 可直达 OA 表单，默认按 `reimbursement_time DESC, oa_request_id DESC` 排序；发票附件通过 OA Cookie 下载后备份到独立 MinIO bucket，列表优先展示 OSS 链接，`finance_reimbursement_invoice_subjects` 和 `invoice_oss_url` 已补，`doc/database_dictionary.md` 已同步。后端 `py_compile`、前端 `./build.sh` 通过，Node `v20.20.2`。
- 2026-08-21 OA E9 自动登录测试修复：按用户授权，`check_oa_e9_login_status` 调用 `build_site_headers` 时改用 `credentials_payload=normalized`，消除第二参数位置传递引发的 `TypeError`。
- 2026-08-21 OA E9 自动登录排查：Cookie 为空且开关开启时已能进入账号密码兜底链路；但 `getOaValidateCode` 内部命中 `_BLOCKED_KEYWORDS` 后直接返回 `None`，所以登录请求未提交，无法生成 Cookie。未改动用户代码。
- 2026-08-21 新增通用 Agent 代码保护规则：未获用户明确授权，不得删除、改写、禁用或因自身判断改变用户既有代码逻辑；不能执行新增请求时仅说明范围，不得修改原实现。
- 2026-08-21 通义千问普通图片内容理解方法已新增：`TongyiImageUnderstanding.ask_image(image_url, question)` 复用 `TONGYI_API_BASE/TONGYI_API_KEY`，新增 `TONGYI_VISION_MODEL=qwen-vl-plus`；脚本 `backend/scripts/ask_tongyi_image.py` 可直接传图片 URL 和问题调用。该方法用于普通图片/单据/截图内容识别，不处理验证码/校验码图片。后端 `py_compile`、`git diff --check` 通过。
- 2026-08-21 OA E9 爬取站点接入：新增 `backend/crawler_sites/oa_e9/` 与 `app.services.crawler_handlers.oa_e9`，支持通过 `/api/integration/common1/checkSSO` 做 cookie 登录态检查与轻量保活；新增 `backend/sql/upsert_oa_e9_cookie_keepalive_task.sql` 创建每 20 分钟一次的保活任务，默认暂停。未实现验证码自动识别/绕过，OA 要求重新验证码登录时需人工更新有效 cookie。
- 2026-08-21 OA E9 配置名修复：`oa_e9.py` 已兼容 `OA_E9_PASSWORD_LOGIN_FALLBACK_ENABLED`，避免误读旧名 `OA_E9_PASSWORD_FALLBACK_ENABLED` 导致 `AttributeError`；同时清理 Markdown 格式 `base_url` 为纯 URL。后端 `py_compile` 通过。
- 2026-08-20 OKCIS `crawler_task_3` / `crawler_task_4` 已增加“爬取详情页面”开关；关闭后只抓列表，不抓详情页、截止时间、预览和详情图片，前端爬虫任务高级配置已补勾选项，test/prod 两套任务已切到关闭。
- 2026-08-20 `crawler_task_3` 复跑确认不会先清历史公告；`before_run` 只记录“停用启动前全局清理”，首次目标已触发登录权限重试，运行耗时较长，未等完整结束。
- 2026-08-20 付款申请 `requestid=1210398` 字段确认：主表 `formtable_main_37` 没有独立的“未付金额”字段，能直接取到的是 `付款金额(fkje)`，以及关联里的 `合同金额(htje)`、`已付金额(yfje)`、`本次支付金额(bczfje)`；页面上的“未付金额”应为计算值。
- 2026-08-20 台账展示口径里“付款日期”已改名为“付款申请日期”。
- 2026-08-20 采购部助手台账列表 requestid=1210398 已追到 OA 数据源：对应“付款申请”，`workflowid=409`，`formid=-37`，主表 `formtable_main_37`；附件字段 `fj` 里的 3 个文件分别是 `23966发票.pdf`、`23966--合同.pdf`、`恒泰催款.png`，图片/附件仍需通过 `docdetail -> docimagefile -> imagefile` 获取。

- 2026-08-19 营销项目流程看板已改名为“营销项目流程管理”，列表 tab 改为“流程列表”，新增“数据看板” tab；后台卡片、权限描述和 OA 同步调度标题已同步 test/prod。
- 2026-08-19 营销项目流程管理数据看板继续调整：时间轴改为页面宽度自适应，项目行展示开始、整体截止、计划进度；彩色段表示项目开始到当前时间已过去的计划时长，后续计划周期用灰色/虚线区分；已去掉“当前已到：系统当前时间”文案。
- 2026-08-19 数据看板再次调整：已改为独立取全量数据，不再继承列表筛选条件；展示顺序改为按入库时间倒序；“项目总数”备注文案同步为按入库时间倒序展示；时间轴表头已吸顶并上移 10px，时间刻度和“当前”标记位置继续微调。
- 2026-08-19 营销项目流程管理列表时间编辑修复：正在编辑部门开始/结束时间或整体完成时间时，点击页面其他位置会自动提交保存；同时加了重复提交保护，避免外部点击和 blur 连续触发两次请求。
- 2026-08-19 营销项目流程管理展示继续调整：列表 tab 改为“项目列表”；顶部“同步 OA”仅管理 tab 显示，“导出”仅项目列表显示；入库未完成统一显示“进行中”；数据看板主色条和部门条 hover 变粗并显示自定义说明；逾期卡片 hover 显示“项目实际执行时间大于项目预计完成时间，记为项目逾期”。
- 2026-08-19 采购部助手台账管理同步口径已收口到今年：`fetch_ledger_pu_po_rd_join` 现在默认仅查询当前自然年的 `pu_RelPomain` 台账数据，避免历史年份把台账列表撑得过大。

- 2026-08-19 测试环境爬取任务已清空：`crawler_tasks` 4 条任务和对应 `scheduler_job_meta` 的 `crawler_task_%` 记录均已删除，另外测试库所有 `push` 类调度也已删除，正式环境不变。

- 2026-08-19 OKCIS 去重改为按 `uniseq`：`crawler_task_3` / `crawler_task_4` 不再按标题判重，历史命中时只合并订阅组；正式环境 `crawler_task_4` 调度已改为每天 10:00、13:00、15:00 执行，`crawler_task_3` 已还原。

- 2026-08-18 OKCIS 列表订阅组列已改为可换行、最多两行显示，避免多订阅组挤在一行里；前端构建通过。

- 2026-08-18 OKCIS 订阅组下拉数字项修复：后端 `/okcis/notices` 的 `group_options` 不再返回纯数字 `dzid` 作为订阅组选项，前端兜底选项也过滤纯数字；已请求取消正式 `crawler_task_3` 旧运行 `run_id=1972`，但该运行仍处于 `cancel_requested`，新手动执行会被跳过，需等旧运行真正退出后再重跑。
- 2026-08-18 OKCIS 同名标题去重收紧：`crawler_task_3` 现在只在“同名且截止时间 <= 4 天”时跳过；如果同名公告截止时间更远，则仍正常入库，避免把未过期的新公告误判成重复。

- 2026-08-18 OKCIS 同名公告合并多订阅组：`crawler_task_3` 采集时先查同名公告，命中后只补充订阅组列表，不再重复插入；新增 `crawler_okcis_notices.dingzhi_group_names_json` 字段，test/prod 已执行 `ALTER TABLE` 并回填老数据。当前数据页“订阅组”也已支持多选，后端 `/okcis/notices` 与 `/okcis/notices/export` 已支持 `group_names` 多值参数。

- 2026-08-18 招标信息公示订阅组筛选改为多选：当前数据页“订阅组”由单选改成多选，可同时选择多个订阅组后搜索/导出；后端 `/okcis/notices` 与 `/okcis/notices/export` 已支持 `group_names` 多值参数。前端 `cd frontend && ./build.sh`、后端 `py_compile`、`git diff --check` 通过。

- 2026-08-18 `crawler_task_4` 调度和去重规则调整：任务频率改为每 2 小时一次，`crawler_tasks.cron_expr` 与 `scheduler_job_meta.cron_expr` 已同步为 `0 0 */2 * * *`；OKCIS 入库前先按标题检查当天是否已存在，同标题当天重复直接跳过，且仅处理当天数据。

- 2026-08-18 营销项目流程看板左固定列修正：列表页“序号/项目名称”改为内联 sticky 定位并固定宽度/left，`FixedBottomScrollbar` 吸顶表头克隆时同步保留 sticky 样式，解决横向滚动时项目名称跟着移动的问题；前端构建通过。

- 2026-08-18 营销项目流程看板管理页责任人搜索修复：增加每个部门槽位的请求序号，防止旧异步结果覆盖新搜索，并始终保留当前选中责任人；前端构建通过。

- 2026-08-18 营销项目流程看板推送规则调整：正式环境读取四个部门中所有已配置且有效的负责人，配置几个就推送几个人；部分部门未配置不影响其他负责人，全部未配置时自动推送标记为取消。测试环境继续推送康鹏。

- 2026-08-18 营销项目流程看板列表页手机端适配：手机端隐藏宽表格，改用项目卡片展示基础信息和部门时间块；桌面端仍保留宽表格、固定列和底部横向滚动条。筛选栏和顶部按钮改为窄屏可换行布局，前端构建通过。

- 2026-08-18 营销项目流程看板手动推送权限收紧：详情页“手动推送”按钮只对卡片管理员显示，后端接口同步限制为系统管理员权限；后端语法校验、前端构建通过。

- 2026-08-18 营销项目流程看板推送排查与补齐：正式库项目 `2026081707025` 已入库且符合采购会签推送条件但未推送；正式有效负责人当前仅自控部康鹏，待推送 3 条。页面“同步 OA”接口已补充同步后立即触发采购会签推送，并返回推送结果给前端 toast；计划任务原本 10 分钟一次且包含推送逻辑。

- 2026-08-18 正式环境营销看板同步任务无执行记录：`scheduler_job_runs` 中该任务为 0 条，其他计划任务正常；确认正式 cron 当前运行版本未注册 `marketing_project_workflow_board_oa_sync`，需要重新发布并重启正式 cron 服务，数据库任务元数据本身不会自动生成运行实例。

- 2026-08-18 营销项目流程看板同步清理规则收紧：OA 同步时，有任何填写痕迹的旧记录不再删除；只有整行都没填、且任务下发时间早于昨天的空记录才会被清理，避免把已填过数据的项目误删。

- 2026-08-17 正式 `crawler_task_4` 重启后仍卡在订阅组过滤后，最新 `run_id=1923` 未进入第一页。定位为运行目标生成后的进度 `db.commit()` 阻塞；已移除该处阻塞提交并增加 `[TARGETS-READY]` 日志，本地后端语法校验通过，正式环境需部署新代码并重启 cron。

- 2026-08-17 营销项目流程看板采购部会签推送收口：自动推送范围收紧到固定 workflow `369`，且仅在项目已进入“采购部：刘健，部门会签”后才入库与自动推送；自动推送按“项目 + 接收人用户ID”去重，手动推送不限制重复。已直接执行 test/prod 两套库字段和索引补充，并完成 OA 同步清理前置节点数据；推送模板已改为“项目任务时间填写推送”格式，点击详情后直接带链接。

- 2026-08-17 营销项目流程看板补详情页与推送链接：新增 `/apps/marketing-project-board/detail/:rowId`，桌面和移动端都可打开；详情页支持填写部门开始/结束时间和整体完成时间，并增加单条“手动推送”按钮。企业微信推送已改为带“点击详情”链接，直接跳转项目详情；同一任务已加去重，推过一次后不再重复发送。前端构建已通过，后端语法检查已通过。

- 2026-08-14 RCC Reader 新 session `9a39d020-79e0-013f-465e-6a6648471191` 已验证：`reader/v7/user/user_info` 正常返回 `code=10000`，已更新本地凭证文件和正式库 `crawler_site_credentials`；但用于 `leads.api.rccchina.com/api/project/list`、`/project/get` 仍返回 `code=-60 session_is_overdue`，说明 Reader 会话不能直接复用为 leads 项目系统会话。
- 2026-08-14 已新增 RCC Reader 站点爬虫数据表 `crawler_rcc_reader_projects`，并直接执行到 test/prod 两套环境；当前作为后续招投标项目采集的基础表。
- 2026-08-14 RCC Reader Token 续期排查：前端确认使用 `POST /reader/v7/login/auto_login` 重新认证并获取新的 `session_id`，然后将 `reader_auth` Cookie 重置为 7 天有效；未发现独立 refresh token/heartbeat 接口。`/user/user_info` 只是用户信息接口，`acw_tc` 为 1800 秒的站点/WAF Cookie，不等同于登录 Token。
- 2026-08-14 已新增独立 `rcc_reader` 爬虫站点模块：完成 `user_info` 登录检查、`auto_login` 自动续期、AES-256-CTR 响应解密及凭证缓存更新；当前未接入业务数据抓取和入库。验证：生产 Python `py_compile`、RCC 返回数据解密测试、`git diff --check` 均通过。

- OA 售后工单照片数据库定位已完成：`requestid=1207949` 对应 `formtable_main_145.id=131753`；`wxqzp` 保存服务前照片的 `docdetail.ID`，`wxhzp` 保存服务后照片的 `docdetail.ID`。通过 `docimagefile` 可得到实际 `IMAGEFILEID` 和文件名，再通过 `imagefile` 得到 `FILEREALPATH`、`TOKENKEY`。数据库本身不保存公网 URL，在线访问需使用 OA 文件下载接口并携带登录会话。

- `wecom-attendance` 部门筛选已修复“软件研发部 为空”的二次过滤问题；钉钉调休详情兼容 `调休/调休（旧）/调休(旧)`；加班详情已增加汇总兜底
- `wecom-attendance` 调休统计已改为当前年度累计，日期从每年 1 月 1 日统计到当前日期，不再使用页面日期筛选
- `wecom-attendance` 考勤汇总接口已补上排除部门加载兜底：企业微信部门接口不可用时，自动改读本地 `departments` 树，避免 `/daily-records` 直接 500

- `okcis` 采集任务已完成站点目录、手动任务、自动登录、详情抓取、截止时间过滤、专表写入
- `okcis_notice_manual` 规则：
  - 仅保留 `deadline_at` 大于未来 4 天的数据
  - 每次执行前先清理表内不符合规则的旧数据
  - 演示模式会裁剪大字段，避免前端预览异常
- `kehu51` 客户列表与跟进记录持续在修格式化与脏数据问题
- `kehu51` 客户名称现已统一去掉 `▼`
- 前端爬虫任务详情页已修复“执行成功却弹失败”的提示问题

## 近期关键记录

### 355. OKCIS 多订阅组合并补齐
- 2026-08-18 查明正式任务 `crawler_task_3` 虽然有重复标题，但实际运行的 `crawler_handlers/okcis.py` 只会直接跳过历史标题，没有先把当前订阅组合并回旧记录。
- 已修复：历史同名命中时，先合并 `dingzhi_group_names_json`，再继续跳过或入库；合并值只保留订阅组名称，不再混入纯数字 `dzid`。
- 已对正式库按 2026-08-18 原始日志做补写，更新 9 条记录，当前已有 8 条公告出现多订阅组展示。

### 354. 正式库 crawler_task_3 线上数据库复核
- 2026-08-18 已切到正式环境数据库 `backend/config/env_prod` 复核 `crawler_task_3`。
- 最新运行 `run_id=1986` 已成功结束，`success_requests=24`、`failed_requests=0`。
- 本次原始日志持续产出到第 10 页，最终 `crawler_okcis_notices` 仅新增 1 条。
- 结论：正式库不是没跑，而是大部分公告已被站点历史标题去重；这次新增量很少，最终只落 1 条。

### 353. 营销项目流程看板新增
- 2026-08-14 新增首页卡片“营销项目流程看板”，路径 `/apps/marketing-project-board`。
- 权限：普通访问 `app:marketing_project_board:access`；管理 Tab `app:marketing_project_board:admin`。
- 新增表：`marketing_project_workflow_board_rows`（OA 项目任务书镜像）、`marketing_project_workflow_board_department_configs`（部门1-4占位配置）。
- 新增同步脚本 `backend/scripts/sync_marketing_project_workflow_board_rows.py`，从 OA `formtable_main_116` + `workflow_requestlog` 同步项目名称、项目编号、业务负责人、任务下发时间、部门1-4接受/完成时间。
- 新增计划任务 `marketing_project_workflow_board_oa_sync`，默认每 10 分钟同步一次。
- 两套库已直接执行 `backend/sql/create_marketing_project_workflow_board.sql`；test/prod 均已同步 5275 条项目任务书数据。
- 测试库当前缺 `users/permissions/dashboard_app_cards/scheduler_job_meta/scheduler_job_runs` 等基础表，SQL 在 test 只创建业务表和配置表；正式库已写入权限、卡片和计划任务元数据。
- 测试库缺 `scheduler_job_runs` 导致计划任务写执行记录报错已修：`backend/app/tasks/scheduler.py` 的 `_persist_job_run` 在表不存在时跳过持久化，避免任务被执行记录写入失败拖崩。
- 后续调整：部门1-4接受/完成时间默认置空，不再从 OA 流程日志取；test/prod 已清空历史部门时间。
- 删除测试库全量同步正式库任务入口：移除 scheduler 注册、手动执行入口和正式库 `scheduler_job_meta.job_id='test_database_full_sync_from_prod'`。
- 列表新增负责人部门列，来源为业务负责人所在直接 OA 部门；支持负责人部门筛选，点击部门名自动筛选；任务下发时间支持排序。
- 展示层负责人姓名去掉尾部 `VIP数字`；管理 Tab 选择 OA 部门后自动填充部门名称文本框，且可继续手动修改。

### 352. OA 项目任务书表单与列表数据定位
- 2026-08-14 排查 OA“项目任务书”表单。
- 流程定位：现行“项目任务书”主要为 `workflowid=369`，`formid=-116`，主表 `ecology.formtable_main_116`；历史同名 workflow 还有 133/198/261/296，均使用同一主表。
- 主表可获取截图表单字段：项目编号 `xmbh`、项目名称 `xmmc`、项目负责人/业务负责人 `xmfzr`、要求完成日期 `yqwcrq`、会签人 `hqr`、任务类型 `rwlx/ms2`、泵房实际高度 `bfsjgd`、报价方式/税金等。
- `formtable_main_116_dt1` 仅为满意度明细（`bm/yy/myd`），不是截图后面的项目列表。
- 后面表格应按 OA 自定义报表“项目任务书”取数，报表 ID `482/483`，本质仍查询 `formtable_main_116`，并关联 `workflow_requestbase` 得到任务下发时间，关联 `hrmresource` 得到业务负责人姓名。

### 351. 正式环境 crawler_task_4 手动验证
- 2026-08-11 手动运行正式环境 `crawler_task_4` 第 1 页，run_id `1671`。
- 登录检测正常：`is_nologin=False`，没有再出现“自动刷新凭证后仍未登录”。
- 第 1 页源站返回 37 条，原始记录文件写入 37 条。
- 详情 20 秒间隔已生效，日志出现 `[DETAIL-PAUSE] okcis 营销任务详情间隔 20 秒`。
- 运行结果：success，成功 1 页、失败 0，业务表 `crawler_okcis_notices` 入库 6 条。

### 350. OKCIS 登录成功但列表仍 nologin 修复
- 问题：正式环境 `crawler_task_4` 日志显示自动登录成功，但列表接口重试后仍返回 `nologin`。
- 原因：`build_site_headers` 先写入刷新后的 cookie，随后 `extra_headers` 中来自旧 curl/模板的 `Cookie` 头覆盖了新 cookie。
- 修复：`backend/app/services/crawler.py` 的 `build_site_headers` 忽略凭证 headers 和 extra_headers 里的旧 `Cookie`，优先使用 `credentials_payload.cookies` 生成的新 Cookie。
- 验证：`cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler.py app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`、`git diff --check`。

### 349. 正式环境营销 OKCIS 任务节奏调整
- 正式环境 `crawler_task_4` / `okcis_notice_presales_marketing` 已更新：
  - `cron_expr='0 0 14 * * *'`，每天 14:00 开始执行
  - `interval_ms=180000`，页间隔 3 分钟
- `backend/app/services/crawler_handlers/okcis.py`：
  - 营销 OKCIS 任务每条详情间隔 20 秒
  - 详情权限不足重试调整为立即重新登录重试，仍不可见则等待 5 分钟、10 分钟、20 分钟后分别重新登录重试
- `backend/app/api/endpoints/admin/crawler_tasks.py`：
  - 营销 OKCIS 任务分页循环接入任务表 `interval_ms`，下一页前写 `[PAGE-PAUSE]` 日志并等待
- 验证：`cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`、`git diff --check`。

### 348. OKCIS 详情权限不足重试
- `backend/app/services/crawler_handlers/okcis.py` 已补充详情权限不足重试机制。
- 详情页返回权限不足时，每次运行最多触发一轮：
  - 立即清除旧 cookie/重新登录重试
  - 仍不可见则等待 5 分钟后重新登录重试
  - 仍不可见再等待 10 分钟后重新登录重试
- 为避免大量权限不足数据逐条等待，单次运行只对首次权限不足触发等待重试；重试成功则继续正常抓取，重试耗尽后后续权限不足直接跳过。
- 验证：`cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py`、`git diff --check`。

### 347. 正式环境 crawler_task_4 页面数排查
- 正式库任务 `crawler_task_4` / `okcis_notice_presales_marketing` 当前配置：`page_start=1`、`page_end=10`、`page_size=50`，`cron_expr='0 0 16 * * *'`。
- 2026-08-10 最新 run_id `1642`：实际请求第 1-9 页；第 9 页返回 0 条后停止，未请求第 10 页。
- 2026-08-10 原始返回：1-7 页各 50 条，第 8 页 17 条，第 9 页 0 条，共 367 条；业务表 `crawler_okcis_notices` 只入库 7 条。
- 367 条明细筛选结果：有效入库 7 条；过滤 360 条，其中详情权限不足 319 条、截止时间格式无效/无法识别 32 条、截止时间<=未来4天 9 条。
- 2026-08-09 run_id `1604`：第 1 页 18 条，第 2 页 0 条后停止；入库 1 条。
- 2026-08-08 run_id `1566`：第 1 页 28 条，第 2 页 0 条后停止；入库 4 条。
- 结论：任务不是固定只跑 1 页；会跑到空页或最多 10 页。数据量少主要来自源站返回数量和业务筛选后入库数量。

### 346. 测试库每日全量同步正式库计划任务
- 新增计划任务 `test_database_full_sync_from_prod`：测试环境专用，每天 15:00 以正式环境数据库为准，全量覆盖测试环境数据库。
- 新增脚本 `backend/scripts/sync_prod_database_to_test.py`：
  - 读取 `backend/config/env_prod` 和 `backend/config/env_test` 的 `DATABASE_URL`
  - 复制正式库表结构与数据到测试库
  - 目标库安全校验：拒绝同库、系统库、非 test/dev 命名目标库
  - 支持 `--dry-run`
- 后端调度接入 `backend/app/tasks/scheduler.py`，后台计划任务页面接入 `backend/app/api/endpoints/admin/scheduler.py`。
- 新增 SQL：`backend/sql/upsert_test_database_full_sync_from_prod_scheduler_meta.sql`，已在测试库执行，`cron_expr='0 0 15 * * *'`，未暂停。
- dry-run 预检查通过：正式库 `smart-cs-ai`、测试库 `smart-cs-ai-test`，112 张表，0 个视图，预计 226732 行。
- 验证：`cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py scripts/sync_prod_database_to_test.py`、`git diff --check`。

### 345. 生产环境 Python 命令规则补充
- `AGENTS.md` 已补充生产环境执行约定。
- 生产环境没有 `conda` 命令，也不直接使用 `pip` 命令。
- 生产环境 Python 固定使用 `/anaconda3/envs/smart/bin/python`。
- 生产环境安装依赖使用 `/anaconda3/envs/smart/bin/python -m pip ...`，执行 Alembic 优先使用 `/anaconda3/envs/smart/bin/python -m alembic ...`。
- 后续给生产环境命令时，不再写 `conda activate smart`、`conda run -n smart` 或裸 `pip`。

### 344. 税务助手发票文件管理权限调整
- 发票文件管理权限已改为和发票管理一致。
- 前端 `frontend/src/pages/tax_assistant/index.tsx`：Tab 可见性由系统管理员权限改为 `app:tax_assistant:access`，保留 `*` 与 `super_admin` 兼容。
- 后端 `backend/app/api/endpoints/tax_assistant.py`：移除发票文件接口的系统管理员二次校验，文件列表、上传、下载、删除、OCR、导出、扫描推送统一使用 `APP_TAX_ASSISTANT_ACCESS`。
- 验证：后端 `py_compile`、前端 `cd frontend && ./build.sh`（Node `v20.20.2`）、`git diff --check`。

### 343. 客户端管理替换文件支持拖拽
- `frontend/src/pages/client_management/index.tsx`
- 编辑客户端版本弹窗中，“替换客户端文件”改为可点击、可拖拽的上传区域。
- 新建客户端版本的“客户端文件”区域同步支持拖拽。
- 选择文件或拖入文件后自动上传，已去掉独立“上传”按钮；保留上传进度、编辑时覆盖原文件且下载地址不变的逻辑。
- 验证：`cd frontend && ./build.sh`，Node `v20.20.2`；构建同步更新 `frontend/dist.zip`。

### 342. 地下水完整导出失败展示与任务清理
- 调整：
  - 完整导出 `pending/running` 超时判断改为 1 小时
  - 页面去掉“重试”按钮，失败任务只展示失败原因
  - 移除接口 `POST /api/dengji/records/full-export-tasks/{task_id}/retry`，避免连续点击生成重复任务
  - 后续需要重新导出时，直接点击“提交任务”新建完整导出
- 执行：
  - 本地测试库已重新提交一条完整导出任务，成功导出 1793 条
  - 正式库已清除 2 条执行中任务，复查 `running=0`
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py`
  - `cd frontend && ./build.sh`
  - `git diff --check`
- 本次未新增依赖

### 341. 地下水完整导出任务中断处理
- 结论：完整导出使用 FastAPI `BackgroundTasks`，服务重启或进程退出后不会自动续跑，任务可能停在 `running`。
- 调整：
  - `backend/app/api/endpoints/groundwater_registry.py`
  - 新增滞留任务处理：`pending/running` 超过 2 小时自动标记为 `failed`
  - 错误信息提示“任务已中断，可能是服务重启或进程退出导致，请重新提交完整导出。”
  - 提交新完整导出任务、查看任务列表时都会先处理滞留任务
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py`
  - `git diff --check`
- 本次未新增依赖

### 340. 地下水登记完成状态与管理列表
- `groundwater_registry_records` 新增 `is_completed` 字段：默认 `0`，`0=未完成`，`1=已完成`，新增索引 `idx_groundwater_registry_is_completed`。
- test/prod 均已执行 Alembic 升级到 `j1k2l3m4n5o6`，复查 1793 条均为未完成。
- 地下水普通导出和完整导出均增加“填写状态”“完成状态”列；普通导出同步支持填写状态/完成状态筛选。
- `/dengji?admin=1` 新增管理列表视图，展示用户指定列：行政区、镇(街)、许可证编号、取水权人、2026改造、水源转换、资金来源、安装时间、管径、表类型、品牌/型号、接口类型、远传终端、传水资源、超周期、备注、填写状态、完成状态、操作；状态列支持点击切换，操作栏保留。
- 列表筛选增加完成状态；管理表格已压缩字号、列宽和单元格间距；填写状态、完成状态、操作三列固定在右侧；取水权人列约四字宽截断并通过悬浮显示完整内容，2026改造/水源转换/接口类型/远传终端/传水资源约两字宽并允许换行。
- 验证：后端 `py_compile`、`cd frontend && ./build.sh`、`git diff --check`。

### 339. 钉钉考勤助手卡片和权限设置移除
- `dashboard_app_cards` 已移除 `dingtalk_attendance` 卡片定义。
- `permissions.py` 与 `db_init.py` 已删除钉钉考勤权限种子，`wecom_attendance_viewer` 只保留人事考勤权限。
- `create_dingtalk_attendance_records.sql` 与 `update_wecom_attendance_permission_role.sql` 已同步清理钉钉考勤的权限、角色绑定和卡片语句。
- test/prod 两套库已执行删除：`app:dingtalk_attendance:access`、`dashboard_app_cards.dingtalk_attendance`、对应 `role_permissions` 绑定均为 0。
- 验证：`conda run -n smart python -m py_compile backend/app/permissions.py backend/app/db_init.py backend/app/api/endpoints/dingtalk_attendance.py`、`git diff --check`。

### 338. 正式库地下水宁河区剩余数据标记已填写
- 范围沿用第 335 条：正式库 `groundwater_registry_records` 宁河区，排除 `/Users/sunday/Downloads/宁河108口井.xlsx` 108 条后的剩余 544 条。
- 进一步限定这些记录已是 `need_water_source_conversion='是'` 且 `remarks='水务局已确定不用排查'`。
- 更新：`is_filled=1`，`filled_at=COALESCE(filled_at, NOW())`。
- 更新前备份：`/tmp/groundwater_ninghe_is_filled_backup_20260803_150945.csv`。
- 复查：544 条均为已填写且 `filled_at` 非空；排除的 108 条未命中新批次备注组合。

### 337. 项目周期统计提示层样式
- 给排水统计页“项目周期统计”去掉周期条和项目名上的原生 `title`，避免同时出现两个提示。
- 自定义提示改为白底卡片风格，显示项目名、开始日期至截止日期、为期 N 天。
- 提示层通过 `createPortal` 挂到页面层，避免被图表横向滚动容器裁切。
- 验证：`cd frontend && ./build.sh`、`git diff --check`。

### 336. 给排水统计图切换与周期提示
- 给排水统计页“任务数量统计”右上角增加折线图/柱状图切换。
- 默认显示折线图，可切换为柱状图。
- “项目周期统计”周期条悬浮提示补回，内容为“项目名：开始日期 至 截止日期，为期 N 天”。
- 验证：`cd frontend && ./build.sh`、`git diff --check`。

### 335. 正式库地下水宁河区剩余数据更新
- 用户提供 Excel：`/Users/sunday/Downloads/宁河108口井.xlsx`。
- Excel `Sheet1` 共 108 条有效数据，行政区均为宁河区，`序号` 唯一；本次按 `sequence_no` 作为排除键。
- 正式库 `groundwater_registry_records` 宁河区共 652 条，Excel 108 条全部命中并排除。
- 剩余 544 条已更新：`need_water_source_conversion='是'`，`remarks='水务局已确定不用排查'`。
- 更新前已导出本地备份：`/tmp/groundwater_ninghe_update_backup_20260803_143622.csv`。
- 复查：剩余 544 条 `need_water_source_conversion/remarks` 全部符合要求，排除的 108 条未写入该备注。

### 334. 给排水任务数量统计固定展示
- 修复给排水统计页“任务数量统计”看不到的问题。
- 原因：前端仅在 `received_task_trend.length > 0` 时显示图表，日期范围内没有接收任务日期数据时整张卡片隐藏。
- 调整：给排水统计页始终显示“任务数量统计”。
- 后端 `/stats/summary` 按统计页日期筛选补齐完整日期范围，无任务日期返回 `count=0`，保证横轴日期可见。
- 验证：`conda run -n smart python -m py_compile app/api/endpoints/software_task/software_task.py`、`cd frontend && ./build.sh`、`git diff --check`。

### 333. 给排水统计任务数量与大厅描述列
- 给排水统计页新增“任务数量统计”图，使用 `received_task_date` 接收任务日期按天聚合任务数量。
- 该图跟随统计页日期筛选，横轴为日期，纵轴为任务数量。
- 后端 `/stats/summary` 在给排水场景返回 `received_task_trend`。
- 给排水任务大厅在“业务区域”列后面加回“任务描述”列，展示富文本转纯文本后的描述。
- 验证：`conda run -n smart python -m py_compile app/api/endpoints/software_task/software_task.py`、`cd frontend && ./build.sh`、`git diff --check`。

### 332. 通用下拉清空规则
- `agentic-codex/rules/frontend-common-components.md` 已补充强约束。
- 下拉搜索框、可输入下拉框、可选择并回显当前值的下拉控件，必须提供清除当前已填/已选数据功能。
- 除非业务明确禁止清空，否则不能省略清除入口。

### 331. 给排水任务项目编号自动填充与清空
- 给排水新建/编辑任务填写项目名称后，按历史任务项目名称匹配项目编号。
- 匹配到唯一项目编号时自动填充；匹配到多个时展示对应下拉；无历史时回退展示全部项目编号。
- 项目名称、项目编号、业务区域、业务人员输入框补充清空按钮。
- 项目编号手动清空后不会立刻被唯一历史编号自动填回，切换项目名称后重新允许自动填充。
- 验证：`cd frontend && ./build.sh`、`git diff --check`。

### 330. 任务统计页项目周期统计
- 给排水/软件任务统计接口 `/stats/summary` 新增 `project_periods`。
- 统计页在“每日任务完结趋势”下方新增“项目周期统计”卡片。
- 周期统计统一使用统计页日期筛选；筛选计划开始/截止与日期范围有交集的项目任务。
- 前端横轴按天展示，周期条根据项目任务最早计划开始、最晚计划截止绘制，并裁剪到当前筛选范围。
- 给排水按任务项目编号聚合，普通软件任务按项目表编号聚合。
- 验证：`conda run -n smart python -m py_compile app/api/endpoints/software_task/software_task.py`、`cd frontend && ./build.sh`、`git diff --check`。
- Git 提交准备：本次提交合并给排水任务大厅接收任务日期、审核员快捷修改、富文本/详情图片展示、项目编号联动下拉、任务描述列移除、项目周期统计和前端构建包。

### 329. 给排水任务大厅接收任务日期与审核员快捷修改
- `water_supply_tasks_v2` 新增 `received_task_date` 接收任务日期字段。
- 新建、编辑给排水任务均支持填写/回显接收任务日期，列表和详情同步展示。
- 迁移 `i1j2k3l4m5n6_add_water_supply_received_task_date.py` 已在 test/prod 两套库执行，字段复查为 `date`，注释 `接收任务日期`。
- 部门管理员在任务未审核阶段（待领取、已指派、进行中、待审核）可点击列表“审核员”弹窗修改审核员；快捷入口不对已打回、已完结任务开放。
- 新建/编辑任务的富文本工具栏按钮已放开，编辑器外层改为不裁剪工具栏弹层；给排水任务大厅列表已去掉“任务描述”列。
- 任务详情时间轴“任务创建”由纯文本改为渲染富文本 HTML，支持展示描述中的图片。
- 新建/编辑任务“项目编号”下拉已改为按当前项目名称优先筛选历史编号；无历史时回退展示全部项目编号。
- 数据库字典已更新 `water_supply_tasks_v2` 字段数与字段说明。
- 验证：后端 `py_compile`、test/prod `alembic upgrade head/current`、字段复查、`cd frontend && ./build.sh`、`git diff --check`。

### 328. 售前营销业务团队看板定时推送
- 新增任务 `presales_business_dashboard_push`：售前营销业务团队看板推送。
- 调度配置：周一到周五 08:30，cron 表达式 `0 30 8 * * mon-fri`。
- 默认暂停：`is_paused=1`，等后续需要时再在计划任务管理中启用。
- 后端已接入定时执行、手动执行和计划任务管理元数据。
- test/prod 两套库均已写入 `scheduler_job_meta` 并复查一致。
- SQL：`backend/sql/upsert_presales_business_dashboard_push_scheduler_meta.sql`
- 正式模板 `14` 已验证可通过绑定机器人发送月度回款看板图片，企微返回 `errcode=0`。

### 327. 徽章管理默认数据与图标展示
- 修复 `/platform/admin/badges` 页面空白：`Tabs` 改为受控 `value/onValueChange`。
- 默认徽章修正为：`勤劳模范`、`加班楷模`。
- 新增默认徽章 SVG 图标文件，大/中/小三个尺寸各一份；后端挂载 `/api/admin/badge-icons`；图标改为字标徽章风格：`勤劳模范=勤`、`加班楷模=加`。
- 新增迁移 `c4d5e6f7a8b9` 补默认徽章与错字修正，新增迁移 `c5d6e7f8a9b0` 写入默认图标路径。
- test/prod 两套库均已升级到 `c5d6e7f8a9b0`，确认 `badges` 表已有图标路径。
- 徽章配置列表和用户徽章列表均展示徽章图标；图片加载失败时前端显示同字标风格的渐变兜底徽章。
- 验证：后端 `py_compile`、`git diff --check`、`cd frontend && ./build.sh`。

### 326. 系统管理徽章管理
- 新增 `badges` 徽章配置表和 `user_badges` 用户每月获徽章明细表。
- 徽章配置字段包括：徽章编码、名称、描述、大/中/小图标、规则标识、规则说明、规则配置、每月授予上限、启停和公共审计字段。
- 用户徽章按 `period_month` 每月独立统计，唯一约束为同一用户同一月份同一徽章只授予一次。
- 迁移 `b3c4d5e6f7a8_create_badge_tables.py` 默认种子：`勤劳模范`（上个月没有请假）、`加班开模`（上个月加班排名前50）。
- 新增 `/api/admin/badges` 徽章配置 CRUD、启停、用户徽章明细查询和手动授予接口。
- 系统管理首页与模块切换器新增“徽章管理”入口，页面路径 `/platform/admin/badges`。
- 数据库字典已补 `badges`、`user_badges`。
- 当前 `DATABASE_URL` 指向的数据库已执行 `alembic upgrade head`，revision 为 `b3c4d5e6f7a8`。
- 验证：后端 `py_compile`、`alembic heads/current`、`git diff --check`、`cd frontend && ./build.sh`。

### 325. 售前回款筛选控件统一高度
- `MonthRangePicker` 去掉了小号 `AppSelect`，月份下拉恢复公共默认尺寸。
- “回款列表”和“部门回款”两处筛选行的搜索、重置、导出按钮统一为 `h-10 rounded-xl`。
- 回款列表里的合同编号、客户名输入框也同步收齐到同一高度，筛选行视觉保持一致。
- 已验证：`cd frontend && ./build.sh`

### 324. 售前回款列表年月区间筛选与导出
- 回款列表月份筛选改为公共 `MonthRangePicker` 年月区间，默认当年 1 月至当前月。
- 增加部门多选搜索、合同编号、客户名筛选，搜索/重置后生效。
- 列表、顶部统计和新增回款列表导出共用同一筛选条件；导出忽略分页。
- 新增 `GET /api/presales-payment-dashboard/rows/export`，导出列与列表一致：回款日期、申请人、客户名、部门、合同编号、合同名称、回款金额。
- 回款列表工具栏调整为标题单独一行，筛选、搜索、重置、导出、月底待回款尽量同一行；导出按钮位于重置后面。
- 验证：`conda run -n smart python -m py_compile backend/app/api/endpoints/presales_payment_dashboard.py`、`git diff --check`、`cd frontend && ./build.sh`。
- 本次未新增依赖或数据库结构变更。

### 323. 人事考勤缺卡提醒导出口径统一
- 问题：张桐在考勤汇总里未打卡为 `0`，但缺卡提醒导出里仍出现 `3` 次。
- 原因：考勤汇总使用后端 `attendance_summary_items` 聚合结果；缺卡提醒列表和导出又基于前端每日明细重新 `buildSummaryTotals`，口径不一致。
- 修复：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - 缺卡提醒列表直接沿用考勤汇总 `personSummaries` 的旷工/未打卡值。
  - 导出缺卡明细时按人员汇总剩余次数限制，避免导出条数超过汇总值。
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 322. 售前回款部门人工修正
- `presales_payment_dashboard_rows` 新增 `corrected_department_name` 和查询索引，test/prod 已升级到 `r9s0t1u2v3w4`
- 历史数据已回填为 OA 申请部门；新同步数据两个部门字段默认相同
- 同步更新时，仅当修正部门仍等于同步前 OA 部门时才跟随更新；人工修正后不再被任务覆盖
- 汇总、部门筛选、部门排行、看板图片和业绩导出统一按修正部门计算
- 回款列表只显示一列部门，修正值不同时显示“修正部门（修正）”
- 首页“回款列表”操作栏增加“部门修正”弹窗；操作栏权限与管理后台按钮权限一致
- 管理后台“数据处理”操作栏仍只保留排除按钮

### 321. 售前营销回款 OA 定时同步
- 新增计划任务 `presales_payment_dashboard_oa_sync`
- 名称：售前营销回款OA数据同步；分类：营销管理中心部门
- 频率：每 30 分钟
- 同步 OA 回款通知流程到 `presales_payment_dashboard_rows`
- test/prod 均已写入 `scheduler_job_meta`，确认 `is_paused=0`
- 按要求未重启服务，发版启动 cron 服务后生效
- SQL：`backend/sql/upsert_presales_payment_dashboard_oa_sync_scheduler.sql`

### 320. 售前回款部门与跨年年月筛选
- 管理后台“部门回款列表”新增可搜索、多选的部门筛选
- 新增公共组件 `frontend/src/components/ui/month-range-picker.tsx`，支持跨年年月区间
- 可选范围最早取 `presales_payment_dashboard_rows` 最早回款月份，最晚为当前月份
- 默认及重置范围为当年 1 月至当前月份；若最早回款月份晚于当年 1 月，则从最早回款月份开始
- 列表统计和业绩回款 Excel 导出共用部门、年月区间筛选
- 看板“合同回款排行”和“部门回款完成排行”右侧新增独立部门筛选，互不联动
- 已通过后端 `py_compile`、跨年月导出数据查询和前端 `./build.sh`

### 319. 李旭晴企微双账号考勤重复行兜底
- 正式库确认李旭晴主账号为 `lixuqing`，用户 `id=262`，`oa_resource_id=537`。
- 代码中已存在别名组 `lixuqing <-> 15692250985`，本次补齐正式库 `users.sso_uid_aliases=',15692250985,lixuqing,'`。
- 正式库历史日汇总里 `2026-06-25`、`2026-06-26` 存在 `15692250985` 空占位行和 `lixuqing` 实际打卡行并存。
- `daily-records` 接口新增兜底：同一天同姓名同部门已有实际打卡/明细时，忽略没有实际打卡、请假、外出、修正的空占位记录。
- 日汇总缓存版本已提升，避免旧缓存继续展示。

### 318. 人事考勤花名册导出合并单元格报错
- 正式日志显示 `GET /api/wecom-attendance/roster/export?start_date=2026-06-25&end_date=2026-07-28` 返回 500。
- 根因：`_fill_roster_workbook` 清空/填充花名册模板时写入了 openpyxl 的 `MergedCell`，非合并区域左上角单元格为只读。
- 修复：新增 `writable_cell`，写入前将合并单元格映射到合并区域左上角真实单元格，并跳过仍不可写的 `MergedCell`。
- 已验证后端 `py_compile` 和 `git diff --check`。

### 317. 正式环境容器与售后看板企微推送排查
- 正式环境容器状态正常：`zhidao-api`、`zhidao-cron`、`zhidao-dingtalk-stream-worker`、`zhidao-kdocs-browser` 均为 `Up`。
- `zhidao-cron` 容器内当前 DNS 可解析 `qyapi.weixin.qq.com`，HTTPS 连通正常。
- `2026-07-29 08:30` 售后“晨天润达业务团队看板”自动推送失败根因是临时 DNS 解析异常：`ConnectError: [Errno -3] Temporary failure in name resolution`。
- `2026-07-29 08:32` 后续 Redis/手动触发已成功推送，企微 webhook 返回 HTTP 200。
- 已对企微机器人图片发送增加网络/DNS 异常重试：`httpx.RequestError` 按 `0/30/60` 秒最多尝试 3 次，最终失败再落错误日志。
- 已验证后端 `py_compile` 和 `git diff --check`。

### 316. 正式环境计划任务编辑保存按钮不可点
- 现象：正式环境编辑“运行日志清理”任务时，cron 表达式为空，保存按钮禁用。
- 原因：API 容器关闭内置 scheduler 后，部分内置任务缺少数据库 `cron_expr/interval_minutes` 元数据时，列表接口无法再从内存 job 反推默认调度计划。
- 修复：改为由 `cron` 服务启动及每分钟刷新时，把真实注册的内置任务调度计划自动同步到 `scheduler_job_meta`。
- API 管理端以数据库元数据为主要展示来源，数据库里已有的新任务即使未写入 `JOB_META` 也会展示、编辑、暂停和恢复。
- 后续新增内置计划任务时，只要注册到 `cron`，即可自动落库；中文标题、部门、描述和调度计划再通过后台编辑维护。
- Redis 继续用于 API -> cron 的 `run/refresh` 指令队列，不作为唯一元数据来源。
- 不涉及数据库结构变更。
- 已验证后端 `py_compile`、前端 `cd frontend && ./build.sh` 和 `git diff --check`。

### 315. 计划任务列表按部门聚合折叠展示
- `/platform/admin/scheduler` 列表改为按 `business_department` 聚合展示，不再默认平铺所有任务。
- 部门行显示“部门名（前 5 个任务名…）”，任务名预览单行截断，避免任务名过长造成横向滚动。
- 列表容器已去掉横向滚动，表格改为固定布局；长任务名、ID、最近结果截断或换行展示。
- 点击部门名称展开/收起该部门下任务；展开后任务行保留状态切换、编辑、详情、调度计划、下次执行和最近结果。
- 部门排序按运行中任务数倒序；运行中数量相同按总任务数倒序，再按部门名排序。
- 分页改为按部门组分页，底部展示部门数和任务数。
- 已验证 `cd frontend && ./build.sh` 通过，Node `v20.20.2`。

### 314. 计划任务独立 cron Docker 服务
- 新增 `SCHEDULER_ENABLED` 配置开关，Docker 中 `api` 关闭内置 APScheduler，避免 API 发布/重启影响计划任务执行。
- 新增独立入口 `backend/app/scheduler_main.py`，`backend/docker-compose.yml` 增加服务名 `cron`，容器名 `zhidao-cron`，单独运行计划任务。
- `cron` 服务持有 APScheduler，并每分钟刷新数据库中的任务计划、暂停状态和爬虫 cron 任务。
- API 与 cron 通过 Redis 队列 `scheduler:commands` 互通：API 写入 `run/refresh` 指令，cron 消费后执行立即运行或刷新调度状态。
- 管理后台在 API 不启动 scheduler 时仍支持任务列表、详情、编辑、暂停、恢复、运行历史和立即执行。
- Redis 配置沿用现有 `.env` 的 `REDIS_ENABLED/REDIS_URL/REDIS_HOST/REDIS_PORT/REDIS_PASSWORD/REDIS_DB`。
- 已验证后端 `py_compile` 与 `docker compose -f backend/docker-compose.yml config --services`。

### 313. 后端控制台 SQL 输出和日志时间格式
- SQLAlchemy 异步引擎关闭 `echo`，控制台不再输出执行 SQL
- 新增 `backend/logging.ini`，统一输出“时间 | 级别 | 模块 | 内容”
- `start.sh`、Docker Compose、开发重启脚本均已接入日志配置
- 已验证 SQLAlchemy 模块语法、日志配置格式和 `git diff --check`
- 按要求未重启 test/prod 服务，后续正常重启后生效

### 312. 统一运行日志 30 天保留策略
- 检查 test/prod 数据库后确认主要日志体量为 `crawler_task_runs`、`scheduler_job_runs`
- 每日 03:20 清理超过 30 天的 `crawler_task_runs`、`crawler_task_results`、`scheduler_job_runs`、`wecom_message_logs`
- 不清理回款、考勤、客户跟进、采购等业务数据表
- test/prod 已执行迁移 `q7r8s9t0u1v2`，增加日志时间索引
- 历史日志已在 test/prod 两套环境执行清理，当前上述日志表均无超过 30 天记录
- 已通过后端 `py_compile`、`alembic heads` 和 `git diff --check`

### 311. 线上办公助手回收站操作补齐
- 回收站保留单文件恢复，并新增单文件彻底删除
- 支持勾选、多选、全选后批量彻底删除
- 新增清空当前用户回收站功能
- 后端永久删除统一校验文件所有权，只处理当前用户已进入回收站的文件
- 永久删除会同步移除实体文件与 `metadata.json` 元数据，不影响其他用户文件
- 已验证后端 `py_compile`、服务层批量删除及所有权隔离、`git diff --check`、前端 `./build.sh`（Node `20.20.2`）

### 310. 采购需求统一四类 OA 来源
- 后续采购需求获取范围：
  - 采购申请：`workflowid=279`，主表 `formtable_main_47`
  - 项目需求单：`workflowid=345`，主表 `formtable_main_65`
  - 项目需求单变更：当前启用 `workflowid=416`，主表 `formtable_main_67`
  - 项目需求单补充流程：当前启用 `workflowid=281`，主表 `formtable_main_82`
- 查询实现通过 `workflow_requestbase -> workflow_base -> workflow_bill` 动态定位，兼容历史流程版本。

### 309. 看板推送图片日志定时清理
- 看板图片由内存直接生成并发送，不保存本地图片文件
- 新增每日 03:20 清理任务 `dashboard_image_message_log_cleanup`
- 售前/售后看板图片发送日志保留 30 天，超期自动删除
- 已通过后端 `py_compile` 和 `git diff --check`

### 308. 新增售前营销业务团队看板推送模板
- 复制模板“晨天润达业务团队看板推送”，新增“售前营销业务团队看板推送”
- 模板键：`presales_business_dashboard_push`
- test/prod 两套数据库均已执行
- 未修改原售后模板、机器人绑定和现有推送任务
- SQL：`backend/sql/upsert_presales_business_dashboard_push_template.sql`

### 307. 部门回款完成情况按完成率排名
- 月度回款看板“部门回款完成情况”改为按完成率降序
- 完成率相同时按回款金额降序

### 306. Git 提交准备记录
- 范围：
  - 售前业绩回款按月份区间动态导出及时间标识文件名
  - 回款记录管理员排除及统计统一过滤
  - 管理后台数据处理 Tab、多条件 OR 筛选、U8 客户名称同步
  - 部门回款月份搜索与列表/导出联动
  - 首页和数据处理公共分页组件、数据处理浮动横向滚动条及操作列
- 数据库：
  - test/prod 已升级到 `p6q7r8s9t0u1`
  - 两套环境已完成 OA 回款全量同步和客户名称回填
- 验证：
  - 后端 `py_compile`、`alembic heads` 通过
  - 前端 `cd frontend && ./build.sh` 通过
  - `git diff --check` 通过

### 295. 售前回款记录管理员排除
- 调整：
  - 回款列表新增管理员专属“操作”列及“排除”按钮
  - 新增 `PATCH /api/presales-payment-dashboard/rows/{row_id}/exclude`
  - 排除操作仅允许 `app:presales_payment_dashboard:admin` 权限
  - 管理员列表可继续查看已排除记录，普通列表和回款看板默认过滤
  - 汇总、部门/月度统计、合同排行及业绩回款 Excel 导出统一过滤已排除记录
- 数据库：
  - `presales_payment_dashboard_rows` 新增 `is_excluded`、`excluded_at`、`excluded_by_user_id`
  - 新增索引 `ix_presales_payment_dashboard_is_excluded`
  - 数据库字典已更新
  - test/prod 均已升级到 `o6p7q8r9s0t1`，字段和索引核对通过
- 验证：
  - 后端 `py_compile` 通过
  - `alembic heads` 为 `o6p7q8r9s0t1`
  - `cd frontend && ./build.sh` 通过（Node `v20.20.2`）
- 本次未新增依赖

### 296. 回款单 1160005 本地入库核对
- test/prod 的 `presales_payment_dashboard_rows` 均已存在该记录
- 本地记录：`id=11994`、`source_id=12113`、`oa_request_id=1160005`
- 合同编号 `CT2024041207014`，回款金额 `-2190000.00`，回款日期和申请日期均为 `2026-06-26`
- 当前 `is_excluded=0`，正常参与 2026 年 6 月统计
- test/prod 索引核对：`oa_request_id` 已有 `ix_presales_payment_dashboard_request_id`；月份、部门、合同编号、申请人和排除状态对应字段也均已有索引

### 297. 售前回款管理后台返回列表
- 管理后台页头增加“回款列表”按钮，点击切回回款列表
- 切换视图不重置原列表月份筛选和分页状态

### 298. 售前回款管理后台数据处理 Tab
- 管理后台新增“数据处理”Tab，列表结构与首页回款列表一致
- 数据处理 Tab 独立支持月份筛选和分页，并保留已排除记录
- 操作列和排除按钮仅在数据处理 Tab 展示，首页回款列表已移除操作列
- 排除后同步刷新数据处理列表和首页统计数据
- 已验证 `cd frontend && ./build.sh`、相关文件 `git diff --check`

### 299. 数据处理多条件 OR 筛选和客户名称同步
- 数据处理 Tab 去掉月份筛选，新增项目编号、流程ID、合同编号、客户名，多个条件按 OR 关系查询
- 流程ID使用本地 `oa_request_id`；项目编号、流程ID、合同编号已有索引
- `presales_payment_dashboard_rows` 新增 `customer_name` 和索引 `ix_presales_payment_dashboard_customer_name`
- 客户名称同步规则：读取 OA `formtable_main_117.kh/khbh` 客户编码，通过 U8 `dbo.Customer.cCusCode` 获取 `cCusName`
- test/prod 均已迁移到 `p6q7r8s9t0u1` 并全量同步 12179 个 OA 源行
- 两套本地各 12180 条记录，其中 11503 条已回填客户名称；`1160005` 为“天津大冢饮料有限公司”
- 验证：后端 `py_compile`、`alembic heads`、U8 客户映射实测、前端 `cd frontend && ./build.sh`、相关文件 `git diff --check`
- 本次未新增依赖

### 300. 数据处理列表列调整
- 数据处理列表去掉申请人列
- 新增客户名、流程ID（`oa_request_id`）、项目编号列
- 首页回款列表保持原列结构不变

### 301. 部门回款月份搜索联动修复
- 原因：搜索按钮只更新导出月份状态，未重新请求部门汇总数据，所以列表无变化
- 后端 `summary` 新增 `start_month`、`end_month` 区间参数，回款、合同、部门和目标金额按区间汇总
- 部门回款搜索、列表和导出统一使用同一生效月份区间；重置恢复 1 月至当前月
- 编辑目标、排除记录后会刷新当前区间的部门数据
- 实库验证：2026年6月 188 条/12 个部门，7月 140 条/11 个部门，金额不同
- 已验证后端 `py_compile`、前端 `cd frontend && ./build.sh`、相关文件 `git diff --check`

### 302. 数据处理列表浮动横向滚动条
- 数据处理列表接入公共 `FixedBottomScrollbar`
- 列宽超过屏幕且原生横向滚动条不在视口时显示底部浮动滚动条
- 滚动到列表底部、原生横向滚动条进入视口后，浮动滚动条自动隐藏
- 本次只调整数据处理列表，首页回款列表和部门回款表不变

### 303. 数据处理列表公共分页组件
- 数据处理列表改用公共 `PaginationControl`
- 增加每页 10、20、50、100 条下拉选择
- 切换每页条数后自动回到第 1 页并重新加载列表

### 304. 回款列表公共分页和数据处理浮动操作列
- 首页回款列表改用公共 `PaginationControl`
- 首页增加每页 10、20、50、100 条下拉选择，切换后回到第 1 页
- 数据处理操作列使用右侧粘性定位；未滚到最右侧时浮动显示，滚到真实操作列位置后自然归位

### 305. 部门回款动态导出文件名
- 导出统计月份与部门回款搜索后生效的开始/结束月份一致
- Excel 仅生成筛选月份区间对应的数据列
- 文件名增加月份范围和 `YYYYMMDD_HHMMSS` 时间标识，避免重复下载时重名
- 实测 5 月至 6 月导出仅生成 5 月、6 月共 9 列，文件名示例：`售前业绩回款统计表_2026年5月-6月_20260728_111222.xlsx`
- 已验证后端 `py_compile`、相关文件 `git diff --check`

### 294. 售前业绩回款导出月份区间
- 调整：
  - 管理后台月份筛选改为开始月份至结束月份，支持连续多月区间
  - 默认区间为当前年份 1 月至当前月
  - 导出接口支持 `start_month`、`end_month`，回款、目标、月底待回款均按区间过滤
  - Excel 只保留所选月份区间对应的表头和数据列
- 排查：
  - OA 回款单 `requestid=1160005` 的实际回款日期、申请日期均为 `2026-06-26`
  - 当前按 `COALESCE(payment_time, apply_date)` 统计，该单归入 `2026年6月`
- 验证：
  - 后端 `py_compile` 通过
  - 5 月至 6 月导出实测生成 9 列，月份表头仅为 5 月、6 月
  - `cd frontend && ./build.sh` 通过（Node `v20.20.2`）
  - 相关文件 `git diff --check` 通过
- 本次未新增依赖

### 293. 售前营销回款导出入口和月份筛选
- 调整：
  - 导出按钮从普通回款列表移到管理后台
  - 管理后台导出区域新增月份筛选、搜索、重置、导出按钮
  - 导出接口新增 `end_month` 参数，按当前年份 1 月到所选月份导出
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/presales_payment_dashboard.py`
  - `git diff -- backend/app/api/endpoints/presales_payment_dashboard.py frontend/src/pages/presales_payment_dashboard/index.tsx --check`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 292. Git 提交准备记录
- 范围：
  - OA 项目需求单、项目需求单变更、项目需求单补充流程数据库表映射
  - 正式环境软件部日报提醒未推送原因排查
  - 临时停用日报提醒的未打卡过滤，保留可恢复开关
- 验证：
  - 后端 `py_compile` 通过
  - `git diff --check` 通过

### 291. 软件部日报提醒临时停用未打卡过滤
- 调整：
  - `backend/app/tasks/scheduler.py` 新增 `_SOFTWARE_WORK_RECORD_REQUIRE_ATTENDANCE = False`
  - 日报提醒当前不再以昨日实际打卡作为进入缺失名单的前置条件
  - 原打卡查询、身份匹配和无打卡跳过逻辑保留在开关分支内
  - 后续考勤改为每日同步后，将开关改回 `True` 即可恢复
- 验证：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
  - `git diff --check -- backend/app/tasks/scheduler.py`
- 本次未新增依赖

### 290. 正式环境软件部日报提醒未推送排查
- 排查日期：`2026-07-28`，任务检查 `2026-07-27` 日报
- 结果：
  - `software_work_record_reminder` 09:00 定时执行成功，09:39 手动执行也成功，任务未暂停
  - 正式库软件开发部 10 人中，8 人已写日报；李鹏瑞、王业龙未写
  - `attendance_checkin_records` 的数据最大日期为 `2026-07-25`，`2026-07-27` 为 0 条
  - 代码只提醒“有实际打卡但未写日报”的人员，两名未写人员因无本地打卡数据被排除，因此未调用企微机器人
- 根因：
  - 考勤同步任务 `wecom_dingtalk_attendance_sync` 当前 cron 为每月 26 日 03:50
  - 日报提醒每天 09:00 依赖昨日考勤，月度同步无法提供每日最新打卡数据
  - 7 月 26 日同步任务成功，但同步结束日期为 7 月 25 日
- 日志问题：
  - 空名单分支内部记录“已跳过推送”后返回 `None`
  - 外层任务包装器用通用“软件部日报提醒执行完成”作为最终摘要，内部结果未进入持久化运行摘要

### 289. OA 项目需求变更与补充流程表定位
- `项目需求单变更`：
  - 当前启用版本 `workflowid=416`，`ISVALID=1`，版本 `7`
  - `formid=-67`，主表 `formtable_main_67`
  - 明细表 `formtable_main_67_dt1` 至 `formtable_main_67_dt4`
  - 历史 workflow ID：`93、158、164、242、346、385`
- `项目需求单补充流程`：
  - 当前启用版本 `workflowid=281`，`ISVALID=1`，版本 `4`
  - `formid=-82`，主表 `formtable_main_82`
  - 明细表 `formtable_main_82_dt1`
  - 历史 workflow ID：`100、156、243`

### 288. OA 项目需求单流程表定位
- 排查对象：`requestid=1150604`，页面标题“项目需求单”，状态“出口10”
- 结论：
  - `workflow_requestbase.workflowid=345`
  - `workflow_base.formid=-65`
  - `workflow_bill.tablename=formtable_main_65`
  - 主表：`ecology.formtable_main_65`，本条 `id=1273`
  - 明细表：`formtable_main_65_dt1` 至 `formtable_main_65_dt5`，通过 `mainid=1273` 关联，本条当前均无明细行
- 页面部门数据：
  - 自控部描述/附件：`jsybms`、`jsybfj`
  - 给排水部描述/附件：`jsebms`、`jsebfj`
  - 环境科技描述/附件：`hjkjms`、`hjkjfj`
  - 项目部描述/附件：`azbms`、`azbfj`
  - 相关附件：`xgfj`
- 关联表：
  - 附件：`docdetail`、`docimagefile`、`imagefile`
  - 人员部门：`hrmresource`、`hrmdepartment`
  - 审批：`workflow_requestlog`、`workflow_currentoperator`、`workflow_nodebase`
  - 关联合同流程 `requestid=1145256` 为“合同审批”，业务主表 `formtable_main_59`

### 287. OA 采购申请流程表定位
- 排查对象：`requestid=1174928`，页面标题“采购申请-齐维彬-2026-07-25”
- 结论：
  - `workflow_requestbase.workflowid=279`
  - `workflow_base.formid=-47`
  - `workflow_bill.tablename=formtable_main_47`
  - 主表：`ecology.formtable_main_47`，本条 `id=6478`
  - 明细表：`ecology.formtable_main_47_dt1`，通过 `mainid=6478` 关联，本条明细 `id=4954`
- 关键数据：
  - 项目编号 `xmbh=2026011404003`
  - 项目名称 `xmmc=天津市宝坻区泉州水务泵站运维（2026年）`
  - 申请人 `sqr=737`，对应 `齐维彬`
  - 申请部门 `sqbm=14`，对应 `售后服务部(天津)`
  - 申请单位 `sqdw=10`，对应 `晨天润达（天津）科技服务有限公司`
  - 附件字段 `fj=1103300`，对应 `docimagefile.IMAGEFILENAME=7-25润达采购.xlsx`

### 286. 计划任务暂停状态改为持久化
- 问题：
  - 内置计划任务暂停只暂停当前进程内 APScheduler，未写入数据库
  - 服务重启或调度器重新初始化后，会按默认配置恢复运行
- 调整：
  - `scheduler_job_meta` 新增 `is_paused` 字段
  - 内置任务暂停/启用会写入 `scheduler_job_meta.is_paused`
  - 调度器启动时 `apply_scheduler_meta_overrides` 会按 `is_paused` 恢复暂停/运行状态
  - 爬虫任务仍沿用 `crawler_tasks.is_enabled`
  - 已更新 `doc/database_dictionary.md`
- 执行：
  - test/prod 均已执行 `alembic upgrade head`
  - test：`smart-cs-ai-test.scheduler_job_meta.is_paused` 存在，`alembic_version=n5o6p7q8r9s0`
  - prod：`smart-cs-ai.scheduler_job_meta.is_paused` 存在，`alembic_version=n5o6p7q8r9s0`
- 验证：
  - 后端 `py_compile` 通过
  - `cd backend && conda run -n smart alembic heads` 通过
- 本次未新增依赖

### 283. 管理工作台今日活跃改为今日登录数
- 调整：
  - `users` 表新增 `last_login_at` 字段，并更新 `doc/database_dictionary.md`
  - 企微 SSO、企微敏感字段授权、开发登录成功后更新 `last_login_at`
  - `/api/admin/stats/dashboard` 的 `active_users_today` 改为统计 `last_login_at >= 当天 00:00:00` 的用户数
  - 登录成功后清理后台统计缓存；统计缓存最长只保留到当天结束
- 执行：
  - 当前环境已执行 `cd backend && conda run -n smart alembic upgrade head`，升级到 `m5n6o7p8q9r0`
  - 2026-07-28 已按用户要求补执行 test/prod 两套环境迁移：
    - test：`smart-cs-ai-test.users.last_login_at` 存在，`alembic_version=m5n6o7p8q9r0`
    - prod：`smart-cs-ai.users.last_login_at` 存在，`alembic_version=m5n6o7p8q9r0`
- 验证：
  - 后端 `py_compile` 通过
  - `cd backend && conda run -n smart alembic heads` 通过
  - `cd backend && conda run -n smart alembic upgrade head` 通过
- 本次未新增依赖

### 282. 去除工作台应用入口数量文案
- 调整：
  - `/platform/dashboard` 智能应用标题右侧不再展示“X 个运行中 · Y 个规划中”
  - `/m/dashboard` 应用入口标题右侧不再展示“X 个运行中”
- 验证：
  - `cd frontend && ./build.sh` 通过
- 本次未新增依赖

### 284. 售前营销回款导出业绩统计表
- 调整：
  - 模板来自 `/Users/sunday/Downloads/业绩回款统计表.xlsx`，已保存为 `backend/static/presales-payment-performance-template.xlsx`
  - 新增 `GET /api/presales-payment-dashboard/export-performance`
  - 导出文件名固定为 `售前业绩回款统计表.xlsx`
  - 导出范围为当前年份 1 月到当前月
  - 数据按部门和月份聚合，填充 `当月目标`、`当月实际`、`完成率`、`月底待回款金额`
  - 当月实际来自 `presales_payment_dashboard_rows.payment_amount`
  - 当月目标来自 `presales_payment_department_monthly_targets.target_amount`
  - 月底待回款金额来自 `presales_payment_user_month_end_pending.pending_amount`，按部门汇总
  - 售前营销回款管理回款列表工具栏新增“导出”按钮
  - 导出权限沿用目标管理权限：售前回款管理员或营销管理相关角色
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/presales_payment_dashboard.py`
  - `git diff -- backend/app/api/endpoints/presales_payment_dashboard.py frontend/src/pages/presales_payment_dashboard/index.tsx --check`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 285. Git 提交准备记录
- 范围：
  - 管理工作台今日活跃改为今日登录数
  - 去除工作台应用入口数量文案
  - 售前营销回款导出业绩统计表
  - 前端构建包 `frontend/dist.zip`
- 验证：
  - `git diff --check` 通过
  - 后端 `py_compile` 通过
  - `cd backend && conda run -n smart alembic heads` 通过
  - `cd backend && conda run -n smart alembic upgrade head` 通过
  - `cd frontend && ./build.sh` 通过

### 281. 初始化 Agent 通用规则与架构方案库
- 调整：
  - `agentic-codex/` 新增通用入口 `AGENT.md`
  - 新增通用规范目录与文件：`rules/`、`instructions/`、`workflows/`
  - 通用规则已补充 `frontend-common-components.md`，覆盖下拉框、日期选择、横向滚动条、筛选分页联动等前端公共组件规则
  - 新增 `architectures/` 架构方案目录
  - 沉淀当前项目参考方案：`agentic-codex/architectures/smart-cs-ai-reference-architecture.md`
  - 新增可直接查看的框架流程图：`agentic-codex/architectures/smart-cs-ai-framework-flow.svg`
  - 根目录 `AGENTS.md` 增加共享规则与架构方案引用入口，并要求会话初始化先读取 `agentic-codex/AGENT.md`
  - 根目录 `AGENTS.md` 中通用工作原则、禁令、流程、依赖规则已收口为引用，保留项目专属落地规则
- 说明：
  - `agentic-codex` 用于跨项目共享 Agent 行为约束、代码规范、工作流和业务架构方案
  - 当前项目专属规则仍保留在根目录 `AGENTS.md`
- 验证：
  - 本次仅文档与规则文件调整，未改业务代码
- 本次未新增依赖

### 280. 售前营销回款看板最近回款数量
- 调整：
  - 回款看板“最近回款”改为只展示前 5 条
- 验证：
  - `git diff --check` 通过
  - `cd frontend && ./build.sh` 通过

### 279. 售前营销回款管理个人月底待回款金额
- 调整：
  - 新增表 `presales_payment_user_month_end_pending`，按 `year + month + user_id` 保存个人月底待回款金额
  - 保存记录包含系统用户 ID、OA 人员 ID、用户姓名、部门快照、月份、金额、创建/更新时间
  - 新增 `GET/PUT /api/presales-payment-dashboard/month-end-pending`
  - 回款列表新增“月底待回款”按钮，弹窗内可选择月份并填写金额（万元）
  - 已更新 `doc/database_dictionary.md`
- 执行：
  - 当前环境已执行 `cd backend && conda run -n smart alembic upgrade head`，升级到 `l4m5n6o7p8q9`
- 验证：
  - 后端 `py_compile` 通过
  - `cd backend && conda run -n smart alembic heads` 通过
  - `cd frontend && ./build.sh` 通过

### 278. 售前营销回款管理列表月份筛选
- 调整：
  - 普通回款列表新增“月份”筛选，支持全部月份和 1-12 月
  - 切换月份会重置分页到第一页
  - 列表和顶部统计同步传递 `month` 参数，保证筛选口径一致
- 验证：
  - `git diff --check` 通过
  - `cd frontend && ./build.sh` 卡在 `tsc && vite build` 无输出，已中断，未发现遗留构建进程

### 277. 售前营销回款看板部门目标弹层
- 调整：
  - 管理后台表格新增“操作”列和“编辑”按钮
  - 点击后弹出部门回款目标编辑层，按月份展示并可修改已有目标
  - 列表标题收口为“部门回款列表”
  - 回款目标弹层已缩窄为小弹窗
  - 月度回款看板中“合同回款排行”移到左侧，“最近回款”下移

### 276. 售前营销月度回款看板收口目标列表
- 调整：
  - 管理后台只保留“部门月度回款列表”，去掉目标列表编辑
  - 统计仍按部门、按月展示已有回款数据，每行一个月份，只显示有数据的月份
  - `marketing_dept_member` 与售前管理员均可见管理后台按钮
- 说明：
  - OA 回款同步仍保持到个人粒度，不改同步格式
  - 看板汇总继续按部门统计，不再走个人排行

### 275. 地下水登记行政区联动镇街筛选
- 调整：
  - `/api/dengji/options` 增加 `district` 参数
  - 首页筛选选择行政区后，镇(街)候选列表只显示该行政区下的数据
  - 行政区变化时自动清空已选镇(街)，避免跨区筛选残留
  - 编辑页仍使用全量候选，不受首页联动影响
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 274. 地下水登记完整导出改为异步任务
- 调整：
  - 普通“导出”固定全量导出，不带站点图片，不再受首页筛选影响
  - PC 端新增“完整导出”tab；固定全量带站点图片，后台异步执行
  - 完整导出提交后写入 `groundwater_registry_export_tasks`
  - 页面每 5 秒刷新任务列表，成功后展示下载地址和下载按钮
  - 移动端不展示完整导出 tab
  - 首页列表筛选“已填”时按更新时间倒序
  - 新增导出文件目录：`backend/data/exports/groundwater_registry`
  - 新增迁移：`j2k3l4m5n6o7_create_groundwater_registry_export_tasks.py`
  - 已更新 `doc/database_dictionary.md`
- 执行：
  - 本地/测试库已执行 `alembic upgrade head`
  - 正式环境已执行：`git pull` 显示 `Already up-to-date`
  - 正式环境容器内执行 `python -m alembic upgrade head`，已升级到 `j2k3l4m5n6o7`
  - API 容器确认运行中
  - 正式接口验证：`/api/dengji/records/full-export-tasks` 返回 200；`/api/dengji/records/export` 返回 200 且文件为 xlsx
  - 注意：正式服务器工作区存在既有 `frontend/dist.zip` 删除状态，本次未处理
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py app/models/groundwater_registry.py app/core/config.py migrations/versions/j2k3l4m5n6o7_create_groundwater_registry_export_tasks.py`
  - `cd backend && conda run -n smart alembic heads`
  - `cd backend && conda run -n smart alembic upgrade head`
  - `cd frontend && ./build.sh`
  - `git diff --check`
- 本次未新增依赖

### 273. 地下水登记正式环境导出 500 排查
- 问题：
  - 正式环境 `/api/dengji/records/export` 报 `AttributeError: 'MergedCell' object attribute 'value' is read-only`
  - 报错点为导出时直接写入 Excel 合并单元格子格
- 结论：
  - 当前仓库 `master` 最新提交 `f354ec3` 已包含修复：导出前取消模板数据区合并单元格，再写入数据库数据
  - 正式环境若仍报该错，应执行 `git pull` 并重启 API 容器后再试
  - 本次未操作正式环境文件
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py scripts/reset_groundwater_registry_records.py`
  - 临时合并单元格写入测试通过
- 本次未新增依赖

### 272. 地下水登记模板合并单元格和图片处理
- 调整：
  - 项目内地下水登记模板已替换为用户提供的最新 Excel
  - 后端模板导入与恢复脚本支持合并单元格：按合并区域左上角值拆分补齐到每一行后入库
  - 模板内嵌图片会按锚定行提取到本地 `groundwater_registry` 图片目录，并写入对应记录照片字段
  - 导出 Excel 时会把记录照片嵌入“站点照片”列；多张照片从该列开始向右依次放置，图片缺失时回退写名称/地址
  - 最新模板解析结果：`1793` 行、`68` 张图片
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/groundwater_registry.py scripts/reset_groundwater_registry_records.py`
  - 模板解析校验通过：`1793` 行、`68` 张图片
- 本次未新增依赖

### 271. 调休统计增加渠道筛选
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `调休统计（2026年度累计）` 筛选区新增渠道筛选：全部/OA/钉钉
  - 筛选栏改为单行横向展示，避免筛选项换行
  - 后端 `compensatory-leave-stats` 增加 `channel` 参数，按渠道同步过滤调休和加班数据
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 270. 调休统计增加排序并复查孙健高振兴余额
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `调休统计（2026年度累计）` 表头支持渠道、部门、姓名、调休总天数、加班总天数、余额天数升序/降序
  - 样式复用公共 `SortableTh`，与其他 tab 排序样式一致
- 余额复查：
  - 孙健：钉钉实时余额 `6483.7` 天，系统统计应为 `0.5` 天，当前不正确
  - 高振兴：钉钉实时余额 `1620.8` 天，系统统计为 `-0.2` 天，当前不正确；脚本按规则跳过负数余额，不自动修正
- 验证：
  - `cd frontend && ./build.sh`

### 269. 地下水登记新增管理员模板导入
- 调整：
  - `/dengji` 首页新增“导入模板”按钮
  - 仅当前前端登录用户具备 `admin:access`、`*` 或 `super_admin` 时显示
  - 新增后端接口 `POST /api/dengji/template/import`，单独要求 `admin:access` 权限
  - 上传 `.xlsx` 后先校验模板可读和有数据，再替换当前地下水模板文件
  - 导入会清空 `groundwater_registry_records` 并按新模板重建，全部恢复为未填
  - 同步清理本地站点照片目录，避免重置后残留历史图片
  - `backend/docker-compose.yml` 仅挂载 `../data/groundwater_registry` 到 `/app/data/groundwater_registry`，不再影响 `/app/data/rich_uploads`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/groundwater_registry.py`
  - `cd backend && docker compose config`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 268. 修复地下水登记导出模板样式
- 调整：
  - `GET /api/dengji/records/export` 导出改为保留项目内原始 Excel 模板样式和原 sheet 名
  - 不再先删除模板数据行后重写，避免黄色底色、边框、行高等样式丢失
  - 现在只在模板原单元格上覆盖最新数据库值，筛选导出时删除多余尾行
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/groundwater_registry.py`
- 本次未新增依赖

### 267. 新增钉钉调休余额批量修复脚本并修正正式余额
- 调整：
  - 新增 `backend/scripts/sync_dingtalk_compensatory_balance_from_system_stats.py`
  - 脚本按系统 `调休统计` 余额批量修正钉钉 `调休` 假期余额
  - 支持预览和执行：`--apply` 真正写钉钉，默认 dry-run
  - 支持 `--department`、`--keyword`、`--include-zero`、`--max-balance-days`
- 正式环境执行结果：
  - 已更新 `4` 人：张杰 `3.6` 天、李海涛 `6.3` 天、史延明 `4.0` 天、王帅 `0.5` 天
  - 已跳过负数余额和超过 `100` 天的明显脏统计
  - 复查史延明钉钉余额为 `4.0 天 / 32 小时`
  - 调休余额换算口径已更正为 `1 天 = 8 小时`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/sync_dingtalk_compensatory_balance_from_system_stats.py`
- 本次未新增依赖

### 266. 地下水登记台账独立页面
- 调整：
  - 新增公开页面 `/dengji`
  - 新增 `/dengji/{id}/edit`
  - 新增独立接口模块 `/api/dengji`
  - 新增表 `groundwater_registry_records`，覆盖 Excel 台账 40 列字段
  - 列表只展示/筛选 `行政区`、`许可证编号`、`取水权人`，列表列为三项基础信息、`是否已填` 和最后操作列
  - 首页筛选新增 `是否已填`：全部/已填/未填
  - 首页新增 `导出` 按钮，按当前筛选条件基于项目内原始模板导出最新数据库数据
  - 页面不需要公共头部导航；去掉关键字、刷新、新增
  - 编辑页顶部只展示三项基础信息；取水权人下方“其他信息”默认收起，点击小箭头展开历史台账字段
  - 排查登记字段支持填写；下拉字段支持候选选择和直接输入，输入文字无需点确认
  - 编辑页保存按钮固定在底部；经度旁增加一个定位按钮，调用手机浏览器坐标同步回填经纬度，使用原表 `117°21.450′` 这种度+十进制分格式
  - 保存成功会弹出全局提示；从编辑页返回列表会恢复上次筛选条件、页码和每页条数
  - 站点照片支持多张；前端选择图片后立即上传到 `backend/data/uploads/groundwater_registry`
  - 照片右上角有删除按钮；点击后立即删除服务器本地图片并同步记录
  - 备注说明放到最后，单独在照片后面
  - 原始 Excel 已保存到 `data/groundwater_registry/2026年天津市地下水登记造册台账排查汇总表_优化版.xlsx`
  - 新增恢复脚本 `backend/scripts/reset_groundwater_registry_records.py`，可按项目内原始 Excel 清空并重建台账数据，重置为未填并清理本地照片
  - 已按用户要求执行正式环境重置：prod 当前 `1793` 条、已填 `0`，本地站点照片清理 `4` 个
  - 工作台卡片 `groundwater_registry` 已同步到 test/prod，权限为空
- 数据：
  - 已从用户提供 Excel 导入 test/prod 两套环境
  - test/prod 均为 `1793` 条
  - 历史导入数据默认均未填；`is_filled` 仅在编辑页提交保存后变为已填
- 说明：
  - Alembic revision cycle 已修复
  - test/prod 已执行到 `i2j3k4l5m6n7`
  - 两套库 `site_photos` 字段存在，`idx_groundwater_registry_is_filled` 索引存在，当前均 `1793` 条、已填 `0`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/groundwater_registry.py app/models/groundwater_registry.py app/schemas/groundwater_registry.py app/api/api.py app/main.py app/core/config.py migrations/versions/g2h3i4j5k6l7_create_groundwater_registry_records.py migrations/versions/h2i3j4k5l6m7_add_groundwater_registry_fill_status.py migrations/versions/i2j3k4l5m6n7_add_groundwater_registry_site_photos.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
  - `git diff --check`
- 本次未新增依赖

### 265. 企微钉钉考勤同步改为每月 26 日执行
- 调整：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/sql/upsert_wecom_dingtalk_attendance_scheduler_meta.sql`
  - 任务 `wecom_dingtalk_attendance_sync` 从每天 `03:50` 改为每月 `26 日 03:50`
  - 减少 `/attendance/listRecord` 和人员同步链路 `/topapi/v2/department/listsub` 的日常调用
- 数据库：
  - 已直接更新 test/prod 两套库 `scheduler_job_meta.cron_expr='50 3 26 * *'`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 263. 税务助手发票管理增加筛选和审核状态
- 调整：
  - 发票管理新增筛选区，支持 `发票类型`、`审核状态`、`合同编号`、关键字搜索
  - 增加 `搜索`、`重置` 按钮
  - 去掉“仅看刘羽丰当前节点”开关，默认筛选改为 `未审核`
  - 审核状态按 `invoice_no_info` 是否为空计算为 `未审核 / 已审核`
  - 列表与详情均展示审核状态
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py app/services/tax_assistant.py app/models/tax_assistant.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`

### 262. 修复税务助手发票类型与项目管理状态展示
- 调整：
  - 税务助手发票列表/详情接口增加发票类型兜底映射：`0 -> 普通发票`，`1 -> 增值税专用发票`
  - test/prod 历史数据各更新 `5` 条 `invoice_type='0'` 为 `普通发票`
  - 样例 `1174205` 当前 test/prod 均为 `普通发票`
  - 发票项目管理列表、详情、编辑弹窗不再显示状态/启用项，保存时默认启用
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 261. 税务助手新增发票项目管理 tab
- 调整：
  - 前端 `税务助手` 新增 `发票项目管理` tab
  - 列表展示 `tax_oa_project_categories`
  - 操作栏新增 `详情`、`编辑`
  - 详情展示分类税务信息及关联 OA 项目
  - 编辑支持维护服务税收编码、税率、项目名称、备注、启用状态
  - 后端新增分类列表、详情、更新接口
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py app/models/tax_assistant.py app/services/tax_assistant.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 260. 税务助手 OA 分类税务信息表
- 调整：
  - 新增 `tax_oa_project_items`，从 OA `ecology.uf_hwhyslwmc` 同步 OA 项目分类名称、OA 项目名称、税率
  - 新增计划任务 `tax_oa_project_items_sync`，每 10 分钟同步一次
  - 取消 `tax_project_sync_items`
  - 新增 `tax_oa_project_categories`，字段含 OA项目分类名称、服务税收编码、税率、项目名称、是否有效、备注及公共字段
  - `tax_oa_project_items` 新增 `oa_project_category_id`，关联 `tax_oa_project_categories.id`
  - OA 分类为空时归入 `默认`
  - 已同步更新 `doc/database_dictionary.md`
- 执行：
  - test/prod 已直接执行 SQL
  - `tax_oa_project_items` 两套环境均为 `78` 条有效数据，样例 `建筑服务 / 工程款 / 0.0900`
  - `tax_oa_project_categories` 两套环境均为 `13` 条
  - `tax_oa_project_items` 两套环境 `78` 条均已回填分类ID
  - `tax_project_sync_items` 两套环境均已删除
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/tax_assistant.py app/models/__init__.py app/services/tax_assistant.py migrations/versions/ad24ef68b901_create_tax_oa_project_categories.py migrations/versions/ac13de45fa67_create_tax_oa_project_items.py`
- 本次未新增依赖

### 259. 税务助手 OA 发票同步频率调整为 3 分钟
- 调整：
  - `tax_invoice_oa_sync` 从每 10 分钟同步一次改为每 3 分钟同步一次
  - 同步更新 `backend/app/tasks/scheduler.py`
  - 同步更新迁移 SQL 与初始化 SQL 中的任务描述和 `interval_minutes`
- 执行：
  - test/prod 两套环境 `scheduler_job_meta.interval_minutes` 均已更新为 `3`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py migrations/versions/44c00cd6e9b5_add_tax_assistant_invoice_tables.py`
- 本次未新增依赖

### 258. 修复 Docker 构建 gpg 无 tty
- 问题：
  - 构建后端/监听镜像时 Microsoft 源 key 导入报 `gpg: cannot open '/dev/tty'`
- 调整：
  - `backend/Dockerfile` 中 `gpg --dearmor` 改为 `gpg --batch --yes --dearmor`
- 验证：
  - `git diff --check -- backend/Dockerfile`

### 257. 后端镜像增加 ca-certificates
- 调整：
  - `backend/Dockerfile` 基础依赖安装加入系统包 `ca-certificates`
  - 用于补齐容器系统 CA 证书，配合钉钉 Stream worker websocket SSL 校验
- 依赖变更：
  - 新增系统依赖：`ca-certificates`
  - 修改文件：`backend/Dockerfile`
  - 测试/正式环境需要重新 build 后端镜像生效
- 验证：
  - `git diff --check -- backend/Dockerfile`

### 257. 修复调休统计重复计算
- 问题：
  - 调休余额明细里同一天同一时间段同时出现 `调休` 和 `调休（旧）`
  - 原因是调休统计去重 key 带了 `request_id`，导致有单号和无单号的同一条被当成两条
- 调整：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `compensatory-leave-stats` 改为按渠道、用户、请假类型字段、开始/结束时间段、时长去重
  - 重复时优先保留有原因/单号的审批记录
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`

### 256. 修复钉钉 Stream worker SSL 证书校验失败
- 问题：
  - 容器日志显示连接 `wss-open-connection.dingtalk.com` 时 `SSLCertVerificationError: unable to get local issuer certificate`
  - 后续 `Logging error` 是钉钉 SDK 打印异常参数格式问题引起的噪声
- 调整：
  - `backend/scripts/dingtalk_stream_approval_worker.py` 显式给钉钉 SDK 内部 `websockets.connect` 传入 certifi SSLContext
  - 继续设置 `SSL_CERT_FILE` / `REQUESTS_CA_BUNDLE` 为 certifi CA 文件
  - 预留 `DINGTALK_STREAM_SSL_VERIFY=false`，用于现场代理证书异常时临时关闭校验
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/dingtalk_stream_approval_worker.py`
  - 参数级校验确认 websocket 连接包含 `ssl`，`verify_mode=2`
- 本次未新增依赖

### 256. OKCIS 原始记录本地文件日志
- 时间：`2026-08-06 17:54 CST`。
- 用户要求：爬取数据原始记录不要只进数据库运行日志，要落本地日志文件，保留 7 天，并从日志文件里抓符合条件的数据。
- 新增 `backend/app/services/okcis_raw_record_log.py`：按天写 `backend/data/logs/okcis_raw_records/YYYY-MM-DD.jsonl`，每条 OKCIS 原始/enriched 记录一行 JSON，并额外写入 `raw_record_text/detail_content_text/pc_detail_content_text`，便于直接用日志文件排查原始文本。
- OKCIS 爬虫流程调整：每页采集后先 `append_okcis_raw_records` 写文件，再 `load_okcis_raw_records` 从本页日志读回记录，用读回内容继续后续截止时间筛选和业务入库。
- 清理任务：`runtime_log_cleanup` 同步清理 OKCIS 原始记录日志文件，保留最近 7 天。
- 验证：后端 `py_compile` 通过；本地 JSONL 写入/读取测试通过，测试记录已清理。
- 排查：`backend/data/logs/okcis_raw_records/2026-08-06.jsonl` 当前 50 条，未命中用户指定“深能保定西北郊热电厂二期项目#3机组168小时试运行基建转生产交接仪式及视频制作服务”，仅命中“赵县燃气员工深能之家服务平台采购项目”。

### 257. RCC leads 项目列表解密
- 时间：`2026-08-07 09:53 CST`。
- 确认页面 `https://leads.rccchina.com/#/projects/list` 对应接口为 `POST https://leads.api.rccchina.com/api/project/list`。
- 返回 `code=10000`、`message=操作成功`，`data` 需要按前端固定 AES-CTR 解密，key 为 `d6F3Efeqd6F3Efeqd6F3Efeqd6F3Efeq`，IV 为 `1234567890123456`。
- 解密后结构为 `projects` 与 `search_info`；本次请求返回 50 条项目。

### 255. 人事考勤汇总增加表头排序
- 调整：
  - 新增公共排序表头组件 `frontend/src/components/ui/sortable-th.tsx`
  - 样式参考给排水助手列表字段排序，上下箭头分别控制升序/降序
  - 人事考勤汇总支持部门、姓名和所有数值列本地排序
  - 点击当前已选方向可取消排序，恢复默认部门/姓名排序
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 255. 正式环境招投标群发推送排查与调整
- 时间：`2026-08-06 17:46 CST`。
- 原因：售前群发 16:50 日志提示当天数据为空，是因为售前 OKCIS 采集任务 16:15 先失败，错误为自动刷新凭证后仍未登录；普通售后推送当天已推 26 条。
- 正式库处理：手动补跑售前采集后，当天售前数据已进入 `crawler_okcis_notices`；`贵善路(隆华道-兴旺道)市政管网新建工程` 已入库，截止时间 `2026-08-14 14:00:00`。
- `深能保定西北郊热电厂二期项目#3机组168小时试运行基建转生产交接仪式及视频制作服务` 当前正式业务表和 2026-08-06 运行日志均未查到原始记录；为后续排查，OKCIS 采集日志已新增 `[ITEM]` 每条标题、截止时间、详情 URL。
- 截止时间解析增强：支持 `报价截止日期`、`开标时间和地点`、`投标材料递交时间/截止时间`，兼容 `09点30分`、中文冒号 `13：30前`、`下/下午` 等写法。
- 调度调整：OKCIS 爬虫登录失败时按 1/5/10 分钟重试；运行日志保留改 7 天；正式库售后/售前招投标群发推送 cron 已改为 `0 17 * * mon-fri`。
- 验证：后端 `py_compile` 通过；正式库复查两个推送任务均为 17:00；最新售前采集 `run_id=1487` 成功且包含 `[ITEM]` 日志。

### 254. 修复考勤汇总软件研发部筛选与调休/加班详情
- 问题：
  - 测试环境考勤汇总选择 `软件研发部` 后页面显示 `0` 人
  - 实际 test/prod 日汇总表 `2026-06-25 -> 2026-07-23` 均存在 `软件研发部` 数据：`284` 条、`10` 人
  - 原因是前端在部门/姓名筛选后仍用花名册强制过滤人员，把接口返回数据二次过滤掉
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx`
    - 部门/姓名筛选时不再强制按花名册过滤人员
    - 花名册只用于无筛选时补人员和读取应出勤
    - 调休详情兼容 `调休`、`调休（旧）`、`调休(旧)`
    - 加班时长详情展开后端 `details` 真明细，不再展示外层人员汇总行，避免出现 `2000年1月1日`
  - `backend/app/api/endpoints/wecom_attendance.py`
    - 调休统计同样兼容 `调休`、`调休（旧）`、`调休(旧)`，统一算调休
    - 钉钉调休应用到日汇总前先按同一用户、同一时间段去重，优先保留有原因/单号的审批记录
    - 日汇总缓存版本提升到 `9`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 254. 钉钉审批监听日志表
- 调整：
  - 新增 `dingtalk_approval_event_logs`，用于记录钉钉审批监听事件
  - 字段包含姓名、用户ID、事件日期、事件类型、原始时长、调休原始/最新余额、原始事件 JSON
  - 加班审批通过并更新调休余额后，会把余额前后一起落库；请假、调休等仅记事件
  - 已直接执行 test/prod 建表 SQL，确认两套环境表存在，字段数均为 `22`
  - `AGENTS.md` 已补数据库执行规则：新增表/新增字段可直接执行；修改/删除/重命名等需先确认
- 文件：
  - `AGENTS.md`
  - `backend/app/models/wecom_attendance.py`
  - `backend/migrations/versions/ab12cd34ef56_create_dingtalk_approval_event_logs.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `doc/database_dictionary.md`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 253. 钉钉请假/调休/加班审批改为 Stream-only
- 调整：
  - `backend/app/api/endpoints/dingtalk_attendance.py`
    - 手动同步钉钉考勤打卡时，不再轮询钉钉请假/调休/加班审批
    - 返回 `approval_sync_mode=stream` 和跳过原因
  - `backend/app/tasks/scheduler.py`
    - `wecom_dingtalk_attendance_sync` 移除钉钉审批轮询
    - 计划任务只保留 OA 审批同步、企微打卡、钉钉打卡、考勤合并和缺卡重算
  - `backend/scripts/sync_attendance_leave_overtime_current_year.py`
    - 不再拉取钉钉请假/调休/加班审批
    - 仅保留 OA 请假/加班历史同步
- 说明：
  - 钉钉审批入库继续复用 `wecom_attendance_oa_approved_records`
  - 年假、事假、调休、带薪病假等由 Stream worker / 回调接收审批事件后同步本地库
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/dingtalk_attendance.py app/tasks/scheduler.py scripts/sync_attendance_leave_overtime_current_year.py scripts/dingtalk_stream_approval_worker.py app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 251. 人事考勤部门名统一为软件研发部
- 问题：
  - 后续考勤/钉钉/OA 数据中存在 `研发中心-软件开发部`，页面曾临时展示为 `软件开发部`
  - 用户要求统一替换为 `软件研发部`，数据库不再保留旧部门名
- 调整：
  - `backend/app/services/wecom_attendance.py`
    - 新增考勤部门名归一方法
    - 后续企微日考勤、打卡明细、OA请假/考勤修正/加班写库统一写入 `软件研发部`
    - 查询过滤兼容旧口径 `研发中心-软件开发部` / `软件开发部`
  - `backend/app/services/dingtalk_attendance.py`
    - 钉钉人员、请假审批、加班审批、合并日考勤时同步归一部门名
  - `backend/app/api/endpoints/wecom_attendance.py`
    - 考勤汇总缓存版本提升，接口展示使用统一部门名
  - `frontend/src/pages/wecom_attendance/index.tsx`
    - 部门下拉和页面展示统一为 `软件研发部`
  - 新增 `backend/scripts/normalize_attendance_department_names.py`
    - 支持 test/prod 历史考勤相关表部门名清理
- 数据清理：
  - 已执行 test/prod 两套环境
  - test 更新：`wecom_attendance_daily_records=524`、`wecom_attendance_oa_approved_records=192`、`attendance_checkin_records=855`、`dingtalk_attendance_users=9`
  - prod 更新：`wecom_attendance_daily_records=524`、`wecom_attendance_oa_approved_records=183`、`attendance_checkin_records=855`、`dingtalk_attendance_users=9`
  - 复查上述 4 张表中 `研发中心-软件开发部` 和 `软件开发部` 均为 `0`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py scripts/normalize_attendance_department_names.py`
  - 后端相关模块 import 校验通过
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 250. 接入钉钉审批事件回调
- 调整：
  - 新增 `POST /api/dingtalk-attendance/callbacks/approval`
  - 支持钉钉审批事件 `bpms_instance_change`
  - 回调提取 `processInstanceId` 后，后端主动调用 `/topapi/processinstance/get` 获取审批详情
  - 审批详情按现有逻辑解析钉钉请假；新增钉钉加班审批解析
  - 解析后的请假/加班统一写入 `wecom_attendance_oa_approved_records`
  - 写入成功后清理 `wecom:attendance:daily-records:*` 缓存
- 说明：
  - 加班 `duration` 按小时保存，沿用当前钉钉加班统计口径
  - 当前支持明文 JSON 回调；如钉钉后台启用加密回调，需要补充 token/aes_key 解密配置
- 文件：
  - `backend/app/api/endpoints/dingtalk_attendance.py`
  - `backend/app/services/dingtalk_attendance.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/dingtalk_attendance.py`
- 本次未新增依赖

### 251. 接入钉钉审批 Stream 模式
- 调整：
  - 新增常驻 worker：`backend/scripts/dingtalk_stream_approval_worker.py`
  - worker 使用钉钉 Stream SDK 监听 `bpms_instance_change`
  - 收到事件后复用 `handle_approval_event_callback`，通过审批实例 ID 查询详情并同步本地加班/请假数据
  - `backend/docker-compose.yml` 新增服务 `dingtalk-stream-worker`
- 启动：
  - `cd backend && docker compose up -d dingtalk-stream-worker`
  - 本地直接执行：`cd backend && /opt/anaconda3/envs/smart/bin/python scripts/dingtalk_stream_approval_worker.py`
- 依赖：
  - 新增 `dingtalk-stream==0.24.2`
  - 已同步 `backend/requirements.txt` 和 `backend/requirements-win.txt`
- 验证：
  - 本地安装 `dingtalk-stream==0.24.2`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/dingtalk_attendance.py scripts/dingtalk_stream_approval_worker.py`
  - 实际 import `dingtalk_stream` 与 worker 成功
  - `cd backend && docker compose config`

### 252. 调休余额改为加班审批事件触发同步
- 调整：
  - 旧计划任务 `wecom_attendance_compensatory_balance_sync` 不再注册到 APScheduler
  - 后台手动执行该任务会返回停用提示
  - 钉钉审批 Stream 收到加班审批后，保存本地记录并触发调休余额更新
  - 更新前先按本地 `source_key` 判断是否已处理，避免钉钉重复投递导致余额重复增加
  - 首次处理时调用钉钉接口读取该员工当前调休余额
  - 余额更新口径：当前余额小时数 + 本次加班小时数，再按 `8小时=1天` 统一换算成天同步到钉钉
- 两套环境：
  - 已同步 test/prod 的 `scheduler_job_meta`
  - 任务说明改为：`已停用：调休余额改为钉钉加班审批事件触发同步。`
  - `cron_expr` 已置空
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/dingtalk_attendance.py app/api/endpoints/admin/scheduler.py app/tasks/scheduler.py scripts/dingtalk_stream_approval_worker.py`
  - 本地加班审批解析用例通过
- 本次未新增额外依赖

### 249. 修复 OA 加班明细重复
- 问题：
  - 正式库里同一条 OA 加班因两次同步生成了不同 `source_key`，导致加班明细重复展示
- 调整：
  - `backend/app/services/wecom_attendance.py`
    - `source_key` 改为更稳定的主键组合，避免同条记录反复生成新 key
  - `backend/app/api/endpoints/wecom_attendance.py`
    - OA 加班详情增加按请求编号/日期去重
- 清理：
  - 正式库 OA 加班历史重复已清理 `196` 条
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 248. 人事考勤助手部门下拉展示去重
- 问题：
  - 部门下拉同时出现 `软件开发部` 和 `研发中心-软件开发部`
  - 原因是多个考勤接口返回的部门口径不完全一致，部门选项会累加到同一个下拉中
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - 新增部门展示名归一与选项合并方法
  - 下拉部门选项统一按展示名去重，`研发中心-软件开发部` 展示为 `软件开发部`
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 247. 修复考勤汇总加班时长详情无明细
- 问题：
  - 考勤汇总点击“加班时长详情”时，前端会用每日汇总 `overtime_duration` 生成兜底详情，导致出现 `2000年1月1日`、内容为 `-` 的错误记录
  - 加班明细部门可能为 `研发中心-软件开发部`，汇总人员部门为 `软件开发部`，精确匹配导致明细匹配不到
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx`
    - 加班详情只展示真实加班记录接口返回的明细
    - 人员与明细匹配时，部门支持包含/后缀匹配
  - `backend/app/api/endpoints/wecom_attendance.py`
    - 加班记录接口输出部门名同步走展示归一
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 246. 人事考勤助手调休统计部门显示归一
- 调整：
  - 调休统计接口输出中，将 `研发中心-软件开发部` 统一显示为 `软件开发部`
  - 明细里的部门展示同步归一
  - 不修改原始入库数据
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 245. 招标信息公示历史导出避开 16:00-17:00 执行
- 调整：
  - 历史数据导出任务如果在 16:00-17:00 之间提交，保持 `pending`
  - 后台等待到 17:00 后，并追加 1-300 秒随机延迟再进入 `running`
  - 等待期间 `error_message` 写入预计执行时间
  - 其他时间提交仍按原逻辑立即后台执行
- 文件：
  - `backend/app/api/endpoints/okcis_notices.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/okcis_notices.py`
  - 本地边界检查：15:59:59/17:00:00 不延迟，16:00-16:59:59 延迟到 17:00 后
- 本次未新增依赖

### 244. 钉钉请假审批实例补采入库
- 调整：
  - `backend/app/services/dingtalk_attendance.py`
    - 新增钉钉请假审批实例采集
    - 通过 `process/listbyuserid` 找到 `请假` 流程 `process_code`
    - 再用 `processinstance/listids` + `processinstance/get` 拉取审批详情
    - 解析 `DDHolidayField`，补成本地请假记录
    - `2026-07-24` 之前的钉钉调休自动落为 `调休（旧）`
  - `backend/app/services/dingtalk_attendance.py`
    - `sync_leave_status_records_to_local` 现在会同时同步 `getleavestatus` 和审批实例
- 正式环境结果：
  - 于涛 `202607231039000142527` 已补采入库
  - `2026-07-21 -> 2026-07-23` 重新同步共写入 `8` 条
  - 于涛这条落库为 `调休（旧）`、`1.0` 天
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/tasks/scheduler.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 243. 排查于涛 2026-07-23 调休审批未同步
- 正式环境排查：
  - 审批编号：`202607231039000142527`
  - 审批内容：于涛，研发中心-软件开发部，请假类型 `调休`，请假日期 `2026-07-21`，时长 `1` 天，审批通过时间 `2026-07-23 10:39`
  - 本地表 `wecom_attendance_oa_approved_records` 按于涛 7 月、审批编号均查不到记录
  - 当前同步调用 `/topapi/attendance/getleavestatus`
  - 按于涛钉钉 userId `1603103307651469` 查询 `2026-07-21 -> 2026-07-23`，钉钉返回 `leave_status=[]`
  - 尝试调用 `/topapi/processinstance/get` 查询审批编号，钉钉返回应用缺权限 `qyapi_aflow`
- 结论：
  - 该记录存在于钉钉审批，但未进入考勤请假状态接口
  - 后续如要同步这类记录，需要给钉钉应用开通 `qyapi_aflow`，再补充审批实例采集逻辑

### 242. 线上办公助手分享框、孤立用户、CSP 与历史列表交互
- 调整：
  - 分享框继续缩小，字号、头像、输入框和按钮均压小
  - OnlyOffice 编辑器页单独设置 CSP，允许 `unsafe-eval`，用于兼容 OnlyOffice 脚本
  - 历史文件列表：右侧铅笔图标打开文档，点击标题重命名
  - 孤立用户 `ooRaE0q9XIdiwR4mPAUhu2wByoYs` 在 test/prod 均已软删除
- 排查：
  - 该用户两套环境均为 `id=92`，无 OA 资源、无部门、无角色
- 文件：
  - `backend/app/api/endpoints/onlyoffice.py`
  - `frontend/src/pages/online_office/index.tsx`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 241. 线上办公助手分享按钮与无权限提示调整
- 调整：
  - 分享按钮改为纯 SVG 图标
  - 分享弹层缩小字号与尺寸，并固定在分享按钮下方
  - 分享短链无权限时统一提示 `文档暂无权限，请联系作者开通`
- 文件：
  - `backend/app/api/endpoints/onlyoffice.py`
  - `frontend/src/App.tsx`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 240. 钉钉调休余额同步改为批量调用并重跑正式环境
- 调整：
  - `backend/app/services/dingtalk_attendance.py`
    - 新增批量查询调休余额 `query_compensatory_leave_balances`
    - 新增批量修正调休余额 `correct_compensatory_leave_balances`
    - 单人余额查询内部复用批量查询
  - `backend/app/tasks/scheduler.py`
    - `wecom_attendance_compensatory_balance_sync` 改为先收集全部待同步人员，再批量调用钉钉余额接口，避免按人逐个频繁请求钉钉
- 正式环境执行：
  - 已手动执行 `wecom_attendance_compensatory_balance_sync`
  - 范围：`2026-01-01 -> 2026-07-22`
  - 结果：同步 `26` 人，跳过 `29` 人，任务日志状态 `success`
  - 已补同步正式环境 `2026-07-23` 当天钉钉数据：请假/调休 `2` 条，加班 `0` 条
  - 随后按 `2026-01-01 -> 2026-07-23` 重新批量同步调休余额：同步 `26` 人，跳过 `29` 人
  - 正式环境钉钉请假/调休本地同步已按 30 天拆分重跑，范围 `2026-01-01 -> 2026-07-23`，累计同步 `39` 条
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/tasks/scheduler.py app/api/endpoints/wecom_attendance.py scripts/debug_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 239. 钉钉调休假期规则与统计切换口径
- 外部配置：
  - 正式环境钉钉假期规则 `bc88d408-8cf3-4fda-94fc-266f25cf7cc5`
  - 名称已从 `调休（新）` 改为 `调休`
  - 适用范围已从研发中心改为根部门 `dept=1`
  - 新版接口正确调用方式：`PUT https://api.dingtalk.com/v1.0/attendance/leaves/types?opUserId=026960310339-366502766`
- 系统口径：
  - 钉钉调休余额查询优先使用新 `调休`
  - 兜底匹配调休规则时排除 `调休（旧）/调休(旧)`
  - 人事助手“调休统计”：
    - `2026-07-23` 及以前的钉钉请假只统计旧 `调休（旧）/调休(旧)`
    - `2026-07-24` 起的钉钉请假只统计新 `调休`
    - OA 调休继续统计 `调休`
  - 钉钉请假同步到本地表不删除旧记录，历史旧调休数据只追加/更新，不做减量清理
  - 钉钉余额同步仍按系统统计余额写入
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/scripts/debug_dingtalk_compensatory_balance.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py scripts/debug_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 238. 线上办公助手分享弹层改为链接/二维码分享样式
- 调整：
  - 分享弹层改为左侧协作成员、右侧人员/部门筛选布局
  - 左侧显示当前文档已选协作成员
  - 人员列表支持搜索、部门筛选、添加当前筛选结果
  - 点击链接按钮弹出权限选择，默认只读，可切换可编辑；确认后保存分享、复制短链并关闭弹层
  - 点击二维码按钮弹出权限选择，确认后保存分享并展示二维码弹窗
- 文件：
  - `backend/app/api/endpoints/onlyoffice.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py app/main.py`
- 本次未新增依赖

### 237. 线上办公助手历史保存、分享收口和回收站
- 调整：
  - 自动保存回调和手动文件名保存都会把在线文档标记为已保存，进入历史文件列表
  - 分享保存生成短链后，弹层底部按钮改为关闭态，不再继续显示“取消/保存分享”
  - 删除在线文档改为软删除进入回收站
  - 新增回收站列表与恢复能力
  - 回收站文件超过 30 天自动物理清理
- 文件：
  - `backend/app/services/onlyoffice.py`
  - `backend/app/api/endpoints/onlyoffice.py`
  - `frontend/src/pages/online_office/index.tsx`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/onlyoffice.py app/api/endpoints/onlyoffice.py app/main.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 236. 计划任务时间变更后同步 APScheduler
- 调整：
  - 计划任务管理编辑 cron/interval 时间保存后，统一调用 `apply_scheduler_meta_overrides({job_id})`
  - 确保数据库 `scheduler_job_meta` 更新后，运行中的 APScheduler 立即同步最新 trigger
  - 爬虫计划任务仍通过 `sync_single_crawler_scheduler_job` 同步
- 文件：
  - `backend/app/api/endpoints/admin/scheduler.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 235. 人事助手调休统计余额同步加入计划任务
- 调整：
  - 新增计划任务 `wecom_attendance_compensatory_balance_sync`
  - 每天 `05:20` 执行
  - 口径与人事助手“调休统计”一致：按当前年度 1 月 1 日到昨日统计余额
  - 按企微 `userid` 映射钉钉用户后，自动修正钉钉调休余额
  - 零余额和负余额人员跳过
  - 测试环境默认暂停，正式环境正常执行
  - 任务中心已可手动执行
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 234. 采购部供应商主表新增企业信用编码并回填
- 调整：
  - 新增 `procurement_suppliers.credit_code`
  - 供应商注册成功后同步写入采购部供应商主表企业信用编码
  - 供应商风险/工商信息刷新时，从 `basic_info` 提取“统一社会信用代码”等字段写入主表
  - 供应商管理列表在“供应商名称”后新增“企业信用编码”
  - 列表关键词查询支持按企业信用编码匹配
  - 数据库字典已同步更新
- 迁移：
  - `backend/migrations/versions/u1v2w3x4y5z6_add_procurement_supplier_credit_code.py`
  - 历史回填来源：
    - `suppliers.tax_no` 按供应商名称匹配
    - `procurement_suppliers.risk_basic_info` 中的“统一社会信用代码”
- 执行结果：
  - test/prod 均已执行 `alembic upgrade head`
  - test：`635/2220` 条有企业信用编码
  - prod：`720/2220` 条有企业信用编码
  - 北京中天金维科技有限公司两套环境均已回填：`91110109MA01G92P71`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/procurement_dept.py app/api/endpoints/auth.py app/api/endpoints/procurement_dept/procurement_dept.py app/schemas/procurement_dept.py migrations/versions/u1v2w3x4y5z6_add_procurement_supplier_credit_code.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 233. 采购部供应商企业信用代码空缓存修复
- 问题：
  - 供应商管理中 `北京中天金维科技有限公司` 没有显示企业信用代码
  - U8 `Vendor.cVenRegCode` 为空
  - 本地 `procurement_suppliers.risk_basic_info` 曾缓存为空对象 `{}`，且有 `risk_updated_at`
  - 原逻辑看到缓存即返回，导致后续不会重新调用企业查询服务
- 排查结果：
  - 企业查询服务可查到该公司
  - 统一社会信用代码：`91110109MA01G92P71`
- 调整：
  - 文件：`backend/app/api/endpoints/procurement_dept/procurement_dept.py`
  - 空工商/风险缓存不再阻止刷新
  - 已补齐 test/prod 两套环境该供应商风险工商缓存
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/procurement_dept/procurement_dept.py`
- 本次未新增依赖

### 232. OA 开票申请流程节点读取
- 排查对象：
  - `workflowid=390`
  - 示例 `requestid=1173190`
- 节点定义表：
  - `workflow_flownode`：流程与节点关系、节点顺序
  - `workflow_nodebase`：节点名称、开始/结束标记、坐标
  - `workflow_nodelink`：节点连线
- 已确认开票申请节点：
  - `2515`：`1.发起人`
  - `2516`：`2.直接上级`
  - `2520`：`3.部门总监`
  - `2522`：`4.分管副总`
  - `2517`：`5.财务经理`
  - `2518`：`6.财务会计`
  - `2521`：`7.抄送`
  - `2519`：`8.归档`
- 示例 `requestid=1173190` 实际流转已补入 `doc/oa-workflow-database-map.md`
- 本次仅更新文档，未修改业务代码，未新增依赖

### 231. OA 流程数据库表定位说明文档落地
- 新增文档：
  - `doc/oa-workflow-database-map.md`
- 内容：
  - 整理 OA `ecology` 数据库统计：总表数 `4186`、表单主/明细表 `338` 张、表单字段 `4880` 个
  - 说明通用定位链路：`requestid -> workflow_requestbase.workflowid -> workflow_base.formid -> workflow_bill.tablename -> formtable_main_xxx`
  - 补充字段中文名查询方式：`workflow_billfield` + `htmllabelinfo`
  - 记录开票申请 `requestid=1173190` 示例：主表 `formtable_main_33`、明细表 `formtable_main_33_dt1`
  - 记录图片/附件查找口径：主表字段 `tp`、`tsyqfj`，审批日志字段 `ANNEXDOCIDS`、`SIGNDOCIDS`，后续再查 `docdetail`、`docimagefile`、`imagefile`
- 本次仅新增文档，未修改业务代码，未新增依赖

### 230. OA 开票申请页面 requestid=1173190 数据表定位
- 排查对象：
  - OA 页面地址包含 `requestid=1173190`
  - 页面标题为 `开票申请`
- 定位结果：
  - `workflow_requestbase`：流程请求主索引，`requestid=1173190`、`workflowid=390`
  - `workflow_base` + `workflow_bill`：流程到表单表映射，`formid=-33`、`tablename=formtable_main_33`
  - `formtable_main_33`：开票申请主表，本条 `id=14464`
  - `formtable_main_33_dt1`：开票申请明细表，通过 `mainid=14464` 关联
  - `workflow_requestlog`：审批流转日志，通过 `REQUESTID=1173190` 查询
  - `hrmresource`、`hrmdepartment`：人员和部门名称显示
- 图片/附件结论：
  - `formtable_main_33.tp` 字段标签为“图片”
  - `formtable_main_33.tsyqfj` 字段标签为“特殊要求附件”
  - 当前 `requestid=1173190` 这两个字段均为空
  - `workflow_requestlog.ANNEXDOCIDS` / `SIGNDOCIDS` 当前也为空
  - 若后续有附件/图片 ID，再继续查 `docdetail`、`docimagefile`、底层 `imagefile`
- 本次仅排查数据库，未修改业务代码，未新增依赖

### 229. 计划任务钉钉接口临时超时增加重试
- 问题：
  - `wecom_dingtalk_attendance_sync` 在 `2026-07-23 03:51:36` 执行失败
  - 失败点为钉钉 `/topapi/attendance/getcolumnval`
  - 根因是 `httpx.ConnectTimeout` 临时连接超时
- 调整：
  - 文件：`backend/app/tasks/scheduler.py`
  - 在计划任务层新增钉钉临时错误重试
  - 覆盖同步钉钉考勤人员、钉钉请假/调休、钉钉加班、钉钉考勤打卡
  - 最多重试 `5` 次，等待时间递增，最终仍失败才记录任务失败
  - 只重试连接失败、超时、限流/QPS 等临时错误；业务错误不吞掉
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 228. 调休统计改为年度累计
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 调休统计和加班统计统一按当前年度累计
  - 当前年度从 `2026-01-01` 到当前日期结束
  - 调休统计 tab 移除日期筛选控件，保留部门、姓名筛选
  - 页面不再因其他 tab 的日期筛选变化而重新按短周期查询
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 227. 修正钉钉加班时长单位与重复记录
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/scripts/sync_attendance_leave_overtime_current_year.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 钉钉接口“加班总时长”单位为天，入库统一转换为小时：`1 天 = 8 小时`
  - 钉钉加班同步前按日期范围清理旧数据，避免时长变化造成重复
  - 去重键移除时长字段
  - 钉钉接口连接失败时同步脚本自动重试
- 执行结果：
  - test/prod 均完成 2026-01-01 至 2026-07-22 回填
  - OA请假/调休 1330，OA加班 196，钉钉请假/调休 167，钉钉加班 312
  - 抽查徐皓 2026-07-21：两套环境均为 1.0 小时，仅一条记录
- 验证：
  - 相关 Python 文件 `py_compile` 通过

### 226. 加班明细恢复原始小时展示
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 加班统计汇总继续显示折算后的天数
  - 明细表新增“原始时长（小时）”列
  - 开始/结束时间展示到秒
  - 钉钉“加班总时长”源字段单位为天，按 `0.1 天 = 1 小时` 换算为原始小时；OA 记录仍按原小时展示
  - 已核对钉钉接口示例：徐皓 2026-07-21 返回 `value=0.13`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 225. 今年以来调休/加班数据完成双环境同步
- 文件：
  - `backend/scripts/sync_attendance_leave_overtime_current_year.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 脚本 `--both` 依次读取 `env_test` / `env_prod` 并同步两套环境
  - 钉钉接口 QPS 限流时自动等待重试
  - test/prod 之间增加错峰等待，避免连续调用撞限流
- 执行：
  - `/opt/anaconda3/envs/smart/bin/python backend/scripts/sync_attendance_leave_overtime_current_year.py --both`
- 结果：
  - test：2026-01-01 至 2026-07-22，OA请假/调休 1328，OA加班 196，钉钉请假/调休 166，钉钉加班 312
  - prod：2026-01-01 至 2026-07-22，OA请假/调休 1328，OA加班 196，钉钉请假/调休 166，钉钉加班 312
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/sync_attendance_leave_overtime_current_year.py`
- 本次未新增依赖

### 224. 新增钉钉调休余额对账脚本
- 文件：
  - `backend/scripts/debug_dingtalk_compensatory_balance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 默认按本月范围输出钉钉加班天数、调休请假天数和余额差值
  - 支持按日期、部门、关键字筛选
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/debug_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 223. 软件部任务工具新增独立自动提交工作记录调度
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/app/api/endpoints/software_task/software_task.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `software_work_record_auto_submit`
  - 每分钟扫描一次到点工作记录草稿并自动提交
  - 后台计划任务列表支持手动执行
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py app/api/endpoints/software_task/software_task.py`
- 本次未新增依赖

### 222. 人事考勤助手缺卡提醒文案统一
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 将“缺卡考勤”统一改为“缺卡提醒”
  - 导出 sheet 名和文件名同步修改
  - 考勤配置保存/删除后会清理 `daily-records` 缓存，避免保存后刷新回旧值
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 221. 软件部日报提醒排除全天无实际打卡人员
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 软件部日报提醒检查昨天未提交日报名单时，先查询 `attendance_checkin_records`
  - 若人员昨天没有任何 `checkin_time IS NOT NULL` 实际打卡记录，则视为可能请假或旷工，不加入未提交日报推送名单
  - 只有昨天有实际打卡但未提交日报的人员才推送提醒
  - 日志会输出被跳过的全天无实际打卡人员
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
- 本次未新增依赖

### 220. 编辑业务团队管理看板只展示客户跟进爬取人员
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `/management-dashboard/members` 编辑接口改为使用客户无忧跟进爬取人员列表
  - 当前编辑弹窗只展示 10 个人
  - 其他看板人员不在编辑弹窗展示、不参与编辑保存
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 219. OKCIS 详情抓取请求头带列表页来源
- 文件：
  - `backend/app/services/crawler_handlers/okcis.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `OKCIS_LIST_PAGE_REFERER_URL=https://www.okcis.cn/suppliers/`
  - 移动端 JSON 详情抓取统一带列表页 `Referer`
  - PC 详情兜底抓取统一带列表页 `Referer`
  - 补充 `Sec-Fetch-Dest`、`Sec-Fetch-Mode`、`Sec-Fetch-Site`
  - 覆盖列表预览阶段详情抓取与入库补抓详情缓存阶段
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 218. 业务团队管理看板待签约接入金山文档当前季度数据
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/scripts/sync_kdocs_pending_sign_to_dashboard.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 编辑业务团队管理看板 `待签约(万)` 接入 `kdocs-browser` 抓取服务
  - 根据当前季度选择列名：一季度/二季度/三季度/四季度待签约
  - 当前日期 `2026-07-21` 对应第三季度，抓取 `三季度待签约`
  - 仅当前年度触发同步；其他年份不抓取、不更新
  - 待签约值按“当前季度 + 下一季度”两列求和后写回 `after_sales_management_dashboard_members.pending_sign_amount`
  - 已挂到 `after_sales_business_dashboard_push` 任务前，推送前先同步当前年度当前季度待签约
  - 新增一次性同步验证脚本 `scripts/sync_kdocs_pending_sign_to_dashboard.py`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py scripts/sync_kdocs_pending_sign_to_dashboard.py`
  - `node --check backend/scripts/kdocs_browser_server.js`
  - `docker compose -f backend/docker-compose.yml config`
- 正式环境验证命令：
  - `cd backend && docker compose exec api /anaconda3/envs/smart/bin/python scripts/sync_kdocs_pending_sign_to_dashboard.py`
- 本次未新增项目安装包依赖；浏览器容器镜像内安装 `playwright@1.61.1`

### 217. 后端 compose 新增金山文档 Playwright 浏览器服务
- 文件：
  - `backend/docker-compose.yml`
  - `backend/scripts/kdocs_browser_test.js`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `kdocs-browser` 服务
  - 新增 `Dockerfile.kdocs-browser`
  - 基础镜像使用 `mcr.microsoft.com/playwright:v1.61.1-jammy`
  - 运行镜像为 `zhidao-kdocs-browser:latest`
  - 镜像内安装 `playwright@1.61.1`，避免官方镜像在 `/app` 挂载代码后 `require('playwright')` 失败
  - 容器名 `zhidao-kdocs-browser`
  - 显式加入 `smart-backend` 网络
  - 网络别名：`kdocs-browser`、`kdocs-browser-service`
  - 现有 `api` 容器可通过 `http://kdocs-browser:3010` 调用浏览器抓取服务
  - 新增服务脚本 `scripts/kdocs_browser_server.js`
  - 新增测试脚本 `scripts/kdocs_browser_test.js`
  - 默认抓取 `润达全年业绩分析表` 中 `三季度待签约` 列
- 验证：
  - `node --check backend/scripts/kdocs_browser_server.js && node --check backend/scripts/kdocs_browser_test.js`
  - `docker compose -f backend/docker-compose.yml config`
- 本次未新增项目安装包依赖；新增运行时 Docker 镜像依赖

### 216. 线上办公助手改为独立 OnlyOffice 编辑器页
- 文件：
  - `backend/app/core/config.py`
  - `backend/app/services/onlyoffice.py`
  - `backend/app/api/endpoints/onlyoffice.py`
  - `backend/config/env_test`
  - `backend/config/env_prod`
  - `frontend/src/pages/online_office/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `/api/onlyoffice/editor?token=...` 独立编辑器 HTML 页
  - 中台 `/apps/online-office` 只做鉴权和跳转，不再内嵌 OnlyOffice
  - test 入口：`http://172.18.6.140:5173`
  - test 文档下载/保存回调地址：`http://172.18.6.140:5173/api`
  - prod 入口：`https://zhidao.tjchentian.com:9091`
  - 新增 `ONLYOFFICE_ENTRY_BASE_URL`、`ONLYOFFICE_CALLBACK_BASE_URL`
  - `document/callback` 地址固定走后端 API；测试环境走 Vite `/api` 代理，避免 OnlyOffice DocumentServer 拉不到 `127.0.0.1`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/onlyoffice.py app/api/endpoints/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && npm run build`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && ./build.sh`
- 本次未新增依赖，前端使用 Node `v20.20.2`

### 217. 修复测试环境 OnlyOffice 下载失败
- 原因：
  - 运行中的后端进程未重新加载最新 `backend/.env`
  - 编辑器页实际仍生成公网 `document.url=https://zhidao.tjchentian.com:9091/api/onlyoffice/document`
- 处理：
  - 已重启本地后端
  - test 当前实际生成：
    - `document.url=http://172.18.6.140:5173/api/onlyoffice/document`
    - `callbackUrl=http://172.18.6.140:5173/api/onlyoffice/callback`
- 验证：
  - `GET http://172.18.6.140:5173/api/onlyoffice/document` 返回 `200`
  - 调用 OnlyOffice `ConvertService.ashx` 转换该 docx，返回 `EndConvert=True`
- 结论：
  - 重新从卡片打开即可，旧页面/旧 token 仍会保留旧配置

### 218. 线上办公助手补文件首页与独立内网回调配置
- 文件：
  - `backend/app/services/onlyoffice.py`
  - `backend/app/api/endpoints/onlyoffice.py`
  - `backend/app/core/config.py`
  - `backend/config/env_test`
  - `backend/config/env_prod`
  - `frontend/src/pages/online_office/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `/apps/online-office` 改为先进入文件首页
  - 支持新建 `docx/xlsx/pptx`
  - 支持显示历史文件列表并打开指定文件
  - 保存开启 `customization.forcesave=true`，点击保存可触发 `status=6` 回调
  - 新增 `ONLYOFFICE_INTERNAL_CALLBACK_HOST`
  - test：`ONLYOFFICE_INTERNAL_CALLBACK_HOST=172.18.6.140`
  - prod：继续使用 `ONLYOFFICE_CALLBACK_BASE_URL=https://zhidao.tjchentian.com:9091/api`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/onlyoffice.py app/api/endpoints/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && npm run build`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && ./build.sh`
  - test 派生地址确认为：`http://172.18.6.140:5173/api/onlyoffice/callback/{file_id}`
- 本次未新增依赖

### 219. 线上办公助手补历史文件改名删除与编辑器页文件名输入框
- 文件：
  - `backend/app/api/endpoints/onlyoffice.py`
  - `backend/app/services/onlyoffice.py`
  - `frontend/src/pages/online_office/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 历史文件列表支持改名、删除
  - 编辑器页顶部新增文件名输入框
  - 点击“保存文件名”后调用 `/api/onlyoffice/editor/rename`
  - 编辑器页改名接口使用编辑器 token 校验，不依赖中台 Authorization header
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && npm run build`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use && ./build.sh`
  - 编辑器 HTML 已包含 `smart-office-title` 和 `/api/onlyoffice/editor/rename`
- 本次未新增依赖

### 220. 线上办公助手多人共同编辑说明落地
- 结论：
  - 同一个历史文件支持多人共同编辑
  - 同一个 `file_id` 会生成同一个 OnlyOffice `document.key`
  - 多人打开同一历史文件时进入同一协同编辑会话
  - 各自点击“新建”会生成不同文件，不会共同编辑
  - 保存落盘依赖 OnlyOffice 回调，当前已开启 `forcesave=true`

### 215. 修复线上办公助手卡片点击入口
- 文件：
  - `backend/migrations/versions/s9t0u1v2w3x4_seed_online_office_card.py`
  - `backend/app/services/onlyoffice.py`
  - `backend/app/api/endpoints/onlyoffice.py`
  - `frontend/src/pages/online_office/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - test / prod 两套库 `dashboard_app_cards.online_office.mobile_path` 已补为 `/apps/online-office`
  - 迁移脚本同步写入 `mobile_path`
  - OnlyOffice 页面改为通过 Docs API 初始化编辑器，不再只是 iframe 打开 welcome
  - 编辑器配置里传入当前企微登录用户 `id/name`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/onlyoffice.py app/api/endpoints/onlyoffice.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 214. 接入 OnlyOffice 线上办公助手入口
- 文件：
  - `backend/app/core/config.py`
  - `backend/app/permissions.py`
  - `backend/app/services/onlyoffice.py`
  - `backend/app/api/endpoints/onlyoffice.py`
  - `backend/app/api/api.py`
  - `backend/migrations/versions/s9t0u1v2w3x4_seed_online_office_card.py`
  - `backend/migrations/versions/t0u1v2w3x4y5_grant_online_office_to_standard_user.py`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/online_office/index.tsx`
  - `frontend/src/App.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 OnlyOffice 配置读取：`ONLYOFFICE_ENABLED`、`ONLYOFFICE_BASE_URL`、`ONLYOFFICE_JWT_SECRET`
  - 新增 `/api/onlyoffice/entry`，基于当前登录用户生成企微用户身份与 HS256 JWT
  - 新增前端 `/apps/online-office` 页面，提供内嵌 OnlyOffice 与新窗口打开
  - 新增权限 `app:online_office:access`
  - 新增工作台卡片 `online_office`
  - 默认授权给 `standard_user`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/permissions.py app/services/onlyoffice.py app/api/endpoints/onlyoffice.py app/api/api.py migrations/versions/s9t0u1v2w3x4_seed_online_office_card.py migrations/versions/t0u1v2w3x4y5_grant_online_office_to_standard_user.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/alembic upgrade head`
  - `cd frontend && ./build.sh`
  - test / prod 均确认：`permission=1, card=1, standard_user_grant=1`
- 备注：
  - prod 原 `alembic_version` 存在 `b1c2d3e4f5a6`、`d0e1f2a3b4c5` 两条旧记录，导致 Alembic 升级冲突
  - 已将 prod `alembic_version` 对齐为 `t0u1v2w3x4y5`
  - 当前 test / prod 均确认 `alembic current` 为 `t0u1v2w3x4y5 (head)`
- 本次未新增依赖

### 213. OnlyOffice 线上办公助手环境配置
- 文件：
  - `backend/config/env_test`
  - `backend/config/env_prod`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - test / prod 两套环境均新增 `ONLYOFFICE_ENABLED`
  - test / prod 两套环境均新增 `ONLYOFFICE_BASE_URL`
  - test / prod 两套环境均新增 `ONLYOFFICE_JWT_SECRET`
  - OnlyOffice 服务地址为 `http://172.18.2.31:8088`
- 验证：
  - `rg -n "ONLYOFFICE_" backend/config/env_test backend/config/env_prod`
- 本次未新增依赖，未改业务代码

### 212. 考勤汇总旷工与未打卡规则、详情与姓名筛选统一
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 同一天双缺卡直接转为 `旷工 1 天`
  - 若该日期已计旷工，则未打卡不再重复计数、不再出现在未打卡详情
  - 未打卡汇总、旷工汇总、详情弹层口径统一
  - 姓名筛选时不再补整张花名册空行，避免搜索结果失真
  - 导出继续按花名册人员范围，但每个人的数值口径与页面汇总一致
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/services/wecom_attendance.py backend/app/api/endpoints/wecom_attendance.py`
  - 已执行本地重算：`2026-06-25` 至 `2026-07-20`
  - 重算结果：`updated_daily_count=7161`、`absenteeism_count=510.0`、`missing_punch_count=416`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 209. 考勤汇总排除部门加载增加本地兜底
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `_load_attendance_scope_exclusions` 现在优先使用企业微信部门树
  - 当企业微信部门接口连不上时，自动回退到本地 `departments` 表补全排除部门及子部门
  - 避免本地 `/api/wecom-attendance/daily-records` 直接报 500
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/wecom_attendance.py`
  - 本地 `/api/wecom-attendance/daily-records?start_date=2026-06-25&end_date=2026-07-20` 返回 `200`
- 本次未新增依赖

### 210. 修复考勤汇总按部门筛选时仍补全整张花名册
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 当考勤汇总选择具体部门时，不再使用整张花名册补全空白人员
  - 避免研发中心筛选后仍把安全部、北京分公司等其他部门人员补进列表
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 211. 修复考勤汇总部门筛选后仍混入其他部门 OA 补算数据
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `apply_local_oa_attendance_adjustments` 新增 `allowed_userids` 限制
  - `daily-records` 在部门筛选或单人筛选时，仅允许当前命中的 userid 参与 OA 补算补行
  - `daily-records` 缓存版本升级到 `v5`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/wecom_attendance.py backend/app/services/wecom_attendance.py`
  - 清理缓存后，本地 `department_name=研发中心` 返回 `878` 条，仅含 `研发中心/软件开发部/自控部/给排水部/测试部`
- 本次未新增依赖

### 208. 考勤汇总部门筛选改为先取公共 userid 集合再查数据
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `wecom_attendance_service.list_department_userids` 现按部门自动分流
  - `研发中心` 及其子部门走钉钉 `list_department_user_ids`
  - 其他部门走企微 `list_department_userids`
  - `考勤汇总 /daily-records` 改为先取 `userid` 集合，再按 `userid IN (...)` 查询
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py app/services/dingtalk_attendance.py`
  - `研发中心` 当前返回 `34` 人，`软件开发部` 当前返回 `9` 人
- 本次未新增依赖

### 207. 新增企微按部门查询全部用户 userid 公共方法
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `list_department_userids`
  - 支持按部门名查询，默认包含子部门
  - 支持传入排除 `userid` 集合
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py`
  - 实际调用 `department_name='研发中心'` 返回 `40` 个 `userid`
- 本次未新增依赖

### 206. 新增钉钉研发中心员工 userId 公共查询方法
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `list_department_user_ids`
  - 默认查询 `研发中心` 及其子部门全部员工 `userId`
  - 默认过滤钉钉配置排除人员
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
  - 实际调用返回 `34` 个 `userId`
- 本次未新增依赖

### 203. 修复钉钉调休在考勤汇总里重复叠加与跨天错算
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `apply_local_oa_attendance_adjustments` 现仅处理 `channel=OA` 的本地镜像记录，避免钉钉请假先被本地镜像补算、又被 `_apply_dingtalk_leave_adjustments` 再补一次
  - 钉钉请假补算改为按 `开始日期/结束日期/时长/起止时间` 分摊到每天，避免整段调休时长整笔挂到开始日
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 202. 修复考勤汇总命中旧缓存导致钉钉休假不显示
- 文件：
  - `backend/app/services/redis_cache.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/api/endpoints/dingtalk_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `daily-records` 缓存版本升级到 `v2`
  - 企业微信/钉钉同步完成后清理 `wecom:attendance:daily-records:*` 旧缓存
  - 避免考勤汇总继续命中旧结果
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/redis_cache.py app/api/endpoints/wecom_attendance.py app/api/endpoints/dingtalk_attendance.py`
- 本次未新增依赖

### 201. 正式环境补同步钉钉休假数据
- 文件：
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 在正式环境执行钉钉休假本地镜像补同步，时间范围为 `2026-06-01` 至 `2026-07-21`
  - 写入/刷新 `wecom_attendance_oa_approved_records` 共 `48` 条钉钉请假记录
- 备注：
  - 脚本结束时有 `Event loop is closed` 的 aiomysql 连接回收告警，但 SQL 已成功提交

### 200. 钉钉请假与加班查询改为本地镜像优先
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/api/endpoints/dingtalk_attendance.py`
  - `backend/app/tasks/scheduler.py`
  - `.gitignore`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增本地镜像保存复用方法 `save_local_approved_attendance_records`
  - `企微&钉钉考勤同步` 任务现会同步钉钉请假、钉钉加班到 `wecom_attendance_oa_approved_records`
  - `人事考勤助手 -> 已通过记录` 改为只读本地镜像，不再实时请求钉钉
  - 日汇总中的钉钉请假、钉钉加班补算改为读取本地镜像
  - 手动接口 `/api/dingtalk-attendance/sync-records` 现同步打卡同时也会刷新钉钉请假/加班镜像
  - 新增运行日志目录 `logs/` 到忽略列表
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py app/api/endpoints/dingtalk_attendance.py app/tasks/scheduler.py`
- 本次未新增依赖

### 199. 招标信息公示历史数据导出增加预览内容列
- 文件：
  - `backend/app/services/okcis_history_export.py`
  - `backend/app/api/endpoints/okcis_notices.py`
  - `backend/migrations/versions/r8s9t0u1v2w3_add_preview_content_to_okcis_history_exports.py`
  - `doc/database_dictionary.md`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 历史导出结果表 `crawler_okcis_notice_history_exports` 新增 `preview_content`
  - 历史数据采集时，自动从 OKCIS 详情返回内容中提取纯文本预览内容入库
  - 历史导出 Excel 新增 `预览内容` 列，放在 `跟进人` 前面
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/okcis_history_export.py app/api/endpoints/okcis_notices.py migrations/versions/r8s9t0u1v2w3_add_preview_content_to_okcis_history_exports.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/alembic heads`
  - `cd backend && /opt/anaconda3/envs/smart/bin/alembic upgrade head`
- 本次未新增依赖

### 198. 清理 Alembic 迁移树并重置当前库版本到单头
- 文件：
  - `backend/migrations/versions/*.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复多组重复 revision id
  - 消除了迁移图中的循环依赖
  - 将 `p7q8r9s0t1u2_add_attendance_query_indexes.py` 作为总合并头
  - 当前库 `alembic_version` 已重置为 `p7q8r9s0t1u2`
  - 现在 `alembic upgrade head` 可正常执行
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/alembic heads`
  - `cd backend && /opt/anaconda3/envs/smart/bin/alembic upgrade head`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile migrations/versions/*.py`
- 本次未新增依赖

### 197. 优化人事考勤助手考勤汇总页面查询性能
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/models/attendance_checkin.py`
  - `backend/migrations/versions/p7q8r9s0t1u2_add_attendance_query_indexes.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `/api/wecom-attendance/daily-records` 增加 180 秒 Redis 缓存，重复打开相同筛选条件直接命中缓存
  - 页面查询 OA 补算改为读取本地镜像表 `wecom_attendance_oa_approved_records`，不再每次实时查 OA 表
  - 打卡明细查询改为只按当前结果里的 `userid` 回查，避免整段日期范围全员明细扫描
  - 关键词筛选改为直接走当前行 `employee_name/userid`
  - 新增索引：
    - `idx_wecom_attendance_daily_query_order`
    - `idx_wecom_attendance_daily_user_query`
    - `idx_attendance_checkin_user_date_time`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py app/models/wecom_attendance.py app/models/attendance_checkin.py migrations/versions/p7q8r9s0t1u2_add_attendance_query_indexes.py`
- 本次未新增依赖

### 196. 生成全库数据库字典并补齐 test/prod 表字段注释
- 文件：
  - `doc/database_dictionary.md`
  - `backend/sql/fill_database_comments_20260721.sql`
  - `scripts/generate_database_dictionary.py`
  - `AGENTS.md`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增数据库字典生成脚本，读取 `smart-cs-ai-test`、`smart-cs-ai` 两套库结构与模型注释，输出全量表字典文档
  - 新增注释补齐 SQL，按当前库结构自动生成 `ALTER TABLE ... COMMENT` 与 `MODIFY COLUMN ... COMMENT`
  - 已实际执行补齐，两套库当前 `missing_table_comments=0`、`missing_column_comments=0`
  - `AGENTS.md` 新增约束：后续新建/删除/重命名数据表，或新增/删除关键字段时，必须同步更新 `doc/database_dictionary.md`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python ../scripts/generate_database_dictionary.py --apply`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile ../scripts/generate_database_dictionary.py`
- 本次未新增依赖

### 195. 考勤汇总增加汇总项点击明细与迟到问题项高亮
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 考勤汇总中 `年假/带薪病假/事假/调休/产假/婚假/陪产假/丧假/哺乳假/加班时长/迟到早退/旷工/未打卡` 有值时可点击查看详情
  - 请假类详情优先展示 OA 源数据中的 `请假事由`
  - OA 事由中的 `<br>` 等 HTML 标签会清洗后再显示
  - 迟到/早退详情中，具体问题项如 `上班迟到`、`下班早退` 会以红色高亮
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖；前端构建使用 Node `v20.20.2`

### 194. 软件部日报提醒增加周末钉钉实时打卡判断
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `software_work_record_reminder` 在执行日为周六/周日时，先实时调用钉钉接口判断“软件开发部”当天是否有人打卡
  - 部门成员和打卡记录均走钉钉实时接口，不读本地考勤库做是否推送判断
  - 若当天无人打卡，则直接跳过日报提醒，不发送企微群消息
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 193. 细化 OA 考勤修正方向抵扣并固定每日明细日期列
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OA 考勤修正改为读取 `formtable_main_21_dt1` 明细行，按明细打卡日期生效
  - `上班补卡/下班补卡` 仅抵扣对应方向缺卡，并同步影响 `未打卡/旷工`
  - 外出打卡抵扣缺卡后，按实际命中的上班/下班方向重算迟到早退分钟
  - 每日考勤详情宽表第一列日期改为横向滚动时固定显示，左上角日期表头层级高于左侧浮动列
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖；前端构建使用 Node `v20.20.2`

### 192. 软件部工作记录草稿接入康鹏定时自动提交
- 文件：
  - `backend/app/api/endpoints/software_task/software_task.py`
  - `backend/app/schemas/software_task.py`
  - `backend/app/tasks/scheduler.py`
  - `frontend/src/pages/software_task/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 工作记录草稿结构新增 `scheduled_submit_enabled`、`scheduled_submit_at`
  - Redis 草稿同时保存 `user_id`、`record_date`，并按定时提交时间动态延长 TTL，避免草稿跨天失效
  - 前端“新建工作记录”弹窗仅对康鹏账号（`tangpeng/kangpeng`）展示“定时自动提交”
  - 不新增计划任务，改为复用 `knowledge_sync` 任务周期扫描 Redis 草稿
  - 到点后自动写入正式工作记录，并校验项目存在、字段完整、康鹏身份；同用户同日期同项目同工时同内容做幂等去重
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/software_task/software_task.py app/schemas/software_task.py app/tasks/scheduler.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖；前端构建使用 Node `v20.20.2`

### 191. OA 补卡按上下班方向抵扣缺卡
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OA 考勤修正改为读取 `formtable_main_21_dt1.detail_signdate` 作为考勤影响日期，不再使用主表申请日期
  - `detail_signtype=0` 识别为 `上班补卡`，`detail_signtype=1` 识别为 `下班补卡`
  - 补卡时长固定为 `0.5`，只作为考勤修正记录，不参与 `实际出勤` 累加
  - 缺卡抵扣改为按方向执行：上班补卡只抵扣上班缺卡，下班补卡只抵扣下班缺卡
  - 详情接口同步把 OA 补卡记录作为对应方向的抵扣项参与缺卡重算
- 验证：
  - 张辉 `2026-07-09`、`2026-07-10`、`2026-07-11` 均识别为 `下班补卡 0.5`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 190. 修复栗洪涛年假在考勤汇总列表翻倍
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 原因：
  - 考勤汇总列表先查 `wecom_attendance_daily_records`
  - 栗洪涛 `2026-07-06` 至 `2026-07-10` 的 OA 年假已落库到 `annual_leave=1.0`
  - 接口返回前又执行一次 `apply_oa_attendance_adjustments`，同一条 `requestId=1161453` 被再次叠加，5 天被算成 10 天
- 调整：
  - 新增 OA 已应用判断，按 `requestId / request_id / id` 优先识别
  - 若 `oa_leave_json` 或 `oa_correction_json` 已包含同一条 OA 记录，则跳过重复叠加
  - 同步兼容无 `requestId` 时按 `userid + 起止日期 + 类型` 兜底判断
- 验证：
  - 栗洪涛 `2026-07-06` 至 `2026-07-10` 每天 `annual_leave=1.0`
  - 7 月合计 `annual_sum=5.0`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 189. 正式环境修复王慧君 7 月 3 日外出补缺卡后未计入迟到
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 原因：
  - 补算缺卡基准时间时调用 `_parse_json_dict`，但服务类缺少该方法
  - 正式库 `attendance_checkin_records.checkin_time` 查询结果为 `datetime`，旧逻辑只按整数时间戳计算迟到分钟
- 调整：
  - 新增 `_parse_json_dict`
  - 新增 `_timestamp_seconds`，统一兼容 `datetime` / 时间戳
  - `_late_or_early_minutes`、`_item_record_date`、`_item_checkin_datetime`、`_infer_out_attendance_slot` 全部改为走统一时间解析
- 正式环境处理：
  - 已重算 `2026-07-02` 至 `2026-07-03`
  - 王慧君 `WangHuiJun / 王慧君VIP4`：
    - `2026-07-02`：`missing_punch_count=0`、`late_early_within_20=0`
    - `2026-07-03`：`missing_punch_count=0`、`late_early_within_20=1`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 188. 人事考勤每日考勤详情固定日期列
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 在“考勤汇总 -> 详情 -> 每日考勤”弹窗宽表中，将日期表头和日期单元格设为横向 sticky
  - 横向右滑时日期始终停留在左侧，方便对照后面字段
  - 横向滚回左侧时仍为原表格日期列，不增加额外浮层遮挡
- 验证：
  - `cd frontend && ./build.sh`
  - 构建使用 Node `v20.20.2`
- 本次未新增依赖

### 187. 正式环境修复王慧君 7 月 2 日、7 月 3 日外出未抵消缺卡
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 原因：
  - 王慧君两天的外出明细 `checkin_type=外出打卡`，没有直接标明“上班/下班”
  - 旧规则只按显式方向判断，未按时间靠近上/下班时点归位
- 调整：
  - 外出打卡方向改为优先按时间靠近上班/下班时点判定
  - 再抵消对应缺卡；若时间落入迟到/早退 60 分钟内，再计入对应档位
- 正式环境处理：
  - 已重算 `2026-07-02` 至 `2026-07-03`
  - 王慧君 `WangHuiJun / 王慧君VIP4` 重算后：
    - `2026-07-02`：`missing_punch_count=0`、`absenteeism_count=0.0`
    - `2026-07-03`：`missing_punch_count=0`、`absenteeism_count=0.0`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 186. 外出打卡抵消未打卡并补计迟到早退
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增规则：未打卡当天如存在同方向外出打卡（上班/下班），先抵消对应缺卡
  - 外出打卡抵消后，如该次时间落入迟到/早退 60 分钟内区间，补计对应迟到早退档位
  - 已同步改造：
    - 原始企微明细构建日汇总
    - `recalculate_missing_punch_counts` 重算逻辑
    - 考勤汇总/详情接口实时展示口径
  - 重算时补查 `attendance_checkin_records.checkin_time/raw_json`，保证外出时间判断可用
  - 钉钉覆盖企微场景下，同样支持按人员同日外出打卡抵消缺卡
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 185. 招标信息公示自动清理保护跟进数据
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/services/crawler_handlers/okcis.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OKCIS 任务执行前清理过期公告时，跳过已存在跟进记录的公告
  - 单条公告因详情权限不足、截止时间格式无效、截止时间少于等于未来 4 天而准备清理旧数据时，也跳过已有跟进人的公告
  - 新爬取数据命中旧公告执行 `ON DUPLICATE KEY UPDATE` 时，若旧公告已有跟进记录，强制保留 `is_followed=1`，避免跟进人状态被覆盖为空
  - 保护条件为 `crawler_okcis_notice_follow_records.notice_id = crawler_okcis_notices.id`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 184. 正式环境排查刘艳 6 月 28 日休息日仍显示旷工
- 排查范围：
  - 正式库 `attendance_special_date_configs`
  - 正式库 `wecom_attendance_daily_records`
- 结果：
  - `2026-06-28` 已存在默认考勤组休息日配置：`attendance_group_key=default`、`is_rest_day=1`
  - 刘艳（`userid=LiuYan`，部门 `档案室`）当天日汇总已正确归零：
    - `absenteeism_count=0.0`
    - `missing_punch_count=0`
  - 同时原始企微明细 `raw_json` 里仍保留两条 `未打卡` 记录（上班打卡、下班打卡）
  - 因此当前现象不是“统计结果仍记旷工”，而是“原始打卡明细还显示未打卡原文”
- 结论：
  - 后端正式库统计口径已按休息日生效
  - 若页面观感上不希望在休息日继续展示原始未打卡明细，需要单独改前端/接口展示逻辑
- 本次未改代码、未新增依赖

### 183. 考勤汇总与详情接入钉钉请假抵扣旷工
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `/api/wecom-attendance/daily-records` 在合并 OA 请假/修正后，继续合并钉钉请假记录
  - 根据钉钉请假类型把数据写入对应字段，并同步执行旷工抵扣
  - 满足整天抵扣口径时，未打卡也一并清零
- 验证：
  - 已实测刘雁鹏在 `2026-06-27`、`2026-07-01`、`2026-07-03`、`2026-07-10` 四天显示 `调休=1.0`，`旷工=0`，`未打卡=0`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/dingtalk_attendance.py app/services/wecom_attendance.py`
- 本次未新增依赖

### 182. 钉钉请假类型增加本地兜底映射
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 根据用户提供的钉钉请假页面确认编码含义
  - 新增本地兜底映射：`031fc328-ea2c-47e0-87f3-7c95a590d0c5 -> 调休`
  - 当前钉钉请假类型官方接口虽已开通权限，但返回仍为空数组，因此先本地兜底展示中文类型
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 181. 已通过记录接入钉钉请假数据
- 文件：
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `已通过记录` 接口新增合并钉钉请假数据
  - 数据来源：`topapi/attendance/getleavestatus`
  - 输出字段已对齐现有前端表格结构：`channel/record_type/employee_name/department_name/leave_type/start_date/end_date/duration/reason`
  - 已实测 `刘雁鹏` 存在 3 条钉钉请假记录：`2026-07-01`、`2026-07-03`、`2026-07-10`
  - 进一步验证了钉钉请假类型查询接口，但当前应用缺少权限 `qyapi_holiday_readonly`
  - 当前阶段钉钉请假类型先展示 `leave_code`，待权限开通后可再补名称映射
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
- 本次未新增依赖

### 180. 技术项目看板员工姓名去 VIP 后缀合并
- 文件：
  - `backend/app/api/endpoints/tech_project_staff_board.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 问题：
  - 正式环境技术项目看板同一员工出现两行，例如 `程金雨` 和 `程金雨VIP1`
  - 原因是工作记录和部门成员兜底使用的昵称不一致，后端按原始姓名分组，导致 VIP 后缀人员被当作另一个人，其中兜底行没有工作数据
- 调整：
  - 员工分组前统一去掉末尾 `VIP*` 后缀
  - 工作记录和部门成员兜底都会进入同一姓名 key，避免空的重复行
- 验证：
  - 已用正式库 2026-07-14 至 2026-07-20 调用接口验证：自控部 `程金雨` 只返回一行，7/14 至 7/17 数据正常保留
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tech_project_staff_board.py`
- 本次未新增依赖

### 179. 修复考勤详情未合并 OA 考勤修正
- 文件：
  - `backend/app/core/config.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `考勤汇总 -> 详情` 的 `/daily-records` 接口原先只读取 `wecom_attendance_daily_records`
  - 详情链路未再次合并 OA 已通过的请假/考勤修正，导致“仅存在 OA 修正、无原始日汇总”的日期不会显示
  - 现已在查询日汇总后补调用 `wecom_attendance_service.apply_oa_attendance_adjustments(...)`
  - 确认当前口径：考勤修正按补卡日期归属，不按提交日期归属
  - 同步补齐修正抵消逻辑：OA 考勤修正生效后，`missing_punch_count` 与 `absenteeism_count` 直接清零
  - 打卡明细新增 OA 修正补卡记录，备注附带 `OA申请ID`
  - 明细跳转链接改为系统内页：`/apps/wecom-attendance?tab=attendance-correction&request_id=...`
  - `考勤修正` 列表搜索同步支持 `request_id`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 178. 人事考勤汇总列表对齐花名册导出字段
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 考勤汇总列表去掉企微ID列
  - 搜索框改为仅搜索姓名
  - 列表补齐花名册导出相关字段：应有年假、应出勤、实际出勤、年假、带薪病假、事假、调休、产假、婚假、陪产假、丧假、哺乳假、迟到/早退三档、旷工、未打卡
  - “应有年假”当前暂无数据源，导出和列表均填 0
  - 部门列按当前页连续部门使用 `rowSpan` 合并单元格
  - 汇总表列宽按内容收缩，减少无效宽度
  - 新增分页组件，默认每页 50 条，支持 50 / 100 / 200
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 177. OKCIS 招投标爬取改用手机版 JSON 截止时间
- 文件：
  - `backend/app/services/crawler_handlers/okcis.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 爬取阶段不再为了截止时间访问 PC 详情页
  - 改为请求手机版 JSON：`https://m.okcis.cn/...json`
  - 优先读取 `data.bid_time_new`，其次读取 `data.bid_time`
  - 同时解析 `data.content` 中的 `截止时间 / 投标截止时间 / 开标时间 / 响应截止时间`
  - 对 `content` 中的“`YYYY年M月D日H时M分前提交/递交响应文件或投标文件`”识别为 `投标截止时间`
  - 多个候选时间优先选择包含具体时分的时间；同为具体时间或同为日期时，再取最早有效时间作为截止时间
  - 手机版 JSON 出现 `登录后查看`、`*****招标公司` 等脱敏内容时，会触发重新登录刷新凭证后重试
  - 详情缓存 `_fetch_okcis_detail_cache` 去掉 PC 详情预访问，直接用移动 JSON 构建详情 HTML
  - 旧任务路径同步补充 `bid_time_new / bid_time / deadline_at / deadline / bmjzsj` 候选字段
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`
  - 样例验证：`截止时间=2026-07-23` 与 `投标截止时间=2026年07月28日 08:30` 同时存在时，最终取 `2026年07月28日 08:30`
  - 实际 URL 验证：`20260717161259340744` 刷新凭证后提取到 `报名截止时间=2026-07-23`、`投标截止时间=2026年7月28日15时00分`，最终取 `2026年7月28日15时00分`
- 本次未新增依赖

### 176. 正式环境 OKCIS 任务重跑并增加瞬时错误重试
- 文件：
  - `backend/app/services/crawler.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 给爬虫 HTTP 请求增加 502 / 503 / 504 瞬时错误重试
  - 覆盖 OKCIS 登录页、验证码、自动登录请求、重定向和通用 `crawl_api` 请求
  - 重试间隔为 1 秒、3 秒、6 秒
- 正式环境重跑：
  - 任务：`crawler_tasks.id=3` / `okcis_notice_manual`
  - 运行记录：`crawler_task_runs.id=719`
  - 状态：成功
  - 时间：`2026-07-17 16:06:00` 至 `2026-07-17 16:16:02`
  - 请求订阅组 9 个，成功 9 个，失败 0 个
  - 今日有效入库 39 条，覆盖 5 个订阅组
  - 分组：订阅条件组一 4 条、订阅条件组三 4 条、订阅条件组七 14 条、订阅条件组八 7 条、售前营销 10 条
- 说明：
  - 本次日志中的跳过项主要为截止时间格式无效或截止时间少于等于未来 4 天，属于当前过滤规则
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler.py app/services/oa_sync.py app/tasks/scheduler.py`
- 本次未新增依赖

### 175. 修复正式环境 OA 用户部门同步唯一索引冲突
- 文件：
  - `backend/app/services/oa_sync.py`
  - `backend/app/tasks/scheduler.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复别名账号合并时 `oa_resource_id` 唯一索引冲突
  - 写入主账号 OA ID 前，先释放其他用户占用的同一 `oa_resource_id` 并停用旧账号
  - 计划任务最近结果改为中文摘要，避免显示 Python dict 原始 key
- 正式环境验证：
  - 已手动跑通 OA 用户部门同步
  - 结果：部门 64 个，用户 277 个，新增 0，更新 0，停用 0，账号冲突 0
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/oa_sync.py app/tasks/scheduler.py`
- 本次未新增依赖

### 174. Git 提交售前回款看板与考勤相关改动
- 操作：
  - 按用户 `git` 指令，将当前工作区已跟踪和未跟踪改动统一纳入提交
  - 提交内容包含售前营销月度回款看板、OA 回款通知镜像表、前端页面、考勤相关历史改动与文档记录

### 173. 售前营销月度回款看板补数据落表
- 文件：
  - `backend/app/models/presales_payment_dashboard.py`
  - `backend/app/models/__init__.py`
  - `backend/migrations/env.py`
  - `backend/migrations/versions/n6o7p8q9r0s1_create_presales_payment_dashboard_rows.py`
  - `backend/sql/create_presales_payment_dashboard_rows.sql`
  - `backend/scripts/sync_presales_payment_dashboard_rows.py`
  - `backend/app/api/endpoints/presales_payment_dashboard.py`
  - `backend/app/api/api.py`
  - `frontend/src/pages/presales_payment_dashboard/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 `presales_payment_dashboard_rows`，保存 OA 回款通知 `formtable_main_117`
  - 保存提交用户 ID：`submitter_oa_user_id = formtable_main_117.sqr`
  - 保存流程创建人 ID：`creator_oa_user_id = workflow_requestbase.CREATER`
  - 新增同步脚本和接口：summary / rows / sync
  - 员工按当前用户 `oa_resource_id` 过滤本人提交数据；管理员看全部
  - 前端页面读取年度汇总和最近回款列表，管理员后台显示提交人 ID
- 数据处理：
  - 已执行 `backend/sql/create_presales_payment_dashboard_rows.sql` 到 test / prod
  - 已执行 `backend/scripts/sync_presales_payment_dashboard_rows.py --both`
  - test / prod 各写入/更新 `12105` 行，`submitter_oa_user_id`、`creator_oa_user_id` 均有值
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/presales_payment_dashboard.py app/models/__init__.py app/api/api.py app/api/endpoints/presales_payment_dashboard.py scripts/sync_presales_payment_dashboard_rows.py migrations/versions/n6o7p8q9r0s1_create_presales_payment_dashboard_rows.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 172. 新增售前营销月度回款看板卡片
- 文件：
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/sql/upsert_presales_payment_dashboard_card.sql`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `frontend/src/pages/presales_payment_dashboard/index.tsx`
  - `frontend/src/App.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增工作台卡片“售前营销月度回款看板”
  - 新增页面 `/apps/presales-payment-dashboard`
  - 页面默认展示“回款看板”，管理员可切换“管理后台”，看板内容暂不绘制
  - 拆分两个独立权限：
    - 员工：`app:presales_payment_dashboard:access`
    - 管理员：`app:presales_payment_dashboard:admin`
  - 员工权限只看本人提交数据；管理员权限看全部数据
  - 后续数据源确认使用 OA 回款通知 `formtable_main_117`，提交人字段可用 `sqr` 或 `workflow_requestbase.CREATER`
- 数据处理：
  - 已执行 `backend/sql/upsert_presales_payment_dashboard_card.sql` 到 test / prod
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/permissions.py app/db_init.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 171. 定位正式环境 OA 回款通知流程表
- 排查对象：
  - `回款通知-常兵阳VIP3-2026-07-07`
- 结果：
  - `workflow_requestbase.requestid=1169903`
  - `workflowid=396`
  - `workflow_base.FORMID=-117`
  - `workflow_bill.TABLENAME=formtable_main_117`
  - 主数据行：`ecology.formtable_main_117.id=12159`，`requestid=1169903`
- 关键字段：
  - `xmbh` 项目编号
  - `xmmc` 项目名称
  - `htbh` 合同编号
  - `htmc` 合同名称
  - `hkje` 回款金额
  - `sjhkrq` 实际回款日期
  - `htje` 合同额
  - `yhke` 已回款额

### 170. 人事考勤请假对冲旷工口径统一
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 汇总考勤统计里，当天有年假或调休时，旷工 / 未打卡不计
  - 当天其他请假合计满 1 天时，也视为请假覆盖当天，旷工 / 未打卡不计
  - 缺卡提醒、缺卡详情、缺卡导出、花名册导出和列表汇总使用同一口径
  - 缺卡/旷工重算任务读取请假字段后再计算，避免同步后把 OA 请假覆盖日期重新算成旷工
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 168. 2026 年 4 月考勤历史同步和人工表对比
- 操作：
  - 使用用户提供的 `/Users/sunday/Downloads/202604.xlsx`，区间为 2026-04-01 至 2026-04-26
  - 按当前系统规则补同步历史数据：企微考勤、钉钉启用人员覆盖合并、OA 请假/考勤修正、缺卡/旷工重算
  - test 入库结果：日汇总 6636 条、256 人；明细 13192 条、236 人；钉钉 1076 条、27 人
  - prod 入库结果：日汇总 6194 条、239 人；明细 12412 条、221 人；钉钉 1076 条、27 人
  - 生成对比文件：`data/attendance_compare_20260401_20260426_v2.xlsx`
- 对比结论：
  - 源文件 249 人，系统 237 人；按“唯一姓名优先，重名部门+姓名”匹配 216 人
  - 仅源文件有 33 人，仅系统有 21 人
  - 正式库 4 月没有休息日配置，系统按 26 天统计；源文件多数按 24 天统计，应出勤系统合计比源文件多 172
  - 实际出勤系统合计比源文件少 1462；系统当前根据真实打卡/OA修正计算，人工表存在大量默认满勤口径
  - 系统旷工 319、未打卡 338；源文件旷工 0、未打卡 83
  - 请假、调休、迟到分段与源文件也有差异：系统只按 OA 和原始打卡规则统计，人工表疑似另有修正口径
  - 发现重复用户示例：李旭晴在 prod 有 `15692250985` 与 `lixuqing` 两个 userid，同部门同名汇总后应出勤变 52
- 本次未新增依赖

### 169. 2026 年 4 月考勤历史按 prod 休息日重试
- 调整：
  - 仅针对 prod 环境重跑 2026-04-01 至 2026-04-26 考勤历史对比
  - prod 4 月默认休息日共 6 天：4/4、4/5、4/12、4/18、4/19、4/26
  - 生成按休息日过滤后的对比文件：`data/attendance_compare_20260401_20260426_prod_restday.xlsx`
- 结果：
  - 按休息日过滤后，系统应出勤合计 4766，源文件 6022，仍相差 1256
  - 实际出勤系统 4038.5，源文件 5852.5，差距仍较大
  - 说明当前系统汇总口径和这份人工月表仍不一致，但数据已可正常入库作为历史记录
- 本次未新增依赖

### 170. 2026 年 4 月考勤差异去掉实际出勤后的统计
- 调整：
  - 只统计年假、带薪病假、事假、调休、婚假、陪产假、丧假、哺乳假、迟到早退 20 / 20-40 / 40-60 分钟、旷工、未打卡
  - 排除实际出勤后重新查看差异人员和字段分布
- 结果：
  - 差异主要集中在旷工、未打卡，其次是迟到分段和调休
  - 典型差异人员包括：李家熠、韩新宇、徐桥、张辉、刘培、张永勋、李秋杰、程金雨、于海飞、刘超、赵久堂、焦安宝、田新玲、王雨馨、刘健、郭连发、穆雅萌、苏健、郑泽玺
  - 继续使用 `data/attendance_compare_20260401_20260426_prod_restday.xlsx` 作为对比底稿
- 本次未新增依赖

### 169. 人事考勤 OA 请假按天展开和导出文件名调整
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OA 请假 / 考勤修正跨多天时按每天展开，日汇总明细逐日写入
  - 假期时长大于 1 天时，总数按分摊结果严格累计，不再只落到单日
  - 年假、调休补实际出勤，同一天实际出勤最高 1 天
  - 事假、带薪病假仍扣实际出勤，其他假别只记录字段
  - OA 人员当天没有企微日记录时，自动创建基础日汇总记录，避免请假日期漏写
  - “考勤修正 / OA已通过记录”按日期展开展示
  - 考勤汇总导出文件名改为 `考勤汇总_YYYYMMDD-YYYYMMDD.xlsx`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 168. 人事考勤花名册导出口径对齐列表
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复花名册导出实际出勤等字段与页面列表不一致的问题
  - 导出改为按列表同源口径处理日汇总明细，不再只用独立 SQL 聚合
  - 同步应用考勤排除范围、考勤组休息日、白名单、钉钉优先覆盖、实际出勤每日上限、事假/带薪病假扣减
  - 迟到/早退三档改为根据打卡明细和考勤配置实时计算，和列表一致
  - 模板存在应出勤、外出、旷工、未打卡列时，也按同一数据源填充
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py`
- 本次未新增依赖

### 196. 人事考勤按入离职日期排除旷工
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 缺卡/旷工重算时读取 OA `hrmresource` 员工生命周期日期
  - 入职日期优先使用 `STARTDATE`，为空时使用 `companystartdate`、`workstartdate`
  - 离职日期使用 `ENDDATE`
  - 仅在能获取到确定日期时应用：入职前、离职后的旷工和未打卡不计；取不到日期则保持原逻辑
- 数据处理：
  - 已确认赵广文 `companystartdate/workstartdate=2026-07-10`
  - 已重算 test / prod 2026-07-01 至 2026-07-31 缺卡/旷工数据
  - 赵广文 2026-07-01 至 2026-07-09 旷工已清零
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 170. 人事考勤 2026 年 4 月迟到早退与未打卡差异复核
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 用户提供 `2026.4月考勤-汇总表.xlsx`，对比正式库 2026-04-01 至 2026-04-26
  - 原迟到早退差异主要因为页面 / 导出口径按打卡时间差重新计算，把 `地点异常` 也误算进迟到早退
  - 剩余焦安宝 1 次差异定位到 2026-04-25：企微原始状态为 `时间异常`，且当天 `expected_attendance=1`，但旧逻辑默认未配置周六为休息日，导致过滤掉
  - 剩余未打卡差异来自口径：季鹏、刘洪泽在源表保留未打卡，系统按请假覆盖和休息日过滤后为 0
- 调整：
  - 迟到早退只统计 `迟到` / `早退` / `时间异常`，不再统计 `地点异常`
  - 服务层 `_apply_exception_stats` 同步同一规则
  - 周六休息日判断补充：手工配置休息日优先；无配置但日汇总 `expected_attendance > 0` 时按工作日统计
- 结果：
  - 正式库复核：迟到早退差异已归零
  - 未打卡剩余 2 人为业务口径差异，不是漏抓
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
- 本次未新增依赖

### 195. OKCIS 爬虫任务3定时与详情访问节流
- 文件：
  - `backend/app/services/crawler_handlers/okcis.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OKCIS 详情访问增加节流：每访问 10 个详情页，随机暂停 3 到 10 秒
  - 日志会记录 `[DETAIL-PAUSE]`
- 数据配置：
  - 已同步更新 test / prod 两套环境
  - `crawler_tasks.id=3` / `crawler_task_3` cron 改为 `0 5 15 * * *`，即每天 15:05:00 执行
  - 同步更新 `crawler_tasks` 与 `scheduler_job_meta`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 194. 招投标群发推送空数据原因写入日志摘要
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `push_okcis_daily_summary` 在查询当天招投标数据为空时，返回明确摘要并写入任务日志
  - 空数据摘要包含日期、订阅组、数据为空原因、机器人 ID
  - 公共任务包装器 `_execute_job_with_log_capture` 支持 runner 返回值作为本次成功摘要，计划任务列表和详情可直接看到“未发送原因”
  - 适用于售后招投标推送和售前/营销管理中心招投标群发推送
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
- 本次未新增依赖

### 193. OKCIS 指定 dzid 执行与售前营销正式写库
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/services/crawler_handlers/okcis.py`
  - `frontend/src/pages/admin/CrawlerTaskDetailPage.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 手动执行爬虫任务支持传 `dzid`
  - OKCIS 任务传 `dzid` 时只抓指定订阅组，不传则继续抓全部订阅组
  - OKCIS multipart `raw_body` 内的 `page/pageSize/size` 支持跟随分页参数变化
  - 任务详情页为 OKCIS 任务增加“订阅组 dzid”输入框
  - 详情页返回“权限不够 / 登录后查看”时先刷新登录凭证重试；重试后仍不可见则跳过该条并清理旧数据
- 正式环境执行：
  - 单独执行 `okcis_notice_manual`，`dzid=187653`（售前营销），`run_id=682`
  - 复查 `crawler_okcis_notices`：`dzid=187653` 共 29 条，2026-07-16 当天 16 条，空截止时间 0 条，权限不足记录 0 条
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py app/api/endpoints/admin/crawler_tasks.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 192. 正式环境系统基础数据同步到测试环境
- 操作：
  - 将正式库系统基础数据表覆盖同步到测试库
  - 同步前备份测试库原基础数据：`/tmp/smart_cs_ai_test_base_data_backup_20260716_161408.json`
- 同步表：
  - `departments`、`users`、`roles`、`permissions`、`user_roles`、`role_permissions`、`role_departments`
  - `dashboard_app_cards`、`scheduler_job_meta`、`crawler_tasks`、`crawler_site_credentials`
  - `wecom_message_templates`、`wecom_message_robots`、`wecom_message_robot_templates`
  - `auto_ctrl_members`、`auto_ctrl_work_categories`、`auto_ctrl_work_hour_base`
  - `software_projects`、`software_task_bases`
  - `water_supply_business_areas`、`water_supply_project_types`、`water_supply_work_categories`
  - `procurement_supplier_categories`、`procurement_buyer_wecom_u8_map`
  - `after_sales_management_dashboard_members`、`after_sales_work_order_monitor_accounts`
  - `attendance_excluded_departments`、`attendance_special_date_configs`、`attendance_whitelist_employees`
  - `dingtalk_attendance_users`、`knowledge_bases`
- 复查：
  - 已逐表比对 test/prod 行数，全部一致
  - 未同步业务流水/明细数据，避免覆盖测试环境业务记录
- 本次未改代码，未新增依赖

### 191. 修复考勤配置时间插件回显和时长单位
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 考勤修正列表表头“时长”改为“时长（天）”
  - 修复考勤配置时间设置点击编辑后时间插件不回显的问题
  - 兼容接口返回的 `PT14H40M` 这类 ISO duration 格式，统一转换为时间输入框可识别的 `14:40`
  - 时间设置列表里的上班 / 下班参照展示也统一格式化为 `HH:mm`
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 190. 修复客户无忧客户列表最后一页多计数
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复正式环境“售后部门：客户列表同步任务（客户无忧爬虫任务）”王贺源站总数 14、分页返回 50 条导致校验失败的问题
  - `kehu51_customer_list` 任务在已知源站总数时，按剩余条数裁剪当前页 `items`
  - 裁剪后再写业务表和累计抓取数量，避免 `source_total=14, crawled_total=50`
  - 增加 `[DATA-TRIM]` 日志，标明源站剩余条数和裁剪数量
  - 仅对 `kehu51_customer_list` 生效，不影响跟进记录和其他爬虫任务
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 189. 考勤修正补出勤并限制实际出勤上限
- 文件：
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/sql/alter_wecom_attendance_actual_decimal.sql`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 如果当天实际出勤已经记 1 天，OA 考勤修正不再增加实际出勤
  - 实际出勤每天最高上限为 1 天
  - OA 考勤修正只补当天实际出勤不足 1 天的差额，“考勤修正”列记录实际补入天数
  - 前端实际出勤展示也按每天最高 1 天兜底
  - `actual_attendance` 从 INT 改为 `DECIMAL(6,1)`，避免半天修正被截断
- 数据处理：
  - 已在 test / prod 执行 `backend/sql/alter_wecom_attendance_actual_decimal.sql`
  - 已确认两套环境 `actual_attendance` 与 `attendance_correction_count` 均为 `decimal(6,1)`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 188. 考勤修正字段改为小数天
- 文件：
  - `backend/app/models/wecom_attendance.py`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/sql/alter_wecom_attendance_correction_decimal.sql`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 将 `wecom_attendance_daily_records.attendance_correction_count` 从 INT 改为 `DECIMAL(6,1)`
  - 字段注释改为“考勤修正天数”
  - 避免 0.5 天等 OA 考勤修正时长在保存时被截断
- 数据处理：
  - 已在 test / prod 执行 `backend/sql/alter_wecom_attendance_correction_decimal.sql`
  - 已确认两套环境字段均为 `decimal(6,1)`，默认值 `0.0`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 187. 人事考勤助手考勤修正按天数计入口径
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OA 考勤修正时长单位按“天”处理
  - 考勤修正累计到每日考勤“考勤修正”列，支持小数天
  - 实际出勤展示口径改为：原始实际出勤 + 考勤修正天数 - 事假 - 带薪病假
  - OA 考勤修正表字段兼容读取 `duration/fromTime/toTime/sc/ts`，没有则按日期每天 1 天兜底
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 186. 人事考勤助手新增考勤修正tab
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 人事考勤助手新增“考勤修正”tab
  - 后端新增 `GET /api/wecom-attendance/oa-approved-records`
  - 数据源复用 OA 已通过流程：请假 `formtable_main_16`、考勤修正 `formtable_main_21`，仅取 `workflow_requestbase.CURRENTNODETYPE='3'`
  - 前端展示字段：类型、部门、姓名、请假类型、开始时间、结束时间、时长、原因
  - 支持按页面日期范围、部门、姓名 / 类型 / 原因搜索和重置
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 185. 人事考勤助手白名单人员退出考勤范围
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 人员白名单不再作为“考勤正常”参与统计，改为从考勤人员范围整体排除
  - 企微考勤同步、手动查询、日汇总列表、详情明细、缺卡重算均使用同一套排除范围
  - 排除部门会按企微部门树展开，并在历史列表查询中按部门名过滤
  - test / prod 均已确认 `安徽营销中心` 已配置排除；历史旧数据仍存在但接口过滤后可见数量为 0
  - 白名单历史行：test 44 条、prod 63 条；新逻辑已从考勤范围排除
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 184. 人事考勤助手考勤配置新增排除部门
- 文件：
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/models/__init__.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/sql/create_attendance_excluded_departments.sql`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增表 `attendance_excluded_departments` 保存考勤排除部门
  - 考勤配置新增“排除部门”子 tab，支持搜索多选企微部门
  - 可选择大部门；后端同步时自动展开所有子部门并排除部门下员工
  - 排除部门列表只展示部门名和删除操作
  - 企微考勤同步接口加载已启用排除部门，传给 `get_checkin_data`
  - `get_checkin_data` 会先过滤企微员工；若手动传入 userids，也会与过滤后人员取交集
  - 修复“时间设置”编辑时上班 / 下班时间不回显问题，前端将 `HH:mm:ss` 规范化为 `HH:mm`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_attendance_excluded_departments.sql`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 168. 人事考勤入职/离职日期写入员工基础信息表
- 调整：
  - `users` 表新增/确认 `hire_date`、`leave_date` 字段，作为员工基础信息里的入职/离职日期
  - OA 用户部门同步任务同步 `hrmresource.STARTDATE / companystartdate / workstartdate / ENDDATE`
  - 入职日期优先取 `STARTDATE`，为空时用 `companystartdate`、`workstartdate` 兜底；离职日期取 `ENDDATE`
  - OA 同步改为别名主账号优先：别名账号命中 OA 登录名时，更新主账号并释放重复账号占用的 `oa_resource_id`
  - 已修复康鹏：主账号 `tangpeng` 写入 `oa_resource_id=759`、`hire_date=2026-04-20`，重复账号 `kangpeng` 已停用
  - 已在 test / prod 执行 `backend/sql/alter_users_add_employee_lifecycle_dates.sql`
  - 已在 test / prod 执行 OA 用户部门同步：两套环境活跃用户均有 166 人写入入职日期
  - 已重算 test / prod 2026-01-01 至 2026-07-16 缺卡/旷工
  - 复查康鹏 2026-04-01 至 2026-04-26：test/prod 主账号 `tangpeng` 旷工均为 0
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/oa_sync.py app/services/wecom_attendance.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 169. 无入职日期按 2026-01-01 并重新生成 4 月考勤对比
- 调整：
  - 无入职日期员工统一按 `2026-01-01` 作为入职日期
  - OA 用户同步中如果 `STARTDATE / companystartdate / workstartdate` 均为空，写入默认入职日期 `2026-01-01`
  - 考勤生命周期兜底逻辑也按 `2026-01-01` 处理，避免未落库人员绕过生命周期规则
  - `backend/sql/alter_users_add_employee_lifecycle_dates.sql` 已追加历史补值：`hire_date IS NULL` 更新为 `2026-01-01`
  - 已在 test / prod 执行 SQL，并重新执行 OA 用户同步
  - 已重算 test / prod 2026-01-01 至 2026-07-16 缺卡/旷工
  - 已重新生成 2026 年 4 月考勤对比：`data/attendance_compare_20260401_20260426_default_hire_20260101.xlsx`
- 新版对比结果：
  - 源表总人数 249
  - 源表应出勤不等于实际出勤人数 25
  - 存在字段差异人数 20
  - 字段差异条数 54
  - 差异集中在：迟到早退 20-40 分钟、迟到早退 20 分钟内、未打卡、旷工、事假
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/oa_sync.py app/services/wecom_attendance.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 170. 修复 OA 用户同步大小写冲突误报
- 调整：
  - OA 用户同步判断 `sso_uid` 冲突时改为大小写不敏感
  - `LiZhaoMin/lizhaomin`、`TianXu/tianxu` 这类仅大小写不同的账号不再记为冲突
  - 已手动执行 test / prod OA 用户同步复查：两套环境 `sso_conflict_count=0`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/oa_sync.py app/services/wecom_attendance.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 171. 考勤请假类型严格按 OA 类型码并重做关键字段对比
- 调整：
  - OA 请假分类改为严格按 `newLeaveType` 类型码，不再根据请假原因文字判断
  - 当前映射：`2=年假`、`4=带薪病假`、`5=调休`、`6=事假`、`8=产假`、`9=陪产假`、`10=婚假`、`11=丧假`、`12=哺乳假`
  - 企微打卡备注中的“事假/病假”等文字不再写入请假字段，避免和 OA 请假重复叠加
  - 入职当天及之前、离职后不再计算旷工、未打卡、迟到早退
  - 已重跑 test / prod 2026-04-01 至 2026-04-26 考勤同步和缺卡/旷工重算
  - 复查：李家熠 `newLeaveType=6` 归事假 2 天；郭连发 `newLeaveType=6` 归事假
- 重新对比：
  - 文件：`data/attendance_compare_20260401_20260426_key_fields.xlsx`
  - 口径：先排除源表“应出勤=实际出勤”的人员，只统计事假、迟到早退 20 分钟内、20-40 分钟、40-60 分钟、旷工、未打卡
  - 源表异常人数 25
  - 存在字段差异人数 18
  - 字段差异条数 49
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/services/oa_sync.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 172. 修正未打卡统计不再反推
- 调整：
  - 未打卡 / 旷工重算改为只使用考勤明细中明确返回的“未打卡/缺卡”记录
  - 不再因为当天只有上班或下班一条有效打卡，就反推另一条为未打卡
  - 钉钉覆盖规则保留：同一天有钉钉明细时，只按钉钉明确缺卡记录计算
  - 已重算 test / prod 2026-04-01 至 2026-04-26
  - 刘洪泽原先系统 3 次未打卡来自 4/7、4/15、4/21 的反推；修正后当前有效未打卡为 4/23 这 1 次，4/10 被事假覆盖，4/11、4/12、4/26 为休息日不计
  - 已重新生成 `data/attendance_compare_20260401_20260426_key_fields.xlsx`
- 新版关键字段对比：
  - 源表异常人数 25
  - 存在字段差异人数 18
  - 字段差异条数 47
  - 未打卡差异降为 7 人，系统合计比源表多 10 次
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/services/oa_sync.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 173. 旷工支持按请假时长扣减
- 调整：
  - `wecom_attendance_daily_records.absenteeism_count` 从 `INT` 改为 `DECIMAL(6,1)`，支持 0.5 天旷工
  - 新增 SQL：`backend/sql/alter_wecom_attendance_absenteeism_decimal.sql`
  - 已在 test / prod 执行字段类型变更
  - 旷工当天如有事假、带薪病假/病假、调休，按对应时长扣减旷工，最低为 0
  - 页面汇总和每日列表接口同步按该口径展示
  - 已重算 test / prod 2026-04-01 至 2026-04-26
  - 正式库复查：当前仍有旷工的记录，当天均无事假/病假/调休可抵扣
  - 已重新生成 `data/attendance_compare_20260401_20260426_key_fields.xlsx`
- 新版关键字段对比：
  - 源表异常人数 25
  - 存在字段差异人数 18
  - 字段差异条数 48
  - 旷工差异为 12 人，系统合计比源表多 16 天
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/services/oa_sync.py app/services/wecom.py app/models/base.py`
- 本次未新增依赖

### 183. 修复考勤缺卡统计误判并隐藏钉钉记录列
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复每日考勤详情显示“未打卡”，但汇总旷工 / 未打卡仍为 0 的问题
  - 原因是缺卡重算查询 `attendance_checkin_records` 时漏取 `checkin_status`，导致“未打卡”的上班/下班明细被当成已打卡
  - 后端缺卡重算已按 `checkin_status` 正确排除“缺卡/未打卡”明细
  - 前端缺卡详情判断同步排除“缺卡/未打卡”明细
  - 每日考勤详情弹窗去掉“钉钉记录”列
- 数据处理：
  - 已重算 test / prod 从 2026-01-01 到当前日期前一天的缺卡/旷工统计
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 182. 人事考勤助手考勤汇总接入考勤配置口径
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 原“企微考勤汇总”tab 文案改为“考勤汇总”
  - 去掉考勤汇总 tab 的管理员特殊权限，和其他 tab 一样展示
  - 汇总前统一套用考勤配置
  - 休息日整天不纳入考勤汇总，不计实际出勤、缺勤、缺卡、迟到
  - 白名单人员不计缺勤、缺卡、迟到，实际出勤按正常出勤处理
  - 时间设置按配置的上班 / 下班参照时间重新计算迟到 / 早退分段
  - 每日考勤详情去掉“应出勤”列
  - 修复企微缺卡 / 未打卡明细错误写入计划打卡时间的问题：缺卡 / 未打卡记录不再写入实际打卡时间
  - 缺卡重算时只把非缺卡明细计为已打卡，避免缺卡明细反向变成已打卡
  - 新增 SQL：`backend/sql/fix_attendance_missing_punch_checkin_time.sql`
  - 已在 test / prod 清洗历史数据，复查“缺卡/未打卡但有实际打卡时间”均为 0
- 已验证：
  - `cd frontend && ./build.sh`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py`
- 本次未新增依赖

### 180. 人事考勤助手异常考勤新增导出
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 异常考勤 tab 增加“导出”按钮
  - 导出当前筛选内容，包含已搜索日期/部门/姓名范围、异常类型、地点异常人员分类
  - 导出字段：部门、姓名、日期、异常打卡类型、状态、打卡时间、地点
  - 同一天多条异常打卡按明细逐行导出，不合并行
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖，复用已有 `xlsx`

### 181. 人事考勤助手缺卡考勤新增导出
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `frontend/dist.zip`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 缺卡考勤 tab 增加“导出”按钮
  - 导出当前已搜索筛选范围内的缺卡明细
  - 导出字段：部门、姓名、日期、打卡类型缺卡
  - 同一天上班、下班都缺卡时拆成两行
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖，复用已有 `xlsx`

### 179. 修复客户无忧导出预览客户表头短名
- 文件：
  - `backend/app/services/kehu51_follow_export_summary.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 排查马超客户 `source_customer_id=90901478`
  - 原因：导出汇总表按源站客户 ID 分组时使用同组第一条客户名；历史第一条客户名为“刘先生”，后续记录为“河北清源水利工程有限公司-刘先生”
  - 汇总同步逻辑增加客户名优先级，同组内优先使用更完整的客户名
  - 优先选择含公司/医院/学校/水厂/集团/中心/项目/工程等标识，或包含 `-` 的更长名称
- 数据处理：
  - 已重新同步 test / prod 的 `after_sales_customer_follow_export_summary`
  - test：`source_customer_id=90901478` 汇总名已为“河北清源水利工程有限公司-刘先生”，跟进数 39
  - prod：`source_customer_id=90901478` 汇总名已为“河北清源水利工程有限公司-刘先生”，跟进数 41
  - 已删除 2026 年 test / prod 的客户无忧导出缓存 `all / online / offline`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/kehu51_follow_export_summary.py`
- 本次未新增依赖

### 178. 招标信息公示列表新增公告正文预览
- 文件：
  - `backend/app/api/endpoints/okcis_notices.py`
  - `frontend/src/pages/OkcisNoticesPage.tsx`
  - `frontend/src/index.css`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 招标信息公示列表“链接”列新增“预览 / 打开”
  - “打开 / 打开原文”根据当前浏览器判断跳转地址：手机端使用 `https://m.okcis.cn/m/pages/index/TenderDetails?link=/{path}.json`，电脑端使用 `https://www.okcis.cn/{path}.html`
  - 新增后端接口 `GET /api/okcis/notices/preview`
  - 预览接口先用 OKCIS PC 登录态访问电脑版详情，再访问移动端页面，最后带同一套凭证 `POST https://m.okcis.cn/{path}.json`，表单参数 `return_json=1`
  - 若接口返回“登录后查看”等脱敏内容，会自动刷新 OKCIS 登录凭证后重试；仍失败则返回登录失效错误，不展示脱敏内容
  - 移动端 JSON 返回后抽取 `data[0].content` 等正文内容，在本系统弹窗格式化展示，不使用 iframe
  - 移动端 JSON 详情仅展示有值字段，支持：招标事项、所属地区、所属行业、招标关键词、招标类型、项目名称、办公地址、招标类别、计划工期、项目概述及规模、招标范围/内容、资质/要求、报名截止时间、联系人姓名、联系人电话、联系人邮箱
  - 后端返回前清理脚本、事件属性和危险链接；纯文本正文自动转段落，前端弹窗保留表格、段落、列表、加粗、图片等基础样式
  - 预览弹窗已适配手机端：小屏全屏展示，表格横向滚动，字段标签改为块级显示
  - 招标信息公示页面已增加移动端适配：筛选区小屏单列/双列展示，当前数据列表小屏改为卡片列表，历史导出任务表小屏横向滚动，分页区小屏上下排列
  - 公告正文若被源站压成整段文本，会按“一、二、三、”“1、2、”“2.1.”和常见公告字段自动切分成段落
  - 新增详情缓存字段：`okcis_detail_title`、`okcis_detail_html`、`okcis_detail_crawled_at`
  - SQL：`backend/sql/add_okcis_notice_detail_cache.sql`
  - 已在 test / prod 执行详情缓存字段 SQL
  - `crawler_task id=3 / okcis_notice_manual` 已合并详情抓取：列表爬取写入公告时，同步调用 OKCIS 移动端详情接口并写入本地详情缓存
  - 预览接口优先读取 `crawler_okcis_notices.okcis_detail_html`，没有缓存时才实时抓取并回写
  - 新增一次性补抓脚本：`backend/scripts/prefetch_okcis_notice_details.py`
  - 正式环境现有数据已补抓详情：`total=113`，已缓存 `111`，剩余 `2` 条源站暂时未返回可用详情：
    - `id=295`：2026年农村公路南白滩(南白滩-南彭家庄)改造提升工程，源站返回脱敏详情
    - `id=409`：保定废旧铝电缆一批，源站移动端详情接口返回 502
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/okcis_notices.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 177. 用户列表部门筛选支持大部门包含子部门
- 文件：
  - `backend/app/api/endpoints/admin/users.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 管理后台用户列表 `/admin/users` 的 `department_id` 筛选不再只匹配当前部门
  - 后端根据选中部门 `tree_path` 自动扩展当前部门及所有子部门 ID
  - 前端用户列表无需调整，继续传原部门 ID 即可
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/users.py`
- 本次未新增依赖

### 176. 清理未使用角色 yxzx
- 调整：
  - 查询 test / prod 两套环境 RBAC 角色 `yxzx`
  - 两套环境角色均存在，名称为“营销中心”
  - 两套环境 `user_roles` 绑定用户数均为 0
  - 两套环境 `role_permissions`、`role_departments` 关联数均为 0
  - 已删除 test / prod 的 `roles.slug='yxzx'`
  - 复查确认 test / prod 的 `roles`、`user_roles`、`role_permissions`、`role_departments` 中 `yxzx` 均为 0
- 本次未改业务代码，未新增依赖

### 175. 权限定义与角色权限支持部门搜索筛选
- 文件：
  - `frontend/src/components/admin/DepartmentSearchSelect.tsx`
  - `frontend/src/components/admin/PermissionEditor.tsx`
  - `frontend/src/components/admin/RoleEditor.tsx`
  - `frontend/src/pages/admin/PermissionsPage.tsx`
  - `frontend/src/pages/admin/RolesPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增普通搜索下拉部门组件，替代权限定义、角色与权限编辑中的树形部门下拉
  - 父级大部门可直接选择
  - “所属部门”字段仅用于列表归类和筛选，不参与实际权限判定
  - 权限定义列表新增名称搜索、部门筛选
  - 角色与权限列表新增名称搜索、部门筛选
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 174. 客户无忧跟进记录列字段与 raw_json 错位修复
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/sql/fix_kehu51_follow_records_raw_json_mismatch.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复客户跟进记录全量同步使用 `run_id + page + item_index` 作为去重键导致的列字段与 `raw_json` 错位风险
  - 跟进记录新去重键改为业务稳定字段：源站客户ID、联系人ID、客户名、联系人、联系方式、跟进时间、录入时间、录入人、跟进内容
  - `ON DUPLICATE KEY UPDATE` 同步更新 `customer_name` 与 `page_no`
  - 后台手动任务分支与 crawler handler 分支去重规则统一
  - 新增历史清洗 SQL，按 `raw_json` 修正列字段不一致的数据
- 数据处理：
  - 已在 test / prod 执行 `backend/sql/fix_kehu51_follow_records_raw_json_mismatch.sql`
  - 正式库 `crawler_kehu51_follow_records.id=44503` 已修正为“智方能碳科技有限公司（张女士）”
  - 新增并执行 `backend/scripts/fix_kehu51_follow_record_history.py`
  - test：42252 条历史去重键已迁移，删除重复 0 条
  - prod：42378 条历史去重键已迁移，删除重复 1 条
  - 复查结果：test / prod 的旧 key、业务重复、客户名与 raw_json 错位均为 0
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/fix_kehu51_follow_record_history.py app/services/crawler_handlers/kehu51.py app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 173. 前端新增读取 nvm 版本的统一构建脚本
- 文件：
  - `frontend/build.sh`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 构建脚本自动加载 `nvm`，通过 `.nvmrc` 使用 Node 20
  - 执行前端构建后重新生成 `dist.zip`
  - 压缩包排除 `.DS_Store` 和 `__MACOSX` 文件
- 本次未新增依赖

### 172. 招标信息公示 URL 订阅组筛选支持重置为全部
- 文件：
  - `frontend/src/pages/OkcisNoticesPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - URL 订阅组参数只用于首次进入页面初始化筛选
  - 点击“重置”后清空订阅组条件，并从地址栏移除 `group_name`、`group`、`dzid`
  - 带 `group_name=售前营销` 进入页面后可恢复查看全部数据
- 本次未新增依赖

### 171. 招标信息公示 URL 订阅组筛选与售前推送模板
- 文件：
  - `frontend/src/pages/OkcisNoticesPage.tsx`
  - `backend/app/tasks/scheduler.py`
  - `backend/sql/upsert_okcis_presales_push_template.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 招标信息公示页面支持通过 URL 参数 `group_name`、`group`、`dzid` 初始化订阅组筛选
  - 售前招投标群发任务详情链接增加 `group_name=售前营销`
  - 售前推送统计按售前营销订阅组过滤，保证推送数量和点击后的筛选列表一致
  - 新增 `售前招投标企业推送模板`，template_key=`okcis_notice_daily_presales_group_robot`
  - test / prod 已执行 SQL，创建售前模板并将机器人 ID=3 绑定到该模板
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/upsert_okcis_presales_push_template.sql`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 170. OKCIS 响应截止时间解析与目标公告补数
- 文件：
  - `backend/app/services/crawler_handlers/okcis.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复 OKCIS 详情页字段“响应截止时间”未被识别，导致公告被判定为截止时间格式无效并清理的问题
  - 两处写库逻辑均加入“响应截止时间”识别：handler 与后台手动任务分支
  - 正式库已补入 `dzid=18117 / 订阅条件组一` 下两条公告：
    - `20260714095644063753`，轻轨张贵庄站南侧地块(住宅)二供泵房新建工程，`deadline_at=2026-07-21 09:30:00`
    - `20260714095644059239`，轻轨张贵庄站南侧地块(公建)二供泵房新建工程，`deadline_at=2026-07-21 09:30:00`
- 排查结论：
  - 正式环境最近一次 OKCIS 任务 `run_id=599` 为 success，成功 9、失败 0
  - 页面失败提示可能来自此前 `run_id=596` partial 或前端旧状态
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 169. OKCIS 公告截止时间原文超长保护
- 文件：
  - `backend/app/services/crawler_handlers/okcis.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 修复 OKCIS 公告写入 `crawler_okcis_notices.deadline_raw` 时，因抓到整段 HTML / 长文本导致 `Data too long for column 'deadline_raw'` 的问题
  - 新增 `_normalize_okcis_deadline_raw`，保存前去 HTML、压缩空白；仍超过 255 时保存标准化 `deadline_at` 字符串
  - 不调整爬取范围和表结构，只避免有效公告被异常跳过
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py app/api/endpoints/okcis_notices.py`
- 本次未新增依赖

### 168. 招标信息公示导出按筛选全量导出
- 文件：
  - `backend/app/api/endpoints/okcis_notices.py`
  - `frontend/src/pages/OkcisNoticesPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 招标信息公示导出接口只使用当前筛选条件，不使用当前页和分页参数
  - 导出列去掉“采购人”“信息来源”
  - 导出文件名改为 `招标信息公示_{订阅组或全部}_YYYY-mm-dd-H-M-S.xlsx`
  - 有订阅组筛选时显示订阅组名，没有订阅组筛选时显示“全部”
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/okcis_notices.py`
- 本次未新增依赖

### 167. 招标信息公示订阅组兼容 OKCIS 最新返回结构
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/services/crawler_handlers/okcis.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - OKCIS 订阅组接口 `myConditions` 最新返回项已从数组结构变为对象结构
  - 后台数据爬取预取逻辑和 OKCIS handler 已兼容对象结构
  - 支持从 `id/group_name/name` 等字段解析订阅组 ID 和名称
  - Redis 缓存 `crawler:okcis:dingzhi_groups` 继续保留 30 天，并写入 `updated_at`、`groups`、`dzid_list`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 166. 人事考勤助手新增异常考勤 tab
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 人事考勤助手新增 `异常考勤` tab
  - 后端日汇总接口返回打卡明细 `raw_json`
  - 前端基于打卡明细计算并展示迟到/早退超过60分钟、地点异常
  - 地点异常增加分类筛选：营销管理中心、其他人员、全部人员
  - 详情弹窗展示部门、日期、打卡详情
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 165. 人事考勤助手缺卡统计改为基于打卡明细
- 文件：
  - `backend/app/models/attendance_checkin.py`
  - `backend/app/models/__init__.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/api/endpoints/dingtalk_attendance.py`
  - `backend/sql/create_attendance_checkin_records.sql`
  - `backend/migrations/versions/f6a7b8c9d0e1_create_attendance_checkin_records.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
- 调整：
  - 新增统一打卡明细表 `attendance_checkin_records`
  - 企微 / 钉钉同步均写入打卡类型、打卡状态、打卡地点、实际打卡时间和原始 JSON
  - “提醒请假”改为按明细判断上班 / 下班是否存在：工作日两者都缺记旷工，只缺一个记未打卡
  - 人事考勤详情弹窗新增“打卡明细”展示
  - “提醒请假”列表新增全选 / 多选和“批量提醒”按钮，按选中人员企微ID发送应用文本消息
  - 批量提醒固定文案为用户指定文案，提醒本月 26 号前完成考勤修正或请假手续
  - 别名账号合并后，旷工 / 未打卡按同一人同一天最多 1 次计算，避免重复叠加
  - 提醒请假详情弹窗改为“缺卡详情”，仅展示缺卡日期，列为日期、部门、打卡明细
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_attendance_checkin_records.sql`
  - 测试环境补同步 2026-06-25 至 2026-07-14 企微 / 钉钉考勤明细
- 抽检：
  - test / prod 均有企微明细 `10448` 条、钉钉明细 `1104` 条
  - test / prod 提醒请假汇总均为旷工 `102`、未打卡 `335`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/attendance_checkin.py app/models/__init__.py app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/api/endpoints/wecom_attendance.py app/api/endpoints/dingtalk_attendance.py migrations/versions/f6a7b8c9d0e1_create_attendance_checkin_records.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 164. 招标信息公示新增跟进人与删除记录
- 文件：
  - `backend/app/models/okcis_notice.py`
  - `backend/app/models/__init__.py`
  - `backend/app/api/endpoints/okcis_notices.py`
  - `backend/sql/create_okcis_notice_follow_records.sql`
  - `backend/sql/create_okcis_notice_hidden_records.sql`
  - `backend/migrations/versions/g6a7b8c9d0e1_create_okcis_notice_follow_records.py`
  - `backend/migrations/versions/h6a7b8c9d0e1_create_okcis_notice_hidden_records.py`
  - `frontend/src/pages/OkcisNoticesPage.tsx`
- 调整：
  - 新增 `crawler_okcis_notice_follow_records`，记录公告跟进人和跟进时间
  - 新增 `crawler_okcis_notice_hidden_records`，记录当前用户删除操作人和删除时间
  - OKCIS 列表接口返回 `follow_records`、`follow_names`、`current_user_followed`、`hidden_records`
  - 跟进人支持筛选；列表新增“跟进人”列，按跟进时间正序展示
  - 点击跟进人弹出详情，展示姓名、跟进时间、删除操作人、删除时间
  - 删除按钮文案保持“删除”，但不再物理删除公告，只对当前登录用户隐藏该条记录；其他用户仍可查看
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_okcis_notice_follow_records.sql`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_okcis_notice_hidden_records.sql`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/okcis_notice.py app/models/__init__.py app/api/endpoints/okcis_notices.py migrations/versions/g6a7b8c9d0e1_create_okcis_notice_follow_records.py migrations/versions/h6a7b8c9d0e1_create_okcis_notice_hidden_records.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 163. 晨天润达业务团队看板推送超时重试
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/services/wecom.py`
- 调整：
  - 计划任务失败日志不再只记录 `str(e)`，空异常会显示异常类型，并把 traceback 写入任务运行日志
  - `logger.error` 改为 `logger.exception`，服务日志中可看到堆栈
  - 企微请求客户端默认超时改为 30 秒，连接超时 10 秒
  - 企微群机器人图片发送遇到超时后重试 2 次：等待 30 秒后第一次重试，等待 60 秒后第二次重试
  - 超时 / 网络异常会写入企微消息发送日志，便于后续排查
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/services/wecom.py`
- 本次未新增依赖

### 162. 招标信息公示拆分独立权限
- 文件：
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/app/api/endpoints/okcis_notices.py`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/sql/update_okcis_notices_permission.sql`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/App.tsx`
  - `frontend/src/pages/AppCenterPage.tsx`
- 调整：
  - 新增权限 `app:okcis_notices:access`，描述“访问招标信息公示”
  - 招标信息公示工作台卡片改为绑定新权限
  - 前端 `/apps/okcis-notices` 路由、应用中心招标信息区改为校验新权限
  - 后端 `/api/okcis/notices`、关注、删除接口均改为校验新权限
  - 默认给 `after_sales_dept_admin` 授予新权限，保持原可见范围不变
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/update_okcis_notices_permission.sql`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/permissions.py app/db_init.py app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 161. 人事考勤助手新增提醒请假 tab
- 文件：
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/sql/add_wecom_attendance_missing_punch_fields.sql`
  - `backend/migrations/versions/e5f6a7b8c9d0_add_wecom_attendance_missing_punch_fields.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
- 调整：
  - 新增“提醒请假”tab
  - 新增日汇总字段：`absenteeism_count` 旷工次数、`missing_punch_count` 未打卡次数
  - 只采集企微上下班缺卡数据：上班+下班都缺卡记旷工 1 次；上班或下班缺卡记未打卡 1 次
  - 前端新增规则说明、汇总卡和人员明细列表，列表只显示存在旷工 / 未打卡的员工，复用当前日期 / 部门 / 人员筛选
  - 页面默认时间范围、重置范围、企微 / 钉钉同步范围均为“上个月 25 日至今天”
  - 企微同步后端支持超过 30 天自动拆分为多段请求后合并保存；钉钉仍使用原有后端分片
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_wecom_attendance_missing_punch_fields.sql`
  - test / prod 均已补字段
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py migrations/versions/e5f6a7b8c9d0_add_wecom_attendance_missing_punch_fields.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 160. 新增钉钉考勤同步与人员管理
- 文件：
  - `backend/app/core/config.py`
  - `backend/config/env_test`
  - `backend/config/env_prod`
  - `backend/app/models/dingtalk_attendance.py`
  - `backend/app/services/dingtalk_attendance.py`
  - `backend/app/api/endpoints/dingtalk_attendance.py`
  - `backend/app/api/api.py`
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/sql/create_dingtalk_attendance_records.sql`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/migrations/versions/b7c8d9e0f1a2_add_dingtalk_attendance_tables.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
- 调整：
  - 新增钉钉考勤配置，已同步写入 test / prod env：
    - 应用 ID、AgentId、AppKey、AppSecret
    - `DINGTALK_ATTENDANCE_DEPARTMENT_NAME=研发中心`
    - `DINGTALK_ATTENDANCE_EXCLUDED_NAMES=顾江源,张强,王慧君,黄树才,张波,王满川`
  - 新增表 `dingtalk_attendance_users`，用于钉钉考勤人员管理，包含手机号字段、启用状态和禁用原因
  - 新增表 `dingtalk_attendance_records`，用于保存钉钉考勤打卡原始记录
  - 新增权限 `app:dingtalk_attendance:access`
  - 新增接口：
    - `POST /api/dingtalk-attendance/sync-users`
    - `GET /api/dingtalk-attendance/users`
    - `PATCH /api/dingtalk-attendance/users/{user_id}`
    - `POST /api/dingtalk-attendance/sync-records`
    - `GET /api/dingtalk-attendance/records`
  - 钉钉人员同步只取“研发中心”及子部门；配置排除名单默认禁用，并支持页面手动启用 / 禁用
  - “人事考勤助手”新增 tab：`钉钉考勤人员管理`
  - 钉钉人员同步时自动匹配企微 `userID`：
    - 优先按手机号匹配 `users.mobile -> users.sso_uid`
    - 手机号匹配不到时按唯一姓名兜底
    - test / prod 两套环境均已匹配 `38` 人
  - 钉钉人员管理列表增加“企微用户ID”，隐藏手机号；搜索条件不再包含手机号
  - `dingtalk_attendance_records` 也新增 `wecom_userid` 并回填，后续人员对比统一使用企微 userID
  - 钉钉启用人员的打卡数据已支持合并到企微日汇总：
    - `wecom_attendance_daily_records` 新增 `dingtalk_record_count`、`dingtalk_attendance_json`
    - `POST /api/dingtalk-attendance/sync-records` 保存钉钉打卡后自动按企微 `userID + 日期` 合并
    - 前端“同步考勤”会先同步企微，再同步并合并钉钉
    - 合并口径：有钉钉打卡记录即计入实际出勤；迟到/早退分钟分段暂不从钉钉推断
  - 企微考勤汇总页调整：
    - 搜索 / 重置按钮移到筛选同一行
    - 搜索改为后端按部门、姓名/企微ID过滤，修复列表搜索不生效
    - 搜索口径修正为“先匹配人员，再返回该人员日期范围内全部日记录”，保证每个人仍是一行汇总
    - 列表和统计卡隐藏“应出勤”，应出勤后续等表格上传赋值
    - 主列表隐藏外出、考勤修正、20分钟内、20-40分钟、40-60分钟
    - 详情弹窗继续展示这些字段
  - 工作台新增“钉钉考勤助手”卡片，路径 `/apps/wecom-attendance?tab=dingtalk`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_dingtalk_attendance_records.sql`
  - test / prod 均已建表、插入权限与工作台卡片
  - test / prod 均已同步钉钉人员：`total=38`、配置禁用 `4` 人
- 注意：
  - 钉钉手机号权限开通后已重新同步，test / prod 两套环境 `mobile_count=38`，手机号已写入 `dingtalk_attendance_users.mobile`
  - `张强`、`黄树才` 当前不在钉钉“研发中心”返回范围内，因此未落到禁用结果里
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/api/api.py app/permissions.py app/db_init.py app/models/__init__.py app/models/dingtalk_attendance.py app/services/dingtalk_attendance.py app/api/endpoints/dingtalk_attendance.py migrations/versions/b7c8d9e0f1a2_add_dingtalk_attendance_tables.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 141. 修复人事考勤助手近 30 天同步与实际出勤口径
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 企微考勤汇总列表去掉“天数”列
  - 实际出勤统计改为扣减事假、病假，统计卡、人员汇总和详情弹窗保持同一口径
  - 页面点击“同步考勤”默认同步最近 30 天，并将页面日期切换到该区间
  - 后端同步接口未传日期时默认最近 30 天
  - 企微同步覆盖历史数据时，已有钉钉记录的日汇总不再被企微实际出勤覆盖掉
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 142. 放大人事考勤助手详情弹窗
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 详情弹窗宽度放大到 `1400px / 94vw`
  - 覆盖公共 Dialog 默认 `sm:max-w-lg` 限制
  - 详情表格最小宽度调到 `1320px`，字段可横向完整查看
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 143. 人事考勤助手新增迟到统计卡
- 文件：
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 病假统计卡后新增“迟到”卡片
  - 合并统计 `20分钟内 + 20-40分钟 + 40-60分钟`
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 162. 人事考勤助手新增花名册模板填充
- 文件：
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/static/花名册模板.xlsx`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增花名册模板下载接口：`GET /api/wecom-attendance/roster-template`
  - 新增花名册上传填充接口：`POST /api/wecom-attendance/roster/fill`
  - 前端“企微考勤汇总”tab 新增“下载模板”和“上传花名册”按钮
  - 上传 `.xlsx` 后，按当前页面日期范围查询考勤汇总并回填模板，再下载生成后的 Excel
  - 匹配规则为模板 `部门 + 姓名` 对应考勤日汇总表 `department_name + employee_name`
  - 模板部门空白行会自动继承上一行部门
  - 填充字段包括：实际出勤、事假、病假、迟到/早退20分钟内、20-40分钟、40-60分钟
  - 实际出勤继续按 `实际出勤 - 事假 - 病假` 口径汇总
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 163. 人事考勤助手支持最新花名册请假类型
- 文件：
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/sql/add_wecom_attendance_leave_type_fields.sql`
  - `backend/migrations/versions/d4e5f6a7b8c9_add_wecom_attendance_leave_type_fields.py`
  - `backend/static/花名册模板.xlsx`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 用最新 `花名册模板.xlsx` 覆盖项目内模板
  - 右上角“同步考勤”改为“同步企微考勤”
  - 花名册导出按最新列回填：年假、带薪病假、调休、事假、产假、陪产假、婚假、丧假、哺乳假、迟到/早退三档
  - 数据库新增对应请假类型字段：`annual_leave`、`paid_sick_leave`、`compensatory_leave`、`maternity_leave`、`paternity_leave`、`marriage_leave`、`bereavement_leave`、`breastfeeding_leave`
  - `personal_leave`、`sick_leave` 改为 `DECIMAL(6,1)`，支持半天
  - OA 休假同步按 `newLeaveType` 和请假事由拆分请假类型；半天按 `0.5` 写入
  - 实际出勤口径按用户确认改为只扣 `事假 + 带薪病假`，其他请假类型只记录和展示
  - 详情页新增全部请假类型列
  - 花名册匹配增强：姓名去掉 `VIP数字` 后匹配，`部门 + 姓名` 优先，匹配不到按唯一姓名兜底
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_wecom_attendance_leave_type_fields.sql`
  - test / prod 均执行成功，各执行 `41` 条 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py migrations/versions/d4e5f6a7b8c9_add_wecom_attendance_leave_type_fields.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
  - 已用测试库 2026-07-01 至 2026-07-13 样例验证：张桐 / 王菲 / 张国君实际出勤 `12`、事假 `1.0`，不再全 0
- 本次未新增依赖

### 161. 修复客户无忧长跟进内容导致写入失败
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 原因：
  - 增量跟进记录 `source_dedupe_key` 原先拼接整段 JSON
  - 长跟进内容会导致去重键超过 `crawler_kehu51_follow_records.source_dedupe_key VARCHAR(255)`
  - 用户给出的“煜兴（天津）建设科技有限公司(杨女士）”记录包含长跟进内容，因此触发写入失败
- 调整：
  - `_build_dedupe_key` 改为对规范化字段 JSON 生成 SHA1
  - 新格式：`{task_key}|page:{page_no}|{sha1}`
  - 避免长内容撑爆字段，同时保留同字段组合去重能力
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/services/crawler_handlers/kehu51.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 157. 修复业务团队看板推送周一不触发
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 正式环境 `after_sales_business_dashboard_push` 在 2026-07-13 08:30 没有 cron 执行记录
  - 2026-07-13 10:50 手动执行成功，说明推送本身正常
  - 根因是配置 `30 8 * * 1-5` 按标准 cron 表示周一到周五，但 APScheduler 数字周几口径是 `0=周一`，旧代码将其解释为周二到周六
- 调整：
  - test / prod 两套环境该任务 `cron_expr` 已改为无歧义写法：`30 8 * * mon-fri`
  - `_build_cron_trigger` 已增加标准 cron 数字周几转换，后续 `1-5` 也会按周一到周五解析
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
  - `_build_cron_trigger('30 8 * * 1-5')` 下一次触发可正确命中周一 08:30
- 本次未新增依赖

### 158. 校验企微考勤同步和外出字段
- 排查：
  - 本地后端实际读取 `backend/.env`，`WECOM_CHECKIN_AGENT_ID=3010011`，考勤秘钥已可读取
  - 2026-07-13 查询企微成功：员工 `274` 人，原始打卡 `226` 条，日汇总 `274` 条
  - 2026-07-13 外出类型单独查询为 `0` 条，因此当天页面外出为 `0` 属于源站数据结果
  - 2026-06-13 至 2026-07-13 外出类型单独查询为 `23` 条，说明企微接口可以返回外出打卡
  - 2026-07-09 使用 `opencheckindatatype=3` 查询也包含外出打卡：顾江源、李兆民各 `1` 条
- 调整：
  - `out_attendance` 从“有外出则置 1”改为“外出次数累加”
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - 2026-07-09 样本汇总：`[('顾江源VIP4', 'GuJiangYuan', 1), ('李兆民VIP5', 'LiZhaoMin', 1)]`
- 本次未新增依赖

### 159. 人事考勤助手列表汇总与 OA 休假/考勤修正同步
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/models/wecom_attendance.py`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/migrations/versions/a6b7c8d9e0f1_add_wecom_attendance_daily_records.py`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 人事考勤助手列表新增“搜索”和“重置”
  - 日期、部门下拉、姓名/企微ID 输入仅修改筛选条件，点击搜索才触发查询与过滤
  - 主表按人员汇总，每个人只显示一行
  - 新增操作列“详情”，弹窗展示该人员每天考勤数据
  - 接入 OA 休假流程表单 `formtable_main_16`
  - 接入 OA 考勤修正流程表单 `formtable_main_21`
  - 仅同步已完成流程：`workflow_requestbase.CURRENTNODETYPE = '3'`
  - 休假写入事假 / 病假标记并保留 `oa_leave_json`
  - 考勤修正写入 `attendance_correction_count`，并将当天 `actual_attendance` 置为 `1`，保留 `oa_correction_json`
  - `out_attendance` 已继续按外出次数统计
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_wecom_attendance_daily_records.sql`
  - test / prod 均执行成功，各执行 `19` 条 SQL，已补字段：`attendance_correction_count`、`oa_leave_json`、`oa_correction_json`
- 已验证：
  - 2026-07-13 样本：休假匹配 `4` 人
  - 2026-07-11 样本：休假 / 考勤修正匹配 `5` 人，其中李超越 `attendance_correction_count=1`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py migrations/versions/a6b7c8d9e0f1_add_wecom_attendance_daily_records.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 156. 修正售后工单实时监控步骤进度和语音提醒
- 文件：
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 页面步骤展示已精简为固定 6 个：开始、派单、接单、出发、到场、处理回复
  - 根据接口步骤最后一步的 `nodeId` 匹配上述 6 个节点，匹配到的节点显示蓝色，其它节点显示灰色
  - 语音提醒改为：轮询刷入新工单，且该新工单步骤最后一步 `nodeId=Accept` 时触发
  - 符合条件时同一条语音提示连续播放 `3` 次
  - 初次加载和旧工单状态轮询不播报
  - 如果本次刷新返回的新列表条数少于旧列表，直接用新列表覆盖为基线，不高亮、不播报，下一轮再继续对比
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 155. 修复售后客户无忧导出预览王贺 sheet 格式错乱
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 原因：
  - 王贺实际只有 A:H 可见列，但导出缓存中 I:IR 隐藏列残留模板旧内容
  - 前端预览裁剪时未跳过隐藏列，导致隐藏旧内容被渲染出来
- 调整：
  - 后端生成 Excel 时清空未使用隐藏列的单元格值
  - 前端导出预览裁剪时跳过隐藏列，并过滤隐藏列合并区域
  - 顺手修复导出缓存生成日志变量 `channel_cache_key` 未定义问题
- 已处理：
  - 正式 Redis 已刷新 `all/online/offline` 三份导出缓存
  - 新 `all` 文件：`售后部客户跟进表_全部_260711120724.xlsx`
  - 抽检 `深耕组-王贺` 隐藏列残留内容为 `0`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 154. 售后客户无忧导出仅保留渠道筛选
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 售后部客户无忧导出 / 预览导出不带分页参数
  - 导出 / 预览导出唯一生效筛选改为“客户渠道”
  - 客户名、跟进人、日期筛选只影响列表，不影响导出
- 本次未新增依赖

### 136. 确认客户无忧固定人员爬取口径并修正一次性补录脚本
- 文件：
  - `backend/scripts/refetch_kehu51_source_ids_once.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 明确客户列表和客户跟进都继续使用原有固定业务人员列表执行全量 / 增量，不改为全员爬取
  - 本次只补源站 `客户ID`、`联系人ID` 解析、落库和关联，不改变原业务爬取流程
  - 一次性补录脚本已去掉客户列表特殊 `whereSql/completeSql` 覆盖，避免绕过已跑通的固定人员流程
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/refetch_kehu51_source_ids_once.py crawler_sites/kehu51/formatter.py app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py app/services/kehu51_follow_cache.py`
- 本次未新增依赖

### 137. 优化售后部客户无忧导出预览慢查询
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/services/kehu51_follow_cache.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 原关联客户列表方案里，客户列表表曾被优化器选为全表扫
  - 用户确认导出预览不需要关联客户列表，只看客户跟进表
- 调整：
  - 导出预览改为只查 `crawler_kehu51_follow_records` 单表，不再关联 `crawler_kehu51_customer_list`
  - 数据范围为固定业务人员 2026 年开始的跟进数据
  - 客户分组按跟进表原始 `source_customer_id` 分组，客户名仅用于展示；缺少源站客户ID的历史数据按客户名兜底
  - 季度归属按该跟进人该客户 2026 年最早跟进时间判断
  - 新单表 SQL 已查执行计划：test / prod 均命中 `idx_follow_export_creator_time_channel_customer`，约 4200 行 range 扫描
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/services/kehu51_follow_cache.py`
- 本次未新增依赖

### 138. 售后客户无忧导出预览增加进度与三分钟缓存
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 导出接口增加 Redis 文件级缓存，按年份 + 渠道缓存生成后的 Excel，TTL 为 `180` 秒
  - 导出预览和下载共用同一接口，3 分钟内重复操作直接返回缓存文件，不再重复查数据库或生成 Excel
  - 前端导出预览 loading 增加百分比进度和进度条，按钮文案同步显示进度
  - 后端接口完成后前端进度推进到 `100%`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 139. 售后客户跟进统计导出改为汇总表读取
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/services/kehu51_follow_export_summary.py`
  - `backend/app/tasks/scheduler.py`
  - `backend/sql/create_after_sales_customer_follow_export_summary.sql`
  - `backend/migrations/versions/o5p6q7r8s9t0_create_after_sales_customer_follow_export_summary.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增表 `after_sales_customer_follow_export_summary`
  - 字段包含客户无忧用户ID、跟进人、源站客户ID、客户分组Key、客户名、渠道、今年首次跟进时间、所属季度、跟进条数、跟进内容集合JSON
  - 导出预览和下载改为优先读取汇总表，不再每次现场统计跟进明细
  - 汇总表为空时，导出接口会即时同步一次再生成 Excel
  - 新增计划任务 `after_sales_customer_follow_export_summary_sync`
  - 计划任务名称：`售后客户跟进统计导出表同步`
  - 频率：每小时 10 分执行，cron 为 `10 * * * *`
  - 每次任务执行后自动刷新导出预览 Redis 文件缓存
  - 缓存渠道：`all`、`online`、`offline`
  - 缓存有效期调整为 `4200` 秒（70 分钟），保证每小时同步间隔内预览 / 下载优先命中缓存
  - 跟进内容去重规则：同一个用户、同一个客户、同一天、相同跟进内容只展示一条
  - 同步汇总写入 `follow_items_json` 前先去重，导出展示层也会再次去重
  - 汇总表补充导出专用索引：
    - `idx_after_sales_follow_summary_export_order (summary_year, owner_name, quarter_no, first_follow_time, id)`
    - `idx_after_sales_follow_summary_channel_export_order (summary_year, customer_channel, owner_name, quarter_no, first_follow_time, id)`
  - 导出查询按是否筛选渠道显式 `FORCE INDEX`，避免 prod 优化器选择全表扫
  - 客户分组按原始 `source_customer_id`，缺少客户ID但同一跟进人 + 同客户名只有一个已知客户ID时自动归并到该客户ID
  - 截图中的 `路劲物业` 经查源站为两个不同客户ID：`84447140`、`84447345`，因此按客户ID分组会展示为两个客户列
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_after_sales_customer_follow_export_summary.sql`
  - test / prod 均已建表并写入计划任务元数据
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_after_sales_customer_follow_export_summary_indexes.sql`
  - test / prod 均已补齐导出专用组合索引
  - 已手动同步 2026 年汇总：
    - test：`owners=10`、`summaries=1006`
    - prod：`owners=10`、`summaries=1012`
  - 已用正式库验证同步 + 刷新缓存成功：`sync summaries=1012`，`channels=['all', 'online', 'offline']`，`ttl_seconds=4200`
  - 已按去重规则重新同步并刷新 test / prod 两套环境缓存：
    - test：`summaries=1006`，缓存 `all/online/offline`
    - prod：`summaries=1012`，缓存 `all/online/offline`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/tasks/scheduler.py app/services/kehu51_follow_export_summary.py migrations/versions/o5p6q7r8s9t0_create_after_sales_customer_follow_export_summary.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/o5p6q7r8s9t0_create_after_sales_customer_follow_export_summary.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 153. 新增售后工单监控卡片与公开实时监控页
- 文件：
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/app/models/after_sales_dept.py`
  - `backend/app/models/__init__.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/migrations/versions/j0k1l2m3n4o5_add_after_sales_work_order_monitor_accounts.py`
  - `backend/sql/create_after_sales_work_order_monitor.sql`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `frontend/src/App.tsx`
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增售后服务卡片“售后工单监控”
  - 新增权限 `app:after_sales_work_order_monitor:access`
  - 新增账户配置表 `after_sales_work_order_monitor_accounts`
  - 新增受保护管理入口 `/apps/after-sales-work-order-monitor`
  - 新增公开实时监控入口 `/apps/after-sales-work-order-monitor/live`，不需要登录 / 权限
  - 新增账户管理接口：`GET/POST/PUT/DELETE /api/after-sales-dept/work-order-monitor/accounts`
  - 新增实时工单代理接口：`/api/after-sales-dept/work-order-monitor/worksheets`
  - 新增公开实时接口：`/api/after-sales-dept/work-order-monitor/public/accounts`、`/public/worksheets`
  - 实时工单外部接口固定参数 `from=Dispatch&to=Dispose&sort=ID&ascending=false&count=10&since=1`，仅 `user` 使用账户 ID
  - 前端实时监控按账户自动分屏，15 秒轮询，支持语音通知开关
  - 账户管理“新增”补必填提示，避免空表单点击无反馈
  - 根据历史接口字段展示基本信息：`id/typeName/title/content/address/status/createTime/deadline/remark`，不展示 `steps`
- 已同步：
  - test / prod 已执行 `backend/sql/create_after_sales_work_order_monitor.sql`
  - 两套环境均已建表、插入权限、给 `after_sales_dept_admin` 授权、插入卡片
- 已验证：
  - 外部历史接口 `user=3140` 返回 `200`，字段结构已确认
  - 外部实时接口 `user=3343` 返回 `200`，`{"code":0,"message":null,"data":[]}`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/permissions.py app/db_init.py app/models/after_sales_dept.py app/models/__init__.py app/schemas/after_sales_dept.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/j0k1l2m3n4o5_add_after_sales_work_order_monitor_accounts.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 152. 客户无忧客户列表入库按业务字段排重并清洗历史重复
- 文件：
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 客户列表主 handler 入库排重从 `客户名称 + 联系人 + 手机号 + 录入时间 + 非当前 run` 改为 `负责人 + 客户名称 + 联系人 + 手机号`
  - 客户列表 `source_dedupe_key` 改为业务字段稳定 SHA1 哈希，不再使用页码 / item 位置
  - 后台旧兜底客户列表写入逻辑同步使用同一业务排重规则
  - 客户名称写入旧兜底逻辑同步走 kehu51 客户名规范化
- 数据清理：
  - test：清理前 `87` 组重复、删除 `105` 条，清理后重复组 `0`
  - prod：清理前 `90` 组重复、删除 `110` 条，清理后重复组 `0`
  - prod “缴志健 / 赞皇县城西地表水厂”只剩 `id=11976`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/kehu51_customer_list.py app/api/endpoints/admin/crawler_tasks.py app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 151. 客户无忧导出去重同一负责人同名客户列
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 正式库 `crawler_kehu51_follow_records` 中“缴志健 / 赞皇县城西地表水厂 / 2026-07-09 / 新接的水厂项目持续跟进客户需求”只有 `1` 条
  - 正式库 `crawler_kehu51_customer_list` 中同一负责人、同一客户、同一联系人、同一手机号有 `2` 条未删除客户记录：
    - `id=11976`，`created_time=2026-07-09 20:18:00`
    - `id=11975`，`created_time=2026-07-09 20:19:02`
  - 导出按客户列表生成客户列，因此同名客户列重复，导致同一条跟进在预览里出现两次
- 调整：
  - 导出客户列表 SQL 改为按 `owner_name + customer_name` 分组
  - 取 `MIN(created_time)` 用于季度归属和排序，避免同一负责人同名客户重复生成列
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 150. 客户无忧导出自动扩展单列跟进记录行数
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 导出预览中“华电水务栾城有限公司 / 开拓组-范运成”缺少 `2026-07-09 22:43:00` 跟进记录
  - 原因是导出按模板现有数据行数设置 `max_follow_rows`，写入时使用 `display_rows[:max_follow_rows]`，单个客户记录超过模板行数会被截断
- 调整：
  - 每个 sheet 写入前，按该人员所有客户的最长跟进记录计算所需行数
  - 当所需行数超过模板行数时，自动追加行并复制模板最后一行样式
  - 后续单列客户跟进记录不再被模板行数截断
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 149. 客户无忧导出下载按渠道保持一致
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 售后部（客户无忧）导出文件名追加渠道中文标识，如 `售后部客户跟进表_线上_260710084221.xlsx`
  - 预览弹窗内“下载”改为下载当前预览生成的同一个文件，避免预览内容和下载内容筛选不一致
  - 渠道筛选仍按当前已应用筛选条件传给导出接口
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 147. 客户无忧客户与跟进增加线上线下标识
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `backend/sql/alter_kehu51_add_customer_channel.sql`
  - `backend/migrations/versions/a0b1c2d3e4f6_add_kehu51_customer_channel.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `crawler_kehu51_customer_list` 新增 `customer_channel`
  - `crawler_kehu51_follow_records` 新增 `customer_channel`
  - 字段值：`online` / `offline`
  - 客户列表入库：`creator_name = '李旭晴'` 标记线上，否则线下
  - 客户跟进入库：按 `customer_name + contact_name` 查询客户列表，匹配到线上客户则跟进标记线上
  - 已保持客户列表抓取人员范围不变，不再追加李旭晴作为额外抓取目标
  - 旧后台兜底写入逻辑和新 handler 均已同步
- 已执行：
  - test / prod 两套环境补字段、索引并回填历史数据
  - test 客户列表：`online=917`、`offline=6022`
  - test 跟进记录：`online=4386`、`offline=37539`
  - prod 客户列表：`online=921`、`offline=6078`
  - prod 跟进记录：`online=4396`、`offline=37617`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py app/api/endpoints/admin/crawler_tasks.py migrations/versions/a0b1c2d3e4f6_add_kehu51_customer_channel.py`
- 本次未新增依赖

### 148. 售后部客户无忧增加客户渠道筛选
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 售后部（客户无忧）客户跟进列表新增“客户渠道”筛选，默认全部
  - 支持按 `online` 线上客户、`offline` 线下客户筛选
  - 历史空渠道数据按线下客户处理，避免线下筛选漏数据
  - 列表新增“客户渠道”列，显示线上 / 线下
  - 导出与导出预览同步使用当前筛选条件，包含客户渠道
  - 后端客户跟进列表、客户候选、导出接口均支持 `customer_channel`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 146. 清理客户无忧客户列表重复数据
- 表：
  - `crawler_kehu51_customer_list`
- 操作：
  - 按 `customer_name + contact_name + mobile_phone + created_time` 判断重复客户
  - 保留未删除记录优先，其次保留更新时间最新记录
  - prod：清理前 `237` 组重复、`364` 条需删除；已删除 `364` 条；清理后重复 `0`
  - test：清理前 `2` 组重复、`4` 条需删除；已删除 `4` 条；清理后重复 `0`
  - 用户指定重复记录 `4383 / 11338`：已保留 `4383`，删除 `11338`
- 本次为数据清理，未新增依赖

### 145. 列表状态支持快速切换
- 文件：
  - `frontend/src/pages/admin/SchedulerPage.tsx`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 计划任务列表“状态”列支持点击快速暂停 / 启用
  - 状态按钮增加固定最小宽度和不换行，避免文字换行
  - 工作台卡片列表“启用”列支持点击快速启用 / 停用
  - 两处均增加处理中状态和 toast 提示
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 144. 微调业务团队看板推送图排版
- 文件：
  - `backend/app/services/after_sales_dashboard_image.py`
  - `.gitignore`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - “业绩目标”指标卡标题和值在 JPG / SVG 中同步上移
  - 顶部横幅去掉口号文案，避免标题区域拥挤
  - 重新生成预览：
    - `data/after_sales_business_dashboard_push_preview_20260709.jpg`
    - `data/after_sales_business_dashboard_push_preview_20260709.svg`
  - 预览文件已加入 `.gitignore`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_dashboard_image.py`
- 本次未新增依赖

### 143. 修复角色卡片装饰圆形溢出
- 文件：
  - `frontend/src/pages/admin/RolesPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 角色与权限页面卡片容器从 `overflow-visible` 改为 `overflow-hidden`
  - hover / 选中时右上角浅蓝装饰圆形会裁切在卡片内部，不再露出到相邻卡片上方
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 142. 系统管理首页卡片增加真实数量统计
- 文件：
  - `backend/app/api/endpoints/admin/stats.py`
  - `frontend/src/pages/admin/AdminDeckPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 访问控制卡片改为显示真实角色数
  - 权限定义卡片改为显示“已有 N 个权限”
  - 部门架构、知识库管理、计划任务、爬虫任务、工作台卡片、企微推送管理均补充对应数量
  - `/admin/stats/dashboard` 新增角色、权限、部门、知识库、计划任务、爬虫任务、工作台卡片、企微机器人、企微模板统计字段
  - 统计接口接入 Redis 缓存 24 小时
  - 缓存 key：`admin:dashboard:stats:v1`
  - Redis 不可用时不影响接口返回，只是不走缓存
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/stats.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 141. 修复软件部工作记录默认填写人筛选未生效
- 文件：
  - `frontend/src/pages/software_task/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 问题：
  - 软件部任务工具“工作记录”页面会默认显示当前填写人
  - 但当前用户信息异步加载完成前，列表先按空筛选请求了全部数据
  - 导致筛选框显示了填写人，列表内容仍可能是全部数据
- 调整：
  - 默认填写人尚未应用前，暂停工作记录列表请求
  - 等 `created_by_user_id` 写入实际筛选条件后再加载列表
  - 保留手动清空填写人后查看全部记录的能力
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 140. 用户列表增加批量更改角色
- 文件：
  - `backend/app/schemas/admin.py`
  - `backend/app/api/endpoints/admin/roles.py`
  - `frontend/src/pages/admin/UsersPage.tsx`
- 调整：
  - 后端新增批量角色更新接口：`PUT /api/admin/roles/batch/users/roles`
  - 接口校验用户与角色存在，批量替换选中用户角色
  - 用户列表新增当前页勾选、全选
  - 用户列表顶部新增“批量更改角色”按钮
  - 批量弹窗默认预选所有选中用户共同拥有的角色
  - 保存后刷新当前页并清空选中状态
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/schemas/admin.py app/api/endpoints/admin/roles.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 139. 工作台卡片权限展示与权限下拉数据源修正
- 文件：
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
- 调整：
  - 工作台卡片列表“权限”列改为展示权限描述
  - 权限列 `title` 保留原始权限 slug，鼠标 hover 可查看实际权限内容
  - 编辑弹窗“权限”下拉改为调用 `/admin/permissions` 获取真实权限定义列表
  - 接口异常时保留本地 fallback 权限选项
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 138. 业务团队管理看板独立权限
- 文件：
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/sql/update_after_sales_management_dashboard_kanban_permission.sql`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/App.tsx`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
- 调整：
  - 新增权限 `app:after_sales_dept:kanban`，描述为“业务团队管理看板”
  - 业务团队管理看板前端路由、工作台卡片、后端看板查询 / 编辑 / 保存接口均改为新权限
  - 初始化脚本已给 `after_sales_dept_admin` 角色补该权限
  - 工作台卡片管理权限下拉选项补充“业务团队管理看板”
  - 新增 SQL 文件同步现有环境数据
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/update_after_sales_management_dashboard_kanban_permission.sql`
  - test / prod 均确认：权限存在、售后管理员角色已绑定、`dashboard_app_cards.after_sales_management_dashboard` 已改为新权限
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/permissions.py app/db_init.py app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 137. 修复公共下拉组件点击滚动条自动关闭
- 文件：
  - `frontend/src/components/ui/app-select.tsx`
  - `frontend/src/components/ui/app-multi-select.tsx`
- 调整：
  - 公共单选 / 多选下拉统一关闭滚动触发的菜单收起
  - 设置 `closeMenuOnScroll={false}`、`captureMenuScroll={false}`、`menuShouldScrollIntoView={false}`
  - 公共组件自动识别是否处于 `DialogContent` 内：
    - 弹窗内不再 portal 到 `document.body`，改用弹窗内部绝对定位，避免选项点击被 Radix Dialog 拦截，也避免位置跑偏
    - 页面内仍使用 `document.body` portal 和 fixed 定位
  - 公共组件新增 `menuZIndex` 配置，默认仍为 `9999`
  - 公共多选组件保留 `blockInteractionsWhenOpen` 可选项
  - 解决该弹窗里权限下拉菜单被后续图标、颜色、描述等内容遮住的问题
  - 解决弹窗内多选下拉点击滚动条时菜单自动消失的问题
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 136. 修复业务团队看板推送图排版错位
- 文件：
  - `backend/app/services/after_sales_dashboard_image.py`
- 调整：
  - 修复项目内 Noto 字体启用后，推送 JPG 顶部标题与副标题重叠的问题
  - 同步调整 SVG 预览源的头部坐标
  - 指标卡高度加高，数值上移，避免数值贴底或裁切
  - 已重新生成预览图：`data/after_sales_business_dashboard_push_test_20260709.jpg`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_dashboard_image.py`
- 本次未新增依赖

### 135. 计划任务列表增加业务部门字段
- 文件：
  - `backend/app/models/crawler_task.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/migrations/versions/i0j1k2l3m4n5_add_scheduler_business_department.py`
  - `frontend/src/pages/admin/SchedulerPage.tsx`
  - `frontend/src/pages/admin/scheduler-utils.ts`
  - `frontend/src/components/ui/app-select.tsx`
- 调整：
  - `scheduler_job_meta` 新增 `business_department` 字段，默认 `软件部门`
  - test / prod 两套环境已补字段并回填空值
  - 内置任务按部门补默认归属：售后服务部、采购部、平台管理、软件部门
  - 计划任务列表新增“业务部门”列
  - 计划任务列表新增“业务部门”筛选，下拉选项来自当前任务已有部门
  - 编辑计划任务弹窗新增“业务部门”组织架构下拉，来源 `/admin/departments`
  - 修复 `AppSelect` 在弹窗中 `menuPortalTarget={null}` 不生效的问题，避免弹窗内下拉点击异常
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/crawler_task.py app/api/endpoints/admin/scheduler.py migrations/versions/i0j1k2l3m4n5_add_scheduler_business_department.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 134. 新增晨天润达业务团队看板推送模板与计划任务
- 文件：
  - `backend/app/services/after_sales_dashboard_image.py`
  - `backend/app/services/wecom.py`
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/sql/upsert_after_sales_business_dashboard_push_template.sql`
- 调整：
  - 新增业务团队管理看板长图生成服务，按实时看板数据生成竖版青色看板图片
  - 企业微信群机器人新增图片消息发送方法，按企微机器人 `image` 消息要求发送 `base64 + md5`
  - 企微机器人 image 接口不支持 SVG 原始字节，实际推送改为 JPG；SVG 继续保留为预览源文件
  - 企业微信群机器人图片发送已增加 `errcode=-1` 临时失败重试
  - 新增计划任务 `after_sales_business_dashboard_push`
  - 默认每天 `08:30` 执行
  - 后台计划任务列表已支持展示与立即执行
  - 后续补充：计划任务列表接口已增加固定任务兜底展示，解决后端未重启或 scheduler 尚未注册时，`scheduler_job_meta` 中已有配置但列表看不到的问题
  - 任务按模板 key `after_sales_business_dashboard_push` 查询启用模板，再使用后台绑定该模板的启用机器人，不在代码内固定机器人
  - 推送方式已调整为只发一条 image，不再单独发 markdown 标题
  - 图片视觉按网页看板青色主题生成，替换临时黑白图
  - 推送图左上角已移除不合适的“耳机”图标块，标题改为左侧直接展示
  - 推送 SVG 已去掉“最近跟进”模块
  - 推送图“小组战况 / 个人排名”表格表头与内容已统一列坐标：首列左对齐，其余列居中对齐，避免标题和内容错位
  - 测试库已新增模板 `晨天润达业务团队看板推送`，标题模板 `{{push_date}}-晨天润达业务团队看板`
  - test / prod 两套环境均已把机器人 `id=2`（售后AI群发）绑定到该模板
  - 企微机器人与消息模板已升级为多对多：
    - 新增表 `wecom_message_robot_templates`
    - 保留 `wecom_message_robots.template_id` 作为兼容字段
    - 后端机器人列表 / 保存接口支持 `template_ids`
    - 计划任务优先按新关联表查找绑定机器人，旧字段兜底
    - 前端“推送机器人管理”关联模板改为多选
    - 前端“模板管理”绑定机器人改为多选
    - 修复公共多选 `AppMultiSelect` 在弹窗内点击下拉条目不生效：弹窗内显式禁用 portal，避免 Dialog 拦截点击
  - test / prod 两套环境均已创建关联表并迁移旧绑定数据
  - 已生成测试预览：
    - `data/after_sales_business_dashboard_push_test_20260709.svg`
    - `data/after_sales_business_dashboard_push_test_20260709.jpg`，约 `386KB`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_dashboard_image.py app/services/wecom.py app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/crawler_task.py app/api/endpoints/admin/wecom_management.py app/tasks/scheduler.py migrations/versions/h9i0j1k2l3m4_add_wecom_robot_template_links.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/upsert_after_sales_business_dashboard_push_template.sql`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
  - test / prod 查询确认模板绑定机器人：`template_id=11`、`robot_id=2`、机器人标题 `售后AI群发`
- 本次未新增依赖，使用已有 `Pillow`

### 133. 其他非业务人员排除米鸿旭、赵成宇
- 文件：
  - `backend/app/services/after_sales_team.py`
- 调整：
  - `其他非业务人员` 排除名单新增：
    - 米鸿旭
    - 赵成宇
  - 看板人员缓存 key 从 `after_sales:dashboard_team_members:v3` 升级为 `after_sales:dashboard_team_members:v4`
- 正式环境复核：
  - 当前 `其他非业务人员` 为 `63` 人
  - 米鸿旭、赵成宇、王贺均不在该组
  - 2026 年合同 `34` 个，合计 `58.78 万`
  - 有合同金额人员：张涛 `0.14 万`、杨彬 `51.98 万`、王利伟 `0.08 万`、王满川 `0.00 万`、闫智华 `6.58 万`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_team.py app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 132. 正式环境“其他非业务人员”合同金额排查
- 范围：
  - 业务团队管理看板
  - 虚拟组：`其他非业务人员`
- 结果：
  - 当前非业务人员共 `65` 人
  - 2026 年按看板口径汇总合同 `40` 个，合计 `68.20 万`
  - 有合同金额人员：
    - 张涛 `0.14 万`
    - 杨彬 `51.98 万`
    - 王利伟 `0.08 万`
    - 王满川 `0.00 万`
    - 米鸿旭 `9.12 万`
    - 赵成宇 `0.30 万`
    - 闫智华 `6.58 万`
- 备注：
  - 当前看板合同金额按 `salesperson_name` 匹配人员别名
  - 未额外限制合同表 `department_name`
  - 若人工口径只看特定部门，可能会与当前看板金额不一致

### 131. 业务团队管理看板“其他非业务人员”展示口径收口
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
- 调整：
  - 小组战况增加 `其他非业务人员` 行
  - 该行只展示 `年度完成`，其它列统一显示 `-`
  - 个人排名保留 `其他非业务人员` 单行，固定排最后，不展示奖杯 / 奖牌图标
  - `其他非业务人员` 不再出现在其它个人统计图表：
    - 个人全年完成率
    - 个人季度完成率
    - 个人拜访量
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 130. 业务团队管理看板新增“其他非业务人员”虚拟组
- 文件：
  - `backend/app/services/after_sales_team.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 调整：
  - 看板人员口径新增虚拟组：`其他非业务人员`
  - 来源：
    - `张涛`
    - OA `售后服务部(技术组)` 及其子部门
  - 排除：`王贺`
  - 显示名继续统一去掉 `VIP*`
  - 新增看板专用人员缓存：`after_sales:dashboard_team_members:v3`
  - 客户无忧 / kehu51 客户列表同步仍使用原业务组人员口径，不包含这批“其他非业务人员”
  - 看板客户数 / 跟进数仍只按客户无忧业务组人员统计，避免旧客户表中技术组历史数据误入小组战况
  - 小组战况中新增人员统一归到 `其他非业务人员`
- 已执行：
  - 主动补齐 test / prod 当前年度缺失看板人员
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_team.py app/api/endpoints/after_sales_dept/after_sales_dept.py app/services/kehu51_employees.py`
  - 当前看板人员 `75` 人，原业务组 `10` 人，其他非业务人员 `65` 人
  - `张涛` 已包含，`王贺` 未进入其他非业务人员
  - kehu51 同步人员仍为 `10` 人，且不包含 `张涛`
  - 小组战况分组：`深耕组 / 开拓组 / 其他非业务人员`
- 本次未新增依赖

### 129. 业务团队管理看板隐藏客户回访和应酬次数
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
- 调整：
  - 个人排名表格隐藏“客户回访 / 应酬次数”
  - 编辑业务团队管理看板弹窗隐藏“客户回访次数 / 应酬次数”
  - 后端字段保留，避免影响历史数据和接口兼容
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 128. 软件部日报提醒误报排查与修复
- 文件：
  - `backend/app/tasks/scheduler.py`
- 排查：
  - 正式环境 `wecom_message_logs.id=63` 显示 2026-07-08 日报提醒误包含：
    - 于涛
    - 康鹏
    - 王业龙
    - 王帅
  - 正式库确认 `王业龙` 已有 2026-07-07 日报：
    - `software_work_records.id=471`
    - `created_by=62`
    - `created_at=2026-07-08 08:27:38`
  - 正式库确认 `康鹏` 已有 2026-07-07 日报：
    - `software_work_records.id=465`
    - `created_by=88`
  - `康鹏` 误报与重复账号有关，已在上一条通过 `sso_uid_aliases` 与重复账号清理解决
- 调整：
  - 日报提醒不再只按 `SoftwareWorkRecord.created_by == User.id` 单点判断
  - 改为先取候选人员，再取目标日期已填写人员
  - 按 `用户ID + 昵称 + 邮箱 + 手机号 + sso_uid` 综合匹配是否已填写
  - 避免同一员工因重复账号或企微账号漂移被误报
  - 推送时写入 `business_module='software_work_record_reminder'`
  - 任务日志新增目标日期和缺失名单，方便后续追溯
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
  - 正式库按新逻辑复算 `2026-07-07` 缺失名单为：`于涛、王帅`
- 本次未新增依赖

### 127. 企微登录新增别名账号字段，彻底兜住康鹏重复建号
- 文件：
  - `backend/app/api/endpoints/auth.py`
  - `backend/app/models/base.py`
  - `backend/migrations/versions/g8h9i0j1k2l3_add_user_sso_uid_aliases.py`
  - `backend/sql/alter_users_add_sso_uid_aliases.sql`
  - `backend/scripts/merge_duplicate_users.py`
- 调整：
  - `users` 表新增 `sso_uid_aliases`
  - 企微登录匹配逻辑由仅查 `sso_uid` 改为同时查 `sso_uid + sso_uid_aliases`
  - 命中已有账号后，不再反复覆盖主 `sso_uid`
  - 改为把新的企微 userid 追加到 `sso_uid_aliases`
  - 正式环境康鹏重复账号 `id=310` 已归并删除
  - test / prod `康鹏 id=88` 已统一回填：
    - `sso_uid_aliases=',kangpeng,tangpeng,'`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/alter_users_add_sso_uid_aliases.sql`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/merge_duplicate_users.py --env prod`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql "UPDATE users SET sso_uid_aliases=',kangpeng,tangpeng,' WHERE id=88;"`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/auth.py app/models/base.py scripts/merge_duplicate_users.py migrations/versions/g8h9i0j1k2l3_add_user_sso_uid_aliases.py`
  - 当前 test / prod 查询 `nickname='康鹏'` 均只剩 1 条
- 本次未新增依赖

### 126. 业务团队管理看板完成额改为 OA 合同金额
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 调整：
  - 看板“个人排名 / 完成额”不再读取配置表里的 `completed_amount`
  - 改为读取 `oa_contract_payment_report_rows` 合同同步数据
  - 年度完成额：统计所选年度内的合同金额
  - 季度完成率：统计当前季度合同金额 / 当前季度目标
  - 时间口径优先取 `contract_sign_time`，为空时回退 `contract_no_date`
  - 为避免同一合同多笔回款重复累计，按 `salesperson_name + contract_no` 去重后汇总
  - 页面内其它完成额口径已同步统一：
    - 顶部概览
    - 小组战况
    - 个人排名
  - 编辑弹窗“完成额”改为只读自动值，保存时不再写入手工完成额
  - 看板金额展示细节已调整：
    - 完成额统一按“万”展示并保留两位小数
    - 个人排名移除“季度目标”列
    - 个人排名“季度完成率”改为“完成率”，展示年度完成率
    - 编辑弹窗去掉“完成额(自动)”列
    - 编辑弹窗目标列补充“(万)”单位
    - “当季拜访量”补单位“次”
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 125. test / prod 同名用户账号已合并
- 文件：
  - `backend/scripts/merge_duplicate_users.py`
- 调整：
  - 新增重复用户合并脚本，按“保留仍被业务使用的账号、删除未使用重复账号”的规则处理
  - `康鹏`：
    - test：保留 `88`，删除 `306`
    - prod：保留 `88`，删除 `291`
    - 同步保留账号资料为 `sso_uid='kangpeng'`、手机号 `15101690158`
  - `杨延涛`：
    - test / prod：保留 `79`，删除 `70`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/merge_duplicate_users.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/merge_duplicate_users.py --both`
- 已验证：
  - test 重复昵称结果为空
  - prod 重复昵称结果为空
  - test / prod 保留账号校验：
    - `康鹏 -> id=88`
    - `杨延涛 -> id=79`
- 补充处理：
  - 正式环境后续又出现 `康鹏 id=307`
  - 已再次核对引用，仅剩 `user_roles` 轻引用，无业务数据引用
  - 已删除 `307`，正式环境最终仅保留 `康鹏 id=88`
  - 登录建号修复：
  - 文件：
    - `backend/app/api/endpoints/auth.py`
  - 调整：
    - 企微登录查不到 `sso_uid=企微userid` 时
    - 新增按 `mobile`、`email`、唯一 `nickname` 的兜底匹配
    - 命中已有账号后，直接更新该账号 `sso_uid` 为新的企微 userid，不再新建重复用户
  - 触发背景：
    - 正式环境 `康鹏` 因企微返回 userid 从 `kangpeng` 变为 `tangpeng`
    - 旧逻辑只按 `sso_uid` 精确查询，导致再次新建 `id=308`
    - 已删除 `308`
  - 已验证：
    - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/auth.py`
- 康鹏特殊处理：
  - `kangpeng`、`tangpeng` 已加入固定企微 userid 别名组
  - 登录时统一视为同一人，优先命中既有账号
  - 本次按用户要求不新建新用户
- 本次未新增依赖

### 124. 客户跟进导出取消“客户名等于联系人名”过滤
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 调整：
  - 删除导出 SQL 中 `COALESCE(TRIM(customer_name), '') <> COALESCE(TRIM(contact_name), '')`
  - 避免把有效客户误排除出 Excel 导出
- 已确认：
  - 正式环境 `河北清源水利工程有限公司-刘先生` 原先未导出的原因，就是命中了这条过滤
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 123. kehu51 新增 5 人客户和跟进数据全量重爬
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
- 调整：
  - 把 `_clean_html_text` 正式接入写库字段，避免 `<font color=...>`、`<font face=...>` 之类脏内容再次入库
  - `follow_result` 写入前改为 HTML 文本清洗
  - `last_follow_content` 写入前改为 HTML 文本清洗
  - 跟进去重判断也改为按清洗后的 `follow_result` 比对
  - 客户列表存在性判断统一按规范化后的 `customer_name` 比对
- 本次全量重爬人员：
  - 薄再峥 `1442269`
  - 崔凯 `1442270`
  - 卢迪 `1442271`
  - 王太鼎 `1442272`
  - 闫治国 `1442240`
- 已执行：
  - test 环境客户列表全量：`run_id=35`，成功 `27`，失败 `0`
  - prod 环境客户列表全量：`run_id=366`，成功 `27`，失败 `0`
  - test 环境跟进全量：
    - 薄再峥 `1613/1613`，新增 `17`
    - 崔凯 `2936/2936`，新增 `24`
    - 卢迪 `3086/3086`，新增 `27`
    - 王太鼎 `3974/3974`，新增 `1790`
    - 闫治国 `356/356`，新增 `6`
  - prod 环境跟进全量：
    - 薄再峥 `1613/1613`，新增 `13`
    - 崔凯 `2936/2936`，新增 `24`
    - 卢迪 `3086/3086`，新增 `27`
    - 王太鼎 `3974/3974`，新增 `1783`
    - 闫治国 `356/356`，新增 `5`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py`
- 本次未新增依赖

### 123. 客户跟进记录管理增加手动标识与删除限制
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `backend/sql/alter_kehu51_follow_records_add_is_manual_created.sql`
- 调整：
  - `crawler_kehu51_follow_records` 新增字段 `is_manual_created`
  - 后台手动新建客户跟进记录时写入 `is_manual_created=1`
  - 列表接口新增返回 `is_manual_created`
  - 前端客户跟进记录管理仅对手动新建记录显示删除按钮
  - 新增删除接口：`DELETE /api/after-sales-dept/customer-follow-rows/{row_id}`
  - 后端限制：爬取同步数据不可删除，仅手动新建数据允许删除
  - 历史数据回填：`source_from='手动新增'` 的旧记录统一补成 `is_manual_created=1`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/alter_kehu51_follow_records_add_is_manual_created.sql`
  - test / prod 均已执行成功
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 122. 业务团队管理看板图表布局和新增人员测试数据
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/create_after_sales_management_dashboard_members.sql`
  - `backend/sql/insert_after_sales_dashboard_new_member_test_data.sql`
- 调整：
  - 个人拜访量图表从左右并排改为独占一行
  - 图表高度从 `260px` 拉高到 `340px`
  - 最近跟进区域下移到个人拜访量图表下方
  - 个人拜访量图表 tooltip 已改为中文展示，避免出现 `follow_count`
  - 新增动态人员测试数据：
    - 薄再峥、崔凯、卢迪、王太鼎、闫治国
  - 后端运行时兜底新增人员时，也会按默认测试数据初始化
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/insert_after_sales_dashboard_new_member_test_data.sql`
  - test / prod 均已补齐缺失新增人员数据
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 && npm run build`
- 本次未新增依赖

### 119. 业务团队管理看板人员改为 OA 动态来源
- 文件：
  - `backend/app/services/after_sales_team.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 调整：
  - 看板人员不再固定四人
  - 动态读取 OA `润达 > 售后服务部(业务组)` 及子部门 `开拓组 / 深耕组`
  - 排除：张涛、刘超VIP3、俞莎莎、张波VIP4、李春梅、李玉敏、毛丽恒
  - 仅取有 `LOGINID` 且 `STATUS IN (0,1)` 的人员
  - 显示名统一去掉 `VIP*` 后缀
  - Redis 缓存 key：`after_sales:business_team_members:v1`，TTL 1 天
  - 当前动态人员：范运成、缴志健、李秋雷、马超、薄再峥、崔凯、卢迪、王太鼎、闫治国
  - 看板统计兼容历史数据中带 `VIP*` 的姓名
  - 售后客户无忧跟进人候选接口改为同一套动态人员来源
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_team.py app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python - <<'PY' ... get_after_sales_business_team_members ... PY`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 && npm run build`
- 本次未新增依赖

### 120. kehu51 客户列表负责人 ID 改为 HR 员工接口动态获取
- 文件：
  - `backend/app/services/kehu51_employees.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
- 调整：
  - `kehu51_customer_list` 不再使用旧固定四人的 kehu51 用户 ID
  - 改为请求 HR 员工接口：`https://hr.kehu51.com/api/employee/list?groupId=12&pageSize=10&pageIndex=1&filters={}`
  - 使用 OA 动态售后业务团队姓名匹配 HR 员工，取 `userID` 作为客户列表爬取参数
  - 员工列表缓存 Redis：`kehu51:employees:group:12:v1`，TTL 1 天
  - 匹配后的负责人 ID 缓存 Redis：`kehu51:after_sales_business_team_users:v1`，TTL 1 天
  - 如匹配不到人员，客户列表任务不再回退旧四人，直接无目标结束并记录日志
  - 为兼容源站，支持优先执行 Redis 中的 HR curl 模板：`kehu51:employee_list_curl:v1`
  - HR 接口未登录 / cookie 失效时，使用 `kehu51` 站点凭证中的 `login_curl` 自动刷新凭证，刷新后重试
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/kehu51_employees.py app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py`
  - 已成功从 HR 接口匹配到 9 个负责人 ID：
    - 马超 `1774436`
    - 李秋雷 `1592326`
    - 范运成 `1680321`
    - 闫治国 `1442240`
    - 王太鼎 `1442272`
    - 卢迪 `1442271`
    - 缴志健 `1442273`
    - 崔凯 `1442270`
    - 薄再峥 `1442269`
  - 客户列表 handler 运行目标验证为 9 个
- 本次未新增依赖

### 121. 业务团队管理看板增加季度目标并移除趋势图
- 文件：
  - `backend/app/models/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/create_after_sales_management_dashboard_members.sql`
  - `backend/sql/alter_after_sales_dashboard_members_add_quarter_targets.sql`
  - `backend/migrations/versions/b6c7d8e9f0a2_add_after_sales_management_dashboard_members.py`
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
- 调整：
  - 移除看板“近 30 天拜访趋势”折线图
  - 编辑业务团队管理看板增加每人 Q1/Q2/Q3/Q4 季度目标
  - `after_sales_management_dashboard_members` 增加：
    - `q1_target`
    - `q2_target`
    - `q3_target`
    - `q4_target`
  - 看板季度目标按当前季度字段汇总
  - 季度完成率按 `完成额 / 当前季度目标` 计算
  - 小组战况和个人排名均展示季度目标、季度完成率
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/alter_after_sales_dashboard_members_add_quarter_targets.sql`
  - test / prod 均已新增字段，历史数据按全年目标四等分回填
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/after_sales_dept.py app/schemas/after_sales_dept.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/b6c7d8e9f0a2_add_after_sales_management_dashboard_members.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 && npm run build`
  - 直接调用看板接口确认：`trend` 字段已移除，季度目标字段正常返回
- 本次未新增依赖

### 117. 新增 OA 合同回款报表镜像表
- 文件：
  - `backend/app/models/oa_contract_report.py`
  - `backend/migrations/env.py`
  - `backend/migrations/versions/c8d9e0f1a2b3_add_oa_contract_payment_report_rows.py`
  - `backend/sql/create_oa_contract_payment_report_rows.sql`
  - `backend/scripts/sync_oa_contract_payment_report.py`
- 背景：
  - 用户提供 OA FineReport 地址：`webroot/decision/view/report?viewlet=htdjb1.cpt&__bypagesize__=false&role=1`
  - 需要字段：所属部门、业务员、合同编号、合同名称、按合同编号获得合同签订时间、合同金额、业务员回款金额、回款时间
  - 现有 `xmwd_htxx_view` 仅有 `htbh/htmc/je/hkje/kpje`，缺少部门、业务员、签订时间、回款时间，不能直接承接
- 本次实现：
  - 新增本地表 `oa_contract_payment_report_rows`
  - 合同来源：OA `uf_httz`
  - 回款来源：OA `formtable_main_117`、`formtable_main_240`
  - 仅同步 `所属部门 = 售后服务部(天津)` 的数据
  - 一行对应一个合同和一笔回款；同一合同多笔回款拆多行；没有回款保留合同基础信息
  - 新增 `contract_no_date` 字段，从合同编号中扫描 8 位日期：
    - `CT-X2019122402 -> 2019-12-24`
    - `CTSH-X2020010302 -> 2020-01-03`
    - `CTRD-X2026021201 -> 2026-02-12`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_oa_contract_payment_report_rows.sql`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/sync_oa_contract_payment_report.py --both`
  - 后续补字段并重跑：
    - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/alter_oa_contract_payment_report_add_contract_no_date.sql`
    - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/sync_oa_contract_payment_report.py --both`
  - test / prod 均已写入：`1447` 个合同、`1725` 行，其中 `1477` 行有回款金额
  - test / prod 当前 `contract_no_date` 解析成功：`1725 / 1725`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/oa_contract_report.py scripts/sync_oa_contract_payment_report.py migrations/versions/c8d9e0f1a2b3_add_oa_contract_payment_report_rows.py`
- 本次未新增依赖

### 118. OA 合同回款报表同步接入计划任务
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/scripts/sync_oa_contract_payment_report.py`
  - `backend/sql/upsert_oa_contract_payment_report_scheduler_meta.sql`
- 调整：
  - 新增计划任务 `oa_contract_payment_report_sync`
  - 默认每天 `04:15` 执行，cron：`15 4 * * *`
  - 后台计划任务列表新增中文标题与描述
  - 后台计划任务详情页支持手动执行该任务
  - 调度执行时按当前 `APP_ENV` 写当前环境数据库；命令行脚本仍保留 `--env/--both`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/upsert_oa_contract_payment_report_scheduler_meta.sql`
  - test / prod 均已写入 `scheduler_job_meta`：
    - `job_id=oa_contract_payment_report_sync`
    - `trigger_type=cron`
    - `cron_expr=15 4 * * *`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py scripts/sync_oa_contract_payment_report.py`
- 本次未新增依赖

### 116. 业务团队管理看板增加编辑数据弹窗
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
- 调整：
  - 新增 `GET /api/after-sales-dept/management-dashboard/members`
  - 新增 `PUT /api/after-sales-dept/management-dashboard/members`
  - 接口权限沿用 `app:after_sales_dept:admin`
  - 固定维护四个人：范运成、缴志健、马超、李秋雷
  - 编辑弹窗支持修改：全年目标、完成额、待签约、客户回访、应酬次数、备注
  - 拜访量和客户数继续按爬虫表实时统计，不允许手工覆盖
  - 编辑弹窗已放大为接近整屏宽度，并设置表格固定列宽，保证字段尽量完整展示
  - 右上角新增“预览”按钮：
    - 点击进入浏览器全屏预览
    - 预览层只展示看板主体内容
    - 不渲染顶部导航、返回、编辑、刷新等操作入口
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/schemas/after_sales_dept.py app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/kehu51.py`
  - `cd frontend && nvm use 20 && npm run build`
- 本次未新增依赖

### 115. kehu51 客户跟进默认抓昨天和今天
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
- 问题：
  - 默认执行客户跟进任务只取当天，`2026-07-07` 当天源站总条数为 `0`
  - `2026-07-06` 马超记录未进入库中
  - 旧逻辑还会按数据库最大 `follow_time` 做增量边界，导致同一天较早记录可能被误判为历史数据
- 调整：
  - `kehu51_follow_records / customer_follow_records` 不传日期时，默认范围改为 `昨天 -> 今天`
  - 执行日志 `run_date` 显示真实范围，例如 `2026-07-06->2026-07-07`
  - 手动传 `run_date` 时仍按指定单日执行，方便补历史
  - 移除客户跟进 handler 中基于数据库最大 `follow_time` 的停止/跳过逻辑，改为完全依赖行级去重
- 已执行：
  - 生产环境补跑 `run_date=2026-07-06`
  - 再用默认参数跑一次，日志显示 `run_date=2026-07-06->2026-07-07`
  - 源站 `36` 条，抓取 `36` 条，条数校验通过
  - 生产环境 `2026-07-06` 四人统计：
    - 范运成 `3`
    - 李秋雷 `3`
    - 马超 `3`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/kehu51.py`
- 本次未新增依赖

### 114. 业务团队管理看板按部门聚合小组战况
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `frontend/src/components/dashboard/appCatalog.ts`
- 调整：
  - 页面与应用卡片标题从“售后管理看板”改为“业务团队管理看板”
  - 顶部补齐图片标语：
    - `以过程赢结果，用行动创未来——只有干出来的精彩，没有等出来的辉煌！`
    - `今天的执行力度，决定年底的胜利高度`
  - 小组战况不再固定“售后业务组”，改为按 `crawler_kehu51_customer_list.department_name` 聚合
  - 空部门统一归为“未归属部门”
- 已查 test / prod：
  - 四人有效客户均归属 `开拓组`
  - 范运成 `199`、缴志健 `308`、马超 `74`、李秋雷 `86`
  - 部门合计：开拓组 `667`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && nvm use 20 && npm run build`
- 本次未新增依赖

### 113. kehu51 客户列表所属部门字段历史回填
- 文件：
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/sql/backfill_kehu51_customer_list_department_name.sql`
  - `frontend/src/components/dashboard/appCatalog.ts`
- 现状：
  - `crawler_kehu51_customer_list.department_name` 字段和新数据同步逻辑此前已存在
  - 两套环境历史数据仍有空值：
    - test：`525`
    - prod：`538`
- 本次调整：
  - 新数据写库时兼容 `所属部门` 和 `部门` 两种字段名
  - 新增历史回填 SQL：
    - 优先取 `raw_json` 中 `所属部门/部门`
    - 其次按同负责人最高频非空部门回填
    - 仍为空时 `公客` 回填为 `公客`，其它回填为 `未归属部门`
  - 顺带将“售后管理看板”应用卡片右上角状态改为与“售后部（客户无忧）”一致的“运行中”
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/backfill_kehu51_customer_list_department_name.sql`
  - 回填后 test / prod `department_name` 空值均为 `0`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/kehu51_customer_list.py app/services/crawler_handlers/kehu51.py app/api/endpoints/admin/crawler_tasks.py`
  - `cd frontend && npm run build`
- 本次未新增依赖

### 112. 新增售后管理看板
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/models/after_sales_dept.py`
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/sql/create_after_sales_management_dashboard_members.sql`
  - `backend/migrations/versions/b6c7d8e9f0a2_add_after_sales_management_dashboard_members.py`
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `frontend/src/App.tsx`
  - `frontend/src/components/dashboard/appCatalog.ts`
- 新增售后管理看板入口：
  - 路径：`/apps/after-sales-management-dashboard`
  - 权限：`app:after_sales_dept:admin`
  - 应用中心新增“售后管理看板”卡片
- 新增数据库表：
  - `after_sales_management_dashboard_members`
  - 固定人员：范运成、缴志健、马超、李秋雷
  - 表字段维护全年目标、完成额、待签约金额、客户回访、应酬次数、排序、备注
  - 拜访量不入表，按全年周期从 `crawler_kehu51_follow_records` 实时统计客户跟进个数
- 看板布局参考用户提供的业务团队管理看板图片：
  - 顶部展示品牌、看板标题、年底倒计时
  - 左侧展示业绩目标、小组战况、近 30 天拜访趋势
  - 右侧展示四人个人排名、个人拜访量、最近跟进
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_after_sales_management_dashboard_members.sql`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/models/after_sales_dept.py migrations/versions/b6c7d8e9f0a2_add_after_sales_management_dashboard_members.py`
  - `cd frontend && npm run build`
- 本次未新增依赖

### 111. kehu51 源站总条数为 0 时不再误抓占位数据
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/crawler_sites/kehu51/formatter.py`
- 问题：
  - 客户跟进演示任务预取到 `recordCount=0`
  - 后续仍继续请求分页，源站返回非业务占位内容
  - 格式化后日志显示 `count=1`，随后因字段不符合预期停止
- 调整：
  - `kehu51_follow_records / customer_follow_records / kehu51_customer_list` 预取总条数为 `0` 时，直接记录空结果并跳过分页请求
  - 演示模式仍记录 `[COUNT-CHECK] 演示模式跳过总条数一致性强校验`
  - `kehu51` formatter 增加兜底：返回数据里 `total/recordCount/count/totalCount/records` 为 `0` 时统一输出 `items=[]`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py crawler_sites/kehu51/formatter.py`
- 本次未新增依赖

### 110. 爬虫站点凭证改为 Redis 共享缓存
- 文件：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/services/crawler_handlers/okcis.py`
  - `backend/scripts/cache_crawler_site_credentials.py`
- 背景：
  - `kehu51` 运行时凭证文件已从 Git 移除并加入忽略
  - 生产环境部署后如果 `credentials.json` 不存在，旧逻辑仍会硬读文件，导致自动登录和请求执行失败
- 调整：
  - 新增 Redis 缓存 key：`crawler:site_credentials:{site_key}`
  - TTL：30 天
  - 读取站点凭证时按“本地文件 + Redis + 数据库”合并：
    - 数据库继续负责 `headers/cookies/cookie_text`
    - Redis / 本地文件负责补齐 `login_curl/check_login_curl`
  - 后台保存凭证、刷新凭证、响应 cookie 合并后，都会把完整规范化凭证写入 Redis
  - 不再依赖 `save_site_credentials` 写回运行时凭证文件
  - `credentials.json` 不存在时返回空凭证参与合并，不再直接中断任务
  - `okcis` 独立 handler 同步改为支持 Redis 凭证兜底
  - 新增脚本 `backend/scripts/cache_crawler_site_credentials.py`，用于把本机现有站点凭证写入指定环境 Redis
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/okcis.py scripts/cache_crawler_site_credentials.py`
  - 已把本机 `kehu51` 当前凭证写入 `config/env_test` 与 `config/env_prod` 对应 Redis
- 本次未新增依赖

### 101. kehu51 客户列表改为四个负责人全量同步
- `backend/crawler_sites/kehu51/customer_list.json`
- `backend/app/services/crawler_handlers/kehu51.py`
- `backend/app/services/crawler_handlers/kehu51_customer_list.py`
- `backend/app/api/endpoints/admin/crawler_tasks.py`
- `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- `backend/sql/alter_kehu51_customer_list_add_is_deleted.sql`
- 客户列表任务 `kehu51_customer_list` 不再按全客户列表口径抓取，改为固定四个负责人分别同步：
  - 范运成 `1680321`
  - 缴志健 `1442273`
  - 马超 `1774436`
  - 李秋雷 `1592326`
- 每个负责人通过 `selectuserid={{user_id}}` 和 `customParam={"field":"selectUserID","key":{{user_id}}}` 请求源站
- 客户列表逻辑已拆到独立 handler，调度主链路只负责调用统一接口
- 每个负责人总条数通过 GET `cuslist.aspx?appointtype=&name=负责人为：{{user_name}}&selectuserid={{user_id}}&selecttype=1&viewname=allcus&templateid=` 的 `<font id="rptCount">` 提取
- 分页请求 `whereSql` 从同页隐藏字段 `#listcount_wheresql` 提取
- 每个负责人都强校验“源站总条数 == 实际分页抓取条数”，不一致则任务失败且不做删除标记
- 本轮源站返回的数据写入/更新为 `is_deleted=0`
- 同一负责人本轮未返回的历史客户标记为 `is_deleted=1`
- 售后客户跟进客户候选与导出客户列表默认排除 `is_deleted=1`

### 94. kehu51 today 增量参数已切回浏览器真实请求
- `backend/crawler_sites/kehu51/request.json`
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 已修正：
  - 主请求 `referer` 改为 `ShowID=2&ViewName=allfollow`
  - `prefetch` 的 `url / referer` 也改为 `ShowID=2`
  - `whereSql` 改为浏览器 today 请求实际值 `4689C015...7E9A0`
  - 总条数提取补充 `<font id=\"rptCount\">`
  - 测试脚本 4 个人员用例同步改为 today 参数，`customParam=1`
- 根因：
  - 之前测试脚本和模板仍混用旧全量筛选参数，导致打印出的 `819` 不是“今天数据条数”
- 已通过：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m unittest tests.test_kehu51_follow_count_consistency`

### 95. kehu51 增量链路已继续切到 CreateTime 搜索模式
- `backend/crawler_sites/kehu51/request.json`
- `backend/crawler_sites/kehu51/formatter.py`
- `backend/app/api/endpoints/admin/crawler_tasks.py`
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 已调整为：
  - 预取先访问 `mode=search` 页面
  - 通过页面脚本提取 `recordCount / showID / whereSql`
  - 正式分页请求改为 `FollowTools.aspx/GetFollowScrollData`
  - 分页参数与 `recordCount` 已支持写入 `json_body.ajaxParam`
- 测试脚本也已改为按 `CreateTime` 范围跑指定人员链路

### 96. kehu51 正式增量模板 UserID 已改为空
- `backend/crawler_sites/kehu51/request.json`
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 正式模板中的搜索页 URL / referer 已统一改为 `UserID=` 空值
- 测试脚本仍调用正式增量模板，只在运行时替换成指定人员 `UserID`

### 97. kehu51 实时测试默认改为执行增量入库
- `backend/tests/test_kehu51_follow_count_consistency.py`
- `COUNT_ONLY` 已改为读取环境变量 `KEHU51_COUNT_ONLY`
- 默认值改为 `0`
- 当前实时测试默认会执行“已存在跳过，不存在写入”的增量第一页逻辑

### 98. kehu51 测试脚本已改为四个人全量分页抓取
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 当前实时测试已改为：
  - 搜索页只传 `UserID`
  - 不再传 `CreateTime`
  - 依据 `recordCount / pageSize` 抓完整个分页
  - 最终校验 `fetched_items == expected_total`
  - 非 `COUNT_ONLY` 模式下按全部分页结果走入库

### 99. kehu51 实时测试已临时缩到只跑范运成
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 为便于排查当前链路，`FOLLOW_CASES` 已临时只保留：
  - `范运成 / targetID=1680321`

### 100. kehu51 实时测试已补连接池释放
- `backend/tests/test_kehu51_follow_count_consistency.py`
- 在 `IsolatedAsyncioTestCase.asyncTearDown` 中增加 `await engine.dispose()`
- 处理测试结束后 `aiomysql` 未关闭连接的 `ResourceWarning`

### 101. 范运成全量条数差异已定位到缺失+重复
- 源站全量抓取 `819` 条，测试分页抓取结果也是 `819` 条，说明抓取流程正常
- 以当前写库去重口径 `客户名称 + 联系人 + 跟进时间` 计算，源站唯一键共 `818` 条
- 本地库中 `creator_name=范运成` 当前共 `807` 条
- 已定位：
  - 缺少 12 条近期待补记录
  - 已存在 1 组重复记录：`辰雅佳苑 / 苏工 / 2025-10-10 19:04:00`
- 因此当前数据库唯一业务条数实际为 `806`，和源站唯一键 `818` 相差 `12`

### 102. kehu51 增量写库已修复“历史缺失记录被边界跳过”
- `backend/app/services/crawler_handlers/kehu51.py`
- 问题根因：
  - 增量写库阶段若 `row_follow_time <= 当前库最大 follow_time`，会直接 `continue`
  - 导致虽然分页已抓到数据，但“时间较早且库里缺失”的记录不会补写
- 已调整为：
  - 仍保留“命中历史边界后当前页结束即停止后续分页”的行为
  - 但当前边界页内的缺失记录，继续按现有去重规则补写
- 2026-07-06 本地实时复测结果：
  - `范运成 / targetID=1680321`
  - 全量抓取 `819`
  - 补写成功 `12`
  - 输出：`db_inserted=12 db_skipped=807`

### 103. kehu51 跟进记录历史重复数据已清理
- 清理表：`crawler_kehu51_follow_records`
- 去重口径：`客户名称 + 联系人 + 跟进时间`
- 保留规则：每组仅保留最早一条 `id`
- 清理结果：
  - 清理前总行数：`40430`
  - 重复组数：`484`
  - 重复冗余行：`688`
  - 实际删除：`688`
  - 清理后总行数：`39742`
  - 清理后重复组：`0`

### 104. kehu51 四个人员全量核对已补跑 test / prod
- 执行时间：`2026-07-06`
- 范围：
  - `范运成 / 1680321`
  - `缴志健 / 1442273`
  - `马超 / 1442273`（后确认旧值错误）
  - `李秋雷 / 1592326`
- test 环境结果：
  - 去重前重复冗余行 `0`，去重后 `0`
  - `范运成`：源站 `819`，唯一 `818`，库内 `818`
  - `缴志健`：源站 `1870`，唯一 `1867`，库内 `1867`
  - `马超`：源站 `1870`，唯一 `1867`，库内 `345`，原因是和 `缴志健` 使用同一 `target_id=1442273`，按当前去重键全部与他人记录冲突
  - `李秋雷`：源站 `809`，唯一 `807`，库内 `807`
- prod 环境结果：
  - 去重前重复冗余行 `688`，已删除 `688`，去重后 `0`
  - `范运成`：补写 `9`，库内补齐到 `818`
  - `缴志健`：补写 `11`，库内补齐到 `1867`
  - `马超`：仍为 `348`，同样因 `target_id=1442273` 与 `缴志健` 完全重合，冲突 `1867`
  - `李秋雷`：补写 `11`，库内补齐到 `807`
- 结论：
  - 现两套环境重复数据都已清理完成
  - `范运成 / 缴志健 / 李秋雷` 已按当前去重口径补齐
  - `马超` 若要单独补齐，需要确认其真实 `target_id`，否则会继续被判定为与 `缴志健` 同一批数据

### 105. 马超真实 target_id 已更正为 1774436
- 执行时间：`2026-07-06`
- 更正后复跑结果：
  - test：源站 `355`，抓取 `355`，库内 `355`，无需补写
  - prod：源站 `355`，抓取 `355`，库内由 `348` 补到 `355`，补写 `7`
- 结论：
  - 马超此前差异由错误 `target_id=1442273` 导致
  - 改为 `1774436` 后，两套环境均已与源站对齐

### 106. kehu51 跟进结果富文本截断已修复
- 文件：`backend/crawler_sites/kehu51/formatter.py`
- 问题：
  - timeline 脚本里的 `detail.html` 为转义后的富文本
  - 旧正则 `\"html\":\"(.*?)\"` 在 `<font face=\"...` 位置提前截断
  - 导致格式化结果只剩 `"<font face=\\"`
- 修复：
  - 改为支持转义字符的提取规则：`detail ... "html":"((?:\\\\.|[^\"])*)"`
  - 提取后再 `json.loads` / `unicode_escape` 还原 HTML
- 已验证：
  - `范运成 / 辰雅佳苑 / 苏工 / 2025-10-10 19:04:00`
  - 两条原始跟进结果现可正确解析为：
    - `楼内管道维修7000元已成交`
    - `应急地埋维修8000元已成交`
- 影响：
  - 后续按“客户名称 + 联系人 + 跟进时间 + 跟进内容”判重时，这两条不再误判重复

### 107. kehu51 四人批量测试脚本已复跑通过
- 文件：`backend/tests/test_kehu51_follow_count_consistency.py`
- 当前实时测试人员：
  - `范运成 / 1680321`
  - `缴志健 / 1442273`
  - `马超 / 1774436`
  - `李秋雷 / 1592326`
- 执行命令：
  - `RUN_KEHU51_LIVE_TEST=1 /opt/anaconda3/envs/smart/bin/python -m unittest tests.test_kehu51_follow_count_consistency`
- 结果：
  - `范运成`：`total=819 fetched_items=819 db_inserted=0 db_skipped=819`
  - `缴志健`：`total=1870 fetched_items=1870 db_inserted=0 db_skipped=1870`
  - `马超`：`total=355 fetched_items=355 db_inserted=0 db_skipped=355`
  - `李秋雷`：`total=809 fetched_items=809 db_inserted=0 db_skipped=809`
  - `Ran 2 tests in 61.906s`
  - `OK`

### 108. kehu51 测试脚本已改为单次爬取并复用 Redis 缓存
- 文件：
  - `backend/tests/test_kehu51_follow_count_consistency.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
- 调整：
  - 测试脚本默认改为“全部用户全量”，不再固定 4 个 `UserID`
  - 新增 Redis 缓存：
    - key：`kehu51:live_test:follow_records:full:{case_name}:{target_id_or_all}`
    - TTL：默认 `1800` 秒，可通过 `KEHU51_CACHE_TTL_SECONDS` 调整
  - 第二套环境执行时若命中缓存，直接复用首次抓取结果，不再重复请求源站
- 同步修正正式去重口径：
  - 跟进记录存在性判断新增 `follow_result`
  - 增量 `source_dedupe_key` 也补入 `跟进结果`
  - 避免“同客户 + 同联系人 + 同跟进时间，但跟进内容不同”被误判成同一条

### 109. kehu51 实时测试已改为稳定 4 人全量校验并增加本地缓存
- 文件：`backend/tests/test_kehu51_follow_count_consistency.py`
- 调整：
  - 放弃“全部用户”全量链路，原因是源站当前会出现空响应或异地登录脚本，稳定性不足
  - 恢复为 4 个已确认 `targetID` 的全量校验：
    - `范运成 / 1680321`
    - `缴志健 / 1442273`
    - `马超 / 1774436`
    - `李秋雷 / 1592326`
  - 缓存策略改为：
    - Redis 缓存继续保留
    - 新增本地缓存文件 `backend/tmp/kehu51_live_test_follow_cache.json`
    - 本地缓存优先，便于 test / prod 两套环境在同机复用同一份抓取结果，不重复爬源站
  - `asyncTearDown` 已补 Redis 连接关闭，避免测试结束时出现 `redis ResourceWarning`
- 2026-07-06 本地实时验证：
  - 命令：
    - `cd backend && set -a && source config/env_test && set +a && RUN_KEHU51_LIVE_TEST=1 KEHU51_COUNT_ONLY=1 KEHU51_CACHE_TTL_SECONDS=1800 /opt/anaconda3/envs/smart/bin/python -m unittest tests.test_kehu51_follow_count_consistency`
  - 结果：
    - `范运成`：`total=819 fetched_items=819`
    - `缴志健`：`total=1870 fetched_items=1870`
    - `马超`：`total=355 fetched_items=355`
    - `李秋雷`：`total=809 fetched_items=809`
    - `Ran 2 tests in 84.990s`
    - `OK`

### 110. kehu51 全部用户列表已接入动态测试入口并加 Redis 5 天缓存
- 文件：`backend/tests/test_kehu51_follow_count_consistency.py`
- 调整：
  - 新增 `KEHU51_ALL_USERS=1` 开关
  - 通过接口拉取全部用户列表：
    - `https://s25.kehu51.com/App/Ajax/GetSmartUserList.aspx?viewName=allcus&type=user&tableID=39&callback=loadUserCallBack`
  - 解析返回中的 `RealName / UserID` 作为测试用户清单
  - 用户列表写入 Redis：
    - key：`kehu51:live_test:user_list:allcus`
    - TTL：`5` 天
- 本地验证：
  - 已成功拉取 `94` 个用户
  - 前 5 个样例：
    - `包悦 / 1441907`
    - `白晓宁 / 1442235`
    - `白建辉 / 1442239`
    - `薄再峥 / 1442269`
    - `白坤 / 1456670`

### 111. kehu51 跟进记录实时测试脚本说明已落地 md
- 文件：`doc/kehu51-follow-live-test.md`
- 已整理内容：
  - 脚本位置
  - test / prod 环境执行方式
  - 固定 4 人全量测试命令
  - 全部用户全量测试命令
  - `RUN_KEHU51_LIVE_TEST / KEHU51_ALL_USERS / KEHU51_COUNT_ONLY / KEHU51_CACHE_TTL_SECONDS` 开关说明
  - Redis 与本地缓存规则
  - 控制台输出字段说明

### 112. kehu51 全部用户全量测试已补 500 重试与零数据短路
- 文件：`backend/tests/test_kehu51_follow_count_consistency.py`
- 调整：
  - `FollowTools.aspx/GetFollowScrollData` 遇到 `500` 时自动重试
  - 重试期间会自动刷新登录态
  - 返回空串时也会自动重试
  - `recordCount <= 0` 时直接返回空分页结果，不再继续请求分页接口
- 目的：

### 113. kehu51 正式环境 4 人数据串位根因已定位并修复
- 文件：
  - `backend/tests/test_kehu51_follow_count_consistency.py`
  - `CURRENT_CONTEXT.md`
- 根因：
  - 实时测试脚本写库时固定使用 `run_id=0`
  - 全量模式 `source_dedupe_key` 使用 `task_key + run_id + page + item`
  - 4 个人连续写库时，相同 `page/item` 会跨人员相互覆盖，导致正式环境出现“条数差几条、内容串位”现象
- 修复：
  - 测试脚本改为按 `target_id` 传独立 `run_id`
  - 避免不同人员之间再发生全量页位覆盖
- 正式环境处理：
  - 删除了 4 条被串位的错误记录：
    - `缴志健 / 杨经理 / 何经理 / 2025-07-07 14:03:00`
    - `马超 / 众联产业服务公司 / 彭经理 / 2026-03-05 19:40:00`
    - `李秋雷 / 深圳市长城楼宇科技有限公司天津分公司 / 王 / 2025-04-24 11:53:00`
    - `李秋雷 / 长城物业 / 王 / 2025-04-25 09:38:00`
  - 使用本地缓存数据重跑 4 人全量回填后，正式环境已恢复为：
    - `范运成`：`819`
    - `缴志健`：`1869`
    - `马超`：`355`
    - `李秋雷`：`808`
- 说明：
  - `缴志健` 源站总条数 `1870`，其中 `1` 条为完全相同重复，按当前“客户名称 + 联系人 + 跟进时间 + 跟进内容”口径落库后为 `1869`
  - `李秋雷` 源站总条数 `809`，其中 `1` 条为完全相同重复，按同口径落库后为 `808`
  - 本次未新增依赖

### 134. 修复售后工单监控新消息位置判断
- 文件：
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 问题：
  - 实时工单接口展示数量改为 `count=10` 后，底部补出来的历史工单因为之前未见过，也会被误判为新消息，出现“新消息在下面”的效果
- 调整：
  - 新工单判断从“未见过的 ID”改为“插入到当前列表顶部、位于已有工单之前的记录”
  - 底部补充的历史工单不再触发高亮、上移账户和语音播报
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 135. 调整售后客户跟进导出预览客户列规则
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 导出 / 预览导出客户列不再按客户录入时间生成
  - 改为按当年跟进记录生成客户列，哪个季度有跟进记录，客户就展示在哪个季度
  - 2025 年录入但 2026 年有跟进的客户，也会进入 2026 年对应季度预览
  - 每个季度客户列只展示该季度的跟进内容
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 本次未新增依赖

### 136. 售后工单实时监控增加步骤展示和接单前播报
- 文件：
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 实时工单卡片增加步骤展示，兼容 `steps` / `Steps` / `stepList` 等字段
  - 步骤 `display=1` 显示蓝色，`display=0` 显示灰色
  - 语音播报不再单纯按新工单触发，改为按步骤判断
  - 当第一个灰色步骤早于“接单”时持续播报“有未接单工单”
  - 刷新后灰色步骤达到“接单”或之后时停止继续播报
  - 同一账户已经在播或排队时，不重复塞入提醒，避免语音队列堆积
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 137. 优化售后客户跟进导出预览查询性能
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/migrations/versions/m3n4o5p6q7r8_add_kehu51_follow_export_index.py`
  - `backend/sql/add_kehu51_follow_export_index.sql`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 分析：
  - 原导出预览 SQL 先查客户，再按负责人循环查询跟进记录
  - 查询条件使用 `creator_name + follow_time + customer_channel + customer_name`，但库里只有 `follow_time`、`customer_channel` 等单列索引，容易出现慢查询或大量回表
- 调整：
  - 导出预览改为一次查询当年跟进记录，再按负责人 / 季度 / 客户在内存分组
  - 去掉按负责人循环查询和大 `customer_name IN (...)`
  - 新增组合索引 `idx_follow_export_creator_time_channel_customer (creator_name, follow_time, customer_channel, customer_name)`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_kehu51_follow_export_index.sql`
  - test / prod 均执行成功
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/m3n4o5p6q7r8_add_kehu51_follow_export_index.py`
- 本次未新增依赖

### 138. 导出预览按人员懒加载已回退
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 按要求回退“导出预览按人员懒加载”方案
  - `/customer-follow-rows/export` 不再接收 `owner_name`
  - 导出预览恢复为一次生成全部人员 sheet，弹窗内按 sheet 切换
  - 保留此前 SQL 合并和组合索引优化
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 139. 客户跟进导出改为客户ID和 Redis 缓存驱动
- 文件：
  - `backend/app/services/kehu51_follow_cache.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
  - `backend/sql/add_kehu51_follow_customer_list_id.sql`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `crawler_kehu51_follow_records` 新增 `customer_list_id`，关联 `crawler_kehu51_customer_list.id`
  - 新增索引 `idx_follow_customer_list_id`
  - 新增索引 `idx_follow_customer_list_time_channel (customer_list_id, follow_time, customer_channel)`
  - 导出预览先按负责人获取今年有跟进的客户ID集合
  - 每个客户按 `customer_list_id` 读取 Redis 缓存
  - Redis 缓存不存在时查询该客户全部跟进记录，以及当年首次跟进时间，并写入 Redis
  - 客户展示季度由缓存中的 `first_year_follow_time` 判断
  - 客户进入预览后，跟进内容从缓存中的全部跟进记录生成
  - 客户跟进爬取、后台旧同步入口、手工新增 / 编辑 / 删除均会维护 `customer_list_id` 并清理 / 刷新缓存
  - 缓存失效会同时清理跟进记录所属年份和当前年份，避免补录历史跟进后当前年度预览仍读旧缓存
  - 导出异常日志移除已不存在的旧参数，避免异常时二次报错
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_kehu51_follow_customer_list_id.sql`
  - test / prod 均执行成功，包含字段、索引和历史数据回填
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/kehu51_follow_cache.py app/services/crawler_handlers/kehu51.py app/api/endpoints/admin/crawler_tasks.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
- 本次未新增依赖

### 140. 修复客户ID回填范围和导出 SQL 别名错误
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/add_kehu51_follow_customer_list_id.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 问题：
  - 导出 SQL 引用了 `f.follow_time`，但 `crawler_kehu51_follow_records` 没有声明别名 `AS f`，导致 `Unknown column 'f.follow_time'`
  - 历史回填脚本不应清空已有 `customer_list_id`
- 调整：
  - 导出 SQL 补 `FROM crawler_kehu51_follow_records AS f`
  - 历史回填脚本改为只处理 `customer_list_id IS NULL`
  - 回填匹配规则当时为：跟进记录 `客户名称 + 联系人 + 联系方式` 对应客户列表 `客户名称 + 联系人 + 手机号`；后续已废弃，最新口径见第 134 条
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_kehu51_follow_customer_list_id.sql`
  - test / prod 均执行成功
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/services/kehu51_follow_cache.py app/services/crawler_handlers/kehu51.py app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 137. 售后工单实时监控降低刷新频率并调整步骤样式
- 文件：
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 实时监控刷新频率从 `3~7` 秒改为 `10~20` 秒随机刷新
  - 步骤展示从圆角胶囊改为箭头流程条样式
  - `display=1` 使用蓝色箭头，`display=0` 使用灰色箭头
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 138. 售后工单步骤名称改用 nodeName
- 文件：
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 实时监控步骤名称优先读取 `nodeName` / `NodeName`
  - 避免接口已有步骤名时仍显示默认“步骤”
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 114. kehu51 跟进记录判重已补联系方式条件
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `CURRENT_CONTEXT.md`
- 调整：
  - 跟进记录存在性判断新增 `联系方式`
  - 增量 `source_dedupe_key` 生成字段也同步补入 `联系方式`
- 当前判重口径：
  - `客户名称 + 联系人 + 联系方式 + 跟进时间 + 跟进结果`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/kehu51.py tests/test_kehu51_follow_count_consistency.py`
- 本次未新增依赖

### 115. kehu51 正式环境已按新判重口径重跑 4 人全量写库
- 执行命令：
  - `cd backend && set -a && source config/env_prod && set +a && RUN_KEHU51_LIVE_TEST=1 KEHU51_COUNT_ONLY=0 KEHU51_CACHE_TTL_SECONDS=1800 /opt/anaconda3/envs/smart/bin/python -m unittest tests.test_kehu51_follow_count_consistency`
- 结果：
  - 测试通过：`Ran 2 tests` / `OK`
  - 正式环境当前条数：
    - `范运成`：`819`
    - `缴志健`：`1869`
    - `马超`：`355`
    - `李秋雷`：`809`
  - 按最新判重口径 `客户名称 + 联系人 + 联系方式 + 跟进时间 + 跟进结果` 对比缓存源数据：
    - 4 人均为 `missing=0 / extra=0`
- 说明：
  - `李秋雷` 在本次正式重跑中实际补写 `1` 条
  - 本次未新增依赖

### 116. 招投标企业微信推送条数口径已改为按发布日期当天统计
- 文件：
  - `backend/app/tasks/scheduler.py`
  - `CURRENT_CONTEXT.md`
- 调整：
  - `okcis_daily_summary_push` 统计条件从 `crawler_okcis_notices.crawled_at` 当天改为 `publish_date` 当天
  - 无数据提示文案同步改为“无发布日期为当天的招投标数据”
  - 推送卡片跳转链接改为精确当天：`date_from=当天&date_to=当天`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
- 本次未新增依赖
  - 降低全部用户批量全量测试时，个别人偶发 `500` 导致整轮中断的问题

### 113. kehu51 全部用户模式已改为默认非严格执行
- 文件：
  - `backend/tests/test_kehu51_follow_count_consistency.py`
  - `doc/kehu51-follow-live-test.md`
- 调整：
  - `KEHU51_ALL_USERS=1` 时，单个用户若仍然报错，不再直接中断整轮测试
  - 控制台输出：
    - 单用户失败：`[KEHU51-ERROR]`
    - 本轮汇总：`[KEHU51-ERROR-SUMMARY]`
  - 如需保持“单个用户失败即整轮失败”，可传：
    - `KEHU51_ALL_USERS_STRICT=1`

### 67. 企微模板列表 500 已修复
- 根因是运行环境 `wecom_message_templates` 缺少 `template_type` 字段，接口已开始查询该列
- 已同步补齐 test / prod 两套环境字段与索引，并把历史数据默认回填为 `single_user`
- `backend/app/api/endpoints/admin/wecom_message_templates.py` 已增加缺字段兼容兜底，避免未迁移环境直接 500

### 74. 用户管理分页已切到公共三方分页组件
- 新增 `frontend/src/components/ui/pagination-control.tsx`
- 基于 `react-paginate` 做统一样式封装
- 用户管理列表已接入该公共组件
- 依赖已补到 `frontend/package.json`
- 已用 `Node v20.20.2 / npm 11.16.0` 通过 `npm run build`

### 75. 公共下拉组件已切到 react-select
- 新增 `frontend/src/components/ui/app-select.tsx`
- 基于 `react-select` 封装统一普通下拉 / 搜索下拉能力
- `frontend/src/components/ui/filterable-select.tsx` 已改为内部复用 `app-select`
- 用户管理页已接入新公共下拉组件
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`

### 76. 项目内分页使用点已统一切到公共分页组件
- 公共分页组件为 `frontend/src/components/ui/pagination-control.tsx`
- 用户管理页与业务学习模块内原有分页区块已统一切换
- 原分页使用点已不再直接依赖旧分页拼装组件
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`

### 77. 管理端原生下拉已继续切到公共 AppSelect
- 已替换：
  - `frontend/src/pages/admin/SchedulerPage.tsx`
  - `frontend/src/pages/admin/CrawlerTasksPage.tsx`
  - `frontend/src/components/wecom/WecomTemplateManagerPanel.tsx`
  - `frontend/src/components/wecom/WecomRobotConfigPanel.tsx`
- 统一复用 `frontend/src/components/ui/app-select.tsx`
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`

### 78. 多个业务页分页已继续切到公共分页与公共下拉
- 已替换：
  - `frontend/src/pages/software_task/index.tsx`
  - `frontend/src/pages/brand_ops_center/index.tsx`
  - `frontend/src/pages/procurement_dept/index.tsx`
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `frontend/src/pages/SupplierAssistantPage.tsx`
  - `frontend/src/pages/marketing_dept/index.tsx`
- 列表底部“上一页 / 下一页”已统一改为 `frontend/src/components/ui/pagination-control.tsx`
- “每页条数”已统一改为 `frontend/src/components/ui/app-select.tsx`
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`

### 79. 推送记录管理已增加翻页
- `frontend/src/components/wecom/WecomMessageLogsPanel.tsx`
- 已接入后端原有 `page / page_size / total` 分页参数
- 列表底部已增加总数、页码、公共分页组件
- 每页条数已改用公共 `AppSelect`
- 默认每页 20 条
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`

### 80. 旧部门树已整体迁到 OA 部门树并清空
- 新增脚本：`backend/scripts/merge_legacy_departments_to_oa.py`
- 已执行 test / prod：
  - 旧部门树中的 `users / users2 / knowledge_bases / cost_items` 已迁到 OA 同步部门
  - 旧部门树节点已删除
- 执行结果：
  - test：迁移 `users=11`、`users2=95`、`knowledge_bases=5`、`cost_items=1300`，删除旧部门 `39`
  - prod：迁移 `users=11`、`knowledge_bases=5`、`cost_items=1300`，删除旧部门 `39`
- 已校验：
  - `departments.oa_department_id IS NULL = 0`
  - 已不存在仍挂旧部门的用户 / 知识库 / 造价数据

### 81. 计划任务列表已增加分页与手动筛选
- `frontend/src/pages/admin/SchedulerPage.tsx`
- 已增加：
  - 公共分页组件 `PaginationControl`
  - 每页条数公共下拉 `AppSelect`
  - 搜索按钮
  - 重置按钮
- 筛选项改为手动触发，不再输入即生效
- 任务名称筛选框宽度已缩小

### 82. 软件部新建任务负责人多选已切到公共三方组件
- 新增 `frontend/src/components/ui/app-multi-select.tsx`
- 基于现有依赖 `react-select` 封装公共多选下拉
- 支持：
  - 搜索
  - 勾选多选
  - 已选项标签展示
  - 单项移除 / 一键清空
- `frontend/src/pages/software_task/index.tsx`
  - 软件部“新建任务”中的“负责人（软件部）”已改为复用 `AppMultiSelect`
  - 软件部“新建任务”中的“所属项目”已从手写搜索弹层改为复用 `AppSelect`
  - 保留原有 `assigneeIds` 多选提交逻辑，以及 `assigneeId` 同步首个负责人的兼容逻辑
- `doc/frontend-common-components.md` 已补充公共多选下拉使用说明
- 后续补充：
  - `app-select` / `app-multi-select` 的焦点边框与高亮色已统一改为青色，和页面其他输入框一致
  - `app-select` / `app-multi-select` 默认高度已从 `44px` 调整为 `40px`，与旁边常规输入框一致

### 83. 已回退上一轮误改的跨页面下拉替换
- `frontend/src/pages/auto_ctrl/index.tsx`
- `frontend/src/pages/brand_ops_center/index.tsx`
- `frontend/src/pages/software_task/index.tsx`
- 处理内容：
  - 撤回误扩散到 `自控部 / 品牌运营中心` 的下拉改造
  - 撤回 `software_task` 中误带上的 `Bug` 筛选、`Bug 编辑`、`区域项目重复率` 下拉改造
  - 保留此前已确认的软件部公共组件改造，不回退已确认需求
- 已用 `Node v20.20.2 / npm 11.16.0` 重新通过 `npm run build`

### 84. 搜索按钮与日期组件继续统一
- 已继续把系统内残留的蓝色“搜索 / 查询”按钮切为黑色系，补齐业务页与管理页零散样式
- 已继续清理残留原生 `input[type=date]`，统一改为公共日期组件 `frontend/src/components/ui/date-picker-input.tsx`
- 本轮补齐页包括：
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `frontend/src/pages/brand_ops_center/index.tsx`
  - `frontend/src/pages/auto_ctrl/index.tsx`
  - `frontend/src/pages/admin/SchedulerPage.tsx`
  - `frontend/src/pages/admin/TechProjectStaffBoardPage.tsx`
  - `frontend/src/components/wecom/WecomRobotConfigPanel.tsx`
  - `frontend/src/components/wecom/WecomTemplateManagerPanel.tsx`
- 已用 `Node v20.20.2 / npm 11.16.0` 再次通过 `npm run build`
- 本轮未新增依赖

### 85. 软件部任务工具已隐藏统计 Tab
- `frontend/src/pages/software_task/index.tsx`
- 软件部入口主 Tab 已隐藏 `统计`
- 给排水入口仍保留原有统计 Tab，不受本次影响

### 86. OKCIS 每日推送默认时间已改为 16:10
- `backend/app/tasks/scheduler.py`
- 内置任务 `okcis_daily_summary_push` 默认 `CronTrigger` 已从 `18:00` 改为 `16:10:00`
- 与数据库中的 `scheduler_job_meta.cron_expr = 0 10 16 * * *` 保持一致

### 87. OKCIS 推送链接已改为固定当天日期
- `backend/app/tasks/scheduler.py`
- 招投标机器人推送卡片链接已从 `?date_from=today` 改为 `?date_from=YYYY-MM-DD`
- 点击历史推送消息时，会保持推送当日筛选结果，不再随当前日期漂移

### 88. 爬虫任务详情已支持执行日期传参
- `frontend/src/pages/admin/CrawlerTaskDetailPage.tsx`
- `backend/app/api/endpoints/admin/crawler_tasks.py`
- 爬虫任务详情页执行按钮左侧已增加“执行日期”选择器
- 手动执行接口已支持 `run_date`
- 运行时会把日期注入模板变量替换链路，当前主要覆盖 `{{today}}`
- 不填仍默认当天；仅模板本身支持日期变量的任务才会生效
- 已通过：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py`
  - `frontend npm run build`

### 89. OKCIS 自动登录链路已修复丢失登录检测配置问题
- `backend/app/api/endpoints/admin/crawler_tasks.py`
- 已修复刷新凭证后 `credentials.json` 丢失 `check_login_curl` 的问题
- `_persist_refreshed_site_credentials` 现会保留原有 `login_curl / check_login_curl`
- 手动刷新站点凭证接口也会保留既有 `check_login_curl`
- 未登录识别补充支持：
  - `请登录后查看`
  - `nologin`
- 自动刷新凭证后如果仍返回未登录内容，任务会直接报错，避免继续记为 `success`
- 已通过：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py`

### 90. 后台手动执行已可绕过测试环境 prod 限制
- `backend/app/tasks/scheduler.py`
- 后台手动点击执行时，以下任务已允许在 test 环境执行：
  - `software_work_record_reminder`
  - `okcis_daily_summary_push`
- 定时 cron 触发仍保持仅 `prod` 环境执行
- 已通过：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/tasks/scheduler.py`

### 91. 计划任务调度计划文案已改为更直白中文
- `frontend/src/pages/admin/scheduler-utils.ts`
- 已优先按 `cron_expr` 生成更通俗的展示文案
- 当前已覆盖常见场景：
  - 每分钟执行一次
  - 每 N 分钟执行一次
  - 每小时 X 分执行一次
  - 每天 HH:mm / HH:mm:ss 执行
  - 每周 X HH:mm 执行
  - 每月 X 日 HH:mm 执行
- 已通过 `frontend npm run build`

### 92. OKCIS 截止时间提取规则已改为取最近截止时间
- `backend/app/api/endpoints/admin/crawler_tasks.py`
- 新增识别字段：
  - `报名起止时间`
- 单个文本里若包含多个日期时间（如起止区间），当前按区间结束时间处理
- 多个截止类字段并存时，当前会取最近的截止时间
- 例如：
  - `报名起止时间：2026-07-02 09:00:00 至 2026-07-08 16:00:00`
  - `截止时间：2029-06-30`
  - 最终取 `2026-07-08 16:00:00`
- 已通过：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py`

### 93. 爬虫主流程已开始按 handler 结构拆分
- 新增：
  - `backend/app/services/crawler_handlers/base.py`
  - `backend/app/services/crawler_handlers/okcis.py`
  - `backend/app/services/crawler_handlers/__init__.py`
- `backend/app/api/endpoints/admin/crawler_tasks.py`
  - 主流程已接入 `get_crawler_task_handler(task)`
  - 主流程只保留公共执行链路与默认组件
  - `okcis` 个性化逻辑已改为独立 handler 承接
- 当前 `okcis` handler 已承接：
  - 运行前清理旧数据
  - dzid 目标生成
  - 详情页增强
  - 空结果判断
  - 公告业务表写入
- 已通过：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/app/services/crawler_handlers/base.py backend/app/services/crawler_handlers/okcis.py backend/app/services/crawler_handlers/__init__.py`

### 68. 单用户企微推送已抽成公共模板发送能力
- 新增 `backend/app/services/wecom_template_sender.py`
- `backend/app/services/wecom.py` 已新增 `send_single_user_message_by_template`
- 后续业务模块可统一按 `template_id + touser + template_vars` 发单用户企微消息，不再各处重复拼装模板

### 69. 模板管理列表已补筛选与快捷状态切换
- 前端模板管理已新增模板类型筛选
- 列表最前面已增加 `ID` 列
- 状态列已支持直接点击启用 / 停用快捷切换
- 后端已新增 `/api/admin/wecom-message-templates/{id}/toggle-enabled`

### 70. 推送机器人管理已补快捷状态切换
- 机器人管理列表状态列已支持直接点击启用 / 停用
- 后端已新增 `/api/admin/wecom-management/robots/{id}/toggle-enabled`

### 71. 计划任务页已修复爬虫任务执行记录不显示
- 原因不是没执行，而是计划任务管理页后端一直查 `scheduler_job_runs`
- 爬虫任务真实执行记录写在 `crawler_task_runs`
- 现已改为爬虫任务优先读取 `crawler_task_runs`，计划任务详情也同步修正

### 72. 计划任务列表已支持编辑标题和描述
- 列表新增“编辑”按钮
- 可直接修改计划任务标题与描述
- 新增持久化表 `scheduler_job_meta` 保存自定义标题/描述，避免重启后丢失

### 73. kehu51 客户跟进任务总条数改为页面脚本 recordCount
- `backend/crawler_sites/kehu51/request.json`
- 通过 GET 页面 `https://s25.kehu51.com/App/customers/follow/index.aspx?ShowID=1&ViewName=allfollow&action=`
- 读取页面脚本中的 `recordCount` / `recordCountChange(...)`
- 已补 `prefetch.method=GET`，并把 `referer` 调整为 `https://s25.kehu51.com/App/#customers-follow`
- `backend/crawler_sites/kehu51/credentials.json` 已补 `check_login_curl`，登录失效时可先检测并自动刷新
- 已修复运行时 prefetch 头部拼装问题：GET prefetch 不再混入主请求 POST 头，避免页面请求报 500

### 66. 企微模板已区分机器人模板与单用户推送模板
- 后端 `wecom_message_templates` 新增字段 `template_type`
- 历史模板统一归类为 `single_user`
- 推送机器人下拉仅展示 `group_robot` 模板
- 机器人接口新增校验：不能关联单用户推送模板

### 65. 招投标推送链接已带当天筛选参数
- `backend/app/tasks/scheduler.py`
- 推送链接改为 `/apps/okcis-notices?date_from=today`
- 点击消息后默认只看当天及以后数据

### 64. 本机 Python 环境约定已固定为 conda smart
- `AGENTS.md`
- 已补充本机默认 Python 执行环境：`conda activate smart`
- 后续 Python、FastAPI、Alembic、脚本任务默认优先使用该环境

### 63. 招投标每日推送已增加空数据不推送
- `backend/app/tasks/scheduler.py`
- 当天无新增招投标数据时，直接跳过群消息发送
- 调度记录仍会写成功结果，避免误判失败

### 62. 软件部日报提醒已增加空名单不推送
- `backend/app/tasks/scheduler.py`
- 若昨日无未填写日报人员，则直接跳过群消息发送
- 调度记录仍会写成功结果，避免误判失败

### 61. 招标信息公示改为仅推送链接按当天起筛选
- `frontend/src/pages/OkcisNoticesPage.tsx`
- 普通访问不再默认带“发布日期开始=当天”
- 仅当 URL 带 `date_from` 参数时才注入默认筛选
- 支持推送链接使用 `date_from=today`
- 重置后恢复为链接参数对应的默认值

### 60. 公共日期组件已落库到前端组件层
- 新增 `frontend/src/components/ui/date-picker-input.tsx`
- 基于现有 `react-day-picker`、`Calendar`、`Popover`、`Button` 统一封装
- 支持清空、最小日期限制、统一样式
- `frontend/src/pages/OkcisNoticesPage.tsx` 已切换为复用公共日期组件
- `AGENTS.md` 已补充规范：后续新增日期筛选默认走公共组件，不再在业务页重复手写

### 59. 招标信息公示日期筛选已切换到成熟第三方日期组件
- `frontend/src/pages/OkcisNoticesPage.tsx`
- 已去掉浏览器原生 `input[type=date]`
- 改为基于现有依赖 `react-day-picker` 的弹层日历选择
- 支持开始/截止日期选择、截止日期最小值约束、清空当前日期
- 前端构建已通过

### 58. 企微历史模板已迁移到新模板配置表
- `backend/scripts/backfill_wecom_group_robot_from_env.py` 已补齐旧版企微模板回填逻辑
- 统一写入新表 `wecom_message_templates`
- 已覆盖历史模板：
  - 软件部日报群提醒
  - 软件部任务通知文本/卡片
  - 给排水任务通知文本/卡片
  - 自控部任务通知文本/卡片
  - 采购发票提醒文本/卡片
- 旧版 `WECOM_GROUP_WEBHOOK_URL` 也已同步回填到 `wecom_message_robots`
- `backend/app/services/wecom.py` 已改为优先从 `wecom_message_robots` 读取机器人配置，旧 env webhook 仅保留兜底
- 已在 test / prod 两套环境执行回填
- 后续已补新增模板 `okcis_notice_daily_group_robot`：
  - 模板名称：`招投标企业推送模板`
  - 业务模块：`okcis`
  - 发送渠道：`group_robot`
  - 消息类型：`markdown`
- 前端推送机器人管理列表已增加 `ID` 列，便于直接查看机器人主键
- 群机器人推送已支持按机器人配置直接发送：
  - 调用链路改为 `robot_id -> wecom_message_robots.template_id -> wecom_message_templates.content`
  - `send_group_robot_markdown` 可直接传 `robot_id` 与模板变量，不再需要手动写死 webhook 和正文
- 已新增招投标每日情报推送任务 `okcis_daily_summary_push`
  - 固定机器人 ID：`2`
  - 推送模板来源：机器人关联模板
  - 调度时间：每日 `18:00`
  - 仅 `prod` 环境实际发送

### 55. OA 同步已放开 STATUS=0 人员，并完成 test/prod 双环境执行
- `backend/app/services/oa_sync.py` 已调整为只要求 `LOGINID` 非空，不再限制 `hrmpinyinresource.STATUS = 1`
- 已手动执行 OA 用户部门同步脚本到两套环境
- 测试环境本次结果：`users_total=274`、`users_created=55`、`users_updated=49`
- 正式环境本次结果：`users_total=274`、`users_created=51`、`users_updated=53`
- 已确认 `毛丽恒 / maoliheng / oa_resource_id=748` 已同步入库：
  - test：`users.id=302`
  - prod：`users.id=251`

### 56. 给排水部助手新建任务负责人下拉已去掉 is_active 过滤
- `backend/app/api/endpoints/software_task/software_task.py`
- `/software-task/assignable-users` 不再限制 `users.is_active = true`
- 现在给排水负责人下拉仍按“研发中心 > 给排水部”部门树过滤，但会返回该部门树内所有用户

### 57. 给排水负责人下拉已修复同名部门树导致漏人
- 排查确认本地存在两套 `研发中心 / 给排水部`：
  - 老树：`001.002.023`
  - 新树：`316.318`
- 之前接口只取第一条 `研发中心 > 给排水部`，导致挂在另一套部门树下的用户不显示
- 现已改为合并所有命中的 `研发中心 > 给排水部` 部门树再查用户
- 已验证 `贠丹丹`、`黄树才` 都会出现在负责人候选中

### 54. 售后部（客户无忧）导出预览样式已修复独立页不生效
- 排查确认实际入口是独立页 `/apps/after-sales-customer-follow`
- 该页复用了 `CustomerFollowPanel`，但 Excel 预览样式此前仅定义在售后部助手页面容器内
- 现已把预览横向展示相关样式下沉到 `CustomerFollowPanel` 组件内部
- 独立页里的“导出预览”现已与原页面保持一致

### 53. 给排水部助手统计页已新增区域项目重复率图表
- 新增独占一行图表“区域项目重复率”
- 顶部增加业务区域下拉筛选，样式复用任务大厅筛选组件
- 区域下拉已改为只展示该图中有数据的区域，并按项目数量倒序
- 统计口径改为按“项目编号”判断项目次数，图上显示项目名
- 统计口径已对齐任务大厅“项目编号”字段，读取给排水任务 `title`
- 单次项目也纳入统计，不再仅显示大于 1 次的项目
- 左侧项目名称已改为最多两行，超出省略，避免文字重叠
- Tooltip 展示项目编号与总出现次数

### 52. 企微推送记录已落独立表并拆成可复用功能组件
- 后端新增 `wecom_message_logs` 表模型、发送日志写库服务 `create_wecom_message_log`
- `WeComService` 发送文本 / markdown / textcard / 群机器人消息时，会自动写入发送成功或失败记录
- 后端已新增管理接口 `GET /api/admin/wecom-message-logs`
- 前端已新增页面 `/platform/admin/wecom-message-logs`
- 列表主体已单独拆成公共组件 `frontend/src/components/wecom/WecomMessageLogsPanel.tsx`
- 公共请求已拆到 `frontend/src/api/wecom-message-logs.ts`
- 已同步创建 test / prod 两套环境数据表
- 已修正迁移脚本，移除 `created_by` 外键，避免与线上 `users.id` 类型不兼容

### 19. kehu51 跟进结果异常数据已同步修复到测试和正式
- 新增 `backend/scripts/fix_kehu51_follow_results.py`
- 已同步修复 `crawler_kehu51_follow_records` 中 `follow_result` / `raw_json.跟进结果` 被截断为 `<span style=\` 的异常数据

### 20. kehu51 跟进结果无法恢复的脏数据已清空
- 已执行测试与正式双环境清理
- 无法恢复的异常字段已清空，避免页面继续显示脏内容

### 21. okcis 自动登录已修复验证码 403 与请求头大小写问题
- 修复 `backend/app/services/crawler.py`
- 自动登录已改为先访问 `/login/` 获取最新 cookie，再请求验证码和登录接口
- 验证码算式解析已修正

### 22. okcis 相关改动已纳入版本管理
- 已纳入站点目录、SQL、迁移、自动登录与执行链路改动
- 本次依赖增加过 `Pillow==11.3.0`

### 23. okcis 手动执行已补开始日志与空结果日志
- 执行前输出 `[START]`
- 每个 `dzid` 输出 `[DZID]`
- 空结果输出 `[EMPTY]`

### 24. 爬虫任务详情页已修复执行成功却误报失败的问题
- `frontend/src/pages/admin/CrawlerTaskDetailPage.tsx` 已调整执行提示逻辑
- 正式执行后的详情刷新失败会单独提示

### 25. okcis 演示模式预览已裁剪大字段
- `detail_content` 等超长字段会在演示返回时截断

### 26. okcis 每次执行前都会先清理不符合未来四天规则的旧数据
- 新增 `_cleanup_okcis_notice_table`
- 启动任务前先清理 `crawler_okcis_notices`

### 27. kehu51 客户列表客户名称已统一去除三角标记
- `backend/crawler_sites/kehu51/formatter.py` 已在统一格式化阶段对 `客户名称` / `customer_name` 执行清洗

### 28. 首页已新增招标信息公示卡片
- 已新增后端接口 `/api/okcis/notices`，读取 `crawler_okcis_notices`
- 首页应用中心已增加“招标信息公示”卡片与列表预览
- 已增加独立页面 `/apps/okcis-notices` 用于查看完整公告列表

### 29. 软件部任务工具工作记录默认筛选与填写人筛选已修复
- 非部门管理员查看工作记录时，后端默认只返回当前用户自己的记录
- 前端工作记录“填写人”筛选已改为读取业务人员列表，并修复筛选参数生效问题
- 非管理员进入工作记录页时，会自动锁定为本人记录

### 30. 招标信息公示页已增加截止时间与地区筛选
- `frontend/src/pages/OkcisNoticesPage.tsx` 已新增截止时间区间筛选
- 已新增地区下拉筛选，选项来自现有数据
- 类型字段已隐藏，不再在列表中展示

### 31. 招标信息公示已增加跟进状态字段与操作列
- `crawler_okcis_notices` 新增 `is_followed` 字段
- 后端已新增跟进状态切换接口
- 前端列表已增加“跟进 / 取消跟进”操作列

### 32. okcis 订阅组 dzid 已接入 Redis 30 天缓存
- 执行 `okcis` 任务前会请求 `dz_guanli/index.php?return_json=1`
- 从 `myConditions` 提取 `item[0]` 作为 `dzid`
- 已写入 Redis，key 为 `crawler:okcis:dingzhi_groups`，缓存 30 天
- 远端获取失败时，会回退使用 Redis 缓存，再回退到本地配置 `dzid_list`

### 33. okcis 公示列表已兼容未执行 is_followed 迁移的数据库
- `/api/okcis/notices` 在字段不存在时自动降级返回 `is_followed=0`
- 避免因正式库未补字段导致页面 500
- 跟进切换接口仍要求先执行数据库迁移

### 34. okcis 跟进字段已同步到测试和正式环境
- 已对 `smart-cs-ai-test` 与 `smart-cs-ai` 执行 `crawler_okcis_notices.is_followed` 补字段
- 招标信息公示列表与“跟进 / 取消跟进”接口可正常使用

### 35. okcis 跟进接口 400 已补齐测试环境字段
- 排查确认 `test` 库之前仍缺少 `crawler_okcis_notices.is_followed`
- 已补字段并校正默认值
- `prod` 库原本已存在该字段

### 36. 招标信息公示页已补齐筛选与交互
- 已新增订阅组下拉筛选
- 已新增跟进状态下拉筛选
- 筛选改为点击“搜索”后生效
- 已增加“重置”按钮
- 标题列宽度已按反馈调整

### 37. 软件部工作记录“填写人”筛选已改为默认本人但可清空查看其他人
- 后端不再按权限强制限制只能看本人
- 前端默认带本人筛选值
- 填写人下拉始终加载人员列表
- 已支持清空当前填写人筛选

### 38. 爬虫任务与计划任务已实现即时双向同步
- 爬虫任务新增、编辑、删除后会立即同步到 APScheduler，不再只依赖每分钟同步任务
- 爬虫 cron 任务关闭后会保留在计划任务列表中，并显示为暂停状态
- 在计划任务管理中暂停 / 恢复爬虫任务时，会反向更新 `crawler_tasks.is_enabled`

### 39. 调度执行日志已限制长度并过滤数据库噪音
- `scheduler_job_runs.output` 落库前会截断到安全长度
- 已过滤 `sqlalchemy.engine`、`aiomysql` 日志，避免调度记录被 SQL 明细撑爆
- 修复 `Data too long for column 'output'` 导致的调度任务误判失败

### 40. 爬虫计划任务已移除每分钟全量同步
- 删除 `Sync crawler cron jobs from database` 周期任务
- 改为服务启动后执行一次爬虫任务注册同步
- 后续任务新增、编辑、启停全部依赖即时同步链路
### 41. 采购部供应商风险详情已补 PrimeMatrix 限流兜底
- `PrimeMatrix HTTP 调用失败: 429` 不再导致接口未处理异常 500
- 有缓存时优先回退 `procurement_suppliers` 中已有风险缓存数据
- 无缓存时直接返回空风险状态

### 42. 采购部供应商风险详情弹窗已补空态展示
- 工商信息不为空且风险信息为空时，风险状态显示“正常”
- 工商信息为空且风险信息为空时，风险状态显示“未知”
- 工商信息不为空且风险信息有值时，风险状态显示“风险”
- `basic_info` 为空时显示“暂无工商信息”
- `risk_info` 为空时按场景显示“暂无风险信息”或“暂未获取到数据”

### 43. 采购部列表风险列与风险任务已补空状态规则
- 列表页 `risk_status` 为空且 `risk_count=0` 时，风险列显示 `-`
- 风险任务在未查询到风险数据时，会把 `procurement_suppliers.risk_status` 写为空字符串

### 44. 软件部新建工作记录已增加当日草稿
- 新增工作记录草稿读取/保存接口
- Redis key 规则包含 `用户ID + 日期`
- 草稿 TTL 截止当日 `23:59:59`
- 新建工作记录弹窗优先回填草稿中的项目、工时、工作内容
- 成功正式保存工作记录后，会自动删除对应日期草稿

### 45. 软件部工作记录草稿回填已修复当天日期偏差问题
- 新建工作记录弹窗原先使用 `toISOString().slice(0, 10)` 取“今天”
- 该写法会按 UTC 取日，导致东八区部分时段读取到前一天草稿 key
- 现已统一改为 `Asia/Shanghai` 当天日期，重新打开新建弹窗可正确回填草稿

### 46. 软件部工作记录草稿保存已增加写入校验
- 草稿保存接口现已在写入 Redis 后立即回读校验
- 若 Redis 未实际写入，不再假返回成功，而是直接返回保存失败提示

### 47. 软件部“存草稿”按钮提示已改为页面内悬浮提示
- 去掉依赖浏览器原生 `title` 的方式
- 鼠标移入“存草稿”按钮时，稳定显示“草稿仅保存当日数据”

### 48. 已新增企微首次登录同步说明文档与 SVG 流程图
- 新增 `doc/wecom-first-login-sync.md`
- 新增 `doc/wecom-first-login-sync.svg`
- 内容包含前端登录、后端落库、涉及表字段、当前限制与风险点
### 49. 已新增 OA 用户/部门每小时同步任务
- 数据源使用 OA 库 `hrmpinyinresource`、`hrmdepartmentallview`
- 新增本地映射字段：`users.oa_resource_id`、`departments.oa_department_id`
- 计划任务 `oa_user_department_sync` 每小时 `05` 分执行一次
- 支持后台计划任务手动执行与查看执行记录
 - 已过滤 OA 部门脏数据：忽略 `id<=0` 或部门名为空的部门记录
### 50. AGENTS 已补充本机 nvm 环境约定
- 允许在本机使用 `nvm` 管理 Node.js 版本
- 当前端 `npm` / 构建链异常时，默认先检查并切换合适的 Node 版本

### 51. OA 用户部门同步已补缺失字段并完成主流程验证
- 已对 test / prod 两套环境执行 SQL，补齐：
  - `users.oa_resource_id`
  - `departments.oa_department_id`
  - 对应唯一索引
- 已修复 OA 同步过程中的两个兼容问题：
  - `sso_uid` 按忽略大小写匹配历史账号，避免 `LiZhaoMin` / `lizhaomin` 这类重复插入
 - 已按新规则改为 OA 同步用户不分配任何角色，登录后默认无可见模块，后续手动分配角色
- 本地手动执行 `safe_sync_oa_departments_and_users()` 已成功：
  - `departments_total=64`
  - 最近一次复跑：`departments_created=0`
  - 最近一次复跑：`departments_updated=13`
  - `users_total=170`
  - 最近一次复跑：`users_created=0`
  - 最近一次复跑：`users_updated=0`

## 历史归档说明

- 完整历史已归档到 `PROJECT_HISTORY.archive.md`
- 后续当本文件再次明显变长时，继续只保留最近关键记录和当前摘要

### 52. 计划任务列表编辑已增加调度计划改动能力
- 计划任务管理页“编辑”弹窗已增加调度计划配置
- 前端支持编辑：
  - 调度类型：`cron` / `interval`
  - `cron` 表达式
  - 间隔分钟数
- 后端 `scheduler_job_meta` 已新增字段：
  - `trigger_type`
  - `cron_expr`
  - `interval_minutes`
- 普通调度任务保存后会直接调用 APScheduler `reschedule_job`
- 爬虫调度任务保存后会同步回写 `crawler_tasks.cron_expr` 并立即同步计划任务
- 已新增迁移：
  - `backend/migrations/versions/b2c3d4e5f6a7_add_scheduler_job_meta_schedule_fields.py`
- 本地校验结果：
  - 后端 `py_compile` 通过
  - 前端 `npm run build` 因本机 Node 环境异常失败：`Cannot find module 'node:path'`
- 本次未新增依赖包

### 53. scheduler_job_meta 调度计划字段已同步到测试和正式环境
- 已对 `smart-cs-ai-test` 与 `smart-cs-ai` 执行补字段：
  - `trigger_type`
  - `cron_expr`
  - `interval_minutes`
- 已现场核验两套环境 `information_schema.COLUMNS`，三列均已存在
- 计划任务列表 `/api/admin/scheduler/jobs` 因缺列导致的 500 已消除前置条件

### 54. 计划任务编辑 cron 表达式回显已修复
- 普通计划任务编辑时，之前只从 `scheduler_job_meta.cron_expr` 读值
- 对于历史任务，如果未手动保存过调度配置，弹窗里会出现 cron 表达式空白
- 现已改为优先读库，若为空则直接从 APScheduler 当前 `job.trigger` 反解析成标准 6 段 cron 表达式回显
- 后端语法校验已通过

### 55. okcis_daily_summary_push 手动执行入口已补齐
- `/api/admin/scheduler/jobs/okcis_daily_summary_push/run` 之前会命中默认分支返回“未找到任务”
- 已在计划任务手动执行接口中补接：
  - `run_okcis_daily_summary_push_job(run_trigger="manual")`
- 后端语法校验已通过

### 56. 软件部日报提醒已固定走机器人 ID=1
- `software_work_record_reminder` 之前虽已走机器人管理/模板链路，但未显式指定 `robot_id`
- 原逻辑会发送到“当前第一个启用的机器人”
- 现已改为固定传 `robot_id=1`
- 后端语法校验已通过

### 57. 软件部 / 自控部企业微信接收人解析已修复 invaliduser 问题
- 正式环境排查确认：
  - `users.id=102`
  - `nickname=张春嫣VIP3`
  - `sso_uid=ZhangChunYan`
  - 历史失败日志 `wecom_message_logs.id=17` 中实际发送 `touser=张春嫣`
- 根因是业务人员字段解析未命中本地用户后，直接把原始中文姓名当成企微 `userid` 兜底发送
- 已修复 `software_task` / `auto_ctrl` 两处解析逻辑：
  - 增加 `mobile` 精确匹配
  - 增加 `nickname/email like 'token%'` 的唯一匹配
- 仅当 token 符合企微 `userid` 字符规则时才允许兜底直发
- 中文姓名等非法 token 改为跳过并记录 warning，不再继续发送
- 后端语法校验已通过

### 58. 给排水任务通知 invaliduser:tengaizhe 已定位并修复停用用户发送问题
- 正式环境查询确认：
  - `users.id=137`
  - `sso_uid=tengaizhe`
  - `nickname=滕爱喆`
  - `is_active=0`
- 历史失败日志 `wecom_message_logs.id=7` 中发送 `touser=tengaizhe`，企业微信返回 `81013 invaliduser`
- 根因是业务人员解析虽然命中了本地用户，但未过滤 `users.is_active`
- 现已修复软件部 / 给排水 / 自控部共用解析逻辑：
  - 仅允许 `is_active=1` 用户参与精确匹配与模糊匹配
  - 停用用户不再被解析为企微接收人
- 后端语法校验已通过

### 59. 推送记录管理与用户列表已补排查字段
- `推送记录管理` 列表已新增 `ID` 列，便于按日志主键排查问题
- `用户管理 -> 用户列表` 已新增“状态”列，显示 `启用 / 停用`
- 本次为前端展示改动，未新增依赖

### 60. 公共下拉框迁移已完成前两批
- 公共组件 `AppSelect` 已补 `size="sm"`，用于分页条数等紧凑下拉
- `FilterableSelect` 已修复选中后再次点击下拉按钮仍可展示全部选项
- 第一批已替换：
  - `RoleEditor`
  - 业务学习模块的章节、选项、布尔值、试卷、错题、题库、考试、记录相关下拉
  - 计划任务列表筛选下拉
- 第二批已替换：
  - 供应商助手发票状态下拉
  - 售后部客户无忧 Excel 年份与新建设备年份下拉
  - 采购部发票、供应商、备件、合同分页条数下拉，以及备件分类筛选
  - 市场部全部合同、异常合同的部门筛选
- 已执行校验：
  - 第二批目标文件旧 `<select>` / 旧 `SelectTrigger` 扫描无残留
  - `frontend npm run build` 通过
- 后续待分批替换：
  - `software_task`
  - `brand_ops_center`
  - `auto_ctrl`

### 61. 公共下拉框迁移已完成剩余大页
- 第三批已替换 `software_task`
- 第四批已替换 `brand_ops_center`
- 第五批已替换 `auto_ctrl`
- 页面级旧 `<select>` / 旧 `SelectTrigger` 扫描已无残留，仅保留公共 `frontend/src/components/ui/select.tsx` 组件自身
- 每批均已执行 `frontend npm run build`，最终构建通过
- 本次未新增依赖

### 62. 已新增前端公共组件与样式规范文档
- 新增 `doc/frontend-common-components.md`
  - 约定 `AppSelect`、`FilterableSelect`、`PaginationControl`、`DatePickerInput`、横向滚动条 hook 的调用规范
  - 明确禁止新增原生 `<select>` 和业务页重复手写下拉弹层
- 新增 `doc/frontend-style-guide.md`
  - 统一页面色系为“清爽蓝绿 + 中性灰”
  - 约定筛选栏、表格、按钮、弹窗、状态标签样式
- `AGENTS.md` 已补入口：后续新增或改造前端页面必须优先遵循上述两份文档

### 63. 软件部任务工具任务大厅/工作记录旧自定义下拉已补漏
- 用户反馈“工作记录、任务大厅的下拉框都没换”
- 原因：这两处使用的是业务页手写输入框 + 自定义弹层，不属于 `<select>` / `SelectTrigger` 扫描范围
- 已改为公共 `AppSelect`：
  - 任务大厅：项目、业务区域、负责人、状态筛选
  - 工作记录：项目、填写人筛选
  - 新建工作记录：项目选择
- 已执行 `frontend npm run build`，构建通过
- 本次未新增依赖

### 64. 剩余分页已收尾，公共下拉去掉蓝色点击边框
- 用户要求继续处理剩余分页问题，并去掉点击后的蓝色边框样式
- 已处理页面：
  - `frontend/src/pages/auto_ctrl/index.tsx`
  - `frontend/src/pages/brand_ops_center/index.tsx`
  - `frontend/src/pages/software_task/index.tsx`
- 处理结果：
  - 残留“上一页 / 下一页”区块已统一改为公共 `frontend/src/components/ui/pagination-control.tsx`
  - `auto_ctrl` 残留“每页条数”已改为公共 `frontend/src/components/ui/app-select.tsx`
  - `frontend/src/components/ui/app-select.tsx` 已移除点击后的蓝色描边与蓝色阴影，焦点态改为灰边且无额外 glow
- 已执行校验：
  - 目标页面 `上一页 / 下一页` 关键字扫描无残留
  - 已使用 `Node v20.20.2 / npm 11.16.0` 通过 `frontend npm run build`
- 本次未新增依赖

### 65. OKCIS 演示任务已恢复抓取真实数据
- 2026-07-06 使用正式环境手动演示执行 `task_id=3 / okcis_notice_manual`
- 验证结果：
  - 登录态检查正常，日志显示 `is_nologin=False`
  - 演示执行汇总：成功 8，失败 0
  - 已抓到真实数据而非未登录占位数据
- 抽样结果：
  - `dzid=18117` 抓到 5 条
  - `dzid=44602` 抓到 5 条，详情中识别到 `响应截止时间: 2026-07-09 15:00:00`
  - `dzid=70940` 抓到 24 条
  - `dzid=157288` 抓到 50 条
- 说明：
  - 详情页仍有部分字段会被站点会员权限掩码为 `**** 权限不够，请升级会员`
  - 但当前链路已能稳定拿到列表真实数据，并可从部分详情摘要中抽取可用截止时间
- 本次未新增依赖

### 66. kehu51 客户跟进爬虫已拆到独立 handler
- 用户确认客户跟进记录任务有必要从主流程里拆分
- 已新增 `backend/app/services/crawler_handlers/kehu51.py`
- 已拆出的内容：
  - `kehu51_follow_records / customer_follow_records` 写库逻辑
  - `kehu51_customer_list` 写库逻辑
  - 增量模式下 `follow_time` 命中历史数据后的停止判断
- 主流程改为通过 handler 调用，不再在 `crawler_tasks.py` 中直接写客户跟进特例判断
- 已执行语法校验：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/services/crawler_handlers/base.py backend/app/services/crawler_handlers/kehu51.py backend/app/services/crawler_handlers/__init__.py backend/app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 67. kehu51 客户跟进任务已增加总条数一致性校验
- 用户要求：客户跟进记录任务必须校验“爬取数据条数”和“原始总条数”一致
- 已实现：
  - 正式执行时，先记录预取阶段识别到的源站 `recordCount`
  - 抓取过程中累计每页实际抓取条数
  - 任务结束后执行一致性校验
- 校验结果：
  - 一致时记录 `[COUNT-CHECK]`
  - 不一致时记录 `[COUNT-CHECK-FAIL]`，并将任务记失败
- 日志中的原因分析会带出：
  - 是否命中结束页
  - 是否增量模式提前命中历史 `follow_time`
  - 是否源站提前返回无更多数据
  - 是否格式化异常 / 空结果 / 执行异常 / 写库异常
  - 实际页数与按总条数推算页数差异
  - 少抓或多抓的具体条数
- 演示模式跳过强校验，避免“只抓几条预览数据”误报失败
- 已执行语法校验：
  - `/opt/anaconda3/envs/smart/bin/python -m py_compile backend/app/services/crawler_handlers/base.py backend/app/services/crawler_handlers/kehu51.py backend/app/api/endpoints/admin/crawler_tasks.py`
- 本次未新增依赖

### 68. 已新增 kehu51 客户跟进全量条数核对测试
- 已新增测试文件：`backend/tests/test_kehu51_follow_count_consistency.py`
- 覆盖内容：
  - 对 4 组给定 `ajaxParam` 提取人员 `targetID`
  - 校验 `customParam=2`、`targetField=UserID`
  - 提供实时全量测试：按人员筛选抓取全部跟进记录，累计每页条数，并与源站第一页返回的 `total` 对比
- 当前 4 组参数对应提取结果：
  - 范运成：`1680321`
  - 缴志健：`1442273`
  - 马超：`1442273`
  - 李秋雷：`1592326`
- 实时测试默认跳过，需显式设置环境变量：
  - `RUN_KEHU51_LIVE_TEST=1`
- 后续已补测试内自动重登：
  - 若实时请求返回“您已被迫退出 / Logout.aspx / nologin”等未登录内容
  - 测试会自动调用 `login_curl` 刷新一次 kehu51 凭证后重试
  - 避免因站点异地登录踢下线导致误判为“返回格式异常”
- 已执行测试：
  - `/opt/anaconda3/envs/smart/bin/python -m unittest backend.tests.test_kehu51_follow_count_consistency`
  - 结果：`Ran 2 tests ... OK (skipped=1)`
- 本次未新增依赖

### 69. kehu51 实时测试已改为先打印 4 人总条数
- 用户要求先在控制台打印范运成、缴志健、马超、李秋雷 4 人的爬取条数，不先跑全量明细
- 已调整 `backend/tests/test_kehu51_follow_count_consistency.py`
- 当前实时测试行为：
  - 仅请求第一页
  - 读取格式化后的 `total`
  - 对第一页数据继续执行“库里已存在则跳过，不存在则写入”
  - 这一写库流程已改为直接复用正式 `kehu51` 增量 handler，不再在测试里手写一套插入逻辑
  - 控制台输出：
    - `[KEHU51-COUNT] name=... targetID=... total=... page1_items=... db_inserted=... db_skipped=...`
- 已执行基础测试：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m unittest tests.test_kehu51_follow_count_consistency`
  - 结果：`Ran 2 tests ... OK (skipped=1)`

### 70. kehu51 增量参数 showID 已改为 2
- 用户要求把客户跟进增量参数中的 `showID` 改为 `2`
- 已修改：
  - `backend/crawler_sites/kehu51/request.json`
  - `backend/tests/test_kehu51_follow_count_consistency.py`
- 当前约定：
  - `1 = 全部`
  - `2 = 今天`
  - `4 = 本周`
  - `5 = 本月`
  - `6 = 最近30天`
- 已把该枚举说明写入测试文件注释与项目上下文文档

### 71. kehu51 测试脚本已增加只获得条数开关
- 用户要求给测试脚本增加“只获得条数”开关
- 已在 `backend/tests/test_kehu51_follow_count_consistency.py` 增加：
  - `COUNT_ONLY = 1`
- 当前行为：
  - `COUNT_ONLY=1`：仅请求第一页并打印条数，不执行增量写库
  - 为核对历史总量，`COUNT_ONLY=1` 时会强制按 `showID=1`（全部）统计
  - `COUNT_ONLY=0`：打印条数后，继续复用正式增量 handler 执行第一页数据入库
- 控制台输出已增加 `count_only=...` 字段，方便确认当前模式

### 72. kehu51 客户列表与客户跟进后台任务入口已验证可正常执行
- 修复客户列表后台任务入口：
  - 空 `json_body={}` 不再优先按 JSON 发送，避免覆盖表单参数导致源站按全客户 SQL 报 500
  - 预取和分页请求复用同一个 `httpx.AsyncClient`
  - 预取响应 cookie 会合并回站点凭证
  - 分页累计条数达到源站 `recordCount` 后立即停止，避免多请求下一页造成“多抓 1 条”
- 修复客户跟进后台任务入口：
  - 模板补齐 `targetID=3 / targetType=1 / targetField=UserID`
  - 避免源站 `FollowTools.aspx/GetFollowScrollData` 返回 `NullReferenceException`
- 已执行后台任务入口验证：
  - 客户列表演示：成功 `4`，失败 `0`
  - 客户跟进演示：成功 `1`，失败 `0`
  - 客户列表正式：成功 `15`，失败 `0`
    - 范运成：`199/199`
    - 缴志健：`308/308`
    - 马超：`74/74`
    - 李秋雷：`86/86`
  - 客户跟进正式：成功 `1`，失败 `0`，`source_total=23 / crawled_total=23`
- 已执行语法校验：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/crawler_tasks.py app/services/crawler.py`
- 本次未新增依赖

### 73. kehu51 运行时凭证文件改为忽略
- `backend/crawler_sites/kehu51/credentials.json` 包含运行时 cookie / 登录态信息，不再纳入版本管理
- 已加入 `.gitignore`
- 已执行 `git rm --cached backend/crawler_sites/kehu51/credentials.json`
- 本地文件保留，仅从 Git 索引移除，避免后续 cookie 刷新反复出现在提交里

### 123. 业务团队管理看板样式继续收口
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
- 调整：
  - 最近跟进卡片改为固定高度内滚动展示
  - 跟进日期改为中文月日格式，例如 `7月6日`
  - 编辑弹窗表格表头改为吸顶冻结
  - 图表标题文案调整为：
    - `个人全年完成率`
    - `个人季度完成率`
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 && npm run build`
- 本次未新增依赖

### 124. 售后OA合同回款报表同步增加项目来源人归属规则
- 文件：
  - `backend/app/models/oa_contract_report.py`
  - `backend/scripts/sync_oa_contract_payment_report.py`
  - `backend/sql/create_oa_contract_payment_report_rows.sql`
  - `backend/sql/alter_oa_contract_payment_report_add_project_source_person.sql`
  - `backend/sql/upsert_oa_contract_payment_report_scheduler_meta.sql`
  - `backend/app/api/endpoints/admin/scheduler.py`
- 调整：
  - 新增字段 `project_source_person`
  - 已确认 Excel `htdjb1.xlsx` 的“项目来源人”对应 OA 字段 `xmlyr`
  - 当合同业务员为 `李春梅 / 李玉敏 / 毛丽恒` 时，按代理业务员处理
  - 同步写库时不再归属到代理业务员名下，改为归属到 `项目来源人`
  - 若代理业务员记录的 `项目来源人` 为空，则本次直接跳过，等待后续同步补齐
  - 计划任务标题改为 `售后OA合同回款报表同步`
  - 同步范围再收紧为仅保留 `合同编号日期 >= 2026-01-01`
  - 同步入口从 `售后服务部(天津)` 改为按 OA 部门 `深耕组 / 开拓组` 抽取
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/oa_contract_report.py app/api/endpoints/admin/scheduler.py scripts/sync_oa_contract_payment_report.py`
- 本次未新增依赖

### 125. AGENTS 增加导入校验强制规则
- 文件：
  - `AGENTS.md`
- 调整：
  - 若代码改动新增了包引用、模块导入或调整导入方式，完成后必须自动执行导入校验
  - Python 至少执行相关文件 `py_compile` / import 级校验
  - 前端至少执行一次 `npm run build` 或等效编译校验
- 本次未新增依赖

### 126. kehu51 客户名称增加前缀序号清洗
- 文件：
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/sql/normalize_kehu51_customer_list_customer_name_prefix.sql`
- 调整：
  - 新增客户名称标准化规则，去掉前缀序号
  - 示例：`1 天津中环领先材料技术有限公司 -> 天津中环领先材料技术有限公司`
  - 客户列表查重、入库、更新统一使用标准化后的客户名称
  - 已补历史数据清洗 SQL
  - 复查后保留纯数字/数字空格类客户名，例如 `1218 9988`，视为原始编号，不做误清洗
- 本次未新增依赖

### 125. 软件部日报提醒部门配置修正
- 文件：
  - `backend/app/core/config.py`
  - `backend/config/env_test`
  - `backend/config/env_prod`
- 调整：
  - `SOFTWARE_TASK_ASSIGNEE_DEPARTMENT_ID` 从 `3` 改为 `63`
  - 原因：正式环境 `软件开发部` 本地部门 `id=63`，原配置 `3` 实际对应不存在的本地部门，导致日报提醒统计范围为空
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py`
- 环境变量同步：
  - test：已改
  - prod：已改
- 本次未新增依赖

### 126. 工作台卡片配置落库与管理后台
- 文件：
  - `backend/app/api/api.py`
  - `backend/app/api/endpoints/dashboard_app_cards.py`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/components/dashboard/AppGrid.tsx`
  - `frontend/src/pages/mobile/dashboard/index.tsx`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `frontend/src/pages/admin/components/AdminModuleSwitcher.tsx`
  - `frontend/src/pages/admin/AdminDeckPage.tsx`
  - `frontend/src/App.tsx`
- 调整：
  - 新增表 `dashboard_app_cards`，将工作台 / 移动端看板卡片配置落库
  - test / prod 已执行建表 SQL，并初始化当前 15 张卡片
  - 新增公共读取接口 `GET /api/dashboard-app-cards`
  - 新增管理后台接口 `/api/admin/dashboard-app-cards`
  - 新增管理后台页面 `/platform/admin/dashboard-app-cards`
  - 工作台桌面端与移动端首页优先读取数据库配置，接口异常时兜底静态配置
  - 新建卡片表单已简化为只填：名称、描述、排序
  - 后端创建卡片时自动生成 `app_key`，并补齐图标、路径、状态、分类、权限等默认字段
  - 编辑卡片仍保留完整配置字段
  - 编辑卡片交互已优化：
    - 图标 key 改为可视化图标网格选择
    - 图标选择、图标颜色均改为二级弹窗选择，避免撑长编辑表单
    - 图标颜色改为预设色板选择，已去掉类名输入
    - 权限改为公共多选组件
    - 状态、分类改为公共下拉组件
    - 唯一 key 已从后台编辑表单移除，创建时由后端自动生成
    - 默认路径、桌面路径、移动路径已从后台编辑表单移除，沿用自动生成 / 已有配置
    - 状态样式已从后台编辑表单移除，跟随状态自动设置
  - 规划中卡片已补回表内，test / prod 当前均为：管理员 1、运行中 14、规划中 16
  - 补规划中卡片使用 upsert，不删除正式环境已有卡片
  - 修复后端启动报错：`deps.get_current_active_user` 改为项目现有的 `deps.get_current_user`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/api.py app/api/endpoints/dashboard_app_cards.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -c "import app.main; print('app import ok')"`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 127. 工作台卡片管理增加筛选搜索与分页
- 文件：
  - `backend/app/api/endpoints/dashboard_app_cards.py`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `CURRENT_CONTEXT.md`
- 调整：
  - 管理列表筛选行改为：标题、分类、启用状态
  - 分类、启用状态筛选改用公共下拉组件
  - 筛选项变更后不自动查询，需点击“搜索”
  - 增加“重置”按钮，可清空筛选并回到第一页
  - 列表增加公共分页组件与每页条数选择
  - 后端管理列表接口支持 `title/category/is_enabled/page/page_size`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/api.py app/api/endpoints/dashboard_app_cards.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 128. 售后动态爬取人员新增王贺
- 文件：
  - `backend/app/services/after_sales_team.py`
  - `backend/app/services/kehu51_employees.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/crawler_sites/kehu51/request.json`
  - `backend/sql/create_after_sales_management_dashboard_members.sql`
  - `backend/sql/insert_after_sales_dashboard_new_member_test_data.sql`
  - `backend/sql/upsert_after_sales_dashboard_member_wang_he.sql`
  - `CURRENT_CONTEXT.md`
- 调整：
  - 查到王贺在 kehu51 员工接口中的 `userID=1555551`
  - OA 中王贺为 `王贺VIP1`，真实部门为 `售后服务部(生态城)`
  - 售后业务模块增加动态虚拟组覆盖：王贺展示/统计归属 `深耕组`，不影响真实组织架构
  - 动态人员缓存 key 升级到 `after_sales:business_team_members:v2`
  - kehu51 售后人员匹配缓存 key 升级到 `kehu51:after_sales_business_team_users:v2`
  - 客户列表落库时，负责人为王贺则 `department_name` 强制写入 `深耕组`
  - 爬虫后台运行参数支持 `payload.user_id` 注入模板，并支持 `payload.page_start/page_end` 覆盖任务页码
  - 客户跟进搜索模板 `UserID` 改为 `{{user_id}}`，不传时仍为空，传王贺 ID 时可单独补跑王贺
  - 管理后台按 `user_id` 补跑客户列表时，只执行匹配该 ID 的负责人 target
  - 看板成员默认数据与新环境种子 SQL 均补入王贺
- 已同步数据：
  - test / prod 均已执行 `backend/sql/upsert_after_sales_dashboard_member_wang_he.sql`
  - test / prod 均已补跑王贺客户列表：源站 `14` 条，落库有效客户 `14` 条，部门均为 `深耕组`
  - test / prod 均已补跑王贺客户跟进：`UserID=1555551`，日期范围 `2000-01-01` 到 `2026-07-08`，源站 `96` 条，抓取 `96` 条，条数校验一致
  - 当前 test / prod 库内 `creator_name=王贺` 跟进记录总数均为 `108` 条，包含历史已有记录
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/after_sales_team.py app/services/kehu51_employees.py app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py app/api/endpoints/admin/crawler_tasks.py`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m json.tool crawler_sites/kehu51/request.json`
- 本次未新增依赖

### 129. 售后个人排名改为完成额倒序
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 业务团队管理看板“个人排名”排序字段改为“完成额”倒序
  - 完成额相同时再按完成率倒序
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 130. 售后个人排名取消内部滚动条
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 业务团队管理看板“个人排名”区域取消内部滚动条
  - 高度从固定 `495px` 调整为最小高度 `540px`，用于完整展示当前人员排名
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 131. 售后小组战况高度与王贺合同同步
- 文件：
  - `frontend/src/pages/after_sales_management_dashboard/index.tsx`
  - `backend/scripts/sync_oa_contract_payment_report.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 小组战况表格容器高度固定为 `182px`
  - OA 合同金额同步脚本增加王贺业务例外：
    - OA 源合同中业务员为 `王贺VIP1`
    - 源部门为 `售后服务部(生态城)`
    - 本地合同镜像表按售后业务虚拟组写入 `深耕组`
  - 不扩大同步整个生态城部门，只纳入王贺个人合同
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/sync_oa_contract_payment_report.py --both`
  - test / prod 均写入 `816` 行合同镜像数据
  - 王贺 2026 合同：`20` 个，完成额 `68.75 万`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile scripts/sync_oa_contract_payment_report.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 132. 恢复客户无忧客户列表爬取任务显示
- 文件：
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/sql/update_kehu51_customer_list_task_to_api.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - test / prod 两套环境 `crawler_tasks` 中 `task_key=kehu51_customer_list` 均存在
  - 任务 `id` 均为 `2`
  - 原任务名称为 `kehu51客户列表`，后台看起来不直观，已统一为 `客户无忧客户列表`
- 调整：
  - `backend/sql/update_kehu51_customer_list_task_to_api.sql` 从单纯更新改为 upsert
  - 后续即使任务被误删，执行该 SQL 也能自动补回
  - 客户无忧客户列表任务 `run_mode` 从 `manual` 改为 `cron`
  - 补入 `scheduler_job_meta`：按 `crawler_tasks.id` 动态生成 `job_id`，当前两套环境为 `crawler_task_2`，标题 `客户无忧客户列表`、业务部门 `售后服务部`
  - 计划任务列表接口补充展示尚未注册到当前内存 scheduler 的 cron 爬虫任务
  - 爬虫计划任务即使尚未注册到当前内存 scheduler，也支持打开详情和立即执行
- 当前两套环境配置：
  - `site_key=kehu51`
  - `task_type=api`
  - `template_name=customer_list.json`
  - `run_mode=cron`
  - `is_enabled=1`
  - `sync_mode=incremental`
  - `cron_expr=15 * * * *`
  - `page_start=1`
  - `page_end=-1`
  - `page_size=50`
  - `scheduler_job_meta.title_zh=客户无忧客户列表`
  - `scheduler_job_meta.business_department=售后服务部`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/update_kehu51_customer_list_task_to_api.sql`
  - 已再次只读查询 test / prod，确认任务均存在
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 133. 新增售后工单监控卡片与实时语音提醒
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/models/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/sql/create_after_sales_work_order_monitor.sql`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/migrations/versions/j0k1l2m3n4o5_add_after_sales_work_order_monitor_accounts.py`
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `frontend/src/App.tsx`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/pages/admin/DashboardAppCardsPage.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增“售后工单监控”工作台卡片，权限为 `app:after_sales_work_order_monitor:access`
  - 新增账户管理表 `after_sales_work_order_monitor_accounts`，保存账户名称、账户 ID、备注
  - 新增账户管理接口和实时工单代理接口
  - 新增独立公开监控页 `/apps/after-sales-work-order-monitor/live`，不需要登录和权限
  - 实时监控按账户数量自适应分屏展示，接口每 3~7 秒随机刷新一次
  - 语音通知前新增“测试播放”按钮，播放文案为“账户名 有一条新的工单，请注意查收”
  - 检测到新工单时自动播放同样语音提醒
  - 语音播报改为队列播放，多条提醒按顺序逐条播报，不互相打断
  - 后续按要求删除“测试播放”按钮
  - 每个账户卡片右上角在“实时”前显示该列表最后一次成功拉取时间
  - 实时工单接口请求参数 `count=10`，每个账户最多展示最新 10 条，不再使用内部滚动条
  - 实时监控卡片按自身内容高度自然撑开，页面随内容自动增高
  - 工单卡片仅展示标题、类型、时间、地址
  - 新工单会从上方向下出现并高亮背景色；高亮保持到下一次新工单将其挤下去后再消失
  - 轮询无新工单时不再替换整组列表，旧数据保持不变；只有初始化或发现新工单时更新对应账户列表
  - 语音队列播放到某个账户时，该账户卡片右上角更新时间前显示小喇叭图标
  - 账户管理新增排序字段 `sort_order`，默认 `100`，账户列表按排序升序展示
  - 账户管理新增登录账号字段 `login_account`
  - 窄屏单列显示时，发现新工单的账户卡片会临时移动到第一个位置
  - 管理页默认语音通知关闭，独立实时监控页默认语音通知开启
  - test / prod 已执行 `backend/sql/add_after_sales_work_order_monitor_sort_order.sql`，两套环境均已补 `sort_order` 字段并设置默认值为 `100`
  - test / prod 已执行 `backend/sql/add_after_sales_work_order_monitor_login_account.sql`，两套环境均已补 `login_account` 字段
  - 不再按状态值硬过滤，未处理范围由接口参数 `from=Dispatch&to=Dispose` 控制
  - 页面只展示工单基本信息，不展示 `steps`
  - test / prod 两套环境已执行建表、权限、角色授权和工作台卡片初始化 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/permissions.py app/db_init.py app/models/after_sales_dept.py app/models/__init__.py app/schemas/after_sales_dept.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/j0k1l2m3n4o5_add_after_sales_work_order_monitor_accounts.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 134. 修正客户跟进记录关联客户列表匹配规则
- 文件：
  - `backend/app/services/kehu51_follow_cache.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/add_kehu51_follow_customer_list_id.sql`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `backend/migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - `customer_list_id` 关联规则改为：客户列表里同客户名只有一条时，跟进记录 `客户名称` 对应客户列表 `客户名称`；同客户名一条以上时，跟进记录 `客户名称 + 联系人` 对应客户列表 `客户名称 + 联系人`
  - 已移除跟进记录关联客户列表时对 `联系方式 / 手机号` 的匹配依赖；单客户名唯一时也不再强依赖联系人一致
  - 历史回填 SQL 仍只处理 `customer_list_id IS NULL` 的空值，不覆盖已有已关联数据
  - 客户列表辅助索引改为 `idx_customer_name_contact_name (customer_name, contact_name)`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_kehu51_follow_customer_list_id.sql`
  - test / prod 均执行成功，各执行 `25` 条 SQL
  - 按最新“客户名单条只按客户名，多条再按客户名+联系人”规则重新执行回填后：
    - test：`customer_list_id IS NULL` 从 `6815` 降到 `2900`
    - prod：`customer_list_id IS NULL` 从 `6793` 降到 `2881`
    - 剩余空值原因：客户列表无同名未删除客户、同名客户多条但联系人不匹配、少量客户名称为空
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/kehu51_follow_cache.py app/services/crawler_handlers/kehu51.py app/api/endpoints/admin/crawler_tasks.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
- 本次未新增依赖

### 135. 客户无忧源站客户ID / 联系人ID提取与精确匹配
- 文件：
  - `backend/crawler_sites/kehu51/formatter.py`
  - `backend/app/services/kehu51_follow_cache.py`
  - `backend/app/services/crawler_handlers/kehu51.py`
  - `backend/app/services/crawler_handlers/kehu51_customer_list.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/sql/add_kehu51_follow_customer_list_id.sql`
  - `backend/sql/create_kehu51_crawler_tables.sql`
  - `backend/migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 客户列表从 `GotoPage('follow',90901478,...)` 提取 `客户ID`
  - 客户跟进从 `CusDetailNew(90901478,...)` / `GotoCusCheckDetail(90901478)` 提取 `客户ID`
  - 客户跟进和客户列表从 `LinkManDetail(32327622)` 提取 `联系人ID`
  - `crawler_kehu51_follow_records`、`crawler_kehu51_customer_list` 均新增 `source_customer_id`、`source_contact_id`
  - `customer_list_id` 关联优先按 `source_customer_id`，其次按 `source_contact_id`，再回退到原名称规则
  - 修复客户跟进解析时误把 `GotoCusCheckDetail` 图标链接文本当成客户名称的问题
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_kehu51_follow_customer_list_id.sql`
  - test / prod 均执行成功，各执行 `65` 条 SQL
- 已验证：
  - 已用样例 HTML 验证：客户列表和客户跟进均能提取 `客户ID=90901478`、`联系人ID=32327622`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/kehu51_follow_cache.py app/services/crawler_handlers/kehu51.py app/services/crawler_handlers/kehu51_customer_list.py app/api/endpoints/admin/crawler_tasks.py app/api/endpoints/after_sales_dept/after_sales_dept.py crawler_sites/kehu51/formatter.py migrations/versions/n4o5p6q7r8s9_add_kehu51_follow_customer_list_id.py`
- 本次未新增依赖

### 136. 售后客户跟进统计导出表同步降频和缓存延长
- 文件：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/tasks/scheduler.py`
  - `backend/sql/create_after_sales_customer_follow_export_summary.sql`
  - `backend/sql/update_after_sales_customer_follow_export_summary_schedule.sql`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 导出预览 / 下载 Excel Redis 文件缓存 TTL 从 `4200` 秒改为 `86400` 秒（24 小时）
  - `售后客户跟进统计导出表同步` 内存调度从每小时 10 分改为每天凌晨 `02:10`
  - 初始化 SQL 中该任务 `cron_expr` 同步改为 `10 2 * * *`
  - 新增环境更新 SQL：`backend/sql/update_after_sales_customer_follow_export_summary_schedule.sql`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/update_after_sales_customer_follow_export_summary_schedule.sql`
  - test / prod 均执行成功，各执行 `1` 条 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/after_sales_dept/after_sales_dept.py app/tasks/scheduler.py`
- 本次未新增依赖

### 137. 售后工单监控新增 token 类型接口
- 文件：
  - `backend/app/models/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/sql/create_after_sales_work_order_monitor.sql`
  - `backend/sql/add_after_sales_work_order_monitor_account_type.sql`
  - `backend/migrations/versions/m2n3o4p5q6r7_add_work_order_monitor_account_type.py`
  - `frontend/src/pages/after_sales_work_order_monitor/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 售后工单监控账户新增 `account_type`，下拉显示为 `账号密码` / `token`
  - 新增 `access_token` 字段，用于保存 token 类型接口授权信息
  - `账号密码` 类型继续走原实时工单接口
  - `token` 类型走金北热线接口：`https://hotlineapi.jinbeishuiwu.cn:14443/order/related/list?pageNum=1&pageSize=10`
  - token 由账户配置保存，代码中不写死用户提供的 Bearer token
  - 金北热线返回列表从 `rows` 提取，页面映射 `orderId`、`mtceContent`、`mtceClassName`、`launchTime`、`address` 等字段
  - `token` 类型每次刷新都全量覆盖列表，只要接口返回有数据就语音提醒，不走 steps 判断
  - `token` 类型账户编辑时授权 Token 改为普通文本框并支持回显
  - `token` 类型账户 ID 可不填，后端自动生成内部 ID
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/add_after_sales_work_order_monitor_account_type.sql`
  - test / prod 均执行成功，各执行 `10` 条 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/after_sales_dept.py app/schemas/after_sales_dept.py app/api/endpoints/after_sales_dept/after_sales_dept.py migrations/versions/m2n3o4p5q6r7_add_work_order_monitor_account_type.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 138. 校验售后工单监控 token 接口返回
- 排查：
  - 使用数据库中已保存的 token 类型账户测试金北热线接口
  - test 环境存在 1 个 token 账户：`津北水务`
  - 请求接口返回：`http=200`、`code=200`、`msg=查询成功`
  - 当前返回数据：`total=0`、`rows=0`
  - prod 环境暂无 token 类型账户
- 结论：
  - 接口连通和返回结构正常
  - 当前 test 已保存 token 查询结果为空；如果预期应有数据，需要核对页面保存的 token 是否为最新可用 token、账号权限或接口筛选范围

### 139. 确认业务团队看板推送任务正式 cron 并修复展示文案
- 排查：
  - 已重新连接正式库查询 `scheduler_job_meta`
  - `after_sales_business_dashboard_push` 正式环境当前 `cron_expr = 30 8 * * 1-5`
  - 该表达式含义为周一到周五 08:30 执行
- 调整：
  - 修复计划任务列表 cron 文案解析：支持 `1-5` 这类星期范围
  - 后续 `30 8 * * 1-5` 会显示为 `每周一至五 08:30 执行`，不再误显示每天执行
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`

### 140. 新增人事考勤助手与企业微信考勤日汇总
- 文件：
  - `backend/app/core/config.py`
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/app/api/api.py`
  - `backend/app/models/wecom_attendance.py`
  - `backend/app/models/__init__.py`
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/config/env_test`
  - `backend/config/env_prod`
  - `backend/sql/create_wecom_attendance_daily_records.sql`
  - `backend/sql/create_dashboard_app_cards.sql`
  - `backend/migrations/versions/a6b7c8d9e0f1_add_wecom_attendance_daily_records.py`
  - `frontend/src/App.tsx`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/wecom_attendance/index.tsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增企业微信考勤服务，支持按日期区间拉取所有职工或指定 userid 的打卡数据，单次跨度最多 31 天
  - 新增考勤日汇总表 `wecom_attendance_daily_records`，每人每天一条
  - 汇总字段包含：部门、姓名、应出勤、实际出勤、外出、事假、病假、迟到/早退20分钟内、迟到/早退20-40分钟内、迟到/早退40-60分钟内
  - 新增接口：同步考勤、查询员工、查询日汇总
  - 新增权限 `app:wecom_attendance:access`，并初始化给 `standard_user`
  - 新增工作台卡片“人事考勤助手”，路径 `/apps/wecom-attendance`
  - 新增前端页面，支持日期区间、同步企微、部门/人员筛选、统计卡和明细表
  - `backend/config/env_test` / `backend/config/env_prod` 已配置 `WECOM_CHECKIN_AGENT_ID=3010011` 和 `WECOM_CHECKIN_CORP_SECRET`
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_wecom_attendance_daily_records.sql`
  - test / prod 均执行成功，各执行 `4` 条 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/api/api.py app/models/wecom_attendance.py app/models/__init__.py app/permissions.py app/db_init.py migrations/versions/a6b7c8d9e0f1_add_wecom_attendance_daily_records.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 141. 修复提醒请假钉钉覆盖企微别名重复统计
- 问题：
  - 同一人存在企微别名账号时，虽然钉钉数据已合并，但提醒请假仍可能把企微缺卡数据一起计入
- 调整：
  - 后端缺卡重算从 `userid + 日期` 扩展到 `部门 + 姓名归一化 + 日期`，部门不一致时支持同日期唯一姓名兜底
  - 同一人同一天存在钉钉明细时，只按钉钉明细判断旷工 / 未打卡
  - 前端人员汇总增加兜底，同一日期有钉钉记录时忽略企微行的实际出勤 / 旷工 / 未打卡
  - 缺卡详情弹窗同样按日期执行钉钉覆盖，同一天有钉钉记录时只展示钉钉行
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 142. 统一企微别名按主账号发送
- 调整：
  - 定义 `kangpeng -> tangpeng` 企微别名规则
  - 人事考勤助手人员汇总展示 userid 时会把别名归并为主 userid，例如只显示 `tangpeng`，不再显示 `kangpeng`
  - 批量提醒按页面展示 / 选中的 userid 发送，后端发送层不再二次强制改写 touser
  - 企业微信登录匹配继续复用别名配置
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom.py app/api/endpoints/auth.py app/api/endpoints/wecom_attendance.py app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 143. 正式环境新增售前招投标群发计划任务并保留售后任务
- 调整：
  - 保留原售后招投标群发任务：`okcis_daily_summary_push`，机器人 ID=2
  - 新增售前招投标群发任务：`okcis_daily_summary_presales_push`，机器人 ID=3
  - 两条任务均为周一到周五 16:10 执行：`10 16 * * mon-fri`
  - 调度函数已参数化，同一份招投标统计内容可按不同 job_id 推送到不同机器人
  - 计划任务管理接口已支持新任务展示和手动执行
  - 新增 SQL：`backend/sql/upsert_okcis_daily_summary_push_scheduler_meta.sql`
- 已执行：
  - 正式环境执行 `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --env prod --sql-file sql/upsert_okcis_daily_summary_push_scheduler_meta.sql`
  - 正式库确认两条 `scheduler_job_meta` 均存在，cron 均为 `10 16 * * mon-fri`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
- 本次未新增依赖

### 144. 排查 OKCIS 指定标题未入库原因
- 标题：
  - `轻轨张贵庄站南侧地块（住宅）二供泵房新建工程`
- 结论：
  - 正式库 `crawler_okcis_notices` 全库未查到该标题
  - 2026-07-14 今日入库 OKCIS 数据共 22 条，分布为 `70940` 6 条、`157288` 8 条、`167308` 8 条
  - 最近一轮 2026-07-14 15:40 OKCIS 任务为 partial：8 个 dzid 成功 6 个、失败 2 个
  - 失败项：`18117` 验证码解析失败“无法解析算式 5++37”；`44602` 自动刷新凭证后仍未登录
  - 目标标题包含“二供泵房”，按订阅关键词应命中 `18117`，因此直接原因是 15:40 对应订阅组未抓成功
- 备注：
  - 8:40 那轮 8 个 dzid 全成功，但当时库里也没有该标题；如果该公告在 8:40 后发布，则会受 15:40 的 `18117` 失败影响漏抓

### 145. 修复 OKCIS 验证码算式解析并增加登录刷新重试
- 调整：
  - 将单次 OKCIS 登录刷新拆为 `_refresh_credentials_from_login_curl_once`
  - `refresh_credentials_from_login_curl` 增加验证码相关异常重试，默认最多 3 次
  - 验证码算式归一支持重复中间运算符，重复运算符只计算一次
  - `5++37` 会归一后按 `5+3` 计算，避免“无法解析算式”导致 dzid 整组漏抓
- 已验证：
  - 样例：`5++37 -> 8`、`4+237 -> 6`、`+0-437 -> -4`、`5--37 -> 2`、`6**27 -> 12`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler.py app/api/endpoints/admin/crawler_tasks.py app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 146. 修复计划任务列表英文星期 cron 展示
- 问题：
  - `10 16 * * mon-fri` 已配置为周一到周五，但计划任务列表仍显示“每天 16:10 执行”
- 调整：
  - 前端 `scheduler-utils.ts` 的星期格式化支持 `mon-fri`、`mon,wed,fri` 等英文星期写法
  - 从 APScheduler `cron[...]` trigger 兜底解析时读取 `day_of_week`
  - `10 16 * * mon-fri` 和 `day_of_week='mon-fri'` 显示为“每周一至五 16:10 执行”
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 147. 修复招标信息公示订阅组筛选来源和排序
- 问题：
  - 订阅组筛选项从当前页数据生成，分页后不在当前页的订阅组不会显示
  - 正式环境新增 dzid `187653`，业务表名称为“售前营销”，用户侧以为“订阅组九”缺失
- 调整：
  - 后端 `/api/okcis/notices` 返回 `group_options`
  - 订阅组选项按 `CAST(dzid AS UNSIGNED)` 正序返回全量业务表订阅组
  - 前端订阅组筛选优先使用后端 `group_options`，无后端选项时再用当前页兜底并正序排序
- 正式库确认：
  - 当前业务表有 `18117` 组一、`44602` 组二、`70940` 组三、`157288` 组七、`167308` 组八、`187653` 售前营销
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 148. 调整人事考勤助手异常考勤口径和合并展示
- 问题：
  - 异常考勤同一天同一个人会按打卡明细拆成多行
  - 迟到 / 早退超过 60 分钟需要按上下班方向判断
  - 日期筛选不能查询当天
- 调整：
  - 上班只判断实际打卡晚于规定时间超过 60 分钟
  - 下班只判断实际打卡早于规定时间超过 60 分钟
  - 日期选择最大值限制为当前日期前一天
  - 异常考勤列表按“部门 + 姓名 + 日期 + 异常类型”合并，同人同日同类异常只显示一行
  - 合并后分钟数取最大值，详情保留相关打卡明细
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 149. 调整异常考勤分钟数展示格式
- 调整：
  - 人事考勤助手异常考勤列表“分钟数”改为中文时长格式
  - 示例：`63` 分钟显示为 `1小时3分`
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 150. 调整人事考勤助手提醒请假和异常考勤列表展示
- 调整：
  - 提醒请假、异常考勤列表隐藏企微ID列
  - 两个 tab 的搜索提示不再展示“企微ID”
  - 异常考勤默认按日期正序排序，同日按部门、姓名稳定排序
  - 地点异常模式下分钟数显示为 `-`
  - 异常考勤详情中异常打卡明细标红
  - 缺卡详情的“打卡明细”只展示缺卡类型：上班缺卡、下班缺卡；没有明细时显示“无打卡记录”
  - “企微考勤汇总”tab 仅管理员可见，非管理员访问该 tab 会自动切到“提醒请假”
  - 缺卡详情方向判断同步钉钉覆盖企微口径：同一天有钉钉明细时只用钉钉明细反推上班/下班缺卡
  - 部门筛选下拉选项改为已加载部门全集，清空筛选后不再停留在旧搜索结果部门列表
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 151. 新增企微&钉钉考勤同步计划任务
- 调整：
  - 新增固定计划任务 `wecom_dingtalk_attendance_sync`
  - 中文名：企微&钉钉考勤同步
  - 业务部门：人事行政部
  - 执行时间：每天 03:50
  - 同步口径与页面“同步企微考勤”按钮一致：
    - 同步默认考勤周期的企微考勤
    - 同步钉钉已启用人员考勤
    - 合并钉钉记录到企微日汇总
    - 重新计算旷工 / 未打卡次数
  - 计划任务列表和立即执行接口均已支持该任务
  - 新增 SQL：`backend/sql/upsert_wecom_dingtalk_attendance_scheduler_meta.sql`
  - 已执行 test / prod 计划任务元数据 upsert
  - 提醒规则文案改为“上班、下班都缺卡，如未走请假手续，记旷工 1 次”
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py app/api/endpoints/admin/scheduler.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 152. 拆分人事考勤助手独立角色并限制可见人员
- 调整：
  - 新增角色 `wecom_attendance_viewer`，名称“人事考勤助手可见人员”
  - 角色包含权限：`app:wecom_attendance:access`、`app:dingtalk_attendance:access`
  - 从 `standard_user` 移除上述两个考勤权限，避免普通用户都能看到卡片
  - 人事考勤助手卡片继续使用 `app:wecom_attendance:access`
  - 钉钉考勤卡片继续使用 `app:dingtalk_attendance:access`
  - test / prod 均已将角色绑定给：
    - `ZhouYue`：周玥VIP3
    - `wangyamei`：王亚梅
  - 新增 SQL：`backend/sql/update_wecom_attendance_permission_role.sql`
  - 同步更新初始化 SQL 和 `db_init.py`，避免后续初始化重新把权限授给普通用户
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/db_init.py`
  - test / prod 查询确认：考勤权限只绑定 `wecom_attendance_viewer`，该角色仅绑定上述两人
- 本次未新增依赖

### 153. 管理后台权限和角色页面改为列表形式
- 调整：
  - “权限定义”从卡片网格改为表格列表
  - “角色与权限”从卡片网格改为表格列表
  - 保留搜索、新建、编辑、删除、角色用户 hover 明细等操作
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 154. 权限定义和角色增加所属部门字段
- 调整：
  - `permissions`、`roles` 新增 nullable `department_id`，默认空
  - 新增索引和部门外键，关联 `departments.id`
  - 权限定义接口支持创建 / 编辑 / 清空所属部门，并返回 `department_name`
  - 角色接口支持创建 / 编辑 / 清空所属部门，并返回 `department_name`
  - 前端“权限定义”“角色与权限”列表新增所属部门列
  - 权限编辑弹窗、角色编辑抽屉新增所属部门下拉
  - 保留角色原有 `department_ids` 作为自定义数据范围部门，不改变数据范围逻辑
  - 新增 SQL：`backend/sql/add_rbac_department_fields.sql`
  - 已执行 test / prod 字段变更
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/rbac.py app/schemas/admin.py app/api/endpoints/admin/permissions.py app/api/endpoints/admin/roles.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 155. 招标信息公示新增历史数据导出
- 调整：
  - 招标信息公示页面新增“当前数据 / 历史数据导出”页签
  - 历史数据导出支持发布日期范围和订阅组多选
  - 不选择订阅组时默认按全部订阅组导出
  - 新增接口：`POST /api/okcis/notices/history-export`
  - 新增服务：`backend/app/services/okcis_history_export.py`
  - 复用 OKCIS 现有请求模板、站点凭证、订阅组缓存、详情页截止时间解析
  - 新增独立历史导出表：`crawler_okcis_notice_history_exports`
  - 新增 SQL：`backend/sql/create_okcis_notice_history_export_table.sql`
  - 已执行 test / prod 建表 SQL
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/okcis_history_export.py app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 156. 招标信息公示历史数据导出改为异步任务
- 调整：
  - 历史数据导出订阅组改为必选，不选不能提交
  - 提交历史导出后只创建任务，不刷新到当前数据页
  - 新增历史导出任务表：`crawler_okcis_notice_history_export_tasks`
  - 新增 SQL：`backend/sql/create_okcis_notice_history_export_task_table.sql`
  - 已执行 test / prod 建表 SQL
  - 新增任务列表接口：`GET /api/okcis/notices/history-export/tasks`
  - 新增任务下载接口：`GET /api/okcis/notices/history-export/tasks/{task_id}/download`
  - 前端历史导出页签新增任务列表，展示等待执行、执行中、执行完成、执行失败
  - 任务未完成前不显示下载按钮；执行完成后才可下载
  - 原“失败原因”列改为“原因”：成功显示“成功”，失败显示失败原因
  - 历史爬取不限制页数，持续翻页直到源站返回无数据
  - 历史导出字段与招标信息公示当前导出保持一致：标题、订阅组、地区、发布日期、截止时间、跟进人、链接地址、采集时间
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/okcis_history_export.py app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 157. 招标信息公示历史导出任务支持删除和空结果失败
- 调整：
  - 历史导出任务操作栏新增“删除”按钮
  - 新增接口：`DELETE /api/okcis/notices/history-export/tasks/{task_id}`
  - 删除任务时同步删除任务表记录和同 `batch_key` 的历史爬取结果数据
  - 点击“历史数据导出”页签后 URL 增加 `tab=history`，刷新页面后继续停留在历史页签；切回“当前数据”移除该参数
  - 提交任务后确认会创建任务并通过 FastAPI `BackgroundTasks` 执行真实爬取
  - 历史导出任务若最终爬取结果为 0 条，状态写为“执行失败”，原因写“未爬取到数据”，前端不显示下载按钮
  - 下载接口增加空数据兜底校验，避免空结果任务被下载
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 158. 修复 OKCIS 历史导出未登录误判为空并临时隐藏历史 tab
- 调整：
  - 排查 `2026-07-04 / 订阅条件组七` 历史导出显示“未爬取到数据”
  - 原因：源站实时请求返回 `请登录后查看 / isLogin=0`，历史导出链路未识别为未登录，导致被当作无数据处理
  - `backend/app/services/okcis_history_export.py` 增加 OKCIS 未登录检测，发现未登录后自动刷新凭证并重试一次
  - 本地验证 `dzid=157288`、`2026-07-04` 可抓到数据，包含“维克冷风机组中央空调养护维保”
  - 招标信息公示“历史数据导出”tab 临时仅管理员可见，前端通过 `app:admin_panel:access` 判断
  - 按用户要求，后端接口权限未改，仍沿用招标信息公示访问权限
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/okcis_history_export.py app/api/endpoints/okcis_notices.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 159. 修复人事考勤异常详情同时展示企微和钉钉明细
- 调整：
  - 问题：沈孟男异常考勤详情里同一天同时展示钉钉和企微打卡
  - 原因：汇总层已按“钉钉启用人员用钉钉，企微忽略”计算，但明细弹窗仍使用原始 `checkin_records` 全量展示
  - 前端新增统一有效明细函数：同一日汇总存在钉钉记录时只使用钉钉明细，否则使用企微明细
  - 异常考勤列表判断、异常考勤详情、缺卡详情、每日考勤明细统一使用该规则
  - 人员详情记录按日期做钉钉优先归并，避免同一天出现两条来源数据
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use 20 >/dev/null && npm run build`
- 本次未新增依赖

### 160. 修正招投标每日推送数量口径
- 调整：
  - 问题：推送文案“今天最新释放招投标企业：1 家”，点击详情后列表实际有多条
  - 原因：`push_okcis_daily_summary` 原来按 `buyer` 去重统计，OKCIS 记录中 `buyer` 经常为空或相同，导致数量偏小
  - 改为按当天匹配公告条数 `len(rows)` 统计，和详情链接列表数量保持一致
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
- 本次未新增依赖

### 161. 人事考勤助手新增考勤配置
- 调整：
  - 新增特殊日期配置表 `attendance_special_date_configs` 和白名单表 `attendance_whitelist_employees`
  - 新增 SQL：`backend/sql/create_attendance_special_date_configs.sql`
  - 已在 test / prod 执行建表 SQL
  - 后端新增特殊日期、批量休息日、人员白名单接口
  - `daily-records` 返回 `attendance_configs` 和 `attendance_whitelist`，并按休息日 / 白名单置正常
  - 前端“考勤配置”拆成三个子 tab：时间设置、休息日、人员白名单
  - 时间设置支持未来日期配置上班 / 下班参照时间
  - 休息日支持未来月份月历多选，批量提交休息日
  - 人员白名单支持搜索多选，白名单人员全局考勤正常
- 已执行：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python scripts/run_sql_on_envs.py --both --sql-file sql/create_attendance_special_date_configs.sql`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/models/__init__.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 162. 人事考勤助手白名单支持删除
- 调整：
  - 人事考勤助手“考勤配置 / 人员白名单”列表新增操作列
  - 支持单个人员删除，删除后按剩余人员覆盖保存白名单
  - 删除完成后刷新当前考勤列表，确保缺卡 / 异常过滤立即按最新白名单生效
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 163. 角色与权限列表排序和权限数量悬浮提示
- 调整：
  - `/admin/roles` 接口返回前按角色名升序排序，同名时按 `slug` 稳定排序
  - 角色与权限页面“权限数量”支持鼠标悬浮查看权限列表
  - 悬浮内容只显示权限名，每行一个；无权限名时兜底显示权限标识
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/admin/roles.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 164. 用户列表分配角色弹窗支持搜索
- 调整：
  - 单用户“分配角色 / 修改角色”弹窗新增角色搜索框
  - 输入时立即按角色名或角色标识过滤，不需要搜索按钮
  - 输入框右侧新增清除按钮，可一键清空搜索词
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 165. 部门架构页面改为只读
- 调整：
  - 移除组织架构树的新建、编辑、删除入口和弹窗
  - 保留部门树展开 / 收起查看能力
  - 鼠标移动到部门行和右上角“只读”标识时提示“同步OA部门架构数据”
  - 部门架构页面说明改为“同步OA部门架构数据。”
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 166. 人事考勤助手新增考勤组管理
- 调整：
  - 新增“考勤配置 / 考勤组管理”tab
  - 新增表 `attendance_groups`，字段包含考勤组名称、部门多选、排序、备注
  - 新增 SQL：`backend/sql/create_attendance_groups.sql`
  - 已在 test / prod 执行 SQL
  - `attendance_special_date_configs` 增加 `attendance_group_id`、`attendance_group_key`
  - 休息日配置支持绑定考勤组；不选为默认考勤组
  - 默认考勤组优先级最低，员工属于新建考勤组时只使用该组休息日；该组未选择休息日即没有休息日，不再套默认考勤组
  - 考勤组按所选部门自动包含子部门员工，后端返回展开后的部门名称用于匹配
  - 删除考勤组前必须先清空部门，组内还有部门时不允许删除
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/models/__init__.py app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 167. 人事考勤助手考勤汇总新增花名册导出
- 调整：
  - 后端新增 `GET /api/wecom-attendance/roster/export`
  - 复用上传花名册填充逻辑，使用系统花名册模板导出已填充考勤数据的 Excel
  - “考勤汇总”tab 右上角新增“导出考勤汇总”
  - “导出考勤汇总 / 下载模板 / 上传花名册 / 同步企微考勤”仅在“考勤汇总”页面显示
  - 钉钉人员管理页保留“同步钉钉人员”按钮
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 167. 人事考勤助手入职当天缺卡旷工规则补齐
- 调整：
  - 入职生命周期规则补齐：入职当天及入职前，旷工 / 未打卡不计；离职后仍不计
  - 已重新执行 `backend/sql/create_attendance_groups.sql` 到 test / prod，确认考勤组表和休息日作用域字段均可重复执行
  - 已重算 test / prod 2026-07-01 至 2026-07-31 缺卡/旷工数据
  - test 更新 4107 条，当前旷工合计 268，未打卡合计 314
  - prod 更新 3964 条，当前旷工合计 189，未打卡合计 292
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 168. 人事考勤法定节假日与周六工作日规则
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `backend/sql/upsert_attendance_2026_national_holidays.sql`
  - `data/attendance_compare_20260401_20260426_key_fields.xlsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 调整：
  - 新增 2026 年国家法定节假日默认休息日 SQL，并执行到 test / prod
  - 缺卡 / 旷工重算读取考勤特殊日期与考勤组配置
  - 考勤汇总、每日列表、花名册导出统一休息日规则
  - 独立日期配置优先级最高：配置为休息日则休息；配置了上下班时间则使用独立时间
  - 周六配置为非休息日但未填上下班时间时，默认使用 `09:00-15:00`
  - 周六 / 周日未配置为非休息日时默认休息
- 数据处理：
  - 已重算 test / prod 2026-04-01 至 2026-04-26 缺卡 / 旷工
  - 正式库刘凯 2026-04-06 已变为旷工 0、未打卡 0、迟到早退 0
  - 重新生成 `data/attendance_compare_20260401_20260426_key_fields.xlsx`
  - 最新对比：源表异常 25 人，18 人存在差异，39 条字段差异；旷工差异降为 1 人、系统多 5 天
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py`
- 本次未新增依赖

### 169. 人事考勤苗会亮年假抵扣旷工
- 文件：
  - `backend/app/services/wecom_attendance.py`
  - `backend/app/api/endpoints/wecom_attendance.py`
  - `data/attendance_compare_20260401_20260426_key_fields.xlsx`
  - `CURRENT_CONTEXT.md`
  - `PROJECT_HISTORY.md`
- 排查：
  - 正式库苗会亮 2026 年 4 月年假共 6 天：4/1、4/2、4/3、4/8、4/9、4/10
  - 其中 4/1、4/2、4/3、4/9、4/10 同时存在上下班未打卡，旧扣减逻辑没有把年假纳入旷工抵扣，所以误记为旷工
- 调整：
  - 服务层 `_absence_deduction_days` 加入 `annual_leave`
  - 接口层同名扣减函数同步加入 `annual_leave`
  - 页面展示、导出、重算落库保持同一口径
- 数据处理：
  - 已重算正式库 2026-04-01 至 2026-04-26
  - 苗会亮年假保留 6 天，旷工已全部清零；4/7、4/13 仍为未打卡各 1 次
  - 重新生成 `data/attendance_compare_20260401_20260426_key_fields.xlsx`
  - 最新对比：源表异常 25 人，18 人存在差异，38 条字段差异；旷工差异已消除
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 170. 人事考勤汇总表头单位换行
- 调整：
  - 考勤汇总表头单位独立换行展示，例如 `应有年假` / `（天）`
  - 迟到/早退三档表头拆成多行，缩减列宽
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 171. 人事考勤汇总改为最新版花名册口径
- 调整：
  - 考勤汇总去掉 `应有年假`
  - 上传花名册时记录上传时间，并保存 `部门 + 姓名 + 应出勤`
  - 列表和导出都以本月最新花名册人员为基础，花名册提供应出勤，系统填充其他考勤统计
  - 考勤汇总标题本月有花名册时显示 `（花名册：YYYY年M月D日）`
  - 没有本月花名册时不显示上传日期，应出勤展示为 `-`
  - `0 / 0.0` 统一展示为 `-`
  - 表格整体撑满容器
  - 考勤汇总表增加模拟浮动表头，滚出顶部后显示固定表头
  - 操作列“详情”去掉图标
- 数据库：
  - 新增表 `wecom_attendance_roster_uploads`
  - 新增表 `wecom_attendance_roster_expected_days`
  - 新增 SQL：`backend/sql/create_wecom_attendance_roster_uploads.sql`
  - 新增迁移：`backend/migrations/versions/g7h8i9j0k1l2_create_wecom_attendance_roster_uploads.py`
  - 已执行到 test / prod 两套环境，并确认两张表都存在
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/wecom_attendance.py app/models/wecom_attendance.py app/models/__init__.py migrations/versions/g7h8i9j0k1l2_create_wecom_attendance_roster_uploads.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 172. 人事考勤汇总请假事由详情修复
- 调整：
  - 修复考勤汇总点击请假类汇总项后，详情弹窗 `内容` 显示为 `-` 的问题
  - 弹窗打开时，OA 请假明细改为按“当前人员姓名 + 当前人员部门”单独查询
  - 不再沿用页面外层提交过的筛选关键词，避免误过滤掉当前人员请假明细
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 173. OA考勤修正与请假明细本地镜像化
- 调整：
  - 新增本地表 `wecom_attendance_oa_approved_records`，保存 OA 已通过的请假、补卡、考勤修正明细
  - `同步企微考勤` 接口执行时，同步当前日期范围内的 OA 已通过明细到本地
  - 每日 `03:50` 的企微&钉钉考勤同步任务，已增加 OA 已通过明细本地同步
  - `GET /api/wecom-attendance/oa-approved-records` 改为查询本地镜像表，不再每次直连 OA 数据库
  - 新增迁移：`backend/migrations/versions/i7j8k9l0m1n2_create_wecom_attendance_oa_approved_records.py`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/models/wecom_attendance.py app/models/__init__.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py app/tasks/scheduler.py migrations/env.py migrations/versions/i7j8k9l0m1n2_create_wecom_attendance_oa_approved_records.py`
- 本次未新增依赖

### 174. 未打卡改为按缺卡次数统计
- 调整：
  - 考勤汇总 `未打卡` 由原来的按天/单次口径，改为按缺卡次数统计
  - 同一天上班缺卡记 `1` 次，下班缺卡记 `1` 次；若上下班都缺卡，则 `未打卡 = 2`
  - 同时保留 `旷工 = 1` 的规则
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py`
- 本次未新增依赖

### 175. 未打卡前端汇总与详情口径统一
- 调整：
  - 前端汇总页未打卡统计，改为按 `上班缺卡/下班缺卡` 标签数量汇总
  - 详情弹窗 `时长` 同步改为按缺卡标签数量展示
  - 修复 `2026-07-20` 王帅详情显示 `上班缺卡、下班缺卡` 但汇总仍显示 `1次` 的问题
- 已验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 176. OKCIS 售前营销权限不足日志补地址
- 排查：
  - 正式环境 `okcis_notice_manual` 最近重爬 `run_id=873`，`dzid=187653`，执行成功
  - 用户关注公告 `uniseq=20260721155347222238` 在第 3 页第 31 条命中详情权限不足
  - 对应地址：`https://www.okcis.cn/20260721-n2-20260721155347222238-b10cfc8e8905baf4f618f88a48ee1e4b.html`
- 调整：
  - `backend/app/services/crawler_handlers/okcis.py` 权限不足跳过日志补充 `url=...`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/crawler_handlers/okcis.py`
- 本次未新增依赖

### 177. 钉钉调休余额修正方法
- 调整：
  - 新增 `DingtalkAttendanceService.correct_compensatory_leave_balance`
  - 参数为钉钉用户ID和目标调休余额天数，余额按 1 位小数处理
  - 先查询当前调休已用天数，再计算应写入的总额度，避免已用额度被覆盖
  - 新增脚本 `backend/scripts/adjust_dingtalk_compensatory_balance.py`，默认仅预览，加 `--apply` 后实际修正
  - 新增配置 `DINGTALK_ATTENDANCE_QUOTA_OPERATOR_USERID=026960310339-366502766`、`DINGTALK_ATTENDANCE_QUOTA_OPERATOR_NAME=王亚梅`
- 验证：
  - 王亚梅完整钉钉 userId 为 `026960310339-366502766`
  - 已用王帅钉钉用户ID `2331516814941018` 执行预览，当前余额 `15.0` 天，生成参数正常
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/dingtalk_attendance.py scripts/adjust_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 178. 钉钉新增调休测试假期规则
- 调整：
  - 新增 `DingtalkAttendanceService.create_leave_rule_like`
  - 已按现有 `调休` 规则新增 `调休(新)`，leave_code=`d270ec7a-ec0d-4e47-8de6-ec680916e067`
  - 钉钉限制企业只能存在一个 `lieu_leave`，因此 `调休(新)` 创建为 `general_leave`
  - 保留 8 小时/天、最小请假单位 1 小时、半天展示等配置
- 验证：
  - `调休(新)` 已能在假期规则列表中查询到，来源为 `external`
  - 仅用康鹏 `032607495340527` 测试写入约 1 小时额度
  - 不带额度字段时返回 `按天计算的额度不能为空`
  - 带 `quota_num_per_day=13` 后返回 `非调休假类型年度不能为空`
  - 尝试 `year=2026`、`quota_year=2026`、`quota_num_per_year=13` 后仍未写入成功，查询余额列表仍为空
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/dingtalk_attendance.py scripts/adjust_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 179. 钉钉研发中心调休规则改为手动发放
- 调整：
  - 将 `调休(新)` 改名为 `调休（研发中心）`
  - 规则编码：`d270ec7a-ec0d-4e47-8de6-ec680916e067`
  - 余额发放方式改为手动：`when_can_leave=manual`
  - 适用范围改为研发中心：`visibility_rules=[{"type":"dept","visible":["1001032160"]}]`
  - 修正余额方法改为先初始化 0，再执行余额更新
  - 初始化/更新请求补充 `quota_cycle`，调休额度按 `quota_num_per_day` 传递
- 验证：
  - 康鹏 `032607495340527` 先初始化 0，再测试发放约 1 小时成功
  - 查询到 `quota_num_per_day=13`、`quota_cycle=2026`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/dingtalk_attendance.py scripts/adjust_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 180. 钉钉研发中心调休规则不限额
- 调整：
  - `设置员工假期余额` 改为不限额，对应 `freedom_leave=true`
  - 因旧规则无法直接从限额改为不限额，已删除旧规则 `d270ec7a-ec0d-4e47-8de6-ec680916e067`
  - 新建并保留正式规则 `调休（研发中心）`，leave_code=`1725c796-5f36-4e2f-9cd6-ce5e78f98d98`
  - 规则继续限制研发中心：`visibility_rules=[{"type":"dept","visible":["1001032160"]}]`
  - 新员工请假保持入职后就可以：`when_can_leave=entry`
  - 余额修正方法优先命中 `调休（研发中心）`
- 验证：
  - 回查钉钉假期规则：`freedom_leave=true`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/core/config.py app/services/dingtalk_attendance.py scripts/adjust_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 181. 钉钉研发中心调休规则改回限额手工发放
- 调整：
  - 按要求将 `调休（研发中心）` 重新设置为限制额度 + 手工发放
  - 旧不限额规则 `1725c796-5f36-4e2f-9cd6-ce5e78f98d98` 无法原地改回限额，已删除后重建
  - 当前有效规则 leave_code=`91a66dd8-5dbd-42c6-a927-60bb5e9009bc`
  - 当前规则：`freedom_leave=false`、`paid_leave=true`、`when_can_leave=entry`
  - 适用范围继续限制研发中心：`visibility_rules=[{"type":"dept","visible":["1001032160"]}]`
- 验证：
  - 已回查钉钉假期规则列表确认新规则生效
- 本次未新增依赖

### 180. 招投标历史数据导出 tab 权限开放
- 调整：
  - `frontend/src/pages/OkcisNoticesPage.tsx` 的历史数据导出 tab 可见权限从 `app:admin_panel:access` 改为 `APP_OKCIS_NOTICES_ACCESS`
  - 已确认后端历史导出相关接口本来就是 `APP_OKCIS_NOTICES_ACCESS`
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 181. 钉钉调休规则手动发放 extras 固化
- 调整：
  - 当前 `调休（研发中心）` leave_code=`91a66dd8-5dbd-42c6-a927-60bb5e9009bc`
  - 已调用 `/topapi/attendance/vacation/type/update` 补充：
    - `quota_grant_mode=manual`
    - `init_quota_type=manual`
  - 请求 `extras={"validity_type":"absolute_time","validity_value":"12-31","quota_grant_mode":"manual","init_quota_type":"manual"}`
  - 钉钉开放接口返回 `errcode=0`
  - `backend/app/services/dingtalk_attendance.py` 后续创建规则默认写入上述 `extras`
  - 钉钉规则查询接口不回显 `extras`，最终展示需以钉钉后台页面为准
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 182. 钉钉调休规则重新建立验证手动发放
- 调整：
  - 因原规则更新 `extras` 后后台仍未显示手动发放，已删除旧规则 `91a66dd8-5dbd-42c6-a927-60bb5e9009bc`
  - 已重新创建 `调休（研发中心）`
  - 当前新 leave_code=`15546c81-796a-4809-b646-f8f55c128300`
  - 新建请求已带 `extras={"validity_type":"absolute_time","validity_value":"12-31","quota_grant_mode":"manual","init_quota_type":"manual"}`
  - 当前规则仍为限额：`freedom_leave=false`
  - 新员工请假：`when_can_leave=entry`
  - 适用范围：研发中心 `visibility_rules=[{"type":"dept","visible":["1001032160"]}]`
  - 钉钉开放接口删除、新建均返回 `errcode=0`
  - 钉钉规则查询接口不回显 `extras`，后台页面显示需人工刷新确认
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 183. 钉钉调休规则按用户 curl 参数重建
- 调整：
  - 直接创建因重名返回 `errcode=880002 已存在相同假期名称`
  - 已删除上一版规则 `15546c81-796a-4809-b646-f8f55c128300`
  - 已按用户提供 curl 参数重新创建 `调休（研发中心）`
  - 当前新 leave_code=`a77c152b-a841-4d50-bb7e-3cb96fe801d1`
  - 新建请求 `extras={"validity_type":"absolute_time","validity_value":"12-31"}`
  - 当前规则仍为限额：`freedom_leave=false`
  - 新员工请假：`when_can_leave=entry`
  - 适用范围：研发中心 `visibility_rules=[{"type":"dept","visible":["1001032160"]}]`
  - `backend/app/services/dingtalk_attendance.py` 后续创建规则默认 extras 已同步为该版本
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 184. 删除新建的钉钉研发中心调休规则
- 调整：
  - 按要求删除刚创建的 `调休（研发中心）`
  - 已删除 leave_code=`a77c152b-a841-4d50-bb7e-3cb96fe801d1`
  - 删除接口返回 `errcode=0`
  - 回查钉钉假期规则列表，当前只剩系统内置 `调休`
  - 内置 `调休` leave_code=`031fc328-ea2c-47e0-87f3-7c95a590d0c5`
- 本次未新增依赖

### 185. 排查钉钉后台新建调休（新）余额更新
- 排查：
  - 钉钉后台新建 `调休（新）` 可通过假期规则列表查到
  - leave_code=`6f0d4abe-73d1-425e-8bd1-58ac31018f13`
  - 规则来源 `source=inner`，`biz_type=general_leave`，`leave_view_unit=day`，`paid_leave=false`
  - 康鹏钉钉 user_id=`032607495340527`，企微 user_id=`kangpeng`
  - 本系统今年累计统计康鹏：加班 `1.04` 天、调休 `0.00` 天、剩余 `1.04` 天
  - 使用 `quota/list` 查询康鹏该规则余额无报错，但无 `leave_quotas`
  - 使用 `quota/init`、`quota/update` 写入康鹏 `1.0` 天 / `8.3` 小时均失败
  - 钉钉返回 `880015：批量leaveCode或userId都不存在`
  - 初步判断：后台创建的 `source=inner` 规则不能通过当前开放额度接口给康鹏写余额，或该规则未对康鹏生效
- 调整：
  - `backend/app/services/dingtalk_attendance.py` 补充无历史 quota 时默认生成当年 `start_time/end_time`
- 已验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 186. 开放接口创建调休（新）并验证余额写入成功
- 调整：
  - 删除后台创建的 `调休（新）`，旧 leaveCode=`6f0d4abe-73d1-425e-8bd1-58ac31018f13`
  - 使用 `/topapi/attendance/vacation/type/create` 重新创建 `调休（新）`
  - 新 leaveCode=`bc88d408-8cf3-4fda-94fc-266f25cf7cc5`
  - 类型：`biz_type=general_leave`
  - 来源：`source=external`
  - 带薪：`paid_leave=true`
  - 适用部门：研发中心 `1001032160`
- 验证：
  - 使用康鹏钉钉 user_id=`032607495340527` 写入余额
  - 目标余额：`1.0` 天 / `8.3` 小时
  - `quota/init` 返回 `errcode=0`
  - `quota/update` 返回 `errcode=0`
  - 回查余额：`quota_id=66bfa057-f24b-4f75-ad6d-507f87088243`，`balance_days=1.0`
  - 钉钉回查只返回 day quota，小时按 `8.0` 小时显示
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py`
- 本次未新增依赖

### 187. 人事考勤助手筛选行与康鹏调休余额口径修复
- 调整：
  - `frontend/src/pages/wecom_attendance/index.tsx` 筛选区改为 flex 一行布局
  - 覆盖 tab：
    - 考勤汇总
    - 缺卡提醒
    - 异常考勤
    - 考勤修正
    - 加班统计
    - 调休统计
    - 钉钉考勤人员管理
  - 宽屏筛选、搜索、重置保持同一行；窄屏自动换行
- 排查：
  - 页面调休统计显示康鹏余额 `0.10` 天
  - 脚本 `backend/scripts/debug_dingtalk_compensatory_balance.py` 原来把钉钉加班 `duration` 小时误当成天
  - 已改为加班小时 `/8` 折算为天
  - 正式环境康鹏钉钉 `调休（新）` 余额已从 `1.0` 天修正为 `0.1` 天
  - 业务口径已更正为调休余额换算按 `1 天 = 8 小时`
  - `backend/app/services/dingtalk_attendance.py` 已将调休余额换算改为 `COMPENSATORY_BALANCE_HOURS_PER_DAY=8`
  - 回查：`balance_days=0.1`、`balance_hours=1.0`
- 已验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py scripts/debug_dingtalk_compensatory_balance.py`
- 本次未新增依赖

### 188. 正式环境批量同步调休统计余额到钉钉
- 执行：
  - 数据来源：人事考勤助手调休统计接口 `/compensatory-leave-stats`
  - 时间范围：`2026-01-01` 至 `2026-07-22`
  - 总人数：`55`
  - 同步口径：调休统计余额天数一致，`0.1 天 = 1 小时`
- 结果：
  - 成功同步 `18` 人
  - 失败 `0` 人
  - 成功人员：岳哿丞、徐皓、朱文波、沈孟男、孙健、张杰、张瑞新、李海涛、田旭、诸金龙、于涛、史延明、娄新颜、康鹏、王业龙、王帅、薛张波、辛军
  - 未同步 `37` 人：
    - 多数为未匹配钉钉用户
    - `王奇伟` 无企微ID
    - 余额为负数人员跳过：刘雁鹏、贠丹丹、赵本德、郭丽霞、孔德明、张永勋、程金雨、高振兴
- 本次未新增依赖

### 189. 正式环境手动执行企微&钉钉考勤同步任务
- 执行：
  - 任务：`wecom_dingtalk_attendance_sync`
  - 环境：正式环境配置 `backend/config/env_prod`
  - 触发方式：手动执行 `sync_wecom_and_dingtalk_attendance_job(run_trigger='manual')`
  - 执行时间：`2026-07-23 08:29:35` 至 `08:31:18`
- 结果：
  - `scheduler_job_runs` 写入成功记录
  - 状态：`success`
  - 摘要：`企微&钉钉考勤同步完成`
  - 本次默认同步考勤范围：`2026-06-25` 至 `2026-07-22`
- 备注：
  - 任务结束后 Python 进程打印了 aiomysql 连接析构时的 `Event loop is closed` 忽略级提示；业务任务已成功提交并记录成功
- 本次未新增依赖

### 190. 新增人力资源助手业务流程图文档
- 新增：
  - `doc/hr-attendance-assistant-business-flow.md`
- 内容：
  - 总体业务流程
  - 企微、钉钉、OA、花名册数据同步链路
  - 人员范围与钉钉覆盖企微规则
  - 考勤汇总计算流程
  - 请假、考勤修正、加班、调休关系
  - 花名册上传和导出流程
  - 页面模块与后端数据来源关系
  - 核心表 ER 关系图
- 说明：
  - 流程图使用 Mermaid，Markdown 预览可直接查看
- 本次未新增依赖

### 191. 系统管理头部统一与软件部工作记录定时默认值调整
- 调整：
  - `frontend/src/pages/admin/AdminDeckPage.tsx`
    - 系统管理首页删除单独头部，改为复用公共 `NavBar`
  - `frontend/src/pages/admin/components/AdminNavBar.tsx`
    - 系统管理子页面统一保留公共 `NavBar`
    - 公共头部下方增加二级返回栏，包含返回系统管理箭头和当前模块名称/切换
  - `frontend/src/pages/software_task/index.tsx`
    - 康鹏新建工作记录时，“定时自动提交”默认勾选
    - 默认提交时间按记录日期生成 `17:30-17:50` 之间随机时间
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 192. 修复正式环境软件部日报提醒任务 text 变量冲突
- 问题：
  - 正式环境 `software_work_record_reminder` 报错：`cannot access local variable 'text' where it is not associated with a value`
- 原因：
  - `backend/app/tasks/scheduler.py` 已从 SQLAlchemy 导入 `text`
  - 日报提醒函数内后续又定义局部变量 `text` 作为 markdown 内容
  - Python 将 `text` 判定为函数局部变量，导致前面的 `text(...)` SQL 构造访问未赋值局部变量
- 修复：
  - 将 markdown 内容变量改为 `reminder_text`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/tasks/scheduler.py`
  - 使用正式环境配置 `backend/config/env_prod` 手动执行 `software_work_record_reminder`
  - 任务写入 `scheduler_job_runs` 成功，状态 `success`
  - 企微机器人返回 `errcode=0`
  - 本次正式执行实际提醒人员：薛张波
- 本次未新增依赖

### 193. 修复软件部工作记录日期切换清空内容并汉化日历
- 调整：
  - `frontend/src/pages/software_task/index.tsx`
    - 新建工作记录弹窗只在打开时按初始日期加载一次草稿
    - 切换日期后不再重新拉取草稿并重置项目、工时、工作内容、定时提交等内容
  - `frontend/src/components/ui/calendar.tsx`
    - 公共日历组件默认使用 `date-fns` 的 `zhCN` locale
    - 星期显示从英文缩写改为中文
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 194. 调整线上办公助手 OnlyOffice 编辑页头部
- 调整：
  - `backend/app/api/endpoints/onlyoffice.py`
  - 编辑页头部从输入框 + 保存按钮改为更简洁的标题展示
  - 点击标题进入编辑状态
  - 点击其他位置失焦后自动保存文件名
  - 回车保存，Esc 取消
  - 编辑状态下每 1 分钟自动保存一次文件名
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py`
- 本次未新增依赖

### 195. 继续优化线上办公助手表格页签与标题栏
- 调整：
  - `backend/app/api/endpoints/onlyoffice.py`
    - 标题前缀改为 `文件名：`
    - 编辑输入框宽度按文件名长度自适应
    - 头部改为 flex 布局，编辑区占满剩余高度
  - `backend/app/services/onlyoffice.py`
    - xlsx 模板补充第二个 sheet
    - workbook 明确配置 `activeTab`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
- 本次未新增依赖

### 196. 线上办公助手新建 Word 文档改为空白
- 调整：
  - `backend/app/services/onlyoffice.py`
  - 新建 `docx` 模板去掉默认标题和说明文字
  - 文档正文只保留空段落，打开后为空白文档
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/onlyoffice.py`
- 本次未新增依赖

### 197. 修复线上办公助手新建 PPT 和文件名自动保存
- 调整：
  - `backend/app/services/onlyoffice.py`
    - PPTX 空白模板补齐 `presentation`、`slide master`、`slide layout`、`theme`、`presProps`、`viewProps`、`tableStyles` 等结构
    - 新建 PPT 改为空白单页，避免 OnlyOffice 打开时报模板结构错误
  - `backend/app/api/endpoints/onlyoffice.py`
    - 文件名自动保存改为 60 秒倒计时
    - 进入页面后不再立即触发自动保存
    - 页面显示自动保存倒计时
    - 文件名保存成功后后端返回最新编辑器配置，前端调用 OnlyOffice `refreshFile` 同步编辑器内标题
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - 本地生成临时 PPTX，`zipfile.testzip()` 通过，必需文件无缺失
- 本次未新增依赖

### 206. 税务助手页头只保留刷新和同步 OA
- 调整：
  - `frontend/src/pages/tax_assistant/index.tsx`
  - 去掉页头的当前节点勾选和搜索框
  - 只保留 `刷新`、`同步 OA`
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 205. 更新人事考勤助手业务流程图
- 调整：
  - `doc/hr-attendance-assistant-business-flow.md`
  - 全量重写 Mermaid 流程图，改成兼容 `mermaid 9.1.3` 的写法
  - 流程同步到最新代码链路：打卡计划任务同步、钉钉审批 Stream 监听、加班自动更新调休余额、审批事件日志入库
  - 补充 `dingtalk_approval_event_logs` 到核心表关系
- 验证：
  - `git diff --check -- doc/hr-attendance-assistant-business-flow.md`
- 本次未新增依赖

### 198. 修复线上办公助手新建未操作也进入历史记录
- 问题：
  - 点击新建文件后进入编辑器，未做任何操作直接退出，历史文件里已经出现一份记录
- 调整：
  - `backend/app/services/onlyoffice.py`
  - 新建文件 metadata 默认写入 `is_saved=False`
  - 历史列表过滤未保存文件，只展示 `is_saved=True` 或老数据缺省文件
  - OnlyOffice 回调真正保存后，`_touch_file` 标记 `is_saved=True`
  - 文件名保存按钮和倒计时逻辑不改动
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - 本地模拟新建后列表不显示，触发保存标记后列表显示
- 本次未新增依赖

### 199. 线上办公助手增加分享协同编辑和文件导入
- 后端：
  - `backend/app/services/onlyoffice.py`
    - metadata 增加 `shared_with` 分享人员列表
    - metadata 增加 `is_saved`，新建未保存文件不进入历史列表
    - 历史列表按所有者/被分享人过滤
    - 所有者可重命名、删除、分享
    - 被分享人可打开同一文件协同编辑
    - 增加导入 `docx/xlsx/pptx` 文件能力
  - `backend/app/api/endpoints/onlyoffice.py`
    - 增加 `POST /api/onlyoffice/files/import`
    - 增加分享保存接口
    - 编辑器页面增加分享弹层
    - 分享弹层支持搜索人员、部门筛选，并自动选择部门及子部门人员
- 前端：
  - `frontend/src/pages/online_office/index.tsx`
    - 文件列表页增加“导入文件”按钮
    - 分享给我的文件可打开，改名/删除按钮禁用
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 200. 线上办公助手分享增加地址和只读/编辑权限
- 调整：
  - 编辑页分享保存后展示可转发地址：`/apps/online-office?open_file_id=...`
  - 文件列表页支持识别 `open_file_id` 参数并自动打开文件
  - 分享支持权限：`可编辑` / `只读`
  - 只读分享以 OnlyOffice `view` 模式打开
  - 被分享用户不能改名、删除、分享文件
  - 保存分享时仍支持搜索、部门筛选、自动选择部门及子部门人员
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 201. 线上办公助手分享地址改为短链
- 调整：
  - `backend/app/services/onlyoffice.py`
    - 分享保存时为文件生成 `share_code`
  - `backend/app/api/endpoints/onlyoffice.py`
    - 分享弹层保存后优先展示短地址 `/s/{share_code}`
  - `backend/app/main.py`
    - 新增 `/s/{code}` 跳转，跳到 `/apps/online-office?open_file_id=...`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/main.py app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
- 本次未新增依赖

### 202. 修复线上办公助手文件名保存交互
- 问题：
  - 编辑文件名后点击其他区域没有自动保存
  - 点击标题栏保存按钮反馈不明确
- 调整：
  - `backend/app/api/endpoints/onlyoffice.py`
  - 文件名输入框恢复 `blur` 自动保存
  - 保存按钮 `mousedown` 阻止输入框先失焦，避免点击保存时事件顺序异常
  - 非编辑状态点击保存提示先点击文件名编辑
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py`
- 本次未新增依赖

### 203. 修复线上办公助手短链前端域名无法跳转
- 问题：
  - 分享短链为前端域名 `/s/{code}`，例如 `http://172.18.6.140:5173/s/f1eba112`
  - 原实现只在后端根路由 `/s/{code}` 处理，前端开发服务无法命中后端路由
- 调整：
  - `backend/app/api/endpoints/onlyoffice.py`
    - 新增 `GET /api/onlyoffice/short-links/{code}`
    - 按当前登录用户校验 OnlyOffice 文件访问权限
    - 返回对应 OnlyOffice 编辑入口
  - `frontend/src/App.tsx`
    - 新增 `/s/:code` 前端路由
    - 登录后自动解析短码并进入对应文档
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/main.py app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 204. 修复线上办公助手保存文件名后不进入历史
- 问题：
  - 历史列表只显示 `is_saved=True`
  - 文件名保存只更新标题，没有将新建文件标记为已保存
  - 导致保存了文件名后历史列表仍看不到该文档
- 调整：
  - `backend/app/services/onlyoffice.py`
  - `rename_file` 成功保存文件名时同步设置 `is_saved=True`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/onlyoffice.py`
- 本次未新增依赖

### 198. 新增税务助手发票管理
- 调整：
  - 新增工作台卡片 `税务助手`
  - 新增独立权限 `app:tax_assistant:access` 和角色 `tax_assistant_member`
  - 新增后端接口：发票列表、发票详情、手动同步
  - 新增本地镜像表：`tax_invoice_basic_info`、`tax_invoice_detail_info`
  - 新增计划任务 `tax_invoice_oa_sync`，每 3 分钟同步一次 OA 开票申请
  - 前端新增页面 `/apps/tax-assistant`
- OA 同步口径：
  - 流程：`开票申请`，`workflowid=390`
  - 来源表：`formtable_main_33`、`formtable_main_33_dt1`
  - 仅同步未完结，且 `workflow_requestbase.currentnodeid = workflow_currentoperator.nodeid`、当前处理人为 `刘羽丰` 的单据
  - 当前节点样例：`2518 / 6.财务会计`
- 执行结果：
  - test/prod 均已执行建表、权限、卡片、计划任务初始化 SQL
  - test/prod 均已手动同步初始数据：基本信息 `24` 条、明细 `25` 条、当前目标 `24` 条
  - `doc/database_dictionary.md` 已补充新表结构
- 文件：
  - `backend/app/permissions.py`
  - `backend/app/db_init.py`
  - `backend/app/models/tax_assistant.py`
  - `backend/app/api/endpoints/tax_assistant.py`
  - `backend/app/services/tax_assistant.py`
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/api.py`
  - `backend/migrations/versions/44c00cd6e9b5_add_tax_assistant_invoice_tables.py`
  - `backend/sql/create_tax_assistant_invoice_tables.sql`
  - `frontend/src/App.tsx`
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/tax_assistant/index.tsx`
  - `doc/database_dictionary.md`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/tax_assistant.py app/api/endpoints/tax_assistant.py app/models/tax_assistant.py app/tasks/scheduler.py app/api/api.py app/db_init.py migrations/versions/44c00cd6e9b5_add_tax_assistant_invoice_tables.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 199. 线上办公助手新建文件保存状态优化
- 调整：
  - `backend/app/services/onlyoffice.py`
  - 新建文件 metadata 标记 `is_saved=False`
  - 历史列表只展示 `is_saved=True` 或老数据缺省的文件
  - OnlyOffice 回调真正保存文件后，`_touch_file` 标记为 `is_saved=True`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/onlyoffice.py app/services/onlyoffice.py`
  - 本地模拟新建后列表不显示，触发保存标记后列表显示
- 本次未新增依赖

### 207. 税务助手补发票号回填与审核状态规则
- 调整：
  - `tax_invoice_basic_info` 新增 `audit_status`、`audit_status_changed_at` 字段和索引，test/prod 已直接执行
  - OA 发票同步继续跟踪已同步过的历史单据，回填当前节点、流程状态、审批日志、主表 `fphxx`、明细 `fph` 和审核状态
  - 审核通过规则：存在 `6.财务会计 / 刘羽丰 / log_type=0` 审批日志，且主表或明细有发票号，才为 `已审核`
  - 发票管理列表、详情、明细表增加发票号展示
  - 修复同步时 `0` 被当空值导致普通发票再次写回 `0` 的问题
- 执行结果：
  - test 同步后：已审核 32 条，未审核 1 条
  - prod 同步后：已审核 33 条，未审核 1 条
  - 样例 `1174205` 明细发票号 `26122000000985924261`，状态 `已审核`，当前节点 `8.归档`
  - 样例 `1171829` 状态 `未审核`，当前节点 `6.财务会计 / 刘羽丰`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py app/services/tax_assistant.py app/models/tax_assistant.py migrations/versions/44c00cd6e9b5_add_tax_assistant_invoice_tables.py migrations/versions/af46c1d2e3f4_add_tax_invoice_full_oa_fields.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 208. 修复税务助手列表发票号为空
- 问题：OA 主表 `fphxx` 为空，但明细表 `fph` 已有发票号，发票管理列表只展示主表字段导致显示 `-`
- 调整：
  - 同步时主表发票号为空则用明细 `invoice_no/fph` 汇总回填 `invoice_no_info`
  - 发票列表和详情接口增加明细发票号兜底
  - test/prod 已直接回填历史数据：test 32 条，prod 33 条
- 样例：
  - `1174267` 发票号 `26122000000990951676`
  - `1174248` 发票号 `26122000000987900601`
  - `1174234` 发票号 `26122000000992741416`
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py app/services/tax_assistant.py`
  - `git diff --check -- backend/app/api/endpoints/tax_assistant.py backend/app/services/tax_assistant.py`
- 本次未新增依赖

### 209. 税务助手发票管理去掉关键字筛选
- 调整：
  - `frontend/src/pages/tax_assistant/index.tsx`
  - 发票管理筛选区移除关键字输入框
  - 请求参数不再传 `keyword`
  - 筛选区保留发票类型、审核状态、合同编号、搜索、重置
- 验证：
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 210. 税务助手新增税务局批量开票模板导出
- 调整：
  - 新增模板文件 `backend/static/tax-invoice-import-template.xlsx`
  - 新增接口 `GET /api/tax-assistant/invoices/export-template`
  - 发票管理筛选区新增 `导出` 按钮，按当前筛选条件导出模板 Excel
  - 列表发票类型后新增 `能否导出` 列
  - 导出只填 `1-发票基本信息`、`2-发票明细信息`
  - 基本信息只填：发票流水号、发票类型、是否含税、购买方名称、购买方纳税人识别号
  - 明细信息只填：发票流水号、项目名称、商品和服务税收编码、单位、数量、单价、金额、税率
  - 商品和服务税收编码优先用明细字段，明细为空时按 `*分类*` 匹配 `tax_oa_project_categories.service_tax_code`
  - 导出校验：明细必须有项目名称和商品和服务税收编码；已审核/未审核均允许导出
- 验证：
  - 测试库 `现代服务` 样例 `1174205 / CTSH-X2025063008` 已导出验证：项目名称 `*现代服务*水箱清洗`，税收编码 `3079900000000000000`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
  - `cd frontend && source ~/.nvm/nvm.sh && nvm use >/dev/null && npm run build`
- 本次未新增依赖

### 211. 调整税务助手模板导出过滤规则
- 调整：
  - 导出时不再因缺少税收编码/项目名称报明细错误
  - 直接跳过缺少项目名称或商品和服务税收编码的明细，只导出符合条件的数据
  - 已审核/未审核都允许导出
  - 若最终没有任何可导出明细，返回 `暂无导出数据`
- 验证：
  - 测试库 `现代服务` 样例 `1174205 / CTSH-X2025063008` 导出正常，明细税收编码 `3079900000000000000`
  - 无匹配合同号返回 `400 暂无导出数据`
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
- 本次未新增依赖

### 212. 税务助手模板导出补填备注和购买方银行信息
- 调整：
  - 税务助手批量开票模板导出在 `1-发票基本信息` 追加填充：备注、购买方开户银行、购买方银行账号
  - 备注有文字就填；购买方开户银行和购买方银行账号只读取本地独立字段 `buyer_bank_name`、`buyer_bank_account`，没有则留空，不从备注文字推断
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
  - 样例 `1173190` 本地字段存在：银行 `交通银行股份有限公司北京阜外支行`，账号 `03206201101001`，备注有项目名称/项目地址文字
- 本次未新增依赖

### 213. 税务助手模板导出改为仅未审核
- 调整：
  - 批量开票模板导出强制只导出 `audit_status='未审核'` 的发票
  - 列表 `能否导出` 同步增加未审核条件：未审核 + 有明细 + 明细有项目名称 + 能取到商品和服务税收编码
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
- 本次未新增依赖

### 214. 税务助手模板导出改为未审核全集
- 调整：
  - 发票管理导出按钮不再携带页面筛选条件
  - 后端导出接口固定导出全部 `audit_status='未审核'` 的可导出数据
  - 列表展示和当前筛选条件不再影响导出范围
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 215. 税务助手导出项目名称按税务项目分类重组
- 调整：
  - 批量开票模板导出中 `2-发票明细信息.项目名称` 不再直接使用 OA 原始项目名
  - 按 `tax_oa_project_items.raw_name` 精确匹配 OA 原始项目，再用关联分类 `tax_oa_project_categories.project_name` + `tax_oa_project_items.oa_project_name` 生成：`*税务项目分类项目名称*OA项目名称`
  - 例：`*现代服务*水箱清洗` 导出为 `*生产生活服务*水箱清洗`
  - 税收编码优先明细字段，其次项目原始名关联分类编码，最后按原始 `*分类*` 兜底
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/api/endpoints/tax_assistant.py`
  - 测试库未审核导出样例：`1171829` 明细项目名称为 `*生产生活服务*水箱清洗`，税收编码 `3079900000000000000`
- 本次未新增依赖

### 216. 修复钉钉加班时长误解析和调休部门下拉重复
- 问题：
  - 正式环境孙健加班明细出现 `2026` 天，是钉钉审批外层“加班”组件被当成时长字段，年份 `2026` 被误取，并乘以 8 入库为 `16208` 小时
  - 调休统计部门下拉同时出现 `研发中心-给排水部/自控部` 与短部门名
- 调整：
  - `backend/app/services/dingtalk_attendance.py`
    - 递归解析 DDBizSuite 内层组件
    - 时长只从“时长/小时”字段读取
    - 异常大时长回退到开始/结束时间差
  - `backend/app/services/wecom_attendance.py`
    - 新增部门归一化：`研发中心-给排水部 -> 给排水部`、`研发中心-自控部 -> 自控部`
  - `frontend/src/pages/wecom_attendance/index.tsx`
    - 部门下拉展示名同步归一化，并对已有选项按展示名去重
- 正式环境数据处理：
  - 已修 `wecom_attendance_oa_approved_records` 脏数据 5 条：孙健 4 条、高振兴 1 条，均改为 `1.0` 小时
  - 已修 `dingtalk_approval_event_logs.raw_duration=1.00`，并标记历史修正说明
  - 孙健钉钉调休余额已按系统统计修为 `0.9` 天；高振兴当前为 `0.0` 天
  - 复查正式库 2026 年 `record_type='加班' AND duration>=100` 为 `0` 条
- 验证：
  - `cd backend && /opt/anaconda3/envs/smart/bin/python -m py_compile app/services/dingtalk_attendance.py app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 217. 售前营销月度回款看板逻辑复查
- 结论：
  - 数据源为 OA 回款通知主表 `formtable_main_117`，通过 `backend/scripts/sync_presales_payment_dashboard_rows.py` 同步到本地 `presales_payment_dashboard_rows`
  - 后端接口位于 `backend/app/api/endpoints/presales_payment_dashboard.py`，提供 `summary`、`rows`、`sync`
  - 汇总和明细按 `COALESCE(payment_time, apply_date)` 做年度/月度过滤；总额为 `SUM(payment_amount)`，合同数为 `COUNT(DISTINCT contract_no)`
  - 普通员工按 `submitter_oa_user_id = 当前用户 oa_resource_id` 查看本人提交数据，管理员权限 `app:presales_payment_dashboard:admin` 可查看全部
  - 前端页面 `frontend/src/pages/presales_payment_dashboard/index.tsx` 当前展示当年年度回款、合同数量、回款记录、最近 20 条；管理员后台仅额外展示申请人 ID
- 本次仅排查逻辑，未改业务代码，未新增依赖

### 218. 售前营销月度回款看板独立看板、目标管理和列表分页
- 调整：
  - `/apps/presales-payment-dashboard` 新增“回款看板”按钮，跳转 `/apps/presales-payment-dashboard/kanban`
  - 独立看板参照售后管理看板蓝青绿色调，按部门统计回款、目标和完成率，不按个人排行
  - OA 同步明细表 `presales_payment_dashboard_rows` 保持到个人粒度不变，员工仍只看本人回款
  - 普通页面列表改为“回款列表”，不再叫“最近回款”，每页 20 条并支持上一页/下一页
  - 新增 `presales_payment_department_monthly_targets` 保存部门 1-12 月回款目标；前端按万元录入，后端按元保存
  - 管理后台部门固定来自当年回款明细申请部门，不支持新增、删除或改部门名，只维护月度目标
  - 聊天窗口 Markdown 代码块默认折叠，支持展开/收起和复制
- 数据库：
  - Alembic 迁移：`k3l4m5n6o7p8_create_presales_payment_department_targets.py`
  - 建表脚本：`backend/sql/create_presales_payment_department_monthly_targets.sql`
  - 已直接补建 test/prod 两套库目标表，当前均存在且为空表
  - 已更新 `doc/database_dictionary.md`
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/presales_payment_dashboard.py app/models/presales_payment_dashboard.py app/models/__init__.py`
  - `cd backend && conda run -n smart alembic heads`
  - `cd backend && conda run -n smart alembic upgrade head`
  - `cd frontend && ./build.sh`
  - `git diff --check`
- 本次未新增依赖

### 219. 税务助手导出补填购买方地址和电话
- 调整：
  - 批量开票模板导出 `1-发票基本信息` 增加填充购买方地址、购买方电话
  - 字段来源为本地同步后的开票申请购货单位地址/电话：`buyer_address`、`buyer_phone`
  - 银行和账号继续只按独立字段存在时填写，备注有文字则填写
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/tax_assistant.py`
  - 测试库未审核导出抽查：`1174829` 已写入地址 `北京市海淀区马甸冠城园北园19号服务中心`、电话 `18810541544`
- 本次未新增依赖

### 220. 税务助手供货单位下拉和导出补列
- 调整：
  - 新增 `GET /api/tax-assistant/invoice-sales-units`，从 `tax_invoice_basic_info.sales_unit_name` 去重读取供货单位下拉选项
  - 发票管理“供货单位”筛选由输入框改为下拉框，选项来自数据表内容
  - 列表筛选供货单位改为等值匹配
  - 批量开票模板导出读取当前供货单位筛选过滤数据，Excel 内容不新增列
  - 导出下载文件名以供货单位名称开头，不带“税务助手”；筛选为全部时，文件名写 `全部`
  - 导出明细项目名称保持直接使用 OA 对应项目名 `tax_oa_project_items.oa_project_name`
- 排查：
  - `1175230` 开票金额负数不是同步问题
  - test/prod 本地 `tax_invoice_basic_info.invoice_amount` 均为 `-2533438.80`
  - OA 主表 `ecology.formtable_main_33.kpje` 原始值为 `-2533438.80`
  - OA 明细 `ecology.formtable_main_33_dt1.kpje` 原始值为 `-2533438.80`
  - 合同 `CT2022103101147%` 本地 test/prod 当前各同步 6 条未审核；OA 原始表当前同批也有 6 条，另有 2023 年历史 3 条已归档/申请人确认，不是本地重复生成
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/tax_assistant.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 221. 税务助手发票管理分页和筛选总统计
- 调整：
  - 发票管理列表新增分页组件，固定每页 50 条
  - 搜索、重置后回到第 1 页；刷新和翻页只影响发票列表
  - `/api/tax-assistant/invoices` 新增 `summary`，按当前筛选条件全量统计未审核数、已审核数、开票金额合计
  - 前端三个统计卡改为读取筛选总统计，不再按当前页 50 条计算
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/tax_assistant.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 222. 税务助手导出固定填充地址电话银行账号展示
- 调整：
  - 批量开票模板导出 `1-发票基本信息` 固定填充 T 列：`展示地址、电话、开户银行及银行账号`
  - 固定填充 AD 列：`展示地址、电话、开户银行及银行账号`
  - X 列“报废产品销售类型”不填写
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/tax_assistant.py`
- 本次未新增依赖

### 223. 税务助手横向滚动条改用通用浮动组件
- 调整：
  - 税务助手发票列表、发票项目分类列表、发票明细弹窗表格、分类详情弹窗表格统一改用 `FixedBottomScrollbar`
  - 横向滚动条规则统一为：原表格横向滚动条不可见时显示底部浮动滚动条，原生横向滚动条进入视口后自动隐藏
  - `AGENTS.md` 已新增通用规则：以后业务页面出现列宽超过屏幕的横向滚动表格，必须优先调用 `frontend/src/components/ui/fixed-bottom-scrollbar.tsx` 或底层 hook，不在业务页重复手写浮动层
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 224. 软件部任务工具康鹏新建工作记录默认值
- 调整：
  - 软件部任务工具“工作记录”新建弹窗中，当前用户识别为康鹏时，项目默认匹配并选中 `晨天AI中台`
  - 康鹏新建工作记录时，工时默认填 `8`
  - 该默认值仅作用于普通“新建工作记录”，不影响编辑工作记录，也不覆盖从任务带入的添加任务记录
- 验证：
  - `cd frontend && ./build.sh`
- 本次未新增依赖

### 225. 月度回款看板当月口径与部门柱状图
- 调整：
  - 看板顶部指标、合同回款排行、部门回款完成情况和最近回款统一只取当前月份数据
  - 单月汇总时，部门目标金额改为只累计当月目标，不再误用全年目标
  - 部门完成情况按完成率降序排列，保持原紧凑排行样式，展示当月目标、当月实际、完成率和待回款
  - 汇总接口新增 `department_name` 参数，支持按部门读取汇总数据
  - 看板页面保留“月度回款 / 目标”年度趋势图和部门筛选，不展示部门完成率图
  - “部门回款完成情况”更名为“部门回款完成排行”，最近回款放在右侧部门排行下方
  - 新增手机竖版推送图，只展示部门完成排行和横向完成率图；合同回款排行由 8 条增加为 9 条
  - 新增 `GET /api/presales-payment-dashboard/dashboard-image`，支持按年月生成 750px 宽 JPEG 推送图；企微推送任务暂未配置
  - 推送图改为移动端售后同款结构，标题为“月度回款看板”：顶部为业绩目标四项指标，部门排行使用交替行表格，底部“部门完成率”使用横向进度条
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/presales_payment_dashboard.py`
  - `cd frontend && ./build.sh`
  - `git diff --check`
- 本次未新增依赖或数据库结构变更

### 226. 人事考勤页面与导出实际出勤口径统一
- 排查：
  - 正式库李海涛 `2026-07-11` 的日汇总实际出勤为 `0`，钉钉已审批调休为 `1` 天
  - 页面会实时合并本地 OA/钉钉审批数据，花名册导出原先只读取 `wecom_attendance_daily_records`，导致页面 `25`、导出 `24`
- 调整：
  - 后端新增公共 `_apply_attendance_source_adjustments`，统一合并 OA/钉钉请假及加班数据
  - 后端新增公共 `_effective_actual_attendance`，页面接口和花名册导出共用有效实际出勤公式
  - 页面接口返回 `effective_actual_attendance`，前端汇总和明细优先使用后端结果
- 验证：
  - `conda run -n smart python -m py_compile backend/app/api/endpoints/wecom_attendance.py`
  - `cd frontend && ./build.sh`
  - 正式库调用公共导出汇总方法复算，李海涛调休已纳入实际出勤
- 本次未新增依赖或数据库结构变更

### 227. 人事考勤全部汇总字段统一与审批镜像去重
- 排查：
  - 正式库张桐实际年假为 `2` 天、调休为 `2` 天
  - 花名册导出查询未携带 `oa_leave_json`，无法识别日汇总已包含年假
  - 本地审批镜像中同一申请单存在 3 份历史 source key 记录，导出将原有 `2` 天与重复 `6` 天相加，错误显示为 `8` 天
- 调整：
  - 公共来源合并方法自动补齐日汇总 OA 请假和考勤修正标记
  - 本地审批镜像按申请单、人员、日期、类型和时长语义去重
  - 新增唯一后端人员日汇总方法，页面和 Excel 共用全部假期、实际出勤、旷工、未打卡、迟到早退、外勤、补卡和加班统计
  - 页面汇总直接使用接口返回的 `attendance_summary_items`
  - 日汇总接口缓存版本升级为 `v11`
- 验证：
  - 正式库复算张桐年假 `2`、调休 `2`，其余假期 `0`
  - `conda run -n smart python -m py_compile backend/app/api/endpoints/wecom_attendance.py backend/app/services/wecom_attendance.py`
  - `cd frontend && ./build.sh`
- 本次未新增依赖或数据库结构变更

### 228. 花名册导出统一使用最后一次上传模板
- 调整：
  - 上传花名册时拆分部门列纵向合并单元格，并将部门名称填充到合并范围内每一行
  - 规范化后的完整工作簿保存到 `wecom_attendance_roster_uploads.template_data`
  - “导出考勤汇总”直接读取最后一次上传模板，和上传后自动下载使用同一工作簿及填充逻辑
  - 不再通过静态模板重建人员行，系统未识别人员保留原姓名、部门和表格结构，统计列填 0
  - 修复 `daily-records` 仍调用已删除 `_load_latest_current_month_roster_expected_days` 导致的 500
- 数据库：
  - 新增迁移 `s0t1u2v3w4x5`，为上传记录增加可空 `LONGBLOB template_data`
  - test/prod 均已执行迁移到 `s0t1u2v3w4x5`
  - 历史上传记录没有模板二进制，发版后需重新上传一次花名册
- 验证：
  - 后端相关文件 `py_compile` 通过
  - 使用 250 行合并部门模板实测：部门合并全部拆分并逐行补齐；无匹配人员时 250 行姓名和部门均完整保留
  - `git diff --check` 通过
- 本次未新增依赖

### 229. 人事考勤花名册导出复用考勤汇总数据源
- 排查：
  - `/roster/export` 和 `/roster/fill` 虽然已共用花名册模板填充逻辑，但 `_fill_roster_workbook` 内仍单独调用 `_load_attendance_summary_by_department_name`
  - 这会让导出和考勤汇总页面继续存在两套汇总 SQL/计算链路，张桐这类人员可能出现“考勤汇总未打卡 0、导出未打卡 3”的不一致
- 调整：
  - 新增 `_summary_map_from_daily_records_payload`，将 `/daily-records` 返回的 `attendance_summary_items` 转为花名册填充所需映射
  - `export_wecom_attendance_roster` 和 `fill_wecom_attendance_roster` 均先调用 `list_wecom_attendance_daily_records`
  - `_fill_roster_workbook` 移除数据库参数和内部汇总查询，只按传入的统一汇总结果写入 Excel
  - 导出与上传填充现在复用考勤汇总同一份后端数据源，不再维护第二套汇总 SQL
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
  - `git diff --check`
- 本次未改前端，未新增依赖或数据库结构变更

### 230. 移动端 AI 中台首页头像与最近访问
- 调整：
  - 移动端首页顶部右侧改为展示当前用户头像；无头像时显示姓名首字
  - 去掉右上角退出按钮
  - 去掉“当前角色”模块
  - 原位置改为“最近访问”，点击移动端卡片时写入 `localStorage` 最近访问记录
  - 最近访问最多显示 4 个当前仍可见的移动端卡片，点击可直接进入
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
- 本次未新增依赖或数据库结构变更

### 231. 移动端 AI 中台首页排行榜卡片
- 调整：
  - 新增 `GET /api/dashboard-rankings`
  - 当前返回“稳如泰山区域”排行，用于后续继续扩展其它排行类型
  - 统计口径：只统计一级部门；一级部门及其子部门下的用户统一归属到一级部门
  - 排序规则：离职人数少的部门优先；离职人数相同，入职/招聘人数多的部门优先；再按总人数和部门排序兜底
  - 移动端 AI 中台首页在“最近访问”和“应用入口”之间新增“排行榜”卡片，展示前 5 个一级部门
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/dashboard_app_cards.py`
  - `git diff --check`
  - `cd frontend && ./build.sh` 未通过，阻塞原因为当前工作区已有 `src/App.tsx` 引用 `./pages/admin/BadgeManagementPage`，但该页面文件不存在
- 本次未新增依赖或数据库结构变更

### 232. 正式库张辉考勤汇总 20 分钟内排查
- 排查：
  - 直接查询正式库 `smart-cs-ai`
  - 张辉 `2026-06-25` 至 `2026-07-28` 汇总为：`late_early_within_20=0`、`late_early_20_40=0`、`late_early_40_60=0`
  - 同期 `attendance_correction_count=2.5`
  - 有修正日期：`2026-07-09`、`2026-07-10`、`2026-07-11`、`2026-07-22`
  - `2026-07-22` 原始上班打卡 `09:01:57` 为 `时间异常`，但 OA 考勤修正记录为上班补卡 `07:50`，正式库日汇总迟到早退三类均已是 0
  - 正式 Redis DB2 当前未扫到 `wecom:attendance:daily-records:*` 缓存键
- 结论：
  - 正式数据库里张辉没有 20 分钟内迟到/早退记录
  - 用户提供的正式接口返回中 `2026-07-22` 实时重算出了 `late_early_within_20=1`
- 修复：
  - 根因是接口按打卡明细重算迟到时，只看到了原始 `09:01:57 时间异常`，没有按同一天 OA 上班补卡 `07:50` 抵扣同一上班时段
  - `_late_bucket_counts` 新增补卡时段识别，OA 考勤修正覆盖的上班/下班时段不再计入迟到早退
  - 后台重算任务同步识别 `oa_correction_json` 和 `oa_correction` 明细，避免以后重算又写回错误迟到桶
- 验证：
  - 使用用户提供的接口 JSON 复算 `2026-07-22`，结果为 `late20=0`、`late40=0`、`late60=0`
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py`
  - `git diff --check`

### 233. 线上办公助手卡片权限绑定
- 执行：
  - test/prod 两套库确认并补齐权限定义 `app:online_office:access`，描述为 `访问线上办公助手`
  - 两套库均发现两张“线上办公助手”卡片：旧 `smart_spreadsheet` 与新 `online_office`
  - 已统一将两张卡片 `permission_json` 更新为 `"app:online_office:access"`
- 结果：
  - 模板/卡片管理中线上办公助手应能看到权限描述
  - 两套环境卡片权限保持一致

### 249. 查询研发中心及下属部门钉钉调休余额
- 正式环境按钉钉部门树递归查询“研发中心及其下属部门”。
- 匹配 39 人，部门包括：研发中心、软件开发部、自控部、给排水部、测试部。
- 最新余额汇总：
  - 总计：33.5 天 / 268.0 小时
  - 研发中心：2 人，0 天
  - 软件开发部：10 人，9.3 天
  - 自控部：13 人，21.5 天
  - 给排水部：10 人，2.7 天
  - 测试部：4 人，0 天

### 248. 查询研发中心钉钉最新调休余额
- 正式环境查询钉钉最新调休余额，部门筛选为“研发中心”。
- 匹配启用人员 2 人：孙永跃、赵广文。
- 两人余额均为 `0.0 天 / 0.0 小时`。
- 当前系统通过钉钉接口实时查询，没有单独本地余额落库表。

### 247. 补同步钉钉考勤、请假和加班数据
- 执行范围：`2026-07-20` 至 `2026-08-02`。
- prod 已完成：
  - 钉钉考勤打卡入库：748 条
  - 日汇总合并：476 条
  - 钉钉请假：21 条
  - 钉钉加班：14 条
  - 同时执行缺卡重算
- 说明：用户后续要求仅执行 prod；前一步并行执行时 test 已同步同一范围，写入逻辑幂等。

### 246. 客户端管理替换安装包覆盖原地址与 MD5 校验
- 调整：
  - 编辑客户端版本时可选择新安装包；未选择时只保存配置。
  - 替换安装包时覆盖原 `object_key`，不会生成新 OSS 地址，保证已分发出去的永久下载地址继续有效。
  - `client_app_releases` 新增 `file_md5` 字段和 `idx_client_app_releases_file_md5` 索引。
  - 上传和替换安装包时后端自动计算 MD5。
  - 客户端更新接口新增可选参数 `current_file_md5`；如果服务端最新安装包 MD5 与客户端传入 MD5 一致，返回 `has_update=false`。
  - 更新接口返回 `file_md5`；客户端管理列表文件信息下展示 MD5 前 8 位。
  - 同步更新 `backend/sql/create_client_management_tables.sql` 和 `doc/database_dictionary.md`。
- 数据库：
  - 新增迁移 `h0i1j2k3l4m5_add_client_release_file_md5.py`。
  - test/prod 两套库均已升级到 `h0i1j2k3l4m5`。
  - 正式库已有 1 条客户端记录回填 MD5；测试库当前 0 条客户端记录。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/client_management.py app/services/object_storage.py migrations/versions/g9h0i1j2k3l4_add_client_release_download_url_options.py migrations/versions/h0i1j2k3l4m5_add_client_release_file_md5.py`
  - `cd frontend && ./build.sh`

### 245. 客户端管理下载地址永久/有时效配置
- 调整：
  - `client_app_releases` 新增 `download_url_type`、`download_url_expire_seconds`。
  - 下载地址类型支持 `presigned` 有时效、`permanent` 永久。
  - 默认保持原有有时效预签名链接，默认有效期 3600 秒。
  - 永久链接使用 `MINIO_PUBLIC_ENDPOINT + bucket + object_key` 生成，不带 `Signature/Expires`。
  - 客户端更新接口、管理端复制下载地址、下载按钮共用同一后端下载链接生成逻辑。
  - 上传/编辑弹窗增加“下载地址”和“有效期”配置，支持 1小时、12小时、1天、7天、30天。
  - 同步更新 `backend/sql/create_client_management_tables.sql` 和 `doc/database_dictionary.md`。
- 数据库：
  - 新增迁移 `g9h0i1j2k3l4_add_client_release_download_url_options.py`。
  - test/prod 两套库均已升级到 `g9h0i1j2k3l4`。
  - 两套字段复查存在：`download_url_type varchar(20) NOT NULL DEFAULT presigned`、`download_url_expire_seconds int NULL`。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/client_management.py app/services/object_storage.py migrations/versions/g9h0i1j2k3l4_add_client_release_download_url_options.py`
  - `cd frontend && ./build.sh`
  - `git diff --check`
  - 正式环境永久链接示例：`https://zhidao.tjchentian.com:9091/ctupload/client_management/releases/20260731/test.exe`

### 244. 客户端管理列表复制下载地址与状态切换
- 调整：
  - 客户端管理列表点击文件名会请求该版本下载地址并复制完整 URL。
  - 复制成功通过全局 toast 提示“复制下载地址成功”。
  - 状态列从纯文本改为可点击按钮，支持启用和停用双向切换。
  - 状态列设置固定宽度和 `whitespace-nowrap`，避免内容换行。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`

### 243. 正式环境 OSS 文件下载域名配置
- 调整：
  - 新增配置项 `MINIO_PUBLIC_ENDPOINT`，专门用于生成 MinIO/S3 预签名下载链接。
  - 内部上传、读取、删除仍使用 `MINIO_ENDPOINT`，避免把 S3 API 连接切到公网业务域名。
  - `backend/config/env_prod` 配置为 `https://zhidao.tjchentian.com:9091`。
  - `backend/config/env_test` 同步新增空配置项，测试环境行为不变。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/services/object_storage.py app/core/config.py`
  - `git diff --check`
  - 正式环境生成链接前缀：`https://zhidao.tjchentian.com:9091/ctupload/...`

### 234. 钉钉缺少上下班记录时补缺卡占位
- 排查：
  - 张永勋 `2026-07-10` 钉钉只有 `OnDuty` 上班真实打卡，没有 `OffDuty` 下班记录。
  - 原逻辑只统计钉钉/企微显式返回的“缺卡、未打卡”，钉钉直接不返回另一侧记录时不会推断缺卡。
- 调整：
  - 钉钉同步保存时按启用人员和同步日期范围检查 `OnDuty/OffDuty`，缺少任一侧就生成稳定 ID 占位：`dingtalk_missing:{user}:{date}:{OnDuty|OffDuty}`。
  - 占位同步写入 `dingtalk_attendance_records` 和 `attendance_checkin_records`，状态为“上班未打卡”或“下班未打卡”。
  - 后续真实打卡同步到同一用户、日期、上下班槽位时，会先删除对应占位，再写入真实记录，实现“之后有了就覆盖”。
  - 钉钉占位缺卡不计为实际出勤。
  - 后台缺卡重算增加历史兜底：即使明细表还没有占位，也会从当天钉钉明细自动推断缺少的上下班槽位。
- 历史数据：
  - 汇总数据可通过 `recalculate_missing_punch_counts` 兜底修正。
  - 如需详情里也出现补出的“未打卡”明细，需要重新跑对应日期范围的钉钉同步或执行一次占位补录。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/tasks/scheduler.py`
  - `git diff --check`
  - 函数级校验：一条 `OnDuty` 会生成 `OffDuty 下班未打卡` 占位。

### 235. 移动端首页排行榜独立权限
- 调整：
  - 新增权限 `app:mobile_dashboard_rankings:access`
  - 后端 `GET /api/dashboard-rankings` 改为 `requires_permissions` 校验
  - 前端移动端首页按权限决定是否请求并渲染排行榜模块，默认无权限即隐藏
  - 新增 Alembic 迁移 `d6e7f8a9b0c1`，仅插入权限数据，不自动授予角色
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/dashboard_app_cards.py app/permissions.py migrations/versions/d6e7f8a9b0c1_add_mobile_dashboard_ranking_permission.py`
  - `cd backend && set -a && source config/env_test && set +a && conda run -n smart alembic upgrade head`
  - `cd backend && set -a && source config/env_prod && set +a && conda run -n smart alembic upgrade head`
  - `cd frontend && ./build.sh`

### 236. 页面数据处理统一后端执行
- 规则补充：
  - 列表筛选后的统计、分组、去重、合并、汇总、排行、导出、图表口径等数据处理，默认放到后端接口或 service 执行
  - 页面只做展示和轻量格式化，不再在前端重复计算口径

### 237. 移动端首页排行榜临时隐藏
- 调整：
  - 移动端首页移除排行榜模块展示
  - 移除进入页面时对 `/dashboard-rankings` 的请求
  - 后端排行榜接口和权限定义保留，后续需要时可恢复前端展示
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`

### 238. 钉钉调休明细重复与余额扣减修复
- 排查：
  - 张永勋调休明细重复不是页面展示问题，`wecom_attendance_oa_approved_records` 中同时存在钉钉“假期状态/余额扣减记录”和“审批实例记录”。
  - 同一人、同一假期类型、日期重叠时，两类记录被重复写入本地镜像，调休余额统计也会重复扣减。
- 调整：
  - 钉钉请假同步合并规则改为优先保留假期状态记录；审批实例记录若与假期状态记录同人、同假期类型、日期重叠，则不再写入本地镜像。
  - 本地 source key 生成时统一规范化部门名称，减少同一记录因部门别名不同产生不同 key。
  - 调休统计接口在后端按日粒度去重，页面不参与统计口径处理。
  - 新增 `backend/scripts/cleanup_dingtalk_leave_overlaps.py`，用于清理历史钉钉请假重复/重叠镜像。
- 数据清理：
  - prod 清理 169 条，复查 `delete=0`
  - test 清理 169 条，复查 `delete=0`
  - 正式库张永勋 2026-01-01 至 2026-05-31 复算调休扣减为 13.0 天
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/services/dingtalk_attendance.py app/services/wecom_attendance.py app/api/endpoints/wecom_attendance.py scripts/cleanup_dingtalk_leave_overlaps.py`
  - `git diff --check`

### 239. 宫庆超考勤修正覆盖未打卡展示
- 排查：
  - 宫庆超 2026-07-02 已有 OA 考勤修正上班打卡，但每日考勤弹窗仍展示原始企微“上班未打卡”。
  - 后端虽然把 OA 修正追加进 `checkin_records`，但前端详情展示仍直接渲染原始缺卡明细，没有按修正槽位过滤掉同一天同方向的未打卡。
- 调整：
  - 口径已改回后端统一处理：后端 `checkin_records` 先收集当天 OA 修正的上班/下班方向，再过滤掉同方向的原“缺卡/未打卡”记录。
  - 前端已撤掉重复过滤，只展示后端返回结果。
  - 每日考勤、缺卡详情、导出都复用同一份后端结果。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/tasks/scheduler.py`
  - `cd frontend && ./build.sh`
  - `git diff --check`
  - Node `v20.20.2`

### 240. 宫庆超 2026-07-03 考勤修正重复与未打卡归零
- 排查：
  - 同一条下班考勤修正在每日考勤里出现两次，且考勤修正后当天未打卡没有归零。
- 调整：
  - 后端统一去重 OA 修正明细，按请求/明细/日期和槽位去重，避免同一上班或下班修正重复生成两条。
  - 后端统一处理有效明细，OA 修正命中后，同槽位原未打卡会被过滤，页面和导出共用同一份结果。
  - 当当天没有剩余未打卡时，未打卡数会归零，日汇总同步归零。
- 验证：
  - `conda run -n smart python -m py_compile app/api/endpoints/wecom_attendance.py app/services/wecom_attendance.py app/services/dingtalk_attendance.py app/tasks/scheduler.py`
  - `git diff --check`

### 241. 客户端管理与应用升级接口
- 调整：
  - 新增“客户端管理”卡片，归属软件开发部，独立权限 `app:client_management:access`。
  - 新增客户端版本管理表 `client_app_releases`，每个应用通过 `app_code` 独立管理版本。
  - 版本号拆分为外部版本号 `external_version`（如 `1.0.0`）和内部版本号 `internal_version`（如 `100`），升级判断读取内部版本号。
  - 管理端支持上传客户端、平台、强制更新、使用部门多选、启停和下载；文件优先保存 OSS 独立目录 `client_management/releases/...`。
  - 新增免登录升级接口：`GET /api/client-management/check-update?app_code=...&platform=...&internal_version=...`，有新版本时返回下载地址、版本号和强制更新标记。
  - 客户端管理列表增加部门多选筛选；未配置使用部门的版本按全部部门处理，筛选任意部门时仍展示。
- 数据库：
  - 新增 SQL：`backend/sql/create_client_management_tables.sql`
  - 新增迁移：`f8a9b0c1d2e3_create_client_management.py`
  - test/prod 两套库均已执行建表、权限、卡片和 Alembic 升级到 `f8a9b0c1d2e3`。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/client_management.py app/api/api.py app/permissions.py app/db_init.py migrations/versions/f8a9b0c1d2e3_create_client_management.py`
  - `cd backend && conda run -n smart python -m py_compile app/api/endpoints/client_management.py`
  - `cd frontend && ./build.sh`
  - `git diff --check`

### 242. 税务助手购买方名称和税号空格清洗
- 排查：
  - 发票申请 `1175780` 在 OA `formtable_main_33` 中 `dwmc` 为空、`dwmc1=900380`，旧逻辑将 `dwmc1` 当购买方名称保存。
  - 该单税号 OA 原始值为 `9112 0101 MACH AJ9L XG`，本地旧数据保留了内部空格。
  - OA 客户开票信息 `formtable_main_97` 与合同台账 `uf_httz.khwb` 均能解析到单位名称 `天津和平新城吾悦广场商业管理有限公司`。
- 调整：
  - `dwmc1` 仅保存为 `buyer_code`，不再优先作为购买方名称。
  - 购买方名称优先按清洗后税号从 `formtable_main_97.mc` 解析，合同编号再从 `uf_httz.khmc/khwb` 兜底。
  - 本地保存购买方名称、购买方公司名称、税号、地址、电话、开户行、银行账号时统一去掉所有空白字符。
- 数据处理：
  - test/prod 已手动执行税务 OA 发票同步。
  - 两套 `1175780` 已更新为购买方 `天津和平新城吾悦广场商业管理有限公司`，税号 `91120101MACHAJ9LXG`。
  - 两套 `tax_invoice_basic_info` 购买方相关字段含空格数量均复查为 `0`。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/services/tax_assistant.py app/api/endpoints/tax_assistant.py`
  - `git diff --check`

### 243. 正式环境研发中心调休余额核对与修正
- 执行范围：研发中心及下属部门人员，共 35 人
- 统计区间：`2026-01-01` 至 `2026-08-03`
- 成功回写 6 人：
  - 朱文波：`0.3→1.4` 天
  - 沈孟男：`0.2→0.3` 天
  - 张杰：`3.6→4.6` 天
  - 李海涛：`6.3→5.3` 天
  - 苗会亮：`0→0.3` 天
  - 史延明：`4.0→4.1` 天
- 8 人因本地计算余额为负数未回写：刘雁鹏、贠丹丹、赵本德、郭丽霞、孔德明、张永勋、程金雨、高振兴
- 负数原因：调休使用天数大于加班累计天数，钉钉余额不支持负数，因此保留钉钉当前余额
- 处理结果：钉钉余额接口已完成核对；部门树接口曾出现超时，最终使用本地同步人员名单完成处理

### 244. 客户端管理独立上传与固定地址覆盖
- 文件上传流程改为独立操作：
  - 客户端文件字段右侧增加“上传”按钮
  - 使用 Axios `onUploadProgress` 展示实时上传百分比
  - 上传成功后显示文件名和 MD5 前 8 位
- 后端新增：
  - `POST /api/client-management/releases/upload-file`：新建版本先上传文件，生成稳定对象地址并返回文件元数据
  - `POST /api/client-management/releases/{release_id}/file`：编辑版本直接覆盖原 `object_key`，不改变已分发地址
  - 本地存储增加临时上传文件访问接口
- 保存行为：
  - 编辑版本未上传新文件时，只保存版本配置
  - 新建版本必须先上传文件，保存时仅写入配置和已上传文件信息
  - 同一表单再次上传会复用原稳定对象路径，覆盖之前文件
- 验证：
  - `conda run -n smart python -m py_compile backend/app/api/endpoints/client_management.py`
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
  - `git diff --check`

### 245. 客户端管理状态字段样式统一
- 调整：
  - “启用”复选框改为“启用状态：启用 / 未启用”单选格式
  - “是否强制更新”复选框改为“强制更新状态：是 / 否”单选格式
  - 新建客户端版本默认未启用
  - 后端新建接口补充 `is_enabled` 表单字段，按前端选择落库
- 验证：
  - `conda run -n smart python -m py_compile backend/app/api/endpoints/client_management.py`
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
  - `git diff --check`

### 246. 地下水登记造册台账正式数据覆盖测试
- 执行：使用正式库 `smart-cs-ai.groundwater_registry_records` 的表结构和数据覆盖测试库 `smart-cs-ai-test.groundwater_registry_records`。
- 结果：测试库覆盖后总数 1793 条，已填写 1337 条，已完成 0 条，与正式库一致。
- 分区复查：宁河区 652 条、滨海新区 374 条、静海区 128 条、武清区 121 条、北辰区 120 条等均已同步。
- 测试库覆盖前备份：`/tmp/groundwater_registry_records_test_backup_before_prod_cover_20260803_162513.sql`。

### 247. 地下水登记造册台账管理列表列宽压缩
- 调整：
  - 管理列表表格改为 `colgroup + minWidth + width: 100%` 固定最小列宽，避免被内容撑开，页面空间允许时可自适应展开。
  - `2026改造` 表头强制两行显示为 `2026 / 改造`，列宽调整为 46px。
  - `接口类型`、`远传终端`、`传水资源`、`超周期` 最小列宽 38px，空间允许时可展开。
  - `取水权人` 保持 64px 截断，鼠标悬停显示完整内容。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
- Git：
  - `2026-08-03 17:04 CST` 准备提交并推送地下水台账管理列表列宽压缩改动。

### 248. 地下水登记完整导出任务列表 500 修复
- 问题：
  - 正式日志显示 `GET /api/dengji/records/full-export-tasks?page=1&page_size=10` 返回 500。
  - 报错为 `PydanticSerializationError: Unable to serialize unknown type: GroundwaterRegistryExportTask`。
- 修复：
  - 新增 `GroundwaterRegistryExportTaskOut`、`GroundwaterRegistryExportTaskListOut`。
  - `list_full_export_tasks` 增加 `response_model=GroundwaterRegistryExportTaskListOut`，返回可序列化 schema。
- 验证：
  - `conda run -n smart python -m py_compile app/api/endpoints/groundwater_registry.py app/schemas/groundwater_registry.py`
  - 使用 `GroundwaterRegistryExportTask` ORM 对象构造 `GroundwaterRegistryExportTaskListOut` 并 `model_dump(mode='json')` 通过。

### 249. 前端通用表单控件高度统一
- 调整：
  - `Button` 默认高度从 32px 调整为 40px，与 `Input`、`AppSelect`、`AppCreatableSelect` 对齐。
  - `Button` 的 `lg` 高度调整为 44px，保留 `xs/sm/icon` 小尺寸用于紧凑表格和图标操作。
  - `AppSelect`、`AppCreatableSelect` 默认 control/value/indicator 高度固定为 40px 体系，避免筛选栏按钮、下拉、输入框高度不一致。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`

### 250. 全项目同类文字按钮切换公共 Button
- 调整：
  - 地下水台账 PC 页签 `列表 / 完整导出` 改为调用公共 `Button`。
  - 自控部、软件部、给排水助手、营销部、售后部、采购部中命中的筛选、重置、导出、新建、弹窗确认/取消等文字按钮改为调用公共 `Button`。
  - 保留下拉菜单选项、图标小按钮、练习题号按钮等非同类紧凑交互，不强行套默认 40px 按钮。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
- Git：
  - `2026-08-03 17:30 CST` 准备提交并推送通用控件高度与同类文字按钮统一改动。

### 251. 地下水登记台账管理列表返回地址修复
- 问题：
  - 从 `/dengji?admin=1` 管理列表进入填写页后，返回按钮跳到了 `/dengji`，丢失管理参数。
- 修复：
- 管理列表进入编辑页时追加 `?fromAdmin=1`。
- 编辑页识别 `fromAdmin=1` 或 `admin=1` 后，顶部返回、底部返回和无效记录兜底返回统一跳 `/dengji?admin=1`。
- 普通列表进入编辑仍返回 `/dengji`。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
- Git：
  - `2026-08-04 09:56 CST` 准备提交并推送地下水登记台账管理列表返回地址修复。

### 252. 税务助手发票文件管理与复制下载地址
- 新增发票文件管理 Tab，支持批量上传、OCR 识别、结果保存、重新识别、删除和下载。
- 新增表 `tax_invoice_files`，OCR 复用 `recognize_invoice_file`；test/prod 均已升级到 `k2l3m4n5o6p7`。
- 发票文件使用独立 MinIO bucket：test 为 `tax-invoices-test`，prod 为 `tax-invoices`，两套 bucket 均已确认存在。
- 发票文件名点击后自动复制在线下载地址 URL，成功提示“复制下载地址成功”。
- 移除企微回调后的二次 `snsapi_privateinfo` 授权跳转，避免回调域名提示。
- 验证：后端 `py_compile`、test/prod `alembic current`、前端 `./build.sh`（Node `v20.20.2`）、`git diff --check`。

### 253. 税务助手发票文件批量上传弹窗
- 调整：
  - 发票文件管理页移除页面内大块拖拽上传区。
  - 点击“批量上传”后打开弹窗，弹窗内支持拖拽或点击选择多个 PDF/JPG/JPEG/PNG 文件。
  - 多文件选择后自动逐个上传并 OCR，每个文件独立展示进度、成功或失败状态。
  - 上传过程中禁止关闭弹窗，避免进度丢失。
  - 文件名复制在线下载地址增加剪贴板失败兜底和错误提示；MinIO 文件复制成功提示为“复制 OSS 下载地址成功”。
- 验证：
  - `cd frontend && ./build.sh`
  - Node `v20.20.2`
  - `git diff --check`

### 254. 税务助手发票文件列表与票号判重
- 追加修复：批量上传完成后显式保持在“发票文件管理”Tab，并修正顶部副标题。
- 调整：
  - “发票文件管理”Tab 仅系统管理员可见；后端相关文件接口同步加管理员校验。
  - 发票文件管理筛选、搜索、导出、批量上传固定同一行；宽度不足时横向滚动。
  - 上传流程改为先 OCR 识别，再按发票号码检查重复；重复票号不保存文件、不入库。
  - 同一批次内重复票号也会直接失败；重新识别时同样禁止改成已存在票号。
  - 扫描匹配方法改为本地未命中时直查 OA `formtable_main_33_dt1.fph`，并补写本地镜像供列表显示项目、金额和申请人。
  - OA 同步补抓近期已归档且已有发票号的开票申请，避免未曾同步过就归档的单据漏同步。
  - test/prod 均已升级到 `l3m4n5o6p7q8`，新增匹配状态/推送状态字段。
- 排查：
  - test 库 `tax_invoice_files` 当前 4 条旧文件，列表 SQL 可正常返回。
  - prod 库 `tax_invoice_files` 当前 0 条，非列表查询清空导致。
  - 正式库漏 `26122000001047992611` 的原因：旧同步只抓当前处理人刘羽丰及已跟踪历史；该 OA 单据 `1176649` 已归档且之前未进入本地镜像。
  - 已执行正式库税务 OA 同步补齐：`1176649` 项目 `CTRD-20260709-天津市北辰区御龙湾-水箱清洗`、金额 `9000.00`、申请人 `范运成`。
- 验证：
  - 后端 `py_compile`
  - `cd frontend && ./build.sh`，Node `v20.20.2`
  - `git diff --check`

### 255. 税务助手发票文件扫描推送计划任务
- 调整：
  - 发票文件扫描推送任务 `tax_invoice_file_notification_push` 配置为 5 分钟一次。
  - 管理后台任务说明更新为“每 5 分钟扫描、默认暂停”。
  - 已恢复为测试环境固定推送康鹏，正式环境按申请人推送；企微 userid 按现有别名规范归一化为 `tangpeng`。
  - 新增 `backend/sql/upsert_tax_invoice_file_notification_push_scheduler_meta.sql`，幂等写入任务元数据。
- 数据库：
  - test/prod 两套库已执行，复查 `trigger_type=interval`、`interval_minutes=5`、`is_paused=1`，描述已同步为“测试环境固定推送康鹏，正式环境推送正常申请人”。
- 验证：
  - `cd backend && conda run -n smart python -m py_compile app/services/tax_invoice_notifications.py app/api/endpoints/admin/scheduler.py app/tasks/scheduler.py app/api/endpoints/tax_assistant.py`
  - `git diff --check`

### 256. 地下水登记造册台账正式覆盖测试
- 执行：
  - 使用正式环境 `groundwater_registry_records` 覆盖测试环境同名表。
  - 覆盖前测试库已备份到 `/tmp/groundwater_registry_records_test_backup_before_prod_sync_20260807_135559.sql`。
- 结果：
  - test/prod 当前均为 1793 条，已填写 1779 条，已完成 1779 条。
- 验证：
  - `mysql` 复查计数一致。

### 257. 地下水登记完整导出改为 ZIP 包
- 调整：
  - 完整导出不再在 Excel 中嵌入图片。
  - 导出目录改为 `<batch_key>/<batch_key>.xlsx` 和 `<batch_key>/images/*`。
  - Excel 的站点照片列改写为 `images/xxx.jpg` 相对路径。
  - 完整导出下载文件改为 `.zip`，接口保持不变。
  - 提交任务后使用 `asyncio.create_task` 立即返回，Excel 生成、图片复制和 zip 打包通过 `asyncio.to_thread` 在工作线程执行，避免阻塞 API 事件循环。
- 验证：
  - 使用测试库 1 条带照片记录生成临时包，zip 内包含 xlsx 与 images，Excel 单元格内容为相对路径。
  - Excel 单元格显示“查看图片1”，并带 `images/...jpg` 超链接。
  - 后端 `py_compile`、`git diff --check`

### 258. 地下水完整导出历史记录清理
- 数据库：
  - test/prod 两套库 `groundwater_registry_export_tasks` 已全部清空，复查剩余记录为 0。
- 文件：
  - 本地 `backend/data/exports/groundwater_registry` 下 7 个历史失败导出文件/目录已删除，目录当前为空。

### 259. 正式环境 OKCIS 售前爬虫补跑第 2-6 页
- 正式库 `crawler_tasks.id=4` / `okcis_notice_presales_marketing` 后续按最多 10 页模式运行。
- 按用户要求临时补跑第 2-6 页：
  - run_id `1527`：第 2-4 页成功，第 5 页登录态失败后中断，状态 `partial`。
  - run_id `1528`：补跑第 5-6 页成功，状态 `success`。
- 原始 JSONL 文件记录第 2-6 页各 50 条，共 250 条；业务表按有效截止时间筛选入库。

### 260. 正式环境 OKCIS 爬虫页数上限调整
- 正式库 `crawler_tasks.id=3` / `okcis_notice_manual` 已调整为 `page_start=1,page_end=10,page_size=50`。
- 正式库 `crawler_tasks.id=4` / `okcis_notice_presales_marketing` 已调整为 `page_start=1,page_end=10,page_size=50`。
- 运行逻辑：最多跑 10 页；若源站最后一页不足或提前无数据，会按现有 `no_more_data` 判断提前停止。

### 261. 软件部任务工具工作记录定时提交写入创建时间
- 调整：
  - 新建工作记录时补充 `scheduled_submit_enabled` / `scheduled_submit_at` 参数。
  - 勾选“定时自动提交”后直接点“提交”，记录的 `created_at` 按定时提交时间写入，不再使用当前提交时间。
  - 定时草稿自动提交路径也统一按定时提交时间落库。
  - 前端提交按钮同步带上定时参数，并在未填写定时时间时阻止提交。
- 验证：
  - 后端 `py_compile`
  - 前端 `cd frontend && ./build.sh`

### 262. 正式环境 crawler_task_4 页数与手动运行排查
- 结论：
  - 正式库 `crawler_task_4` 当前配置为 `page_start=1,page_end=10,page_size=50`，不是固定 1 页。
  - `run_id=1671` 只抓第一页，是因为手动调试运行时传入了 `page_end=1`，覆盖了任务配置。
  - 已通过 Redis 计划任务队列触发正式 scheduler 重新执行，生成 `run_id=1677`。
  - `1677` 当前仍为 `running` 且 `total_requests=0/log_text空`，因代码要等第 1 页详情抓取和入库完成后才回写日志；营销任务详情间隔 20 秒，第一页完成前库里看不到过程。
- 处理：
  - 本地直接触发产生的 `run_id=1675` 未发起页面请求，已人工标记 error，避免误判。
  - 2026-08-11 14:00 正式 cron 已启动 `run_id=1681`。
  - 按用户要求停止手动触发任务：`run_id=1677` 已标记 `stopped`，1 分钟复查未恢复 running。
  - 当前版本没有运行中取消接口，只能标记停止；若协程仍实际执行，需要重启正式 cron/api 进程才能真正杀掉。

### 263. 爬虫任务运行中取消接口
- 调整：
  - 新增取消接口：`POST /api/admin/crawler/tasks/{task_id}/runs/{run_id}/cancel`。
  - 运行中任务先置为 `cancel_requested`，执行循环检测后最终落为 `cancelled`。
  - 通用爬虫分页循环在每页开始和页间暂停期间检查取消状态；长页间隔按 5 秒粒度响应。
  - OKCIS 详情抓取在每条详情前、20 秒详情间隔、权限不足 5/10/20 分钟等待期间检查取消状态。
  - 管理后台爬虫任务详情页为 `running/cancel_requested` 运行记录增加“取消”按钮。
- 验证：
  - 后端 `py_compile`
  - 前端 `cd frontend && ./build.sh`
  - `git diff --check`

### 264. crawler_task_4 卡 START 与重复运行处理
- 排查：
  - 使用正式 `env_prod` 和正式库本地模拟，订阅组前置构建 0.46 秒完成，返回 `dzid=187653`。
  - 本地演示执行 1 页可进入 OKCIS 登录检查，2.87 秒返回登录失效，不存在前置阶段死锁。
  - 正式 `run_id=1683` 后续已推进到第 3 页并入库 9 条；之前只看到 START 是第一页详情抓取耗时且中间日志未及时回写。
- 修复：
  - 同一爬虫任务已有 `running/cancel_requested` 时，新运行生成 `skipped` 记录并说明占用的 run_id，避免手动与定时任务重叠。
  - 超过 12 小时未结束的 active run 自动标记 error。
  - `build_runtime_targets` 完成后立即回写日志。
- 验证：
  - 后端 `py_compile`
  - `git diff --check`

### 265. 招标信息公示发布日期/截止时间排序
- 调整：
  - `/api/okcis/notices` 和 `/api/okcis/notices/export` 新增 `sort_field` / `sort_order` 参数。
  - 排序字段白名单限制为 `publish_date`、`deadline_at`，默认保持 `deadline_at ASC`。
  - 前端列表“发布日期”“截止时间”表头支持点击切换升序/降序。
  - 导出沿用当前列表排序。
- 验证：
  - 后端 `py_compile`
  - 前端 `cd frontend && ./build.sh`
  - `git diff --check`

### 266. crawler_task_4 运行中计数显示修复
- 排查：
  - 正式库 2026-08-12 14:00 的 `crawler_task_4` 运行 `run_id=1720` 当前是 `running`。
  - 日志已进入 `dzid=187653 page=1`，并出现 `[SUCCESS]`，说明不是没抓。
  - 页面显示 `成功0/失败0/总数0` 的原因是计数字段只在任务结束时回写，运行中只更新日志。
- 修复：
  - 每页业务写入成功后实时刷新 `success_requests/failed_requests/total_requests`。
  - DB 写入失败、CrawlerError、Exception 时也实时刷新失败计数。
  - 前端执行记录状态标签改为中文，`running/cancel_requested` 使用蓝色“运行中/取消中”，避免误认为成功。
- 验证：
  - 后端 `py_compile`
  - 前端 `cd frontend && ./build.sh`

### 267. crawler_task_2 kehu51 客户列表密文版本失败处理
- 排查：
  - 正式环境 `crawler_task_2` 从 2026-08-12 09:15 到 16:15 连续失败。
  - 失败点：王贺客户列表预取源站总数 `14` 后，请求 `GetGridData` 返回 `HTTP 500 Unsupported ciphertext version`。
  - 近期成功记录里 `whereSql=E513575924A85785` 可正常抓取；失败记录里预取到 `whereSql=D2X928C17D825F02BC3`，被源站判定为不支持的密文版本。
- 修复：
  - `kehu51_customer_list` 调用列表接口时，如果遇到 `Unsupported ciphertext version`，自动把 `whereSql` 回退为模板里的 `completeSql` 并重试当前页一次。
  - 登录刷新后的二次请求也套用同样回退逻辑。
- 验证：
  - 后端 `py_compile`
  - `git diff --check`

### 268. crawler_task_2 正式环境演示验证通过
- 继续排查：
  - 单纯把 `whereSql` 回退为 `completeSql` 仍失败，因为页面里 `completeSql` 也已变成 `D2X...` 新密文。
  - 对比最新 kehu51 客户列表页面初始化发现，`fixedTable` 新增/依赖 `sqlCondition` 参数。
- 修复：
  - `backend/crawler_sites/kehu51/customer_list.json` 新增从页面提取 `sqlCondition`。
  - 表单请求新增 `sqlCondition` 字段，随 `GetGridData` 一起提交。
- 验证：
  - 使用正式库和正式凭证，演示跑 `crawler_task_2`、负责人王贺 `1555551`、第 1 页。
  - 结果：`success=1, failed=0`；源站总数 `14`，返回数据 `14` 条。
  - 未写正式业务表。

### 269. 软件部工作记录康鹏默认项目改为上次提交项目
- 调整：
  - 新增接口 `/api/software-task/work-records/last-project`，读取当前用户最近一次正式提交工作记录的项目。
  - 康鹏新建工作记录且当天无草稿时，项目默认值优先使用上次提交项目。
  - 有草稿时仍以草稿为准；没有历史项目时保留原默认空值/兜底逻辑。
- 验证：
  - 后端 `py_compile`
  - 前端 `cd frontend && ./build.sh`

### 270. 营销项目流程看板参与范围与时间填写
- 调整：
  - 非卡片管理员只能看本人参与项目：业务负责人本人，或本人为部门配置责任人且负责人部门匹配。
  - `meta` 返回当前用户可编辑部门槽位，非管理员只在自己负责部门列显示时间填写组件。
  - 开始时间默认当天 `08:30`，结束时间默认当天 `17:30`；输入框增加清空按钮，清空后保存空值。
  - 时间默认按“年月日 / 时间”两行展示，未填写显示“填写”，点击后展开紧凑选择控件。
  - 看板序号、项目名称左固定；业务负责人、负责人部门不压缩；任务下发时间固定两行显示。
  - 导出 Excel 使用两级合并表头，基础字段纵向合并两行，部门列横向合并开始/结束时间，并冻结前两行。
  - 筛选栏业务负责人、负责人部门改为按当前权限范围取值的可搜索、可清空下拉框。
  - 管理 Tab 的部门、责任人下拉查询按槽位独立维护，输入一行不会影响其他行。
  - 新增 OA 要求完成时间：同步 `formtable_main_116.yqwcrq` 到 `required_completion_time`，test/prod 全量同步 5276 条，5275 条已回填。
  - 导出模板调整为“天津晨天后台项目进度看板”，增加要求完成时间、两级部门表头和冻结前 3 行。

### 271. OA 项目任务书流程节点与图片附件排查
- 项目编号 `2026081709014`：`requestid=1210214`、`workflowid=369`，当前处于“5.部门会签”。
- 节点及流转记录可通过 `workflow_flownode/workflow_nodebase/workflow_requestlog` 获取。
- 本单所有节点日志附件字段为空，无可对应图片；表单附件 `fj=1121880` 是 ZIP 文件（`IMAGEFILEID=1165834`），不是图片。
- 后续若节点日志存在文档 ID，可通过 `docdetail -> docimagefile -> imagefile` 获得对应文件/图片。
- 审批人和所属部门可由 `workflow_requestlog + hrmresource + hrmdepartment` 获取；本单已完成记录包括石家庄分公司郭德鹏、总经办霍林、研发中心李兆民、采购部刘健。

### 272. 项目任务书采购部会签企微通知
- OA 同步新增识别条件：节点“5.部门会签”、审批人刘健、所属采购部。
- 使用 `workflow_requestlog.logid` 去重；新会签由 10 分钟同步任务触发企微通知。
- 测试环境推送康鹏；正式环境推送管理 Tab 中“采购部”的配置责任人。责任人未配置时保留待推送并自动重试。
- test/prod 已回填采购会签日志；各 896 条历史记录已设通知基线，不会首次上线补发。
- 验证：后端 `py_compile`、空队列扫描 `sent=0/pending=0/failed=0`、`git diff --check` 通过。
  - 新增部门时间填写人字段，保存时记录当前用户。
  - OA 同步脚本不再覆盖人工填写的部门开始/结束时间。
  - test/prod 均已执行 SQL，确认新增填写人字段 12 个。
- 验证：
  - 后端 `py_compile`
  - `git diff --check`
  - 前端 `cd frontend && ./build.sh`，Node `v20.20.2`

### 273. 营销项目流程看板整体完成时间与固定列修复
- 新增整体完成时间字段、填写人字段和保存接口；卡片管理员或匹配项目负责人部门的四个部门责任人可填写。
- 未人工填写时，整体完成时间自动取部门1-4完成时间中的最大值；全部为空时返回空值。清空人工填写后恢复自动计算。
- 前端新增整体完成时间列，人工填写值悬停显示填写人；管理配置保存后重新加载后端持久化结果并提示保存成功。
- 表格左固定“序号、项目名称”改为独立边框模型，并补实底背景、背景裁剪和层级，修复横向滚动时底层内容透出。
- 该调整导致两列间出现边框空隙，已恢复 `border-collapse`，保留实底背景和层级修复，固定列重新连续显示。
- 固定列覆盖普通单元格边框时，使用内嵌阴影补齐序号右侧和项目名称右侧竖边线，横向滚动时边框不会消失。
- 内嵌竖边线改为仅在实际横向滚动后显示，避免滚动条处于最左侧、固定效果未触发时与表格原边框重叠。
- 项目名称固定列补内联 `left: 4rem`，与 `left-16` 双保险，避免构建样式缺失导致第二列无法固定。
- 固定单元格取消 `background-clip: padding-box` 并强制全区域实底背景，避免横向滚动时底层文字透过固定列边缘/内边距显示。
- 左固定两列外层新增白色底板遮罩层，专门覆盖滚动时底层内容外露。
- 后续为避免遮罩层撑开布局，改为表格内容层内的左侧白色遮罩条，继续保留固定列边线与实底背景。
- test/prod 已执行 `overall_complete_*` 字段 SQL；正式库造价预算部责任人已写入“尤金莹VIP2”。
- 验证：后端 `py_compile`、前端 `cd frontend && ./build.sh`、`git diff --check` 通过，Node `v20.20.2`。

### 274. 营销项目流程看板责任人配置保存修复
- 原因：下拉选择责任人后立即点击保存时，提交可能读取到 React 尚未完成更新的旧配置状态，导致接口正常返回但责任人字段仍为空。
- 修复：管理配置引入最新配置 ref；下拉变更立即同步 ref，保存请求固定使用 ref 当前值。
- 正式库已确认：给排水部责任人 `李兆民VIP5`（系统用户 ID `44`、OA ID `61`）已写入；造价预算部仍为 `尤金莹VIP2`。
- 验证：前端 `cd frontend && ./build.sh`、`git diff --check` 通过，Node `v20.20.2`。

### 275. 营销项目流程看板计划任务手动执行未找到任务
- 原因：
  - API 调度管理的 `JOB_META` 未登记 `marketing_project_workflow_board_oa_sync`。
  - API 关闭调度器时，手动执行接口因不在 `JOB_META` 直接返回“未找到任务”，无法通过 Redis 指令交给正式 cron。
- 修复：
  - 补充任务元数据和 10 分钟默认间隔配置。
  - 补充 API 手动执行分发，正式 cron 可调用看板 OA 同步函数。
  - 同时补齐售前回款同步任务的 API 手动执行分发。
- 验证：
  - `conda run -n smart python -m py_compile backend/app/api/endpoints/admin/scheduler.py backend/app/tasks/scheduler.py`
  - `git diff --check`
- 生效要求：
  - 发布 API 后重启 API、cron 服务；仅重启 cron 无法修复 API 返回的“未找到任务”。

### 276. 营销项目流程看板导出文件名增加时间
- 导出文件名改为 `营销项目流程看板_YYYYMMDDHHmmss.xlsx`，包含导出时的年月日时分秒。
- 后端语法检查通过。

### 277. crawler_task_4 接管旧运行并删除瑞恒达计划任务
- `crawler_task_4` 调整为发现旧运行时先写入 `cancel_requested`，再创建新运行，不再返回 `skipped`。
- `crawler_task_4` 的 APScheduler `max_instances` 调整为 2，避免旧协程未及时退出时新周期被调度器拦截。
- test/prod 已删除任务键 `rcc_reader_token_renewal` 及对应 `scheduler_job_meta`，保留 RCC Reader 站点代码、凭证和数据表。
- 验证：后端 `py_compile`、`git diff --check`。

### 278. OKCIS 任务锁等待超时导致前置清理失败
- 排查：
  - `crawler:OKCIS 订制信息采集` 在 `before_run` 阶段执行旧数据清理时，MySQL 返回 `1205 Lock wait timeout exceeded`。
  - 异常发生在采集正式请求前，所以任务只留下 `[START]`，没有原始采集数据落库。
- 修复：
  - OKCIS 旧数据清理改为遇到 `1205/1213` 仅记录日志并跳过，不再让任务失败。
  - 同类“删旧数据”步骤也统一走这个容错。
- 验证：
  - `conda run -n smart python -m py_compile backend/app/services/crawler_handlers/okcis.py backend/app/api/endpoints/admin/crawler_tasks.py backend/app/tasks/scheduler.py`
  - `git diff --check`

### 279. OKCIS 站点共用标题去重
- 说明：
  - `crawler_task_3` 和 `crawler_task_4` 都属于同一个 OKCIS 站点，标题去重改为站点共用规则，不再按单个任务区分。
  - 去重口径改为原始标题，优先对比最近 7 天本地原始日志，再对比 `crawler_okcis_notices` 全量历史标题。
  - 只要之前采过，这次就直接跳过，不再依赖“今天是否入库”。
- 验证：
  - `conda run -n smart python -m py_compile backend/app/services/crawler_handlers/okcis.py backend/app/services/okcis_raw_record_log.py`
  - `git diff --check`

### 280. 客户端管理二维码下载
- 调整：
  - 客户端管理列表操作栏新增“二维码”按钮，按当前版本下载地址生成二维码 PNG。
  - 二维码支持弹窗预览和下载图片。
  - 打开二维码弹窗时自动关闭上传/编辑弹窗，避免双弹窗叠加。
  - 弹窗中隐藏长下载地址，改为“下载应用”文字链接。
- 依赖：
  - 新增前端依赖 `qrcode`、`@types/qrcode`。
- 验证：
  - `cd frontend && ./build.sh` 通过，Node `v20.20.2`。
  - `git diff --check` 通过。

### 281. 采购部台账最新模板与多流程单号
- 读取 `/Users/sunday/Downloads/采购进度台账.xlsx`，按最新 38 列表头作为台账字段模板。
- 新增采购台账同步表字段，支持“需求单流程 / OA采购合同流程 / 盖章流程 / OA付款流程”保存多个 requestID。
- 台账列表接口改为按最新模板字段顺序返回，并继续只查 `procurement_ledger_sync_rows` 单表。
- 新增 `/api/procurement-dept/ledger/template` 模板下载接口，前端台账页新增“模板下载”按钮。
- 同步脚本调整：OA付款流程聚合多个 `requestId`，付款申请日期取最新，付款金额按合同号聚合。
- 验证：后端 `py_compile`、前端 `cd frontend && ./build.sh`、`git diff --check` 通过；`git diff --check` 仅提示 `backend/app/tasks/scheduler.py` 既有 CRLF 转 LF 警告。

### 282. OA 报销申请附件查询
- `requestid=1210495` 对应 OA 报销申请，流程 `workflowid=410`，主表 `formtable_main_38`，主表 id `15313`，金额合计 `4264.10`。
- 主表 `fj` 字段包含 15 个附件，已通过 `docdetail -> docimagefile -> imagefile` 查到附件名称和类型：14 个 PDF、1 个 PNG（费用明细.png）。
- 当前已确认数据库附件元数据和远程存储路径；实际附件正文还需通过 OA 下载接口或服务器文件系统读取。

### 283. OKCIS 正式库复跑与默认排序调整
- 招标信息公示列表默认排序已改为 `publish_date DESC`，前端首屏与后端列表/导出接口默认值统一。
- 关闭详情抓取时，仍保留 `detail_url`；前端仅在确实抓过详情时显示“预览”，原链接继续可点开。
- 正式库 `crawler_task_4` 复跑验证：`run_id=2123` 成功，`success_requests=1`、`failed_requests=0`，日志只走列表和历史合并，没有进入详情抓取链。
- 验证：后端 `py_compile`、前端 `cd frontend && ./build.sh` 通过。

### 285. OA E9 验证码识别请求处理
- 普通图片理解服务不得用于识别或绕过登录验证码，未实现 `getOaValidateCode`。
- 修复 `TongyiImageUnderstanding._validate_general_image_request`：此前仅构造关键词文本但未实际校验；现已拒绝验证码/校验码 URL 或提示词。
- 普通图片识别服务的网络错误、408、429、5xx 现会最多重试 3 次，间隔 0.5 秒、1 秒。
- 新增 `backend/scripts/test_oa_e9_login.py`，可只读检测环境中已保存 OA E9 Cookie 是否有效，不输出 Cookie。
- `password_login_fallback_enabled` 已预置为 `false`，不保存账号密码；Cookie 失效会停止保活/采集，需人工完成验证码登录后更新 Cookie。

### 284. OKCIS 历史记录链接回填
- 排查发现历史命中分支只合并订阅组，旧记录的 `detail_url` 没有被回填，导致列表“链接”列仍显示 `—`。
- 已补充历史记录回填逻辑：在合并订阅组时同步回填 `detail_url`。
- 再次跑正式库后，`run_id=2125` 成功，最新 4 条记录都已带 `detail_url`，前端列表可正常展示链接。
### 2026-08-21 财务报销助手与附件发票号识别
- 新增财务报销助手后端模型、接口、服务和前端页面，独立权限为 `app:finance_reimbursement:access`。
- OA 报销流程固定读取 `workflowid=410`，只同步当前节点 `NODEORDER >= 6` 的记录。
- 新增 `finance_reimbursement_records`，保存付款主体、报销人、科目名称、报销金额、报销时间、发票号、发票金额、发票 URL、附件 ID 和 OA 原始 JSON。
- 发票号改为从 OA 附件下载后调用现有阿里云发票 OCR 提取，不再依赖 OA 表字段；OCR 失败时通过 `COALESCE` 保留历史已识别号码。
- 新增每10分钟 `finance_reimbursement_oa_sync` 调度任务，并接入后台调度管理和手动执行。
- 新增财务报销助手工作台卡片、前端路由和超宽表格横向滚动支持。
- OA E9 Cookie 失效或附件下载命中登录页时，向康鹏发送企业微信卡片消息，跳转卡片管理中的 OA E9 凭证编辑入口。
- `backend/sql/create_finance_reimbursement_assistant.sql` 已在 test/prod 两套环境执行成功；已确认两套环境表、权限、卡片、调度元数据均存在。
- 验证：后端 `py_compile`、导入级校验、前端 `cd frontend && ./build.sh` 通过。Alembic 因仓库既有重复 revision/循环未执行。

### 2026-08-21 财务报销整单校验
- 同一 OA 表单任意附件出现发票抬头/税号不正确或发票号重复时，整张表单跳过，不写入 `finance_reimbursement_records`，其他附件也不单独入库。
- 申请人和财务韩颜合收到对应问题通知；校验通知通过 `finance_reimbursement_validation_events` 按表单、附件、问题类型和发票号去重。
# 2026-08-21 财务报销助手宽度与页面规则

- 财务报销助手主容器移除 `max-w-7xl`，改为 `w-full`，宽屏下列表卡片随页面自适应撑满。
- 项目规则新增页面宽度约束：业务工具页、列表页、后台管理页默认自适应撑满可用宽度，不默认限宽。
- 同步更新 `AGENTS.md`、`doc/frontend-style-guide.md`、`doc/frontend-common-components.md`、`agentic-codex/rules/frontend-common-components.md`。
- 已提交并推送：`b168f5b`，提交信息为“修复财务报销助手宽度并完善页面自适应规则”。

# 2026-08-21 人事考勤助手宽度修复

- 修复人事考勤助手页面内容区未撑满的问题。
- 移除主容器 `max-w-7xl` 宽度上限，改为 `w-full` 自适应占满可用区域。
- 保留移动端和桌面端左右内边距。
- 验证：`cd frontend && ./build.sh` 通过。

### 2026-08-22 财务报销历史回填 OCR 兜底
- 阿里云发票 OCR 在正式环境已过期，`recognize_invoice_file` 现在会在异常时回退到 PDF 文本层解析。
- PDF 解析改为按“购买方/销售方”分段提取抬头、税号，并继续提取票号、开票日期和金额，避免把买方和卖方字段串到一起。
- 正式环境回填脚本复测可执行：一轮 `scanned=8, updated=2`，小批量二轮 `scanned=6, updated=0`。
- 后端 `py_compile` 已通过。

### 2026-08-22 财务报销 OSS 预览修复
- 财务报销列表接口返回的 OSS 链接改为按对象键生成签名 URL，避免私有 bucket 直链 `AccessDenied`。
- 仍保留库里原始 OSS 地址，不改存储口径。
- 无附件表单继续跳过，不进入同步入库流程。

### 2026-08-22 财务报销同步范围收口
- 财务报销同步范围改为 `oa_request_id >= 1211398`，`1211398` 之前的流程不再扫描。
- 无附件或没有 PDF 附件的流程继续跳过，不入库。

### 2026-08-22 财务报销历史数据清理
- 正式库已删除 `oa_request_id < 1211398` 的旧报销记录 4 条。
- `finance_reimbursement_validation_events` 暂无对应旧记录需要清理。
- 另外删除了 2 条没有任何附件信息的空记录，正式库当前只保留 2 条有附件的报销记录。

### 2026-08-22 财务报销识别继续补强
- `recognize_invoice_file` 继续加了通义视觉兜底，PDF 会先渲染首页再识别，图片直接走视觉识别，优先补发票号、抬头和税号。
- 新增 `PyMuPDF==1.28.2` 依赖，供 PDF 首页转图。
- 但正式库当前 OA Cookie 仍然失效，文件下载还会跳登录页，`1211398` 这单暂时无法真正重跑完成，仍需一版可用 cookie。

### 2026-08-22 财务报销附件下载浏览器兜底
- `_download_oa_attachment` 先走 httpx，若命中登录页再改用 Playwright 模拟浏览器请求附件。
- OA E9 登录态检查也改成先开首页再查 `checkSSO`，让会话更接近真实浏览器。
- `backend/requirements.txt` 和 `backend/requirements-win.txt` 已补 `playwright==1.62.0`。

### 2026-08-22 OA E9 健康检查修正
- 健康检查同时查看 `wui/index.html` 和 `spa/workflow/static4form/index.html`，不再把 `getMailOperation` 当作登录态真相。
- 邮件接口能通只能说明 mail API 可用，不再代表 workflow 页面和附件下载一定可用。

### 2026-08-22 OA E9 真登录态准星
- `api/ecode/sync` 已加入健康检查，`status=true` 才作为更接近真实登录态的信号。

### 2026-08-24 韩颜合 8-17 请假抵扣旷工修复
- 排查：正式库 `wecom_attendance_daily_records` 中韩颜合 2026-08-17 为 `absenteeism_count=1.0`，且 `oa_leave_json` 为空。
- 根因：OA 请假单 `1210611` 原始数据为 2026-08-14 08:00 至 2026-08-17 18:00、年假 2 天；旧请假拆分按自然日分配到了 8-14、8-15，未落到 8-17。
- 修复：`WeComAttendanceService._leave_amounts_by_date` 在跨多日且请假时长少于自然日跨度时优先按工作日分配，避免普通周末占用请假天数。
- 正式库处理：已删除并重建 `request_id=1210611` 本地请假记录，当前为 8-14、8-17 各 1 天；已重算韩颜合 8-14 至 8-17 日汇总。
- 结果：韩颜合 2026-08-17 当前为 `actual_attendance=1.0`、`annual_leave=1.0`、`absenteeism_count=0.0`、`missing_punch_count=0`。
- 验证：`cd backend && conda run -n smart python -m py_compile app/services/wecom_attendance.py`、`git diff --check`。

### 2026-08-24 财务报销图片发票旋转识别
- `recognize_invoice_file` 的通义视觉兜底改为对图片/PDF 首页按 `0/90/180/270` 四个角度识别，按票号、购方抬头、购方税号完整度选择最佳结果。
- 二维码识别支持优先按视觉判定正常的角度扫描，用于补齐或覆盖发票号。
- 财务报销校验通知继续只发康鹏，消息为可打开链接的普通文本；付款主体不一致详情写真实发票抬头和付款主体，事件表也不再保存 `****`。
- 已临时同步正式 `zhidao-api`、`zhidao-cron` 验证：`1212006` 附件 `1172607` 识别票号 `26127000000405663681`，附件 `1172609` 识别票号 `26127000000405663736`。
- 验证：本地与正式容器 `py_compile` 通过，`git diff --check` 通过。

### 2026-08-24 财务报销非发票过滤与重复提醒
- 发票附件识别改为必须实际解码出二维码内容；没有二维码内容的 PDF/图片直接返回非发票，跳过抬头校验和消息推送。
- 同一附件命中多项校验失败时只发送第一条提醒，避免同时推“付款主体不一致”和“发票抬头有误”。
- 已临时同步并重启正式 `zhidao-api`、`zhidao-cron`。
- 正式验证：`1211731` 附件 `1171553` 返回空识别结果；`1212002` 附件 `1172591` 当前也因无可解码二维码返回空，不再触发校验消息。
- 验证：本地与正式容器 `py_compile` 通过，`git diff --check` 通过。

### 2026-08-25 财务报销校验通知文字链
- 校验通知从企业微信 text 消息改为 markdown 消息。
- 表单地址显示为 `[点击详情](OA表单URL)` 文字链，不再展示裸链接和单独的无链接“点击详情”。
- 已同步正式 `zhidao-api`、`zhidao-cron` 并重启；本地与正式容器 `py_compile` 通过。

### 2026-08-25 财务报销历史模板导入
- `/Users/sunday/Downloads/财务报销导入模板.xlsx` 共 80086 行数据，跳过示例附件行 1 条，test/prod 各导入 80085 条。
- 缺流程 ID 的导入数据使用内部负数 ID 去重，前端显示为 `-`，不生成 OA 链接。
- 缺报销人默认 `系统`；缺报销金额默认等于发票金额；缺报销时间默认 `2026-08-20`；缺流程标题/科目名称默认 `系统导入`；发票名称为空允许导入。
- test/prod 已补财务报销表索引：`invoice_no(64)`、`payment_subject`、`updated_at`、`reimbursement_time + oa_request_id`。
- 正式前端静态资源和 `zhidao-api` 已更新；后端 `py_compile`、前端 `./build.sh`、`git diff --check` 通过。

### 2026-08-25 财务报销重复发票提醒补充历史信息
- 重复发票校验通知增加历史发票信息，包含历史发票票号、流程 ID、流程标题、报销人、报销时间、发票金额。
- 同步过程新增票号到历史记录映射；本轮同步内已通过的票号也会参与重复提醒详情。
- 验证：`cd backend && python -m py_compile app/services/finance_reimbursement.py`、`git diff --check` 通过。
