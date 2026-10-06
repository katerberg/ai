---
name: simplify-pr
description: >-
  Audit a PR or branch diff for agent-style complexity: oversized classes,
  premature abstractions, duplicated helpers, ceremony, and other additive
  smells. Prefer findings that remove, simplify, or consolidate. Use when
  implementing a plan, or when the user asks to simplify a PR, declutter a
  diff, hunt agent bloat, reduce complexity, or consolidate overlapping code
  before merge. On repos that require it, run before the no-comments skill on
  every code-changing plan.
---

# Simplify PR

Hunt additive agent bloat on a scoped diff. Goal: **delete, simplify, or
consolidate** — not a general correctness or security review.

When the host repo's plans require this skill, run it after
implementation/verification and **before** `no-comments`. Skip only for pure
docs/tooling with no application-source edits.

## Scope

Use the caller's files or diff. Otherwise diff against the base branch
(default `main`), including the working tree:

```bash
git diff main...HEAD --stat
git diff main...HEAD
git diff main  # working tree vs base when uncommitted
```

Large diffs (>~2000 lines): review path-by-path. Prefer application source;
skip pure lockfile/noise unless it encodes a new dependency smell.

If `docs/ARCHITECTURE.md`, `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/`, or
`.agents/` guidance exists, read them before judging structure — project rules
beat generic taste.

## Stance

- Bias toward **less code**. Every finding should name something to remove,
  shrink, merge, or inline.
- Flag **new** complexity harder than pre-existing neighbors unless the PR
  made the neighbor worse.
- Do not invent refactors outside the diff fence.
- Do not pad. If three real smells exist, report three.
- Prefer concrete edits (`delete X`, `inline Y into Z`, `merge A+B`) over
  taste lectures.

## Smell lenses (agent-additive)

Work the diff with these lenses. Details and examples:
[references/smells.md](references/smells.md).

| Lens                      | Look for                                                                                      |
| ------------------------- | --------------------------------------------------------------------------------------------- |
| **Size**                  | Files/classes/functions that grew fat; modules doing too many jobs                            |
| **Premature abstraction** | Manager/Facade/Helper/Utils/Base* wrapping one call site; generic “framework” for one feature |
| **Duplication**           | Near-copy of an existing helper; parallel constants/types                                     |
| **Indirection**           | Pass-through functions, option bags used once, config objects with one consumer               |
| **Ceremony**              | Speculative extension points, unused params, dead branches, TODOs that ship code              |
| **Layer violations**      | Logic in the wrong layer per project architecture; god mutable state                          |
| **Additive deps**         | New packages without a concrete need stated in the PR                                         |

Also treat project hard rules (from AGENTS / Architecture) as always-actionable
when in scope.

## Steps

1. Gather the scoped diff and skim project convention docs when present.
2. For each changed file, apply the smell lenses. Trace call sites for new
   helpers/classes — one consumer usually means inline or delete. **Exception:**
   keep a pure, unit-tested extraction when project rules require moving
   decisions out of UI/presentation layers (cite the rule).
3. Rank findings by **simplification value** (bytes/concepts removed, fewer
   types/layers, clearer ownership).
4. **Apply** every actionable in-scope finding (remove / simplify /
   consolidate). Smallest edits only; do not widen the fence. Leave open only
   items that need product judgment or would change behavior outside the PR
   intent — name each in the report.
5. Re-scan the scoped diff once after edits. Fix any new obvious additive
   smells introduced by step 4.
6. Report using the format below. Plan/implementation gates are not done while
   actionable in-scope findings remain unapplied.
7. If code changed, run the repo's verify / verification skill as appropriate.
   Then proceed to the `no-comments` skill when this was a plan gate.

**Report-only mode:** if the user asks only to audit/review (no apply, not a
plan gate), stop after the ranked report and do not edit.

## Output format

Lead with a one-line verdict (`clean`, `a few cuts`, `needs consolidation`).

Then a ranked list (applied and left-open):

```markdown
## N. <short title>

**Action:** remove | simplify | consolidate
**Status:** applied | open
**Where:** `path/to/file.ts` L<a>-L<b> (and related paths if consolidating)
**Smell:** <lens name>
**Why:** <1-2 sentences — what the additive pattern is costing>
**Do this:** <concrete edit: delete / inline / merge into X / move rule to Y>
```

End with:

- **Keep:** anything in the diff that looks rightly sized (optional, brief)
- **Out of scope:** smells noticed outside the fence (optional, one line each)

## Never

- Turn this into a full PR review (bugs/security/tests) — stay on simplify.
- Propose new abstractions to “clean up” complexity.
- Recommend dependencies as the fix.
- Claim architecture violations without citing the project rule.
- Skip this skill on a code-changing plan when the host repo requires it, while
  still claiming the plan done.
