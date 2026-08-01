---
name: Create a ThanoSQL table and load records
description: Create a table in a schema, insert records or upload a CSV/Excel/JSON file, then read the records back.
api: openapi/smartmind-thanosql-openapi.json
operations:
  - create_table_api_v1_table__table_name__post
  - insert_records_api_v1_table__table_name__records_post
  - upload_table_from_csv_api_v1_table__table_name__upload_csv_post
  - get_records_api_v1_table__table_name__records_get
  - get_table_api_v1_table__table_name__get
---

# Create a ThanoSQL table and load records

This skill provisions a table and populates it, then verifies the load.

## Prerequisites
- Workspace engine URL + API token; send `Authorization: Bearer <JWT>` on every request.
- Base path `/api/v1`.

## Steps
1. **Create the table.** Call `create_table_api_v1_table__table_name__post`
   (`POST /api/v1/table/{table_name}`) with the column/constraint definition. Use the
   `if_not_exists` guard to avoid conflicts — there is no idempotency key
   (`conventions/smartmind-conventions.yml`).
2. **Load data**, either:
   - **Row insert:** `insert_records_api_v1_table__table_name__records_post`
     (`POST /api/v1/table/{table_name}/records`), or
   - **File upload:** `upload_table_from_csv_api_v1_table__table_name__upload_csv_post`
     (`POST /api/v1/table/{table_name}/upload/csv`) — sibling operations exist for
     `upload/excel` and `upload/json`. Use `on_conflict`/`overwrite` to control conflict
     behavior.
3. **Verify.** Call `get_table_api_v1_table__table_name__get`
   (`GET /api/v1/table/{table_name}`) to confirm the shape, then
   `get_records_api_v1_table__table_name__records_get`
   (`GET /api/v1/table/{table_name}/records`) with `limit`/`offset` to read rows back.

## Rules
- Writes (create/insert/upload) are `acting`/write operations — see
  `agentic-access/smartmind-agentic-access.yml`; audit is recommended.
- On `422`, read `detail[]` in the `HTTPValidationError` envelope and correct the request.
