# Claude Code Agent (Docker)

在 Ubuntu 22.04 容器中运行 [Claude Code CLI](https://github.com/anthropics/claude-code)，可作为 multi-agent 平台的「agent 员工」镜像使用。这是一个**面向可信项目开发的较完整镜像**（非极简）。系统依赖与桌面类 `.deb` 应用在构建阶段安装，运行时使用无 sudo 权限的普通用户。

镜像的全部构建过程通过 [docker-compose.yaml](docker-compose.yaml) 的 `build.dockerfile_inline` 内联，**无需单独的 Dockerfile**。

## 镜像内容

- **基础系统**：Ubuntu 22.04 (jammy)，时区 `Asia/Shanghai`，UTF-8 locale
  - 选 22.04 而非 24.04，是因为 24.04 的 [t64 改名](#关于桌面应用-deb-的安装)会让 `libgtk-3-0` / `libatspi2.0-0` 等旧包名「装不上」，导致很多预编译 `.deb` 失败
- **开发工具**：`git` `curl` `wget` `jq` `ripgrep` `fd-find` `vim` `unzip` `zip` `build-essential` `python3` / `pip` / `venv` 等
- **桌面运行时依赖**：预装 `libgtk-3-0` `libnss3` `libnotify4` `libxss1` `libxtst6` `libatspi2.0-0` `libsecret-1-0` `libappindicator3-1` `libgbm1` `libasound2` `xdg-utils` 等，方便安装 Electron/桌面类 `.deb`
- **Node.js 24 LTS**：从官方 `node:24-bookworm-slim` 镜像复制 Node 和 npm，不执行远程安装脚本
- **Claude Code CLI**：固定安装 `@anthropic-ai/claude-code@2.1.311`，禁用自动更新；升级通过修改版本、验证和重新构建完成
- **非 root 用户**：`agent`（UID 1000，无 sudo 权限）
- **运行时权限**：保留 Docker 默认 seccomp，设置 `no-new-privileges`，移除全部 Linux capabilities
- 默认工作目录 `/workspace`

## 前置要求

- Docker Compose **v2.17+**（`dockerfile_inline` 需要此版本）
- 一份鉴权信息，二选一：
  - **官方直连**：`ANTHROPIC_API_KEY`
  - **第三方网关**：`ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_BASE_URL`

## 快速开始

> 在本目录用模板生成 `.env`，填写一种鉴权方式；Compose 会自动读取。`.env` 与 `.env.*` 默认被 Git 和构建上下文忽略（保留 `.env.example`），不要强制加入版本控制：
> ```bash
> cp .env.example .env   # 然后填入鉴权信息，启动时即可省略命令行注入
> chmod 600 .env         # Linux/macOS；Windows 请限制文件 ACL 为当前用户可读写
> ```

```bash
# 1. 构建并后台启动（容器会保持运行，便于 exec 进入或被平台调度）

docker compose up -d --build

# 2. 进入 Claude Code 交互界面
docker compose exec claude-agent claude

# 3. 或进入容器 shell
docker compose exec claude-agent bash
```

> PowerShell 下也建议编辑 `.env`，避免将真实令牌写入命令历史：
> ```powershell
> Copy-Item .env.example .env
> # 编辑 .env 并限制其文件权限后启动
> docker compose up -d --build
> ```

远程网关的 `ANTHROPIC_BASE_URL` 使用 HTTPS，例如 `https://gateway.example.com`，以保护令牌和请求内容。HTTP 仅适用于可信隔离的本地链路；不要通过公网或不可信内网明文传输。

> ⚠️ **优先级**：若挂载的 `claude-config/settings.json` 里写了 `env`（例如 `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_BASE_URL`），它会**覆盖**上面通过命令行注入的同名变量。排查"key 设了却没生效"时优先检查该文件。

## 环境变量

| 变量 | 说明 |
| --- | --- |
| `ANTHROPIC_API_KEY` | 官方直连方式的密钥，通过宿主机环境变量注入，未设置则为空 |
| `ANTHROPIC_AUTH_TOKEN` | 第三方网关方式的令牌，与 `ANTHROPIC_BASE_URL` 配合使用 |
| `ANTHROPIC_BASE_URL` | 第三方网关地址，例如 `https://gateway.example.com` |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 已设为 `1`，关闭非必要的遥测/流量 |
| `DISABLE_AUTOUPDATER` | 已设为 `1`，CLI 升级通过重新构建固定版本完成 |
| `WORKSPACE_DIR` | 可选，工作区在宿主机上的路径，未设置则默认 `./workspace` |
| `CLAUDE_CONFIG_DIR` | 可选，配置/登录态在宿主机上的路径，未设置则默认 `./claude-config` |

### apt 镜像源

构建阶段已**固定使用清华镜像源**（国内访问官方 `archive.ubuntu.com` 易 502/超时），替换基于 Ubuntu 22.04 的经典格式文件 `/etc/apt/sources.list`，无需任何额外参数。如需改回官方源或换其他镜像，直接修改 [docker-compose.yaml](docker-compose.yaml) 中对应的 `sed` 行即可。

## 关于桌面应用 .deb 的安装

运行中的 agent 不具备安装系统软件的权限。需要额外系统包时，在内联 Dockerfile 的 `USER agent` **之前**添加构建指令，例如：

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends <package-name> \
    && rm -rf /var/lib/apt/lists/*
```

安装 `.deb` 时，先确认发布者和校验值，再用 `COPY` 和 `RUN apt-get install` 在构建阶段安装。构建过程会以 root 执行软件包安装脚本，只安装可信软件。完成后重新构建镜像。

若 GUI 应用确实与默认 seccomp 不兼容，应先定位所需系统调用，再评估专用的最小权限配置；不要为普通 CLI 使用关闭默认隔离。

> **为什么基础镜像用 22.04 而不是 24.04？**
> Ubuntu 24.04 (noble) 做了 **t64（64 位 time_t）改名**：`libgtk-3-0` → `libgtk-3-0t64`、`libatspi2.0-0` → `libatspi2.0-0t64`，`libappindicator3-1` 被移除。很多按旧版 Ubuntu 打包、依赖里写死旧包名的 `.deb`，在 24.04 上会报 `Depends: libgtk-3-0 but it is not installable`，**即使 `apt-get update` 也装不上**。22.04 (jammy) 仍保留这些旧包名，因此兼容性更好。
>
> 注意：这是无显示的 headless 容器，装上 GUI 应用主要用于打包/依赖校验等；真要运行图形界面需额外配置 X 转发或 `xvfb`。

## 数据卷

| 卷（宿主机路径） | 挂载点 | 作用 |
| --- | --- | --- |
| `${WORKSPACE_DIR:-./workspace}` | `/workspace` | 工作区，宿主机可直接看到 agent 的产出 |
| `${CLAUDE_CONFIG_DIR:-./claude-config}` | `/home/agent/.claude` | 持久化登录态 / 配置，重建容器后无需重新配置 |

默认情况下 `./workspace` 与 `./claude-config` 目录会在首次启动时自动创建于本目录下：把要让 agent 处理的代码放进 `workspace`，登录态与配置则落在 `claude-config`，宿主机可直接查看与备份。

如需把卷挂到其他宿主机目录，通过 `WORKSPACE_DIR` / `CLAUDE_CONFIG_DIR` 覆盖（未设置则回退到上述默认值）：

```bash
# 命令行临时指定
WORKSPACE_DIR=/data/proj CLAUDE_CONFIG_DIR=/data/cfg docker compose up -d --build
```

```powershell
# PowerShell
$env:WORKSPACE_DIR="D:\data\proj"; $env:CLAUDE_CONFIG_DIR="D:\data\cfg"; docker compose up -d --build
```

也可以把这两个变量写进与 compose 文件同目录的 `.env`，Compose 会自动读取：

```dotenv
WORKSPACE_DIR=/data/proj
CLAUDE_CONFIG_DIR=/data/cfg
```

> **⚠️ Linux 宿主机的挂载权限**
> 容器内以非 root 用户 `agent`（`uid=1000`）运行。在 **Linux 宿主机**上，bind mount 的目录若属主不是 `1000`（Docker 自动创建或 root 创建时通常为 `root:root`），容器内写入会报 `Permission denied`（首次执行 `claude` 即失败）。启动前先把目录属主改为 `1000`：
> ```bash
> mkdir -p ./workspace ./claude-config            # 或你用 WORKSPACE_DIR/CLAUDE_CONFIG_DIR 指定的目录
> sudo chown -R 1000:1000 ./workspace ./claude-config
> ```
> Windows（Docker Desktop）和 macOS 由文件共享层自动处理属主，**无需此步**。

## 安全边界与更新

- 本配置不是运行任意恶意代码的安全沙箱。项目脚本与 agent 使用相同身份，可以读取传入的 API 凭据和挂载的登录配置；改用 secret 文件也不能隔离同一身份的读取权限。
- 使用专用、限额、尽可能短期的凭据。不同项目或信任域分别使用工作目录和 `claude-config`，不要复用个人主账号的登录目录。配置目录及其备份同样按敏感信息保护。
- 仅挂载专用项目目录；不要挂载宿主机根目录、整个用户目录、SSH/云凭据目录或 Docker socket。工作区与配置卷可写，容器内修改和删除会作用于宿主机文件；需要只读输入时单独以 `:ro` 挂载。
- 本 Compose 未实施出站网络白名单，默认网络可能访问互联网及宿主机可达的内网。无人值守或处理不可信内容前，在宿主机、专用网关或隔离虚拟机侧限制出站访问，仅放行必要 API 和依赖服务。不要仅依赖提示词、权限弹窗或环境变量作为安全边界。
- 不要默认开启 `--dangerously-skip-permissions`。需要执行不可信代码时，使用额外隔离环境和专门的网络、凭据策略。
- 环境变量可被容器内进程及有 Docker 管理权限的人读取；`docker compose config` 的完整输出也可能包含真实凭据。检查配置时使用 `docker compose config --quiet`，不要公开完整配置或 inspect 输出。
- Ubuntu、Node 基础镜像仍使用可更新标签，以便获取安全更新；CLI 固定版本不代表永远安全。定期检查安全公告，更新 CLI 版本，并执行 `docker compose build --pull --no-cache`，验证后重建容器。需要严格复现构建时，额外固定镜像 digest 并维护更新流程。
- 构建会检查 Node、npm 和 Claude CLI 能否启动，任何检查失败都会使构建失败；这不替代镜像漏洞扫描或运行时安全验证。

## 常用命令

```bash
# 查看运行状态
docker compose ps

# 查看 Claude 版本
docker compose exec claude-agent claude --version

# 停止并移除容器（宿主机的 ./claude-config 配置保留）
docker compose down

# 清空登录态/配置：直接删除宿主机目录
rm -rf ./claude-config
```
