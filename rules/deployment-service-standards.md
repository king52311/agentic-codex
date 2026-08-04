# 部署服务拆分通用规范

## 适用范围

- 后端应用通过 Docker Compose、Kubernetes、systemd、Supervisor 或类似进程管理方式部署时适用。
- 适用于 Web API、定时任务、队列消费者、事件监听、浏览器自动化、文件处理等多进程职责。

## 基本原则

- 一个运行服务只承担一类长期职责，避免同一容器或同一进程同时跑 API、cron、worker。
- API 服务只负责 HTTP/RPC 请求处理，不默认启动 APScheduler、Celery beat、队列消费者或长轮询监听。
- cron/scheduler 服务独立运行计划任务，不对外暴露 API 端口。
- worker 服务独立运行队列消费、事件监听、流式回调、浏览器自动化等后台任务。
- 多个服务可以复用同一个镜像、代码目录、环境变量和网络，但必须用不同 `command`、服务名和容器名表达职责。

## Docker Compose 约定

- Compose 中按职责拆分服务，例如：
  - `api`：启动 `uvicorn`、`gunicorn`、`fastapi`、`express` 等 Web 服务。
  - `cron` 或 `scheduler`：启动项目的计划任务入口，例如 `python -m app.scheduler_main`。
  - `worker`、`stream-worker`、`queue-worker`：启动后台消费者或事件监听脚本。
- API 服务显式关闭计划任务，例如 `SCHEDULER_ENABLED=false`。
- cron/scheduler 服务显式开启计划任务，例如 `SCHEDULER_ENABLED=true`。
- 只有需要外部访问的 API 服务映射端口；cron/worker 默认不映射端口。
- 所有服务应设置明确的 `restart` 策略、`working_dir`、`command` 和必要的时区配置。
- 涉及本地时间的服务应统一配置 `TZ`，生产环境可挂载 `/etc/localtime` 和 `/etc/timezone`。
- 服务名、容器名、网络 alias 应表达职责，避免多个职责都叫 `backend`。

## 启动命令约定

- 服务拆分后，文档和脚本中的启动命令必须同步改为按服务启动，不能只保留旧的单体后端命令。
- Docker Compose 推荐命令：
  - 首次或全量启动：`docker compose up -d api cron`
  - 只启动 API：`docker compose up -d api`
  - 只启动计划任务：`docker compose up -d cron`
  - 只重启 API：`docker compose restart api`
  - 只重启计划任务：`docker compose restart cron`
  - 查看 API 日志：`docker compose logs -f api`
  - 查看计划任务日志：`docker compose logs -f cron`
- 如果存在 worker 服务，应按职责提供对应命令，例如 `docker compose up -d queue-worker`、`docker compose logs -f stream-worker`。
- 发布脚本不要默认 `docker compose restart` 或 `docker compose up -d` 重启所有服务；除非本次变更确实影响全部服务。
- README、AGENTS、部署文档、CI/CD 配置和运维脚本中出现旧启动命令时，要同步补充拆分后的服务级命令。
- 本地开发命令也应区分 API 与调度器：
  - API 示例：`uvicorn app.main:app --reload --port 8000`
  - cron 示例：`python -m app.scheduler_main`
  - API 本地开发默认不启动 cron，除非明确设置调度开关。

## Docker 日志命令约定

- 服务拆分后，排查日志必须先确认目标职责，不要只看旧后端容器日志。
- 使用 Compose 服务名查看日志：
  - API 请求日志：`docker compose logs -f api`
  - 计划任务日志：`docker compose logs -f cron`
  - 指定最近行数：`docker compose logs --tail=200 -f api`
  - 指定时间之后：`docker compose logs --since=30m -f cron`
- 使用容器名查看日志时，命令也要区分容器：
  - API 请求日志：`docker logs -f <api-container-name>`
  - 计划任务日志：`docker logs -f <cron-container-name>`
  - 指定最近行数：`docker logs --tail=200 -f <api-container-name>`
  - 指定时间之后：`docker logs --since=30m -f <cron-container-name>`
- 如果项目约定容器名，例如 `zhidao-api`、`zhidao-cron`，文档中应明确写成：
  - `docker logs -f zhidao-api`
  - `docker logs -f zhidao-cron`
- API 报错、HTTP 访问、鉴权、接口 500 优先看 API 日志；定时同步、自动推送、队列消费失败优先看 cron/worker 日志。
- worker 类服务同理使用独立服务名或容器名，例如 `docker compose logs -f stream-worker` 或 `docker logs -f <worker-container-name>`。
- 发布后验证日志时，应分别确认 API 启动成功、cron 注册任务成功、worker 连接外部服务成功。

## 应用代码约定

- 应用入口应支持通过环境变量控制后台调度是否启动。
- API 进程启动时，如果当前环境是 API 服务，应跳过 scheduler/cron 初始化。
- cron/scheduler 应有独立入口文件，负责启动调度器并保持进程存活。
- 管理后台如果需要控制计划任务，应考虑 API 进程和 cron 进程分离后的通信方式，例如数据库状态、Redis 命令队列、消息队列或专用控制接口。
- 手动触发任务时，API 不应假设本进程持有 scheduler 实例；应能把执行请求转交给 cron/worker。

## 发布与运维

- 更新 API 代码只需重启 API 服务时，不应误重启 cron/worker，除非任务代码或共享依赖确实变更。
- 更新计划任务逻辑后，应明确重启 cron/scheduler 服务。
- 发布说明中要写清楚本次影响哪些服务：`api`、`cron`、`worker` 或全部。
- 排查线上问题时，先确认请求日志和任务日志来自哪个服务，不要把 API 容器日志当成全部后端日志。
- 计划任务只允许一个主调度实例运行；如果横向扩容 cron/scheduler，必须先实现分布式锁、任务租约或单主机制。

## 验证清单

- API 服务启动后不会自动执行周期任务。
- cron/scheduler 服务启动后可以正常注册并执行计划任务。
- worker 服务异常退出后能按预期重启。
- API 与 cron/worker 使用同一套必要配置，但端口暴露、命令和职责不同。
- 管理后台的任务暂停、恢复、手动执行等功能，在 API/cron 分离后仍能生效。
