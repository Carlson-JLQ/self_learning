# API 设计

## 设计约束

- API 统一使用 `v1` 前缀
- 所有请求携带租户上下文和追踪 ID
- 管理接口与问答接口分域，避免控制面和数据面混用
- 响应统一包含 `code`、`message`、`request_id`、`data`

## 对外 API

### 租户与用户

- `POST /api/v1/tenants`
- `GET /api/v1/tenants/{tenant_id}`
- `POST /api/v1/users`
- `GET /api/v1/users/{user_id}`
- `POST /api/v1/roles`
- `POST /api/v1/permissions/grants`

### 知识库与文档

- `POST /api/v1/knowledge-bases`
- `GET /api/v1/knowledge-bases/{kb_id}`
- `POST /api/v1/knowledge-bases/{kb_id}/documents`
- `GET /api/v1/documents/{document_id}`
- `POST /api/v1/documents/{document_id}/reindex`
- `GET /api/v1/index-jobs/{job_id}`

### 检索与问答

- `POST /api/v1/query/search`
- `POST /api/v1/query/answer`
- `POST /api/v1/sessions`
- `GET /api/v1/sessions/{session_id}`
- `POST /api/v1/query/{query_id}/feedback`

### 评测

- `POST /api/v1/evaluations/datasets`
- `POST /api/v1/evaluations/runs`
- `GET /api/v1/evaluations/runs/{run_id}`

## 核心请求响应

### 问答请求 `POST /api/v1/query/answer`

```json
{
  "knowledge_base_id": "kb_123",
  "question": "请总结合同中的付款条款",
  "session_id": "sess_001",
  "filters": {
    "document_tags": ["contract"],
    "language": "zh-CN"
  },
  "options": {
    "top_k": 8,
    "enable_rerank": true,
    "response_mode": "grounded"
  }
}
```

### 问答响应

```json
{
  "code": "OK",
  "message": "success",
  "request_id": "req_001",
  "data": {
    "query_id": "qry_001",
    "answer": "合同约定在验收后 30 天内付款。",
    "citations": [
      {
        "document_id": "doc_001",
        "chunk_id": "chk_010",
        "quote_text": "乙方提交发票并完成验收后 30 日内支付。",
        "score": 0.93
      }
    ],
    "usage": {
      "prompt_tokens": 1480,
      "completion_tokens": 220
    },
    "trace_id": "trace_001"
  }
}
```

## 内部服务接口

### 文档处理任务

- `DocumentIngestRequested`
- `DocumentParsed`
- `ChunksGenerated`
- `EmbeddingGenerated`
- `IndexBuildCompleted`
- `IndexBuildFailed`

### 检索服务接口

- 输入：`RetrievalRequest`
- 输出：`RetrievalCandidate[]`
- 要求：显式返回权限过滤前后数量、召回通道来源、耗时

### 生成服务接口

- 输入：问题、候选片段、模板配置、会话上下文
- 输出：答案、引用、模型使用量、拒答原因、Trace 信息

### 审计事件接口

- 统一事件结构：事件类型、主体、资源、结果、时间、Trace、扩展载荷

## Provider 抽象

```python
class LLMProvider:
    async def generate(self, prompt, options): ...

class EmbeddingProvider:
    async def embed_texts(self, texts, model): ...

class RerankProvider:
    async def rerank(self, query, documents, top_k): ...

class VectorStoreProvider:
    async def upsert(self, items, namespace): ...
    async def search(self, vector, filters, top_k): ...

class DocumentParser:
    async def parse(self, source_uri, options): ...
```

## 错误码分类

- `AUTH_*`：认证失败、权限不足、租户不可用
- `DOC_*`：文档格式不支持、解析失败、文件损坏
- `INDEX_*`：Embedding 失败、索引构建失败、版本冲突
- `QUERY_*`：知识库不可用、无可用上下文、模型超时
- `PROVIDER_*`：模型服务异常、向量库异常、对象存储异常
- `SYSTEM_*`：依赖不可达、配置错误、内部未处理异常
