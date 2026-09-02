# WeKnora v0.7.2 产品基线运行记录

## 结论

2026-09-02 23:57 CST，官方 WeKnora `v0.7.2` 默认核心 Compose 栈在本机 Apple Silicon / Docker Desktop 环境启动成功。

## 源码与仓库

- 产品分支：`main`
- 官方 Tag：`v0.7.2`
- 源码提交：`3d5d8bfcdfeeea266b292b71cea616847af28d0f`
- 产品远端：`origin = https://github.com/JferLao/marketing_agent.git`
- 官方远端：`upstream = https://github.com/Tencent/WeKnora.git`

## 运行配置

- Compose Project：`agent`
- WeKnora 镜像标签：`v0.7.2`
- Web UI：`http://127.0.0.1/`
- Backend API：`http://127.0.0.1:18080/`
- 宿主机 `8080` 已由既有 Node 进程占用，因此只把 App 的宿主机端口改为 `18080`；容器内部仍使用 `8080`。
- `.env` 使用本地开发凭证并由 `.gitignore` 排除。
- 未启用任何可选 Compose Profile。

## 服务状态

| 服务 | 镜像 | 验证状态 |
| --- | --- | --- |
| app | `wechatopenai/weknora-app:v0.7.2` | running / healthy |
| frontend | `wechatopenai/weknora-ui:v0.7.2` | running |
| docreader | `wechatopenai/weknora-docreader:v0.7.2` | running / healthy |
| postgres | `paradedb/paradedb:v0.22.2-pg17` | running / healthy |
| redis | `redis:7.0-alpine` | running |

三个 WeKnora 镜像的 OCI revision 均为 `3d5d8bfcdfeeea266b292b71cea616847af28d0f`，与源码基线一致。

## HTTP 验证

| 请求 | 结果 |
| --- | --- |
| `GET http://127.0.0.1:18080/health` | HTTP 200，`{"status":"ok"}` |
| `GET http://127.0.0.1/` | 连续 3 次 HTTP 200 |
| `GET http://127.0.0.1/api/v1/auth/config` | HTTP 200，`{"registration_mode":"self_serve","success":true}` |

受限命令沙箱内访问 Docker Desktop 暴露的宿主机端口会得到 `connection refused`；同一时刻容器保持 running/healthy 且端口映射存在。在获准的宿主机网络环境中重新执行相同请求后，Web、代理与后端均返回 HTTP 200。因此该现象属于验证环境边界，不是 WeKnora 或 Docker Desktop 服务波动。

## 代码验证

在本机执行以下全量 Go 测试，退出码为 `0`：

```bash
env -u LOG_FORMAT \
  CGO_CXXFLAGS="-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1" \
  go test ./...
```

- macOS Command Line Tools 环境需要显式提供 libc++ 头文件路径，否则 `gojieba` 会报 `fatal error: 'algorithm' file not found`。
- 宿主机原有 `LOG_FORMAT=json`，但 WeKnora 将 `LOG_FORMAT` 解释为日志模板而不是格式枚举；测试时临时移除该变量后，先前失败的 `internal/datasource/connector/feishu/wiki` 日志断言及全量测试均通过。
- `docreader/client` 的两项测试在 `localhost:50051` 不可用时按测试逻辑标记为 `SKIP`；默认 Compose 只在容器网络内暴露该端口。

## 发现与限制

- 官方 Git Tag 为 `v0.7.2`，Docker Hub 的实际镜像 Tag 同样带 `v` 前缀。使用 README 示例风格的 `0.7.2` 会得到 `manifest unknown`。
- App 日志显示未挂载 `config/builtin_models.yaml`，因此本次没有验证任何 LLM、Embedding 或 Rerank 模型。
- 本次没有验证注册登录、文档导入、知识检索、RAG 对话、Agent、MCP 或 Skill 执行。
- Neo4j、Langfuse、MinIO、SearXNG、Sandbox 等可选服务未启动。
- 镜像拉取后宿主机可用空间约 12 GiB；没有删除任何已有镜像、容器、卷或缓存。
- 默认 Compose 将 Web `80` 和 API `18080` 发布到所有宿主机接口。本配置仅作为本地开发基线，不是公网生产部署方案。
- 机器上原有的 `WeKnora-langfuse-*-dev` 容器未被修改。

## 常用命令

```bash
docker compose ps
docker compose logs --tail=200 app docreader postgres redis frontend
docker compose up -d
docker compose down
```

`docker compose down` 默认保留命名卷；不得使用 `down -v`，除非明确决定删除本地基线数据。
