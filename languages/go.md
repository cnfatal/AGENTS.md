# Go coding rules

Applies to any task that reads, writes, reviews, or refactors Go code in a
project that adopts these rules. The global [AGENTS.md](../AGENTS.md) provides
the baseline; more specific project rules take precedence when they conflict.
Project-specific Go conventions live in the owning repository's `AGENTS.md`.

## Exported surface

- Decide visibility from the package interface and caller value, not from file
  placement. Files organize one package implementation; crossing a file does
  not by itself justify exporting a declaration.
- Export a declaration when callers must name, construct, configure, embed,
  wrap, inspect, or branch on it as part of the supported package interface.
- Export built-in and default adapters as ordinary concrete types when they
  occupy an open interface seam. Make their zero value usable when practical,
  and have constructors return the concrete type so callers can compose or
  wrap it without losing capabilities.
- Prefer accepting interfaces and returning concrete types. Return an interface
  only when the concrete identity is intentionally outside the contract and
  exposing it would provide no supported caller operation.
- Keep compiled representations, parser state, reflection metadata, routing,
  merge machinery, caches, and other invariant-bearing implementation details
  unexported, even when several files in the package collaborate through them.
- An exported struct should normally expose the fields callers are expected to
  configure. Keep fields private only when direct construction or mutation
  would bypass an invariant; document that invariant on the type or field.
- Do not mechanically comment every exported struct field. A field whose name,
  type, tags, and containing type already make its meaning obvious needs no
  comment. Add a field comment only for a non-obvious contract such as zero or
  nil semantics, units, ownership, security boundaries, or invariants that the
  declaration itself cannot express.
- Export constructors and operations needed to use an exported type without
  duplicating package logic. Do not export implementation helpers merely to
  make package-internal reuse visible.
- Treat every exported declaration as a compatibility commitment. Document it
  and test its observable behavior through an external-package test.

## Package organization

- Never create or use a Go `internal` package, and never place Go code under an
  `internal/` directory. This prohibition applies in all scenarios, including
  production code, tests, generated code, shared implementations, adapters,
  and development tooling.
- Put shared or implementation-specific code in a regular package whose name
  describes its responsibility. Export and document the declarations that
  other packages need to use; do not use `internal` as an import restriction
  mechanism.
- Organize files by owned behavior, not by declaration kind. A file named after
  its package is a conceptual entry point only; Go does not give it special
  initialization or loading semantics. Use it for the package documentation,
  primary domain types, and primary interface when that forms a cohesive file.
- Do not create `types.go` by default. Keep a type with the interface or
  behavior that owns it. Use `types.go` only when a substantial set of domain
  types is shared by several equally important implementations and no cohesive
  domain file owns them.
- When a package owns an HTTP adapter, `api.go` may contain that adapter and its
  route registration. Use `API` when HTTP is the package's only protocol
  adapter. Use a protocol-specific name such as `HTTPAPI` when the package has
  or is expected to have multiple protocol adapters.
- Do not create generic `helper.go`, `helpers.go`, `misc.go`, or `utils.go`
  files. Keep a small helper with its primary caller. Extract a file only for a
  named, cohesive responsibility such as `validation.go` or `pagination.go`.
- Treat an implementation file and its test file as one maintained pair:
  `foo.go` and `foo_test.go`. Do not create a test file without the
  corresponding implementation file.
- Put a test that spans several implementations in the test file of the
  implementation that owns the primary behavior. Do not create generic files
  such as `extension_test.go`, `misc_test.go`, or `helpers_test.go` merely to
  hold a small amount of related test code.
- When renaming, moving, splitting, merging, or deleting an implementation
  file, inspect and update its paired test file in the same change. Move and
  reorganize the test contents as well as the filename; do not leave stale or
  orphaned test files behind.
- Before finishing a refactor, check for orphaned `*_test.go` files and review
  newly created low-content Go files. Pair and justify them or merge their
  contents into the appropriate domain file.

### Program entry points and dependency assembly

```go
// Forbidden: package variable creates a global singleton.
var defaultCache = NewCache()

// Forbidden: init performs implicit initialization outside the composition root.
func init() {
  RegisterCache(NewCache())
}

// Each structure declares only the direct dependencies it uses.
type API struct {
  Store Store
  Cache *Cache
}

type Worker struct {
  Cache *Cache
}

type Runtime struct {
  API    *API
  Worker *Worker
}

// Run is the composition root. Options contain configuration, not constructed
// dependencies. Context controls lifetime; it is not a dependency container.
func Run(ctx context.Context, options *Options) error {
  storage, err := NewStore(ctx, options.Store)
  if err != nil {
    return err
  }

  // Shared lifetime does not require a global singleton. Create one instance
  // here and inject it into every structure that directly depends on it.
  cache := NewCache(storage)

  target := &Runtime{
    API:    NewAPI(storage, cache),
    Worker: NewWorker(cache),
  }
  return target.Run(ctx)
}
```

