# Change specification: Remove the BookCatalog index User-Agent display

## Metadata

- **Owner:** Workshop application team
- **Status:** Approved
- **Source root:** `legacy-src/dotnet-bookcatalog/shared-legacy-app`
- **Related issue:** Not created

## Problem and outcomes

### Problem

The .NET BookCatalog index page reads the HTTP `User-Agent` header in
`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
as `HttpContext.Request.UserAgent`, stores it in `ViewBag.UserAgent`, and renders
the value in a "YOUR BROWSER" box in
`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`.
This is unnecessary request-header presentation and legacy presentation coupling.
The current behavior is not shown to disclose the value to another user or persist
it; it is rendered back to the same visitor.

### Desired outcomes

1. The catalog index does not read or render the request `User-Agent` value.
2. Active-book filtering, title ordering, routes, catalog models, CRUD links, and
   existing user-visible catalog behavior remain unchanged.

### Success measures

- The controller contains no `HttpContext.Request.UserAgent` read and no
  `ViewBag.UserAgent` assignment.
- The index view contains no `YOUR BROWSER` label or `user-agent-box`.
- The existing Release validation passes.
- The existing catalog remains usable under the documented IIS Express/LocalDB
  smoke check, if that runtime check is performed with the required prerequisites.

## Functional requirements

| ID | Requirement | Evidence or rationale |
| --- | --- | --- |
| FR-1 | `BooksController.Index()` shall continue returning active `Book` records ordered by `Title`. | `src/BookCatalog.Web/Controllers/BooksController.cs` defines the existing EF6 query. |
| FR-2 | `BooksController.Index()` shall not read `HttpContext.Request.UserAgent` or assign `ViewBag.UserAgent`. | The requested behavior is complete removal of the header read and view binding. |
| FR-3 | `Views/Books/Index.cshtml` shall not render a `User-Agent` value or the `YOUR BROWSER` label. | The existing visible block is tied directly to `ViewBag.UserAgent`. |
| FR-4 | Existing catalog routes, columns, CRUD actions, EF6 model behavior, and database behavior shall remain unchanged. | These contracts are outside the approved removal scope. |

## Constraints and out of scope

### Constraints

- **Allowed paths:**
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`
- **Primary validation command:** `.\scripts\validation\validate-dotnet.ps1 -Configuration Release`
- **Compatibility:** Preserve ASP.NET MVC 5, .NET Framework 4.8, EF6,
  `packages.config`, the classic solution layout, active-only filtering, title
  ordering, existing routes, and CRUD links.
- **Dependencies:** Add no package, project, configuration, or service dependency.
- **Data and security:** Use only synthetic catalog data. Do not point the
  application at a production or business database. Do not add secrets or
  credentials.

### Exclusions (out of scope)

- No ASP.NET Core migration.
- No EF6 or Entity Framework Core migration.
- No database schema, initialization, or seed-data change.
- No package, configuration, `BookCatalog.sln`, or validation-wrapper change.
- No test-project creation, test framework change, or test-source addition.
- No route, model, CRUD, filtering, sorting, or styling cleanup outside removal
  of the `User-Agent` read and visible display.
- No logging, telemetry, analytics, broader privacy, Azure, or infrastructure
  work.

## Edge cases

| Type | Condition | Expected behavior |
| --- | --- | --- |
| Normal | Active and inactive books exist | Only active books appear ordered by title, with existing CRUD actions. |
| Failure | The HTTP request has no `User-Agent` header | The index page renders normally; no diagnostic block or error is added. |
| Boundary | The `User-Agent` value is empty or unusually long | No value or label is read or rendered. |
| Compatibility | Existing routes, models, columns, and CRUD links are used | Their behavior remains unchanged. |

## Acceptance criteria

### AC-1: The catalog index no longer reads or binds browser metadata

- **Given** the catalog index action is available
- **When** a request opens `/Books`
- **Then** `BooksController.Index()` does not read
  `HttpContext.Request.UserAgent` and does not assign `ViewBag.UserAgent`
- **Verification:** Source review of the controller and the allowed-path diff

### AC-2: The catalog index no longer displays browser metadata

- **Given** a request includes any `User-Agent` header
- **When** a user opens `/Books`
- **Then** the response contains no `User-Agent` value and no `YOUR BROWSER` label
- **Verification:** Manual IIS Express browser check with a distinctive header,
  plus source review of `Views/Books/Index.cshtml`

### AC-3: Catalog behavior remains unchanged

- **Given** active and inactive books exist
- **When** a user opens `/Books`
- **Then** only active books appear ordered by title and existing View, Edit, and
  Delete actions remain available
