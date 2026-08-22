# Requirement interpretation and professional judgment

- Treat user input as evidence of the desired outcome, constraints, and current
  understanding, not as proof that the stated diagnosis or proposed mechanism
  is correct. Requirements often describe a symptom or an assumed solution;
  identify the underlying goal before choosing the implementation.
- Before changing anything, identify the request's underlying purpose,
  motivation, and invariant. Treat named files, declarations, and examples as
  evidence of the affected behavior, not as the default limit of the work,
  unless the user explicitly limits the scope to those exact targets.
- Inspect every related call path and analogous implementation that shares the
  same purpose or invariant. Apply the change consistently to relevant
  interfaces, implementations, adapters, tests, and documentation in the same
  task; do not wait for the user to point out equivalent cases one by one. This
  does not authorize unrelated cleanup or expansion beyond the request's real
  purpose.
- Exercise independent technical judgment. Inspect the repository, applicable
  specifications, documented contracts, and observed behavior instead of
  agreeing with a proposal by default. Clearly distinguish verified facts,
  inferred intent, assumptions, and recommendations.
- Do not silently implement a request that conflicts with an applicable
  specification, established invariant, security or data-safety requirement,
  or the user's stated goal. Explain the concrete conflict and recommend the
  smallest compliant alternative. Ask for clarification when different valid
  interpretations would materially change the result; refuse the conflicting
  part when no correct implementation exists.
- Do not use possible hidden intent as permission to override a clear and valid
  user decision, expand the task without authorization, or stall on harmless
  ambiguity. When the likely intent is well supported and the action is safe
  and reversible, proceed with the simplest reasonable interpretation and
  state material assumptions.
- Professional judgment requires constructive disagreement, not reflexive
  opposition. Base objections on evidence and explain their consequence; once
  the goal and constraints are coherent, carry them through consistently.

# Design principles

## Required behavior and reuse

- The simplest correct implementation is the default. Prioritize behavior
  required by the current task, an existing public contract, or a protocol the
  code explicitly implements.
- Before implementing a feature, search the current repository and its existing
  dependencies for components that already provide the required behavior. Also
  inspect closely related components that may provide part of the behavior and
  can be composed or extended.
- Prefer reusing an existing component over implementing a local replacement.
  Implement new functionality only after confirming that the available
  components cannot satisfy the concrete requirement, and record the specific
  missing capability that makes new code necessary.
- When reviewing or modifying code, remove unnecessary code instead of
  preserving it for possible future use. Fewer concepts, branches, and states
  are preferred when behavior remains correct.
- After each refactor or update, review the wider affected area for
  consistency, duplication, and simplification opportunities. Explicitly
  identify implementation, branches, tests, configuration, schema,
  documentation, and compatibility logic that became unnecessary because of
  the change, and remove them in the same change unless a current contract
  still requires them.

## Natural generalization

- Generalize a required mechanism to the narrowest domain-independent contract
  supported by its actual semantics. If removing the triggering domain noun
  leaves the operation, invariants, and failure modes unchanged, place the
  mechanism in a reusable module; keep domain-specific naming, key construction,
  authorization, and orchestration in the caller.
- Generalization changes ownership and vocabulary, not the amount of behavior
  being implemented. Implement only the operations, guarantees, and adapters
  required by current callers. Do not add neighboring capabilities, options,
  backends, or extension points merely because the module is reusable.
- Every abstraction and every runtime branch must have a concrete reason. A
  current requirement is sufficient reason to place its mechanism at the
  natural reusable seam; it is not sufficient reason to implement unrelated
  behavior. A well-understood near-term direction may justify low-cost passive
  structure when it avoids a likely breaking interface change, but it does not
  justify unused execution paths or additional runtime states.

## Module shape

- Prefer narrow interfaces, stable result types, and descriptive metadata that
  hide required complexity and can grow without changing existing callers.
  Judge the design by its total complexity and caller leverage, not by whether
  every reusable module currently has multiple callers.
- Prefer direct standard-library operations and concrete types for behavior
  whose semantics are genuinely local. Do not use this preference to embed a
  domain-independent mechanism in its first caller. Extract a helper only when
  it removes meaningful duplication or names a real domain operation. Do not
  hide a short, obvious operation behind a generic utility.
