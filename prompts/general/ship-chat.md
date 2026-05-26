# Ship chat

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want to push session work, open a PR, and optionally run the full ship pipeline (review wait, CI, merge).

Trigger phrases: `/ship-chat`, "ship this", "push and open a PR", "commit and PR", or similar.

**Flags (when supported):**

- **Default:** Commit only files you edited or created in this conversation. After PR creation, run **Post-ship pipeline** unless `--pr-only`.
- **`--all`:** Commit all modified and untracked files in the active shipping context.
- **`--pr-only`:** Stop after PR creation — no Copilot wait, CI watch, or merge.
- **`--review`:** Request an automated PR review (e.g. Cursor Bugbot) — omit unless the user asked.
- **`--worktree`:** Force ship from a new **ship** worktree even when in-place would be safe.

---

## Speed rules (read first)

**Goal:** reach `git push` in one agent turn when commits already exist on a feature branch.

| Phase | Rule |
|-------|------|
| **Before push** | One `git fetch`. Diff + log only. **Do not** run full local CI unless the user explicitly asked — CI after push is the merge gate. |
| **Push + PR** | Prefer your repo’s **ship push script** if one exists (e.g. `scripts/ship-chat-push.sh`); otherwise manual `git push` + `gh pr create`. |
| **After push** | Copilot/review wait + `gh pr checks --watch` + merge (unless `--pr-only`). Waiting belongs here. |
| **Discovery** | Current worktree / branch first. Scan other worktrees only if ambiguous or the parent chat has no local edits. |
| **Worktrees** | Commit during implementation in a **feature** worktree. Never create `ship-*` when commits already exist there. |

Do **not** re-fetch `origin/main` in a separate “pre-push” step after discovery or after the push script.

---

## When to use

User wants to: push existing work, create a branch, commit, open a PR, and (by default) complete the full ship pipeline through squash merge.

Applies to:

- **Feature worktree already done** — commits on `feat/*` or `feature/*` in `.worktrees/…` (common after subagents or parallel worktrees).
- **Uncommitted chat edits** — files changed in this session on `main` or a branch.
- **Background / coordinator sessions** — parent chat may have zero local edits; commits may live only in a linked worktree.

## Preconditions (post-ship pipeline)

Skip when `--pr-only`.

- `gh` authenticated with repo write access (`gh auth status`).
- For Copilot wait (optional): `gh extension install k1LoW/gh-copilot-review` and Copilot PR review enabled on the repo.

## Branch naming

Use **namespace/slug-in-kebab-case**:

- Namespaces: `feat`, `feature`, `fix`, `hotfix` — **prefer matching an existing branch** on the worktree you ship.
- Examples: `feat/operator-dashboard`, `feature/image-upload-policy`, `fix/feed-score-calculation`
- If a feature worktree already has a branch name, **reuse it**.

## Worktree roles

| Worktree | Purpose |
|----------|---------|
| **Feature worktree** (e.g. `.worktrees/feat-my-change`) | Implementation; commits live here. **Ship from here** when commits exist. |
| **Ship worktree** (e.g. `.worktrees/ship-feature-my-change`) | Only when **main is dirty** and you commit a **subset** without touching unrelated pending files. |

Do **not** create `ship-*` and copy files when work is already committed in a feature worktree.

For **new** worktrees (not shipping), use your repo’s worktree/bootstrap scripts if available, or:

```bash
git worktree add .worktrees/<branch-sanitized> -b <branch> main
```

## Workflow overview

```
0. Discovery (minimal)
   ├─ Feature worktree ahead of main, clean? → FAST PATH → push script or manual push [→ post-ship]
   └─ Else → commit path (steps 1–5) → push [→ post-ship]

Post-ship pipeline (unless --pr-only) — waits happen here
Worktree cleanup (when no longer needed)
```

---

## Step 0 — Discovery (minimal)

**Prefer the active worktree / branch:**

```bash
git status -sb
git fetch origin main
git log origin/main..HEAD --oneline
```

If that shows commits ahead of `origin/main` on a feature branch with a clean tree → **fast path** (below).

**Only if needed** (parent chat with no edits, or multiple candidates):

