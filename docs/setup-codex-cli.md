# Optional: Codex CLI as plan reviewer

Use this with **[plan-review](../prompts/general/plan-review.md)** when the orchestrator runs with **`--codex`**. Codex reviews the plan **read-only**; your primary agent applies patches to the working copy.

Official references:

- [Codex CLI (GitHub)](https://github.com/openai/codex)
- [Codex CLI docs](https://developers.openai.com/codex/cli)
- [Authentication](https://developers.openai.com/codex/auth) (ChatGPT sign-in or API key — configure locally, never in git)

## What you need

1. **`codex` on PATH** — install via npm, Homebrew, or the official install script (see the GitHub README).
2. **Authentication** — `codex login` / sign in with ChatGPT, or API key via environment variables per OpenAI’s auth docs. **Do not** paste keys into prompts or commit them to this repo.
3. **A reviewer prompt file** — one markdown file per review session that includes:
   - Absolute path to the **working plan** copy
   - The **Reviewer output contract** from [plan-review.md](../prompts/general/plan-review.md#reviewer-output-contract)
   - Paths to public project docs (`CONTRIBUTING.md`, `README.md`, etc.) — not secrets

## Minimal workflow

From your repo root (paths are examples — adjust to your tree):

```bash
# 1. Working copy (orchestrator usually does this)
cp docs/plans/my-feature.plan.md /tmp/plan-review/my-feature.working.plan.md

# 2. Write /tmp/plan-review/codex-review-prompt.md with:
#    - "Read only. Do not edit files."
#    - WORKING_PLAN absolute path
#    - Full reviewer output contract (VERDICT trailer + optional patch)

# 3. Read-only review (verify exact flags against `codex exec --help` on your install)
codex exec -s read-only -C "$(pwd)" "$(cat /tmp/plan-review/codex-review-prompt.md)"
```

Use **read-only** sandbox (`-s read-only` or your CLI version’s equivalent). Do not use full-access modes for plan review.

## Orchestrator responsibilities

The agent running plan-review should:

1. Confirm `codex` is available (`command -v codex`).
2. Write a **session-specific** prompt file (do not reuse a shared path across parallel reviews).
3. Run Codex read-only and capture stdout or the CLI’s report output.
4. Parse **`VERDICT:`** only from output that clearly belongs to **this** plan path and iteration.
5. Apply `## Plan patch (unified diff)` with `patch --batch --forward -p0` on the working plan, or follow `## Reviewer instructions`.
6. On timeout or ambiguous output, retry once or fall back to same-session review (no `--codex`).

## Optional: relay wrapper

Some teams wrap `codex exec` in a script that:

- Dry-runs once before the first live call
- Stores run artifacts (prompt copy, exit code, report) under a gitignored directory
- Enforces timeouts

That wrapper is **project-specific**. This repo ships prompts only; copy the pattern into your private repo if you need manifest correlation and operator guardrails.

## Safety checklist

- Read-only sandbox for review passes
- No secrets in prompt files (redact URLs with tokens, credentials, private hostnames you do not intend to publish)
- Reviewer does not edit the repo; only the orchestrator updates the plan working copy
- Do not commit `/tmp` prompt files or Codex run logs

## Troubleshooting

| Issue | What to try |
|-------|-------------|
| `codex: command not found` | Reinstall CLI; open a new shell |
| Auth errors | `codex login` or fix API key env per official auth docs |
| No `VERDICT:` line | Treat as failed round; do not apply patches |
| Patch does not apply | Use reviewer instructions for minimal manual edits |
