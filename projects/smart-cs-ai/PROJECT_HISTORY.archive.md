# 项目历史记录（持续维护）

用于记录近期关键改动、部署注意事项与排查结论，便于下次进入项目快速恢复上下文。

## 最近重点改动

### 0.86 新增 okcis 采集站点目录骨架
- 已新增：
  - `backend/crawler_sites/okcis/credentials.json`
  - `backend/crawler_sites/okcis/request.json`
  - `backend/crawler_sites/okcis/formatter.py`
- 当前状态：
  - 已完成 `okcis` 站点基础目录初始化
  - 后续可继续补自动登录 curl、请求模板、结果格式化规则

### 0.87 okcis 请求模板已支持 dzid 列表与当天日期变量
- 已调整：
  - `backend/crawler_sites/okcis/request.json`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - `okcis` 模板已写入目标接口地址和基础请求头
  - `dzid` 改为变量位 `{{dzid}}`
  - `search-start-time-input`、`search-end-time-input` 改为 `{{today}}`
  - 模板内已保存 8 个 `dzid`
  - 运行时会自动把 `{{today}}` 替换成当天日期，`{{dzid}}` 默认取任务覆盖值，没有覆盖值时取列表第一个
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/crawler_sites/okcis/formatter.py` 通过

### 0.88 okcis 手动采集任务已同步到测试与正式环境
- 已新增：
  - `backend/sql/upsert_okcis_manual_task.sql`
- 当前规则：
  - 新增任务 `okcis_notice_manual`
  - 任务名称：`OKCIS 订制信息采集`
  - 站点：`okcis`
  - 模板：`request.json`
  - 执行方式：`manual`
  - 同步模式：`full`
  - 每页 50 条，起止页默认 1
- 已执行同步：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/upsert_okcis_manual_task.sql`
  - `test`：`[OK] 已执行 2 条 SQL`
  - `prod`：`[OK] 已执行 2 条 SQL`

### 0.89 okcis 已补基础格式化，避免第一页因结构不符直接停止
- 已调整：
  - `backend/crawler_sites/okcis/formatter.py`
- 当前规则：
  - `okcis` 返回结果会统一转成 `{ items: [...] }` 结构
  - 会自动尝试从 `items/list/rows/data/result/records` 中提取列表
  - 暂时先保留 `raw`、`raw_text` 方便继续看真实返回结构
- 本次验证：
  - `python -m py_compile backend/crawler_sites/okcis/formatter.py` 通过

### 0.90 okcis 已支持读取 cur_dingzhizu_name、list 并按 link 补抓详情
- 已调整：
  - `backend/crawler_sites/okcis/formatter.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - `okcis` 结果优先读取 `cur_dingzhizu_name` 与 `list`
  - `list.link` 会自动拼成 `https://www.okcis.cn/...` 详情地址
  - 执行时会继续抓详情页
  - 日志会输出 `cur_dingzhizu_name` 和 `list`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/crawler_sites/okcis/formatter.py` 通过

### 0.91 okcis 详情页已提取截止时间类字段
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 详情页会额外提取：
    - `截止时间`
    - `报名截止时间`
    - `获取招标文件截止时间`
    - `投标截止时间`
    - `开标时间`
  - 提取结果挂到每条数据的 `deadline_fields`
  - 日志里会输出截止时间提取结果
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.92 okcis 列表接口已改为只解析前段 JSON 并同步最新请求参数
- 已调整：
  - `backend/crawler_sites/okcis/request.json`
  - `backend/crawler_sites/okcis/formatter.py`
- 当前规则：
  - 当返回内容是 `JSON + HTML` 拼接时，仅截取前面的 JSON 对象解析
  - 后面的 HTML 不再参与列表解析
  - `okcis` 请求模板已同步最新 `curl` 参数：
    - `result-class-type-form=brn`
    - `result-infotype-form=bn`
    - `searchUniseq=f38d4f778e8e5a7a25183e856d056641`
- 本次验证：
  - `python -m py_compile backend/crawler_sites/okcis/formatter.py` 通过

### 0.93 okcis 执行日志已增加原始返回内容
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - `okcis` 任务执行时会额外输出 `[RAW] page=x ...`
  - 该日志直接打印接口原始返回结果，便于对照排查，不经过格式化
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.94 okcis 请求已改为按原始 multipart body 发起
- 已调整：
  - `backend/crawler_sites/okcis/request.json`
- 当前规则：
  - 不再用普通 `form_body` 方式发送
  - 改为按用户提供的原始 `curl --data-raw` multipart 内容发送
  - 已保留 `{{dzid}}`、`{{today}}` 变量替换
- 结论：
  - 之前结果不一致的原因，是请求体构造方式和原始 `curl` 不一致

### 0.95 okcis 最新 cookie 与请求参数已同步到站点配置
- 已调整：
  - `backend/crawler_sites/okcis/credentials.json`
  - `backend/crawler_sites/okcis/request.json`
- 当前规则：
  - 已写入最新 `cookie`
  - 已写入最新 `ticket`
  - 已写入最新 `boundary`
  - 已写入最新 `searchUniseq`
  - 当前 `okcis` 任务已不再使用空凭证
- 本次验证：
  - `python -m py_compile backend/crawler_sites/okcis/formatter.py backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.96 okcis 截止时间提取增强为优先匹配原始 HTML
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 提取截止时间时，先直接从详情页原始 HTML 匹配
  - 再回退到纯文本逐行识别
  - 更适合 `截止时间`、`报名截止时间` 这类表格字段场景
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.97 okcis 截止时间提取已修正空单元格误识别
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 截止时间类字段优先按详情页表格的相邻 `td/th` 配对提取
  - 当值单元格为空时不再跨行误取下一字段
  - 当前样本里 `截止时间` 为空时，`deadline_fields` 正确返回空对象
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过
  - 使用用户提供详情样本独立验证，提取结果为 `{}`

### 0.98 okcis 已接入专表写入与截止时间过滤规则
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
  - `backend/sql/create_okcis_notice_table.sql`
  - `backend/migrations/versions/z9b8c7d6e5f4_add_okcis_notice_table.py`
- 当前规则：
  - 新增 `crawler_okcis_notices` 专表，按 `uniseq` 唯一更新
  - 仅当截止时间能解析为标准时间时才允许入库
  - 截止时间小于等于“当前日期 + 4 天”的数据直接忽略
  - 每次任务执行前会先清理库里已进入未来 4 天内的旧数据
  - 第二天再次执行同一任务时，命中的 `uniseq` 会自动更新刷新
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/migrations/versions/z9b8c7d6e5f4_add_okcis_notice_table.py` 通过

### 0.99 okcis 专表已同步到测试与正式环境
- 已执行：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/create_okcis_notice_table.sql`
- 执行结果：
  - `test`：`[OK] 已执行 1 条 SQL`
  - `prod`：`[OK] 已执行 1 条 SQL`

### 1.00 okcis 手动执行已支持自动遍历 dzid 列表
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - `okcis` 手动任务会自动遍历 `request.json` 里的 `variables.dzid_list`
  - 无需手工改单个 `dzid`
  - 执行日志会额外带上 `dzid=...`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 1.01 okcis 截止日期已兼容冒号前缀并补零点
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - `：2026-07-22`、`截止时间：2026-07-22` 这类值可正常解析
  - 只有日期没有时分秒时，统一按 `00:00:00` 入库
  - 带时分的数据按原时间入库，不再误变整点
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过
  - 样例校验通过：`2026-07-22 -> 2026-07-22 00:00:00`、`2026-07-22 13:05 -> 2026-07-22 13:05:00`

### 1.02 okcis 已补登录接口 curl 到站点凭证配置
- 已调整：
  - `backend/crawler_sites/okcis/credentials.json`
- 当前规则：
  - `okcis` 自动刷新凭证将优先使用新的 `/signed/` 登录接口 curl
  - 后续检测到登录失效时可直接走该登录配置尝试刷新 cookie

### 1.03 okcis 已补登录状态检测 curl 并接入自动登录判断
- 已调整：
  - `backend/crawler_sites/okcis/credentials.json`
  - `backend/app/services/crawler.py`
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 请求前会先调用 `check_login_state.php`
  - 如果返回 `nologin`，会先走 `/signed/` 登录接口刷新凭证
  - 刷新成功后再继续执行采集
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/app/services/crawler.py` 通过

### 1.04 okcis 自动登录已接入验证码抓取与算式识别
- 已调整：
  - `backend/app/services/crawler.py`
- 当前规则：
  - 访问 `https://www.okcis.cn/login/` 后自动抓取验证码图片
  - 验证码地址按 `code.php?refresh=false&{urlencode时间串}` 拼接
  - 自动识别算式验证码并计算 `yzm`
  - `/signed/` 登录时会动态替换原来的固定验证码
- 本次验证：
  - `python -m py_compile backend/app/services/crawler.py backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 1.05 补充 Pillow 依赖声明
- 已调整：
  - `backend/requirements.txt`
  - `backend/requirements-win.txt`
- 当前规则：
  - `okcis` 验证码识别依赖 `Pillow`
  - 已补到依赖清单，避免新环境缺包

### 1.06 爬虫任务控制台日志改为实时输出
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 爬虫任务执行过程中写入 `log_lines` 时会同步实时输出到控制台
  - 不再等任务结束后一次性看到日志
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 1.07 演示模式实时日志补充 print 输出
- 已调整：
  - `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则：
  - 演示模式下日志除 `logger.info` 外，会额外直接 `print(..., flush=True)` 到控制台
  - 避免被日志级别拦掉导致看不到实时输出
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 1.08 okcis 验证码地址改为从登录页解析真实 src
- 已调整：
  - `backend/app/services/crawler.py`
- 当前规则：
  - 自动登录时先访问登录页
  - 从页面 `#setcode` 节点读取真实验证码图片地址
  - 仅在页面未给出地址时才回退到拼接默认地址
- 本次验证：
  - `python -m py_compile backend/app/services/crawler.py` 通过

### 1.09 okcis 验证码地址提取增强为正则优先并输出来源
- 已调整：
  - `backend/app/services/crawler.py`
- 当前规则：
  - 验证码地址优先用正则从登录页原始 HTML 提取 `setcode` 的 `src`
  - 抓不到再回退 DOM
  - 登录日志会输出 `captcha_src_source`
- 本次验证：
  - `python -m py_compile backend/app/services/crawler.py` 通过

### 1.10 okcis 验证码地址兼容 setcode4 并输出明确失败原因
- 已调整：
  - `backend/app/services/crawler.py`
- 当前规则：
  - 验证码提取会额外兼容 `#setcode4` 和任意 `code.php` 图片地址
  - 验证码获取失败时会直接输出 `status/source/url`
- 本次验证：
  - `python -m py_compile backend/app/services/crawler.py` 通过

