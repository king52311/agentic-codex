# Agentic Codex

> Centralized Engineering Standards, Execution Rules, Architecture Patterns, and Best Practices for AI Agents.
> 专为 AI Agent 打造的代码规范、行为约束、协同协议与业务架构方案知识库。

## 仓库定位

本仓库用于沉淀跨项目共享内容：

- Agent 通用行为规则
- 代码规范
- Git / Bug 修复等工作流
- 可复用业务架构方案

业务项目仍应保留自己的 `AGENTS.md`，用于描述项目专属规则。

## 目录结构

```text
.
├── rules/                  # 语言与框架规范
│   ├── agent-work-principles.md
│   ├── security-and-permissions.md
│   ├── dependency-management.md
│   ├── java-backend-standards.md
│   ├── python-standards.md
│   ├── typescript-standards.md
│   ├── frontend-common-components.md
│   ├── database-design.md
│   └── project-script-standards.md
├── instructions/           # Agent 角色约束
│   ├── code-reviewer.md
│   ├── architect-agent.md
│   └── refactoring-agent.md
├── workflows/              # 任务执行 SOP
│   ├── context-memory.md
│   ├── validation-checklist.md
│   ├── git-commit-rules.md
│   ├── maven-multi-module-debug.md
│   ├── integration-self-test-scripts.md
│   ├── release-shortcut-scripts.md
│   └── bug-fixing-flow.md
├── architectures/          # 可复用业务架构方案
│   ├── README.md
│   └── smart-cs-ai-reference-architecture.md
└── AGENT.md                # Agent 通用入口
```

## 推荐接入方式

业务项目根目录继续保留自己的 `AGENTS.md`，并在其中引用本仓库：

```md
通用 Agent 规则优先参考：agentic-codex/AGENT.md
通用代码规范参考：agentic-codex/rules/
通用工作流参考：agentic-codex/workflows/
通用架构方案参考：agentic-codex/architectures/
```

项目规则与用户当前要求优先级高于本仓库通用规则。

Agent 进入业务项目后，如项目内存在 `agentic-codex/AGENT.md`，应先读取它作为通用规则入口，再读取项目自己的规则和上下文文件。

## 架构方案沉淀方式

后续遇到可复用业务架构时，优先新增到 `architectures/`。

每个方案建议包含：

- 适用场景
- 功能特点
- 推荐目录结构
- 数据同步或业务流
- 权限方案
- 前端页面组织
- 验证方式
