---
name: stack-commit
description: >-
  Commit all working-tree changes onto the current Graphite branch with
  `gt modify -c -a -m`, tracking the branch first if Graphite does not know it,
  writing the commit message from the actual diff, then offering to run
  `gt submit --stack --no-verify`. Use when the user asks to commit with
  Graphite, run gt modify, add a commit to the stack, or commit and submit
  the stack.
---

# Stack Commit

Create a new commit on the current Graphite branch from everything in the
working tree, with a message derived from what is actually being committed,
then ask before submitting the stack.

## Progress

```
Task Progress:
- [ ] Step 1: Ensure the branch is tracked
- [ ] Step 2: Inspect what will be committed
- [ ] Step 3: Write the commit message
- [ ] Step 4: Run gt modify -c -a -m
- [ ] Step 5: Offer gt submit --stack --no-verify
```

## Step 1: Ensure the Branch Is Tracked

`gt modify` refuses to run on a branch Graphite is not tracking. Verified
against gt 1.8.6, it exits with:

```
ERROR: Cannot perform this operation on untracked branch <name>.
You can track it by specifying its parent with gt track.
```

Nothing is committed when this happens, so check before committing rather than
after.

1. Read `gt state` (JSON: the trunk entry has `"trunk": true`; every tracked
   branch appears as a key). If `git branch --show-current` is absent from that
   output, the branch is untracked.
2. Track it with:

   ```bash
   gt track -f
   ```

   `-f` sets the parent to the **most recent tracked ancestor**, which skips the
   interactive parent prompt. For a branch cut from the root branch that
   ancestor *is* the root branch, so this gives the intended result — and for a
   branch stacked on another tracked branch it gives the correct stack parent
   instead. Prefer it over naming a parent by hand.
3. Read the confirmation line, which reports the chosen parent and the commit
   count it attributed to the branch:

   ```
   Tracked branch my-feature with parent main (includes 2 commits).
   ```

   Sanity-check that count against `git log --oneline` for the branch. If it is
   larger than the number of commits you actually wrote on this branch, the
   parent is wrong and the branch has swallowed another branch's commits — stop
   and fix it with `gt untrack -f` followed by `gt track` naming the real
   parent. Submitting in that state opens a PR containing someone else's work.
4. Only if `gt track -f` fails, fall back to tracking against the root branch
   explicitly. Discover the root branch — do not hardcode `main` or `master`:

   ```bash
   gt track --parent "$(gt trunk)"
   ```

5. If the repository itself is not initialized for Graphite (`gt state` errors),
   say so and stop. Do not run `gt init` on the user's repo without asking.

## Step 2: Inspect What Will Be Committed

Always stage everything with `-a`. Do not ask the user which files to include
and do not stage a subset by hand — but do look at the whole working tree
first, because `-a` commits all of it.

1. Run `git status --short` and `git diff HEAD --stat`.
2. Understand exactly what `-a` sweeps in. Verified against gt 1.8.6:
   - Modified tracked files, **plus deletions, plus untracked files** — this is
     broader than `git commit -a`, which only stages tracked changes.
   - Files matching `.gitignore` are **not** staged.
   - Anything already staged is included as well.
   - With nothing to commit, `gt modify` exits with `ERROR: No changes to commit.`
   - `-u` / `--update` would limit staging to tracked files; this skill does not
     use it unless the user asks.
3. Read the diff itself (`git diff HEAD`, not just `--stat`) plus the contents
   of any untracked files, so the message describes real behavior rather than
   filenames. For a large diff, read the stat first, then the diff of the files
   carrying the actual change.
4. Before committing, scan the untracked list for anything that clearly should
   not land in history — scratch files, `.env` or credential files, editor
   droppings, large binaries, debug output. If you see one, name it and ask
   whether to commit it, ignore it, or delete it. Do not silently sweep it in.
   Otherwise proceed without asking.
5. If `git status` is clean, say so and stop. Do not run `gt modify`.

## Step 3: Write the Commit Message

Write the message from the diff, not from the conversation's intent — what
ended up in the working tree and what was discussed can differ.

1. Match the repository's existing style. Check `git log --oneline -20` for
   conventional-commit prefixes, ticket keys, capitalization, and typical
   length, and follow whatever is dominant there.
2. Subject line: imperative mood, roughly 50–72 characters, no trailing period.
   It should say what the change does, not which files moved.
3. Add a body only when the change needs it — non-obvious reasoning, a
   behavioral consequence, or a constraint that explains the approach. Wrap at
   72 characters. Skip the body for small, self-evident changes.
4. Do not list every touched file, do not restate the diff, and do not include
   co-author or tool-attribution trailers unless the repository's own history
   or instructions use them.
5. If the working tree holds several unrelated changes, write a message that
   honestly covers them and say so in the report. Because this skill commits
   everything, splitting the work into separate commits is the user's call —
   offer it, but do not stall waiting for an answer.

## Step 4: Run gt modify

Run the command with the message filled in (single `-m` for a subject-only
message; a second `-m` for the body):

```bash
gt modify -c -a -m "Subject line here"
```

1. Keep `-c` and `-a` on every invocation: `-c` creates a new commit instead of
   amending the branch's existing one, `-a` stages the full working tree.
2. `gt modify` restacks descendants automatically. If it reports a conflict,
   stop and hand off to the `merge-conflicts` skill instead of improvising a
   resolution.
3. If it still reports an untracked branch, return to Step 1 — do not work
   around the error with a plain `git commit`.
4. Confirm the result with `git log -1 --stat` and `git status --short`. The
   working tree should be clean afterwards apart from ignored files.
5. If the commit fails a hook, report the failure and the hook output. Do not
   retry with `--no-verify` unless the user asks.

## Step 5: Offer to Submit

Ask the user explicitly, in chat:

> Committed as `<subject>` on `<branch>`. Run `gt submit --stack --no-verify`?

1. Wait for a clear yes. Submitting pushes branches and creates or updates pull
   requests — never run it on assumption, on silence, or because it was
   approved earlier in the session.
2. On yes, run:

   ```bash
   gt submit --stack --no-verify
   ```

3. Report the branches pushed and the PR URLs it prints.
4. On no or no answer, stop with the commit in place and say the stack was not
   submitted.

## Report

Summarize:

- The branch and the commit message used
- Whether the branch had to be tracked, and the parent it was tracked against
- What was included, calling out any untracked files that `-a` swept in
- Whether the restack succeeded
- Whether the stack was submitted, and the resulting PR links
