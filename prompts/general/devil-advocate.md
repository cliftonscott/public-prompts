# Devil's advocate (challenge ideas, no edits)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want the agent to **push back** on an idea, plan, or solution instead of defaulting to agreement — assumptions, failure modes, and honest verdict, **without edits** unless you ask separately.

Trigger phrases: `/devil-advocate`, "devil's advocate", "push back", "stress-test", "challenge this idea", "what am I missing?", "convince me this is right".

Aliases: push-back, stress-test, challenge.

---

## Purpose

Doesn't always agree with the user. Play **devil's advocate** on ideas, solutions, architecture choices, product bets, and tradeoffs: name what could be wrong, what was skipped, and what a skeptic would argue — without being performatively contrarian or rude.

## When not to use

| Situation | Use instead |
|-----------|-------------|
| Prioritized review of a diff, plan file, or UI spec | [critique](critique.md) |
| Pick one path and own it | [wwyd](wwyd.md) |
| Shortest neutral answer | [brief](brief.md) |
| Implement now | [patch](patch.md) or [wwyd](wwyd.md) with `--do` after pushback |
| Second pass on work already done | [another-pass](another-pass.md) |
| Formal proof after a change | [prove](prove.md) |

## Not the same as

| Prompt | Difference |
|--------|------------|
| [critique](critique.md) | Reviews **artifacts** (diff, files, plan) with severities; `red-team` there means code/design review |
| [wwyd](wwyd.md) | **Recommends** one path; devil's advocate **opposes or complicates** before you choose |
| [brief](brief.md) | Compresses an answer; does not argue against you by default |
| [another-pass](another-pass.md) | Re-runs the **same completed objective** for gaps; not idea stress-testing |

## Parameters (optional)

| Flag | Default | Description |
|------|---------|-------------|
| `--mild` | off | One strongest objection + one mitigation; abbreviated output |
| `--no-steel` | off | Skip steel-man; go straight to assumptions and failure modes |
| `--alt` | off | End with 1–2 concrete alternative approaches |

Echo: **`MODE: devil-advocate`** (one line), **`TOPIC:`** (what is being challenged).

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). No file edits, deploys, or production mutations unless the user separately requests patch/apply.

## Steps

1. **Restate the claim** — One sentence: what the user is proposing or assuming.
2. **Steel-man (default)** — Strongest honest version of their position (2–4 sentences). Skip if `--no-steel`.
3. **Challenge assumptions** — List 2–5 unstated premises ("this assumes …"). Mark **load-bearing** premises (if false, the plan breaks).
4. **Failure modes** — How this fails in practice: edge cases, ops pain, cost, security, adoption, maintenance, team fit, irreversibility.
5. **Counter-arguments** — What a smart skeptic would say; cite repo/docs evidence when the topic is technical.
6. **What would change my mind** — Falsifiable signals or cheap experiments (not vague "more research").
7. **Verdict** — `weak` | `mixed` | `solid` with one-line why — **do not** default to `solid` to please the user.
8. **Optional `--alt`** — 1–2 alternatives with tradeoffs; **Recommended:** one if you'd bet against the original.

## Tone

- Argue the **idea**, not the person.
- Be direct; no filler agreement ("great idea, but…").
- If the proposal is actually strong, say so — then give the **best** remaining objection anyway.
- `--mild`: cap at one killer objection + one mitigation.

## Output format

```markdown
## Devil's advocate — <topic>

**MODE:** devil-advocate | **TOPIC:** …

### Your claim
…

### Steel-man
…

### Assumptions challenged
1. … (**load-bearing**)
2. …

### Failure modes
- …

### Skeptic's case
…

### What would change my mind
- …

### Verdict
**<weak|mixed|solid>** — …

### Recommended alternative (if `--alt` or clearly better path exists)
**Recommended:** … — …
```

**With `--mild`:** use only **Your claim**, **Strongest objection**, **Mitigation**, **Verdict**.

## Examples

- `/devil-advocate We should cache all user profiles in the client`
- `/devil-advocate --mild Ship feature X behind a feature flag only`
- `/devil-advocate --alt Replace WebSocket with polling for this screen`
- `/devil-advocate --no-steel --mild Skip the nice version — just tell me why this is risky`
