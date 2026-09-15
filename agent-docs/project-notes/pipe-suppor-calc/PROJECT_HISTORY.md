# pipe-suppor-calc 项目历史

## 2026-09-15

- 修复 `Dockerfile` 在 Python 3.13 slim 镜像中读取不存在的 `/etc/apt/sources.list` 问题，兼容 `.list` 和 `.sources` 配置文件。
- 调整 Debian 软件源替换为阿里云镜像。
- 将 Pillow 安装约束调整为 `>=11.0.0`，并改用 PyPI 安装，解决部分清华镜像无法提供 Python 3.13 Pillow 包的问题。
- Docker 实际构建验证未完成：本机 Docker daemon 未运行；`docker compose config` 校验通过。
