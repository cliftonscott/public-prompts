# OODA (Observe → Orient → Decide → Act)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** A problem is **complex or ambiguous** and you want structured iteration: evidence → interpretation → one next move → small verified step.

Trigger phrases: `/ooda`, "ooda loop", "decide", "operator loop".

Aliases: decide, loop, operator.

---

## Purpose

Structured loop: gather evidence, interpret constraints, pick the **single safest next move**, act in **small verified steps**, repeat until done or blocked.

## When not to use

- Simple Q&A → [brief](brief.md)
- Read-only map only → [scout](scout.md)
- Known small fix → [patch](patch.md)

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--max-cycles N` | Stop after N cycles (default 3) and summarize |
| `--read-only` | Observe/Orient/Decide only; no Act |

Echo each cycle: **`OODA cycle N`**, then the four phases.

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). Each **Act** is small; no deploy or production mutation without explicit ask.

## Phases (per cycle)

### Observe

- Repo state: files, tests, logs, config.
- Separate **facts** from **assumptions**.

### Orient

- Map facts to architecture, constraints, likely causes.
- Narrow to 1–3 hypotheses ranked by likelihood.

### Decide

- Choose **one** next action: read more, run one check, or minimal patch.
- **Recommended:** state the action and why (one sentence).
- If blocked on a human-only decision, say so and stop.

### Act

- Execute only what was decided — smallest step.
- Run narrow validation after code changes.
- If Act fails, next cycle starts with new Observe data.

### Close

After done or max cycles: **Outcome**, **Evidence**, **Next Step**.

## Output format

```markdown
### OODA cycle <N>

**Observe**
- …

**Orient**
- …

**Decide**
**Recommended:** …

**Act**
- … (or _skipped — read-only_)

**Result**
- …
```

## Examples

- `/ooda API returns 500 for some users but not others`
- `/ooda --read-only Should we add an index or change the query?`
