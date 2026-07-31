# DummyJSON API Quality Suite

## Assessment objective

This repository contains a professional Postman and Newman API test suite for a QA Engineer technical assessment. It provides risk-based coverage of the implemented DummyJSON health, authentication, product, search, category, pagination, projection, and error-handling scenarios.

The suite emphasizes meaningful response semantics, safe runtime state, documented execution dependencies, and a clear distinction between official API contracts and experimentally observed behavior.

## Technology stack

- Postman Collection Format v2.1
- Postman environments and collection variables
- Newman 6, installed as a local npm development dependency
- Node.js 16 or later
- npm scripts for local, folder-specific, full-suite, and JUnit-reporting execution
- JSON and JavaScript assertions using the Postman sandbox API and Chai syntax

## Repository structure

```text
.
├── postman/
│   ├── DummyJSON-API-Quality-Suite.postman_collection.json
│   └── DummyJSON-Local.postman_environment.json
├── docs/
│   ├── api-observations.md
│   ├── test-matrix.md
│   └── test-strategy.md
├── reports/
│   └── .gitkeep
├── AGENTS.md
├── CLAUDE.md
├── PLAN.md
├── prompts.txt
├── package.json
├── package-lock.json
└── README.md
```

Generated Newman reports and `node_modules` are ignored by Git.

## Risk-based test strategy

The suite prioritizes failures that would most seriously undermine confidence in an API integration:

- Service availability and basic response integrity
- Authentication, current-user identity, and refresh-token workflows
- Product discovery and existing-resource access
- Correct handling of DummyJSON's simulated create, update, and delete operations
- Search, category filtering, field projection, and adjacent-page behavior
- Authentication failures, invalid identifiers, malformed input, permissive validation, and unsupported routing

Assertions avoid exact catalog totals, mutable product names, fixed category counts, and response-time thresholds. Dynamic discovery and relational comparisons are used where public data can change.

## Implemented scope

The collection contains 36 requests across five top-level areas:

- `GET /test` health check
- Authentication login, current-user, refresh, and six negative cases
- Product list, single-product read, simulated create, PUT, PATCH, and DELETE
- Nonexistent product read, update, and delete characterization
- Product search with results and a generated no-result query
- Category-list discovery, valid category filtering, and nonexistent category characterization
- Two-page pagination comparison, field selection, and `limit=0`
- Invalid product IDs, malformed JSON, unexpected field types, empty creation, negative pagination, and unsupported product-resource POST behavior

The traceability IDs are maintained in [docs/test-matrix.md](docs/test-matrix.md), and dated evidence for undocumented behavior is recorded in [docs/api-observations.md](docs/api-observations.md).

## Deliberately excluded scope

The implemented suite does not cover unrelated DummyJSON resources such as users, carts, posts, recipes, comments, todos, or quotes. It also excludes sorting, artificial response delays, cookie-first authentication, token-expiry waiting, cryptographic JWT signature verification, broad fuzzing, load or performance testing, and security penetration testing.

No persistence guarantee is tested for simulated product mutations.

## Collection organization

```text
00 - Health Check
01 - Authentication
├── Positive
└── Negative
02 - Products CRUD
├── Read
├── Create
├── Update
└── Delete
03 - Search Filtering and Pagination
├── Search
├── Categories
└── Pagination
04 - Error Handling
```

The numeric prefixes provide a clear full-run order. The supported npm folder commands execute complete top-level areas, including any discovery request required by their nested children.

## Variable strategy

### Environment variables

The committed local environment contains:

| Variable | Purpose |
|---|---|
| `baseUrl` | DummyJSON base URL, defaulting to `https://dummyjson.com` |
| `username` | Documented public DummyJSON fixture username |
| `password` | Documented public DummyJSON fixture password |

The public fixture credentials are included for reproducible assessment execution. Private credentials and secrets must not be committed.

Access and refresh tokens are deliberately absent from the environment.

### Collection variables