- Keep executable `cmd/...` code limited to process configuration, context,
  logging, error reporting, and dispatch to `Run` or `Run<Role>`.
- Organize files by the runtime or domain behavior that owns the code. Do not
  split files mechanically by options, dependencies, constructors, or startup
  declarations.

## Code style

- Format Go source with `gofmt`. Its tab indentation is the Go-source
  exception to the global space-indentation rule; Go examples in Markdown use
  spaces.

### Completion checks

- Before finishing a Go change, run the repository's canonical formatting,
  generation, test, vet, lint, and static-analysis commands that apply to the
  affected scope. When the repository has no canonical checks, run `gofmt` on
  changed Go files, `go test ./...`, `go vet ./...`, and a maintained static
  analyzer compatible with the module's Go version, such as `staticcheck`.
- Also run the official Go `modernize` analyzers on the affected packages,
  including tests and applicable build tags. Passing tests, `go vet`, or
  Staticcheck does not establish modernization coverage. Verify the selected
  tool's available analyzers and enabled checks; `gopls` and `go fix` need not
  expose or enable the same set. Report unavailable checks as unverified.
- Review modernization diagnostics or suggested diffs before applying fixes.
  Apply changes compatible with the supported Go versions and required
  semantics, then rerun the affected checks.
- For sequential benchmarks targeting Go 1.24 or newer, prefer `for b.Loop()`
  where applicable and review `bloop` findings on `b.N` loops. Check timer
  boundaries and compiler-optimization effects, and rerun affected benchmarks
  after migration. Preserve `b.RunParallel` semantics and retain a different
  loop when the benchmark's measured behavior requires it.
- Review every diagnostic. Completion requires no unexplained finding in the
  changed scope; remove unused declarations, unreachable branches, ineffective
  assignments, stale compatibility code, and helpers orphaned by the change.
  Fix the owning code rather than suppressing a valid diagnostic.
- Inspect generated Go and schema artifacts through their owning generator.
  Regenerate them after source contract changes and verify that a second run
  produces no diff.

### Supported-version migrations

- Read the supported Go language, toolchain, and dependency versions from the
  repository's module and build configuration. Write for that declared range;
  do not preserve compatibility with older versions unless an explicit
  external contract requires it.
- Treat deprecation and version-compatibility diagnostics in the affected call
  path as migration work. Confirm the replacement against the version actually
  selected by the module, preserve observable semantics, and migrate every
  equivalent use in the affected scope.
- A language, toolchain, or dependency migration does not itself authorize
  narrowing exported names, visibility, or signatures. Preserve the supported
  public surface unless an explicit versioned contract requires a breaking
  change.
- When the supported minimum version advances, remove compatibility branches,
  wrappers, build constraints, and tests whose only purpose was the previous
  version. Update generated artifacts and caller documentation when the
  migration changes an observable contract.

### Local variables

- Declare every local variable as close as possible to its first use. Do not
  create stores, result objects, buffers, or other dependencies early when
  unrelated logic appears before their use. When a value has multiple uses,
  declare it immediately before the smallest contiguous block containing those
  uses.

### Function naming

- Name functions and methods by their complete domain operation or returned
  concept. A reader should understand what is obtained, created, refreshed,
  validated, or changed without relying on the call site for meaning.
- Avoid vague names such as `Process`, `Handle`, `Execute`, `Configuration`, or
  `Data` when the name omits the domain object or result. Prefer explicit names
  such as `GetClientConfiguration`, `RefreshClientConfiguration`, or
  `ValidateAccessToken`.
- When a function's responsibility or return concept changes, rename it in the
  same change. Do not retain a formerly accurate name after extracting part of
  its behavior.

### Function declarations

- Keep a function or method declaration on one line when the complete
  declaration remains reasonably short and readable. Do not vertically expand
  parameters merely because there is more than one parameter.
- Split a declaration across lines only when the single-line form is too long
  or when the parameter structure materially benefits from grouping.

### Interface declarations

- Order interface methods by semantic capability. Keep closely related methods
  together and separate distinct groups with one blank line.
- Make each group evident through method names and contracts. Group comments
  carry semantics not expressed by the methods themselves.

### Variadic options

Choose the option representation from its lifecycle and extension semantics.
Do not assume either a function or an interface is inherently faster; use
benchmarks and escape analysis for performance claims.

#### Operation options

Operation options are expanded on recurring calls such as Store List, Get,
Create, Watch, or controller event processing. Model them as open interfaces
implemented by concrete values:

