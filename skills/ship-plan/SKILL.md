---
name: ship-plan
description: >-
  Finish an implemented plan unattended: verify, simplify-pr, no-comments, verify,
  pr-review, fix-pr-findings, verify, then push a branch and open the PR. Use as
  the last step of every code-changing plan, or when the user says ship it, land
  it, or finish and open the PR. Safe to run in a cloud agent with nobody watching.
---

# Ship plan

Run after the plan's implementation steps are done. Nobody may be watching: do
not stop to ask, and do not report success you did not observe. Every gate below
must actually pass before moving on.

## Verify command

Prefer, in order:

1. A `verification` skill in this repo or personal skills, if present
2. `npm run verify` / `pnpm verify` / `yarn verify` when that script exists
3. Otherwise the repo's documented CI gate (`test`, `check`, `lint`, etc.)

Never loosen or skip the gate to get past it. If the repo documents a live /
runtime check (probe, e2e, screenshot pass), run it for every surface the diff
touches and record what you ran and saw.

## Evidence (when the repo enforces it)

If the repo has an `artifacts/ship-plan/` convention, `npm run check:ship`,
`npm run verify:record`, or a hook that blocks PR creation until evidence
exists, follow that repo's rules exactly:

- Write each step's report under `artifacts/ship-plan/` by **invoking the
  skill** and saving its output. Never hand-write evidence to bypass a hook.
- Each report that the hook requires should start with
  `Commit: <git rev-parse HEAD>` (full sha) for the commit it covered.
- Re-run review steps if `src/` (or the repo's equivalent) changes after review.

If the repo has **no** evidence hook, still run the pipeline below and put the
review / fix-pr-findings output in the PR body — just skip the artifact files
and `check:ship` unless the repo asks for them.

## Pipeline

1. **Verify** (see above). Fix → rerun until green. Run any documented live
   check for surfaces the diff touches.
2. **`simplify-pr`** on the scoped diff. Apply in-scope cuts. Save a report
   when the repo expects evidence files.
3. **`no-comments`** on the scoped diff. Commit the cuts.
4. **Verify again** (and re-run live checks if runtime code changed).
5. **`pr-review`** against the default base branch (`main` / `master`), on a
   committed HEAD. Keep the full ranked findings (or an explicit "no findings"
   line) for the PR body and for `fix-pr-findings`.
6. **`fix-pr-findings`** using those findings. Fix only in-scope, worth-it
   items. Commit fixes before recording the report's `Commit:` line when
   evidence is required.
7. **Verify a final time** on the committed, clean HEAD you will push. Prefer
   `npm run verify:record` when that script exists. Re-run live checks if fixes
   touched runtime code.
8. **Push and open the PR** (below).

Skip steps 2–3 only for pure docs/tooling changes that touch no application
source (typically `src/`). Never skip a verify step.

If a gate cannot be made green after a real attempt, stop, push the branch
anyway, open the PR as a **draft**, and lead the body with what failed and the
exact output. A draft with an honest failure beats a green-looking PR.

## Push and open the PR

- Branch: `git switch -c <short-kebab-name>` from the current base if still on
  the default branch. Never push to `main` / `master`.
- Commit with focused messages. Let pre-commit hooks run; do not `--no-verify`
  unless the user asks.
- `git push -u origin HEAD`, then open the PR with `gh pr create` (or the
  host's equivalent). Use `.github/pull_request_template.md` when present.
- If push/PR tools fail, keep the pushed branch (if any), write the body to
  `artifacts/pr-body.md` when useful, and end with the branch name and compare
  URL. Do not claim a PR exists.
- PR body sections (adapt names to the template): **Summary**, **Verification**
  (commands + results, including live checks), **Review pass** (`pr-review`
  findings verbatim plus `fix-pr-findings` fixed/hollered/skipped),
  **Skipped / needs a human** (every Holler and Skip, named individually).
- After opening, read CI once. If the verify job failed, fix and push; do not
  merge and do not enable auto-merge.
- **Final chat message** must include the `pr-review` verdict/finding count and
  every hollered/skipped item by name — not just the PR link.

Done means: verify was green in this session, the PR URL exists (or draft with
honest failure), the body has the review sections above, and deferred findings
are named in chat.
