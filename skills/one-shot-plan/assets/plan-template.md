# Plan template

After lock, write a complete plan with **no open questions**. Adapt section titles to the work; keep every section below or an explicit “N/A — \<reason\>”.

```markdown
# <Plan title>

## Goal

One paragraph: what ships and why.

## Locked decisions

| ID    | Decision | Source                       |
| ----- | -------- | ---------------------------- |
| 1 / A | …        | user / default / approve all |

## Today’s world (brief)

What exists now that the implementer must respect.

## Approach

Ordered steps. Name files, modules, types, and call order. Prefer concrete paths over vague “wire up X”.

## Data / contracts

Types, schemas, API shapes, persistence. “None” only with reason.

## Failure behavior

What happens on error, empty, partial, conflict. User-visible vs logged.

## Docs

Exact paths to add/update, or **Docs: none** + one-line reason. Same PR as code when the host repo expects that.

## Acceptance tests

Checklist a weaker model can verify (commands, scenarios, assertions). For
runtime / UI / gameplay changes, include a **live-check recipe** when the host
repo documents one (exact command, assertions, what screenshots or outputs must
show). If project rules require moving a decision out of a UI/presentation
layer, name the pure function it moves to and the unit test that fails without
the change.

## Out of scope

Explicit non-goals from the interrogation (v1 cuts).

## Implementation order

Numbered sequence the implementer should follow. End with:

1. Required verification for the change (project verify docs / `npm run verify`
   / equivalent), including any live-check recipe from Acceptance tests.
2. **Run the `ship-plan` skill**. Do not stop before the PR exists.
```

### Quality bar

- A weaker model can implement from this plan alone without asking preference questions.
- No TBD, “Option A vs B”, “prefer X if Y”, or “optional” left unresolved.
- Every Decide answer and every approved Proposed lock appears in **Locked decisions**.
- Code-changing plans end with the `ship-plan` step.
