# Containerization

## Overview

The repository includes a production-oriented container image and example Kubernetes manifests.
The image is shared by API and Celery worker workloads so deployment environments only need to manage one build artifact.

## Docker Image

Build the image from the repository root:

```bash
docker build -t enterprise-rag-platform:latest .
```

Run the API container:

```bash
docker run --rm -p 8000:8000 --env-file .env.production.example enterprise-rag-platform:latest
```

Run the worker container:

```bash
docker run --rm --env-file .env.production.example enterprise-rag-platform:latest \
  python -m celery -A enterprise_rag.tasks.celery_tasks.celery_app worker -l info
```

## Docker Compose Stack

For a full local stack with PostgreSQL, Redis, API, and worker:

```bash
docker compose -f docker-compose.app.yml up -d --build
```

Stop the stack with:

```bash
docker compose -f docker-compose.app.yml down
```

## Kubernetes Manifests

Example manifests are under `deploy/k8s/`:

- `namespace.yaml`
- `configmap.yaml`
- `secret.example.yaml`
- `api-deployment.yaml`
- `api-service.yaml`
- `worker-deployment.yaml`

These manifests assume external PostgreSQL and Redis endpoints are already reachable in the cluster network.

## Deployment Notes

- Keep `ENABLE_DOCS=false` in production.
- Run `alembic upgrade head` before scaling the API deployment.
- Replace `OPENAI_API_KEY` and database credentials via `Secret`, not `ConfigMap`.
- If the platform does not use OpenAI-compatible endpoints, only the provider-related environment variables need to change.
