# Brief (short answer)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want the **shortest useful answer**—recommendation or fact first, no fluff.

Trigger phrases: `/brief`, "be brief", "concise", "short", "no fluff".

Aliases: concise, short, no-fluff.

---

## Purpose

Answer in the **fewest clear sentences**: **lead with the recommendation or fact**, then only essential context. No throat-clearing, no repeated summaries, no deep architecture unless asked.

## When not to use

- User asked for audit, runbook, or full RCA depth
- Implementation requested → [patch](patch.md) or normal agent mode
- Need file discovery → [scout](scout.md)
- Need severitized findings → [critique](critique.md)
- Want the agent to **pick a path and own it** → [wwyd](wwyd.md)

## Parameters (optional)

| Flag | Description |
|------|-------------|
| `--bullets` | ≤6 bullets; first bullet is the answer |
| `--command` | One copy-paste command block when directly useful |

Echo: **`MODE: brief`** (optional one line).

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). **Do not edit files** unless the user also asks to implement. Do not invent commands or paths.

## Steps

1. **Answer first** — First line = takeaway or **Recommended:** …
2. **Essential context only** — One short paragraph or ≤6 bullets.
3. **Code/commands** — Only when needed to run something; one block max.
4. **Stop** — No "here's what I'll do", no duplicate closing summary.

## Output format

**Default:**

```markdown
<One-line answer or **Recommended:** …>

<Optional 1–3 sentences of essential context.>
```

**With `--bullets`:** ≤6 bullets; first is the answer.

## Examples

- `/brief Do we need a database index for this query?`
- `/brief --command How do I run the project's test suite for one package?`
