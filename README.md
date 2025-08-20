## Scholarships API (v2)

This document describes the Scholarships API endpoint for listing scholarships.

- Endpoint: GET /api/v2/scholarships

## Authentication

OAuth 2.0 Client Credentials flow.

- Token URL (placeholder): https://<idp>/oauth2/token
- Request body (x-www-form-urlencoded):
  - grant_type=client_credentials
  - client_id=YOUR_CLIENT_ID
  - client_secret=YOUR_CLIENT_SECRET

API requests must include:
- Authorization: Bearer <access_token>

401 Unauthorized: Returned when the token is missing or invalid.

### Token acquisition (curl)

```bash
# Obtain an access token (example)
curl -X POST "https://<idp>/oauth2/token" ^
  -H "Content-Type: application/x-www-form-urlencoded" ^
  --data-urlencode "grant_type=client_credentials" ^
  --data-urlencode "client_id=YOUR_CLIENT_ID" ^
  --data-urlencode "client_secret=YOUR_CLIENT_SECRET" ^
  --data-urlencode "scope=scholarships:read"
```

The response should include an access token usable as a Bearer token.

## Endpoint

GET /api/v2/scholarships

### Query parameters

| Name         | Type     | Required | Default | Constraints            | Description                                                                 |
|--------------|----------|----------|---------|------------------------|-----------------------------------------------------------------------------|
| search_text  | string   | no       | —       | —                      | Case-insensitive “contains” across relevant scholarship text fields.        |
| updated_from | datetime (UTC, ISO 8601) | no | — | inclusive lower bound | Inclusive lower bound compared to the scholarship updatedDate.              |
| updated_to   | datetime (UTC, ISO 8601) | no | — | inclusive upper bound | Inclusive upper bound compared to the scholarship updatedDate.              |
| limit        | integer  | no       | 20      | 1..200                 | Page size. Capped at 200.                                                   |
| offset       | integer  | no       | 0       | ≥ 0                    | Zero-based offset into the filtered result set.                             |

Notes:
- If updated_from and updated_to are both provided, updated_from must be <= updated_to.

### Response

Envelope:

| Field   | Type     | Description                                                          |
|---------|----------|----------------------------------------------------------------------|
| total   | integer  | Total number of records after applying filters.                      |
| hasNext | boolean  | true when (offset + limit) < total.                                  |
| items   | array    | Array of Scholarship objects.                                        |

### Scholarship object

Every Scholarship item includes all fields listed below exactly as serialized by the current implementation. Field casing is preserved. Types and nullability reflect runtime behavior.

Core “sample-compatible” fields (UPPERCASE):
- NO: string, non-null. Line number for the current page (stringified), falls back to entity id when available.
- TITLE: string, nullable. Scholarship title.
- DESCRIPTION: string, nullable. Rich/long description for display.
- CRITERIA: string, nullable. Special criteria text.
- SCHOLARSHIP_CODE: string, nullable. Scholarship code.
- ELIGIBILITY: string, nullable. Eligibility description.
- AMOUNT_OF_AWARD: string, nullable. Human-readable award amount (or amount if display value missing).
- AMOUNT: string, nullable. Raw amount string.
- DEADLINES: string, nullable. A primary deadline (first found among program links).
- DEADLINES_LIST: string[], non-null (may be empty). All distinct deadlines linked via programs.
- GROUPINGS: string, nullable. A primary grouping (first found among groups).
- APPLICATION_FORMAT: string, nullable. How to apply (display text).
- PROGRAMME: string, nullable. A primary programme/career value (lowercased).
- IS_ACTIVE: "1" or "0", non-null. Active flag as string.
- UPDATED_DATE: string, nullable. Last update in "Y-m-d H:i:s" format.

Arrays (UPPERCASE):
- REGION: string[], non-null. Regions (labels).
- GROUPING: string[], non-null. Groupings (labels).
- GENDER: string[], non-null. Genders (labels).
- CAMPUS: string[], non-null. Campus codes (deduplicated).
- NATIONALITY: string[], non-null. ISO3 nationality codes.

Program-derived (UPPERCASE):
- PROGRAM_IDS: integer[], non-null (may be empty). Program IDs linked via scholarshipPrograms.
- ACCEPTS_CURRENT_TERM: "1" | "0" | null. Aggregate of program flags (1 if any program is true, 0 if any are false and none true, null if none present).
- ACCEPTS_CURRENT_TERM_LIST: string[], non-null. Per-program flags as "1"/"0".
- APPLICATION_ONLINE: "1" | "0" | null. Aggregate of program applicationOnline flags (same aggregation logic).
- APPLICATION_ONLINE_LIST: string[], non-null. Per-program flags as "1"/"0".
- FUND_ACCOUNT_CODE: string[], non-null. Distinct fund account codes from programs.
- SERVICE_INDICATOR_REFERENCE: string[], non-null. Distinct service indicator references.
- SUPPORT_DOCUMENT_DEADLINE: string[], non-null. Dates as "Y-m-d".
- TYPE: string[], non-null. Distinct program types (enum values).
- FREQUENCY: string[], non-null. Distinct frequencies (enum values).
- VALID_UNTIL: string[], non-null. Distinct valid-until values (enum values).
- DECISION_BY: string[], non-null. Distinct decision-by values (enum values).

