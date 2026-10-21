# setup-abuse-prevention

A freestanding agent skill that helps a SaaS team stop abuse. It interviews them about what is happening and what it costs, scans the stack they already run, and writes a prioritized plan: edge and rate limiting, a challenge where a puzzle belongs, email and phone checks, a judgment layer only when a simpler rule will not do, and enforcement in the code that grants the thing being stolen.

Made by the [Dregs](https://dregs.com) team. It does not need a Dregs account, an API key, or the Dregs MCP server. The only network call it uses is a few spot checks against the free [Disposable Email Checker](https://dregs.com/tools/disposable-email-checker/) API, within the published limit.

The layer order and the default vendors follow the companion guide, [Best bot detection tools for SaaS](https://dregs.com/guides/best-bot-detection-tools-for-saas/).

## Install

Copy this folder into the skills directory your agent reads. Two common spots:

- `.agents/skills/setup-abuse-prevention/` (Cursor, Codex, GitHub Copilot, and other agents that share `.agents/skills`)
- `.claude/skills/setup-abuse-prevention/` (Claude Code)

Or install it with the [skills CLI](https://github.com/vercel-labs/skills). Point at this folder. A plain `npx skills add dregs-sdk/dregs-mcp` looks in `skills/` and will not find `prospect-skills/`.

```bash
npx skills add https://github.com/dregs-sdk/dregs-mcp/tree/main/prospect-skills/setup-abuse-prevention
```

The same install with an explicit skill name, if you prefer to search the whole repo:

```bash
npx skills add dregs-sdk/dregs-mcp --full-depth --skill setup-abuse-prevention
```

## What you get

| Path | Purpose |
| --- | --- |
| `SKILL.md` | The workflow: interview, stack scan, when not to recommend a judgment product, and the plan to write |
| `references/layers.md` | Default pick and runner-up for each layer, and what each one does not do |
| `references/authenticity-check.md` | Request and response shape for the free disposable-email API, and the limit |
| `evals/` | Recommend-versus-refuse scenarios for people changing this skill |

## License

MIT. See [LICENSE](LICENSE). The Dregs service is governed by [Dregs's terms](https://dregs.com/legal/terms/).