| Variable | Purpose |
|---|---|
| `accessToken`, `refreshToken` | Runtime authentication state; committed values are empty |
| `authenticatedUserId` | Runtime identity captured after validated login |
| `existingProductId` | Existing ID discovered from the product list |
| `runId` | Runtime uniqueness seed |
| `testProductTitle`, `testProductPrice` | Generated simulated-create values |
| `validCategory` | Category slug discovered from the API |
| `pageOneIds` | JSON-encoded first-page IDs used for overlap comparison |
| `pageLimit` | Pagination size, default `5` |
| `searchQuery` | Positive search term, default `phone` |
| `nonexistentProductId` | Configurable characterization ID, default `99999999` |

Runtime tokens and dependent state are written only after the relevant response has passed defensive validation. Newman runtime changes are not written back to the committed collection file.

## Authentication and JWT validation

The positive authentication flow executes in this order:

1. Log in with the documented public fixture credentials.
2. Validate identity fields and capture the authenticated user ID and tokens.
3. Call `/auth/me` with the captured access token and compare the returned identity.
4. Exchange the captured refresh token and replace runtime tokens only after the refresh response is validated.

Negative authentication requests use request-specific missing or invalid values. They explicitly avoid overwriting the valid runtime session and neutralize cookie fallback where necessary.

JWT checks are structural only. The suite verifies that each token has exactly three non-empty segments, Base64URL-decodes the header and payload, and confirms that both decode to JSON objects. It checks numeric `iat` and `exp` claims returned by DummyJSON and verifies `exp > iat`.

The suite does **not** cryptographically verify JWT signatures and does not claim token authenticity from structural decoding.

## Products CRUD and simulation limitations

DummyJSON product mutations are simulations:

- `POST /products/add` returns a generated product representation but does not persist it.
- PUT and PATCH return simulated modifications but do not update server data.
- DELETE returns deletion metadata but does not remove the product.

The suite validates returned IDs, submitted values, and deletion metadata without creating a false persistent lifecycle. It never uses a simulated create ID as though it were a real stored resource and does not issue follow-up reads expecting mutation persistence.

## Search, filtering, and pagination

Search coverage validates the documented response envelope and requires a non-empty result for the configured positive query. Matching-field behavior is treated as characterization: at least one reasonable searchable field in at least one result must correspond to the query, without requiring every field or result to match.

A unique, unlikely query is generated for the no-result characterization. Category coverage discovers a valid slug from `/products/category-list`, then verifies that every returned product belongs to that runtime category. A generated nonexistent category characterizes the observed empty response.

Pagination uses `pageLimit=5`. The first request stores product IDs as JSON; the second requests the adjacent page and verifies bounded size, unique IDs within the page, and no overlap with the first page. Because DummyJSON is a mutable public service, this comparison assumes a sufficiently stable snapshot across the two requests.

Field projection verifies only `title`, `price`, and the API-provided `id` inside product objects. The `limit=0` test validates the documented all-items behavior through response relationships rather than an exact global total.

## Error-handling philosophy

Undocumented failure behavior is explored before strict assertions are written. Characterization tests validate status, response representation, and useful semantics without presenting conventional REST expectations as official DummyJSON contracts.

The suite records actual permissive behavior honestly. For example, the simulated create endpoint currently accepts unexpected product field types and an empty object. Those tests verify permissive echo behavior and API limitations; they are not disguised rejection tests.

## Contract and characterization tests

- **Contract tests** assert behavior explicitly supported by the official DummyJSON documentation.
- **Characterization tests** assert dated, experimentally observed behavior where the documentation is silent.
- **Conventional expectations** may motivate an experiment but are not used as contract assertions without evidence.

Observed behavior remains characterization even when it is stable. The oracle hierarchy and evidence are maintained in the test strategy, matrix, and API observations.

## Installation

Prerequisites:

- Node.js 16 or later
- npm

Install the locked local dependencies from the repository root:

```bash
npm ci
```

Newman is installed locally through `devDependencies`; no global Newman installation is required.

