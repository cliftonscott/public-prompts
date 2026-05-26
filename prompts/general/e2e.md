# E2E (end-to-end trace)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You need to **trace a feature or bug across layers**—UI, API, backend, storage, tests, logs, deploy impact—before changing anything.

Trigger phrases: `/e2e`, "trace this", "full flow", "end-to-end".

Aliases: trace, full-flow, end-to-end.

---

## Purpose

Map a **complete behavior path** from user-visible action through clients, APIs, backend services, databases/rules, external integrations, tests, and observability. Verify claims against code. Report **gaps and missing links**. **Patch only if explicitly requested.**

## When not to use

- Small fix with a known file → [scout](scout.md) or [patch](patch.md)
- Review without a layer map → [critique](critique.md)
- Live production outage → your project's incident workflow

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--scope <path>` | Anchor starting area |
| `--save [path]` | Persist report to a file when requested |

Echo: **`MODE: e2e`** (read-only unless patch requested), **`FLOW:`** (one sentence).

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). Default: **no edits**. Production reads only when allowed; never paste secrets.

## Steps

1. **Define the flow** — Entry point (route, click, job, webhook) and expected outcome.
2. **Client layer** — UI, state management, navigation; note auth/session.
3. **API boundary** — Routes, payloads, error shapes.
4. **Backend logic** — Services, queues, schedulers; feature flags and config when relevant.
5. **Data layer** — Tables/collections, migrations, access rules; who can write what.
6. **Side effects** — Email, search, webhooks, analytics.
7. **Tests** — What covers this path; gaps.
8. **Observability** — Logs, metrics, dashboards worth checking (filters only—no "watch for hours" homework).
9. **Deploy impact** — Services, migrations, client release — **informational** unless deploy asked.
10. **Gaps** — Missing links, duplication, untested branches.
11. **Stop** — Unless user said implement, recommend [patch](patch.md) or [scout](scout.md).

## Output format

```markdown
## E2E trace — <flow name>

**MODE:** e2e | **FLOW:** …

### Layer map
| Layer | Components | Key files |
|-------|------------|-----------|

### Step-by-step path
1. …

### Tests and coverage
…

### Observability
…

### Deploy / release notes (informational)
…

### Gaps and risks
…

### Recommended next step
…
```

## Examples

- `/e2e Trace signup from form submit through database and welcome email`
- `/e2e Why scheduled job skips some records — full path`