```go
type ListOptions struct {
  Size int
  Sort string
}

type ListOption interface {
  ApplyToList(*ListOptions)
}

type DefaultSizeOption int

func (option DefaultSizeOption) ApplyToList(options *ListOptions) {
  if options.Size == 0 {
    options.Size = int(option)
  }
}

func DefaultSize(size int) DefaultSizeOption {
  return DefaultSizeOption(size)
}
```

- Use an exported `ApplyToXxx(*XxxOptions)` method and an exported accumulator
  when external packages may define operation options. Do not seal the option
  vocabulary behind a private method or an unexported accumulator.
- Have constructors return concrete option types. Do not return a captured
  closure for an operation option.
- When one semantic option applies to several operations, use one concrete type
  that implements each applicable `ApplyToXxx` interface.
- Provide one `ApplyXxxOptions` expansion function per accumulator. Store and
  adapter implementations use it instead of repeating option loops.
- Keep option values immutable while applying them. Scalar fields may replace
  earlier values in declaration order; collection semantics must be explicit.

##### Operation option naming

- Name the accumulator `XxxOptions`, its variadic interface `XxxOption`, the
  application method `ApplyToXxx`, and the expansion function
  `ApplyXxxOptions`.
- Name a concrete option by its semantic setting as `<Stem>Option`. Do not put
  the target operation in the concrete type or constructor name. One concrete
  option implements every applicable operation interface.
- Keep the constructor suffix and concrete type stem identical:
  `WithSort` returns `SortOption`, `WithFieldRequirements` returns
  `FieldRequirementsOption`, and `DefaultSize` returns `DefaultSizeOption`.
- Use `WithXxx` to set, append, or enable a value unconditionally. Use
  `DefaultXxx` only to fill an omitted value. Use `WithXxxFromYyy` when the
  constructor converts another representation.
- Model a boolean capability as a parameterless `WithXxx()`. Do not use
  `WithXxx(bool)`. Add `WithoutXxx` only when callers currently need an
  explicit disabling operation.
- Do not duplicate a shared semantic option as operation-prefixed constructors
  such as `WithGetResourceVersion` or `WithDeleteResourceVersion`; use one
  `WithResourceVersion` constructor and implement the applicable interfaces.

#### Construction options

Functional options remain appropriate for low-frequency construction and
startup configuration when callers intentionally need arbitrary accumulator
mutation or the option itself can fail:

```go
// Bad: external packages cannot add or compose options.
type ProgramOption interface {
  apply(*programOptions)
}

// Good: the option vocabulary and accumulator are open.
type ProgramOptions struct {
  Sources []Source
}

type ProgramOption func(*ProgramOptions)

func WithSources(sources ...Source) ProgramOption {
  return func(options *ProgramOptions) {
    options.Sources = sources
  }
}
```

- Use construction functional options only outside recurring request, storage,
  event, and reconciliation hot paths.
- Export both the function type and accumulator type, and export the fields
  that external construction options need to modify.
- A captured closure in a construction option is acceptable only when its
  initialization-only lifecycle makes that allocation immaterial or a
  benchmark demonstrates that it is acceptable.
- Resolve defaults first, then apply option functions once in declaration
  order. Keep validation with the module that owns the resulting settings.

### Error flow

- Never write success-path checks such as `if err == nil`. Handle `err != nil`
  first and let the successful path continue linearly. This also applies to
  `if err == nil { ... } else { ... }` forms.
- For expected errors such as not-found, handle that case inside the
  `err != nil` branch. Do not invert the condition to make the expected error
  the outer success branch.

### Composite literals

```go
// Good: a short two-field value stays compact.
user := UserInfo{Name: "user", Enabled: true}

// Good: expand when the value has several fields or complex nesting.
config := Config{
  Name:    "worker",
  Enabled: true,
  Policy:  Policy{Mode: "strict", Retries: 3},
  Targets: []Target{{Name: "primary", Weight: 100}},
}
```

- Keep a keyed struct literal with two or three short fields on one line when
  the complete value remains easy to scan. The same applies to short nested
  literals and short slice or map elements.
- Expand a literal when it has more than three fields, contains long
  expressions, needs comments, or its nesting makes the single-line form hard
  to read. Use one field per line at the level that needs expansion; simple
  nested values may remain compact.
- Do not expand a readable literal mechanically. When touching a dense literal,
  reformat only the level whose current shape harms readability.

### Chained calls

```go
// Bad
result := builder.WithName("worker").WithEnabled(true).Build()

// Good
result := builder.
  WithName("worker").
  WithEnabled(true).
  Build()
```

- In a call chain, put every function or method call on its own line.
- Do not manually align chained calls or add tab indentation. Use simple
  continuation indentation and let `gofmt` determine the final whitespace.
- When touching a compact call chain, reformat the complete chain.
- Before finishing Go changes, inspect every call chain in the full changed
  scope; `gofmt` does not expand compact chains.
