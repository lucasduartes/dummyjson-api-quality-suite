# Test Strategy

## Purpose and quality risks

The suite will provide fast evidence that DummyJSON is reachable and that its documented authentication and product workflows remain usable. Priority is based on user impact and dependency: authentication and basic reads are P0; simulated writes, core discovery, and error isolation are also P0 because they expose the most consequential integration failures. Useful options and secondary negative cases are P1.

The dominant risks are broken authentication chains, incorrect product shapes, wrong filtering/paging behavior, false assumptions about simulated writes, and misleading green tests caused by weak assertions.

## Scope

In scope: `/test`; `/auth/login`, `/auth/me`, `/auth/refresh`; product list/read/add/update/delete; product search, category filtering, pagination, and field selection; and the specified negative inputs.

Out of scope: users as a tested resource, carts, posts, recipes, comments, todos, quotes, IP utilities, sorting, artificial delay, cookies as the primary auth mechanism, token-expiry waiting, JWT signature validation, performance/load testing, penetration testing, broad fuzzing, persistence guarantees, CI configuration, and reporting infrastructure. Credentials may refer to a documented DummyJSON user, but no user-resource tests are added.

## Oracle hierarchy

1. **Documented contract:** explicit behavior in the cited DummyJSON pages. These are contract tests.
2. **Observed behavior:** a dated result from a controlled live request, recorded in `api-observations.md`. Undocumented behavior remains a characterization test even after observation.
3. **Conventional expectation:** a general protocol expectation used only to choose an experiment, never to invent an exact DummyJSON status code.

If documentation is silent, the first implementation run records status, headers, and sanitized error shape. A stable, useful observation may then become a narrowly labeled characterization assertion. Contract assertions must not be weakened to match an accidental observation.

## Authentication strategy

- Supply username/password at runtime; never commit live tokens or private credentials.
- Valid login asserts a successful response, expected user identity relationship, and non-empty string access/refresh tokens. It stores tokens only in runtime scope.
- `/auth/me` depends on valid login and sends `Authorization: Bearer {{accessToken}}` explicitly.
- Refresh depends on valid login and uses the returned refresh token in JSON. It asserts usable replacement token strings, not token inequality, signature validity, or expiry timing.
- Missing and malformed authentication explicitly override inherited authorization to prevent a false positive from a valid collection token.
- Invalid/missing credential and invalid-refresh cases are independent characterization tests where exact negative status/body is undocumented.
- JWTs may receive lightweight shape checks (three non-empty dot-separated segments), but no cryptographic verification and no claim about claims beyond documented response fields.

## Products strategy

### Read and contract shape

List tests assert JSON, a `products` array, numeric non-negative `total`, `skip`, and `limit`, and internally coherent lengths. They do not assert the example total. A known ID may bootstrap discovery, but the suite should capture an existing ID from the list to reduce dependence on a particular record.

Single-product reads assert identity using the captured ID and stable field types, not exact mutable content. A nonexistent ID is chosen dynamically above the returned IDs/total where practical; exact error status is characterized because the docs do not specify it.

### Simulated CRUD

- Create asserts that submitted fields are echoed and an ID is returned. Minimal create checks only behavior supported by the documentation's title-only example.
- PUT and PATCH operate on an existing captured product ID and use unique runtime values. PUT must not be interpreted as full replacement unless documented/observed; both verbs are documented as simulated updates returning modified data.
- DELETE asserts the original ID, `isDeleted: true`, and an ISO-like `deletedOn` value.
- No request depends on a created ID being readable later. No assertion expects an update or delete to persist. An optional follow-up GET of the original existing product demonstrates non-persistence and expects the baseline value to remain, avoiding mutation assumptions.

## Search, category, pagination, and projection

- Derive a search term from a product discovered earlier when possible. Assert at least one result and that results satisfy a defensible case-insensitive match over documented searchable text only after confirming behavior.
- Use a deliberately improbable run-specific term for no-results; assert an empty array and coherent zero metadata without fixing an HTTP status absent documentation.
- Obtain a valid category slug from `/products/categories` or `/products/category-list`, then assert every returned product has that category.
- Invalid category uses an improbable slug and characterizes response status/shape; do not invent 404.
- Request two equal-sized pages with different `skip` values. Assert echoed paging metadata, page-size bounds, unique IDs within each page, and no overlapping IDs across adjacent pages. Avoid asserting global ordering unless documented.
- `limit=0` asserts the documented removal of the limit through relational properties (`products.length === total` when the response is a complete snapshot), not a hardcoded total.
- Field selection asserts requested fields and identity behavior while allowing server-required fields such as `id`; it should not compare entire objects.
- Invalid pagination parameters are separate characterization rows because accepted formats and error statuses are undocumented.

## Error-handling strategy

Negative tests first prove that the request reached DummyJSON and returned parseable or intentionally non-JSON error information. Exact status codes and message text are asserted only when documented or dated as observed. Otherwise assertions focus on safety: no false authenticated identity, no successful mutation claim for malformed input, coherent response type, and absence of server-side 5xx where a conventional client-error expectation is being characterized. Unsupported methods, malformed JSON, unexpected field types, and invalid IDs remain characterization tests.

## Test data and variables

- `baseUrl`: collection/environment value, defaulting to `https://dummyjson.com`.
- `username`, `password`: runtime/environment inputs; assessment-safe public demo defaults may be considered later, but never tokens.
- `accessToken`, `refreshToken`: empty in committed files; runtime only.
- `existingProductId`, `baselineProductTitle`, `validCategory`: derived during the run.
- `nonexistentProductId`, `uniqueSuffix`, `searchHitTerm`, `searchMissTerm`: generated/derived per run.
- Paging inputs (`pageSize`, `pageOneSkip`, `pageTwoSkip`) use small deterministic values.

Prefer collection variables for non-secret defaults and local/runtime variables for transient data. Environment/current values must not be exported with secrets. Avoid global variables to prevent cross-collection contamination.

## Execution dependencies

```text
Health
  -> Login -> accessToken -> Current user
           -> refreshToken -> Refresh
  -> Product list -> existingProductId/baseline fields
                  -> Read, PUT, PATCH, DELETE
                  -> Search hit seed
                  -> Pagination comparison
  -> Category list -> validCategory -> Valid category
```

Negative tests should construct their own invalid inputs and not depend on positive tokens except where deliberately corrupting a known token. Folder-level setup must make focused Newman runs reproducible.

## Public API instability and anti-flakiness controls

DummyJSON is a shared, mutable public service: catalog size/content, demo credentials, availability, throttling, latency, or undocumented error behavior can change without this repository changing. Mitigations are dynamic discovery, relational assertions, small request volume, clear connectivity failures, no retries that hide defects, and dated observation records.

False-positive risks include checking only status, allowing empty arrays in positive cases, stale runtime tokens, inherited auth on negative tests, scripts that skip assertions after parse errors, and comparing a value to itself. Every request should assert content type/parseability where appropriate, core semantics, and test preconditions explicitly.

Brittleness risks include exact error text, full-response snapshots, fixed totals, fixed catalog strings, timestamps with exact values, response-time thresholds on the public internet, token equality/inequality, and undocumented ordering. Prefer type/relationship checks and narrow schemas with required stable fields.

## Newman validation approach

The future collection will use Postman Collection v2.1.0. Newman requires Node.js 16+ according to the cited installation page and runs exported collections with `newman run`; environments are supplied with `-e`. After each meaningful implementation change, run the affected folder, then run the complete collection before completion. Generated reports will remain uncommitted.
