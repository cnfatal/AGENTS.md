# TypeScript coding rules

Applies to any task that reads, writes, reviews, or refactors TypeScript or
React code in a project that adopts these rules. The global
[AGENTS.md](./AGENTS.md) provides the baseline; more specific project rules
take precedence when they conflict. Project-specific frontend conventions (UI
library, request layer, form library) live in the owning repository's
`AGENTS.md`.

## Exported surface

- Decide visibility from the module's public interface and caller value, not
  from file placement. Export only what external callers must name, construct,
  configure, or branch on.
- Keep implementation details unexported, even when several files collaborate
  through them. `index.ts` names the module's public API when the module has
  one; do not use it to re-export internal helpers.
- Treat every exported declaration as a compatibility commitment: document it
  and test its observable behavior through the public seam.

## Types and naming

- Write TypeScript types the way a static compiled language (Go, Java, C#)
  writes them. Every type feature used must have a direct compiled-language
  counterpart and must lower encoding or comprehension cost; do not use
  TypeScript-only type-level programming.
- Allowed: `string`, `number`, `boolean`, `null`, `undefined`, `void`, arrays
  and tuples, object types (`interface` / `type` literal), `enum`, `class`,
  generics, and ordinary literal values with explicit annotations.
- Use generics as a compiled language does: a concrete type parameter on a
  container or operation that removes real duplication, such as `List<T>` or
  `Optional<T>`. The instantiation must stay obvious at the call site; a
  generic a reader must trace through to understand its callers fails the
  comprehension test.
- Prohibited TypeScript-only type machinery with no compiled-language
  counterpart: union types (`A | B`), intersection types (`A & B`), conditional
  types, mapped types, template literal types, indexed access types
  (`T['key']`), utility types such as `Partial`, `Omit`, `Pick`, `Record`,
  `ReturnType`, and `Awaited`, `keyof` and `typeof` in type position, `as
  const`, `satisfies`, and type predicates (`x is T`).
- `any` / `unknown` are allowed where a compiled language would use `Object`
  or `interface{}`: a genuine external seam whose concrete type is unknown,
  such as interop, deserialization, or a heterogeneous container, and for
  temporary local transition while such values are being narrowed. Narrow or
  cast the value to its domain type before it leaves the function; never
  expose `any` / `unknown` on a public interface, domain type, or stored
  state. Prefer `unknown` over `any` when the value must be narrowed, because
  `any` silently disables checking downstream.
- Model domain state as an `enum` plus an explicit `switch`, as a compiled
  language does. Do not model state as unions or discriminated unions.
- A cast or a non-null assertion is prohibited unless a verified contract
  requires it; do not use them for convenience.
- Declare exactly what current callers need. Do not write speculative types,
  options, or extension points because a type is reusable.
- Name domain types by their domain concept, such as `Organization`. Name an
  operation's domain input with the operation plus `Input`, optional query or
  execution controls with `Options`, and domain results as the domain concept
  or the operation plus `Result`.
- Name variables and parameters by their role in the local operation, not by
  repeating a type already explicit from the declaration.

## Package and file organization

- Organize files by owned behavior. Do not create generic `utils.ts`,
  `helpers.ts`, or `types.ts` files; keep a small helper with its primary
  caller and a type with the interface or behavior that owns it. A shared
  `types.ts` is justified only when a substantial set of domain types is owned
  by several equally important modules.
- Treat an implementation file and its test as one maintained pair:
  `Foo.tsx` with `Foo.test.tsx`. Do not create a test file without its
  implementation, and rename or move the test together with the
  implementation.
- Components own their rendering and local state. Extract a custom hook only
  when the logic names a real domain operation that is reused or too large for
  one component; do not create a hook layer mechanically.

## Code style

- Format with the repository's configured Prettier settings. Indentation uses
  spaces, not tabs.
- Name functions and hooks by their complete domain operation or returned
  concept. Avoid vague names such as `process`, `handle`, `fetchData`, or
  `getInfo` when the name omits the domain object or result.
- Use one term for one concept consistently across components, hooks, types,
  tests, and user-facing contracts.

## React

- Use function components and hooks. Keep components declarative: derive
  displayed state during render from the authoritative source instead of
  synchronizing state with effects.
- Prohibit bidirectional state synchronization and feedback loops: no
  `useEffect` + `setState` mirrors, paired writers, write-back callbacks, or
  reconciliation loops between components.
- Keep data flow unidirectional: commands and mutation intent flow toward the
  owning data layer; results flow outward to render. A child must not write
  back into the parent's source of truth except through the parent's owned
  mutation interface.
- Let each observer own its independent reaction. Do not coordinate sibling
  effects or infer another flow's completion from local state.
- Hooks obey the rules of hooks. A dependency-suppression comment must name
  the verified contract that makes the omission safe.

## Async and data flow

- Route requests through the repository's owned request layer; do not call
  the raw request executor, `fetch`, or bare `axios` from business code.
- Handle the states the contract defines (loading, error, data) explicitly at
  the owner of the fetch; do not hide them behind silent fallbacks.
- Do not mask races with sleeps, timing margins, or guessed retries. Cancel or
  ignore stale results through the request layer's owned mechanism, or redesign
  the seam.

## No defensive programming

- Validate a domain rule once where its owner establishes the value (for
  example, the schema at the API boundary), then trust the typed value
  downstream. Do not add speculative null checks, optional-chaining sprinkles,
  or `??` fallbacks once an invariant is established.
- `?.` and `??` express contract-required absence, not defensive masking.
- Do not add tests solely to justify speculative defensive branches; test
  required behavior and reproduced failures.

## Serialization contracts

- Add schema definitions or tags (Zod, JSON Schema) only where the type is
  part of a wire, persistence, or validation contract. Keep protocol decoding
  models separate from verified domain results when their representations
  differ.
- Define domain types explicitly by hand; do not derive them through type
  operations such as `z.infer`. Keep the schema and the domain type
  consistent, and assert that consistency with a test.

## Testing

- Test observable behavior through the public seam: render the component,
  interact with it, and assert what the user sees, not implementation details
  such as internal state or effect ordering.
- Regression tests must assert the required invariant at the public seam. A
  passing test does not justify an implementation whose correctness depends on
  timing, scheduling, or undocumented behavior.
