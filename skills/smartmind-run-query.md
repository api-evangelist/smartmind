---
name: Run a ThanoSQL query and review its log
description: Authenticate to a ThanoSQL workspace, execute a SQL/ThanoSQL query, and review the query log to confirm state and destination.
api: openapi/smartmind-thanosql-openapi.json
operations:
  - post_query_api_v1_query__post
  - get_query_logs_api_v1_query_log_get
  - get_schemas_api_v1_schema__get
---

# Run a ThanoSQL query and review its log

ThanoSQL is an analytical RDB with a built-in LLM/DL/ML query layer. This skill runs a query
against a workspace and inspects the resulting query log.

## Prerequisites
- A workspace **engine URL** (`THANOSQL_ENGINE_URL`) and **API token** (`THANOSQL_API_TOKEN`),
  both from the Developer tab of your workspace settings.
- Every request must send `Authorization: Bearer <JWT>` (see `authentication/smartmind-authentication.yml`).
- Base path for the data API is `/api/v1` on the engine URL.

## Steps
1. **(Optional) Discover schemas.** Call `get_schemas_api_v1_schema__get`
   (`GET /api/v1/schema/`) to list available schemas before querying.
2. **Execute the query.** Call `post_query_api_v1_query__post`
   (`POST /api/v1/query/`) with your query string. Use `dry_run=true` first to validate
   without executing. A write query (e.g. one that produces a destination table) is an
   `acting`/write operation — see `agentic-access/smartmind-agentic-access.yml`.
3. **Review the log.** Call `get_query_logs_api_v1_query_log_get`
   (`GET /api/v1/query/log`) with `limit`/`offset` (offset pagination; response includes
   `total`). Inspect each `QueryLog`'s `state`, `error_result`, and
   `destination_table_name`/`destination_schema`.

## Rules
- On `422` the request failed validation — read the `detail[]` in the `HTTPValidationError`
  envelope (`errors/smartmind-problem-types.yml`) and fix the offending field.
- There is **no idempotency key**; do not blindly retry a write query. Re-check the query
  log to confirm whether a prior attempt already ran.
