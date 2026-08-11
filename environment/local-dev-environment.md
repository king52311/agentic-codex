# 本机开发环境配置

本文件记录当前机器的通用开发环境。Agent 进入任意业务项目后，除读取项目自己的 `AGENTS.md`、`PROJECT_HISTORY.md` 外，也应先读取本文件，避免误判本机缺少运行环境。

## Python

- 本机有 `conda`。
- 遇到 Python 版本、依赖、虚拟环境问题时，先检查 conda 环境，不要只看系统 Python。
- 常用检查命令：

```bash
conda --version
conda info --envs
python --version
```

- 需要指定环境运行时，优先使用：

```bash
conda run -n <env_name> python --version
conda run -n <env_name> python -m pip --version
```

## Node.js

- 本机有 `nvm`。
- 遇到 Node.js、npm、pnpm、前端构建或脚本检查问题时，先加载 nvm 并检查当前 Node 版本，不要只看系统 Node。
- 常用检查命令：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
nvm --version
nvm ls
node --version
npm --version
```

- 如果项目存在 `.nvmrc`，优先执行：

```bash
nvm use
```

## 使用原则

- 项目内明确声明的运行环境优先于本文件。
- 本文件只记录本机通用能力，不替代项目依赖文件、Dockerfile、compose 或启动脚本。
- 运行验证失败时，应优先确认是否需要激活 conda 环境或 nvm Node 版本。
