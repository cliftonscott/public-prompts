# Workflow skills (index)

**Tool:** General (Cursor, Claude Code, Codex, ChatGPT, etc.)  
**Use when:** You want a quick map of the inspect → change → prove → review slash-style prompts and how they chain together.

Trigger phrases: "workflow skills", "scout patch prove", links from other prompts in this repo.

---

Seven complementary prompts for everyday coding. Paste one file per task, or chain them.

| Prompt | Edits files? | Summary |
|--------|----------------|---------|
| [**scout**](scout.md) | No | Inspect before edit; smallest safe plan |
| [**patch**](patch.md) | Yes | Minimal implementation after inspect |
| [**prove**](prove.md) | Yes (fix loop) | Run checks; prove working or fix until proven |
| [**critique**](critique.md) | No | Senior review with severities |
| [**e2e**](e2e.md) | No* | Full path: UI → API → backend → data → tests |
| [**brief**](brief.md) | No | Shortest useful answer |
| [**ooda**](ooda.md) | Per cycle | Observe → Orient → Decide → Act loops |

\* **e2e** patches only if you also ask to implement.

## Typical chains

1. **Feature or bug (unknown scope):** scout → patch → prove → critique (optional)
2. **Cross-layer bug:** e2e → ooda or patch
3. **Quick decision:** brief or [wwyd](wwyd.md)

## Shared safety (all workflow prompts)

1. **Inspect before editing** — read relevant code and docs first.
2. **Prefer small, reversible changes** — minimal diff; one concern per change.
3. **No unrelated refactors** — do not clean up adjacent code unless required.
4. **No invented APIs** — paths, commands, and architecture must exist in the repo or docs.
5. **Follow project conventions** — contributing guides, agent instruction files, package READMEs.
6. **Preserve tests** unless wrong; document why before changing expectations.
7. **Narrow validation first** — run the smallest check set for what you touched.
8. **No deploy** unless the user explicitly asks.
9. **No secrets** — never commit, paste, or log credentials.
10. **Consolidate** — use a narrower project-specific skill when one exists.

## Related prompts in this repo

| Prompt | Relationship |
|--------|----------------|
| [another-pass](another-pass.md) | Second pass on the *same* completed task |
| [prove](prove.md) | Evidence loop after change; another-pass is broader quality, not proof-first |
| [wwyd](wwyd.md) | Pick one recommendation (+ optional `--do`) |
| [plan-review](plan-review.md) | Iterate a plan to READY_FOR_BUILD |
| [ship-chat](ship-chat.md) | Commit, PR, CI, merge |
