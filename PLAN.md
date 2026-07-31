# DummyJSON Postman Quality Suite Plan

## Objective

Build a reviewable Postman/Newman suite for the documented DummyJSON health, authentication, and product capabilities. This phase defines the approach only; it does not implement a collection, environment, package, CI workflow, or generated report.

## Assessment interpretation

### Explicit requirements

- Cover every supplied P0 and P1 scenario at its assigned priority.
- Use DummyJSON documentation as the API contract and Postman Collection v2.1.0 as the future collection format.
- Keep documented expectations distinct from observed or conventional expectations.
- Treat undocumented edge behavior as characterization, without inventing status codes.
- Model product POST, PUT, PATCH, and DELETE as simulations that do not persist.
- Avoid fixed global totals, JWT signature verification, expiry waiting, unrelated resources, load testing, and penetration testing.
- Make the eventual suite runnable by folder and as a complete collection with Newman.

### Implicit evaluation criteria

- Traceability from requirement to named test and source.
- Sound risk prioritization, not merely endpoint coverage.
- Deterministic execution, safe token handling, and understandable variable scope.
- Assertions that detect real regressions without binding to mutable sample records.
- Clear request names, focused tests, useful failure messages, and maintainable folder structure.
- Honest handling of gaps in the published contract.
- Evidence of incremental and full-suite validation.

## Proposed collection structure

- `00 - Health Check`
- `01 - Authentication`
  - `Positive`
  - `Negative`
- `02 - Products CRUD`
  - `Read`
  - `Create`
  - `Update`
  - `Delete`
- `03 - Search Filtering and Pagination`
  - `Search`
  - `Categories`
  - `Pagination`
- `04 - Error Handling`

The numeric prefixes make top-level execution order explicit. A nested folder run must execute the setup request it owns or receive its documented prerequisites as inputs; otherwise it must fail early with a clear missing-variable message. `01 - Authentication/Positive` owns valid-token creation, `02 - Products CRUD/Read` owns existing-product discovery, and `03 - Search Filtering and Pagination/Categories` owns valid-category discovery. The complete collection runs top to bottom in the structure above.

## Delivery gates and implementation sequence

1. Freeze traceability: review `docs/test-matrix.md`, resolve any assessment ambiguity, and record any live observations separately.
2. Scaffold a schema-valid Postman v2.1 collection and a safe example environment. The documented public DummyJSON fixture username/password may be committed for reproducibility. Access tokens, refresh tokens, private credentials, and secrets remain runtime-only and empty in committed files.
3. Add collection-level helpers and variables: base URL, fixture credentials, response parsing, structural checks, and cleanup of transient tokens.
4. Implement and run `00 - Health Check`; it separates connectivity/service outage from functional failures.
5. Implement valid login, capture tokens, then current-user and refresh flows. Run `01 - Authentication/Positive`.
6. Implement authentication negatives independently so malformed or missing tokens cannot reuse a valid inherited header. Run `01 - Authentication/Negative`, then its parent folder.
7. Implement product list and dynamic fixture discovery, then single-product reads. Run `02 - Products CRUD/Read`.
8. Implement simulated create, PUT, PATCH, and DELETE. Assert echoed changes and simulation metadata; verify non-persistence only against the original existing product, never against a generated create ID.
9. Implement and run the `Search`, `Categories`, and `Pagination` children of `03 - Search Filtering and Pagination`, using relational assertions rather than fixed catalog values.
10. Implement undocumented negative-input characterization cases in `04 - Error Handling`. Baseline behavior must be recorded in `docs/api-observations.md` before pinning any observed status or error shape.
11. Validate the collection against the Postman v2.1 schema, run each changed nested folder, its affected parent folder, then the complete collection. Review for secret leakage and false-positive patterns.
12. Add package/CI/reporting artifacts only in a later explicitly authorized phase.

## Completion criteria for the future implementation

- Every matrix row maps to at least one request/test and is runnable.
- All P0 folders and then the full suite pass against a documented or explicitly baselined expectation.
- Documented public DummyJSON fixture credentials may be committed in the example environment; access tokens, refresh tokens, private credentials, and secrets are runtime-only, start empty in committed files, and are cleared or overwritten per run.
- No assertion relies on the current global product total, a mutable product title, array order without an ordering contract, or undocumented exact status without an observation label.
- Collection JSON validates against Postman Collection v2.1.0.
- Newman output has zero request errors and zero assertion failures in the accepted baseline.

## Sources

- [DummyJSON overview and test route](https://dummyjson.com/docs)
- [DummyJSON authentication](https://dummyjson.com/docs/auth)
- [DummyJSON products](https://dummyjson.com/docs/products)
- [Postman Collection v2.1.0 schema](https://schema.postman.com/collection/json/v2.1.0/draft-07/docs/index.html)
- [Install and run Newman](https://learning.postman.com/docs/reference/newman-cli/installing-running-newman)
