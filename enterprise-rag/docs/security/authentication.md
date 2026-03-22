# Authentication

## Supported Modes

The platform now supports five authentication modes through `AUTH_MODE`:

- `headers`: development mode using `X-Tenant-Id`, `X-User-Id`, and `X-Roles`.
- `api_token`: machine-to-machine access using managed API tokens.
- `oidc`: bearer token validation using HS256 JWT verification.
- `hybrid`: API token first, then bearer token, then header fallback.
- `trusted_proxy`: trust identity headers injected by an upstream SSO gateway.

## API Token Flow

Routes:

- `GET /api/v1/auth/me`
- `POST /api/v1/auth/api-tokens`
- `GET /api/v1/auth/api-tokens`
- `DELETE /api/v1/auth/api-tokens/{token_id}`

Token format:

```text
ragtk_<token_id>.<secret>
```

The plain token is only returned once at creation time. The database stores a hash, not the raw secret.

## OIDC / SSO Flow

Current bearer-token validation supports:

- issuer validation via `OIDC_ISSUER`
- audience or client-id validation via `OIDC_AUDIENCE` / `OIDC_CLIENT_ID`
- tenant, user, and roles mapping from configurable claims
- `HS256` shared-secret validation via `OIDC_JWT_SECRET`
- `RS256` validation via `OIDC_JWKS_URL`

Current trusted-proxy validation supports:

- `X-Authenticated-User`
- `X-Authenticated-Roles`
- `X-Authenticated-Tenant-Id`

This is suitable when an ingress gateway or identity proxy already terminates enterprise SSO.

## Environment Variables

Key variables:

```bash
AUTH_MODE=hybrid
API_TOKEN_HEADER_NAME=X-API-Token
OIDC_ISSUER=https://sso.example.com
OIDC_AUDIENCE=enterprise-rag-platform
OIDC_CLIENT_ID=enterprise-rag-platform
OIDC_JWT_SECRET=replace_me
OIDC_JWKS_URL=https://sso.example.com/.well-known/jwks.json
OIDC_JWKS_CACHE_TTL_SECONDS=300
OIDC_USERNAME_CLAIM=sub
OIDC_ROLES_CLAIM=roles
OIDC_TENANT_CLAIM=tenant_id
SSO_USER_HEADER_NAME=X-Authenticated-User
SSO_ROLES_HEADER_NAME=X-Authenticated-Roles
SSO_TENANT_HEADER_NAME=X-Authenticated-Tenant-Id
```

## Current Scope

- OpenAI-compatible LLM and Embedding providers now support configurable timeout and retry behavior.
- OIDC bearer validation supports both shared-secret `HS256` and `JWKS / RS256`.
- For production IdPs, prefer `RS256` with JWKS and keep `OIDC_JWT_SECRET` empty unless the gateway explicitly uses symmetric signing.
- The platform exposes `GET /api/v1/auth/me` for principal inspection during integration.
