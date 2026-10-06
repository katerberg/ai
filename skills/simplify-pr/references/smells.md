# Agent-additive smells (reference)

Read from the simplify-pr skill when judging borderline cases.

## Size

- **Fat module**: one file owning unrelated concerns (I/O + business rules +
  presentation bookkeeping).
- **Fat class**: new class with many methods when a few functions would do;
  “service” objects that only hold collaborators.
- **Long function**: deep nesting, multi-phase procedural blobs — split only if
  the split removes duplication; otherwise simplify the logic in place.

Heuristic (not a hard rule): new or heavily edited units over ~150–200 lines,
or functions over ~40–60 lines, deserve a hard look — but size alone is not a
finding without a simpler shape.

## Premature abstraction

Classic agent residue:

- `FooManager` / `FooService` / `FooController` / `FooHandler` for one feature
- `BaseFoo` / `AbstractFoo` / `IFoo` with a single implementer
- `createFooFactory` / registry / plugin hooks with one registration
- Wrapper that only forwards to a library API
- “Sync engine” or mirror framework instead of one local map at the boundary

Prefer: delete the type, keep the function; or inline at the single call site.

## Duplication / consolidation

- Same clamp/countdown/math already elsewhere in the tree
- Parallel helpers that should share one module
- Copy-pasted checks instead of extending an existing helper
- Near-identical UI/update blocks that should share a style or component

Prefer: consolidate into the existing helper; delete the new twin.

## Indirection & ceremony

- Function whose body is one call with renamed args
- Options/config object constructed once, read once, never extended
- Feature flags or strategy enums with a single live branch
- Unused parameters “for future use”
- Dead code paths left beside the new path
- Comments that narrate what the next line does (out of scope for this skill —
  use the `no-comments` skill)

## Layer & architecture

Flag when the PR introduces or worsens layering that the project's
`docs/ARCHITECTURE.md` / `AGENTS.md` forbid. Common patterns (only when they
match the host repo):

| Bad                                                 | Prefer                                      |
| --------------------------------------------------- | ------------------------------------------- |
| Business rules in UI / scene / view code            | Domain helper or framework-free system      |
| Framework imports in a “pure” domain layer          | Keep domain pure                            |
| View objects as source of truth for state           | Model/state layer; render mirrors           |
| New manager hierarchy around a simple data setup    | Keep the repo's existing composition style  |
| Global mutable gameplay / app singleton             | Explicit ownership (world, context, params) |
| New npm dependency for a 20-line helper             | stdlib / existing code                      |

Cite the project doc when filing these.

## Additive dependency smell

A new package is a smell unless the PR clearly needs it and nothing in-tree
suffices. Finding should say: remove the dep and keep the small local helper,
or justify why local code is worse.

## What not to flag

- Necessary bridge/adapter files that must touch the framework
- Small pure helpers even if numerous — composition is wanted
- Tests that look “verbose” but pin behavior
- Pre-existing debt untouched by the PR
- Style nits already owned by formatter/lint

## Action verbs

| Action          | Means                                                       |
| --------------- | ----------------------------------------------------------- |
| **remove**      | Delete the symbol, file, branch, or dependency              |
| **simplify**    | Shrink in place: flatten control flow, drop options, inline |
| **consolidate** | Merge two+ overlapping units into one existing home         |