### 0.85 售后部客户无忧拆为独立页面与独立路由
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
  - `frontend/src/pages/after_sales_customer_follow/index.tsx`
  - `frontend/src/App.tsx`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/AppCenterPage.tsx`
- 当前规则：
  - `售后部（客户无忧）` 不再复用售后部助手 tab
  - 改为独立页面、独立路由 `/apps/after-sales-customer-follow`
  - 权限仍为 `app:after_sales_dept:admin`
- 本次验证：
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.84 首页新增售后部-无忧客户卡片并限制管理员可见
- 已调整：
  - `frontend/src/constants/appPermissions.ts`
  - `frontend/src/components/dashboard/appCatalog.ts`
  - `frontend/src/pages/AppCenterPage.tsx`
- 当前规则：
  - 首页新增 `售后部-无忧客户` 卡片
  - 入口直达 `/apps/after-sales-dept?tab=customer_follow`
  - 仅拥有 `app:after_sales_dept:admin` 权限的用户可见
- 本次验证：
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.83 售后部客户跟进新建时间组件优化并补客户名称清空
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 新建跟进时间改为弹层日历 + 时间输入组合
  - 客户名称可检索下拉新增清空当前选择
- 本次验证：
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.82 售后部客户跟进新建客户名称改为可检索下拉校验
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 新建跟进记录里的客户名称改为可检索下拉选择
  - 候选数据来源于 `crawler_kehu51_customer_list`
  - 下拉交互样式沿用当前助手列表筛选风格
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.81 backend README 补充生产环境依赖安装命令
- 已调整：
  - `backend/README.md`
- 当前规则：
  - README 已补充生产环境使用 conda 环境安装依赖命令
  - 使用清华源：`/anaconda3/envs/smart/bin/python -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple`

### 0.80 计划任务执行记录改为落数据库并优先从库读取
- 已调整：
  - `backend/app/tasks/scheduler.py`
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `backend/app/models/crawler_task.py`
  - `backend/app/models/__init__.py`
  - `backend/migrations/versions/a8b9c0d1e2f3_add_scheduler_job_runs_table.py`
- 当前规则：
  - 系统计划任务执行后会写入 `scheduler_job_runs`
  - 手动执行记录为 `manual`
  - 定时执行记录为 `cron`
  - 计划任务管理页优先读取数据库执行记录，不再依赖进程内存
- 本次验证：
  - `python -m py_compile backend/app/tasks/scheduler.py backend/app/api/endpoints/admin/scheduler.py backend/app/models/crawler_task.py backend/app/models/__init__.py backend/migrations/versions/a8b9c0d1e2f3_add_scheduler_job_runs_table.py` 通过

### 0.79 计划任务执行记录表已同步到测试与正式环境
- 已调整：
  - `backend/app/models/crawler_task.py`
  - `backend/app/models/__init__.py`
  - `backend/migrations/versions/a8b9c0d1e2f3_add_scheduler_job_runs_table.py`
- 已同步 SQL：
  - `scheduler_job_runs` 表已执行到 `test` 与 `prod`
- 当前规则：
  - 后续计划任务执行记录将支持持久化落库
  - 本次建表 SQL 已双环境执行成功
- 本次执行结果：
  - `test`：`[OK] 已执行 1 条 SQL`
  - `prod`：`[OK] 已执行 1 条 SQL`

### 0.78 爬虫定时任务接入 APScheduler 自动注册
- 已调整：
  - `backend/app/tasks/scheduler.py`
- 当前规则：
  - 后台配置的爬虫任务会按 `cron_expr` 自动注册到 APScheduler
  - 调度器每分钟自动同步一次数据库里的爬虫任务配置
  - `KNOWLEDGE_SYNC_INTERVAL_MINUTES <= 0` 时，不再误伤整个调度器
- 本次验证：
  - `python -m py_compile backend/app/tasks/scheduler.py backend/app/api/endpoints/admin/crawler_tasks.py backend/app/main.py` 通过

### 0.77 售后部客户跟进导出模板迁移到 backend/static
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/static/客户无忧新客户跟进表.xls`
- 当前规则：
  - 客户跟进导出模板不再依赖仓库根目录资料文件夹
  - 改为统一读取 `backend/static/客户无忧新客户跟进表.xls`
  - 更适合容器镜像直接打包，避免漏挂载
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过

### 0.76 售后部客户跟进导出失败增加控制台强制打印
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 导出接口报错时，除了 logger 外，还会直接 `print` 参数并输出完整 traceback
  - 即使容器日志配置不打印 `logger.exception`，控制台也能看到详细错误
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过

### 0.75 售后部客户跟进导出失败补充完整异常日志
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 导出接口异常时，会在控制台输出完整堆栈
  - 日志会附带筛选参数和模板路径，方便直接定位
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过

### 0.74 售后部客户跟进导出补齐 xls 模板转换依赖
- 已调整：
  - `backend/requirements.txt`
  - `backend/requirements-win.txt`
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 客户跟进导出仍走现有 `.xls` 模板转换逻辑
  - 已补齐 `xlrd`、`xls2xlsx` 依赖，避免环境缺包导致导出接口 500
  - 未使用的 `xlutils` import 已移除
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过

### 0.73 售后部客户跟进新建提示改为按钮悬浮说明
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 页面内单独的 `手动新增` 提示块已移除
  - 改为鼠标悬浮在 `新建` 按钮上显示说明文字
- 本次验证：
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.72 售后部客户跟进导出按钮移入预览头部并改为系统级全屏
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 列表页外层 `导出全部` 按钮已移除
  - 下载按钮已移动到导出预览层头部，位于全屏按钮左侧
  - 预览层 `全屏` 已改为浏览器 Fullscreen API，全屏时铺满整个屏幕
- 本次验证：
  - `cd frontend && source ~/.nvm/nvm.sh && npm run build` 通过

### 0.71 售后部客户跟进管理新增手动新建入口
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 新增能力：
  - 客户跟进管理列表页新增 `新建` 按钮
  - 新增手动新建跟进记录弹窗
  - 新增后端接口：`POST /after-sales-dept/customer-follow-rows`
- 页面提示：
  - 鼠标移动到说明图标时，会提示“客户无忧的跟进记录会自动同步，手动新建的记录不会被自动同步数据覆盖”
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py backend/app/schemas/after_sales_dept.py` 通过
  - `cd frontend && npm run build` 通过

### 0.70 售后部客户跟进导出预览改为自定义固定层
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 导出预览不再使用 `Dialog` 组件
  - 改为自定义固定定位预览层
  - 预览层内单独控制关闭、全屏、sheet 切换、横向滚动
- 本次验证：
  - `cd frontend && npm run build` 通过

### 0.69 售后部客户跟进导出预览新增全屏并限制结果列宽
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - 预览弹窗新增 `全屏 / 退出全屏`
  - 预览 HTML 会按列识别日期列和结果列
  - 跟进结果列超过 `1000px` 后自动换行
  - 预览区继续保留独立横向滚动容器
- 本次验证：
  - `cd frontend && npm run build` 通过

### 0.68 售后部客户跟进列表新增 Excel 导出预览
- 已调整：
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 当前规则：
  - `客户跟进管理` 列表页新增 `导出预览` 按钮
  - 点击后会直接调用现有导出接口，前端读取生成的 Excel
  - 支持按 sheet 切换在线预览
  - 预览方式改为 `sheet_to_html`，展示更接近 Excel，合并单元格表现更完整
- 本次验证：
  - `cd frontend && npm run build` 通过

### 0.67 售后部客户跟进管理新增编辑跟进结果
- 已调整：
  - `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
  - `backend/app/schemas/after_sales_dept.py`
  - `frontend/src/pages/after_sales_dept/index.tsx`
- 新增后端接口：
  - `PUT /after-sales-dept/customer-follow-rows/{row_id}`
- 当前规则：
  - 客户跟进管理列表新增“操作”列
  - 每条记录可打开弹窗编辑 `follow_result`
  - 保存时后端会先做同样的 HTML 清洗，再写回数据库
  - 如果 `raw_json` 为可解析对象，会同步更新其中的“跟进结果”
- 关联说明：
  - 旧库 `follow_result` 中 `<font face=` 异常值此前已完成清空/修复处理
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py backend/app/schemas/after_sales_dept.py` 通过
  - `cd frontend && npm run build` 通过

### 0.66 kehu51 空跟进结果补抓脚本增加控制台日志
- 已调整 `backend/scripts/refetch_kehu51_empty_follow_results.py`
- 当前日志输出：
  - 启动时输出待补抓空记录数
  - 每页输出页码与格式化后的条数
  - 命中待补抓记录时，输出客户、联系人、跟进时间
  - 输出格式化后的 `follow_result`
  - 真正更新时输出 `row_id`
- 本次验证：
  - `python -m py_compile backend/scripts/refetch_kehu51_empty_follow_results.py` 通过
  - 已执行小范围预检，控制台日志正常输出

### 0.65 新增 kehu51 跟进结果空值专项补抓脚本
- 已新增脚本：
  - `backend/scripts/refetch_kehu51_empty_follow_results.py`
- 当前规则：
  - 只针对 `crawler_kehu51_follow_records.follow_result` 为空的数据
  - 仍然走现有 kehu51 爬取模板与格式化流程
  - 抓到新结果后，仅当数据库当前该条 `follow_result` 仍为空时才更新
  - 如果数据库该条后来已经有值，则跳过，不覆盖
- 已完成预检验证：
  - `python -m py_compile backend/scripts/refetch_kehu51_empty_follow_results.py` 通过
  - 预检命令：`/opt/anaconda3/envs/smart/bin/python backend/scripts/refetch_kehu51_empty_follow_results.py --env test --limit 20 --page-end 5`
  - 结果：扫描 5 页，命中更新 0 条，脚本可正常执行

### 0.64 kehu51 跟进结果旧库 `<font face=` 残留已清理到测试和正式
- 已调整 `backend/scripts/fix_kehu51_follow_results.py`
- 当前脚本规则：
  - 同时扫描 `<span style=` 与 `<font face=` 异常/HTML 跟进结果
  - 可恢复的转为纯文本
  - 无法恢复的直接清空
- 已执行双环境同步：
  - `/opt/anaconda3/envs/smart/bin/python backend/scripts/fix_kehu51_follow_results.py --both --apply --clear-unrecoverable`
- 本次结果：
  - `test`：命中 `2483` 条，恢复 `7` 条，清空 `2476` 条
  - `prod`：命中 `2483` 条，恢复 `7` 条，清空 `2476` 条

### 0.63 售后部客户跟进展示与导出补充清洗旧库 HTML 跟进结果
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 客户跟进管理列表读取数据库时，会先把 `follow_result` 中的 HTML 标签转成纯文本
  - 导出 Excel 时同样先清洗，避免 `<font face=` 之类旧脏值继续显示
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 样例 `<font face=...>已报价</font>` 清洗结果验证通过

### 0.62 售后部客户跟进导出空季度去掉额外占位列
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 季度之间不再额外插入空白占位列
  - 空季度现在只占 2 列单元格
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 已本地重新导出校验，第三季度空白时为两列

### 0.61 软件部任务工具 tab 切换改为始终同步 URL 参数
- 已调整 `frontend/src/pages/software_task/index.tsx`
- 当前规则：
  - 点击 tab 时会同步更新地址栏 `tab` 参数
  - `任务大厅` 现在也会显式带上 `?tab=tasks`
  - 页面无 `tab` 参数时，会自动补成默认 `?tab=tasks`
- 修复结果：
  - 从 `?tab=records` 进入后，再切换其他 tab 不会被拉回工作记录
- 本次验证：
  - `cd frontend && npm run build` 通过

### 0.60 售后部客户跟进导出修复空季度只保留两列并隐藏尾部残留列
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 本季度无客户时，季度区域仍保留 2 列单元格和边框
  - 最后有效季度之后的模板残留列统一隐藏，避免最后一个 sheet 继续露出空白边框列
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 已本地重新导出验证最后一个 sheet 尾部残留列已隐藏

### 0.59 售后部客户跟进导出增加换行行高自适应
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 跟进内容出现换行时，导出行高会按行数自动放大
  - 最大不超过 5 行高度
  - 空白行会恢复模板默认高度
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 已本地重新导出验证

### 0.58 售后部客户跟进导出修复最后一个 sheet 季度表头残留样式
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前规则：
  - 模板拆分合并单元格后，先清空旧季度标题和客户标题
  - 新扩展列复制样式时，不再把旧季度文案一起复制过去
- 修复结果：
  - 最后一个 sheet 的第三季度、第四季度样式和其他 sheet 保持一致
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 已本地重新导出校验最后一个 sheet 表头正常

### 0.57 kehu51 跟进结果清洗并入爬取规则
- 已调整 `backend/crawler_sites/kehu51/formatter.py`
- 当前规则：
  - 跟进结果在格式化阶段先做 HTML 清洗
  - 类似 `<span style=\\` 的异常脏值直接清空，不再写入数据库
  - 正常 HTML 内容会提取为纯文本后再入库
- 本次验证：
  - `python -m py_compile backend/crawler_sites/kehu51/formatter.py` 通过
  - 样例清洗验证通过

### 0.56 kehu51 客户名称清洗补充去除 ▼
- 已调整 `backend/crawler_sites/kehu51/formatter.py`
- 当前规则：
  - 客户名称清洗时会统一去掉末尾或中间的 `▼`
  - 同时保留原有方括号去除逻辑
- 示例：
  - `泰达鸿泰（王先生） ▼ -> 泰达鸿泰（王先生）`
- 本次验证：
  - `python -m py_compile backend/crawler_sites/kehu51/formatter.py` 通过

### 0.55 售后部客户跟进导出修正日期展示与同日合并
- 已调整 `backend/app/api/endpoints/after_sales_dept/after_sales_dept.py`
- 当前导出规则：
  - 跟进日期改为 `mm月dd日`
  - 同一天的多条跟进记录不再拼成一格
  - 改为保留多行跟进内容，并将左侧日期单元格纵向合并
  - 日期列宽已加宽，避免显示不全
- 当前导出文件名：
  - `售后部客户跟进表_YYmmddHis.xlsx`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - 已本地直接执行导出逻辑生成测试文件通过

### 0.54 新增 kehu51 跟进结果修复脚本
- 已新增脚本：
  - `backend/scripts/fix_kehu51_follow_results.py`
- 用途：
  - 扫描 `crawler_kehu51_follow_records`
  - 修复 `follow_result` / `raw_json.跟进结果` 中异常的 `<span style=\\` 脏值
- 预检查结果（limit=1000）：
  - 总计命中：`886`
  - 可恢复：`4`
  - 不可恢复：`882`
- 结论：
  - 大部分库内 `follow_result` 和 `raw_json.跟进结果` 已经同步被截断
  - 不能仅靠现有表内数据完整还原
  - 这类数据更适合清空后重抓

### 0.53 售后部助手新增客户跟进管理与导出全部
- 已在 `frontend/src/pages/after_sales_dept/index.tsx` 新增 tab：
  - `客户跟进管理`
- 已新增客户跟进列表：
  - 展示字段：跟进时间、客户名称、联系人、联系方式、跟进结果、跟进阶段、跟进人、录入时间、来源
  - 支持关键词查询
  - 支持分页
- 已新增导出全部按钮：
  - 导出为 `.xlsx`
  - 导出全部部门成员的客户跟进数据
- 已新增后端接口：
  - `GET /after-sales-dept/customer-follow-rows`
  - `GET /after-sales-dept/customer-follow-rows/export`
- 数据源：
  - `crawler_kehu51_follow_records`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/after_sales_dept/after_sales_dept.py` 通过
  - `cd frontend && npm run build` 通过

