# AGENTS.md

本文件用于指导在本仓库内工作的 AI Agent、自动化脚本与协作者，目标是减少误改、降低沟通成本，并保证改动可验证。

## 0. 会话初始化（必读）

- 通用 Agent 规则优先读取：`agentic-codex/AGENT.md`
- 通用代码规范参考：`agentic-codex/rules/`
- 通用工作流参考：`agentic-codex/workflows/`
- 通用架构方案参考：`agentic-codex/architectures/`
- 涉及前端下拉框、日期选择、超宽表格、筛选分页联动时，先读取：`agentic-codex/rules/frontend-common-components.md`
- 若通用规则与本项目规则冲突，以用户当前要求和本项目 `AGENTS.md` 为准。
- 每次重新载入项目、Codex 插件重启、会话恢复或上下文压缩后，按顺序阅读：
  1. `agentic-codex/AGENT.md`
  2. 根目录 `CURRENT_CONTEXT.md`
  3. 根目录 `PROJECT_HISTORY.md`
  4. 如需更早细节，再查 `PROJECT_HISTORY.archive.md`
- 每次完成重要改动、提交、发布或排查后，需优先更新 `CURRENT_CONTEXT.md` 与 `PROJECT_HISTORY.md`。
- 当 `PROJECT_HISTORY.md` 明显过长、已影响上下文恢复效率时，应执行压缩：
  - 主文件仅保留当前摘要与最近关键记录
  - 旧内容移入 `PROJECT_HISTORY.archive.md`
  - 不要把大段日志、原始 HTML、超长 JSON 直接写进主历史文件

## 1. 项目概览

Smart CS AI 是一个企业级智能客服 AI 平台，当前仓库是单体仓（monorepo），包含 3 个核心子系统：

- `backend/`：Python `FastAPI` 异步后端，负责认证、RBAC、聊天、知识库、造价部等业务接口
- `frontend/`：`React 19 + TypeScript + Vite` 前端应用
- `edge_compute/`：本地语音识别服务，基于 `FastAPI + faster-whisper`

核心能力包括：

- 企业微信 OAuth 登录
- Coze Bot 流式对话
- 本地 ASR 音频转写
- 完整 RBAC 权限体系
- 造价部 Excel 文件解析与条目检索
- 知识库同步与管理

## 2. 目录约定

根目录主要内容：

- `backend/`：后端服务与数据库迁移
- `frontend/`：前端页面、组件、类型、接口封装
- `edge_compute/`：ASR 边缘服务
- `scripts/`：辅助脚本
- `data/`、`doc/`：数据与文档

后端重点目录：

- `backend/app/main.py`：FastAPI 入口
- `backend/app/api/`：路由注册与接口实现
- `backend/app/models/`：SQLAlchemy 模型
- `backend/app/schemas/`：Pydantic Schema
- `backend/app/services/`：业务逻辑与外部服务封装
- `backend/app/core/`：配置、安全、调度器等基础设施
- `backend/app/db/`：数据库会话与依赖
- `backend/migrations/`：Alembic 迁移脚本

前端重点目录：

- `frontend/src/App.tsx`：路由入口
- `frontend/src/pages/`：页面
- `frontend/src/components/`：复用组件
- `frontend/src/api/`：接口封装
- `frontend/src/store/`：Zustand 状态管理
- `frontend/src/types/`：类型定义

## 3. 技术栈

- 后端：`FastAPI`、`SQLAlchemy 2.x`、`Alembic`、`aiomysql`、`APScheduler`
- 前端：`React 19`、`TypeScript`、`Vite 7`、`Tailwind CSS v4`、`Zustand`、`shadcn/ui`
- 边缘服务：`FastAPI`、`faster-whisper`
- 外部集成：企业微信、Coze、通义

## 4. 常用开发命令

### 4.1 后端

首次安装：

