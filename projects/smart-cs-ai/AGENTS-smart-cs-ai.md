<INSTRUCTIONS>
每次回答，过程忽略，直接告诉结果，尽量简短。
发送 `git` 文本时，自动把变更记录落到 `PROJECT_HISTORY.md`，再执行 `git commit` 和 `git push`，提交信息用中文；如有未纳入版本控制的文件，先 `git add`。
发送 `发布版本-*.*.*` 文本时，先总结改动，生成版本更新描述，并自动请求接口更新后台版本数据。
每次进入项目、插件重新加载、会话恢复或上下文压缩后，先检查项目根目录是否存在 `AGENTS.md` 并遵循其中规则。
如果项目根目录存在 `PROJECT_HISTORY.md`，必须优先读取它，用于恢复历史进度。
每次完成重要改动、提交、发布或排查后，应把结果追加写入项目根目录 `PROJECT_HISTORY.md`。
出现 `exceeded retry limit, last status: 429 Too Many Requests` 时，自动重试三次，再停止。
如果项目中有 `agentic-codex`，每当 `agentic-codex` 项目有文件改动，git 自动拉取最新文件，并在写入完成后自动提交。
除删除数据外，表结构更改、新增表、新增数据默认直接执行到两套环境，不要反复询问是否执行。
具备正式环境登录和发版能力时，线上发版必须先在本地完成 Git 提交并推送，再登录服务器进入对应程序目录执行 `git pull`，最后按改动范围重启或重载相关服务并验证；禁止只向服务器复制本地未提交文件作为正式发版。
正式环境涉及前端改动时，服务器执行 `git pull` 后必须进入 `frontend` 目录执行 `./build.sh`，完成 Docker 构建并切换前端产物后再验证；不能只拉取代码。
涉及不属于当前仓库的代码或配置时，必须进入该程序自己的 Git 仓库处理：先拉取最新代码，再在对应仓库修改、验证、提交并推送，随后重启或重载该程序；禁止把其他项目的文件提交到当前仓库。

本地开发环境约定：
- Python 统一使用 Conda 环境 `smart`，执行后端命令优先使用 `conda run -n smart python ...` 或 `/opt/anaconda3/envs/smart/bin/python ...`；不要使用系统 Python 3.9。
- 后端自测统一使用 `conda run -n smart python -m pytest`，语法检查使用 `conda run -n smart python -m compileall`。
- 本机已安装 MySQL 客户端，数据库联查或 SQL 执行优先使用 `mysql` 客户端；项目脚本 `backend/tasks/run_sql_on_envs.py` 可用于按 `env_test`、`env_prod` 执行 SQL。
- 本地数据库交互测试至少执行一次 `SELECT 1`，涉及表结构或配置表时同时核对字段和实际数据。
- `backend/config/env_test` 与 `env_prod` 如出现相同数据库地址或库名，先记录并提示确认，禁止默认认为是隔离环境。
</INSTRUCTIONS>
