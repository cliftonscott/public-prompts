# Optional setup guides

These docs wire **[plan-review](../prompts/general/plan-review.md)** to external reviewers. They are **optional** — the default plan-review path works in a single agent session with no extra tooling.

| Guide | Use when |
|-------|----------|
| [setup-codex-cli.md](setup-codex-cli.md) | You want a **read-only** [Codex CLI](https://github.com/openai/codex) pass as the reviewer (`--codex`) |
| [setup-mcp-second-opinion.md](setup-mcp-second-opinion.md) | You expose a **`review_plan`** MCP tool in Cursor (`--mcp`) |

**Security:** Never commit API keys, tokens, or `.env` files. Configure credentials only through your OS environment or IDE MCP settings.
