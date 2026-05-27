# Critique (senior review, no edits)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want a **senior-engineer review** of code, a plan, UI, or design—with prioritized findings and concrete fixes, **without edits** unless you ask separately.

Trigger phrases: `/critique`, "critique this", "review", "audit", "red-team".

Aliases: review, audit, red-team.

---

## Purpose

Review **what the user points at** (diff, files, plan, UI description, architecture sketch): bugs, edge cases, security, regressions, maintainability, performance, UX. **Prioritize by severity.** **Do not edit** unless explicitly asked.

## When not to use

- You want implementation → [patch](patch.md) (optionally "apply FINDING-001 only")
- You want pushback on an **idea** before code exists → [devil-advocate](devil-advocate.md)
- Whole-repo architecture survey → a dedicated audit prompt if your project has one
- Plan iteration to READY_FOR_BUILD → [plan-review](plan-review.md)

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--scope <path>` | Limit review to paths |
| `--plan <path>` | Review a specific plan file |
| `--severity min` | Output only `critical` and `high` when set |

Echo: **`MODE: critique`** (read-only), **`TARGET:`**.

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). No file edits, deploys, or production mutations unless the user separately requests patch/apply.

## Steps

1. **Identify target** — Diff, files, plan, or UI spec from the user message.
2. **Read context** — Surrounding code, tests, security rules, docs.
3. **Evaluate** — Correctness, edge cases, auth, data integrity, performance, maintainability, UX/a11y, observability.
4. **Findings** — Each: **ID**, **severity** (`critical`|`high`|`medium`|`low`), **location** (`path:line` or symbol), **evidence**, **impact**, **concrete fix**.
5. **Prioritize** — Critical → low; cap long lists (top 5 per lower tier).
6. **Summary** — Count by severity; **Recommended:** one finding to fix first + why.
7. **Stop** — Offer patch with finding ID if implementation is wanted.

## Output format

```markdown
## Critique — <target>

**MODE:** critique | **TARGET:** …

### Summary
| Severity | Count |
|----------|-------|

### Findings

#### FINDING-001 — <title>
- **Severity:** …
- **Location:** `path:line` — `symbol`
- **Evidence:** …
- **Impact:** …
- **Fix:** …

### Recommended first fix
**Recommended:** FINDING-00N — …
```

## Examples

- `/critique Review this PR diff for regressions`
- `/critique --plan docs/plans/feature-x.plan.md`
