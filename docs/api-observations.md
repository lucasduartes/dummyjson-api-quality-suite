# API Observations and Contract Gaps

## Status of observations

The authentication negative cases below were observed through controlled live requests on 2026-07-31. Other contract gaps remain unobserved. Documentation examples are not treated as empirical evidence.

When implementation begins, append observations with date/time, request (with secrets removed), status, relevant headers, sanitized body shape, repeat count, and whether the behavior is safe to assert. Never overwrite earlier observations when the public API changes.

## Collection organization and oracle hierarchy

- `00 - Health Check`
- `01 - Authentication`: `Positive`, `Negative`
- `02 - Products CRUD`: `Read`, `Create`, `Update`, `Delete`
- `03 - Search Filtering and Pagination`: `Search`, `Categories`, `Pagination`
- `04 - Error Handling`

The shared oracle hierarchy is documented contract, then dated observed characterization, with conventional expectations used only to design experiments.

## Documented facts

| Area | Documented fact | Testing consequence |
|---|---|---|
| Health | `/test` accepts multiple methods and the example returns status and method | Use GET as the P0 connectivity sentinel; other methods are not needed for health coverage |
| Login | `/auth/login` accepts username/password and optional `expiresInMins`; response includes access and refresh tokens and user data | Capture runtime tokens and user ID; do not validate token signature |
| Current user | `/auth/me` accepts an access token in a Bearer header | Make it dependent on successful login and explicitly control the header |
| Refresh | `/auth/refresh` accepts a refresh token in JSON or cookie and returns tokens | Test JSON token flow; do not depend on cookies or wait for expiry |
| Product list | Default response has a products envelope and default page of 30 | Assert envelope and bounded default page, not example catalog total |
| Pagination | `limit` and `skip` paginate; `limit=0` returns all items | Use relational paging and total assertions |
| Projection | `select` accepts comma-separated values | Cover comma-separated form in P1 |
| Search | `/products/search?q=...` searches products | Cover hit and miss; matching algorithm is a gap |
| Category | Category list and category-product endpoints are documented | Discover a valid slug dynamically and assert category membership |
| Create | `/products/add` simulates creation, returns a new ID, and does not add to the server | Assert response echo only; never chain a GET to the new ID as if persisted |
| Update | PUT/PATCH simulate modification, return modified data, and do not update the server | Assert returned modification; do not expect a later GET to retain it |
| Delete | DELETE simulates deletion, returns `isDeleted` and `deletedOn`, and does not delete on server | Assert deletion metadata; do not expect subsequent 404 |
| Postman format | Official schema identifies Collection Format v2.1.0 and links its raw schema | Validate future collection JSON against v2.1.0 |
| Newman | Newman runs exported collection JSON; environment uses `-e`; current docs require Node.js 16+ | Plan folder/full CLI runs and keep environment values safe |

## Contract gaps requiring characterization

Characterization scenarios follow the collection organization defined consistently across the plan and strategy: authentication gaps live in `01 - Authentication/Negative`, CRUD edge cases in the relevant `02 - Products CRUD` child, query gaps in `03 - Search Filtering and Pagination`, and general malformed/unsupported cases in `04 - Error Handling`.

| Gap | Planned experiment | Assertion policy before observation |
|---|---|---|
| Invalid login status/body | Wrong username/password | Assert no tokens/authenticated identity; do not assert exact status/message |
| Missing username/password | Omit each field independently | Record validation behavior; no exact status |
| Missing/malformed bearer | Remove header; then send corrupt Bearer value | Assert no authenticated identity; exact status/body open |
| Invalid refresh token | Send well-formed but invalid value | Assert no usable session; exact status/body open |
| Nonexistent/invalid product IDs | Test dynamically large, zero, negative, and nonnumeric IDs | Assert no unrelated product; exact status/body open |
| Invalid category | Use improbable slug | Assert no category products; status vs empty envelope open |
| Search matching semantics | Derive terms from title and possibly description/category | Do not assert every result contains title substring until matching fields are observed |
| Invalid pagination | Negative and nonnumeric limit/skip | Record rejection/coercion/defaulting separately |
| Malformed JSON | Truncated JSON with JSON content type | Assert no successful creation; parser status/message open |
| Unexpected types | Wrong-type title/price values | Record reject/coerce/echo behavior; avoid assuming validation |
| Unsupported method | Select a method only after exploratory observation; HTTP infrastructure may handle HEAD and OPTIONS specially | Record status, Allow header, and body; do not assume 405 |
| PUT semantics | Send partial PUT payload | Only assert submitted modification; full replacement versus merge is unspecified |

## Observation template

