# 快捷发版脚本推荐方案

## 目标

项目根目录可以提供快捷发版脚本，用来降低重复命令成本，并统一构建、校验、打包、发布描述生成等步骤。

脚本应服务于“稳定复用”，不是把一次性命令堆进仓库。

## 推荐目录结构

简单项目：

```text
.
├── build.sh
├── release.sh
└── start.sh
```

复杂项目：

```text
.
├── build.sh              # 根入口，转发到 scripts/build.sh
├── release.sh            # 根入口，转发到 scripts/release.sh
└── scripts/
    ├── build.sh
    ├── release.sh
    ├── deploy.sh
    ├── self_test.sh
    └── lib/
        ├── common.sh
        └── env.sh
```

## 根入口脚本思路

根目录入口脚本保持短小：

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
exec "$ROOT_DIR/scripts/build.sh" "$@"
```

好处：

- 用户在项目根目录能快速找到入口。
- 复杂逻辑集中在 `scripts/`。
- 后续多个脚本可复用 `scripts/lib/common.sh`。

## build.sh 推荐能力

适合承载：

- 构建单模块。
- 构建全部模块。
- 跳过测试或运行测试。
- 输出产物路径。

推荐参数：

```text
./build.sh all
./build.sh <module>
./build.sh <module> --skip-tests
./build.sh <module> --profile prod
```

Maven 项目可封装：

```bash
mvn -pl "$MODULE" -am clean package -DskipTests
```

多模块项目建议维护模块别名，避免用户记长路径。

## release.sh 推荐能力

适合承载：

- 检查 Git 工作区状态。
- 读取或校验版本号。
- 执行构建和最小自测。
- 生成版本更新描述。
- 调用版本后台接口。
- 输出部署提醒。

推荐参数：

```text
./release.sh 1.2.3
./release.sh 1.2.3 --env test
./release.sh 1.2.3 --dry-run
```

建议流程：

1. 校验版本号格式。
2. 检查当前分支和未提交变更。
3. 汇总 Git 变更或最近提交。
4. 执行构建和必要自测。
5. 生成 release note。
6. dry-run 时只打印请求内容。
7. 非 dry-run 时调用发布接口或提示下一步部署。

## deploy.sh 推荐能力

适合承载：

- 上传 jar、静态资源或镜像。
- 重启目标服务。
- 查询部署后健康状态。

高风险要求：

- 生产环境必须确认目标环境、版本号和服务名。
- 支持 `--dry-run`。
- 不把生产凭证写入仓库。

## self_test.sh 推荐能力

适合承载：

- API 冒烟测试。
- MQ 消费链路测试。
- 数据库最终状态查询。
- 定时任务联动验证。

自测脚本应支持环境变量覆盖，并输出可读断言结果。

## common.sh 推荐内容

可复用方法建议放在 `scripts/lib/common.sh`：

```bash
log_info() { printf '[INFO] %s\n' "$*"; }
log_error() { printf '[ERROR] %s\n' "$*" >&2; }
die() { log_error "$*"; exit 1; }
require_cmd() { command -v "$1" >/dev/null 2>&1 || die "缺少命令: $1"; }
confirm() {
  local prompt="${1:-确认继续?}"
  read -r -p "$prompt [y/N] " answer
  [ "$answer" = "y" ] || [ "$answer" = "Y" ]
}
```

注意：

- 公共方法只放真正复用的逻辑。
- 不把业务变量藏在公共方法里。
- 公共方法变更后要回归所有引用脚本。

## 创建脚本时的检查清单

- 是否有清晰职责。
- 是否支持 help。
- 是否能从任意目录执行。
- 是否避免个人路径和敏感信息。
- 是否有 dry-run 或确认机制。
- 是否补充使用文档。
- 是否更新 `.gitignore`。
- 是否执行过最小验证。