### 0.52 爬虫成功日志改为只输出编号和结果数
- 已调整 `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前成功日志输出规则：
  - 不再打印整段正常数据内容
  - 改为只输出：
    - `page`
    - `count`
- 当前异常日志仍保留详细内容，便于排错
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.51 爬虫详情页日志返回再次缩减
- 已调整 `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前详情页日志返回规则：
  - 只展示最后 `6000` 个字符
  - 不再展示前段大日志
- 当前效果：
  - `admin/crawler/:id` 页面加载更快
  - 更容易直接看到最近报错和最新执行结果
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.50 爬取任务新增先查库再处理规则
- 已调整 `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前执行爬取任务时：
  - 先查询数据库业务表中是否已存在该条数据
  - 已存在则直接忽略
  - 不再重复进入写库逻辑
  - 继续处理下一条数据
- 当前判重规则：
  - 跟进记录：`客户名称 + 联系人 + 跟进时间`
  - 客户列表：`客户名称 + 联系人 + 手机号 + 录入时间`
- 页面日志会显示：
  - `跟进记录已存在，跳过`
  - `客户列表已存在，跳过`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.49 爬虫写库改为单条跳过继续执行
- 已调整 `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前规则改为：
  - 如果数据库已存在该条数据，按原有 `ON DUPLICATE KEY` 正常跳过/更新，不中断任务
  - 如果单条数据因时间格式等问题写库失败，不再终止整页整任务
  - 直接记录该条为 `ROW-SKIP`，继续执行下一条
- 当前日志输出：
  - 页面执行日志会写入 `[ROW-SKIP]`
  - 后端控制台会输出对应 `warning`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.48 kehu51客户列表补充纯日期带星期格式解析
- 已根据最新报错补充时间格式：
  - `2024-6-1 (周六) -> 2024-06-01`
- 本次卡点字段：
  - `last_follow_time`
- 本次报错定位信息：
  - `task_key=kehu51_customer_list`
  - `page=218`
  - `item_index=12`
  - `客户名称=葛洲坝 ▼`
- 已更新：
  - `backend/crawler_sites/kehu51/formatter.py`
- 本次验证：
  - `python -m py_compile backend/crawler_sites/kehu51/formatter.py` 通过
  - 样例转换验证通过

### 0.47 爬虫写库失败时已补控制台定位信息
- 已增强 `backend/app/api/endpoints/admin/crawler_tasks.py`
- 当前业务表写库失败时：
  - 任务运行日志会直接带上具体页码、条目序号和关键字段
  - 后端控制台会输出完整异常堆栈
- 当前定位信息示例包含：
  - `task_key`
  - `page`
  - `item_index`
  - `客户名称`
  - `联系人`
  - `手机号 / 跟进时间 / 录入时间 / 最后跟进时间 / 首次跟进时间`
- 这样后续再卡住时，不需要再从长 SQL 里反推是哪条数据有问题
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.46 kehu51客户列表中途停在第36页的原因已定位并修复
- 已确认客户列表任务不是总数问题
  - 任务 `id=2` 当前为 `page_end=-1`、`sync_mode=full`
  - 实际执行到 `page=36` 时写库报错中断
- 已定位具体错误：
  - `Incorrect datetime value: '2025-12-31  (周三) 19点02分' for column 'created_time'`
  - 因此只成功写入前 `35` 页，约 `35 * 15 = 525` 条，所以看起来只有 `500` 多条
- 已修复 `backend/crawler_sites/kehu51/formatter.py`
  - 新增兼容格式：`YYYY-MM-DD  (周X) HH点MM分`
  - 示例：`2025-12-31  (周三) 19点02分 -> 2025-12-31 19:02:00`
- 本次验证：
  - `python -m py_compile backend/crawler_sites/kehu51/formatter.py` 通过
  - 时间转换样例验证通过

### 0.45 kehu51客户列表任务支持先进页面取总条数
- 已更新 `backend/crawler_sites/kehu51/customer_list.json`
  - 客户列表任务补充预取配置
  - 运行前先访问 `https://s25.kehu51.com/App/customers/cuslist.aspx?ShowID=3&ViewName=allcus&templateID=0`
  - 优先读取页面 `#page_RecordCount`
  - 取不到时回退解析页面脚本里的 `recordCount`
- 已按当前参数固定模板基线：
  - `showID=3`
  - `viewName=allcus`
  - `gridName=cuslistnew2`
  - `whereSql=E513575924A85785`
  - `sortName=CreateTime`
  - `sortMode=desc`
  - `pageSize=15`
  - `recordCount=6939`
- 当前执行时会自动用页面最新总条数覆盖模板中的 `recordCount`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/crawler_sites/kehu51/formatter.py` 通过

### 0.44 kehu51客户跟进记录任务支持先进页面取总条数
- 已更新 `backend/crawler_sites/kehu51/request.json`
  - 跟进记录任务参数切换为新提供的一组：
    - `showID=1`
    - `viewName=allfollow`
    - `gridName=followlist`
    - `whereSql=E513575924A85785`
    - `customParam=1`
    - `targetID=3`
    - `targetType=1`
    - `targetField=UserID`
- 已新增预取配置：
  - 运行前先访问 `https://s25.kehu51.com/App/customers/follow/index.aspx`
  - 优先读取页面 `#page_RecordCount`
  - 若页面节点不存在，则回退读取脚本里的 `recordCount`
- 已完成后端运行逻辑：
  - 自动把最新总条数回填到 `ajaxParam.recordCount`
  - 预取阶段同样支持登录失效自动刷新凭证后重试
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/app/services/crawler.py backend/crawler_sites/kehu51/formatter.py` 通过

### 0.43 爬虫正式执行改为逐页实时写库
- 已调整正式执行流程：
  - 不再先把全部分页结果缓存到内存后统一写库
  - 改为每抓取一页，立即写入业务表并提交
- 当前效果：
  - 降低内存占用
  - 即使中途停止，前面已成功抓取的页面数据也会保留
  - 日志可直接定位到具体页的写库结果
- 演示模式仍保留内存预览逻辑
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.42 爬虫正式执行补业务表写入日志
- 已补充正式执行阶段日志：
  - 业务表写入结果 `synced / stop_after_incremental_boundary`
  - 未命中业务表映射时，明确记录回退写入 `crawler_task_results`
  - 业务表写入异常时，明确记录 `DB-ERROR`
- 本次排查结论：
  - `2026-06-29 15:07` 用户反馈那次没有产生新的 `crawler_task_runs` 记录
  - 说明请求未完整走到正式落库收尾阶段，不是单纯“写表失败”
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.41 全量模式每次执行都按新批次写入
- 已调整全量模式去重规则：
  - 不再复用原有 `source_dedupe_key`
  - 改为按 `task_key + run_id + page_no + item_index` 生成新的批次键
- 当前效果：
  - 全量模式下，每次执行都会按新一轮写入业务表
  - 不受通用结果表或上一次业务表已写入数据影响
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.40 爬虫任务新增增量/全量模式并支持增量提前停止
- 已为爬虫任务新增字段：
  - `sync_mode`
  - 默认值：`incremental`
- 已新增 SQL：
  - `backend/sql/alter_crawler_tasks_add_sync_mode.sql`
- 已执行同步：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/alter_crawler_tasks_add_sync_mode.sql`
  - `test` 成功
  - `prod` 成功
- 已完成后端规则：
  - 默认增量更新
  - 全量更新时不判断 `follow_time`
  - 增量模式下按 `follow_time` 判断新数据
  - 增量模式一旦遇到 `follow_time <= 数据库当前最大 follow_time` 的记录，立即停止本次任务
