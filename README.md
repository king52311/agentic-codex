# Agentic Codex 🤖📜

> Centralized Engineering Standards, Execution Rules, and Best Practices for AI Agents.  
> 专为 AI Agent（智能体）打造的代码规范、行为约束与开发原则知识库。

---

## 🎯 仓库定位

本仓库旨在为团队中的 AI Agent（如 Cursor, Windsurf, AutoGPT, Claude Code 等）及开发者提供统一的**指令规范（System Instructions）、代码规范（Coding Standards）与协同协议**。

通过模块化的规则定义，确保 Agent 生成的代码具有高一致性、可维护性与安全性。

---

## 📂 目录结构

```text
.
├── rules/                  # 语言与框架规范 (Coding Standards)
│   ├── python-standards.md
│   ├── typescript-standards.md
│   └── database-design.md
├── instructions/           # Agent 系统提示词与角色约束 (System Prompts)
│   ├── code-reviewer.md
│   ├── architect-agent.md
│   └── refactoring-agent.md
├── workflows/              # 任务执行 SOP (Standard Operating Procedures)
│   ├── git-commit-rules.md
│   └── bug-fixing-flow.md
└── AGENT.md                # Agent 入口性引导文件（AI 优先读取）
