# Test Matrix

`Documented` means the cited DummyJSON pages explicitly support the expectation. `Observed` is reserved for behavior captured in `api-observations.md`. `Conventional` supplies only a hypothesis for undocumented behavior; it never supplies an invented exact status. Characterization rows must be baselined before strict status/body assertions are implemented.

| Test ID | Priority | Endpoint | Method | Scenario | Risk addressed | Preconditions | Expected result | Source of expectation | Classification | Potential brittleness |
|---|---|---|---|---|---|---|---|---|---|---|
| HC-001 | P0 | `/test` | GET | Health check | Outage makes all functional results misleading | Base URL reachable | JSON reports `status: ok` and request method `GET` | Documented | Contract | Low; service/network availability |
| AUTH-001 | P0 | `/auth/login` | POST | Valid login | Users cannot establish a session | Valid runtime credentials | User identity fields plus non-empty access and refresh token strings | Documented | Contract | Demo credentials/data may change |
| AUTH-002 | P0 | `/auth/me` | GET | Current authenticated user | Token is unusable or mapped to wrong user | AUTH-001 access token | Returned user identity matches login identity | Documented | Contract | Demo user mutation |
| AUTH-003 | P0 | `/auth/refresh` | POST | Refresh token | Session cannot be extended | AUTH-001 refresh token | Non-empty access and refresh token strings returned | Documented | Contract | Token contents/rotation policy |
| AUTH-004 | P0 | `/auth/login` | POST | Invalid credentials | Invalid users might be authenticated | Known-invalid username/password | Authentication is not granted; baseline exact status and error shape | Conventional | Characterization | Exact status/message undocumented |
| AUTH-005 | P0 | `/auth/me` | GET | Missing authentication | Protected identity leaks without a token | Authorization explicitly absent | No authenticated user is returned; baseline exact response | Documented | Characterization | Required bearer documented, failure status not documented |
| AUTH-006 | P0 | `/auth/me` | GET | Malformed authentication | Parser accepts corrupt bearer values | Explicit malformed Bearer token | No authenticated user is returned; baseline exact response | Conventional | Characterization | Exact status/message undocumented |
| AUTH-007 | P1 | `/auth/login` | POST | Missing username | Incomplete credentials accepted | Password only | Authentication not granted; characterize response | Conventional | Characterization | Validation rules/status undocumented |
| AUTH-008 | P1 | `/auth/login` | POST | Missing password | Incomplete credentials accepted | Username only | Authentication not granted; characterize response | Conventional | Characterization | Validation rules/status undocumented |
| AUTH-009 | P1 | `/auth/refresh` | POST | Invalid refresh token | Forged/stale refresh accepted | Known-invalid token | No usable authenticated session is granted; characterize response | Conventional | Characterization | Exact status/message undocumented |
| PROD-001 | P0 | `/products` | GET | List products | Catalog unavailable or envelope broken | None | JSON has products array and coherent numeric `total`, `skip`, `limit`; default page bounded by documented 30 | Documented | Contract | Default size could be revised in docs |
| PROD-002 | P0 | `/products/:id` | GET | Read existing product | Lookup returns wrong/malformed record | Existing ID captured from PROD-001 | Product ID matches and stable core fields have expected types | Documented | Contract | Product details mutable |
| PROD-003 | P0 | `/products/:id` | GET | Read nonexistent product | Missing record misrepresented as real | Dynamically safe nonexistent ID | No product with requested ID is represented as found; characterize exact error | Conventional | Characterization | Exact status/body undocumented; ID space may grow |
| PROD-004 | P0 | `/products/add` | POST | Simulated create | Create echo/ID simulation broken | Unique valid payload | Returned object has a new ID and echoes submitted fields; no persistence assumed | Documented | Contract | Generated ID value/catalog growth |
| PROD-005 | P0 | `/products/:id` | PUT | Simulated PUT | Update simulation loses requested values | Existing ID and unique payload | Same ID returned with submitted modification; persistence not asserted | Documented | Contract | PUT merge/replacement semantics not fully specified |
| PROD-006 | P0 | `/products/:id` | PATCH | Simulated PATCH | Partial update simulation broken | Existing ID and unique field value | Same ID returned with requested field modified; persistence not asserted | Documented | Contract | Unspecified handling of untouched fields |
| PROD-007 | P0 | `/products/:id` | DELETE | Simulated delete | Delete simulation metadata broken | Existing ID | Same product ID, `isDeleted: true`, parseable `deletedOn`; record persistence not assumed | Documented | Contract | Timestamp formatting precision |
| PROD-008 | P1 | `/products/add` | POST | Minimal create payload | Undocumented mandatory fields overconstrained | Title-only body | New simulated product includes ID and supplied title | Documented | Contract | Example implies minimality but does not state validation rules |
| SRCH-001 | P0 | `/products/search?q=:term` | GET | Search with results | Search misses discoverable products | Term derived from current product data | Non-empty coherent product envelope; results relate to query under baselined matching behavior | Documented | Contract | Search matching fields/algorithm undocumented |
| SRCH-002 | P0 | `/products/search?q=:term` | GET | Search without results | Empty search response malformed or false hits | Run-unique improbable term | Empty products array with coherent zero-result metadata | Documented | Contract | Future catalog could coincidentally match |
| CAT-001 | P0 | `/products/category/:slug` | GET | Valid category | Filter leaks other categories | Slug obtained from category endpoint | Coherent envelope; each result category equals slug | Documented | Contract | Empty category if public data changes |
| CAT-002 | P0 | `/products/category/:slug` | GET | Invalid category | Unknown categories handled inconsistently | Run-unique invalid slug | No products for the invalid category; characterize status/envelope | Conventional | Characterization | Exact status and empty/error shape undocumented |
| PAGE-001 | P0 | `/products?limit=:n&skip=:s` | GET | Compare two pages | Skip ignored, duplicate pages, metadata wrong | At least `2n` products; small n | Correct echoed paging metadata, bounded sizes, unique IDs, no cross-page overlap | Documented | Contract | Dataset changes during two requests; ordering not guaranteed |
| PAGE-002 | P1 | `/products?select=id,title,price` | GET | Field selection | Projection ignored or removes identity | Products available | Items contain requested fields and no unexpected business fields beyond server-required identity | Documented | Contract | Server may add mandatory metadata |
| PAGE-003 | P1 | `/products?limit=0` | GET | Unlimited results | Special limit semantics regress | Stable response snapshot | All items returned; array length relates to response `total`, without fixed total | Documented | Contract | Large/mutable dataset during request |
| PAGE-004 | P1 | `/products?limit=:bad&skip=:bad` | GET | Invalid pagination parameters | Bad inputs crash/coerce unpredictably | Table of negative/non-numeric values | Capture safe handling and coherent response; no invented exact status | Conventional | Characterization | Coercion and exact errors undocumented |
| ERR-001 | P0 | `/products/:id` | GET | Invalid product IDs | Path parsing causes false lookup/server error | Cases: zero, negative, nonnumeric, oversized | No unrelated product is returned; record each case's exact behavior | Conventional | Characterization | Different invalid forms may legitimately differ |
| ERR-002 | P0 | `/products/add` | POST | Malformed JSON | Parser accepts corrupt payload or crashes | Invalid raw JSON; JSON content type | No successful simulated creation; characterize status/body and flag 5xx | Conventional | Characterization | Gateway/parser response may change |
| ERR-003 | P0 | `/products/add` | POST | Unexpected field types | Weak validation silently corrupts data | Valid JSON with wrong types | Record whether values are rejected or echoed; assert only baselined safe behavior | Conventional | Characterization | Dummy API may intentionally accept arbitrary types |
| ERR-004 | P1 | `/products/:id` | Unsupported (for example OPTIONS or HEAD after baseline) | Unsupported method | Router exposes unintended operation | Existing ID; method chosen after observation | Characterize allowed/status/body; do not prescribe 405 | Conventional | Characterization | HTTP infrastructure may handle method specially |

## Coverage notes

- `ERR-001` is one matrix entry with multiple data cases so zero, negative, nonnumeric, and oversized IDs remain separately visible in Newman iteration names.
- `PAGE-004` likewise uses distinct data cases; no single coercion result is generalized to all invalid parameters.
- No row assumes that create/update/delete affects a later GET.
- No row hardcodes the global `total`, current generated create ID, product titles, category counts, or token bytes.

## Source links

- [Test route and generic pagination](https://dummyjson.com/docs)
- [Authentication endpoints](https://dummyjson.com/docs/auth)
- [Product endpoints and simulated writes](https://dummyjson.com/docs/products)
