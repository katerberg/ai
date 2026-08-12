---
name: fix-lockfile
description: >-
  Repair a broken or out-of-sync pnpm lockfile, reinstall dependencies, and
  verify tests pass. Use when pnpm-lock.yaml has merge conflicts, install fails
  with lockfile errors, package.json and the lockfile disagree, CI fails on
  frozen-lockfile, or the user asks to fix the lockfile / reinstall deps.
---

# Fix Lockfile

Repair `pnpm-lock.yaml`, reinstall with pnpm, and confirm the relevant test
suite passes. Do not commit or push unless the user explicitly asks.

## Progress

```
Task Progress:
- [ ] Step 1: Diagnose
- [ ] Step 2: Repair lockfile
- [ ] Step 3: pnpm install
- [ ] Step 4: Verify tests
- [ ] Step 5: Report
```

## Step 1: Diagnose

1. Confirm the package manager is pnpm (`packageManager` in root `package.json`,
   or presence of `pnpm-lock.yaml`). If the repo uses npm/yarn instead, adapt
   the same workflow to that lockfile and stop after stating what you used.
2. Discover **every** `pnpm-lock.yaml` in the repo (root and nested). This repo
   commonly has separate lockfiles at:
   - `/pnpm-lock.yaml` (Meteor app + `packages/*` workspace)
   - `apps/vibe/pnpm-lock.yaml`
   - `apps/mcp/pnpm-lock.yaml`
   - `services/*/pnpm-lock.yaml` when present
3. Run `git status` and note which of those lockfiles (and related
   `package.json` files) are conflicted, modified, or missing.
4. Inspect failure signals **per lockfile**, not only at the root:
   - Conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
   - Install errors mentioning an outdated or incompatible lockfile
   - CI / local `--frozen-lockfile` failing
5. Run `pnpm install --frozen-lockfile` in **each** directory that owns a
   lockfile (correct Node via `.nvmrc` / `nvm use`). A green root install does
   **not** prove nested lockfiles are in sync.
6. Read changed `package.json` files. Prefer regenerating lockfiles over
   hand-editing them.

### Shared `packages/*` caveat (required)

`apps/vibe` and `apps/mcp` are independent pnpm workspaces that include shared
packages via `pnpm-workspace.yaml` (e.g. `../../packages/legos`,
`../../packages/foundry`, `../../packages/palette`, and for mcp also
`../../packages/common`).

If a change updates any of those shared `packages/*/package.json` files (or a
dependency they declare), you **must** also refresh every nested lockfile whose
workspace includes that package — even when the root lockfile is already
correct. Dependabot often updates only the root lockfile and misses
`apps/vibe/pnpm-lock.yaml` / `apps/mcp/pnpm-lock.yaml`.

## Step 2: Repair Lockfile

Choose the smallest safe repair for each affected lockfile:

1. **Merge conflict in a `pnpm-lock.yaml`:**
   - Do not try to manually merge lockfile JSON/YAML by hand.
   - Accept one side as a base only if needed to clear conflict markers, then
     regenerate with install (Step 3). Prefer deleting conflict markers by
     checking out a clean base version of the lockfile, then reinstalling:
     ```bash
     git checkout --ours pnpm-lock.yaml   # or --theirs if that side has the dependency intent
     pnpm install
     ```
   - If both sides changed dependencies, keep the intended `package.json`
     resolutions (usually the merged `package.json`), then regenerate the
     lockfile entirely via `pnpm install`.
2. **Out of sync / corrupt lockfile (no conflict markers):**
   - In that lockfile's directory, run `pnpm install` so pnpm rewrites the
     lockfile to match its workspace manifests.
   - Only as a last resort: remove that `pnpm-lock.yaml` and reinstall. Say so
     in the report if you do this.
3. **Which directories to repair:**
   - Always repair the root lockfile when root / `packages/*` manifests changed.
   - Also repair `apps/vibe` and/or `apps/mcp` when their lockfile fails
     frozen-lockfile **or** when a shared package they include (`packages/legos`,
     `foundry`, `palette`, `common`) changed — do not skip them because root
     already passed.
   - Repair `services/*` (or other nested workspaces) when that package's own
     manifests/lockfile changed.
4. Do not bump dependency versions unless required to make install succeed.
   Prefer restoring a consistent lockfile for the current `package.json`.
   Keep nested regenerations scoped: after `pnpm install`, review the diff and
   avoid shipping unrelated version churn if it appears; re-check against the
   branch's intended manifests.

## Step 3: pnpm Install

1. Install in every directory that needed repair (or failed frozen-lockfile):

   ```bash
   # root
   pnpm install

   # nested examples — only when that workspace is in scope
   (cd apps/vibe && pnpm install)
   (cd apps/mcp && pnpm install)
   ```

2. If install still fails:
   - Fix the underlying `package.json` issue (bad range, missing peer, private
     registry auth) with the smallest change.
   - Re-run `pnpm install` until it succeeds.
3. Re-run `pnpm install --frozen-lockfile` in **each** touched lockfile
   directory. Confirm no conflict markers remain and `git status` shows a
   coherent lockfile change (or no change if already correct).
4. Do not use `--force` / wholesale cache wipes unless install cannot succeed
   otherwise; if you do, note it in the report.

## Step 4: Verify Tests

1. Discover the default fast test command from `package.json` scripts and any
   `AGENTS.md` / nested workspace instructions (e.g. `pnpm run vitest`,
   `pnpm test`, workspace-specific Vitest/Go tests).
2. Run suites for every workspace whose lockfile you repaired or revalidated:
   - Root / `packages/*` → root `pnpm run vitest` (and package checks if docs
     require them)
   - `apps/vibe` lockfile touched → `pnpm test` from `apps/vibe`
   - `apps/mcp` lockfile touched → `pnpm test` from `apps/mcp`
   - `services/*` → that service's test script
3. If tests fail because of the lockfile/install (missing modules, wrong
   versions, resolution errors), fix and re-run install + tests.
4. If failures look pre-existing and unrelated to dependency resolution, do not
   expand scope into product fixes unless the user asks — report them clearly.
5. Prefer the fast unit suite. Only run slower full suites when the repo docs
   require them for dependency changes or when unit tests cannot exercise the
   affected workspace.

## Step 5: Report

Summarize:

- What was wrong with the lockfile
- How it was repaired (ours/theirs base, regenerate, delete+reinstall, etc.)
- Whether `pnpm install` succeeded
- Which tests ran and whether they passed
- Remaining risk (skipped slow suites, pre-existing failures, auth/registry issues)

Do not commit unless asked.
