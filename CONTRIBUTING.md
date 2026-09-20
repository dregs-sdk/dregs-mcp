# Contributing

Thanks for helping keep the Dregs MCP setup guides accurate. Client apps change their MCP configuration often, so the
most useful contributions are corrections to snippets that no longer work.

- The [AI Agents chapter of the Dregs manual](https://dregs.com/manual/ai-agents/) is the source of truth for client
  setup. If a snippet here disagrees with the manual, the manual wins; please mention the discrepancy in your pull
  request so both get fixed.
- Keep snippets minimal and copy-pasteable, with `YOUR_TOKEN` as the placeholder for MCP tokens.
- Skills in `skills/` describe workflows, not tool argument schemas. Clients read the live tool schemas from the
  server, so don't duplicate them here.
- Run `claude plugin validate . --strict` before opening a pull request that touches the plugin.

Questions about the server itself, feature requests, or account problems go to
[support@dregs.com](mailto:support@dregs.com).
