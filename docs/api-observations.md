# API Observations and Contract Gaps

## Status of observations

Authentication, product CRUD, query, pagination, and error-handling gaps below were observed through controlled live requests on 2026-07-31. Documentation examples are not treated as empirical evidence, and every undocumented observation remains characterization behavior.

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
| Postman format | Official schema identifies Collection Format v2.1.0 and links its raw schema | Validate the collection JSON against v2.1.0 |
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

## Product CRUD observations

The existing product ID below was selected dynamically from a one-item list response. These requests confirm response behavior only; no simulated mutation was treated as persistent.

### OBS-2026-07-31-007 — PROD-003 nonexistent product read

- Timestamp: `2026-07-31T13:52:17.055Z`
- Request: `GET /products/99999999`.
- Response status: `404`
- Relevant response body: `{ "message": "Product with id '99999999' not found" }`
- Documentation status: Single-product reads are documented; nonexistent-ID behavior is not.
- Classification: Observed characterization.

### OBS-2026-07-31-008 — PROD-004 simulated full create

- Timestamp: `2026-07-31T13:52:17.242Z`
- Request: `POST /products/add`; controlled title, numeric price, description, and category.
- Response status: `201`
- Relevant response body: positive generated `id` plus all submitted fields with matching values.
- Documentation status: Simulated, non-persistent creation and returned ID/fields are documented; the numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-009 — PROD-005 simulated minimal create

- Timestamp: `2026-07-31T13:52:17.976Z`
- Request: `POST /products/add`; title-only JSON body.
- Response status: `201`
- Relevant response body: positive generated `id` and matching submitted title.
- Documentation status: The official create example supplies a title while omitting other product data from the example; persistence is explicitly excluded. Numeric status is observed.
- Classification: Documented example contract plus observed status.

### OBS-2026-07-31-010 — PROD-006 simulated PUT

- Timestamp: `2026-07-31T13:52:18.695Z`
- Request: `PUT /products/{dynamically-selected-existing-id}`; controlled title, price, description, and category.
- Response status: `200`
- Relevant response body: original positive `id` plus all submitted modifications with matching values.
- Documentation status: Simulated, non-persistent PUT and returned modified data are documented; numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-011 — PROD-007 simulated PATCH

- Timestamp: `2026-07-31T13:52:18.880Z`
- Request: `PATCH /products/{dynamically-selected-existing-id}`; controlled title only.
- Response status: `200`
- Relevant response body: original positive `id`, matching submitted title, and other original product fields.
- Documentation status: Simulated, non-persistent PATCH and returned modified data are documented; numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-012 — PROD-009 update nonexistent product

- Timestamp: `2026-07-31T13:52:19.067Z`
- Request: `PUT /products/99999999`; controlled title.
- Response status: `404`
- Relevant response body: `{ "message": "Product with id '99999999' not found" }`
- Documentation status: Update simulation is documented only for existing products; nonexistent-ID behavior is not.
- Classification: Observed characterization.

### OBS-2026-07-31-013 — PROD-008 simulated delete

- Timestamp: `2026-07-31T13:52:19.255Z`
- Request: `DELETE /products/{dynamically-selected-existing-id}`.
- Response status: `200`
- Relevant response body: matching `id`, `isDeleted: true`, and a parseable ISO `deletedOn` string.
- Documentation status: Simulated non-persistent deletion, `isDeleted`, and `deletedOn` are documented; numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-014 — PROD-010 delete nonexistent product

- Timestamp: `2026-07-31T13:52:19.441Z`
- Request: `DELETE /products/99999999`.
- Response status: `404`
- Relevant response body: `{ "message": "Product with id '99999999' not found" }`
- Documentation status: Delete simulation is documented only for existing products; nonexistent-ID behavior is not.
- Classification: Observed characterization.

## Search, category, and pagination observations

All catalog-dependent values below are described relationally. Exact counts, IDs, category slugs, and product values are intentionally not adopted as assertions.

### OBS-2026-07-31-015 — QUERY-001 search matching

