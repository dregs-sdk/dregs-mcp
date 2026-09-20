# Security

This repository holds documentation, a Claude Code plugin, and directory metadata. It contains no server code. The
Dregs MCP server runs at `https://dregs.com/mcp` as part of the Dregs service.

## Reporting a vulnerability

Please report security issues in the Dregs MCP server, its OAuth flow, or anything else in the Dregs service to
[security@dregs.com](mailto:security@dregs.com). Dregs's [security page](https://dregs.com/legal/security/) describes
how reports are handled. Please don't open a public GitHub issue for security reports.

Issues with the contents of this repository (for example a setup snippet that would leak a token, or a skill that
encourages an unsafe practice) can go to the same address.

## Tokens and connections

- MCP tokens act as the user who created them, within the team where they were created. Treat them like passwords,
  use one per agent or machine, and revoke unused tokens under **Settings → AI Agents** in the Dregs dashboard.
- OAuth connections can be reviewed and revoked on the same page.
- Never commit a token to a repository. The snippets here use placeholders, environment variables, or editor-managed
  secret inputs for that reason.

## What an agent sees

An agent connected to Dregs reads the same identity attributes, event data, and device details your team sees in the
dashboard. That data is written by the users being scored and can contain anything, including text aimed at the agent.
The server's instructions tell agents to treat it as untrusted input, and the skills in this repository repeat that,
but a client that ignores instructions is outside Dregs's control. Review what an agent proposes before it writes.
