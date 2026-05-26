# Plan review

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want to review a `.plan.md` (or similar implementation plan) in a revise loop until it is **ready to build** or you hit an iteration cap.

Trigger phrases: `/plan-review`, "review this plan", "plan review loop", or attach `@path/to/plan.md`.

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `<path>` | From chat context | Explicit path to the plan file |
| `--max-iterations <n>` | `5` | Stop after N review rounds |
| `--second-opinion` | off | Use a separate reviewer (second model, MCP, or read-only CLI) instead of reviewing in the same chat |

---

## Purpose

Review a plan markdown file, apply revisions, repeat until **`READY_FOR_BUILD`** (or max iterations).

| Path | Mechanism |
|------|-----------|
| **Default** | You review the plan in this session (read working copy, apply patch or manual edits). |
| **`--second-opinion`** | A **read-only** external reviewer returns a structured verdict + optional unified diff; you apply changes to the working copy only. |

## Constraints

1. Edit **only** the target plan file (via a working copy, then copy back).
2. Do **not** implement product or source code during plan review.
3. Keep full plan fidelity — do not truncate or substitute the plan.
4. Include the absolute **`PLAN_FILE`** path in user-visible outputs.

---

## Step 1 — Identify plan file

Use only a plan explicitly referenced in this chat:

- Path argument, or
- attached / `@`-referenced `.plan.md`, or
- explicitly named plan path.

Strip flags (`--max-iterations`, `--second-opinion`) when detecting the plan path.

If none: ask for a plan path or attachment and stop.

Resolve **`REPO_ROOT`**: `git -C "$(dirname "$PLAN_FILE")" rev-parse --show-toplevel` (plan must live under the repo).

## Step 2 — Initialize tracking

- `ITERATION = 1`
- `MAX_ITERATIONS = 5` (or `--max-iterations` override)
- `USE_SECOND_OPINION = true` if `--second-opinion` in the user message
- `SUMMARY_OF_PREVIOUS_CHANGES = ""`
- `TEMP_DIR = /tmp/plan-review` (or a project temp dir)
- `WORKING_PLAN = <TEMP_DIR>/<basename>.working.plan.md`
- `PLAN_FILE` = absolute path to the real plan

Create todo entries per iteration if your workflow uses them.

## Step 2.5 — Preflight

1. Confirm `PLAN_FILE` exists.
2. Create `TEMP_DIR` if missing.
3. If **`USE_SECOND_OPINION`**: confirm your reviewer is available (read-only CLI, MCP tool, or API). If missing, report and stop or ask to fall back to default (same-session review).

## Step 3 — Review loop

Loop while `ITERATION <= MAX_ITERATIONS`.

### 3.1 — Working plan

- Iteration 1: `cp <PLAN_FILE> <WORKING_PLAN>` (no need to load the full plan into chat first).

### 3.2 — Prior summary

For `ITERATION > 1`, keep a compact `previousIterationSummary` (1–3 short lines).

### 3.3 — Run review

**Default (same session):** Read `WORKING_PLAN` from disk. Review against:

- Clarity, scope, risks, and test/verification plan
- Fit with project instructions (`AGENTS.md`, `CONTRIBUTING.md`, architecture docs) when present
- Concrete, minimal revisions over broad rewrites

Apply the **Reviewer output contract** below to your own verdict.

**`--second-opinion`:** Invoke your read-only reviewer with:

- Absolute path to `WORKING_PLAN`
- `iterationNumber`, `maxIterations`, `previousIterationSummary`
- Project instruction sources (paths, not pasted secrets)
- The **Reviewer output contract** (send as instructions; reviewer must not edit files)

If your repo has a **read-only review script** (e.g. Codex relay), use it in read-only mode; correlate the run’s report to **this** plan path before trusting output. On timeout or ambiguous correlation, retry once or fall back to same-session review.

Print: `Iteration N/MAX — reviewing <WORKING_PLAN>`

### 3.4 — Parse verdict

From the reviewer (or your own review), extract:

1. **`VERDICT`:** After a line containing only `---`, the next line must be exactly:
   - `VERDICT: READY_FOR_BUILD`, or
   - `VERDICT: NEEDS_REVISION`
2. **`NEEDS_REVISION` + patch:** Section `## Plan patch (unified diff)` — unified diff for `patch --batch --forward -p0`
3. **`NEEDS_REVISION` + fallback:** Section `## Reviewer instructions` (or `## Cursor instructions`) — minimal manual edits when patch is missing or fails

If verdict is missing or ambiguous: treat as tool failure; do not apply changes from that round.

### 3.5 — Branch handling

#### `READY_FOR_BUILD`

1. `cp <WORKING_PLAN> <PLAN_FILE>`
2. Print short summary + verdict.
3. Stop.

#### `NEEDS_REVISION`

1. If patch non-empty: write `<TEMP_DIR>/<basename>.iter-<N>.patch`, run `patch --batch --forward -p0 -i <patch_file>` on **`WORKING_PLAN`**. On success: update summary, increment `ITERATION`, continue loop.
2. Else if reviewer instructions: apply **minimal** edits to **`WORKING_PLAN`** only; increment `ITERATION`; continue.
3. Else: report failure; run **When NOT READY_FOR_BUILD**; stop.

### 3.6 — Tool failure

On reviewer timeout, spawn error, or parse failure:

1. Print diagnostics (plan path, iteration, error).
2. Do **not** apply revisions from that round unless the operator explicitly asks.
3. Run **When NOT READY_FOR_BUILD**.

## Step 4 — End summary

Always print: iterations attempted, review path (same-session vs second-opinion), absolute **`PLAN_FILE`**.

### When NOT READY_FOR_BUILD

1. `cp <WORKING_PLAN> <PLAN_FILE>` when the working copy was updated.
2. **Summary of updates:** what changed across iterations.
3. **Re-run:** offer exact plan path; mention `--second-opinion` if the default path struggled.

---

## Reviewer output contract

Give this block to a **read-only** reviewer (or follow it yourself). The reviewer must **not** edit files or implement code.

```markdown
Read the plan from the absolute path provided in the session header. Read-only.

## Scope

- Judge clarity, scope, risks, and fit with project instructions when provided.
- Prefer concrete, minimal revisions over broad rewrites.
- Do not implement product code or expand scope beyond reviewing this plan.

## Output contract (required)

Write analysis in prose, then end with this machine-parsed trailer:

1. A horizontal rule: a line containing only `---`
2. On the **next line**, exactly one of:
   - `VERDICT: READY_FOR_BUILD`
   - `VERDICT: NEEDS_REVISION`
3. If and only if `NEEDS_REVISION`, include after that line:
   - `## Plan patch (unified diff)` — optional complete unified diff for `patch --batch --forward -p0` against the plan file
   - `## Reviewer instructions` — optional prose when a mechanical diff is not practical

If `READY_FOR_BUILD`, omit optional sections.

Do not put `VERDICT:` anywhere except immediately after the `---` separator.
```

---

## Example invocations

```text
@plan-review @docs/plans/my-feature.plan.md
@plan-review .cursor/plans/my_feature.plan.md --max-iterations 3
@plan-review @docs/plans/my-feature.plan.md --second-opinion
```

---

## Manual validation checklist

- Iteration 1 uses `cp` to `WORKING_PLAN`.
- Revisions: `patch` first, manual only on failure.
- Only the plan file is edited; no product code.
- Second-opinion runs are correlated to the correct plan path before applying patches.
