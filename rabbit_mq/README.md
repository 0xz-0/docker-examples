# RabbitMQ

用于本地开发的单节点 RabbitMQ 示例，使用官方 `rabbitmq:4.3.7-management` 镜像，包含管理界面。通过 `docker-compose.yaml` 运行，无需另外构建 Dockerfile。

## 快速开始

在本目录执行，需要 Docker Engine 和 Docker Compose v2。

```bash
cp .env.example .env
cp rabbitmq_example.conf rabbitmq.conf
chmod 600 .env
```

PowerShell：

```powershell
Copy-Item .env.example .env
Copy-Item rabbitmq_example.conf rabbitmq.conf
```

编辑 `.env`，填写独立的高强度随机密码，不要使用 `guest` 用户或生产凭据。模板不提供可直接使用的密码，密码为空时 Compose 会拒绝解析。Windows 请限制 `.env` 的文件 ACL。密码包含 `$` 或 `#` 时用单引号包裹整个值，避免插值或注释解析；不要把真实密码输入命令历史或提交入库。

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs --tail=100 rabbitmq
```

必须先复制 `rabbitmq.conf`，缺失时挂载会报错，不会自动生成同名目录。默认账号与 vhost 仅在空数据目录首次启动时初始化。示例账号具有管理权限，仅用于本地管理；业务应用应另建用户并仅授予所需 vhost 权限。

## 连接与配置

| 项目 | 地址或说明 |
| --- | --- |
| AMQP | `127.0.0.1:5672`，账号密码来自 `.env`，默认 vhost 为 `app` |
| 管理界面 | <http://127.0.0.1:15672>，使用同一初始化账号登录 |
| 同一 Compose 网络内的客户端 | 主机名 `rabbitmq`、端口 `5672`；不能使用客户端容器自己的 `localhost` |
| `.env.example` | 初始化账号、密码和 vhost 模板，复制成 `.env` 后编辑 |
| `rabbitmq_example.conf` | 服务配置模板，复制成 `rabbitmq.conf` 后编辑 |
| 数据卷 | Compose 管理的 `rabbitmq_data` 命名卷，挂载到 `/var/lib/rabbitmq` |

配置文件只读挂载到 `/etc/rabbitmq/conf.d/20-custom.conf`。端口冲突时，仅修改 Compose 映射中间的宿主机端口，例如 `127.0.0.1:5673:5672`。

健康检查确认 RabbitMQ 应用运行且无本地资源告警，不等于完成客户端认证或消息收发测试。空闲磁盘低于样例的 1GB 阈值时会触发告警，需释放空间或按实际环境调整阈值。

## 持久化与维护

固定 hostname 保持节点名称稳定。容器重建和 `docker compose down` 会保留命名卷；不要随意更改 hostname、Compose 项目名或数据卷名称，否则可能使用新的节点目录或不同数据卷。

```bash
# 修改 rabbitmq.conf 后重启
docker compose restart rabbitmq

# 停止服务，保留数据
docker compose down
```

持久化已有用户不会因修改 `.env` 而自动修改密码。轮换密码请通过管理界面或官方用户管理工具更新已有账号，再同步客户端配置。不要为了更换密码删除数据卷。

`docker compose down -v` 会删除该项目的命名卷和其中的用户、队列及消息，仅在确认可丢弃全部数据时使用。持久卷也不保证所有消息重启后保留：应用仍需使用 durable 队列、持久化消息和 publisher confirms，并制定备份策略。

镜像固定到补丁版本以避免跨版本漂移；定期检查官方安全公告与支持周期，按升级路径更新版本并验证。升级前备份，不要通过降级镜像直接打开新版数据目录。

## 安全边界

- 默认仅向宿主机 `127.0.0.1` 发布 AMQP 和管理端口，不发布集群通信端口。同一 Docker 网络内的容器仍可直接连接，不要将不可信容器加入该网络。
- AMQP 和 HTTP 未启用 TLS，仅用于可信本地开发。远程使用必须另行配置 TLS、网络访问控制及最小权限用户，不要直接改为绑定 `0.0.0.0` 后公开服务。
- 密码以环境变量传入，Docker 管理员及容器内进程可以读取；`.env`、实际配置和数据卷均需保护。不要公开完整 `docker compose config` 或 `docker inspect` 输出。
- 保留官方入口：先修正数据目录属主，再以 `rabbitmq` 用户运行。没有配置 privileged、宿主机网络或 Docker socket。不要直接添加 `cap_drop: ALL`，以免破坏初始化所需的 chown/setuid。
- `.env` 与实际 `rabbitmq.conf` 已被 Git 忽略，样例文件只放无敏感信息的配置；不要使用 `git add -f` 提交真实配置。

## 官方参考

- [Docker 官方镜像](https://hub.docker.com/_/rabbitmq)
- [RabbitMQ 配置](https://www.rabbitmq.com/docs/configure)
- [用户与权限](https://www.rabbitmq.com/docs/access-control)
- [版本支持周期](https://www.rabbitmq.com/release-information)
- [升级说明](https://www.rabbitmq.com/docs/upgrade)
