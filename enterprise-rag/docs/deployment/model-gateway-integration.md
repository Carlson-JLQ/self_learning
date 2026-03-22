# Model Gateway Integration

## Goal

This guide covers the minimal path to connect the platform to a real OpenAI-compatible model gateway and verify that:

- LLM calls succeed
- Embedding calls succeed
- provider diagnostics reflect the real backend
- the RAG query path uses the configured embedding model

## 1. Prepare Environment

Start from:

- `.env.openai-compatible.example`

Required values:

```bash
LLM_BACKEND=openai_compatible
EMBEDDING_BACKEND=openai_compatible
OPENAI_BASE_URL=https://your-model-gateway.example.com/v1
OPENAI_API_KEY=replace_me
OPENAI_CHAT_MODEL=your-chat-model
OPENAI_EMBEDDING_MODEL=your-embedding-model
```

If the gateway is protected by enterprise SSO, also configure either:

- `AUTH_MODE=oidc` with `OIDC_JWKS_URL`
- or `AUTH_MODE=trusted_proxy` behind an identity gateway

## 2. Start Platform Dependencies

For local validation:

```bash
docker compose -f docker-compose.app.yml up -d --build
```

Run migrations:

```bash
rag_env\Scripts\python.exe -m alembic upgrade head
```

## 3. Check Provider Endpoints

Use the smoke test script:

```bash
rag_env\Scripts\python.exe scripts/provider-smoke-test.py --base-url http://localhost:8000
```

The script validates:

- `/readyz`
- `/api/v1/system/info`
- `/api/v1/providers/settings`
- `/api/v1/providers/diagnostics`
- `/api/v1/providers/probe`

## 4. Expected Results

For a healthy model gateway integration:

- provider settings show `openai_compatible`
- provider diagnostics show configured base URL and model names
- provider probe returns:
  - `llm.status=ok`
  - `embedding.status=ok`

## 5. Knowledge Base Strategy

Set a knowledge-base strategy so the query path uses the intended model parameters:

```json
{
  "retrieval_profile": "precision",
  "retrieval_top_k": 5,
  "rerank_enabled": true,
  "prompt_template": "Answer the question using the context only.\nQuestion: {question}\nContext: {context}",
  "llm_model": "your-chat-model",
  "embedding_model": "your-embedding-model",
  "temperature": 0.1,
  "max_tokens": 512
}
```

## 6. Acceptance Checklist

- API starts with the target environment variables.
- `/api/v1/providers/probe` succeeds.
- A document can be indexed without embedding errors.
- `/api/v1/query/answer` succeeds with real citations.
- Retrieval and audit logs are written for the request.

## 7. Common Failures

- `401` on API routes: authentication mode does not match the request path or token type.
- `OIDC_CONFIG_ERROR`: missing `OIDC_JWT_SECRET` for `HS256` or missing `OIDC_JWKS_URL` for `RS256`.
- provider probe `llm.status=error`: invalid base URL, API key, or chat model.
- provider probe `embedding.status=error`: invalid embedding model, gateway path mismatch, or dimension mismatch downstream.
