---
name: Query a DataGol workbook table
description: Read schema and filtered rows out of an Events.com DataGol (Saasxl) no-code workbook table, and export the result.
api: openapi/eventscom-datagol-platform-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - getTables
  - getTable
  - columnsCatalog
  - getFilterData
  - getRow
  - getStatisticsData
  - downloadCSV
---

# Query a DataGol workbook table

Reads data out of a DataGol workbook. Everything here is scoped to a `workspaceId` (the API's name for
what the product calls a **workbook**) and a `tableId`.

## Before you start

- **Auth.** Every call needs `Authorization: Bearer <jwt>` — security scheme `s_jwt`, `bearerFormat: JWT`.
  There is no API-key or OAuth path. See `authentication/eventscom-authentication.yml`.
- **Base URL.** `https://datagol-be.events.com/`
- **No idempotency.** This API declares no `Idempotency-Key`. Reads are safe to retry; do not blindly retry
  writes. See `conventions/eventscom-conventions.yml`.
- **Responses declare `*/*`**, not `application/json`. Send `Accept: application/json` and parse defensively.

## Steps

1. **List the tables in the workbook** — `getTables`
   `GET /noCo/api/v2/workspaces/{workspaceId}/tables`
   Pick the `tableId` you need. Do not guess ids.

2. **Read the table definition** — `getTable`
   `GET /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}`

3. **Read the column catalog before filtering** — `columnsCatalog`
   `GET /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}/columnCatalog`
   Filters are expressed against column ids, not column names. Resolve names to ids here first.

4. **Fetch filtered rows** — `getFilterData`
   `POST /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}/data`
   This is a POST because the filter travels in the body. Use this, **not** `getTableData`
   (`GET .../data`) or `getCursorData` (`POST .../cursor`) — both are marked `deprecated: true` in the spec.

5. **Read a single row when you already hold its id** — `getRow`
   `GET /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}/rows/{rowId}`

6. **Aggregate rather than paging everything** — `getStatisticsData`
   `POST /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}/statistics`
   Prefer this over pulling the full table when you only need counts or summaries.

7. **Export** — `downloadCSV`
   `POST /noCo/api/v2/workspaces/{workspaceId}/tables/{tableId}/downloadCSV`
   For large tables use `downloadCSVChunk`; `downloadExcel`, `downloadParquet` and `downloadQueryResult`
   are also available.

## Pagination

Pagination is inconsistent across this API. Depending on the operation you will see `page`+`size`
(Spring `Pageable`), `pageSize`, or `limit`. Read the parameter list of the specific operation rather than
assuming — and note that most collection endpoints declare no pagination at all.

## Errors

Errors come back in the `SaasxlResponseDTO` envelope: `{success, version, date, data, error}` where
`error` is `{statusCode, requestId, message, details, errorCodes[]}`.

- `403` and `500`/`501`/`503` are declared on every operation.
- `429` is declared on 14 operations, with **no** `Retry-After` header — back off exponentially.
- `error.errorCodes[]` has no published registry, so branch on HTTP status, not on code strings.
- Capture `error.requestId` in any support report; it is the only correlation id this API returns.

See `errors/eventscom-problem-types.yml`.
