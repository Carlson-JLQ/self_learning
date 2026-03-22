# 项目脚手架与目录结构

## 目录设计

```text
enterprise_rag/
  docs/
    api/
  src/
    enterprise_rag/
      api/
      core/
      domain/
      providers/
      repositories/
      schemas/
      services/
      tasks/
      main.py
  tests/
  scripts/
  artifacts/
  pyproject.toml
  .env.example
```

## 分层约束

- `api/` 只处理路由、鉴权入口、请求响应映射，不直接写业务逻辑
- `schemas/` 定义 API 契约，保持与传输层稳定
- `domain/` 放领域模型和跨模块共享类型
- `services/` 承担应用编排逻辑，调用仓储、Provider 和任务层
- `providers/` 只定义外部能力抽象及具体适配器
- `repositories/` 负责数据访问实现；当前脚手架先提供内存实现
- `tasks/` 预留异步任务和事件契约
- `tooling/` 提供 OpenAPI 与 Postman 等交付产物导出能力

## 当前脚手架状态

- 已提供 FastAPI 应用入口和 `v1` 路由骨架
- 已提供租户、知识库、文档、检索问答、会话、反馈、评测接口占位
- 已提供 `LLMProvider`、`EmbeddingProvider`、`RerankProvider`、`VectorStoreProvider`、`DocumentParser` 抽象
- 已提供 SQLAlchemy ORM、Repository 分层、SQLite/PostgreSQL 兼容配置
- 已提供请求上下文中间件和基础 RBAC 依赖
- 已提供 Provider 工厂与适配器骨架，可按配置切换 Stub、本地内存、OpenAI 兼容实现
- 已提供 PostgreSQL `pgvector` 向量存储适配器骨架，可接入真实企业数据库
- 已提供文档索引任务模型、`local/celery` 双调度模式和 Worker，占位异步索引链路
- 已提供审计事件落库与查询接口，覆盖关键写操作和问答请求
- 已提供 Chunk 持久化、QueryRecord 与 RetrievalLog 落库，支持溯源和质量分析
- 已提供知识库级检索策略持久化和运营指标接口
- 已提供评测样本、评测结果和基础质量指标聚合
- 已提供 `/api/v1/system/info`、OpenAPI 导出和 Postman 集合生成能力
- 已提供 Alembic 初始化迁移骨架
- 已提供最小测试，验证服务启动、问答响应和角色控制

## 下一步实施建议

- 将 `repositories/` 从当前最小实现扩展到 PostgreSQL、Redis、对象存储等正式实现
- 在 `providers/` 中将 OpenAI 兼容实现扩展到企业模型网关、Milvus/pgvector、OCR 和搜索引擎适配器
- 为 `tasks/` 接入 Celery 或消息队列消费者，将当前本地 dispatcher 替换为真正异步链路
- 增加认证、权限中间件、审计事件和错误码体系
