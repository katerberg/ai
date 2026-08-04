---
name: merge-conflicts
description: >-
  Restack a Graphite branch when needed, resolve Git merge conflicts
  conservatively, continue the interrupted operation with gt continue, and
  summarize the resolution and its risks without pushing. Use when the user asks
  to fix merge conflicts or continue a conflicted Graphite operation.
---

# Merge Conflicts

Resolve the current merge conflict conservatively. Preserve the intent of both
sides when they are compatible, and avoid unrelated cleanup or refactoring.

## Step 1: Inspect the Conflict

1. Run `git status` to identify the interrupted operation and conflicted files.
2. If there is no interrupted operation or conflict, inspect the Graphite stack.
   If the current branch needs restacking, run `gt restack`, then restart this
   skill from Step 1. If no restack is needed, report that there is no conflict
   to resolve and stop.
3. Read each conflicted file, its conflict markers, and enough surrounding code
   to understand both sides.
4. Inspect relevant history or diffs when the intended resolution is unclear.
5. Ask the user a focused question only when the correct behavior cannot be
   determined safely from the code and history.

## Step 2: Resolve Conservatively

1. Resolve every conflict with the smallest change that preserves intended
   behavior.
2. Do not discard either side solely because it is "ours" or "theirs."
3. Do not make unrelated edits, reformat files unnecessarily, or change public
   behavior beyond what the conflict requires.
4. Follow repository instructions and run checks appropriate to the files
   changed.

## Step 3: Continue with Graphite

1. Stage only the resolved files.
2. Run `gt continue`.
3. If Graphite reports another conflict, repeat this workflow.
4. Verify the operation completed and inspect the final status and diff.
5. Do not push.

## Step 4: Verify Against Semantic Conflicts

Lint and type checks alone do not catch semantic conflicts — code where both
sides merge cleanly (or typecheck cleanly) but interact incorrectly at
runtime. After the operation completes:

1. Run the test suites that cover the conflicted files and any files both
   branches touched — not just the fast unit suite. If the repository's CI or
   preflight runs additional suites (e.g. a slow integration or Meteor/Mocha
   suite), run those too, or explicitly report them as not run.
2. If a suite is too slow to run in full and cannot be scoped, say so in the
   report rather than implying it passed.
3. Treat any new failure as part of the conflict resolution: diagnose whether
   it comes from the interaction of the two sides and fix it before reporting
   success.

## Step 5: Report

Summarize:

- Which files conflicted and how each conflict was resolved
- Which checks and test suites were run and whether they passed, and which
  suites were skipped
- Any behavioral uncertainty or risk introduced
- Whether the Graphite operation completed successfully

Be explicit when no meaningful risk was identified. Never claim the resolution
is risk-free if checks were skipped or intent remained ambiguous.
