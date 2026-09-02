# WeKnora 产品 Fork 基线启动设计

## 目标

以腾讯 WeKnora 官方 `v0.7.2` 为代码基线，在 `JferLao/marketing_agent` 中保留完整 Git 历史和持续同步上游的能力，并在本机启动、验证默认核心服务。

## 已确认决策

- 主技术栈沿用 WeKnora 原生 Go + Vue + Python。
- `origin` 指向 `https://github.com/JferLao/marketing_agent.git`。
- `upstream` 指向 `https://github.com/Tencent/WeKnora.git`。
- 产品 `main` 从官方 Tag `v0.7.2`（提交 `3d5d8bfcdfeeea266b292b71cea616847af28d0f`）创建。
- 首次只运行默认核心服务，不启用 `full`、Neo4j、Langfuse、MinIO 等可选 Profile。
- 本阶段不设计或实现任何营销业务模块。

## 运行拓扑

默认 Compose 启动以下服务：

- `frontend`：Vue Web UI，对外提供本地 Web 入口。
- `app`：Go API、Agent Runtime 和异步任务处理。
- `docreader`：Python gRPC 文档解析服务。
- `postgres`：ParadeDB/PostgreSQL，保存业务数据并提供默认检索能力。
- `redis`：流管理与 Asynq 任务队列。

## 配置策略

- 本地 `.env` 不提交 Git。
- `WEKNORA_VERSION` 固定为 Docker Hub 实际发布标签 `v0.7.2`，禁止用 `latest` 作为本次验收基线。
- 采用本地开发凭证；凭证只用于本机基线运行，不得用于生产。
- 启用自动数据库迁移，使用默认 PostgreSQL、Redis、本地文件存储和 PostgreSQL 检索后端。
- Ollama 保持可选；未安装模型不应阻断平台基础启动。

## 验收标准

1. `git status --short --branch` 显示产品分支为 `main`。
2. `origin` 与 `upstream` 分别指向已确认的产品仓库和官方仓库。
3. `docker compose config --quiet` 成功，展开镜像均使用明确版本而非 WeKnora `latest`。
4. 默认核心服务成功创建并运行，带健康检查的服务进入 `healthy`。
5. `GET http://127.0.0.1:18080/health` 返回 HTTP 200；宿主机 `8080` 已被既有 Node 进程占用，容器内部端口仍保持官方默认 `8080`。
6. `GET http://127.0.0.1/` 返回 HTTP 200。
7. `GET http://127.0.0.1/api/v1/auth/config` 返回 HTTP 200，证明前端反向代理到后端可用。
8. 运行证据记录到 `docs/marketing-agent/weknora-v0.7.2-baseline.md`。
9. 基线说明提交后，`main` 推送到 `JferLao/marketing_agent`。

## 安全与环境边界

- 不删除现有 Docker 容器、镜像、卷或构建缓存。
- 不执行 `docker compose down -v`、`docker system prune` 或其他清理命令。
- 当前宿主机可用磁盘约 20 GiB；若拉取镜像时空间不足，停止并报告，不擅自清理。
- 只管理本仓库默认 Compose 创建的核心服务；机器上既有的 `WeKnora-langfuse-*-dev` 容器不在本次范围内。
- 不把本地开发密钥提交到 Git。

## 上游同步边界

后续升级通过 `git fetch upstream --tags` 获取官方 Tag，在 `upgrade/weknora-vX.Y.Z` 分支合并和验证。只有通过迁移、健康检查和关键契约验证后，升级分支才能进入产品 `main`。