- 已完成前端页面配置：
  - 任务管理新建/编辑页增加“同步模式”选项
  - 详情页增加当前同步模式展示
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/app/models/crawler_task.py` 通过
  - `source ~/.nvm/nvm.sh && cd frontend && nvm use && npm run build` 通过

### 0.39 kehu51客户跟进记录按follow_time判断新数据
- 已调整客户跟进记录落库增量规则：
  - 先读取表 `crawler_kehu51_follow_records` 当前最大 `follow_time`
  - 仅当本次采集记录的 `follow_time` 晚于当前最大值时，才认定为新数据并写入
- 当前规则不再依赖“本次执行全部覆盖”逻辑
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.38 kehu51客户跟进记录任务键名不一致导致未落业务表
- 已排查出原因：
  - 任务表中的 `task_key` 实际为 `customer_follow_records`
  - 后端业务表落库判断使用的是 `kehu51_follow_records`
  - 导致任务执行虽然成功，但数据被写入通用表 `crawler_task_results`，没有写入 `crawler_kehu51_follow_records`
- 已兼容两种键名：
  - `kehu51_follow_records`
  - `customer_follow_records`
- 已同步把现网任务键名修正为统一值：
  - `kehu51_follow_records`
- 本次验证：
  - 已确认 `crawler_task_results` 中存在本次跟进记录结构化数据
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过

### 0.37 kehu51客户跟进记录补齐联系人和来源字段
- 已修复 `kehu51` 跟进记录新返回结构的格式化
  - 当前 `gridHtml` 时间轴结构可解析为结构化 `items`
- 已确认新增字段：
  - `联系人`
  - `来源`
- 已补建表基线：
  - `backend/sql/create_kehu51_crawler_tables.sql`
- 已新增增量 SQL：
  - `backend/sql/alter_kehu51_follow_records_add_contact_and_source.sql`
- 已补后端落库映射：
  - `contact_name <- 联系人`
  - `source_from <- 来源`
- 已执行同步：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/alter_kehu51_follow_records_add_contact_and_source.sql`
  - `test` 成功
  - `prod` 成功
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/crawler_sites/kehu51/formatter.py` 通过

### 0.36 kehu51客户跟进记录任务更新为最新curl
- 已按最新提供的 curl 更新：
  - `backend/crawler_sites/kehu51/request.json`
- 当前 `kehu51_follow_records` 已切换为：
  - `POST https://s25.kehu51.com/App/customers/follow/index.aspx`
  - `form_body.ajaxParam` 请求模式
- 已补后端分页参数覆盖逻辑：
  - 之前只会覆盖顶层 `pageIndex/pageSize`
  - 现已支持自动改写 `ajaxParam` JSON 内部的 `pageIndex/pageSize`
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py` 通过
  - 已确认模板中的 `viewName=allfollow`、`gridName=followlist`、分页字段可正常更新

### 0.35 kehu51自动刷新登录curl读取修复
- 已修复 `crawler_tasks` 读取站点凭证时遗漏 `login_curl` 的问题
  - 之前即使 `credentials.json` 里已经保存了登录 curl
  - 运行任务时仍可能提示“站点未保存自动刷新登录 curl”
- 已将当前 `kehu51` 登录 curl 写入：
  - `backend/crawler_sites/kehu51/credentials.json`
- 当前自动刷新登录凭证优先可直接走站点配置文件
- 本次验证：
  - `python -m py_compile backend/app/api/endpoints/admin/crawler_tasks.py backend/app/services/crawler.py` 通过
  - 已确认 `credentials.json` 中存在 `login_curl`

### 0.25 爬虫站点凭证支持数据库存储与后台更新
- 已新增爬虫站点凭证表模型：
  - `crawler_site_credentials`
- 已补 Alembic 迁移：
  - `backend/migrations/versions/a8b9c0d1e2f3_add_crawler_site_credentials_table.py`
- 当前凭证读取规则已调整为：
  - 优先读数据库中的站点凭证
  - 若数据库没有该站点，则回退读取 `backend/crawler_sites/<site_key>/credentials.json`
- 已新增后台站点凭证接口：
  - `GET /api/admin/crawler/sites/{site_key}/credentials`
  - `PUT /api/admin/crawler/sites/{site_key}/credentials`
- 已支持两种更新方式：
  - 直接粘贴 Cookie 字符串更新
  - 粘贴完整 curl 自动提取 headers / cookie 更新
- 已在后台 `爬虫任务管理` 列表中新增 `凭证` 入口
  - 可单独打开站点凭证弹窗
  - 可查看当前来源是数据库还是文件
  - 可维护备注
- 已兼容原有文件模式
  - 后台更新数据库凭证时，会同步刷新站点目录下 `credentials.json`
  - 便于脚本模式与后台模式共用
- 已更新 `kehu51` 本地站点凭证文件为最新提供的 cookie
- 已生成并执行站点凭证建表 SQL：
  - `backend/sql/create_crawler_site_credentials_table.sql`
- 已通过同步脚本执行到两套环境：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/create_crawler_site_credentials_table.sql`
  - `test` 成功
  - `prod` 成功
- 已同步初始化 `kehu51` 凭证到数据库表 `crawler_site_credentials`
- 本次验证：
  - `python -m py_compile backend/app/services/crawler.py backend/app/api/endpoints/admin/crawler_tasks.py backend/app/models/crawler_task.py` 通过
  - `source ~/.nvm/nvm.sh && cd frontend && nvm use && npm run build` 通过

### 0.26 kehu51客户列表增加所属部门字段
- 已为 `kehu51客户列表` 落库映射增加字段：
  - `department_name`
  - 来源表头：`所属部门`
- 已更新建表基线文件：
  - `backend/sql/create_kehu51_crawler_tables.sql`
- 已新增增量 SQL：
  - `backend/sql/alter_kehu51_customer_list_add_department_name.sql`
- 已执行同步：
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/alter_kehu51_customer_list_add_department_name.sql`
  - `test` 成功
  - `prod` 成功

### 0.27 kehu51客户列表任务参数更新
- 已更新模板文件：
  - `backend/crawler_sites/kehu51/customer_list.json`
- 已按最新 curl 调整：
  - `sortName=CreateTime`
  - `recordCount=6935`
  - 补充 `accept / content-type / origin / referer / x-requested-with` 请求头
- `pageIndex` 仍保留模板默认值 `1`
  - 实际执行时继续由后端按分页自动覆盖

### 0.28 kehu51客户列表增加录入人与获得日期
- 已为客户列表落库增加字段：
  - `creator_name`，来源 `录入人`
  - `get_date`，来源 `获得日期`，若无则回退 `获得时间`
- 已更新建表基线：
  - `backend/sql/create_kehu51_crawler_tables.sql`
- 已新增增量 SQL：
  - `backend/sql/alter_kehu51_customer_list_add_creator_and_get_date.sql`
- 已执行同步：
  - `test` 成功
  - `prod` 成功

### 0.29 kehu51时间字段口语化转标准格式
- 已增强 `backend/crawler_sites/kehu51/formatter.py`
- 当前会自动把口语化时间转为标准格式后再落库
  - 例如：`昨天 (周五) 15点25分` -> `2026-06-26 15:25:00`
  - `昨天` -> `2026-06-26 00:00:00`
  - `今天` / `昨天` / `前天`
  - `xx分钟前` / `xx小时前` / `刚刚`
- 规则：
  - `...时间` 字段统一转 `YYYY-MM-DD HH:MM:SS`
  - `...日期` 字段统一转 `YYYY-MM-DD`

### 0.30 kehu51客户列表三项时间字段改为 DATETIME
- 已将以下字段改为真正的 `DATETIME` 类型：
  - `created_time`（录入时间）
  - `last_follow_time`（最后跟进时间）
  - `first_follow_time`（首次跟进时间）
- 已新增清洗脚本：
  - `backend/scripts/normalize_kehu51_customer_list_times.py`
- 已执行：
  - `python backend/scripts/normalize_kehu51_customer_list_times.py --both`
  - `python backend/scripts/run_sql_on_envs.py --both --sql-file backend/sql/alter_kehu51_customer_list_time_columns_to_datetime.sql`
- 执行结果：
  - 时间清洗：`test 0 条`，`prod 0 条`
  - 字段类型变更：`test` 成功，`prod` 成功

### 0.31 kehu51纯时分时间补当天日期
- 已补充时间解析规则：
  - `10点40分` -> 当天 `YYYY-MM-DD 10:40:00`
  - `10:40` -> 当天 `YYYY-MM-DD 10:40:00`
  - `10点40分12秒` -> 当天 `YYYY-MM-DD 10:40:12`

### 0.32 kehu51中文月日时间格式补齐
- 已补充时间解析规则：
  - `6月25日 (周四) 17点25分` -> `2026-06-25 17:25:00`
  - `6月25日` -> `2026-06-25`

### 0.33 kehu51整点时间格式补齐
- 已补充时间解析规则：
  - `6月25日 (周四) 17点整` -> `2026-06-25 17:00:00`
  - `昨天 (周五) 17点整` -> 对应日期 `17:00:00`
  - `17点整` -> 当天 `17:00:00`

### 0.34 kehu51登录 curl 支持自动刷新凭证
- 已新增后端能力：
  - `POST /api/admin/crawler/sites/{site_key}/credentials/refresh`
- 支持用登录 curl 自动执行刷新，并把新 cookie 写回站点凭证
- 已兼容 kehu51 两段式登录：
  - 第一步 `center.kehu51.com`
  - 第二步自动提交到 `s25.kehu51.com/users/login.aspx`
- 已支持返回并识别 `Set-Cookie` 中的过期时间
- 实测结果：
  - 可自动刷新出新的 `ASP.NET_SessionId`
  - 可自动刷新出 `kehu51cookie`
  - 可自动刷新出 `teamCreateManID`
  - 可拿到 `kehu51_server_username / kehu51_server_serverid` 的 `expires`
- 已补前端入口：
  - `爬虫任务管理 -> 凭证弹窗` 新增 `自动登录刷新` 按钮
  - 现在演示场景可直接粘贴登录 curl，点击按钮触发自动刷新
- 已补执行期自动兜底：
  - 如果任务请求返回“未登录 / 被迫退出 / Logout.aspx”等提示
  - 会自动读取已保存的登录 curl 刷新凭证
  - 刷新成功后自动重试当前请求一次

### 0.24 新增独立爬虫模块
- 已新增独立爬虫模块，支持两种抓取模式：
  - 页面 HTML 元素抓取
  - API 接口抓取
- 已新增后端接口：
  - `POST /api/crawler/html`
  - `POST /api/crawler/api`
- 已新增独立命令行脚本：
  - `backend/scripts/run_crawler.py`
- 当前可作为单独模块执行命令，不依赖具体业务页面
- 已补充 API 抓取请求体支持：
  - `json_body`
  - `form_body`
  - `raw_body`
- 已补充站点独立配置目录方案：
  - 每个站点一个目录
  - `credentials.json` 存 cookie / headers / 凭证
  - `request.json` 存默认抓取模板
- 已新增示例站点目录：
  - `backend/crawler_sites/kehu51/`
- 已继续补充爬虫任务管理后端打底：
  - 新增任务定义 / 执行记录 / 结果表模型
  - 新增爬虫任务管理接口
  - 已补 Alembic migration
- 已新增后台 `爬虫任务` 管理页面入口与列表页
- 当前页面只做配置管理，不执行真实任务，先用于确认后台管理可正常打开和增删改
- 已新增“通过 curl 导入站点凭证”能力
- 当前创建/编辑爬虫任务时可直接粘贴 curl：
  - 自动提取 headers
  - 自动提取 cookie
  - 自动提取 form body / raw body
  - 自动写入站点目录中的 `credentials.json` 和模板文件
- 已新增爬虫任务详情页
- 详情页支持调试执行：
  - 勾选“仅演示”时，不写入数据库
  - 取消勾选时，正式执行一次并写入执行记录与结果
- 已补充执行日志结果摘要输出，便于直接判断采集结果是否正确
- 已支持每个站点独立格式化处理：
  - 站点目录下新增 `formatter.py`
  - 每个站点可自行定义 `format_result`
- 已调整调试展示：
  - 日志摘要不再只打印 keys，改为直接打印部分内容预览
  - 演示模式最多只展示 10 条预览数据
- 已增强 `kehu51` 站点格式化结果：
  - 自动尝试解包 JSON 字符串
  - 自动提取 `items`
  - 自动提取 `total`
  - 自动补充分页字段
- 已修正演示模式返回限制：
  - 不再按 10 个页面块截断
  - 改为最多返回 10 条识别后的数据记录
- 已增强 `kehu51` 字段格式化：
  - 自动尝试识别并解码可读的 base64 字段
  - 解码后如果还是 JSON，会继续自动解包
- 已修复爬虫站点目录根路径错误：
  - 原先错误指向仓库根目录下 `crawler_sites`
  - 现已改为正确读取 `backend/crawler_sites`
- 已修复 `kehu51` 跟进记录接口返回仍显示 `gzip:...` 压缩串问题
  - 已确认 `gridHtml` 实际为 gzip+base64 压缩后的 HTML 表格
  - 已按表头 + 表体结构解析为可识别记录
  - 当前日志摘要会直接输出类似 `跟进时间 / 客户名称 / 跟进结果 / 录入人` 的结构化数据
  - 当前单页可正确识别 `15` 条记录，演示模式仍按最多 `10` 条预览返回
- 已简化爬虫任务创建弹窗
  - 默认只保留 `任务名称 / 站点目录 / 任务标识 / curl 导入`
  - 其余配置改为“高级配置”折叠区域，默认不展开
- 已确认同一站点支持复用一份凭证文件并拆分多个模板文件
  - `kehu51` 当前继续共用 `credentials.json`
  - 已新增客户列表模板 `backend/crawler_sites/kehu51/customer_list.json`
- 已补通 `kehu51客户列表` HTML 任务解析
  - 当前页面抓取目标为 `form#form1` 与 `#rptContentDiv`
  - 已识别 `#rptContentDiv` 内部同样是 `gzip+base64` 压缩后的表格 HTML
  - 已复用站点格式化器自动解压并提取客户列表字段
  - 当前已验证可正常返回 `客户名称 / 标签 / 联系人 / 手机号 / 所在城市 / 客户状态 / 最后跟进内容` 等结构化数据