## Local execution

All commands below run against the committed local environment.

### Folder-specific execution

```bash
npm run test:health
npm run test:auth
npm run test:products
npm run test:query
npm run test:errors
```

These commands execute the corresponding complete top-level collection folder. Some nested child folders consume runtime values created by earlier requests in their parent folder and are not intended as independent entry points without those prerequisites.

### Full Newman execution

```bash
npm test
```

This runs the entire collection in its defined order and does not suppress Newman failure exit codes.

### JUnit report execution

```bash
npm run test:report
```

This runs the full suite with CLI and JUnit reporters and writes:

```text
reports/newman-results.xml
```

The generated report is ignored by Git. The repository provides this CI-compatible execution command but does not include a provider-specific CI workflow.

## Importing into Postman

1. Open Postman and select **Import**.
2. Import `postman/DummyJSON-API-Quality-Suite.postman_collection.json`.
3. Import `postman/DummyJSON-Local.postman_environment.json`.
4. Select the **DummyJSON Local** environment.
5. Run the complete collection or one of the supported top-level folders.

Tokens should remain empty in exported or committed files. If the collection is exported after an interactive run, review variable current values before saving it to the repository.

## AI-assisted development workflow

AI supported repository analysis, test design, implementation, review, debugging, documentation, and command execution. The human selected the assessment scope, priorities, constraints, and acceptance criteria, and reviewed reported diffs and test results throughout the work.

AI suggestions were not accepted blindly. Assertions were checked against the official documentation or controlled observations. When an assertion failed or behavior was uncertain, the cause was investigated before changing the test. Undocumented behavior was explicitly classified as characterization rather than guessed from REST conventions.

The append-only `prompts.txt` file contains the interaction trail, including the instructions received and summaries of the work performed.

## Human decisions and oversight

Human direction established the P0/P1 scenarios, approved the public fixture credential strategy, prohibited persistent-CRUD assumptions and cryptographic JWT claims, constrained scope to the relevant DummyJSON resources, and required incremental and complete Newman validation.

Human review remained the decision boundary for scope and acceptance. AI-assisted implementation did not replace engineering judgment, source verification, diff review, or interpretation of test evidence.

## Known limitations

- DummyJSON is a shared, mutable public API; availability, latency, fixture data, totals, categories, and undocumented responses can change independently of this repository.
- Characterization behavior was observed at specific timestamps and may drift without a documentation update.
- Adjacent pagination requests can straddle an upstream data change.
- The configured nonexistent product ID could eventually become valid if the public dataset grows substantially.
- Negative pagination currently establishes the combined request's invalid-limit response; negative `skip` is not independently characterized because limit validation occurs first.
- Nested mutation and second-step query folders depend on discovery performed earlier in their top-level parent folder.
- JWT checks establish parseable structure and claim relationships, not cryptographic authenticity.

## Trade-offs

- Dynamic discovery reduces brittle fixture coupling but introduces explicit request-order dependencies.
- Relational assertions tolerate mutable public data but intentionally avoid full response snapshots.
- Defensive checks are repeated before collection-variable writes because Postman continues executing later test blocks after an earlier assertion failure.
- Characterization assertions provide regression visibility for undocumented behavior while accepting that upstream implementation changes may require deliberate re-characterization.
- Public fixture credentials improve reproducibility but are suitable only for this documented demonstration service.

## Potential future improvements

The following are possible extensions, not current capabilities:

- Add a provider-specific CI workflow that invokes the existing Newman execution command.
- Add a provider-specific CI workflow that invokes the existing `test:ci` command.
- Add automated schema-validation tooling as a repository script.
- Re-characterize invalid pagination variants independently if broader edge coverage is authorized.
- Add controlled observation repetition to distinguish stable behavior from one-off public-service variance.
- Add documented prerequisite tooling for intentionally running dependent nested folders in isolation.

Any extension should preserve the existing scope boundaries, oracle hierarchy, runtime secret controls, and simulated-CRUD model.
