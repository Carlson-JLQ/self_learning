# 本地联调启动说明

## 目标

本说明用于在本地启动企业级 RAG 平台的最小依赖，便于开发、接口联调与基础验证。

## 依赖组件

- PostgreSQL + `pgvector`
- Redis
- Python 运行时与项目虚拟环境

## 使用 `docker-compose`

在项目根目录执行：

```bash
docker compose up -d
```

启动后默认端口：

- PostgreSQL: `5432`
- Redis: `6379`

## 推荐环境变量

```bash
DATABASE_URL=postgresql+asyncpg://enterprise_rag:enterprise_rag@localhost:5432/enterprise_rag
TASK_DISPATCHER_BACKEND=celery
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1
VECTOR_STORE_BACKEND=pgvector
PGVECTOR_AUTO_CREATE=true
VECTOR_DIMENSION=8
EMBEDDING_BACKEND=deterministic
PARSER_BACKEND=simple
RERANK_BACKEND=passthrough
```

## 启动顺序

1. 启动基础依赖：`docker compose up -d`
2. 执行数据库迁移：`alembic upgrade head`
3. 启动 API：

```bash
uvicorn enterprise_rag.main:app --reload --app-dir src
```

4. 启动 Celery Worker：

```bash
celery -A enterprise_rag.tasks.celery_tasks.celery_app worker -l info
```

## 联调检查

- `GET /healthz`
- `GET /readyz`
- `GET /api/v1/system/capabilities`
- `GET /api/v1/system/config-checks`

## 注意事项

- 本地联调默认使用示例级 Provider，不等于生产级模型接入
- 若使用真实 OpenAI 兼容网关，需要单独设置 `OPENAI_API_KEY` 与对应模型名
- 若 Celery 不可用，可先将 `TASK_DISPATCHER_BACKEND=local` 进行同步调试