- 已新增通用 SQL 同步脚本
  - `backend/tasks/run_sql_on_envs.py`
  - `backend/scripts/run_sql_on_envs.py`
  - 支持 `--env test` / `--env prod` / `--both`
  - 后续生成 SQL 后，如用户允许同步，可直接用该脚本同时对 `test/prod` 执行
- 已新增并同步 `kehu51` 两张采集表到 test/prod
  - SQL 文件：`backend/sql/create_kehu51_crawler_tables.sql`
  - 已执行：
    - `crawler_kehu51_follow_records`
    - `crawler_kehu51_customer_list`
  - 同步结果：
    - `test` 成功
    - `prod` 成功
- 已调整爬虫正式执行落库策略
  - 若任务已配置独立业务采集表，则优先写入独立表
  - 若任务没有独立表映射，才写入通用表 `crawler_task_results`
  - 当前已接入：
    - `kehu51_follow_records -> crawler_kehu51_follow_records`
    - `kehu51_customer_list -> crawler_kehu51_customer_list`
- 已支持分页结束页小于 `0` 的持续抓取规则
  - 当 `page_end < 0` 时，不按固定结束页停止
  - 会持续翻页抓取，直到返回结果无数据为止
  - 前端详情页分页展示已同步改为显示 `直到无数据`
- 已将 `kehu51客户列表` 任务改为 API 抓取
  - 模板文件 `backend/crawler_sites/kehu51/customer_list.json` 已切换为 `POST /app/Customers/CusTools.aspx?action=GetGridData`
  - 已支持按当前页码自动覆盖 `form_body.pageIndex`
  - 已同步任务配置到 `test/prod`
- 已修正演示模式执行上限
  - 后端不再只限制返回预览数量
  - 现改为演示模式执行过程中累计采到 `10` 条数据后立即停止
  - 避免 `page_end < 0` 时演示执行仍按正式模式持续跑完整流程
- 已调整给排水部助手任务大厅状态筛选
  - 状态下拉不再隐藏 `已指派 / 进行中 / 已打回`
  - 当前筛选项已与列表状态展示保持一致
- 已修正统计页时间筛选联动
  - 原先统计页查询时间仅作用于“人员排行榜”
  - 现已统一作用于统计概览、图表、排行表
  - 后端 `/software-task/stats/summary` 已支持 `date_from/date_to`
  - 前端统计页已改为整页共用同一组时间范围
- 已为给排水统计页补充图表类型切换
  - `项目类型统计` 支持柱状图 / 扇形图切换
  - `业务区域（部门）统计` 支持柱状图 / 扇形图切换
- 已在统计页最下方新增 `工作类别统计` 图
  - 后端统计概览新增 `task_by_task_type`
  - 前端已接入并按工作类别名称展示统计柱状图
- 已调整给排水统计页“各人员实际（天）统计”口径
  - 人员排行榜中的 `合计(天)` 规则为：`审核(天) + 任务(天)`
  - 当前下方 `各人员实际（天）统计` 已改为使用该合计值
- 已修正给排水统计页“工作类别统计”口径
  - 原先误按 `task_type` 统计，数据不对应给排水实际工作类别
  - 已改为按给排水任务真实使用的 `task_no`（工作类别名称）统计任务数量
- 已调亮统计页“工作类别统计”图颜色
- 已将统计页时间筛选提到最上方
  - 当前统计页所有统计图表、统计卡片、人员排行榜统一跟随最上面的时间筛选联动
- 已继续扩展统计页图表切换
  - `各人员任务统计` 支持柱状图 / 饼图切换
  - `各人员实际（天）统计` 支持柱状图 / 饼图切换
  - `Bug 数排名` 支持柱状图 / 饼图切换
  - `Bug 提报量排名` 支持柱状图 / 饼图切换
  - `工作类别统计` 支持柱状图 / 饼图切换

### 0.23 计划任务筛选样式与软件部入口调整
- 已将计划任务管理页的状态、调度类型筛选切换为统一的公共筛选下拉组件样式
- 已按项目规范对齐为自控部任务大厅列表筛选交互
- 已将“软件部任务工具”卡片默认跳转地址改为：
  - `/apps/software-task?tab=records`
- 当前从工作台/应用中心进入软件部任务工具，默认打开 `工作记录` Tab

### 0.22 下拉框样式规范补充
- 已将“后续下拉框默认复用自控部助手任务大厅列表筛选下拉框样式”正式写入 `AGENTS.md`
- 当前约定：
  - 默认优先使用仓库公共筛选下拉组件
  - 保持可输入、可清空、可展开收起、支持下拉筛选
  - 后续同类需求按该规范直接执行，不再每次单独强调

### 0.21 前端启动脚本接入 nvm
- 已调整 `frontend/start.sh`
- 已调整 `scripts/restart_frontend.sh`
- 前端启动前会优先尝试加载本机 `nvm`
- 默认读取 `frontend/.nvmrc`
- 当前已落地 `frontend/.nvmrc=20`
- 作用：启动前端时自动切到兼容 Vite 7 的 Node 20 版本，减少本机 Node 版本不匹配导致的启动失败

### 0.20 计划任务管理改为列表加详情
- 已将后台 `计划任务管理` 页面改为列表形式，支持按关键字、状态、调度类型筛选
- 已新增任务详情页：
  - 点击列表中的 `详情` 进入
  - 可查看任务说明、调度计划、下次执行、最近结果
  - 可在详情页执行 `立即执行 / 暂停 / 恢复`
- 已为计划任务管理接口补充详情接口与执行历史返回
- 已增加任务执行日志缓存：
  - 详情页可查看最近执行日志
  - 点击 `立即执行` 后，日志会在详情页输出

### 0.19 供应商活跃度计划任务拆分
- 已将供应商活跃度刷新从 `procurement_supplier_risk_refresh` 中拆出
- 已新增独立计划任务：
  - `procurement_supplier_activity_refresh`
  - 每天 `02:30` 执行
- 当前任务分工：
  - `procurement_supplier_risk_refresh`：每天 `02:00`，仅刷新供应商风险缓存
  - `procurement_supplier_activity_refresh`：每天 `02:30`，按 U8 采购订单刷新活跃度缓存
- 已同步更新后台任务元数据与说明文档：
  - `backend/app/api/endpoints/admin/scheduler.py`
  - `SCHEDULER_AND_SUPPLIER_ACTIVITY_NOTES.md`

### 0.18 供应商注册成功后自动进入主页
- 已调整 `供应商注册` 页成功流转：
  - 注册成功后不再回到登录页
  - 前端直接写入 `fp_supplier_profile`
  - 自动跳转到 `/fp/assistant`
- 当前效果：供应商注册成功后可直接进入供应商主页，无需再次手动登录

### 0.17 计划任务与供应商活跃度说明文档整理
- 已新增根目录文档 `SCHEDULER_AND_SUPPLIER_ACTIVITY_NOTES.md`
- 已整理当前系统计划任务列表、执行周期、作用说明
- 已确认现有 `procurement_supplier_risk_refresh` 任务已包含供应商活跃状态刷新能力，无需额外新增独立任务
- 已确认当前供应商活跃度排序已匹配最新规则：
  - 先分活跃 / 不活跃
  - 活跃供应商按最近下单时间倒序
  - 最近下单时间相同时按配置周期内下单次数倒序
- 已补充“如果后续要把活跃字段从发票口径改为订单口径”可选 SQL，当前不是必须执行

### 0.16 U8 数据库表说明文档整理
- 已新增根目录文档 `U8_TABLE_REFERENCE.md`
- 已整理当前仓库中已确认接入或查询过的 U8 表说明，包含：
  - `Vendor`
  - `pu_RelPomain`
  - `PO_Podetails`
  - `rdrecords32`
  - `Inventory`
  - `InventoryClass`
  - `fitemss00`
  - `PurInVoice`
- 文档内容包含每张表的用途、关键字段、当前代码用途、本地镜像关系和后续查表建议

### 0.15 采购部供应商管理活跃度规则调整
- 采购部 `供应商管理` 列表中的 `是否活跃` 已改为 `活跃度`
- 活跃度规则已改为基于 U8 `pu_RelPomain` 采购订单判断，不再按历史发票判断
- 当前规则：
  - 最近配置周期内有采购订单，判定为 `活跃`
  - 最近配置周期内没有采购订单，判定为 `不活跃`
- 活跃供应商排序规则已调整为：
  - 先按最近一次下单时间倒序
  - 最近一次下单时间相同时，再按配置周期内下单次数倒序
- 已新增后端配置项 `ACTIVE_SUPPLIER_ORDER_DAYS`，默认值 `720`
- 已同步环境配置：
  - `backend/config/env_prod`
  - `backend/config/env_test`
- 前端活跃度提示文案已改为不写死天数，避免后续调整配置后页面文案不一致
- 前端表头已改为 `活跃度`，并增加鼠标悬浮提示，使用通俗文案说明计算规则
- 后端语法校验已通过：
  - `python3 -m py_compile backend/app/services/sqlserver.py backend/app/api/endpoints/procurement_dept/procurement_dept.py`

### 0.14 供应商注册提示优化
- 已优化供应商注册接口提示文案：
  - 当 `Vendor` 中不存在对应企业名称时，返回 `供应商名称不存在，请填写系统中的企业名称`
  - 当 `suppliers` 中企业名称已存在时，返回 `该企业名称已注册，请直接登录或联系管理员重置密码`
  - 当 `suppliers` 中税号已存在时，返回 `该统一社会信用代码 / 纳税人识别号已注册，请直接登录或联系管理员重置密码`
