# Another pass

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want a deliberate second pass on the agent's most recent work—gaps, edge cases, verification, and polish—without restarting from zero.

Trigger phrases: `/another-pass`, "another pass", "follow-up review", "one more time", "double-check", or similar.

---

## Goal

Re-execute the **same objective** the agent just finished (or was working toward), with a **different lens**: deeper correctness, missed requirements, consistency with project conventions, and evidence-backed verification.

## Recover the objective

1. From the transcript, identify the **last concrete user goal**: what "done" meant (feature, fix, doc, refactor, answer).
2. If ambiguous, state your **best one-sentence restatement** of that objective, then proceed. Only ask the user a clarifying question if two interpretations would change the work materially.

## What this pass is not

- Not a blank-slate rewrite unless the first pass was wrong or the user asked to redo.
- Not unrelated scope creep—stay tied to that objective.

## Checklist

Work through these in order; skip items that clearly do not apply (e.g. no code → skip build steps).

1. **Requirements fit** — Does the result fully match the original ask and implicit constraints (platform, files touched, "minimal change," etc.)?
2. **Correctness** — Logic, types, null/error paths, security and auth boundaries where relevant.
3. **Edge cases** — Empty states, failures, retries, offline vs production assumptions.
4. **Project alignment** — Naming, patterns, contributing guides and agent instruction files, existing abstractions (reuse over reimplement).
5. **Verification** — Run the smallest set of checks that prove the change: targeted tests, analyze/lint/build as appropriate to what changed. If checks cannot run, say what blocked them and what you validated by inspection.
6. **Residual risk** — Brief note of anything still uncertain or deferred.

## Output

- Lead with **what changed** in this pass (or "no material gaps found") in a few bullets.
- If you fixed issues, mention **files modified** briefly.
- Do not duplicate the entire first-pass explanation unless the user needs it for context.
