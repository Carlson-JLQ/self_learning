# API Client Assets

## Overview

The project can export two client-facing artifacts from the live FastAPI schema:

- `artifacts/openapi.json`: OpenAPI schema for Swagger-compatible tooling.
- `artifacts/postman_collection.json`: Postman collection for backend and frontend integration testing.

## Commands

Use the project virtual environment and run:

```bash
python scripts/export_openapi.py
python scripts/generate_postman_collection.py
```

Or use the convenience targets:

```bash
make openapi
make postman
```

## Notes

- The export uses the same FastAPI application factory as runtime.
- Generated files are intended for local validation, API governance, and CI artifact publishing.
- Postman generation includes default tenant headers so role-based routes are easier to test interactively.
- Provider introspection endpoints such as `/api/v1/providers/settings` and `/api/v1/providers/diagnostics` are included in the generated artifacts.