- 已补充供应商注册税号格式校验：
  - 纯数字税号仅接受标准位数 `15 / 18 / 20`
  - 含字母的统一社会信用代码要求为 `18` 位大写字母数字组合
- 已优化前端 `供应商注册` 页错误提示：不再统一显示模糊文案，改为优先展示后端返回的具体错误原因
- 已确认 `雄新管业科技（天津）有限公司` 在 `Vendor` 中存在，且在 `suppliers` 中已注册，税号为 `91120224MA7EFAPG1T`

### 0.13 正式库 Vendor 同步排查
- 已按 `backend/config/env_prod` 查询正式 MySQL 库 `smart-cs-ai`
- 正式库本地 `Vendor` 表存在，当前 `COUNT(*) = 2113`
- U8 源库为 `UFDATA_001_2019.dbo.Vendor`，按非空 `cVenCode` 统计 `2142` 个编码
- 正式本地按 `cVenCode` 对比 U8 源库结果：
  - 本地缺少 `32` 个 U8 编码
  - 本地多出 `1` 个编码：`000527`
  - 本地有 `2` 个编码名称与 U8 不一致：`000932`、`020483`
- 正式本地 `Vendor` 表当前没有索引，且存在重复 `cVenCode`：
  - `000932`
  - `90289`
- 结论：正式库并非完全没有同步数据，而是同步副本已落后/不一致；当前同步脚本依赖 `cVenCode` 唯一索引做 UPSERT，但正式表没有该索引且存在重复编码，后续同步任务可能在创建唯一索引阶段失败或无法正确更新
- 进一步核对重复与差异明细：
  - `000932`：正式库存在两条，分别是 `天津市吉达企业管理咨询有限公司` 与 `天津晨天自动化设备工程有限公司`；U8 源库仅保留 `天津市吉达企业管理咨询有限公司`
  - `90289`：正式库存在两条同名记录 `雄新管业科技（天津）有限公司`，其中一条带联系人 `周猛 / 13582815544 / 杨宁`；U8 源库仅 1 条，且带联系人信息
  - `000527`：正式库存在 `天津建电电气有限公司`，U8 源库已无该编码
  - `020483`：正式库名称为 `浙江天目科技有限公司`，U8 源库名称为 `浙江宏天目科技有限公司`
  - 抽样确认若干缺失编码（如 `002345` ~ `002348`、`020518` ~ `020522`）正式库均不存在，但 U8 源库存在
- `2026-06-25 13:42:17 CST` 手动执行 `python backend/tasks/sync_vendor_to_mysql.py` 后，脚本输出 `2142 rows upserted`，但正式库 `Vendor` 被清空为 `0` 条
- 已定位根因：旧脚本按 `1000` 条一批执行 `DELETE ... WHERE cVenCode NOT IN (...)`，会把后一批尚未处理的编码提前删掉，最终把整表删空
- 已修复脚本 [backend/tasks/sync_vendor_to_mysql.py](/Users/sunday/work/CHENTIAN/ai/smart-cs-ai/backend/tasks/sync_vendor_to_mysql.py:130)：
  - 改为先写入临时表 `Vendor_sync_seen_codes_tmp`
  - 再通过 `LEFT JOIN ... IS NULL` 一次性删除不在 U8 源表中的本地残留编码
  - 已通过 `python3 -m py_compile backend/tasks/sync_vendor_to_mysql.py` 静态校验
- 当前正式库 `Vendor` 数据仍需恢复后，再使用修复后的脚本重新同步

### 0.12 采购部供应商管理白屏修复与 Excel 导入
- 修复采购部助手 `供应商管理` 页白屏：`supplierPriceLevelFilter` 等筛选状态在初始化前被 `useEffect` 引用，已调整声明顺序
- 新增导入脚本 `backend/scripts/import_procurement_suppliers_from_excel.py`
- 已将 `/Users/sunday/Downloads/供应商分类20251101.xlsx` 的主表数据导入 `procurement_suppliers`
- 导入规则：按 `供应商名称` 更新或新增；名称匹配时统一中英文括号；过滤 Excel 里的说明行脏数据
- 已执行导入结果：`rows=342 inserted=0 updated=342`
- 已将本次导入名单中的 `335` 家唯一供应商统一标记为 `活跃`
- 采购部供应商编辑弹窗中的 `供大类` 下拉已改为与筛选区一致的可输入筛选下拉样式
- 供应商管理列表与编辑弹窗中的 `供价格高中低 / 供服务高中低` 文案已收口为 `供价格 / 供服务`
- 供应商管理中的 `是否活跃` 排序已改为与自控部助手列表排序一致的表头双箭头点击样式
- 项目规范已补充：后续页面内新增下拉框、排序筛选默认使用现有“筛选下拉”样式，并与自控部助手 `任务大厅-优先级` 样式保持一致
- 项目规范已补充：表格超宽出现横向滚动时，默认参照给排水助手列表的固定底部横向滚动条方案
- 采购部 `供应商管理` 列表已接入固定底部横向滚动条：首屏底部常驻，原生横向滚动条进入当前视口后自动隐藏固定条
- 已抽出公共复用能力：
  - `frontend/src/components/ui/filterable-select.tsx`
  - `frontend/src/components/ui/use-fixed-bottom-scrollbar.ts`
- 项目规范已补充：筛选下拉与固定底部横向滚动条优先走公共组件 / hook 复用
- 已修正公共固定底部横向滚动条显示条件，改为与给排水助手一致，首屏进入视口即显示
- 已补强公共固定底部横向滚动条宽度计算：按表格真实内容宽度判断并监听内容尺寸变化，避免外层容器判断正常但固定条不显示
- 已修正采购部助手页面挂载位置错误：固定底部横向滚动条现在挂在 `供应商管理` 列表本体，不再误挂到 `发票管理`
- 已将供应商列表表格改为 `min-w-max`，确保列不会被压缩，从而真实触发横向滚动
- 已修正供应商管理筛选交互：采购负责人、供价格、供服务、供大类均改为“点击搜索才执行”，下拉点选不再自动触发查询
- 已修正供价格、供服务筛选不生效问题：搜索按钮会统一把输入值落到实际查询参数
- 已修正供应商列表筛选 500：`/procurement-dept/suppliers` 的 total 统计 SQL 补齐 `procurement_suppliers ps` 关联，避免 `ps.service_level / ps.price_level` 在 where 中找不到列
- 已统一采购部助手下拉数据源：`供应商管理-采购负责人` 与 `发票管理-采购员` 共用 `/supplier/procurement-users`
- 已统一采购负责人显示名：供应商管理列表/编辑弹窗以下拉框名称为准展示，`VIP*` 后缀会自动补齐同步
- 已批量同步数据库采购负责人名称到下拉框名字：
  - `Vendor.cCreatePerson` 更新 `184` 条
  - `procurement_suppliers.buyer_name` 更新 `193` 条
- 采购部 `供应商管理` 列表已新增 `联系人 / 电话` 排序，样式与自控部列表排序一致
- 采购部 `供应商管理` 列表中的 `经营范围（品牌）/经营范围（产品）` 已支持拖拽改宽，默认宽度调整为 `供应商名称` 列宽的约 50%
- 采购部 `供应商管理` 经营范围列拖拽手柄样式已对齐给排水部助手 `项目名称` 列
- 已清理异常说明行 `确定项目用途：给水、污水用，营业执照、检测报告、合格证`
- 抽样确认：`格兰富水泵(上海)有限公司 / 德·威世特水泵设备(北京)有限公司 / 安徽赛晟机电设备有限公司 / 上海中韩杜科泵业制造有限公司 / 森福(天津)科技咨询有限公司` 已回填类型、品牌、价格等级、服务等级、采购策略、供大类

相关文件：
- `frontend/src/pages/procurement_dept/index.tsx`
- `backend/scripts/import_procurement_suppliers_from_excel.py`
- `AGENTS.md`

### 0.11 Codex 插件历史进度自动读取配置
- 已加强项目级 `AGENTS.md`：Codex 插件重启、会话恢复、上下文压缩后优先读取 `PROJECT_HISTORY.md`
- 已加强全局 `~/.codex/AGENTS.md`：进入项目时先遵循项目级 `AGENTS.md`，如存在 `PROJECT_HISTORY.md` 则优先读取
- 已约定重要改动、提交、发布、排查后追加写入 `PROJECT_HISTORY.md`，方便后续恢复上下文
- 已补充企微 OAuth 用户敏感字段落库：`users.mobile` 字段、用户 Schema、管理员用户搜索、企微登录同步 `email/mobile`
- 已同步前端用户类型、后台用户列表展示、用户编辑弹窗手机号字段
- 邮箱继续使用已有 `users.email` 字段，手机号新增 `users.mobile`
- 手动 SQL：`ALTER TABLE users ADD COLUMN mobile VARCHAR(50) NULL COMMENT '手机号' AFTER email;`
- SQL Server 默认驱动已对齐为 `ODBC Driver 18 for SQL Server`，现场可用，不再依赖 `FreeTDS`
- 软件部日报提醒已改为直接按“昨日未填写日报名单”推送，不再按部门全量逐个排除
- 补充 `backend/Dockerfile`，为后端容器安装 `unixodbc-dev` 与 `msodbcsql18`
- 计划任务管理页面“下次执行”改为优先展示真实 next_run_time，并修正时间格式为 `YYYY-MM-DD HH:mm:ss`
- 采购部合同管理新增 Redis 查询缓存，支持关键词模糊查询，TTL 10 分钟
- Redis 缓存连接失败时自动降级直查，避免合同列表接口被缓存超时拖挂
- 新增采购合同同步表 `procurement_contract_sync_rows`，合同管理可切换为本地同步表查询
- 新增凌晨 04:00 的采购合同同步任务，从 Ecology 合同视图全量同步到本地表
- 采购合同管理已收口为只读本地同步表 `procurement_contract_sync_rows`，本地无数据时直接返回空，不再回退 Ecology 视图
- 采购合同同步表补充 `hkje / kpje` 两个金额列，并从同步原始数据中提取回填，方便前端直接展示核开金额与开票金额
- 合同管理前端改为强制合并显示 `hkje / kpje`，并兼容接口字段缓存，避免列名丢失导致前端看不到金额列
- 采购合同列表查询已进一步收口为固定字段 `htbh/htmc/je/hkje/kpje` 直出，并增加 `has_amount` 排序字段，减少 `raw_data` 解析开销

相关文件：
- `AGENTS.md`
- `~/.codex/AGENTS.md`
- `PROJECT_HISTORY.md`
- `backend/app/models/base.py`
- `backend/app/schemas/user.py`
- `backend/app/api/endpoints/auth.py`
- `backend/app/api/endpoints/admin/users.py`
- `backend/migrations/versions/a0b1c2d3e4f6_add_mobile_to_users.py`
- `frontend/src/types/index.ts`
- `frontend/src/pages/admin/UsersPage.tsx`
- `frontend/src/components/admin/UserEditor.tsx`
- `backend/app/core/config.py`
- `backend/config/env_prod`
- `backend/config/env_test`
- `backend/.env`
- `backend/app/tasks/scheduler.py`
- `backend/Dockerfile`
- `backend/app/api/endpoints/admin/scheduler.py`
- `frontend/src/pages/admin/SchedulerPage.tsx`
- `backend/app/services/redis_cache.py`
- `backend/app/services/ecology_mysql.py`
- `backend/requirements.txt`
- `backend/requirements-win.txt`

