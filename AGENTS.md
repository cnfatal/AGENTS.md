# Requirement interpretation and professional judgment

- Treat user input as evidence of the desired outcome, constraints, and current
  understanding, not as proof that the stated diagnosis or proposed mechanism
  is correct. Requirements often describe a symptom or an assumed solution;
  identify the underlying goal before choosing the implementation.
- Before changing anything, identify the request's underlying purpose,
  motivation, and invariant. Treat named files, declarations, and examples as
  evidence of the affected behavior, not as the default limit of the work,
  unless the user explicitly limits the scope to those exact targets.
- Separate normative requirements from examples. A concrete mechanism may
  illustrate a rule, but it is not the required design unless its semantics are
  themselves part of the applicable contract.
- Before user input becomes a repository contract through code, tests, schemas,
  configuration, or documentation, verify it against the underlying goal,
  applicable specifications, repository evidence, and established invariants.
  Synthesize the accepted semantics in domain vocabulary; user wording,
  diagnoses, examples, and proposed mechanisms remain source material rather
  than adopted design.
- Preserve source wording only when its exact text is required by an applicable
  contract or the user explicitly requests a quotation. Identify the source and
  keep quoted material distinct from project decisions.
- Classify each accepted requirement by lifetime and owner before choosing its
  artifacts. A task-scoped instruction governs only the current work or its
  requested output and does not by itself require repository documentation. An
  observable product contract belongs in implementation, executable tests, and
  caller documentation where required. A durable design constraint that future
  maintainers must preserve belongs in the narrowest owning `DESIGN.md`.
- Let task scope limit persistence, not quality. Any change that remains in the
  repository must still follow its current design, root-cause, consistency, and
  cleanup rules.
- Treat requirements in an ongoing conversation as an evolving working model,
  not as a permanently cumulative list. When the user proposes a materially
  different direction, re-evaluate earlier requests and assumptions against
  the new direction instead of preserving them mechanically.
- Give the user's current goal and latest coherent requirements priority when
  they conflict with earlier product or implementation preferences. Use prior
  requests as context for understanding the problem, not as active constraints,
  unless the user explicitly keeps them or they express an applicable public
  contract, specification, security property, or data-safety requirement.
- Do not combine superseded and current designs merely to satisfy the full
  conversation history. Remove or revise work that only served the old
  direction, and keep the resulting design internally consistent with the
  current request.
- Treat every code change as a design review of its entire affected area.
  Inspect every related call path and analogous implementation, then
  re-evaluate their interfaces, ownership, flows, and abstractions against the
  current goal and invariants. Existing code remains unchanged only when that
  review establishes its current shape still belongs to the simplest coherent
  design, never because preservation is the default.
- Before editing, enumerate the affected surface from the owning interface:
  every exported type, field, and method; every adapter and call path; and the
  paired tests, `README.md`, and `DESIGN.md`. The review is complete only when
  each item has been inspected and either changed or affirmatively justified by
  the current caller contract.
- Require every public wrapper, nested input, field, and result to carry an
  independent caller-observable meaning, invariant, or lifecycle. Internal
  reuse, implementation vocabulary, anticipated backends, and speculative
  extensibility do not establish caller value. Reshape the complete interface
  when one exposed concept fails this test, then repeat the affected-surface
  inventory against the new shape.
- Apply the resulting design consistently to every affected interface,
  implementation, adapter, test, and document in the same task; do not wait for
  the user to identify equivalent cases one by one. This does not authorize
  unrelated cleanup or expansion beyond the request's real purpose.
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

## Simplicity, consistency, and reuse

- The simplest correct implementation is the default. Prioritize behavior
  required by the current task, an existing public contract, or a protocol the
  code explicitly implements.
- Judge simplicity by total system complexity: code, concepts, runtime
  branches, states, authoritative execution paths, interface knowledge, and
  caller coordination. Fewer lines are a possible result, not an independent
  goal; complexity may concentrate inside its owning deep module when that
  makes the system and its callers simpler.
- Special cases require a concrete business rule, protocol requirement, or
  verified difference in required behavior. An object's name, position, or
  first use case alone does not justify different treatment. Remove exceptions
  without such a basis; route equivalent behavior through the same mechanism.
- Before adding configuration, branches, or abstractions to solve a problem,
  first consider removing unnecessary behavior, using the existing default
  mechanism, or correcting ownership. Preserve required behavior and choose
  the solution with the least total complexity.
- For common behavior, start with existing capabilities and their documented
  configuration. Search the repository, related implementations, standard
  library, and installed dependencies before designing custom logic.
- Read the relevant module documentation and the dependency documentation for
  the actual version in use: supported options, defaults, examples, and limits.
  Consult official documentation when local material is insufficient; inspect
  implementation or reproduce behavior when the contract remains unclear.
  Memory, API names, and an unsuccessful text search do not establish a missing
  capability.