```bash
cd backend
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

本机默认 Python 执行环境约定：

```bash
conda activate smart
```

后续在本项目内执行 Python、Alembic、FastAPI、脚本任务时，如无特殊说明，默认优先使用 `smart` conda 环境，不再混用系统 Python。

生产环境 Python 执行环境约定：

- 生产环境没有 `conda` 命令，也不直接使用 `pip` 命令。
- 生产环境执行 Python 固定使用 `/anaconda3/envs/smart/bin/python`。
- 生产环境安装依赖时使用 `/anaconda3/envs/smart/bin/python -m pip ...`。
- 生产环境执行 Alembic 时优先使用 `/anaconda3/envs/smart/bin/python -m alembic ...`。
- 后续给生产环境命令时，不要再写 `conda activate smart`、`conda run -n smart` 或裸 `pip`。

Windows 环境优先使用：

```bash
pip install -r requirements-win.txt
```

本地开发：

```bash
conda activate smart
cd backend
uvicorn app.main:app --reload --port 8000
```

数据库迁移：

```bash
conda activate smart
cd backend
alembic revision --autogenerate -m "description"
alembic upgrade head
```

### 4.2 前端

```bash
cd frontend
npm install
npm run dev
./build.sh
```

如果本机 `node` / `npm` 环境异常，允许优先使用 `nvm` 切换或安装合适版本后再执行前端命令。后续遇到类似 `npm` 缺失内置模块、版本过旧、构建链不兼容等问题，默认先检查当前 `node -v`、`npm -v`，必要时通过 `nvm use` / `nvm install` 处理，并把实际使用的 Node 版本记录到结果说明里。

### 4.3 根目录联调

如果环境已经准备完成，可以在仓库根目录执行：

```bash
conda activate smart
python dev.py
```

仅启动后端：

```bash
conda activate smart
python dev.py --backend-only
```

### 4.4 ASR 边缘服务

```bash
cd edge_compute
docker-compose up -d
docker-compose down
```

## 5. 配置约定

后端配置文件位于：

- `backend/.env`

环境变量变更约定：

- 以后凡是新增、修改、删除环境变量，默认同时检查并同步 `prod` 和 `test` 两套环境配置
- 如当前仓库只改到其中一套，需明确补齐另一套，不能只改一个环境后直接结束
- 提交结果时要说明本次哪些环境变量已经同步到 `prod/test`
- 如果仓库当前只看到单套本地 env 文件，也要在结果里明确说明另一套环境需同步的同名配置项，不能省略说明

关键环境变量包括：

- `DATABASE_URL`
- `SECRET_KEY`
- `ACCESS_TOKEN_EXPIRE_MINUTES`
- `WECOM_CORP_ID`
- `WECOM_AGENT_ID`
- `WECOM_CORP_SECRET`
- `COZE_API_KEY`
- `COZE_MASTER_BOT_ID`
- `COZE_SPACE_ID`
- `TONGYI_API_KEY`
- `ASR_API_URL`
- `KNOWLEDGE_SYNC_INTERVAL_MINUTES`

## 6. 本项目工作补充

通用工作原则、安全权限、依赖变更和验证清单已迁移到：

- `agentic-codex/rules/agent-work-principles.md`
- `agentic-codex/rules/security-and-permissions.md`
- `agentic-codex/rules/dependency-management.md`
- `agentic-codex/workflows/validation-checklist.md`

本节仅保留 Smart CS AI 项目专属补充。

### 6.1 数据库与架构边界

- 路由层负责请求与响应，不要塞过多业务逻辑
- 业务逻辑优先放入 `services/`
- 数据结构变更要同步检查 `models`、`schemas`、接口返回、前端类型
- 涉及数据库结构调整时，必须补 Alembic migration
- 以后凡是新增数据表、删除数据表、重命名数据表，或新增/删除关键字段时，必须同步更新 `doc/database_dictionary.md`；如表注释或字段注释有变更，也要同步刷新该文档与对应 SQL
- 数据库执行规则：新增表、新增字段这类不影响线上现有业务运行的变更，可以直接执行数据库操作；修改字段、删除字段、删除表、重命名表或可能影响线上读写兼容性的变更，必须先询问用户是否直接执行，得到明确肯定后再执行。

### 6.2 权限与安全优先

本项目带有完整 RBAC，不要随意绕过权限控制。涉及以下内容时必须额外审查：

- 登录与鉴权
- 用户、角色、权限、部门
- 管理后台路由
- 文件上传
- 外部 API 调用与密钥使用

## 7. 按模块改动时的注意事项

### 7.1 Backend

- 默认使用异步风格，保持与现有代码一致
- 数据库访问遵循现有 session 依赖注入方式
- 新接口优先接入 `backend/app/api/api.py` 的统一路由体系
- 修改模型后，检查关联查询、序列化与迁移

### 7.2 Frontend

- 前端通用组件规则优先读取 `agentic-codex/rules/frontend-common-components.md`
- 前端 UI 统一使用 `shadcn/ui` 体系，新增界面或组件时优先复用 `shadcn/ui` 组件，不要随手写一套自定义 UI
- 新增或改造前端页面时，必须优先遵循 `doc/frontend-common-components.md` 与 `doc/frontend-style-guide.md`
- 业务工具页、列表页、后台管理页主内容宽度默认必须自适应撑满可用区域，使用 `w-full` 加响应式内边距；不要默认使用 `max-w-7xl`、`max-w-6xl` 等固定最大宽度导致宽屏右侧留空。阅读型详情、表单向导等确需限制行长的页面除外。
- 凡是列表筛选后的统计、分组、去重、合并、汇总、排行、导出、图表口径等数据处理，默认放到后端接口或 service 执行，页面只负责展示和轻量格式化，不在页面里重复算口径
- 优先复用已有页面结构、组件模式和 API 封装
- 优先组合 `Button`、`Card`、`Dialog`、`Sheet`、`Table`、`Form` 等现成组件，而不是自己拼低质量基础样式
- 样式优先使用语义化 token 和组件变体，不要在业务页面里到处散落硬编码颜色、圆角、阴影、间距规则
- 页面内新增下拉框、排序筛选默认统一复用“自控部助手-任务大厅”现有列表筛选下拉框样式与交互，优先使用仓库里的公共筛选下拉组件实现；默认要求保持：可输入、可清空、可展开收起、支持下拉筛选；除非明确要求使用原生 `select` 或已有组件约束，否则后续不再单独强调该规范
- 页面内新增日期选择、日期筛选默认统一复用公共组件 `frontend/src/components/ui/date-picker-input.tsx`，不要继续在业务页内手写原生 `input[type=date]` 或重复造一套日期弹层逻辑
- 表格列宽超过屏幕、需要横向滚动时，默认参照给排水助手列表的横向滚动条处理：首屏底部常驻一个与原生横向滚动条观感一致的固定滚动条；当表格原生横向滚动条已经出现在当前视口内时，固定滚动条自动隐藏
- 以后凡是业务页面出现列宽超过屏幕、需要横向滚动条的表格，必须优先调用通用组件 `frontend/src/components/ui/fixed-bottom-scrollbar.tsx`（或其底层 hook `use-fixed-bottom-scrollbar.ts`），统一使用“原表格横向滚动条不可见时显示底部浮动横向滚动条、原生横向滚动条进入视口后自动隐藏”的规则；不要在业务页重复手写浮动层或只放普通 `overflow-x-auto`
- 上述两类交互优先抽成公共组件或 hook 复用，不要在业务页重复手写同一套状态、事件和样式
- 如果现有仓库已经封装了 `shadcn/ui` 二次组件，优先复用，不要重复造轮子
- 页面改动要同步检查路由、权限可见性和类型定义
- 不要为了小改动引入新的状态管理方案
- 构建前至少确保 `cd frontend && ./build.sh` 能通过；前端验证默认直接执行 `build.sh`，不要单独执行 `npm run build`

### 7.3 Edge Compute

- 该服务是独立部署单元，不要把业务逻辑硬塞进去
- 仅承载音频转写相关能力
- 涉及模型、容器、端口、挂载目录变更时，要明确记录影响

## 8. 提交前最低验证要求

如果改动了代码，至少做与改动范围匹配的验证，不要嘴上说“理论可行”。

推荐最低标准：

- 改后端：至少运行相关服务启动或相关测试；如涉及迁移，检查迁移是否可执行
- 改前端：至少运行 `cd frontend && ./build.sh`
- 改接口联动：检查前后端字段是否一致
- 改配置：说明新增环境变量及默认值策略

常用验证命令：

```bash
cd frontend && ./build.sh
cd backend && source .venv/bin/activate && uvicorn app.main:app --reload --port 8000
cd backend && source .venv/bin/activate && alembic upgrade head
```

## 9. 通用禁令与工作流程

- 通用禁止行为见：`agentic-codex/rules/agent-work-principles.md`
- 通用 Bug 修复流程见：`agentic-codex/workflows/bug-fixing-flow.md`
- 通用上下文记忆流程见：`agentic-codex/workflows/context-memory.md`

## 10. 给 Agent 的直接建议

这个仓库不是玩具项目，权限、数据、外部集成都不少。别自作聪明乱抽象，别拿“看起来更优雅”当理由乱改结构。先保证正确，再谈漂亮；先保证可验证，再谈速度。

## 11. 依赖变更通知规则

- 通用依赖变更规则见：`agentic-codex/rules/dependency-management.md`
- 本项目本机 Python 导入校验默认使用 `conda activate smart` 后执行相关文件的 `py_compile` / import 级校验；生产环境校验必须使用 `/anaconda3/envs/smart/bin/python -m py_compile ...`
- 本项目前端 TypeScript / JavaScript 校验默认执行 `cd frontend && ./build.sh`，不要单独执行 `npm run build`

## 12. 本机环境约定

- 本机允许安装并使用 `nvm` 管理 Node.js 环境
- 遇到前端构建、`npm`、`vite`、`typescript` 等工具链异常时，默认可先用 `nvm` 校正 Node 版本
- 如果因为切换 `nvm` 版本解决了问题，结果里要顺带说明当前使用的 Node 版本
- 本机 Python 默认使用 `conda activate smart`，后续执行 Python 相关命令优先基于该环境
- 生产环境 Python 默认使用 `/anaconda3/envs/smart/bin/python`，依赖安装用 `/anaconda3/envs/smart/bin/python -m pip`，不要使用 `conda` 或裸 `pip`
