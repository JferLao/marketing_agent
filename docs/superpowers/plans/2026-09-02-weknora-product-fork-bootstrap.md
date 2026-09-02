# WeKnora Product Fork Bootstrap Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Import WeKnora v0.7.2 as the product baseline, run its default core stack locally, record runtime evidence, and push the verified baseline to `JferLao/marketing_agent`.

**Architecture:** Preserve the official WeKnora Git history and keep Tencent's repository as the `upstream` remote. The product `main` branch starts at the official v0.7.2 tag; runtime configuration stays in an ignored local `.env`, and only the default Docker Compose services are started.

**Tech Stack:** Go, Vue, Python DocReader, Docker Compose, ParadeDB/PostgreSQL, Redis

**Spec:** `docs/superpowers/specs/2026-09-02-weknora-product-fork-bootstrap-design.md`

## Global Constraints

- The baseline commit is `3d5d8bfcdfeeea266b292b71cea616847af28d0f` tagged `v0.7.2`.
- Do not implement marketing features in this plan.
- Do not enable optional Docker Compose profiles.
- Do not delete existing containers, images, volumes, or caches.
- Never commit `.env` or local credentials.

---

### Task 1: Establish the product Fork

**Files:**
- Create: `docs/superpowers/specs/2026-09-02-weknora-product-fork-bootstrap-design.md`
- Create: `docs/superpowers/plans/2026-09-02-weknora-product-fork-bootstrap.md`

**Interfaces:**
- Consumes: official WeKnora tag `v0.7.2`
- Produces: product `main`, `origin`, and `upstream` Git references

- [x] **Step 1: Clone the official tag with full Git history**

```bash
git clone --origin upstream --branch v0.7.2 https://github.com/Tencent/WeKnora.git .
```

- [x] **Step 2: Create the product branch**

```bash
git switch -c main
```

- [x] **Step 3: Add the product remote**

```bash
git remote add origin https://github.com/JferLao/marketing_agent.git
```

- [x] **Step 4: Verify the repository identity**

```bash
git rev-parse HEAD
git tag --points-at HEAD
git remote -v
git status --short --branch
```

Expected: HEAD is `3d5d8bfcdfeeea266b292b71cea616847af28d0f`, tag is `v0.7.2`, branch is `main`, and both remotes are present.

### Task 2: Configure the local pinned baseline

**Files:**
- Create: `.env` (ignored local runtime configuration)

**Interfaces:**
- Consumes: `.env.example`, `docker-compose.yml`
- Produces: an explicit v0.7.2 Compose configuration

- [x] **Step 1: Create the minimal local configuration**

Create `.env` with the following exact development-only values:

```dotenv
WEKNORA_VERSION=v0.7.2
GIN_MODE=release
TZ=Asia/Shanghai
AUTO_MIGRATE=true
APP_HOST=app
APP_PORT=18080
FRONTEND_PORT=80
DOCREADER_ADDR=docreader:50051
DOCREADER_TRANSPORT=grpc
DB_DRIVER=postgres
DB_HOST=postgres
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=weknora-local-postgres
DB_NAME=WeKnora
STREAM_MANAGER_TYPE=redis
REDIS_ADDR=redis:6379
REDIS_PASSWORD=weknora-local-redis
REDIS_DB=0
STORAGE_TYPE=local
LOCAL_STORAGE_BASE_DIR=/data/files
RETRIEVE_DRIVER=postgres
OLLAMA_OPTIONAL=true
OLLAMA_BASE_URL=http://host.docker.internal:11434
JWT_SECRET=weknora-local-jwt-change-before-production
SYSTEM_AES_KEY=0123456789abcdef0123456789abcdef
```

- [x] **Step 2: Confirm that local secrets are ignored**

```bash
git check-ignore -v .env
git status --short
```

Expected: `.env` is ignored and does not appear in Git status.

- [x] **Step 3: Validate Compose interpolation**

```bash
docker compose config --quiet
docker compose config --images
docker compose config --services
```

Expected: configuration validation succeeds; WeKnora images end with `:v0.7.2`; services are the default core set only.

### Task 3: Pull and start the default core stack

**Files:**
- Read: `docker-compose.yml`
- Read: `.env`

**Interfaces:**
- Consumes: validated Compose configuration
- Produces: running frontend, app, docreader, postgres, and redis services

- [x] **Step 1: Record the pre-start disk state**

```bash
df -h .
docker system df
```

- [x] **Step 2: Pull version-pinned images**

```bash
docker compose pull
```

Expected: the default images download successfully without enabling optional profiles.

- [x] **Step 3: Start the default services**

```bash
docker compose up -d
```

- [x] **Step 4: Wait on service health conditions**

Poll `docker compose ps --format json` until the services with health checks report `healthy`, or until their configured startup windows have expired. Do not use a fixed long sleep.

- [x] **Step 5: Inspect startup failures if any service is unhealthy**

```bash
docker compose ps
docker compose logs --tail=200 app docreader postgres redis frontend
```

Expected: use logs only when the health condition does not converge.

### Task 4: Verify and record the runtime baseline

**Files:**
- Create: `docs/marketing-agent/weknora-v0.7.2-baseline.md`

**Interfaces:**
- Consumes: running default core services
- Produces: reproducible runtime evidence for the product baseline

- [x] **Step 1: Verify the backend health endpoint**

```bash
curl --fail --silent --show-error http://127.0.0.1:18080/health
```

Expected: HTTP request succeeds and returns the WeKnora health payload.

- [x] **Step 2: Verify the Web UI**

```bash
curl --fail --silent --show-error --output /dev/null --write-out '%{http_code}\n' http://127.0.0.1/
```

Expected: `200`.

- [x] **Step 3: Verify the frontend-to-backend proxy**

```bash
curl --fail --silent --show-error --output /dev/null --write-out '%{http_code}\n' http://127.0.0.1/api/v1/auth/config
```

Expected: `200`.

- [x] **Step 4: Record exact evidence**

Write `docs/marketing-agent/weknora-v0.7.2-baseline.md` with the verified commit, image tags, container health, endpoint results, execution date, and any environment limitations. Do not claim model chat, retrieval, document ingestion, or optional services were verified unless those flows were actually executed.

- [x] **Step 5: Validate the documentation diff**

```bash
git diff --check
git status --short --branch
```

- [x] **Step 6: Commit the product baseline documentation**

```bash
git add docs/superpowers/specs/2026-09-02-weknora-product-fork-bootstrap-design.md docs/superpowers/plans/2026-09-02-weknora-product-fork-bootstrap.md docs/marketing-agent/weknora-v0.7.2-baseline.md
git commit -m "docs: establish WeKnora v0.7.2 product baseline"
```

- [x] **Step 7: Push the verified baseline**

```bash
git push -u origin main
```

Expected: `origin/main` points at the verified documentation commit whose parent history contains official WeKnora v0.7.2.
