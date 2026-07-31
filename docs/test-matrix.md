# Test Matrix

`Documented` means the cited DummyJSON pages explicitly support the expectation. A future `Observed` expectation requires a dated live result in `api-observations.md`; none exists yet. `Conventional` is only an experiment hypothesis and never supplies an invented exact status. Characterization rows must be observed before strict status/body assertions are implemented.

## Collection mapping

- `00 - Health Check`: HEALTH tests.
- `01 - Authentication/Positive`: AUTH-001 through AUTH-003; `Negative`: AUTH-004 through AUTH-009.
- `02 - Products CRUD/Read`: PROD-001 through PROD-003; `Create`: PROD-004 and PROD-005; `Update`: PROD-006 and PROD-009; `Delete`: PROD-008 and PROD-010. PROD-007 is in `Update` as PATCH.
- `03 - Search Filtering and Pagination/Search`: QUERY-001 and QUERY-002; `Categories`: QUERY-003 through QUERY-005; `Pagination`: QUERY-006 through QUERY-009.
- `04 - Error Handling`: ERR-001 through ERR-007.

The oracle hierarchy is documented contract, then dated observed characterization, with conventional expectations used only to design undocumented experiments. Priorities preserve the assessment: split rows retain the source scenario's priority; newly requested edge cases with no supplied priority are P1.