Additional (UPPERCASE):
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

CamelCase fields (complete set):
- id: integer, nullable. Scholarship ID.
- code: string, nullable.
- title: string, nullable.
- isInseadManaged: boolean, nullable.
- isActive: boolean, nullable.
- isValid: boolean, nullable.
- displayNew: boolean, nullable.
- displaySpecialCriteria: string, nullable.
- displayDescription: string, nullable.
- displayEligibility: string, nullable.
- displayAmountAwarded: string, nullable.
- isOnline: boolean, nullable.
- displayApplicationFormat: string, nullable.
- displayFooter: string, nullable.
- displayFileUrl: string, nullable.
- displayImagePath: string, nullable.
- displayImageUrl: string, nullable.
- isApplicationView: boolean, nullable.
- updatedDate: string (ISO 8601), nullable. e.g., "2025-08-19T08:15:30+00:00".
- startYear: string, nullable.
- isSupportDocumentRequired: boolean, nullable.
- amount: string, nullable.
- regions: string[], non-null.
- groups: string[], non-null.
- genders: string[], non-null.
- nationalities: string[], non-null.
- campuses: string[], non-null.

Nested programs:
- scholarshipPrograms: array of objects, non-null (may be empty). Each item:
  - programId: integer, nullable.
  - programCode: string, nullable.
  - career: string, nullable.
  - plan: string, nullable.
  - subPlan: string, nullable.
  - deadline: string, nullable.
  - acceptsCurrentTerm: boolean, nullable.
  - fundAccountCode: string, nullable.
  - serviceIndicatorReference: string, nullable.
  - supportDocumentDeadline: string (Y-m-d), nullable.
  - type: string, nullable.
  - frequency: string, nullable.
  - validUntil: string, nullable.
  - decisionBy: string, nullable.
  - applicationOnline: boolean, nullable.

### Examples

Note: Use a relative path. If needed, define a base URL in your client.

#### List scholarships (curl)

```bash
# <BASE_URL> is the server root (not included in the request path below)
curl -X GET "/api/v2/scholarships?limit=20&offset=0&search_text=need&updated_from=2025-01-01T00:00:00Z" ^
  -H "Authorization: Bearer <access_token>" ^
  -H "Accept: application/json"
```

#### List scholarships (JavaScript fetch)

```javascript
// const BASE_URL = 'https://your-api';
const res = await fetch('/api/v2/scholarships?limit=20&offset=0', {
  headers: {
    'Authorization': 'Bearer <access_token>',
    'Accept': 'application/json'
  }
});
const data = await res.json();
console.log(data);
```

#### List scholarships (Python requests)

```python
import requests

# BASE_URL = 'https://your-api'
resp = requests.get(
    '/api/v2/scholarships',
    headers={
        'Authorization': 'Bearer <access_token>',
        'Accept': 'application/json'
    },
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

### Example response payload

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
      "id": 1234,
      "code": "MIF-NEED-1",
      "title": "INSEAD Need-Based Scholarship",
      "isInseadManaged": true,
      "isActive": true,
      "isValid": true,
      "displayNew": false,
      "displaySpecialCriteria": "Financial need; strong academics.",
      "displayDescription": "Scholarship for candidates demonstrating financial need.",
      "displayEligibility": "Admitted students to MIM or MBA.",
      "displayAmountAwarded": "Up to €20,000",
      "isOnline": true,
      "displayApplicationFormat": "Online application",
      "displayFooter": null,
      "displayFileUrl": null,
      "displayImagePath": null,
      "displayImageUrl": null,
      "isApplicationView": false,
      "updatedDate": "2025-08-18T15:42:10+00:00",
      "startYear": "2025",
      "isSupportDocumentRequired": true,
      "amount": "20000",
      "regions": ["Europe", "Asia"],
      "groups": ["Need-based"],
      "genders": ["All"],
      "nationalities": ["FRA", "SGP", "USA"],
      "campuses": ["FR", "SG"],
      "scholarshipPrograms": [
        {
          "programId": 101,
          "programCode": "MIM",
          "career": "MIM",
          "plan": "PLAN-A",
          "subPlan": null,
          "deadline": "2025-09-01",
          "acceptsCurrentTerm": true,
          "fundAccountCode": "FAC-123",
          "serviceIndicatorReference": "SVC-9",
          "supportDocumentDeadline": "2025-08-25",
          "type": "NEED",
          "frequency": "ANNUAL",
          "validUntil": "UNTIL_ENROLLMENT",
          "decisionBy": "ADMISSIONS",
          "applicationOnline": true
        }
      ]
    }
  ]
}
```

## Status codes

- 200 OK
- 400 Bad Request — e.g., invalid datetime format, or updated_from > updated_to.
- 401 Unauthorized
- 404 Not Found
- 500 Internal Server Error

## Behavior notes

- Sorting is deterministic: results are ordered by updatedDate DESC, code ASC, id ASC.
- hasNext is computed as (offset + limit) < total.
- Only the query parameters listed above are supported for this documentation scope. No undocumented parameters, aliases, or diagnostics are exposed here.