### 0.10 企微头像同步排查结论
- 已确认企微登录流程会在 `POST /api/login/sso` 中调用企业微信成员详情接口，并尝试同步 `avatar`
- 已调整同步逻辑：只有企微返回非空头像时才覆盖本地头像，避免空值把现有头像冲掉
- 已确认 `康鹏` 当前企微成员详情返回中未包含 `avatar / thumb_avatar / qr_code` 字段，因此右上角会继续使用默认头像兜底
- 排查期加入过临时日志，当前已移除，登录逻辑恢复正常
- 已新增独立 `privateinfo` 敏感字段授权流程，企微内登录且当前头像为空时，会自动追加一次 `snsapi_privateinfo` oauth2 授权
- 已取消 `康鹏/tangpeng` 试点限制，所有用户都可走这条敏感字段补充流程

相关文件：
- `backend/app/api/endpoints/auth.py`
- `backend/app/services/wecom.py`
- `frontend/src/pages/AuthCallback.tsx`
- `backend/.env`

### 0.9 软件部日报提醒链接页签修正
- 软件部日报提醒中的一键填写链接已由 `?tab=work-records` 改为 `?tab=records`
- 软件部任务工具页补充旧参数兼容，历史消息即使仍带 `work-records` 也会自动切到“工作记录”页签
- 软件部日报提醒测试完成，已去掉单发 `康鹏` 的测试逻辑，改为通过企业微信群机器人推送群消息
- 新增 `WECOM_GROUP_WEBHOOK_URL` 配置项，用于配置企微机器人 webhook 地址
- 顶部右上角头像改为优先显示已同步的企业微信头像；若无企微头像，再回退现有默认头像
- 开发登录不再覆盖已有企业微信头像，避免本地调试后把右上角头像改成测试头像
- 开发者快速登录新增密码校验，密码改为从 `DEV_LOGIN_PASSWORD` 环境变量读取

相关文件：
- `backend/app/core/config.py`
- `backend/app/services/wecom.py`
- `backend/app/api/endpoints/auth.py`
- `backend/app/tasks/scheduler.py`
- `frontend/src/components/layout/NavBar.tsx`
- `frontend/src/pages/admin/components/AdminNavBar.tsx`
- `frontend/src/pages/software_task/index.tsx`
- `backend/app/api/endpoints/admin/scheduler.py`

### 0.8 采购部供应商管理增加企业风险缓存与详情
- 采购部助手 `供应商管理` 列表新增 `风险` 列
- 风险数据改为计划任务定时刷新，不在列表页实时调用外部接口
- 每天凌晨 `02:00` 执行供应商风险扫描
- 风险状态为 `normal` 的供应商按隔天刷新
- 风险状态为 `risk` 的供应商按每天刷新
- 点击风险列可弹出企业工商信息与风险详情
- 后端新增供应商风险详情接口，并支持缓存字段回填
- `是否活跃` 改为基于 U8 `PurInVoice` 发票主表，通过 `cVenCode -> Vendor.cVenCode` 关联判断最近两年是否有采购发票
- 后台新增 `计划任务管理` 页面，可查看任务、手动执行、暂停、恢复
- 计划任务管理补充中文名称与用途说明，便于直接理解任务作用
- 补充纳入计划任务管理的 `U8供应商同步` 任务，每天 `01:30` 执行
- 新增 `软件部日报提醒` 任务，每天 `09:00` 检查昨天未填写工作记录的员工；当前仅控制台输出提醒模板，不推送企微
- 软件部日报提醒仅按“昨天是工作日”时统计；若昨天是休息日则跳过
- 软件部日报提醒支持按 `.env` 配置节假日列表，命中节假日时也跳过
- 软件部日报提醒中的“一键填写”链接已改为完整前端地址
- 补充排查记录：软件部日报提醒链接当前使用 `?tab=work-records`，但软件部页面内部实际工作记录页签 key 为 `records`，因此点击后不会自动切到工作记录页签，后续需改链接参数或补页签别名兼容

相关文件：
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`
- `backend/app/tasks/scheduler.py`
- `backend/app/api/endpoints/admin/scheduler.py`
- `backend/app/services/sqlserver.py`
- `backend/app/models/procurement_dept.py`
- `backend/app/schemas/procurement_dept.py`
- `frontend/src/pages/procurement_dept/index.tsx`
- `frontend/src/pages/admin/SchedulerPage.tsx`

### 0.7 后端接入 PrimeMatrix 外部 MCP 企业信息服务
- 后端新增 PrimeMatrix 企业信息 MCP 客户端接入
- 支持两种调用方式：
  - `http`：直连远端 MCP 服务地址
  - `stdio`：通过 `npx prime-matrix-data` 启动本地 MCP 客户端
- 新增企业信息查询接口 `/api/prime-matrix/company`
- 查询流程已封装为：
  - 先调用 `company_name` 做模糊企业名转精确名
  - 再调用 `basic_info` 查询工商信息
  - 可按参数追加司法信息和风险信息
- PrimeMatrix 密钥和连接配置已加入 `backend/.env`

相关文件：
- `backend/app/api/endpoints/prime_matrix.py`
- `backend/app/services/prime_matrix_mcp.py`
- `backend/app/api/api.py`
- `backend/.env`

### 0.6 自控部任务表单与软件部详情展示调整
- 软件部任务详情时间轴中的“任务创建”已改为显示任务描述，不再显示项目标题
- 任务描述富文本转纯文本时，保留了换行、标点、列表符等基础格式
- 自控部助手新建任务的“工作类别”已去掉默认值
- 自控部助手新建任务的“工作类别 / 所属项目 / 业务人员”输入框已支持点击 `×` 清空当前选择
- 业务人员下拉接口已去掉软件部权限限制，登录用户均可获取列表
- 更新了 `frontend/dist.zip`，用于同步当前前端发布包

相关文件：
- `frontend/src/pages/software_task/index.tsx`
- `frontend/src/pages/auto_ctrl/index.tsx`
- `backend/app/api/endpoints/software_task/software_task.py`
- `frontend/dist.zip`

### 0.5 企业微信卡片增加后端登录中转
- 企业微信卡片链接已改为先走后端中转地址
- 中转后跳到前端 `/login?redirect=...`，用于兜底触发企微登录，再回到采购部 `发票管理` 页签

相关文件：
- `backend/app/api/endpoints/auth.py`
- `backend/app/api/endpoints/supplier.py`

### 0.4 采购员关联表增加启用状态并收口下拉读取
- `procurement_buyer_wecom_u8_map` 增加启用/禁用状态字段，默认启用
- 供应商助手采购员下拉已改为只读取关联表中 `enabled` 状态的用户
- 采购部采购员下拉也优先只读取关联表中 `enabled` 状态的用户
- 企业微信消息推送匹配采购员时，同样只认 `enabled` 映射

相关文件：
- `backend/app/models/procurement_dept.py`
- `backend/app/api/endpoints/supplier.py`
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`
- `frontend/dist.zip`

### 0.3 前端构建产物更新
- 更新了 `frontend/dist.zip`，用于同步当前前端页面最新改动后的发布包

相关文件：
- `frontend/dist.zip`

### 0.2 供应商助手导入发票流程修正与测试采购员
- 修复了供应商助手“先选采购员再上传”流程中的错误校验，避免确认时提示“无法保存，请重新点击上传”
- 为测试企业微信消息推送，供应商助手采购员下拉临时增加 `康鹏`

相关文件：
- `backend/app/api/endpoints/supplier.py`
- `frontend/src/pages/SupplierAssistantPage.tsx`

### 0.1 采购部采购员关联表与供应商上传发票流程调整
- 新增 `procurement_buyer_wecom_u8_map`，用于关联采购部企微用户与 U8 采购员名称
- 支持按名字自动初始化 `users` 与 U8 `Vendor.cCreatePerson` 的映射
- 采购员下拉读取 U8 名单时，会同步刷新这张关联表中的 `name_auto` 自动映射数据
- 供应商助手 `/fp/assistant` 的“导入发票”已改成：先选择采购员，再打开文件上传窗口
- 上传发票时会直接写入采购员，不再上传后再选
- 上传成功后，如果能匹配到企微人员，会向采购员推送企业微信卡片消息
- 企业微信卡片点击后直达采购部助手 `发票管理` 页签
- `/fp/assistant` 中“选择采购员”弹窗已改为统一下拉样式，并修复下拉层被弹窗裁切的问题

