# Agent 文档归档目录

本目录用于集中存放跨项目可复用的 Agent 相关 Markdown 文档。

## 归档规则

- 业务项目中临时沉淀的 Agent 规则、排查记录、上下文说明、执行约定，优先迁移到本目录或本仓库对应目录。
- 业务项目根目录只保留必要入口文件，例如 `AGENTS.md`；通用内容不要散落在业务仓库。
- 后续新增的 Agent 相关 `.md` 文件，默认落到 `agentic-codex/agent-docs/`，再按主题沉淀到 `rules/`、`workflows/`、`architectures/` 或 `projects/`。
- 业务项目若需要忽略本地脚本和文档，推荐在 `.gitignore` 中加入 `start.sh`、`stop.sh`、`build.sh`、`*.md`；必须保留的文档用白名单显式排除。

## 推荐子目录

- `common/`：跨项目通用 Agent 约定。
- `project-notes/`：业务项目迁移出来的上下文和排查记录。
- `templates/`：可复制到新项目的 `AGENTS.md`、历史记录模板。

