# Scholarships API (v2)

This document describes the Scholarships API endpoints for exporting the scholarship catalogue.

- Base path: /api/v2
- Endpoints:
  - GET /api/v2/scholarships — paginated list
  - GET /api/v2/scholarships/all — full export (no pagination; streams internally)

## Authentication

Authorization header is required. Two modes are supported by the current implementation:

- Basic authentication
  - Header: Authorization: Basic base64(client_id:client_secret)
  - The provided client_id/client_secret must match a configured API client and must include the scope scholarships:read.
- Bearer token (optional, only when enabled)
  - Header: Authorization: Bearer <token>
  - Accepted forms:
    - HS256 JWT signed with configured jwt_key
    - Opaque token matching either a configured explicit token or base64(client_id:client_secret) for a configured client
  - Bearer is rejected when the instance is configured as Basic-only.

Required scope for both modes: scholarships:read

Authorization failures:
- 401 Unauthorized — missing/invalid Authorization header, invalid credentials/token
- 403 Forbidden — authenticated but missing required scope

### Examples (Windows cmd)

Basic auth with -u:

```bat
curl -X GET "https://<BASE_HOST>/api/v2/scholarships?limit=20&offset=0" ^
  -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" ^
  -H "Accept: application/json"
```

Bearer token (if enabled):

```bat
curl -X GET "https://<BASE_HOST>/api/v2/scholarships?limit=50" ^
  -H "Authorization: Bearer %YOUR_TOKEN%" ^
  -H "Accept: application/json"
```

## Endpoints

### GET /api/v2/scholarships

Paginated list, sorted deterministically by updatedDate DESC, code ASC, id ASC.

Query parameters:

| Name | Type | Req | Default | Constraints | Description |
|------|------|-----|---------|-------------|-------------|
| limit | integer | no | 20 | 1..200 | Page size; alias: per_page |
| offset | integer | no | 0 | ≥ 0 | Zero-based offset into filtered set |
| search_text | string | no | — | — | Case-insensitive contains across core fields and related program data; aliases: search, q |
| updated_from | datetime (ISO 8601) | no | — | inclusive | Inclusive lower bound on updatedDate; alias: from |
| updated_to | datetime (ISO 8601) | no | — | inclusive | Inclusive upper bound on updatedDate; alias: to |
| fields | string | no | — | mask | Inclusion mask for response projection; comma-separated paths with dot-notation and "*" wildcards |
| exclude | string | no | — | mask | Exclusion mask applied after fields |

Additional filters (all optional; comma-separated lists are supported where noted):

- Scholarship string fields (LIKE): title, code, description, eligibility, criteria, amount, amount_of_award, application_format, footer, file_url, image_path, image_url, start_year
- Scholarship booleans (accepted values: 1/0, true/false, yes/no, y/n): is_active, is_valid, is_insead_managed, display_new, is_online, is_application_view, is_support_document_required
- Relations (comma-separated): region, grouping, gender, nationality, campus
  - Aliases: regions→region, groups/groupings→grouping, genders→gender, nationalities→nationality, campuses→campus
- Program-derived filters:
  - LIKE: fund_account_code, service_indicator_reference, deadlines
  - Booleans: accepts_current_term, application_online
  - Enums/lists (comma-separated): type, frequency, valid_until, decision_by
  - Dates: support_document_deadline (exact Y-m-d), support_document_deadline_from (>=), support_document_deadline_to (<=)
  - Program selectors: program_ids (ids list), program_code (LIKE, comma-separated), program_code_exact (exact match list)
  - Aliases: program_id→program_ids, programme/program→program_code, programme_exact/program_exact→program_code_exact

Notes:
- If updated_from and updated_to are both provided, updated_from must be <= updated_to.
- Date parsing accepts common ISO-8601 formats and Y-m-d.

Response envelope:

| Field | Type | Description |
|-------|------|-------------|
| total | integer | Total number of records after filters |
| hasNext | boolean | True when (offset + limit) < total |
| items | array | Array of Scholarship objects |

### GET /api/v2/scholarships/all

Returns the full result set using internal batching (200 per batch). Query parameters are the same as the paginated endpoint except limit/offset are ignored. hasNext is always false.

## Scholarship object

Fields are returned in the following UPPER_CASE shape. Types and nullability reflect current runtime behavior.