- Timestamp: `2026-07-31T14:01:59.261Z`
- Request: `GET /products/search?q=phone`.
- Response status: `200`
- Relevant response body: non-empty `products` array with numeric metadata; sampled matches contained the query in one or more of `title`, `description`, or `tags`.
- Documentation status: The search endpoint and envelope are documented; exact searchable fields and relevance semantics are not.
- Classification: Documented contract plus observed characterization of matching fields.

### OBS-2026-07-31-016 — QUERY-002 unique no-result search

- Timestamp: `2026-07-31T14:01:59.447Z`
- Request: `GET /products/search?q={unique-run-specific-unlikely-query}`.
- Response status: `200`
- Relevant response body: `{ "products": [], "total": 0, "skip": 0, "limit": 0 }`
- Documentation status: Search is documented; the exact no-result envelope is not.
- Classification: Observed characterization.

### OBS-2026-07-31-017 — QUERY-003 category list

- Timestamp: `2026-07-31T14:01:59.651Z`
- Request: `GET /products/category-list`.
- Response status: `200`
- Relevant response body: non-empty array in which every entry was a non-empty string; the first slug was selected dynamically.
- Documentation status: The category-list endpoint and string-array response are documented; numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-018 — QUERY-004 valid category filter

- Timestamp: `2026-07-31T14:01:59.840Z`
- Request: `GET /products/category/{dynamically-selected-slug}`.
- Response status: `200`
- Relevant response body: non-empty products envelope; all returned product categories matched the requested slug and pagination metadata was coherent.
- Documentation status: Category filtering and its envelope are documented; numeric status is observed.
- Classification: Documented contract plus observed status.

### OBS-2026-07-31-019 — QUERY-005 nonexistent category

- Timestamp: `2026-07-31T14:02:00.023Z`
- Request: `GET /products/category/{unique-run-specific-nonexistent-slug}`.
- Response status: `200`
- Relevant response body: `{ "products": [], "total": 0, "skip": 0, "limit": 0 }`
- Documentation status: Valid category filtering is documented; nonexistent-category semantics are not.
- Classification: Observed characterization.

### OBS-2026-07-31-020 — QUERY-006 and QUERY-007 adjacent pages

- Timestamp: `2026-07-31T14:02:00.212Z` and `2026-07-31T14:02:00.399Z`
- Request: adjacent `GET /products` pages using the configured positive limit and `skip=0` then `skip={limit}`.
- Response status: `200` for both requests.
- Relevant response body: each page respected requested `skip` and `limit`, IDs were unique within each page, and no ID overlapped across the two observed pages.
- Documentation status: `limit` and `skip` pagination are documented; no-overlap is a relational expectation for the observed stable snapshot, not a documented ordering guarantee.
- Classification: Documented pagination contract plus observed page relationship.

### OBS-2026-07-31-021 — QUERY-008 field selection

- Timestamp: `2026-07-31T14:02:00.583Z`
- Request: `GET /products?limit=5&select=title,price`.
- Response status: `200`
- Relevant response body: every product object contained exactly `id`, `title`, and `price`; envelope metadata remained outside product objects.
- Documentation status: Comma-separated field selection is documented; inclusion of `id` is shown by the documentation example and confirmed by observation.
- Classification: Documented contract plus observed response shape.

### OBS-2026-07-31-022 — QUERY-009 limit zero

- Timestamp: `2026-07-31T14:02:00.959Z`
- Request: `GET /products?limit=0`.
- Response status: `200`
- Relevant response body: `products.length` equaled response `total`, `skip` was zero, and response `limit` represented the returned item count rather than remaining zero.
- Documentation status: `limit=0` returning all items is documented; returned metadata normalization is observed.
- Classification: Documented contract plus observed metadata behavior.

## Error-handling observations

All outcomes below are undocumented by the official products documentation. Conventional REST expectations were used only to select experiments; assertions are based on these live results. Each case was observed once.

### OBS-2026-07-31-023 — ERR-001 non-numeric product ID

- Timestamp: `2026-07-31T14:12:00.980Z`
- Request: `GET /products/not-a-number`.
- Response status: `404`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: `{ "message": "Product with id 'not-a-number' not found" }`
- Documentation status: Single-product reads are documented; non-numeric identifier handling is not.
- Classification: Experimentally observed characterization; documentation gap.

### OBS-2026-07-31-024 — ERR-002 negative product ID