| Test ID | Priority | Endpoint | Method | Scenario | Risk addressed | Preconditions | Expected result | Source of expectation | Contract or characterization classification | Potential brittleness |
|---|---|---|---|---|---|---|---|---|---|---|
| HEALTH-001 | P0 | `/test` | GET | API health check | Outage makes all functional results misleading | Base URL reachable | JSON reports `status: ok` and method `GET` | Documented | Contract | Service/network availability |
| AUTH-001 | P0 | `/auth/login` | POST | Valid login | Users cannot establish a session | Documented fixture or valid runtime credentials | User identity plus non-empty access and refresh token strings | Documented | Contract | Public fixture identity may change |
| AUTH-002 | P0 | `/auth/me` | GET | Current authenticated user | Token unusable or mapped to wrong user | AUTH-001 access token | Identity matches AUTH-001 | Documented | Contract | Public fixture data may change |
| AUTH-003 | P0 | `/auth/refresh` | POST | Refresh session | Session cannot be extended | AUTH-001 refresh token | Non-empty access and refresh token strings | Documented | Contract | Token contents/rotation policy |
| AUTH-004 | P0 | `/auth/login` | POST | Invalid password or invalid credentials | Invalid user might authenticate | Known-invalid credential pair | Authentication is not granted; exact response awaits characterization | Conventional | Characterization | Status/message undocumented |
| AUTH-005 | P1 | `/auth/login` | POST | Missing username | Incomplete credentials accepted | Password only | Authentication is not granted; characterize response | Conventional | Characterization | Validation rules undocumented |
| AUTH-006 | P1 | `/auth/login` | POST | Missing password | Incomplete credentials accepted | Username only | Authentication is not granted; characterize response | Conventional | Characterization | Validation rules undocumented |
| AUTH-007 | P0 | `/auth/me` | GET | Missing access token | Protected identity leaks | Authorization explicitly absent | No authenticated identity; exact response awaits characterization | Documented bearer requirement | Characterization | Failure status/body undocumented |
| AUTH-008 | P0 | `/auth/me` | GET | Malformed bearer token | Corrupt token accepted | Explicit malformed Bearer value | No authenticated identity; exact response awaits characterization | Conventional | Characterization | Failure status/body undocumented |
| AUTH-009 | P1 | `/auth/refresh` | POST | Invalid refresh token | Invalid token creates a session | Known-invalid token | No usable session; exact response awaits characterization | Conventional | Characterization | Failure status/body undocumented |
| PROD-001 | P0 | `/products?limit=5&skip=0` | GET | List products using limit=5 and skip=0 and store an existing ID | Broken envelope/pagination blocks dependent tests | None | `products` is an array; length > 0 and <= 5; `skip` = 0; `limit` corresponds to 5; `total` is numeric and non-negative; store one ID; assert no exact total | Documented | Contract | Dataset may change between requests |
| PROD-002 | P0 | `/products/:id` | GET | Get existing product | Lookup returns wrong record | ID from PROD-001 | Returned ID matches; stable core fields have expected types | Documented | Contract | Product details mutable |
| PROD-003 | P0 | `/products/:id` | GET | Get nonexistent product | Missing record represented as real | Dynamically safe nonexistent ID | No product with requested ID is represented as found; exact error awaits characterization | Conventional | Characterization | ID space and error shape may change |
| PROD-004 | P0 | `/products/add` | POST | Simulated valid product creation | Create simulation loses submitted data | Unique valid payload | New ID and submitted fields returned; no persistence assumed | Documented | Contract | Generated ID/catalog growth |
| PROD-005 | P1 | `/products/add` | POST | Simulated minimal product creation | Minimal documented example stops working | Title-only body | New ID and supplied title returned; no persistence assumed | Documented example | Contract | Validation minimum is not explicitly stated |
| PROD-006 | P0 | `/products/:id` | PUT | Simulated PUT | Update simulation loses requested values | Existing ID and unique payload | Same ID with submitted modification; no persistence assumed | Documented | Contract | Replacement versus merge unspecified |
| PROD-007 | P0 | `/products/:id` | PATCH | Simulated PATCH | Partial update simulation broken | Existing ID and unique value | Same ID with requested field modified; no persistence assumed | Documented | Contract | Untouched-field behavior unspecified |
| PROD-008 | P0 | `/products/:id` | DELETE | Simulated DELETE | Delete metadata broken | Existing ID | Same ID, `isDeleted: true`, parseable `deletedOn`; no persistence assumed | Documented | Contract | Timestamp precision/format |
| PROD-009 | P1 | `/products/:id` | PUT or PATCH selected for the experiment | Update nonexistent product | Update fabricates or mishandles missing target | Dynamically safe nonexistent ID | Capture whether rejected or simulated; do not prescribe status/body | Conventional | Characterization | Verb-specific undocumented behavior |
| PROD-010 | P1 | `/products/:id` | DELETE | Delete nonexistent product | Delete fabricates or mishandles missing target | Dynamically safe nonexistent ID | Capture behavior; do not prescribe status/body | Conventional | Characterization | Undocumented behavior |
| QUERY-001 | P0 | `/products/search?q=:term` | GET | Search with results | Search endpoint/envelope or matching semantics regress | Search seed from current data | Documented product envelope is coherent and non-empty; exact searchable fields and relevance relationship are asserted only after characterization | Documented envelope + Conventional matching hypothesis | Contract + Characterization | Matching algorithm undocumented and not yet observed |
| QUERY-002 | P0 | `/products/search?q=:term` | GET | Search without results | No-result handling is inconsistent | Run-unique improbable term | Characterize exact array and metadata behavior; do not guarantee emptiness from the term alone | Conventional | Characterization | Future data may match; shape undocumented for no results |
| QUERY-003 | P0 | `/products/category-list` | GET | Get category list and store a valid slug | Category tests use stale hardcoded data | None | Non-empty string array; store a returned slug | Documented | Contract | Category list may change |
| QUERY-004 | P0 | `/products/category/:slug` | GET | Filter using a valid category | Filter leaks other categories | Slug from QUERY-003 | Coherent envelope; every returned product category equals slug | Documented | Contract | Category may become empty during data change |
| QUERY-005 | P0 | `/products/category/:slug` | GET | Invalid category | Unknown category handled inconsistently | Unique invalid slug | Characterize status and empty/error shape; do not prescribe 404 | Conventional | Characterization | Undocumented response |
| QUERY-006 | P0 | `/products?limit=5&skip=0` | GET | First pagination page | Baseline page metadata wrong | At least ten products | Store IDs; array length > 0 and <= 5; `skip` 0 and `limit` 5 | Documented | Contract | Dataset can mutate between page calls |
| QUERY-007 | P0 | `/products?limit=5&skip=5` | GET | Second pagination page with no overlap | Skip ignored or pages duplicate | QUERY-006 IDs | `skip` 5 and `limit` 5; bounded unique IDs; no overlap with first page | Documented | Contract | Cross-request mutation/order instability |
| QUERY-008 | P1 | `/products?select=id,title,price` | GET | Field selection | Projection ignored or overinclusive | Products available | Requested fields present; no unexpected business fields beyond required identity | Documented | Contract | Server-required metadata may change |
| QUERY-009 | P1 | `/products?limit=0` | GET | limit=0 | Unlimited semantics regress | Stable response snapshot | All items returned and array length relates to response `total`; no fixed total | Documented | Contract | Large/mutable dataset |
| ERR-001 | P0 | `/products/not-a-number` | GET | Non-numeric product ID | Path parser returns unrelated record or crashes | None | No unrelated product; characterize exact status/body | Conventional | Characterization | Router behavior undocumented |
| ERR-002 | P0 | `/products/-1` | GET | Negative product ID | Negative ID returns unrelated record or crashes | None | No unrelated product; characterize exact status/body | Conventional | Characterization | Router behavior undocumented |
| ERR-003 | P0 | `/products/add` | POST | Malformed JSON | Parser accepts corrupt body or crashes | Truncated raw JSON with JSON content type | No successful simulated creation; characterize response | Conventional | Characterization | Gateway/parser response may change |
| ERR-004 | P0 | `/products/add` | POST | Unexpected product field types | Weak validation silently corrupts values | Valid JSON with wrong types | Record reject, coerce, or echo behavior before asserting it | Conventional | Characterization | Fake API may intentionally accept arbitrary types |
| ERR-005 | P1 | `/products/add` | POST | Empty create body | Empty body produces misleading creation | Empty JSON object | Record acceptance/rejection and returned shape; no prescribed status | Conventional | Characterization | Validation requirements undocumented |
| ERR-006 | P1 | `/products?limit=:bad&skip=:bad` | GET | Invalid pagination values | Bad inputs crash or coerce unpredictably | Separate negative and nonnumeric cases | Record reject/coerce/default behavior separately; no exact status | Conventional | Characterization | Parameter coercion undocumented |
| ERR-007 | P1 | Product endpoint and method selected after exploratory observation | To be selected after observation | Unsupported HTTP method selected only after observation | Router exposes unintended operation | First explore candidate methods; account for special infrastructure handling of HEAD and OPTIONS | After selecting a genuinely unsupported method, characterize status, headers, and body; do not prescribe 405 | Conventional | Characterization | HTTP infrastructure varies; method not yet selected |

## Coverage controls

- Each supplied P0 and P1 scenario maps to one unique ID. Split invalid-ID and two-page scenarios have distinct IDs for independently diagnosable requests.
- Invalid pagination exists only as ERR-006.
- PROD-009, PROD-010, and all ERR rows are characterization tests.
- No row assumes simulated create, update, or delete persists.
- No row hardcodes the global total, generated ID, catalog strings, category counts, token bytes, or undocumented exact status.

## Source links

- [Test route and generic pagination](https://dummyjson.com/docs)
- [Authentication endpoints](https://dummyjson.com/docs/auth)
- [Product endpoints and simulated writes](https://dummyjson.com/docs/products)