- Before substantial custom logic, whenever branches or coordinated state keep
  growing, and before completion, repeat this evidence-based self-review:
  1. Which current requirement or reproduced failure makes this logic necessary?
  2. Is it required behavior or a hypothetical edge case? Implement rare cases
     only when a current contract or verified failure requires them; rarity
     does not waive correctness, security, or data-safety requirements.
  3. Can an existing component, native option, or complete library operation own
     the behavior? Check correct usage, composition, and suitable supported
     alternatives before adding interception, synchronization, or compensation.
  4. What exact capability remains missing, and what is the smallest change at
     its natural owner that fills only that gap?
- Proceed with substantial custom logic only after naming the checked existing
  capabilities and their concrete mismatch in the working explanation. When
  this review finds an existing solution, remove the redundant custom mechanism;
  invested effort and passing tests are not reasons to retain it. Keep research
  and repeated self-review with the agent; ask the user only about unresolved
  requirements or tradeoffs that materially change the result.
- Choose whether to reuse, deepen, reshape, replace, or remove existing modules
  from the current semantics and total complexity. Existing code has no
  preservation preference: reuse it only when its ownership and interface
  remain correct for the current design, and change it instead of layering
  around a design that no longer fits.
- Introduce a new module only after confirming that no existing module can
  naturally own the required behavior, and record the capability or ownership
  mismatch that makes the new module necessary.
- When required semantics already have an authoritative implementation, reuse
  the complete behavior through its interface instead of extracting incidental
  helpers. Reuse must reduce duplicated decisions and execution paths, not
  merely duplicate fewer lines.
- A code change is incomplete until the affected area has been reviewed for
  consistency, duplication, and simplification. Remove implementation,
  branches, tests, configuration, schema, documentation, and compatibility
  logic made unnecessary by the resulting design unless a current contract
  still requires them.

## Natural generalization

- Generalize a required mechanism to the narrowest domain-independent contract
  supported by its actual semantics. If removing the triggering domain noun
  leaves the operation, invariants, and failure modes unchanged, place the
  mechanism in a reusable module; keep domain-specific meaning, policy, and
  orchestration with their respective owners.
- Give behavior with the same meaning, invariants, failure modes, and reasons to
  change one authoritative owner. Keep behavior separate when only its code
  shape is similar; incidental resemblance does not justify a shared helper or
  configurable abstraction.
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

- Organize implementation into deep modules: each module owns a cohesive
  capability and its invariants behind the smallest practical interface. Judge
  depth by caller leverage, not implementation size or the number of callers.
- Hide required internal steps, ordering, and mechanisms that callers do not
  need to use the module correctly. Expose stable domain results and metadata
  when callers need them to decide what to do next; do not expose raw adapter
  or protocol details.
- Minimize the knowledge, coordination, and mutable state shared between
  modules rather than mechanically minimizing dependency counts. A dependency
  must represent a real capability reached through its owner's interface.
- Implement behavior directly when its semantics and variation are genuinely
  local. Do not use this preference to embed a domain-independent mechanism in
  its first caller. Extract a helper only when it removes meaningful
  duplication or names a real domain operation. Do not hide a short, obvious
  operation behind a generic utility.
- Inline a small helper that has only one caller. Extract a single-caller
  helper when it names a stable domain operation, creates a useful seam, or
  materially shortens and clarifies the caller. Compatibility wrappers,
  aliases, and forwarding helpers still require a current external contract;
  extensibility alone does not justify them.
- Remove shallow modules that only forward calls, expose internal phases, or
  make callers coordinate invariants owned by the module.

## Authority, ownership, and dependency direction

- Give each capability one authoritative interface and owner.
- Place each guarantee with the narrowest module that can own it completely so
  every caller receives it without reproducing its implementation.
- Cross modules through owned interfaces, not implementation details.
- Direct dependencies toward stable interfaces. A cycle signals misplaced
  ownership or a misplaced seam.
- Keep mechanics, invariants, and policy with their owners. Callers express
  intent instead of reproducing that complexity.

## Canonical core flow

- Give behavior that shares the same invariants, ordering, effects, consistency
  guarantees, and failure semantics one authoritative execution path. Identify
  that path by its semantics, not by entry point or use-case names.
- Keep entry points thin. Perform only source-specific adaptation before
  normalizing equivalent inputs into one domain value and converging on the
  authoritative path. Adapt presentation only after receiving its domain
  result.
- Give every domain fact one logical source of truth, one authoritative owner,
  and one mutation path. Logical authority does not require a single physical
  representation. Route every command and mutation intent through the owner's
  interface rather than creating parallel write paths.