```markdown
### OBS-YYYY-MM-DD-NNN — short title

- Timestamp:
- Endpoint and method:
- Sanitized request:
- Repetitions:
- Status and relevant headers:
- Sanitized response shape:
- Classification (complete only after live execution):
- Stability/brittleness assessment:
- Proposed assertion (if any):
```

## Authentication observations

All request values were controlled test fixtures. No access token or refresh token returned by the service was printed or recorded. Each result below is a single observation, so exact messages remain more brittle than status and error semantics.

### OBS-2026-07-31-001 — AUTH-004 invalid password

- Timestamp: `2026-07-31T13:28:31.866Z`
- Request: `POST /auth/login`; documented username fixture plus a request-specific invalid password; JSON body.
- Response status: `400`
- Relevant response body: `{ "message": "Invalid credentials" }`
- Documentation status: Negative status and error body are not specified by the official auth documentation.
- Classification: Observed characterization.

### OBS-2026-07-31-002 — AUTH-005 missing username

- Timestamp: `2026-07-31T13:28:32.078Z`
- Request: `POST /auth/login`; password field only; JSON body.
- Response status: `400`
- Relevant response body: `{ "message": "Username and password required" }`
- Documentation status: Missing-field behavior is not specified by the official auth documentation.
- Classification: Observed characterization.

### OBS-2026-07-31-003 — AUTH-006 missing password

- Timestamp: `2026-07-31T13:28:32.265Z`
- Request: `POST /auth/login`; username field only; JSON body.
- Response status: `400`
- Relevant response body: `{ "message": "Username and password required" }`
- Documentation status: Missing-field behavior is not specified by the official auth documentation.
- Classification: Observed characterization.

### OBS-2026-07-31-004 — AUTH-007 missing access token

- Timestamp: `2026-07-31T13:28:32.455Z`
- Request: `GET /auth/me`; Authorization header absent and authentication cookies absent/explicitly empty.
- Response status: `401`
- Relevant response body: `{ "message": "Access Token is required" }`
- Documentation status: Bearer authentication is documented; the missing-token failure response is not.
- Classification: Observed characterization.

### OBS-2026-07-31-005 — AUTH-008 malformed bearer token

- Timestamp: `2026-07-31T13:28:32.641Z`
- Request: `GET /auth/me`; request-specific malformed Bearer value, redacted from this document.
- Response status: `401`
- Relevant response body: `{ "message": "Invalid/Expired Token!" }`
- Documentation status: Bearer authentication is documented; malformed-token behavior is not.
- Classification: Observed characterization.

### OBS-2026-07-31-006 — AUTH-009 invalid refresh token

- Timestamp: `2026-07-31T13:28:32.832Z`
- Request: `POST /auth/refresh`; request-specific invalid refresh value, redacted; `expiresInMins: 30`.
- Response status: `403`
- Relevant response body: `{ "message": "Invalid refresh token" }`
- Documentation status: Refresh-token exchange is documented; invalid-token behavior is not.
- Classification: Observed characterization.

## Mutable-public-API risks

- Product totals, IDs, fields, titles, categories, and search matches can change.
- Shared demo credentials or returned identity data can change.
- Edge responses can change independently of published documentation.
- Requests can straddle a data refresh, producing internally valid but different pages.
- Network, TLS, throttling, or upstream outages can mimic product defects.

Controls: dynamic discovery, same-response relational checks, explicit health diagnostics, low request volume, no fixed response-time threshold, narrow schemas, repeat observations before pinning, and a visible documented/observed distinction.

## False-positive and brittleness watchlist

- A `200`-only test can pass on an error object.
- A JSON parse failure must fail the request tests, not silently skip them.
- Negative-auth requests can accidentally inherit a valid token.
- An empty positive-result array can pass vacuous `every()` assertions; require non-empty first.
- Comparing response values only with other response values may prove nothing; compare with request inputs or captured baselines.
- Full snapshots and exact messages bind tests to mutable data and incidental wording.
- Retrying failed assertions can conceal genuine functional defects.
- Persisted Postman current values can leak tokens or make isolated runs order-dependent.

## Source review date

Official sources reviewed on 2026-07-31:

- [DummyJSON documentation](https://dummyjson.com/docs)
- [DummyJSON auth documentation](https://dummyjson.com/docs/auth)
- [DummyJSON product documentation](https://dummyjson.com/docs/products)
- [Postman Collection v2.1.0 schema documentation](https://schema.postman.com/collection/json/v2.1.0/draft-07/docs/index.html)
- [Newman installation and execution documentation](https://learning.postman.com/docs/reference/newman-cli/installing-running-newman)
