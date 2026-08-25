# Project Rules

- 主项目为 `omsapi`，相关文档、排查记录、数据库迭代脚本优先落到本项目。
- 每次进入项目或恢复上下文时，先读取 `PROJECT_HISTORY.md`。
- 每次完成重要改动、提交、发布或排查后，把结果追加写入 `PROJECT_HISTORY.md`。
- 每次编写、迁移、更新与 Agent 上下文相关的 Markdown 文档时，必须写入 `agentic-codex` 项目对应目录，不再写到业务项目根目录；写完后自动在 `agentic-codex` 中执行 `git pull --rebase`、`git add`、中文 `git commit`、`git push`。
- 每次涉及数据库结构或初始化数据变更时，必须对照 `db/baseline/djwms-lite.sql`，并同步新增或更新 `db/migrations/` 下的迭代 SQL。
- 生产发版时只执行 `db/migrations/` 下尚未执行的迭代 SQL，不直接改基线 SQL。
- 页面设计规则：按钮、搜索框、卡片标题和正文文字都不要贴边沿，容器必须保留足够内边距，避免出现元素紧贴卡片或页面边缘的布局。
- 页面列表规则：凡是列表存在单条删除功能，默认都需要增加多选批量删除功能，批量删除按钮仅在选中数据后显示。
