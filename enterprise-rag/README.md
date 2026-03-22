# Enterprise RAG Platform

面向企业私有化/内网部署的 RAG 平台设计文档集。该方案以 `Python + FastAPI` 为主技术栈，强调多租户、权限隔离、审计、安全治理、可插拔模型与向量能力，目标是支撑企业知识库问答平台的一期落地。

## 一期目标

- 提供统一的知识库接入、解析、索引、检索、问答闭环
- 支持多租户、RBAC、知识库级和文档级权限控制
- 支持关键词检索、向量检索、重排的混合检索链路
- 支持私有化部署、模型与向量库可替换、全链路审计
- 提供管理控制台、开放 API、评测与运维观测基础能力

## 设计原则

- 平台优先：先定义稳定的平台接口和治理能力，再扩展具体模型和数据源
- 异步优先：文档导入、解析、切片、索引构建、重建全部异步化
- 可插拔优先：模型、Embedding、重排、向量库、OCR、搜索引擎均通过 Provider 抽象接入
- 安全优先：多租户、鉴权、审计、配额、脱敏不是后补能力
- 可观测优先：所有关键链路必须具备日志、指标、Tracing 和审计事件

## 文档导航

- [总体架构](docs/architecture/overview.md)
- [模块设计](docs/architecture/modules.md)
- [数据模型](docs/architecture/data-model.md)
- [API 设计](docs/architecture/api-design.md)
- [项目脚手架](docs/architecture/project-structure.md)
- [私有化部署](docs/deployment/private-cloud.md)
- [本地联调](docs/deployment/local-bootstrap.md)
- [安全与合规](docs/security/compliance.md)
- [运维与可观测性](docs/operations/observability.md)
- [测试与验收](docs/testing/acceptance.md)
- [演进路线图](docs/roadmap.md)

## 推荐技术栈

- API 与平台服务：FastAPI、Pydantic、SQLAlchemy
- 任务编排：Celery 或自研任务编排层，消息中间件使用 Kafka/RabbitMQ
- 关系数据库：PostgreSQL
- 缓存：Redis
- 对象存储：S3 兼容对象存储或 MinIO
- 搜索引擎：Elasticsearch/OpenSearch
- 向量检索：Milvus 或 pgvector
- 观测：Prometheus、Grafana、OpenTelemetry、Loki/ELK

## 当前脚手架

- `src/enterprise_rag/main.py` 提供 FastAPI 应用入口和健康检查
- `src/enterprise_rag/api/` 提供 `v1` 路由骨架
- `src/enterprise_rag/services/` 提供应用服务和仓储编排
- `src/enterprise_rag/providers/` 提供模型、向量库、解析器抽象和 Stub 实现
- `src/enterprise_rag/db/`、`src/enterprise_rag/repositories/` 提供 SQLAlchemy ORM 与 Repository 基座
     - `src/enterprise_rag/core/auth.py`、`src/enterprise_rag/api/middleware.py` 提供租户上下文和基础 RBAC 依赖
- `src/enterprise_rag/providers/factory.py` 提供可配置 Provider 工厂，支持 `stub`、`memory`、`deterministic`、`openai_compatible` 等后端
- `src/enterprise_rag/tasks/` 提供 `local` 与 `celery` 两种任务调度模式，文档上传后会创建索引任务并推进状态
- `src/enterprise_rag/api/v1/routes/audit.py` 提供审计事件查询接口，关键写操作和问答请求会落审计日志
- `src/enterprise_rag/api/v1/routes/traces.py` 提供文档切片和检索日志查询接口，支持问答溯源与质量排查
- `src/enterprise_rag/api/v1/routes/operations.py` 提供租户级运营指标接口
- `src/enterprise_rag/api/v1/routes/evaluations.py` 已支持评测样本、评测执行结果查询
- 知识库已支持检索策略和 Prompt 模板持久化配置
- 全局异常处理已统一为标准错误响应格式，并会记录失败审计事件
- `alembic/` 与 `alembic.ini` 提供数据库迁移骨架
- `tests/test_app.py` 提供接口与权限最小测试

## 启动方式

在安装 Python 3.11+ 后执行：

```bash
pip install -e .[dev]
uvicorn enterprise_rag.main:app --reload --app-dir src
pytest
```

常用脚本：

```bash
powershell -ExecutionPolicy Bypass -File scripts/bootstrap.ps1
powershell -ExecutionPolicy Bypass -File scripts/run-api.ps1
powershell -ExecutionPolicy Bypass -File scripts/run-worker.ps1
powershell -ExecutionPolicy Bypass -File scripts/test.ps1
make install
make test
```

健康检查：

```bash
GET /healthz
GET /readyz
GET /api/v1/system/capabilities
GET /api/v1/system/config-checks
```

统一错误响应格式：

```json
{
  "code": "INDEX_JOB_NOT_FOUND",
  "message": "Index job not found: job_missing",
  "request_id": "req_xxx",
  "data": null
}
```

数据库迁移命令：

```bash
alembic upgrade head
alembic revision -m "describe change"
```

Provider 相关环境变量示例：

```bash
LLM_BACKEND=stub
EMBEDDING_BACKEND=deterministic
RERANK_BACKEND=passthrough
VECTOR_STORE_BACKEND=memory
PARSER_BACKEND=simple
```

任务调度相关环境变量示例：

```bash
TASK_DISPATCHER_BACKEND=local
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/1
CELERY_TASK_ALWAYS_EAGER=false
```

Celery Worker 启动示例：

```bash
celery -A enterprise_rag.tasks.celery_tasks.celery_app worker -l info
```

如需切换到 PostgreSQL `pgvector`：

```bash
DATABASE_URL=postgresql+asyncpg://user:password@host:5432/enterprise_rag
VECTOR_STORE_BACKEND=pgvector
VECTOR_DIMENSION=8
PGVECTOR_TABLE=rag_chunks
PGVECTOR_AUTO_CREATE=true
```

说明：
- `pgvector` 适配器当前会在首次访问时尝试执行 `CREATE EXTENSION IF NOT EXISTS vector`
- 若开启 `PGVECTOR_AUTO_CREATE=true`，会自动创建向量表和基础索引

## 非目标

- 一期不建设通用 Agent 平台或复杂工作流市场
- 一期不覆盖复杂 BI、低代码、跨企业数据交换平台能力
- 本轮产物是开发与落地文档，不包含可运行代码实现

powershell -ExecutionPolicy ByPass -c {
  $env:UV_INSTALL_DIR = "D:\uv";
  irm https://astral.sh/uv/install.ps1 | iex
}
