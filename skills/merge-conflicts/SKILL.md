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

## Step 4: Report

Summarize:

- Which files conflicted and how each conflict was resolved
- Which checks were run and whether they passed
- Any behavioral uncertainty or risk introduced
- Whether the Graphite operation completed successfully

Be explicit when no meaningful risk was identified. Never claim the resolution
is risk-free if checks were skipped or intent remained ambiguous.
