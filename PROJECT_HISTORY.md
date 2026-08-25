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
# 2026-08-25

- 按 Agentic Codex 归档规则新增 `agent-docs/project-notes/cbys-omsapi/` 独立目录。
- 从 `omsapi` 根目录迁移说明类 Agent 回顾文档：`API_SCHEDULED_TASKS.md`、`COLLECT_DATA_PROJECT.md`、`PROJECT_RUNNING_LOGIC.md`、`DOCUSERDOCUMENT_PRODUCT_DIFF.md`。
- `omsapi` 继续保留 `AGENTS.md`、`PROJECT_HISTORY.md`、`README.md` 作为业务项目必要入口。
- 新增全局 Agent 文档落地规则：所有项目的 Agent 相关 Markdown 统一写入 `agentic-codex` 对应目录，不写入业务项目根目录；写完必须在 `agentic-codex` 中先 `git pull --rebase`，再 `git add`、中文提交并 `git push`。
