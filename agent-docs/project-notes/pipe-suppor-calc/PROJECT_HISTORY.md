# pipe-suppor-calc 项目历史

## 2026-09-15

- 修复 `Dockerfile` 在 Python 3.13 slim 镜像中读取不存在的 `/etc/apt/sources.list` 问题，兼容 `.list` 和 `.sources` 配置文件。
- 调整 Debian 软件源替换为阿里云镜像。
- 将 Pillow 安装约束调整为 `>=11.0.0`，并改用 PyPI 安装，解决部分清华镜像无法提供 Python 3.13 Pillow 包的问题。
- Docker 实际构建验证未完成：本机 Docker daemon 未运行；`docker compose config` 校验通过。
- 静态资源和 API 统一迁移到 `/pipe-suppor-calc/` 子路径，后端支持子路径路由，前端请求、下载地址和 Docker 健康检查同步调整。
- 已验证 `/pipe-suppor-calc/` 页面、静态文件、`/pipe-suppor-calc/api/meta` API，Python 编译、JavaScript 语法和 Compose 配置均通过。