相关文件：
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`
- `backend/app/api/endpoints/supplier.py`
- `backend/app/models/procurement_dept.py`
- `backend/app/models/__init__.py`
- `backend/migrations/versions/a9b8c7d6e5f4_add_procurement_buyer_wecom_u8_map.py`
- `frontend/src/pages/SupplierAssistantPage.tsx`
- `frontend/src/pages/procurement_dept/index.tsx`

### 0. 采购部发票管理列表补齐字段与导出/作废能力
- 发票管理列表字段已调整为：`序号 / 合同编号 / 收到状态 / 发票信息 / 开票时间 / 发票抬头 / 发票号 / 开票金额 / 税率 / 发票状态 / 采购员 / 税额`
- `收到状态` 当前固定显示为 `是`
- `发票信息` 当前固定显示为 `增值税发票`
- `发票抬头` 使用 `销方名称`
- 新增导出按钮，导出文件名为 `采购部发票管理列表_日期时间.xlsx`
- 导出内容与当前查询结果一致
- 发票状态新增 `作废`，支持筛选 `作废`
- 操作列新增 `作废` 按钮，作废后不再允许继续流转

相关文件：
- `frontend/src/pages/procurement_dept/index.tsx`
- `backend/app/api/endpoints/procurement_dept/procurement_dept.py`
- `backend/app/schemas/procurement_dept.py`
- `backend/app/models/procurement_dept.py`

### 1. 软件部任务支持“多负责人拆分”
- 新增任务基础表：`software_task_bases`
- 原任务表新增关联字段：`software_tasks.task_base_id`
- 新建任务支持多负责人：前端多选后，后端按负责人拆分生成多条任务
- 为统计归并保留基础任务信息（基础表）

相关文件：
- `backend/app/models/software_task.py`
- `backend/app/schemas/software_task.py`
- `backend/app/api/endpoints/software_task/software_task.py`
- `backend/migrations/versions/a8f1d2c3e4b5_add_software_task_base_table_and_fk.py`

### 2. 任务列表增强
- 多人任务在标题后展示双人图标（依据 `task_base_id`）
- 新建/编辑任务中负责人支持“下拉复选框”多选

相关文件：
- `frontend/src/pages/software_task/index.tsx`

### 3. 筛选框样式统一
已将以下模块筛选项逐步统一为“可输入 + 下拉 + 清空 + 展开”风格：
- 任务大厅：项目、负责人、状态
- 工作记录列表：项目、填写人
- Bug 列表：项目、状态、负责人
- 新建工作记录：项目
- 登记 Bug：项目、关联任务、Bug负责人（当前为可输入联想样式）

### 4. 自控/给排水近期调整
- 自控部助手“任务跟踪”支持任务类别手动输入保存
- 给排水部助手新建/编辑任务负责人支持单选输入，并补充清空 `X`
- 给排水部统计页人员排行榜调整为“审核 / 任务 / 合计”口径，去掉单独实际天数
- 给排水部审核任务放宽为“管理员或对应审核人”可操作
- 自控部图表分析时间范围统一为查询后刷新，避免日历切月上下箭头交互异常

## 部署说明（正式环境）
- 服务器：`172.18.2.31`
- 项目路径：`/opt/docker-data/smart-cs-ai/`
- 后台更新：
  - `cd /opt/docker-data/smart-cs-ai/backend`
  - `git pull`
- 前台发布：
  - 本地 `npm run build`
  - 上传 `dist` 到 `/opt/docker-data/smart-cs-ai/frontend`

## 数据库执行提醒
如果尚未执行迁移，请运行：
```bash
cd backend
alembic upgrade head
```

## 常见问题

### 1) SQLAlchemy mapper 初始化报错（SoftwareTaskBase.tasks）
报错关键词：
- `Could not determine join condition ... SoftwareTaskBase.tasks`

处理：
- 已在模型中为 `SoftwareTaskBase.tasks` 显式指定 `foreign_keys=SoftwareTask.task_base_id`

### 2) 前端白屏（isWaterSupply is not defined）
处理：
- 删除了 `software_task/index.tsx` 文件末尾误插入的组件外孤立代码

### 5. 软件中心日报提醒仅允许 prod 发送
- 软件中心“日报提醒”定时任务增加环境限制，只有 `APP_ENV=prod` 时才会向企微群发送提醒
- `test` 或未设置环境时，任务会直接跳过，并在调度结果中记录“仅 prod 发送”的原因
- 后端配置新增 `APP_ENV`，并已同步环境文件：
- `backend/config/env_prod` 设置为 `APP_ENV=prod`
- `backend/config/env_test` 设置为 `APP_ENV=test`

### 6. 给排水部助手任务大厅横向滚动条固定到底部
- 给排水部助手 `任务大厅` 列较多时，新增一个常驻视口底部的横向滚动条
- 固定滚动条与表格本体横向滚动同步，用户在首屏底部即可左右拖动查看列
- 仅在给排水列表且确实发生横向溢出时显示，不影响软件部列表
- 固定条样式已调整为更接近系统原生滚动条；当固定条出现时，列表自身原有横向滚动条会隐藏
- 本地未完成前端构建校验：当前环境 `npm` 所用 Node 版本缺少 `node:path` 模块，需切到较新的 Node 后再执行 `npm run build`

### 7. 采购部助手供应商管理补充经营与评估字段
- 供应商管理列表新增字段：`供应商类型`、`经营范围（品牌）`、`经营范围（产品）`、`账期`、`货期`、`供价格高中低`、`供服务高中低`、`采购策略/是否建议做年协`
- 供应商编辑弹窗同步支持维护上述字段
- 后端供应商列表接口和保存接口已支持这些字段，数据落在 `procurement_suppliers` 表，不直接改 U8 `Vendor` 表结构
- 已补 Alembic migration：`backend/migrations/versions/a1b2c3d4e5f7_add_procurement_supplier_extended_fields.py`
- 后端语法已通过 `python3 -m compileall`
- 前端构建未执行：当前环境 `npm` 所用 Node 版本缺少 `node:path` 模块，需切到较新的 Node 后再执行 `npm run build`

### 8. 采购部发票管理采购员下拉补回未映射人员
- 采购员下拉原先在 `procurement_buyer_wecom_u8_map` 存在启用数据时，只返回映射表中的启用项
- 这会导致像 `王凯VIP1` 这类采购部用户因未映射到 U8 `王凯` 而不出现在下拉中
- 现已调整为：优先返回启用映射项，同时把采购部用户、U8 Vendor 建档人、历史发票采购员中的未映射项追加到后面
- 这样 `杨宁` 这类已映射人员仍优先展示，`王凯VIP1` 这类未映射人员也不会再丢失
- 后续确认发票管理页面实际调用的是 `/supplier/procurement-users`，该接口也已同步改成同样规则，避免前后端命中不同接口导致看起来“改了但没生效”

### 9. 采购部发票管理采购员下拉改为仅采购部人员
- 根据最新要求，发票管理页采购员下拉不再混入 U8 建档人、历史发票采购员、映射表名称
- `/supplier/procurement-users` 现仅返回 `采购部` 及其下级部门中的启用用户
- 测试库实际结果已确认包含：`杨宁`、`王凯VIP1`

### 10. 供应商管理支持按是否活跃排序
- 供应商管理列表新增“是否活跃”表头排序
- `desc` 时为“活跃在前”，`asc` 时为“不活跃在前”
- 前端点击表头即可切换，后端按 `procurement_suppliers.is_active` 排序返回
- 重置筛选时默认恢复为“活跃在前”

### 11. 供应商管理新增供大类及多项筛选
- 供应商管理新增筛选条件：`采购负责人`、`供价格`、`供服务`、`供大类`
- `供价格`、`供服务` 使用 `高 / 中 / 低` 下拉筛选
- 新增 `procurement_supplier_categories` 供应商大类表，并在 `procurement_suppliers` 上增加 `supplier_category_id`
- 供应商列表接口、供应商编辑弹窗、供应商列表展示已支持 `供大类`
- 后端语法已通过 `python3 -m compileall`
- 前端构建未执行：当前环境 `npm` 所用 Node 版本缺少 `node:path` 模块，需切到较新的 Node 后再执行 `npm run build`

### 12. 供应商名称改为单行显示
- 供应商管理列表中的 `供应商名称` 单元格已改为 `whitespace-nowrap`
- 现在名称不会再因列宽被自动折成多行，而是保持单行展示，配合横向滚动查看全名

### 13. 供应商管理按供应商名称唯一并清洗历史重复
- `procurement_suppliers` 的唯一约束由 `supplier_name + buyer_name` 改为仅 `supplier_name`
- 历史重复数据会在迁移中按 `supplier_name` 合并为一条
- 合并规则：`buyer_name` 采用“后面的非空覆盖前面的”，其余扩展字段也优先取后面最近一条非空值
- 风险、活跃、发票计数等缓存字段按最大值或最新时间合并
- 后端后续写入与更新也改为按 `supplier_name` 单键维护，不再按 `supplier_name + buyer_name` 拆多条

### 14. 供应商管理风险列加宽
- 供应商管理列表 `风险` 列宽已从较窄状态调大
- 表头与单元格同步增加最小宽度，降低“正常 / X项风险”显示拥挤的问题

### 15. 供应商管理筛选样式对齐自控部下拉框
- 供应商管理中的 `采购负责人`、`供价格`、`供服务`、`供大类` 已改成与自控部列表一致的可输入下拉样式
- 支持输入过滤、展开收起、清空 `X`、点击外部关闭
- 当前环境未执行前端构建：`npm` 对应 Node 版本过旧，仍需切到较新的 Node 后再跑 `npm run build`

### 16. 供应商管理重置按钮补全排序清理
- 采购部助手 `供应商管理` 页重置按钮已补齐清理 `联系人 / 电话` 列排序状态
- 现在点击重置会同时清空筛选条件、恢复 `是否活跃` 默认排序，并把分页回到第一页
- 已通过前端类型检查验证；当前机器上的 `npm run build` 仍受 Node 14 过旧影响，无法直接跑通

### 17. 供应商类型列宽度样式对齐品牌列
- 采购部助手 `供应商管理` 列表中的 `供应商类型` 已改成与 `经营范围（品牌）` 同款可拖拽列宽样式
- 现在 `供应商类型` 与 `经营范围（品牌）` 共用同一宽度基准，拖拽表现保持一致

### 18. 供应商活跃状态改为只增不减
- 采购部供应商活跃状态自动化刷新已改为保留历史已活跃状态
- 现在定时任务只会把“有发票/已活跃”的供应商继续维持为活跃，不会再把已经标记为活跃的供应商改回不活跃
- Excel 导入后置为活跃的供应商也会保持活跃，不会被后续自动化任务覆盖掉

---

> 建议：每次完成较大功能后，追加一段变更摘要到本文件。

### 19. kehu51 跟进结果异常数据已同步修复到测试和正式
- 新增脚本 `backend/scripts/fix_kehu51_follow_results.py`，用于修复 `crawler_kehu51_follow_records` 中 `follow_result` 与 `raw_json.跟进结果` 被截断为 `<span style=\` 的异常数据
- 已执行测试与正式双环境同步修复命令：`python scripts/fix_kehu51_follow_results.py --both --apply`
- 本次执行结果一致：命中 886 条，成功修复 4 条，剩余 882 条因数据库原始内容已同时截断，当前无法直接还原
- 如后续需要彻底修复剩余异常数据，需清空异常值后重新爬取补数

### 20. kehu51 跟进结果无法恢复的脏数据已清空
- 已执行测试与正式双环境清理命令：`python scripts/fix_kehu51_follow_results.py --both --apply --clear-unrecoverable`
- 本次执行后，测试与正式环境各自共处理 886 条异常记录
- 其中可恢复的 4 条保留修复结果，其余 882 条无法恢复的 `follow_result` / `raw_json.跟进结果` 已清空，避免页面继续显示 `<span style=\` 脏内容

### 21. okcis 自动登录已修复验证码 403 与请求头大小写问题
- 修复 `backend/app/services/crawler.py` 中 `User-Agent`、`Accept-Language` 等请求头按小写取值导致丢失的问题，避免验证码图片请求被站点拦截为 `403`
- `okcis` 自动登录流程已改为先访问 `https://www.okcis.cn/login/` 获取最新 cookie，再带最新 cookie 请求验证码与登录接口
- 验证码算式解析已改为只取单个数字运算，忽略尾部 `=`、`?` 误识别内容
- 本地实测：`refresh_credentials_from_login_curl()` 已能成功拿到验证码并完成登录，随后调用 `check_login_state.php` 返回已登录页面内容，不再是 `nologin`

### 22. git 提交前已补充 okcis 采集能力与依赖变更记录
- 已整理本次 `okcis` 相关改动，包含站点目录、手动任务 SQL、专表建表 SQL、迁移脚本、自动登录修复与任务执行链路调整
- 本次依赖变更已包含 `backend/requirements.txt` 与 `backend/requirements-win.txt` 中的 `Pillow==11.3.0`
- 本次提交会一并纳入未跟踪的 `backend/crawler_sites/okcis/`、迁移文件与 SQL 文件，方便后续回溯

### 23. okcis 手动执行已补开始日志与空结果日志
- `backend/app/api/endpoints/admin/crawler_tasks.py` 已增加 `[START]`、`[DZID]` 控制台日志，点击演示或正式执行后会立即输出任务启动信息
- `okcis` 返回 `没有找到相关定制信息,请定制条件` 时，已按正常空结果处理并输出 `[EMPTY]`，不再误判为异常

### 24. 爬虫任务详情页已修复执行成功却误报失败的问题
- `frontend/src/pages/admin/CrawlerTaskDetailPage.tsx` 已调整执行提示逻辑：接口调用成功后优先提示执行完成
- 正式执行后的详情刷新失败会单独提示“执行完成，任务详情刷新失败”，不再整体弹出“执行失败”
- 前端错误提示已改为优先显示后端返回的 `detail`，便于直接定位问题

### 25. okcis 演示模式预览已裁剪大字段
- `backend/app/api/endpoints/admin/crawler_tasks.py` 已新增演示预览清洗逻辑
- `detail_content` 等超长字段会在演示返回时截断，避免前端因返回体过大或预览渲染过重而异常

### 26. okcis 每次执行前都会先清理不符合未来四天规则的旧数据
- `backend/app/api/endpoints/admin/crawler_tasks.py` 已新增 `_cleanup_okcis_notice_table`
- `okcis_notice_manual` 每次启动时都会先检查 `crawler_okcis_notices`，仅保留截止时间大于未来4天的数据，其余已到期或即将到期数据先删除
- 原先写库阶段的重复清理已移除，避免每页重复执行删除逻辑

### 27. kehu51 客户列表客户名称已统一去除三角标记
- `backend/crawler_sites/kehu51/formatter.py` 已在统一格式化阶段对 `客户名称` / `customer_name` 执行 `_clean_customer_name`
- 现在无论来自表格 HTML 还是嵌套 JSON，客户名称都会自动去掉 `▼`