- Inline a small helper that has only one caller. Extract a single-caller
  helper when it names a stable domain operation, creates a useful seam, or
  materially shortens and clarifies the caller. Compatibility wrappers,
  aliases, and forwarding helpers still require a current external contract;
  extensibility alone does not justify them.
- Keep functions linear and small. Avoid converting a straightforward
  operation into a framework, pipeline, runtime, strategy, or configurable
  subsystem unless the current requirements demand those semantics.

## Authority, ownership, and dependency direction

- Give each capability one authoritative interface and owner.
- Cross modules through owned interfaces, not implementation details.
- Direct dependencies toward stable interfaces. A cycle signals misplaced
  ownership or a misplaced seam.
- Keep mechanics, invariants, and policy with their owners. Callers express
  intent instead of reproducing that complexity.

## Domain and protocol semantics

- Model built-in, default, and externally supplied values as ordinary domain
  values. Built-in objects and omitted-input defaults must follow the same
  validation, resolution, and execution path as explicitly supplied objects;
  do not create a privileged path for the system itself.
- Keep protocol identifiers distinct from persistence identifiers. Store IDs
  may represent internal entity relationships, but must not be accepted as
  protocol identifiers or exposed as protocol defaults. Never make one value
  ambiguous by trying an internal-ID lookup and then a protocol-identifier
  lookup.
- When implementing a specification, follow its defined semantics exactly. Do
  not infer or invent identifiers, fields, states, lifecycle rules, fallback
  behavior, or operations that the specification does not define. Add
  non-standard behavior only when an explicit product requirement defines it
  as an extension, and never present an extension as specification behavior.
- Normalize different input sources into one canonical domain value before
  applying business behavior. Input selection may branch where the sources
  differ, but validation, authorization, and execution must continue through
  one shared path after normalization.

## Serialization contracts

- Add `json`, `yaml`, database, validation, or similar struct tags only when
  the type is explicitly part of that wire, persistence, or validation
  contract. Types used only as domain results, runtime state, or internal
  coordination must not carry speculative tags for possible future use.
- Keep protocol decoding models separate from verified domain results when
  their representations differ. A protocol claims structure may carry the
  protocol field tags needed for decoding; embedding or projecting those
  claims into a result type does not make the result itself a wire model.

## Executable constraints

- Prefer executable constraints for current required behavior over prose-only
  rules when practical. Cleanup history is not itself a behavior contract.
- Document their purpose and guarantees without duplicating their enforcement.

# Security reasoning

- Define security as a concrete, end-to-end property with a threat, trust
  boundary, enforcement owner, and observable guarantee. A security label does
  not prescribe an implementation.
- Reject indirection, extra configuration, wrappers, manual prerequisites, or
  responsibility transfer unless they materially improve that property across
  the complete data and action path.
- The boundary owner provides the simplest closed workflow and hides internal
  mechanics. External references require real external ownership and lifecycle;
  documentation and tests verify the property, not the chosen mechanism.

# No defensive programming

- Defensive programming is strictly prohibited. Do not add guards for
  programmer errors or impossible internal states. Let incorrect internal use
  fail at its natural point.
- Do not add speculative nil checks, input normalization, arbitrary limits,
  copies, fallback behavior, retries, recovery, or validation. Such behavior is
  allowed only when required by the public contract, a protocol or security
  boundary, or a reproduced failure.
- Validate data once at the boundary that owns the validation rule. Do not
  repeat validation in constructors, helpers, and downstream methods.
- Trust invariants established by callers and previously completed validation.
  Do not re-check them throughout the call chain.
- Do not replace errors already produced clearly by the standard library with
  custom pre-validation. Add error context only when it identifies the failed
  domain operation.
- Tests must cover required behavior and reproduced failures. Do not create
  tests solely to justify speculative defensive branches.
- Test observable behavior and invariants through the public seam rather than
  incidental implementation details.

# Root-cause fixes only

- Fix the invariant or protocol contract that causes a failure. Do not mask a
  symptom with guessed retry counts, repeated polling, sleeps, timing margins,
  magic thresholds, or branches that merely make the current reproduction
  pass.
- Every synchronization, retry, or fallback must be justified by an explicit
  invariant, public contract, or protocol guarantee. Comments must name the
  requirement that makes the mechanism necessary; observed implementation
  behavior is not a substitute for one.
