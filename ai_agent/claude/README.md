# Claude Code Agent (Docker)

在 Ubuntu 22.04 容器中运行 [Claude Code CLI](https://github.com/anthropics/claude-code)，可作为 multi-agent 平台的「agent 员工」镜像使用。这是一个**面向开发的较完整镜像**（非极简），容器内可直接 `apt install` 安装依赖与桌面类 `.deb` 应用。

镜像的全部构建过程通过 [docker-compose.yaml](docker-compose.yaml) 的 `build.dockerfile_inline` 内联，**无需单独的 Dockerfile**。

## 镜像内容

- **基础系统**：Ubuntu 22.04 (jammy)，时区 `Asia/Shanghai`，UTF-8 locale
  - 选 22.04 而非 24.04，是因为 24.04 的 [t64 改名](#关于桌面应用-deb-的安装)会让 `libgtk-3-0` / `libatspi2.0-0` 等旧包名「装不上」，导致很多预编译 `.deb` 失败
- **开发工具**：`git` `curl` `wget` `jq` `ripgrep` `fd-find` `vim` `unzip` `zip` `build-essential` `python3` / `pip` / `venv` 等
- **桌面运行时依赖**：预装 `libgtk-3-0` `libnss3` `libnotify4` `libxss1` `libxtst6` `libatspi2.0-0` `libsecret-1-0` `libappindicator3-1` `libgbm1` `libasound2` `xdg-utils` 等，方便安装 Electron/桌面类 `.deb`
- **Node.js 20 LTS**（Claude Code 需要 Node 18+）
- **Claude Code CLI**：`@anthropic-ai/claude-code` 全局安装
- **保留 apt 索引**：未清理 `/var/lib/apt/lists`，容器内 `sudo apt install` 无需先 `apt-get update`
- **非 root 用户**：`agent`（拥有免密 `sudo`），多数 agent 平台要求非 root 运行
- 默认工作目录 `/workspace`

## 前置要求

- Docker Compose **v2.17+**（`dockerfile_inline` 需要此版本）
- 一份鉴权信息，二选一：
  - **官方直连**：`ANTHROPIC_API_KEY`
  - **第三方网关**：`ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_BASE_URL`

## 快速开始

> 推荐先用模板生成 `.env`（含密钥，已被 `.gitignore` 忽略，不会入库），变量会被 Compose 自动读取：
> ```bash
> cp .env.example .env   # 然后填入鉴权信息，启动时即可省略命令行注入
> ```

```bash
# 1. 构建并后台启动（容器会保持运行，便于 exec 进入或被平台调度）

# 方式 ①：官方直连
ANTHROPIC_API_KEY=sk-xxx docker compose up -d --build

# 方式 ②：第三方网关
ANTHROPIC_AUTH_TOKEN=sk-xxx ANTHROPIC_BASE_URL=http://your-gateway:4000 docker compose up -d --build


# 2. 进入 Claude Code 交互界面
docker compose exec claude-agent claude

# 3. 或进入容器 shell
docker compose exec claude-agent bash
```

> PowerShell 下设置环境变量：
> ```powershell
> # 官方直连
> $env:ANTHROPIC_API_KEY="sk-xxx"; docker compose up -d --build
> # 第三方网关
> $env:ANTHROPIC_AUTH_TOKEN="sk-xxx"; $env:ANTHROPIC_BASE_URL="http://your-gateway:4000"; docker compose up -d --build
> ```

> ⚠️ **优先级**：若挂载的 `claude-config/settings.json` 里写了 `env`（例如 `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_BASE_URL`），它会**覆盖**上面通过命令行注入的同名变量。排查"key 设了却没生效"时优先检查该文件。

## 环境变量

| 变量 | 说明 |
| --- | --- |
| `ANTHROPIC_API_KEY` | 官方直连方式的密钥，通过宿主机环境变量注入，未设置则为空 |
| `ANTHROPIC_AUTH_TOKEN` | 第三方网关方式的令牌，与 `ANTHROPIC_BASE_URL` 配合使用 |
| `ANTHROPIC_BASE_URL` | 第三方网关地址，例如 `http://your-gateway:4000` |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 已设为 `1`，关闭非必要的遥测/流量 |
| `IS_SANDBOX` | 已设为 `1`，标记容器为沙箱环境 |
| `WORKSPACE_DIR` | 可选，工作区在宿主机上的路径，未设置则默认 `./workspace` |
| `CLAUDE_CONFIG_DIR` | 可选，配置/登录态在宿主机上的路径，未设置则默认 `./claude-config` |

### apt 镜像源

构建阶段已**固定使用清华镜像源**（国内访问官方 `archive.ubuntu.com` 易 502/超时），替换基于 Ubuntu 22.04 的经典格式文件 `/etc/apt/sources.list`，无需任何额外参数。如需改回官方源或换其他镜像，直接修改 [docker-compose.yaml](docker-compose.yaml) 中对应的 `sed` 行即可。

## 关于桌面应用 .deb 的安装

容器内可直接安装 Electron/桌面类预编译 `.deb`（依赖已预装）：

```bash
docker compose exec claude-agent bash
sudo apt install ./multica-desktop-0.1.0-linux-amd64.deb   # 或 sudo dpkg -i ... && sudo apt -f install
```

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