- **Verification:** Manual IIS Express/LocalDB smoke check if prerequisites are
  available

### AC-4: The existing Release build remains valid

- **Given** the documented Windows and Visual Studio MSBuild prerequisites are
  installed
- **When** `.\scripts\validation\validate-dotnet.ps1 -Configuration Release` runs
  from the repository root
- **Then** restore and the classic solution build complete successfully
- **Verification:** Record the command and exit status

## Non-functional requirements

| ID | Quality | Testable requirement | Verification |
| --- | --- | --- | --- |
| NFR-1 | Scope | The implementation diff changes only the two allowed controller and view files. | `git diff --name-only` |
| NFR-2 | Maintainability | No package, project, test infrastructure, or configuration changes are introduced. | Solution, project, and package diff review |
| NFR-3 | Compatibility | The active catalog behavior and classic ASP.NET MVC/EF6 contracts remain unchanged. | Manual smoke check and allowed-path diff review |

## Assumptions, dependencies, and risks

| Item | Type | Validation or mitigation |
| --- | --- | --- |
| The header display has no undocumented functional consumer. | Risk | Stop if repository evidence or maintainer review identifies a documented or operational consumer. |
| Windows, Visual Studio MSBuild, IIS Express, LocalDB, and .NET Framework 4.8 prerequisites are available. | Dependency | If any prerequisite is unavailable, stop and report it; do not claim runtime validation. |
| Removing the display addresses presentation cleanup rather than proven cross-user disclosure. | Assumption | Keep the specification limited to current-user rendering; do not add claims about logging or privacy exposure. |
| No automated test project exists for this baseline. | Dependency | Do not add test infrastructure. Verify the source diff and build, and perform a manual smoke check only if the runtime environment is available. |

## Plan-mode review checkpoint

### Evidence

- **Current behavior:**
  `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:BooksController.Index()`
  reads `HttpContext.Request.UserAgent` and assigns `ViewBag.UserAgent`.
- **Current display:**
  `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`
  renders the `user-agent-box` and `YOUR BROWSER` label when
  `ViewBag.UserAgent` is present.
- **Catalog behavior:**
  `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:BooksController.Index()`
  queries `ApplicationDbContext.Books`, filters `IsActive`, orders by `Title`,
  and passes the result to the view.
- **Existing tests:** No automated test project or test source files were found
  under `legacy-src/dotnet-bookcatalog/shared-legacy-app`.
- **Baseline validation evidence:** The maintainer ran
  `.\scripts\validation\validate-dotnet.ps1 -Configuration Release` and it
  returned without an error. This result verifies the existing Release restore
  and build baseline on the available Windows environment. It does not verify
  runtime behavior or CRUD correctness.
- **Runtime smoke status:** Runtime smoke testing through IIS Express/LocalDB
  has not been done.

### Unresolved questions

- None. The human decisions are:
  remove the behavior completely, add no test project or solution change, and
  stop rather than claim runtime validation if required prerequisites are missing.

### Boundary and validation review

- [x] Evidence citations were checked against the repository.
- [x] Allowed paths include all anticipated implementation changes and nothing unrelated.
- [x] The primary validation command is safe, non-interactive, and runnable from the repository root.
- [x] Acceptance criteria cover the normal catalog case, missing or unusual
      headers, and compatibility.
- [x] Exclusions prevent migrations, test infrastructure, dependency changes,
      telemetry, and unrelated cleanup.

### Stop conditions

Stop and request human guidance if:

- repository evidence contradicts this specification;
- an unresolved question could materially change behavior or scope;
- satisfying a criterion requires a path outside the allowed paths;
- the baseline or primary validation fails for a reason caused by the proposed
  change, or the command is missing, destructive, or irreproducible;
- the build, IIS Express, LocalDB, MSBuild, or .NET Framework 4.8 prerequisite
  prevents required evidence from being collected;
- runtime smoke testing cannot be performed, and do not claim it passed;
- a test project, solution edit, package, schema, configuration, secret,
  production resource, or destructive action would be required;
- catalog filtering, sorting, routing, models, CRUD, or database behavior would
  change outside this specification;
- implementation or validation cannot satisfy the acceptance criteria without
  expanding the approved contract.

### Human approval

- [x] I reviewed the evidence, unresolved questions, allowed paths, acceptance
      criteria, validation command, exclusions, risks, and stop conditions.
- **Reviewer:** Workshop participant
- **Approval date:** 2026-11-23
- **Approved specification commit:** ff02f8d5fd9a765efea3ffd957e3127a27b9f2c0
