# Test Matrix

`Documented` means the cited DummyJSON pages explicitly support the expectation. `Observed` means a dated live result exists in `api-observations.md`; it remains characterization rather than becoming contract. `Conventional` is only an experiment hypothesis and never supplies an invented exact status. Characterization rows require recorded evidence before strict status/body assertions are implemented.

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
| AUTH-004 | P0 | `/auth/login` | POST | Invalid password or invalid credentials | Invalid user might authenticate | Known-invalid credential pair | JSON 400 with invalid-credential semantics and no session | Observed (OBS-2026-07-31-001) | Characterization | Status/message remain undocumented |
| AUTH-005 | P1 | `/auth/login` | POST | Missing username | Incomplete credentials accepted | Password only | JSON 400 identifying required credentials | Observed (OBS-2026-07-31-002) | Characterization | Validation rules remain undocumented |
| AUTH-006 | P1 | `/auth/login` | POST | Missing password | Incomplete credentials accepted | Username only | JSON 400 identifying required credentials | Observed (OBS-2026-07-31-003) | Characterization | Validation rules remain undocumented |
| AUTH-007 | P0 | `/auth/me` | GET | Missing access token | Protected identity leaks | Authorization explicitly absent | JSON 401 requiring an access token, with no identity | Documented bearer requirement + Observed (OBS-2026-07-31-004) | Characterization | Failure status/body remain undocumented |
| AUTH-008 | P0 | `/auth/me` | GET | Malformed bearer token | Corrupt token accepted | Explicit malformed Bearer value | JSON 401 with invalid/expired-token semantics and no identity | Observed (OBS-2026-07-31-005) | Characterization | Failure status/body remain undocumented |
| AUTH-009 | P1 | `/auth/refresh` | POST | Invalid refresh token | Invalid token creates a session | Known-invalid token | JSON 403 with invalid-refresh semantics and no session | Observed (OBS-2026-07-31-006) | Characterization | Failure status/body remain undocumented |
| PROD-001 | P0 | `/products?limit=5&skip=0` | GET | List products using limit=5 and skip=0 and store an existing ID | Broken envelope/pagination blocks dependent tests | None | `products` is an array; length > 0 and <= 5; `skip` = 0; `limit` corresponds to 5; `total` is numeric and non-negative; store one ID; assert no exact total | Documented | Contract | Dataset may change between requests |
| PROD-002 | P0 | `/products/:id` | GET | Get existing product | Lookup returns wrong record | ID from PROD-001 | Returned ID matches; stable core fields have expected types | Documented | Contract | Product details mutable |
| PROD-003 | P0 | `/products/:id` | GET | Get nonexistent product | Missing record represented as real | Configured safely large nonexistent ID | JSON 404 identifies the requested ID as not found | Observed (OBS-2026-07-31-007) | Characterization | ID space and error shape may change |
| PROD-004 | P0 | `/products/add` | POST | Simulated valid product creation | Create simulation loses submitted data | Unique valid payload | New ID and submitted fields returned; no persistence assumed | Documented | Contract | Generated ID/catalog growth |
| PROD-005 | P1 | `/products/add` | POST | Simulated minimal product creation | Minimal documented example stops working | Title-only body | New ID and supplied title returned; no persistence assumed | Documented example | Contract | Validation minimum is not explicitly stated |
| PROD-006 | P0 | `/products/:id` | PUT | Simulated PUT | Update simulation loses requested values | Existing ID and unique payload | Same ID with submitted modification; no persistence assumed | Documented | Contract | Replacement versus merge unspecified |
| PROD-007 | P0 | `/products/:id` | PATCH | Simulated PATCH | Partial update simulation broken | Existing ID and unique value | Same ID with requested field modified; no persistence assumed | Documented | Contract | Untouched-field behavior unspecified |
| PROD-008 | P0 | `/products/:id` | DELETE | Simulated DELETE | Delete metadata broken | Existing ID | Same ID, `isDeleted: true`, parseable `deletedOn`; no persistence assumed | Documented | Contract | Timestamp precision/format |
| PROD-009 | P1 | `/products/:id` | PUT | Update nonexistent product | Update fabricates or mishandles missing target | Configured safely large nonexistent ID | JSON 404 identifies the requested ID as not found | Observed (OBS-2026-07-31-012) | Characterization | Undocumented behavior may change |
| PROD-010 | P1 | `/products/:id` | DELETE | Delete nonexistent product | Delete fabricates or mishandles missing target | Configured safely large nonexistent ID | JSON 404 identifies the requested ID as not found | Observed (OBS-2026-07-31-014) | Characterization | Undocumented behavior may change |
| QUERY-001 | P0 | `/products/search?q=:term` | GET | Search with results | Search endpoint/envelope or matching semantics regress | Configured search query | Documented product envelope is coherent and non-empty; at least one reasonable searchable field matches based on OBS-2026-07-31-015 | Documented envelope + Observed matching behavior | Contract + Characterization | Matching algorithm remains undocumented and mutable |
| QUERY-002 | P0 | `/products/search?q=:term` | GET | Search without results | No-result handling is inconsistent | Run-unique improbable term | JSON 200 with empty products and zero metadata | Observed (OBS-2026-07-31-016) | Characterization | Future data may match; no-result shape remains undocumented |
| QUERY-003 | P0 | `/products/category-list` | GET | Get category list and store a valid slug | Category tests use stale hardcoded data | None | Non-empty string array; store a returned slug | Documented | Contract | Category list may change |
| QUERY-004 | P0 | `/products/category/:slug` | GET | Filter using a valid category | Filter leaks other categories | Slug from QUERY-003 | Coherent envelope; every returned product category equals slug | Documented | Contract | Category may become empty during data change |
| QUERY-005 | P0 | `/products/category/:slug` | GET | Invalid category | Unknown category handled inconsistently | Unique invalid slug | JSON 200 with empty products and zero metadata | Observed (OBS-2026-07-31-019) | Characterization | Undocumented response may change |
| QUERY-006 | P0 | `/products?limit=5&skip=0` | GET | First pagination page | Baseline page metadata wrong | At least ten products | Store IDs; array length > 0 and <= 5; `skip` 0 and `limit` 5 | Documented | Contract | Dataset can mutate between page calls |
| QUERY-007 | P0 | `/products?limit=5&skip=5` | GET | Second pagination page with no overlap | Skip ignored or pages duplicate | QUERY-006 IDs | `skip` 5 and `limit` 5; bounded unique IDs; no overlap with first page | Documented | Contract | Cross-request mutation/order instability |
| QUERY-008 | P1 | `/products?select=id,title,price` | GET | Field selection | Projection ignored or overinclusive | Products available | Requested fields present; no unexpected business fields beyond required identity | Documented | Contract | Server-required metadata may change |
| QUERY-009 | P1 | `/products?limit=0` | GET | limit=0 | Unlimited semantics regress | Stable response snapshot | All items returned and array length relates to response `total`; no fixed total | Documented | Contract | Large/mutable dataset |
| ERR-001 | P0 | `/products/not-a-number` | GET | Non-numeric product ID | Path parser returns unrelated record or crashes | None | JSON 404 identifies the non-numeric ID as not found | Observed (OBS-2026-07-31-023) | Characterization | Router behavior remains undocumented |
| ERR-002 | P0 | `/products/-1` | GET | Negative product ID | Negative ID returns unrelated record or crashes | None | JSON 404 identifies the negative ID as not found | Observed (OBS-2026-07-31-024) | Characterization | Router behavior remains undocumented |
| ERR-003 | P0 | `/products/add` | POST | Malformed JSON | Parser accepts corrupt body or crashes | Truncated raw JSON with JSON content type | JSON 400 with non-empty parser-error semantics and no created ID | Observed (OBS-2026-07-31-025) | Characterization | Parser wording may change |
| ERR-004 | P0 | `/products/add` | POST | Unexpected product field types | Weak validation silently corrupts values | Valid JSON with numeric title and string price | Simulated create accepts and echoes the original wrong types with a generated ID | Observed (OBS-2026-07-31-026) | Characterization | Permissive fake API may change validation |
| ERR-005 | P1 | `/products/add` | POST | Empty create body | Empty body produces misleading creation | Empty JSON object | Simulated create accepts the object and returns only a generated ID | Observed (OBS-2026-07-31-027) | Characterization | Validation requirements remain undocumented |
| ERR-006 | P1 | `/products?limit=-5&skip=-5` | GET | Negative pagination values | Bad inputs crash or coerce unpredictably | Negative limit and skip in one request | JSON 400 reports limit must be positive; skip is not independently characterized because limit fails first | Observed (OBS-2026-07-31-028) | Characterization | Validation order and negative-skip behavior remain undocumented |
| ERR-007 | P1 | `/products/1` | POST | Unsupported POST to a product resource | Router exposes unintended operation | None | Characterize the observed HTML 404 route response; do not prescribe conventional 405 behavior | Observed (OBS-2026-07-31-029) | Characterization | HTTP infrastructure and response representation may change |

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
