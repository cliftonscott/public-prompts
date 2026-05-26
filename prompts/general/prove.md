# Prove (evidence before done)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want **runnable proof** that recent changes work or the problem is fixed—not "should work." The agent runs checks, may fix failures in a bounded loop, and reports an explicit verdict.

Trigger phrases: `/prove`, "prove it", "show me it works", "verify your fix", "is it fixed", "proof", "demonstrate".

Aliases: verify, demonstrate, show-me-it-works.

---

## Purpose

Close the loop on **the current thread's work**: define what "working" means, **run** checks that can fail, **fix** when proof fails (unless `--no-fix`), and end with **PROVEN**, **NOT_PROVEN**, **BLOCKED**, or **WAIT_WITH_AUTOMATION**. Do not hand back manual-only checklists when the agent can run or script the check.

## When not to use

| Situation | Use instead |
|-----------|-------------|
| Scope unknown — nothing to verify yet | [scout](scout.md) then [patch](patch.md) |
| Rollout/canary/shadow **promotion** over time | Your project's proof-to-completion / rollout workflow |
| Review-only | [critique](critique.md) |
| Broader quality pass without a formal verdict | [another-pass](another-pass.md) |
| Recommendation only | [wwyd](wwyd.md) (add `--do` to execute) |

## Not the same as

| Prompt | Difference |
|--------|------------|
| [another-pass](another-pass.md) | Broader requirements/edge-case pass; no formal PROVEN verdict |
| [patch](patch.md) validation | Patch runs checks once; prove owns proof condition + fix loop |
| Generic "verify before done" habit | Prove is an explicit command with structured output |

## Parameters (optional)

| Flag | Default | Description |
|------|---------|-------------|
| `--scope <path>` | thread-driven | Limit proof to package or file |
| `--no-fix` | off | Report `NOT_PROVEN` only; do not edit code |
| `--max-attempts <n>` | `3` | Fix-and-reprove loops before `BLOCKED` |

Echo at start: **`MODE: prove`**, **`OBJECTIVE:`** (one sentence), **`PROOF:`** (measurable condition).

## Safety

Follow [workflow-skills shared safety](workflow-skills.md#shared-safety-all-workflow-prompts). Production reads only when your project allows; no secret paste. No deploy or production config writes unless explicitly requested in the same message.

## Steps

1. **Recover the objective** — What was built or fixed; **done when** (test green, API returns X, UI shows Y). One clarifying question only if proof path differs materially.
2. **Choose proof methods** — Smallest set that can fail: lint/analyze, targeted tests, build, project smoke scripts, read-only logs or staging checks when relevant. Use your repo's documented validation commands.
3. **Run proof** — Execute checks **in this turn**; cite pass/fail and the smallest failing excerpt.
4. **On failure** — If `--no-fix`: `NOT_PROVEN` + evidence + fix hypothesis. Else: minimal fix ([patch](patch.md) discipline), re-run proof, repeat until **PROVEN** or max attempts → **BLOCKED**.
5. **Deferred proof** — If proof needs future traffic: run what you can now, add or reuse an automated script/CI job with explicit pass/fail, report how to invoke it and where output lands → **WAIT_WITH_AUTOMATION** (not **PROVEN**).

## Output format

```markdown
## Prove — <short objective>

**MODE:** prove | **VERDICT:** PROVEN | NOT_PROVEN | BLOCKED | WAIT_WITH_AUTOMATION

### Proof condition
- …

### Evidence
| Check | Result | Notes |
|-------|--------|-------|
| `command` | pass / fail | … |

### Fixes applied (if any)
- `path` — one line each

### If not PROVEN
- What failed | what blocks proof | exact next action
```

When proving a bugfix, end with **Problem / Root Cause / Solution / Next Step** if your project uses that style.

## Validation expectations

- **Never** claim PROVEN without at least one executed check you cite.
- Prefer fixing over manual homework; cap loops with `--max-attempts`.

## Examples

- `/prove` — prove whatever we just changed in this thread
- `/prove --scope src/features/bookmarks/` — narrow checks
- `/prove --no-fix` — evidence only, no edits
- `/prove Confirm analyze and widget tests pass after the null-guard fix`

## Related

- [patch](patch.md) — implement the change  
- [scout](scout.md) — inspect when proof path is unclear  
- [another-pass](another-pass.md) — quality pass after PROVEN  
- [workflow-skills](workflow-skills.md) — index and chains
