# Go coding rules

## Cross-file visibility

```go
// Bad
type Config struct {
  name    string
  enabled bool
}

// Good
type Config struct {
  Name    string
  Enabled bool
}
```

- Any function, method, type, constant, or variable used from a different
  source file than the file where it is declared must be exported. Do not keep
  a declaration private merely because Go package visibility allows another
  file in the same package to access it.
- Export declarations that define interaction between distinct structures or
  modules, even when the current declarations happen to be in the same source
  file. An interaction boundary is part of the package interface and should be
  named as such.
- Use an unexported declaration only when its implementation must be hidden.
  Locality alone is not a reason to make a declaration unexported. When a
  declaration becomes an interaction boundary, export it, update every caller,
  and add the documentation required for the exported symbol in the same
  change.
- An exception requires a concrete reason such as generated code, a
  platform-specific file set that forms one implementation, or another clear
  maintenance constraint. Treat exceptions as uncommon and document the
  reason near the declaration.
- During refactoring and review, check cross-file references in both production
  and test files. Tests in a paired `foo_test.go` file also count as cross-file
  use; prefer testing through the exported interface rather than relying on
  package-private declarations.
- Fields of an exported struct should generally also be exported. If its fields
  should remain private, first reconsider whether the struct itself should be
  unexported.
- Helper functions whose primary purpose is to construct, convert, parse, or
  otherwise operate on an exported struct should generally also be exported so
  callers do not have to duplicate that logic. Keep such a helper unexported
  only when it depends on private invariants or is solely an internal
  implementation step.
- Keep a field private inside an exported struct only when a concrete invariant
  requires controlled construction or mutation; document that reason on the
  field or struct.

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

## Code style

- Format Go source with `gofmt`. Its tab indentation is the Go-source
  exception to the global space-indentation rule; Go examples in Markdown use
  spaces.

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