Core fields:
- NO: string, non-null. Scholarship id as string when available; otherwise a 1-based sequence number within this response.
- TITLE: string, nullable. Scholarship title.
- DESCRIPTION: string, nullable. Rich/long display description.
- CRITERIA: string, nullable. Special criteria text (display).
- SCHOLARSHIP_CODE: string, nullable. Scholarship code.
- ELIGIBILITY: string, nullable. Eligibility description (display).
- AMOUNT_OF_AWARD: string, nullable. Human-readable award amount (or amount if display missing).
- AMOUNT: string, nullable. Raw amount string.
- DEADLINES: string, nullable. Primary deadline (first found among program links).
- DEADLINES_LIST: string[], non-null (may be empty). All distinct deadlines linked via programs.
- GROUPINGS: string, nullable. Primary grouping (first found among groups).
- APPLICATION_FORMAT: string, nullable. How to apply (display text).
- PROGRAMME: string, nullable. Primary programme/career value (lowercased).
- IS_ACTIVE: "1" or "0", non-null. Active flag as string.
- UPDATED_DATE: string, nullable. Last update in "Y-m-d H:i:s".

Arrays:
- REGION: string[], non-null. Regions (labels).
- GROUPING: string[], non-null. Groupings (labels).
- GENDER: string[], non-null. Genders (labels).
- CAMPUS: string[], non-null. Campus codes (deduplicated).
- NATIONALITY: string[], non-null. ISO3 nationality codes.

Program-derived (aggregated across links):
- PROGRAM_IDS: integer[], non-null (may be empty).
- ACCEPTS_CURRENT_TERM: "1" | "0" | null. Aggregate of program flags.
- ACCEPTS_CURRENT_TERM_LIST: string[], non-null. Per-program flags as "1"/"0".
- APPLICATION_ONLINE: "1" | "0" | null. Aggregate of program flags.
- APPLICATION_ONLINE_LIST: string[], non-null. Per-program flags as "1"/"0".
- FUND_ACCOUNT_CODE: string[], non-null.
- SERVICE_INDICATOR_REFERENCE: string[], non-null.
- SUPPORT_DOCUMENT_DEADLINE: string[], non-null. Dates as "Y-m-d".
- TYPE: string[], non-null.
- FREQUENCY: string[], non-null.
- VALID_UNTIL: string[], non-null.
- DECISION_BY: string[], non-null.

Additional:
- IS_INSEAD_MANAGED: boolean, nullable.
- IS_VALID: boolean, nullable.
- DISPLAY_NEW: boolean, nullable.
- DISPLAY_FOOTER: string, nullable.
- DISPLAY_FILE_URL: string, nullable.
- DISPLAY_IMAGE_PATH: string, nullable.
- DISPLAY_IMAGE_URL: string, nullable.
- IS_ONLINE: boolean, nullable.
- IS_APPLICATION_VIEW: boolean, nullable.
- START_YEAR: string (length 4), nullable.
- IS_SUPPORT_DOCUMENT_REQUIRED: boolean, nullable.

Per-program details:
- SCHOLARSHIP_PROGRAMS: array<object>, non-null (may be empty). Each item has UPPER_CASE keys:
  - PROGRAM_ID: integer, nullable.
  - PROGRAM_CODE: string, nullable.
  - CAREER: string, nullable.
  - PLAN: string, nullable.
  - SUB_PLAN: string, nullable.
  - DEADLINE: string, nullable.
  - ACCEPTS_CURRENT_TERM: "1" | "0" | null.
  - FUND_ACCOUNT_CODE: string, nullable.
  - SERVICE_INDICATOR_REFERENCE: string, nullable.
  - SUPPORT_DOCUMENT_DEADLINE: string (Y-m-d), nullable.
  - TYPE: string, nullable.
  - FREQUENCY: string, nullable.
  - VALID_UNTIL: string, nullable.
  - DECISION_BY: string, nullable.
  - APPLICATION_ONLINE: "1" | "0" | null.

Note: CamelCase duplicates (e.g., id, title, scholarshipPrograms, etc.) are not included by this endpoint.

## Sorting and paging

- Sorting is deterministic: updatedDate DESC, code ASC, id ASC.
- hasNext is computed as (offset + limit) < total.

## Field projection (fields/exclude)

- fields: comma-separated list of paths to include. Use dot-notation for nesting and "*" as a wildcard for array/object keys.
  - Example: fields=TITLE,SCHOLARSHIP_PROGRAMS.*.DEADLINE
- exclude: comma-separated list of paths to remove after inclusion.

## Examples

List scholarships (Basic auth):

```bat
curl -X GET "https://<BASE_HOST>/api/v2/scholarships?limit=20&offset=0&search_text=need&updated_from=2025-01-01T00:00:00Z" ^
  -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" ^
  -H "Accept: application/json"
```

