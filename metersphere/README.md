# MeterSphere

用于本地体验和开发的 MeterSphere 社区版单容器示例，使用官方 all-in-one 镜像。`docker-compose.yaml` 直接运行镜像，无需自定义 Dockerfile；`.env.example` 提供镜像版本和宿主机端口配置。

当前固定版本为 `v3.6.10-lts`，官方 Docker Hub 标签提供 `linux/amd64` 和 `linux/arm64` 镜像。不使用浮动 `latest`，避免重建时意外升级。

## 环境要求

- Docker Engine 和 Docker Compose v2；Windows 使用 Docker Desktop 的 Linux 容器模式。
- 官方在线安装文档列出的服务器参考要求：2 核 CPU、4 GB 内存、200 GB 磁盘空间，并能稳定访问互联网。Docker Desktop 还需给 Linux 虚拟机分配足够资源。
- 此示例用于可信本地开发。生产环境请使用官方推荐的离线部署方案，并单独配置备份、TLS、访问控制和监控。

## 快速开始

在本目录执行：

```bash
cp .env.example .env
```

PowerShell：

```powershell
Copy-Item .env.example .env
```

按需修改 `.env`，然后运行：

```bash
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=200 metersphere
```

首次启动需要初始化内置组件和数据，容器显示 Running 不代表应用已经就绪。等待日志中的启动结果，并通过浏览器实际访问确认。

- 默认地址：<http://127.0.0.1:8081>；修改端口后使用相应地址。
- 官方文档列出的初始用户名：`admin`，初始密码：`metersphere`。
- **首次登录后立即在账户设置中修改为独立的高强度密码，再导入项目、凭据或测试数据。** 这不是可用于生产的默认凭据。

本示例未声明初始化密码环境变量：没有验证该镜像支持通过环境变量覆盖初始密码。不要套用 RabbitMQ 的密码参数，或认为编辑 `.env` 能修改 MeterSphere 已有账号。

## 配置说明

| 项目 | 说明 |
| --- | --- |
| `METERSPHERE_IMAGE` | 官方镜像及明确版本，默认 `metersphere/metersphere-ce-allinone:v3.6.10-lts` |
| `METERSPHERE_HTTP_PORT` | 宿主机 HTTP 端口，默认 `8081`；容器端口保持 `8081` |
| `metersphere_data` | Compose 管理的命名卷，挂载 `/opt/metersphere/data` |
| `metersphere_conf` | 应用及内置组件配置，挂载 `/opt/metersphere/conf` |
| `metersphere_logs` | 日志，挂载 `/opt/metersphere/logs` |

`.env` 是 Compose 参数文件，不是 MeterSphere 应用配置文件，也不会作为环境变量整体注入容器。实测此版本将数据、配置和日志分别放在上述三个目录，因此分别声明命名卷，避免镜像默认的匿名配置卷和日志卷在 `down` 后无法自动复用。首次创建空卷时 Docker 会复制镜像中该目录的初始文件。

采用命名卷避免旧 Windows 命令中相对路径挂载的差异。它不依赖宿主机 `./data` 目录，因此已有 `./data` 内容也不会自动迁移进卷；如需迁移，先备份并按对应版本的官方流程处理。

## 持久化、停止与升级

```bash
# 停止并移除容器，保留数据卷
docker compose down

# 使用同一个 Compose 项目重新启动
docker compose up -d
```

不要随意更改 Compose 项目名或卷名，否则可能创建不同的数据卷。`docker compose down -v` 会删除此项目的数据卷及项目、账号、测试数据，只能用于确认可以清空的测试环境，不能用作修复安装或重置密码的常规步骤。

升级前先备份数据和配置并验证恢复方案，核对目标版本的升级说明，再修改 `.env` 中的明确版本并执行 `docker compose pull`、`docker compose up -d`。已有配置卷不会自动获取新镜像中的默认配置，需按发行说明合并必要的配置变更。不要直接将旧版镜像指向升级后的数据卷以尝试回滚。固定版本需要定期主动检查安全更新。

## 安全边界

- 仅向宿主机 `127.0.0.1` 发布端口，避免在默认密码尚未修改时暴露给局域网或公网。本机其他用户及同一 Docker 网络内的容器仍可能访问服务。
- HTTP 未启用 TLS。远程使用需另行设计 TLS、网络访问控制和认证策略，不要简单将绑定改为 `0.0.0.0`。
- 测试平台可以发起网络请求、执行测试内容；仅导入可信脚本和项目，不要授予不可信用户访问权限，并限制其所在主机的内网访问范围。
- 数据卷可能含 API 凭据、请求记录和报告，应限制 Docker 管理权限并保护备份。`.env` 已被仓库忽略，不要把实际凭据或数据加入版本控制。
- 不挂载 Docker socket，不启用 privileged 或 host 网络，也不向宿主机单独开放镜像内的数据库端口。

## 排查安装问题

原分支仅记录“目前安装报错”，没有保留错误日志，无法确定当时原因。本例整理了官方部署入口，但不宣称已复现或修复该历史报错。

1. 镜像拉取失败：检查 Docker Hub 网络、代理和 DNS，确认 `.env` 中的版本存在；不要通过关闭证书校验解决问题。
2. 端口占用：修改 `METERSPHERE_HTTP_PORT`，例如 `18081`，重新执行 `docker compose up -d`。
3. 容器退出或反复重启：检查 `docker compose logs --tail=200 metersphere`，核对内存、磁盘和 Docker Desktop 资源配置。
4. 容器运行但网页不可用：等待初始化完成，查看应用日志，确认访问端口正确。此示例未配置未经验证的 HTTP 健康检查端点。
5. 共享日志前删除令牌、密码、内部地址和业务数据；不要为排错直接删除数据卷。

## 官方参考

- [在线安装与环境要求](https://metersphere.io/docs/v3.x/installation/online_installation/)
- [生产环境推荐的离线安装](https://metersphere.io/docs/v3.x/installation/offline_installation/)
- [官方镜像及标签](https://hub.docker.com/r/metersphere/metersphere-ce-allinone/tags)
- [项目源码及发行说明](https://github.com/metersphere/metersphere)
