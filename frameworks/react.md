# React coding rules

Applies to any task that reads, writes, reviews, or refactors React code in a
project that adopts these rules. The global [AGENTS.md](../AGENTS.md) and
[TypeScript coding rules](../languages/typescript.md) provide the baseline;
more specific project rules take precedence when they conflict.
Project-specific frontend conventions such as the UI, request, and form
libraries live in the owning repository's `AGENTS.md`.

## Components and hooks

- Use function components and hooks. Keep components declarative: derive
  displayed state during render from the authoritative source instead of
  synchronizing state with effects.
- Components own their rendering and local state. Extract a custom hook only
  when the logic names a real domain operation that is reused or too large for
  one component; do not create a hook layer mechanically.
- Name hooks by their complete domain operation or returned concept. Avoid
  vague names such as `useData` or `useInfo` when the name omits the domain
  object or result.
- Hooks obey the rules of hooks. A dependency-suppression comment must name
  the verified contract that makes the omission safe.

## State and data flow

- Prohibit bidirectional state synchronization and feedback loops: no
  `useEffect` + `setState` mirrors, paired writers, write-back callbacks, or
  reconciliation loops between components.
- Keep data flow unidirectional: commands and mutation intent flow toward the
  owning data layer; results flow outward to render. A child must not write
  back into the parent's source of truth except through the parent's owned
  mutation interface.
- Let each observer own its independent reaction. Do not coordinate sibling
  effects or infer another flow's completion from local state.

## Requests and asynchronous state

- Route requests through the repository's owned request layer; do not call
  the raw request executor, `fetch`, or bare `axios` from business code.
- Handle the states the contract defines (loading, error, data) explicitly at
  the owner of the fetch; do not hide them behind silent fallbacks.
- Do not mask races with sleeps, timing margins, or guessed retries. Cancel or
  ignore stale results through the request layer's owned mechanism, or redesign
  the seam.

## Testing

- Test observable behavior through the public seam: render the component,
  interact with it, and assert what the user sees, not implementation details
  such as internal state or effect ordering.