- After establishing an accepted state transition, let the authoritative owner
  expose the resulting domain fact through its result and, when independent
  downstream owners require fan-out, through an owned notification or event
  interface. Publish domain changes, not incidental storage operations.
- Keep data flow unidirectional: commands and mutation intent flow toward the
  authoritative owner; results, snapshots, and events flow outward. Treat
  caches, indexes, views, projections, and replicas as replaceable downstream
  representations. They may serve reads under an explicit freshness and
  consistency contract, but they must not decide mutations or write the
  observed fact back; route requested changes to the owner as commands.
- Let each observer own its independent reaction. It may update its own derived
  state or start a distinct flow for a different owned fact, but it must not
  coordinate other observers, depend on observer order, infer completion of the
  originating flow, or feed changes back into that flow. Any reaction that
  affects the originating mutation's validity, result, consistency, or success
  belongs in the authoritative owner or its explicit canonical flow.
- Bidirectional state synchronization and feedback loops between modules are
  prohibited. Do not use paired writers, mutual observers, write-back callbacks,
  reconciliation loops, last-writer-wins behavior, or availability-based source
  switching to align duplicated mutable representations. Remove the duplicated
  authority and converge through the owning interface.
- When an external protocol requires changes to travel in multiple directions,
  isolate it in one adapter that owns its mapping, ordering, conflict, and
  failure semantics. Publish only its accepted canonical result into the domain;
  the protocol must not create multiple domain sources of truth.
- Call a module directly for a simple operation that needs no orchestration.
  Canonical flows may nest through deep modules; they do not require a wrapper
  per use case or one system-wide orchestrator.
- Write orchestration as ordinary, explicit, linear, and small domain code that
  declares what happens and delegates how it happens to deep modules. Introduce
  a general-purpose orchestration abstraction or runtime variability only when
  the required semantics need it.
- Add a feature to the narrowest owner of its semantics. If an existing module
  already represents that operation, deepen the module instead of exposing a
  new top-level flow step merely to make the feature visible.
- Let the owner represent genuine variations through its domain interface and
  contain the necessary branching behind a stable result. Keep flows separate
  when their lifecycle, invariants, ordering, effects, consistency guarantees,
  or failure semantics differ; do not merge semantically different behavior
  merely to share a flow.
- Give the module that owns an end-to-end invariant responsibility for its
  complete failure and consistency semantics. Entry points and subordinate
  modules must not infer global progress and independently recover the flow.

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
- Require every regression test to exercise behavior through its owning seam
  and compare the result with an independent semantic oracle. It must be able
  to go red when the required behavior breaks, not merely when implementation
  text is edited.
- Do not create echo tests that read source, render a template, inspect generated
  output, or assemble a command only to assert a literal, field, path, flag,
  registration, or structure copied directly from the implementation. This
  includes one-off grep, awk, jq, snapshot, and wrapper-script checks whose
  assertion has no independent semantics. Such a test is justified only when
  it verifies meaningful branching, composition, schema conformance,
  compatibility, or a cross-artifact invariant that can fail independently of
  the declaration it examines.
- For a declarative-only change with no independent behavioral oracle, use the
  format's parser, validator, linter, renderer, compiler, or build plus focused
  review. Do not manufacture a test or permanent test harness merely to restate
  the declaration.
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
- Give shared authorization policy one authoritative owner, while the protected
  operation owns enforcing the policy before the protected effect. The policy
  may be implemented by another module, but entry-point checks must not be its
  only enforcement when another path can reach the operation.

# No defensive programming

- Defensive programming inside the domain is strictly prohibited. Once a
  module establishes an invariant, internal functions and downstream modules
  trust it. Let programmer errors and impossible internal states fail at their
  natural use point, then fix the code that violated the invariant.
- Do not add speculative nil checks, input normalization, arbitrary limits,
  copies, fallback behavior, retries, recovery, or validation. Such behavior is
  allowed only when required by the public contract, a protocol or security
  boundary, or a reproduced failure.
- At an external seam, let its adapter validate only the structure, protocol,
  and security rules it owns, then map its representation to the owning
  module's input. Representations owned by an external seam must not leak into
  the domain.
- Validate a domain rule once where its owner establishes the domain value. A
  module interface is a validation seam only when it owns the rule and accepts
  data that has not already been validated; crossing a module alone does not
  justify revalidation.
- Trust values from sources whose owned contracts already establish the
  required invariants. When a reproduced violation contradicts such a
  contract, repair its owner rather than adding routine downstream validation.
- Do not replace errors already produced clearly by the standard library with
  custom pre-validation. Add error context only when it identifies the failed
  domain operation.

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
- Keep the working tree as a model of current requirements. When a requirement
  or feature is removed, remove every artifact whose only purpose was to
  implement, expose, reject, test, configure, document, migrate, or remember it
  in the same change.
