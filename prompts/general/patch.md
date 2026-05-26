# Patch (smallest safe change)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want the **minimal** code change that satisfies a clear task—usually after [scout](scout.md).

Trigger phrases: `/patch`, "patch this", "fix", "implement", "small change".

Aliases: fix, implement, small-change.

---

## Purpose

Implement the **minimal** change: inspect relevant files first, preserve architecture, avoid drive-by refactors, update tests when appropriate, run focused validation.

## When not to use

- Scope unknown → [scout](scout.md) first
- Review only → [critique](critique.md)
- Full layer trace before any edit → [e2e](e2e.md)
- Production incident without root cause → your project's debug/RCA workflow first

**Note:** Some projects use `--patch` on a *domain* audit skill (security audit, codebase audit). This prompt is a **generic** minimal-change workflow, not those specialized modes.

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--scope <path>` | Limit edits to path(s) |
| `--no-test` | Skip tests only when the user explicitly accepts risk |

Echo at start: **`MODE: patch`**, **`SCOPE:`**, **`TASK:`**.

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). If the fix needs **more than ~3 production files** or crosses package boundaries without approval, stop and output a short plan for confirmation.

## Steps

1. **Quick inspect** — Read files you will touch; confirm APIs/paths exist; note existing tests.
2. **Plan in one paragraph** — What will change and what will not.
3. **Minimal edit** — Smallest diff; match naming, imports, and patterns.
4. **Tests** — Add/update when behavior changes; skip only for trivial doc/copy with no logic change.
5. **Validate** — Run the narrowest checks your project documents (lint, unit tests, build for touched packages).
6. **Summarize** — Files changed, validation run, residual risk. Deploy only if explicitly requested.

## Output format

```markdown
## Patch — <task summary>

**MODE:** patch

### What changed
- `path` — one line each

### Validation
- `command` — pass | fail (brief)

### Residual risk
…

### Next step
…
```

When fixing a bug, end with a short **Problem / Root Cause / Solution / Next Step** if your project uses that style.

## Examples

- `/patch Fix null guard when list item has no payload`
- `/patch --scope src/features/settings/ Apply scout plan step 2 only`

## Related

- [scout](scout.md) — inspect first  
- [another-pass](another-pass.md) — second pass on the same objective  
- [ship-chat](ship-chat.md) — open a PR when ready
