# WWYD (What would you do?)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want the agent’s **concrete recommendation** for the current task or decision—not an unprioritized option list and not the decision pushed back unless a human-only choice is required.

Trigger phrases: `/wwyd`, `wwyd`, “what would you do”, “WWYD”, “pick a path and own it”.

**Apply mode:** `/wwyd --do` or “wwyd --do” — same reasoning, then **execute** what the agent recommended (edits, checks, PRs, etc.).

Invoke this prompt **explicitly**; it is not meant to run on every message.

---

## Goal

Give **your** call: one primary recommendation and why.

## When not to use WWYD

Prefer **brainstorming / plan mode / requirements exploration** when the goal is still fuzzy, there is no concrete next move, or incompatible product directions need discovery first. WWYD is for **picking the next action or approach** once the thread (or repo context) is enough to commit.

Do **not** treat WWYD as a shortcut around mandatory **pre-reads** for the task (see Anti-patterns).

## Principles (when multiple options exist)

- State **Recommended:** one option plus one sentence why.
- Lead with the takeaway; stay brief unless stakes are high or the user asked for depth.
- Before claiming work is **complete**, **fixed**, or **passing** (especially in `--do` mode), run relevant checks and cite evidence—do not assert success from memory alone.
- After doing work, run verification yourself; avoid long manual-only follow-up lists for the human.

---

## `--do` (apply mode)

When the user invokes **`/wwyd --do`**, **`wwyd --do`**, or clearly signals **apply what you would do**:

1. Run the same steps as default **anchor → recommend → why** (briefly; one recommended path).
2. **`--do` preflight** — Do not execute on **half-known** state: if the uncertainty gate below applies, resolve it first (one message, then stop). Otherwise proceed.
3. **Then execute** that path: edits, commands, tests, deploy only if already in task scope, etc.—whatever you would do next if you owned the task.
4. **Still stop** for things that require explicit human approval (e.g. irreversible data loss, production toggles the user did not ask to change, secrets). State the blocker in one line and do the rest.
5. End with a short **what shipped / what ran**; if you changed code or fixed a bug, include a brief outcome summary (what failed, why, what you did).

If `--do` was **not** given, stay in recommendation-only mode (optional one-line offer to implement; no execution).

---

## Instructions

1. **Anchor on context** — Use the active task, files, constraints, and project rules (`AGENTS.md`, `CONTRIBUTING.md`, etc.) already in the thread. If **one** essential fact is missing, ask **one** focused question; otherwise proceed.
2. **Uncertainty gate** — If **two or more** blocking unknowns exist, do **not** recommend or execute as if you know the answer. Send **one** message listing up to **three** concrete questions or assumptions; then **pause** until the user answers or confirms. In **`--do`** mode, do **not** implement until that gate is cleared.
3. **State what you would do** — Lead with the **single** action or approach you would take next, in plain language.
4. **If multiple options exist** — **Recommended:** … plus one sentence why; mention other paths only when they materially differ.
5. **Keep it proportional** — Short ask → short answer. Deeper tradeoffs only when high-stakes or the user asked for depth.
6. **Optional close** (default mode only) — You may offer to implement the recommended path in one line; do not turn this into a long manual checklist.
7. **Self-check before sending** — Exactly one primary action? No unprioritized option dump? If `--do`, did you run (or schedule) verification instead of handing work back?

---

## Output shape (default)

- **What I would do:** …
- **Why:** …
- **If wrong assumption:** … (one line, or omit if none)

**Optional (when stakes or ambiguity warrant):**

- **Confidence:** high | medium | low
- **Reversibility:** easy revert | needs review/PR | irreversible — human decision only

Skip throat-clearing (“Here’s what I’ll do…”); get to the recommendation.

In **`--do`** mode, lead with **What I'm doing** (or **What I did** after execution), then **Why**; keep prose short and let diffs/commits carry detail.

---

## Examples (shape only)

**Default**

- **What I would do:** Add the missing database index (or config entry) the error message names, then re-run the failing test.
- **Why:** The failure points at indexing/config, not application logic.
- **Confidence:** high

**`--do`**

- **What I'm doing:** Adding the index entry, running the project’s index/schema tests, then summarizing what passed.
- **Why:** Fastest fix that matches the logged failure mode.

---

## Anti-patterns

- **Substituting WWYD for pre-reads** — Do not skip project instructions, security rules, or task-specific docs the change would normally require; WWYD picks among approaches, not whether to read the repo.
- **Bypassing irreversible gates** — No WWYD or `--do` override for data loss, security changes the user did not ask to ship, or production config they did not request to change.
- **Fake confidence** — If the uncertainty gate fired, do not proceed with a firm recommendation without labeling assumptions or waiting for answers.

Pure recommendation-only WWYD with no failure story and no edits does not need a formal postmortem block; use one when you explain a bug, tradeoff after a decision, or **any code change**.