- Version control owns ordinary implementation history. Do not preserve removed
  behavior as deprecated APIs, aliases, stubs, disabled flags, compatibility
  paths, rejection branches, tombstone tests, comments, examples, schemas, or
  documentation that only records its removal.
- Retain an artifact related to removed behavior only when a current explicit
  public, protocol, security, persistence, or data-lifecycle contract still
  requires observable behavior. Name that contract and keep only the minimum
  required mechanism with its owner.
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
  `applicationVersion *ApplicationVersion`, or `applicationIndex *ApplicationIndex`
  unless the longer name distinguishes multiple values that
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
- Create or expand documentation only when callers or maintainers need durable
  knowledge that code, configuration, tests, and existing documentation do not
  adequately convey. Prefer updating the narrowest existing document.
- `README.md` and `DESIGN.md` are optional. A new module or a change to its
  interface, persistence, lifecycle, or dependencies does not by itself justify
  creating either file; do not add documents merely to complete a file set or
  restate upstream documentation and self-evident implementation details.
- `README.md` is the caller guide: purpose, interface, usage, guarantees, and
  failure modes.
- `DESIGN.md` is the maintainer guide. Record an adopted durable design or a
  selected plan for a material design change as verified goals, invariants,
  ownership, seams, decisions, and necessary rationale in the project's domain
  vocabulary.
- Write `DESIGN.md` as an implementation-independent specification of the
  intended system. State the end-state goals, invariants, ownership, seams,
  observable contracts, and failure semantics that guide implementation. Do
  not assume or report implementation progress, describe the repository's
  existing behavior, or infer the design from code. Evaluate implementation
  separately against the design; keep completion status, gaps, migration
  progress, and task tracking outside `DESIGN.md`.
- Keep each `DESIGN.md` within its owning module's boundary. Describe another
  module only through the input, output, guarantee, or dependency direction the
  owner requires, and name the external owner of every cross-module guarantee.
  Do not absorb a neighboring module's policy, implementation, or lifecycle
  into the document merely to make the local design self-contained.
- Write `README.md` entirely from the caller's required knowledge. Place policy
  paradigms, algorithms, backend topology, implementation candidates, and
  design alternatives in `DESIGN.md` unless one is itself an observable caller
  contract. Audit the exported interface with the same test: when a public
  field can be explained only through hidden implementation structure, replace
  it with caller-owned domain semantics or one opaque provider-owned value.
- Exclude request chronology, restated user prompts, unverified diagnoses,
  examples promoted into rules, and proposed mechanisms that were not adopted.
  Record a rejected alternative only when its rejection rationale still
  constrains future decisions.
- Keep both documents concise, stable, and current. Remove repetition,
  implementation narration, speculation, and filler.
- When repository-wide testing guidance needs a standalone owner, keep it in
  the repository root `TESTING.md`. Create or update it only for stable,
  non-obvious guidance shared across modules; do not use it to duplicate
  executable configuration, narrate individual tests, or hold details owned by
  one module.
- Use headings only for distinct topics that benefit navigation. Keep short,
  related content in one section.
- When a material change affects a necessary documented design constraint,
  update its owning document before implementation. Apply the same necessity
  test before creating a new document.
- Reconcile applicable existing documentation, tests, and implementation in
  the same change. Remove stale claims.
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

# Commits

- Keep task-process artifacts outside the repository. Research logs and notes,
  working plans and checklists, status or handoff summaries, generated reports,
  and other intermediate documents remain transient even when placed under
  `docs/`. Keep them in the conversation or an external temporary location;
  do not stage or commit them.
- Persist a task finding only when it becomes current, durable knowledge needed
  to use or maintain the repository. Rewrite that knowledge into its narrowest
  owning `README.md`, `DESIGN.md`, `TESTING.md`, or `AGENTS.md`; never preserve
  the investigation or planning process that produced it.
- Before staging or committing, inspect every added or modified documentation
  file. The change is ready only when each document owns current, durable
  repository knowledge; remove every task-process artifact from the working
  tree and exclude it from the commit.

# Formatting

- Use spaces for indentation, including Markdown code blocks. Follow tabs only
  where a canonical formatter or file format requires them.

# Language and framework rules

- For any task that reads, writes, reviews, or refactors Go code, read and
  follow [languages/go.md](./languages/go.md) before acting.
- For any task that reads, writes, reviews, or refactors TypeScript code, read
  and follow [languages/typescript.md](./languages/typescript.md) before acting.
- For any task that reads, writes, reviews, or refactors React code, read and
  follow both [languages/typescript.md](./languages/typescript.md) and
  [frameworks/react.md](./frameworks/react.md) before acting.
