# Agentic Codex 通用入口

本文件是跨项目共享的 Agent 规则入口。业务项目仍应保留自己的 `AGENTS.md`，项目规则优先级高于本仓库通用规则。

## 使用方式

在业务项目根目录 `AGENTS.md` 中引用本仓库：

```md
通用 Agent 规则优先参考：agentic-codex/AGENT.md
通用代码规范参考：agentic-codex/rules/
通用工作流参考：agentic-codex/workflows/
通用架构方案参考：agentic-codex/architectures/
```

Agent 进入业务项目后，如项目内存在 `agentic-codex/AGENT.md`，应把它作为通用规则入口读取；再读取当前项目自己的 `AGENTS.md`、上下文和历史文件。

## 优先级

1. 用户当前明确要求
2. 当前业务项目 `AGENTS.md`
3. 当前业务项目上下文文件，如 `CURRENT_CONTEXT.md`、`PROJECT_HISTORY.md`
4. 本仓库通用规则与架构方案
5. Agent 默认能力与通用工程经验

若规则冲突，以上层规则为准。

## 通用工作原则

- 先读现有实现，再决定改动位置。
- 优先小步修改，避免无关重构。
- 优先复用项目已有组件、工具函数、接口风格和验证命令。
- 涉及权限、认证、数据删除、生产配置、密钥、迁移时，必须额外谨慎。
- 改完要做与影响范围匹配的验证。
- 重要改动要留下可回顾记录，便于后续 Agent 恢复上下文。

## 规则索引

- `rules/python-standards.md`：Python 通用规范
- `rules/java-backend-standards.md`：Java 后端通用规范
- `rules/typescript-standards.md`：TypeScript / 前端通用规范
- `rules/frontend-common-components.md`：前端通用组件使用规范
- `rules/client-application-standards.md`：客户端应用创建通用规范
- `rules/database-design.md`：数据库设计与迁移规范
- `rules/project-script-standards.md`：项目脚本通用规范
- `rules/agent-work-principles.md`：Agent 通用工作原则
- `rules/security-and-permissions.md`：安全与权限规范
- `rules/dependency-management.md`：依赖变更规范
- `instructions/code-reviewer.md`：代码审查角色约束
- `instructions/architect-agent.md`：架构分析角色约束
- `instructions/refactoring-agent.md`：重构角色约束
- `workflows/git-commit-rules.md`：Git 提交流程
- `workflows/bug-fixing-flow.md`：Bug 修复流程
- `workflows/context-memory.md`：上下文记忆流程
- `workflows/validation-checklist.md`：验证清单
- `workflows/maven-multi-module-debug.md`：Maven 多模块本地联调方法
- `workflows/integration-self-test-scripts.md`：集成自测脚本方法
- `workflows/release-shortcut-scripts.md`：快捷发版脚本推荐方案
- `architectures/`：可复用业务架构方案库
- `architectures/microservice-parent-reference-architecture.md`：微服务 Parent 工程参考架构
- `architectures/client-app-scaffold-generation-architecture.md`：客户端应用生成框架参考架构