```bash
git worktree list
# For ambiguous non-main worktrees only:
git -C <worktree-path> log origin/main..HEAD --oneline
```

### Fast path — ship existing feature worktree

Use when a non-main worktree has commits ahead of `origin/main`, a clean tree, and matches the user's goal.

- **One** clear candidate → ship it.
- **Multiple** → ask user which worktree/branch.
- **None** → Step 1.

**Fast path steps:**

1. From repo root: run your ship push script with `--dir <worktree-path>` if supported (add `--review` if requested), else push from that worktree manually.
2. Parse output for `PR_URL`, `PR_NUMBER`, `BRANCH` (or read from `gh pr list`).
3. **Unless `--pr-only`:** **Post-ship pipeline**.
4. **Worktree cleanup** when no longer needed.
5. Do not checkout `main` in the main repo if that would disturb unrelated work.

**Do not:** create `ship-*`, bulk `cp`, re-fetch before push, or run local build/test before push.

Record `featureWorktreePath = <worktree-path>` for cleanup.

---

## Step 1 — Determine what to commit (commit path only)

Skip on fast path.

- **Default:**
  1. List every file you modified or created in this conversation. If unsure, ask the user.
  2. Run `git status` in the tree where edits live (main or active worktree).
  3. **Scan `git status` for related paths** not edited in this chat. Include when they meet the criteria below. When in doubt, ask the user; never stage `.env`, build outputs, or secrets.
     **Should include:**
     - **Plan files:** Any `*.plan.md` (or your project’s plan convention) untracked or modified — commit with implementation.
     - **Same feature/area** as a chat-edited file.
     - **Tests** targeting the same feature.
     - **Co-located / part files** changed together by convention.
     - **Direct dependency** (imports/callers) that are part of the same change.
     - **Project config** (`package.json`, `pubspec.yaml`, etc.) if the chat added a dependency the change references.
  4. **Check for plan files** — every plan file in `git status` must be in the commit list.
  5. Reconcile:
     - **Include:** Chat-edited paths in `git status` plus related paths from step 3.
     - **Exclude:** Unrelated modified/untracked files.
     - **Drop:** Chat-edited paths not in `git status` (reverted/unsaved).
- **`--all`:** All modified and untracked files in the active tree; confirm `git status` is non-empty.

If nothing to commit and no ahead worktree → stop (see edge cases).

Keep this step **short** — do not run full validation here.

---

## Step 2 — Choose shipping strategy (commit path only)

From `git status` in the tree holding uncommitted work, collect modified/untracked paths (ignore ignore-worthy only-deletes).

| Condition | Strategy |
|-----------|----------|
| Ahead commits, clean tree | **Fast path** (step 0) |
| `--worktree` flag | **Ship worktree** |
| Reconciled list ≠ all pending paths | **Ship worktree** |
| Else | **In-place** |

---

## Step 3 — Branch name (commit path only)

Pick `namespace/slug` or **keep current feature branch**.

---

## Step 4 — Execute: ship worktree

When Step 2 chose ship worktree (including `--worktree`).

1. Ensure `.worktrees` is gitignored.
2. `git worktree add .worktrees/ship-<sanitized> -b <branch> main` (or your project’s worktree helper).
3. If branch exists elsewhere → use that worktree or add without `-b`.
4. Stage via git (prefer stash over `cp`):
   - Option A: From main, `git stash push -m ship-chat -- <paths>`, in ship worktree `git stash pop`, commit.
   - Option B: `cp` only reconciled untracked paths (last resort).
5. `git add <path>` per reconciled file; commit.
6. Run ship push script from ship worktree, or `git push -u origin <branch>` + `gh pr create`.
7. **Unless `--pr-only`:** Post-ship pipeline.
8. Do not checkout main in main repo.
9. **Worktree cleanup:** remove `ship-*` worktree after successful push/PR.

Record `shipWorktreePath = .worktrees/ship-<sanitized>` for cleanup.

---

## Step 5 — Execute: in-place

