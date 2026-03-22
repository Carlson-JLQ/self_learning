# 总体架构

## 目标

平台面向企业知识库问答场景，解决“文档可接入、知识可索引、查询可回答、过程可治理、结果可审计”的落地问题。设计重点不是单次问答效果，而是可部署、可管控、可扩展、可运维。

## 分层架构

### 1. 接入层

- Web 管理控制台：租户管理、知识库管理、文档管理、任务监控、模型配置、运营报表
- 开放 API：对外提供知识库、文档、索引任务、问答、会话、反馈、评测接口
- 统一认证接入：OIDC/SAML、企业 SSO、API Token、服务到服务认证

### 2. 平台服务层

- 租户与组织服务：租户生命周期、配额、资源隔离
- 用户与权限服务：用户、角色、资源、动作、数据作用域
- 知识库服务：知识库定义、访问控制、检索配置、索引策略
- 文档接入服务：上传、解析任务、状态跟踪、版本管理
- 任务编排服务：导入、切片、Embedding、索引构建、重建、重试
- 审计与运营服务：审计日志、调用记录、反馈闭环、指标聚合
- 配置中心：模型路由、检索参数、限流策略、特性开关

### 3. RAG 核心层

- 文档解析：PDF、Word、Markdown、HTML、纯文本、OCR 增强
- 文本清洗与切片：结构化提取、标题层级保留、表格和附件处理
- Embedding 生成：批处理、幂等重试、版本化
- 索引构建：向量索引、倒排索引、元数据索引
- 混合检索：关键词召回、向量召回、布尔过滤、权限过滤
- 重排：Cross Encoder 或兼容服务
- Prompt 编排：系统模板、问答模板、引用约束、拒答策略
- 答案生成：上下文拼装、会话记忆裁剪、引用返回
- 结果溯源：引用片段、文档来源、召回记录、模型调用记录

### 4. 能力适配层

- `LLMProvider`
- `EmbeddingProvider`
- `RerankProvider`
- `VectorStoreProvider`
- `DocumentParser`
- 搜索引擎、对象存储、OCR、内容抽取器等适配器

### 5. 基础设施层

- PostgreSQL：平台主数据、配置、审计、任务元数据
- Redis：缓存、会话态、限流、短期任务状态
- Kafka/RabbitMQ：异步任务与事件总线
- 对象存储：原始文档、解析中间产物、导出文件
- Elasticsearch/OpenSearch：全文索引与过滤查询
- Milvus/pgvector：向量索引与语义召回
- Prometheus/Grafana/OpenTelemetry：监控、告警与链路追踪

## 核心业务流

### 文档导入

1. 用户上传文档或配置外部数据源
2. 平台写入对象存储并创建 `Document`、`IndexJob`
3. 任务服务异步完成解析、清洗、切片、Embedding、索引构建
4. 知识库状态切换为可检索，并记录索引版本

### 问答请求

1. 用户提交问题和目标知识库
2. 鉴权与权限过滤生成可访问资源范围
3. 混合召回产生候选片段
4. 重排服务输出高相关上下文
5. Prompt 编排服务生成约束化提示词
6. LLM 生成答案并附带引用
7. 平台写入审计、会话与反馈基线数据

## 关键设计决策

- 一期默认采用混合检索，不提供“仅大模型直答”的主路径
- 文档处理和索引生命周期全部异步化，避免 API 长事务
- 多租户、RBAC、知识库级和文档级权限从一期落地
- 所有模型与存储能力通过 Provider 抽象暴露，不允许业务层直接耦合具体厂商 SDK
- 答案必须带引用和请求追踪信息，便于合规与人工校验

## 逻辑拓扑

```text
Console / Open API / SSO
            |
        API Gateway
            |
  Platform Services (tenant, kb, document, auth, audit)
            |
  Orchestrator / Task Queue / Event Bus
            |
RAG Core Services (parser, chunker, embed, retrieve, rerank, generate)
            |
Providers (LLM, vector DB, ES, OCR, object storage)
            |
Infra (PostgreSQL, Redis, MQ, Object Storage, ES, Milvus, Observability)
```
