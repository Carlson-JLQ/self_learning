# 数据模型

## 设计原则

- 业务主数据、配置与审计存放在关系数据库
- 原始文件和中间产物进入对象存储
- 向量、全文索引与主数据通过稳定主键关联
- 所有关键实体支持租户字段和审计字段

## 核心实体

### Tenant

- `id`
- `name`
- `status`
- `plan`
- `quota_policy`
- `created_at`
- `updated_at`

### User

- `id`
- `tenant_id`
- `username`
- `display_name`
- `email`
- `status`
- `identity_provider`

### Role

- `id`
- `tenant_id`
- `name`
- `description`

### PermissionGrant

- `id`
- `tenant_id`
- `principal_type`
- `principal_id`
- `resource_type`
- `resource_id`
- `action`
- `scope`

### KnowledgeBase

- `id`
- `tenant_id`
- `name`
- `description`
- `status`
- `retrieval_profile`
- `default_model_profile`
- `visibility`

### Document

- `id`
- `tenant_id`
- `knowledge_base_id`
- `source_type`
- `source_uri`
- `title`
- `language`
- `classification`
- `version`
- `status`
- `checksum`
- `metadata_json`

### Chunk

- `id`
- `tenant_id`
- `knowledge_base_id`
- `document_id`
- `chunk_no`
- `content`
- `token_count`
- `parent_section`
- `metadata_json`
- `embedding_version`

### IndexJob

- `id`
- `tenant_id`
- `knowledge_base_id`
- `document_id`
- `job_type`
- `status`
- `attempt`
- `error_code`
- `error_message`
- `started_at`
- `finished_at`

### ChatSession

- `id`
- `tenant_id`
- `user_id`
- `knowledge_base_id`
- `title`
- `status`
- `last_activity_at`

### QueryRecord

- `id`
- `tenant_id`
- `session_id`
- `user_id`
- `question`
- `retrieval_snapshot`
- `prompt_snapshot`
- `answer_text`
- `latency_ms`
- `trace_id`

### Citation

- `id`
- `query_record_id`
- `document_id`
- `chunk_id`
- `quote_text`
- `score`
- `rank`

### Feedback

- `id`
- `tenant_id`
- `query_record_id`
- `user_id`
- `rating`
- `label`
- `comment`

### AuditEvent

- `id`
- `tenant_id`
- `actor_id`
- `event_type`
- `resource_type`
- `resource_id`
- `result`
- `trace_id`
- `payload_json`
- `created_at`

## 索引与存储映射

- PostgreSQL：`Tenant`、`User`、`Role`、`PermissionGrant`、`KnowledgeBase`、`Document`、`IndexJob`、`ChatSession`、`QueryRecord`、`Feedback`、`AuditEvent`
- 向量库：`Chunk.id`、向量、租户字段、知识库字段、文档字段、过滤元数据
- Elasticsearch/OpenSearch：`Chunk` 文本、标题、标签、语言、权限过滤字段
- 对象存储：原始文档、解析结果、失败样本、导出包

## 关系约束

- `Tenant` 对所有业务实体是一对多
- `KnowledgeBase` 对 `Document`、`Chunk`、`IndexJob`、`ChatSession` 是一对多
- `Document` 对 `Chunk`、`IndexJob` 是一对多
- `QueryRecord` 对 `Citation`、`Feedback` 是一对多

## 版本化建议

- 文档、切片、Embedding、索引配置均保留版本号
- 索引重建不覆盖历史记录，通过“激活版本”切换线上读流量
- Prompt 模板和模型配置变更写入审计事件，支持回放问题排查