- When the current dependency or backend cannot provide the required
  guarantee, narrow the advertised capability or redesign the seam. Do not
  claim stronger semantics by approximating them probabilistically.
- Regression tests must assert the required invariant at the public seam. A
  passing test does not justify an implementation whose correctness still
  depends on scheduling, timing, buffering, or undocumented behavior.

# Refactoring and compatibility

- Keep the selected model consistent across design, implementation,
  persistence, callers, tests, and documentation.
- Removing a requirement removes its implementation, tests, schema, and
  documentation in the same change. Do not replace deleted tests with inverse
  or tombstone assertions that old types, flags, routes, fields, or behavior
  remain absent. Test absence or rejection only when a current explicit
  product, protocol, compatibility, or security contract requires it.
- Versioning and compatibility require an explicit user or protocol contract.

# Naming

- Base all naming and design on domain meaning and caller-observable semantics.
  Public interfaces express stable capabilities in the domain vocabulary;
  protocol, persistence, and mechanism vocabulary belongs to the adapters that
  own it.
- Use one term for one concept consistently across design, documentation,
  interfaces, implementations, tests, and user-facing contracts.

## Domain, transport, and persistence types

- Name domain results by their domain concept, such as `Organization`. Name an
  operation's domain input with the operation plus `Input`, such as
  `CreateOrganizationInput`. Name optional query or execution controls with
  `Options`, such as `ListOrganizationsOptions`.
- Domain interfaces use inputs, options, and domain results. Transport-specific
  representations use `Request` or `Response` when the wire contract differs.
- Name persistence representations for their adapter role, such as
  `StoredOrganization`. Shared adapter contracts are exported; adapter-local
  representations remain private.
- Name variables and parameters by their role in the local operation, not by
  repeating a type that is already explicit from the declaration or context.
  Prefer concise role names such as `old`, `current`, `target`, `result`, or
  `existing`. Avoid redundant names such as `currentApplication *Application`,
  `applicationVersion *ApplicationVersion`, or `applicationIndex
  *ApplicationIndex` unless the longer name distinguishes multiple values that
  genuinely have different domain roles.
- Keep parameter names in interface method declarations. Choose names that
  communicate each parameter's role; do not omit them merely because Go allows
  unnamed interface parameters.
- Document every interface method at the declaration. State the operation's
  contract, including ordering, pagination, mutation, or ownership semantics
  that callers and implementations must share.

# File organization and refactoring

- Keep rules with their narrowest owner: global guide, language guide, module
  `README.md`, or module `DESIGN.md`.
- Consult the nearest `README.md` for a module's usage contract and the nearest
  `DESIGN.md` for its design constraints before work that depends on them.
- A complete reusable module with a stable interface and implementation must
  contain both `README.md` and `DESIGN.md`.
- `README.md` is the caller guide: purpose, interface, usage, guarantees, and
  failure modes.
- `DESIGN.md` is the maintainer guide: goals, seams, ownership, invariants, and
  concrete plans and decisions.
- Keep both documents concise. Record stable facts and necessary rationale;
  remove repetition, implementation narration, speculation, and filler.
- Use headings only for distinct topics that benefit navigation. Keep short,
  related content in one section.
- Update `DESIGN.md` before material changes to a module's interface,
  persistence, lifecycle, or dependencies. Then update code and tests.
- Reconcile `README.md`, `DESIGN.md`, tests, and implementation in the same
  change. Remove stale claims.
- Keep rules coherent at each ownership level. Consolidate overlap and remove
  obsolete guidance instead of accumulating competing formulations.
- Organize files around one cohesive domain or closely related behavior.
  Prefer adding related code to an existing cohesive file over creating a
  small new file.
- Create a separate small file only when it represents a real independent
  module, ownership seam, generated artifact, platform-specific
  implementation, or another clear maintenance reason.
- Before finishing a refactor, check for orphaned test files and review newly
  created low-content files. Pair and justify them or merge their contents into
  the appropriate domain file.

# Formatting

- Use spaces for indentation, including Markdown code blocks. Follow tabs only
  where a canonical formatter or file format requires them.

# Language-specific rules

- For any task that reads, writes, reviews, or refactors Go code, read and
  follow [Golang.md](./Golang.md) before acting.
