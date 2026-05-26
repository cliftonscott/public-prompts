# Scout (inspect before edit)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want the agent to **read the repo first** and produce a smallest safe plan **without changing any files**.

Trigger phrases: `/scout`, "scout this", "inspect first", "reconnaissance", "plan-first".

Aliases: inspect, reconnaissance, plan-first.

---

## Purpose

**Read-only** reconnaissance for a concrete task: what to touch, how it works today, what could break, and the smallest safe implementation path. **Stop before patching.**

## When not to use

- Trivial one-line fix where the file is already known
- Whole-repo architecture audit (use a dedicated audit prompt if your project has one)
- Live production outage (use your project's incident/debug workflow)
- You only want a short factual answer → [brief](brief.md)

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--scope <path>` | Limit search/read to path(s) |
| `--save [path]` | Write the report to a file when requested |

Echo at start: **`MODE: scout`** (read-only), **`SCOPE:`**, **`TASK:`** (one sentence).

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). **Scout never edits code, configs, secrets, or deploys.**

## Steps

1. **Anchor the task** — Restate the goal and definition of done in one sentence.
2. **Read instruction sources** — Contributing guides, `AGENTS.md` / `CLAUDE.md` (if present), architecture docs, security rules.
3. **Locate surfaces** — Search for symbols, routes, APIs, feature folders; list **likely files** with one-line roles.
4. **Trace the flow** — Happy path: trigger → handlers → reads/writes → side effects. Note client vs server vs data boundaries.
5. **Patterns** — Existing abstractions to reuse; conventions to follow.
6. **Risks and unknowns** — Auth, data integrity, config flags, tests, observability; label **verified** vs **assumed**.
7. **Smallest safe plan** — Ordered steps (≤8 bullets).
8. **Stop** — Do not patch. Suggest [patch](patch.md), [e2e](e2e.md), or [ooda](ooda.md) as next step.

## Output format

```markdown
## Scout — <task summary>

**MODE:** scout | **SCOPE:** …

### Task
…

### Likely files and roles
| Path | Role |
|------|------|

### Current flow (short)
…

### Patterns to follow
- …

### Risks and unknowns
| Item | Severity | Notes |
|------|----------|-------|

### Smallest safe plan
1. …

### Recommended next step
patch / e2e / critique / ooda — one line with task hint
```

## Examples

- `/scout Add user preference sync to the settings screen`
- `/scout --scope src/api/ Why list endpoint returns empty for some IDs`