1. On `main` without branch → `git checkout -b <branch-name>`; else keep branch.
2. Stage reconciled paths; commit if needed.
3. Ship push script or manual push + PR.
4. **Unless `--pr-only`:** Post-ship pipeline; then `git checkout main && git pull origin main`.
5. **`--pr-only`:** `git checkout main`.

---

## Push and PR

**Prefer** a repo ship script that fetches once, shows `git log` / `git diff --stat`, pushes, opens a PR if missing, and prints `PR_URL` / `PR_NUMBER` / `BRANCH`.

**Manual fallback:**

1. `git push -u origin <branch-name>`
2. `gh pr list --head <branch-name> --json number,url -q '.[0]'` — do not duplicate create.
3. **Create PR** with **PR defaults** below.

### PR defaults

- **Title:** Top commit subject or concise summary.
- **Body:** HEREDOC with `## Summary` and `## Test plan` (checkboxes).
- Return PR URL.

---

## Post-ship pipeline (default)

Skip when `--pr-only`.

Announce: `Running post-ship pipeline: wait for review, fix feedback, CI, squash merge.`

**Local validation** runs here only when fixing review/CI failures — not before the initial push.

### 1. Resolve execution context

- `<pr-number>`, `<branch-name>`, worktree vs in-place checkout.
- **Record paths** for cleanup: `shippingWorktreePath`, `shipWorktreePath`, `featureWorktreePath`.
- Fixes run in PR branch checkout.
- Review bot id (e.g. `copilot-pull-request-reviewer[bot]` if using GitHub Copilot).

### 2. Review loop (max 3 rounds)

For each round (1–3):

1. **Wait for automated review** — e.g. `gh copilot-review --wait <pr-number> --wait-timeout 10min --wait-interval 30sec`; round 2+ add `--force` or re-request reviewer per your setup.

2. **Collect feedback** — triage actionable items from the review bot or human threads.

3. **No actionable feedback** → proceed to CI.

4. **Fix, validate, commit, push** — scoped validation for the fix; reply/resolve threads.

5. **Stop without merge:** `needs-human`, still actionable after round 3, or unfixable push/validation failure.

### 3. Wait for CI green

```bash
gh pr checks <pr-number> --watch --fail-fast
```

On failure: diagnose, fix, push, re-watch. Sync with `main` if the PR is behind (`git merge origin/main` or your repo’s sync workflow).

### 4. Squash merge

When review loop is clean and checks green:

```bash
gh pr merge <pr-number> --squash --delete-branch
gh pr view <pr-number> --json state,mergedAt,mergeCommit
```

- **In-place:** `git checkout main && git pull origin main`
- **Worktree:** `git fetch origin main` in the shipping worktree (if still present).

### 5. Worktree cleanup (default — do not skip)

Run from **repo root** after push succeeds (`--pr-only`) or after merge succeeds (full pipeline).

**Always remove** a **`ship-*` worktree** created for this run once push + PR steps succeed.

**Remove a feature worktree** when **all** are true:

- PR is **MERGED** (full pipeline), **or** `--pr-only` and push + PR creation succeeded
- Worktree working tree is **clean**
- No unpushed commits after fetch
- Pipeline did not stop early (review/CI/merge failure)

**Do not remove** when pipeline stopped early or worktree has unpushed/uncommitted work.

```bash
git fetch origin --prune
git worktree remove .worktrees/ship-feat-my-change
# After merge:
gh pr view <pr-number> --json state -q '.state'
git worktree remove .worktrees/feat-my-change
git branch -d feat/my-change 2>/dev/null || true
git worktree list
```

---

## Edge cases

- **No chat-edited files in parent chat:** Minimal discovery; ship ahead worktree via fast path.
- **`--all` and nothing pending:** Tell user; do not ship.
- **Unrelated files on main:** Reconciled subset → ship worktree.
- **Branch already exists:** Fast path; check existing PR before creating.
- **Push or PR fails:** Stop; no post-ship.
- **Subagent / background work:** One ahead worktree → fast path without scanning every worktree.

## Output

**`--pr-only`:** PR URL, branch name, worktree cleanup result.

**Default (full pipeline):** PR URL, review rounds summary, CI status, merge result, worktree cleanup result.