JavaScript fetch (Basic auth):

```javascript
const creds = btoa('YOUR_CLIENT_ID:YOUR_CLIENT_SECRET');
const res = await fetch('/api/v2/scholarships?limit=20&offset=0', {
  headers: {
    'Authorization': `Basic ${creds}`,
    'Accept': 'application/json'
  }
});
const data = await res.json();
console.log(data);
```

Python requests (Basic auth):

```python
import requests

resp = requests.get(
    'https://<BASE_HOST>/api/v2/scholarships',
    auth=('YOUR_CLIENT_ID', 'YOUR_CLIENT_SECRET'),
    headers={'Accept': 'application/json'},
    params={
        'limit': 20,
        'offset': 0,
        'search_text': 'need',
        'updated_from': '2025-01-01T00:00:00Z'
    }
)
resp.raise_for_status()
print(resp.json())
```

Example response (truncated to one item):

```json
{
  "total": 234,
  "hasNext": true,
  "items": [
    {
      "NO": "1",
      "TITLE": "INSEAD Need-Based Scholarship",
      "DESCRIPTION": "Scholarship for candidates demonstrating financial need.",
      "CRITERIA": "Financial need; strong academics.",
      "SCHOLARSHIP_CODE": "MIF-NEED-1",
      "ELIGIBILITY": "Admitted students to MIM or MBA.",
      "AMOUNT_OF_AWARD": "Up to €20,000",
      "AMOUNT": "20000",
      "DEADLINES": "2025-09-01",
      "DEADLINES_LIST": ["2025-09-01", "2025-10-15"],
      "GROUPINGS": "Need-based",
      "APPLICATION_FORMAT": "Online application",
      "PROGRAMME": "mim",
      "IS_ACTIVE": "1",
      "UPDATED_DATE": "2025-08-18 15:42:10",
      "REGION": ["Europe", "Asia"],
      "GROUPING": ["Need-based"],
      "GENDER": ["All"],
      "CAMPUS": ["FR", "SG"],
      "NATIONALITY": ["FRA", "SGP", "USA"],
      "PROGRAM_IDS": [101, 205],
      "ACCEPTS_CURRENT_TERM": "1",
      "ACCEPTS_CURRENT_TERM_LIST": ["1", "0"],
      "APPLICATION_ONLINE": "1",
      "APPLICATION_ONLINE_LIST": ["1", "1"],
      "FUND_ACCOUNT_CODE": ["FAC-123", "FAC-456"],
      "SERVICE_INDICATOR_REFERENCE": ["SVC-9"],
      "SUPPORT_DOCUMENT_DEADLINE": ["2025-08-25"],
      "TYPE": ["NEED"],
      "FREQUENCY": ["ANNUAL"],
      "VALID_UNTIL": ["UNTIL_ENROLLMENT"],
      "DECISION_BY": ["ADMISSIONS"],
      "IS_INSEAD_MANAGED": true,
      "IS_VALID": true,
      "DISPLAY_NEW": false,
      "DISPLAY_FOOTER": null,
      "DISPLAY_FILE_URL": null,
      "DISPLAY_IMAGE_PATH": null,
      "DISPLAY_IMAGE_URL": null,
      "IS_ONLINE": true,
      "IS_APPLICATION_VIEW": false,
      "START_YEAR": "2025",
      "IS_SUPPORT_DOCUMENT_REQUIRED": true,
      "SCHOLARSHIP_PROGRAMS": [
        {
          "PROGRAM_ID": 101,
          "PROGRAM_CODE": "MIM",
          "CAREER": "MIM",
          "PLAN": "PLAN-A",
          "SUB_PLAN": null,
          "DEADLINE": "2025-09-01",
          "ACCEPTS_CURRENT_TERM": "1",
          "FUND_ACCOUNT_CODE": "FAC-123",
          "SERVICE_INDICATOR_REFERENCE": "SVC-9",
          "SUPPORT_DOCUMENT_DEADLINE": "2025-08-25",
          "TYPE": "NEED",
          "FREQUENCY": "ANNUAL",
          "VALID_UNTIL": "UNTIL_ENROLLMENT",
          "DECISION_BY": "ADMISSIONS",
          "APPLICATION_ONLINE": "1"
        }
      ]
    }
  ]
}
```

## Status codes

- 200 OK
- 400 Bad Request — e.g., invalid datetime format, or updated_from > updated_to
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 500 Internal Server Error

## Diagnostics

- Add debug=1 to include a debug object with parsed inputs (limit, offset, search, updated_from/updated_to, filters), for troubleshooting only.