- Timestamp: `2026-07-31T14:12:01.796Z`
- Request: `GET /products/-1`.
- Response status: `404`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: `{ "message": "Product with id '-1' not found" }`
- Documentation status: Negative identifier handling is not documented.
- Classification: Experimentally observed characterization; documentation gap.

### OBS-2026-07-31-025 — ERR-003 malformed JSON

- Timestamp: `2026-07-31T14:12:01.981Z`
- Request: `POST /products/add`; Content-Type `application/json`; truncated raw body `{"title": "broken"`.
- Response status: `400`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: a non-empty `message` describing the JSON parser position and expected delimiter.
- Documentation status: Simulated valid creation is documented; malformed-body handling is not.
- Classification: Experimentally observed characterization; documentation gap. The parser error is not currently treated as an API defect.

### OBS-2026-07-31-026 — ERR-004 unexpected product field types

- Timestamp: `2026-07-31T14:12:02.193Z`
- Request: `POST /products/add`; controlled JSON body with numeric `title` and string `price`.
- Response status: `201`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: positive generated `id`, numeric `title` echoed unchanged, and string `price` echoed unchanged.
- Documentation status: Simulated creation and field echo are documented for normal example data; validation rules and wrong-type behavior are not.
- Classification: Experimentally observed characterization; permissive validation and expected simulated behavior/API limitation. This is not represented as rejection.

### OBS-2026-07-31-027 — ERR-005 empty create object

- Timestamp: `2026-07-31T14:12:02.379Z`
- Request: `POST /products/add`; JSON body `{}`.
- Response status: `201`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: an object containing only a positive generated `id`.
- Documentation status: No minimum create payload or empty-body validation behavior is documented.
- Classification: Experimentally observed characterization; permissive validation and expected simulated behavior/API limitation.

### OBS-2026-07-31-028 — ERR-006 negative pagination values

- Timestamp: `2026-07-31T14:12:02.572Z`
- Request: `GET /products?limit=-5&skip=-5`.
- Response status: `400`; Content-Type: `application/json; charset=utf-8`.
- Relevant response body: `{ "message": "Invalid 'limit' - should be a positive number" }`
- Documentation status: Positive pagination and `limit=0` are documented; negative parameter handling is not. Because limit failed first, this observation does not independently establish negative-skip behavior.
- Classification: Experimentally observed characterization; documentation gap.

### OBS-2026-07-31-029 — ERR-007 unsupported product-resource POST

- Timestamp: `2026-07-31T14:12:02.757Z`
- Request: `POST /products/1`; JSON body `{}`.
- Response status: `404`; Content-Type: `text/html; charset=utf-8`; no `Allow` header.
- Relevant response body: HTML error text identifying `Cannot POST /products/1`.
- Documentation status: GET, PUT/PATCH, and DELETE are documented for a product resource; POST is documented only for `/products/add`. Unsupported-method behavior is not documented.
- Classification: Experimentally observed characterization; documentation gap. A conventional `405 Method Not Allowed` expectation is explicitly not used as an assertion.

## Unexpected-behavior classification

| Observation | Classification | Rationale |
|---|---|---|
| Invalid IDs return JSON 404 | Documentation gap | Sensible observed behavior, but the official contract does not specify it |
| Malformed JSON returns a parser-oriented JSON 400 | Documentation gap | Useful behavior with undocumented status/body; exposing parser details could merit separate product review but is not labeled a defect here |
| Wrong product field types are accepted and echoed | Permissive validation; expected simulated behavior | DummyJSON simulates writes and documents no product schema enforcement |
| Empty product object is accepted | Permissive validation; expected simulated behavior | The simulator returns a generated ID without defining required fields |
| Negative pagination reports invalid limit | Documentation gap | Negative-input semantics and validation order are undocumented |
| Unsupported product POST returns HTML 404 without `Allow` | Documentation gap; potential API defect | It is testable and internally consistent with route handling, but an API consumer might reasonably prefer structured JSON and method-oriented signaling |
| No implementation/test mismatch observed | No test defect found | Assertions encode the observed behavior and label their oracle accurately |
| No transport variance observed in the controlled probes | No environmental instability observed | Public-service/network instability remains an execution risk, not an observed outcome in this run |

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
