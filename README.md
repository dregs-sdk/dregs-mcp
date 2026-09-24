# Dregs MCP

[Dregs](https://dregs.com) scores a site's users for fraud and abuse: fake signups, bots, duplicate accounts,
referral fraud, and account takeover, each explained by the observations behind the score. Its built-in
[Model Context Protocol](https://modelcontextprotocol.io/) (MCP) server lets AI agents like Claude, ChatGPT, Codex,
Cursor, and VS Code investigate identities, review escalations, and tune rules in your Dregs account.

This repository is the public home for connecting to that server: setup snippets for each client, the Claude Code
plugin, and the metadata behind Dregs's listings in MCP directories. The server itself is hosted by Dregs at
`https://dregs.com/mcp`, and its source is not published here.

The maintained, full-length guide is the [AI Agents chapter of the Dregs manual](https://dregs.com/manual/ai-agents/).
This README is the short version.

## The Dregs MCP endpoint

| | |
| --- | --- |
| URL | `https://dregs.com/mcp` |
| Transport | Streamable HTTP |
| Authentication | OAuth 2.1 (preferred), or an MCP token in an `Authorization: Bearer` header |
| Registry name | `com.dregs/dregs-mcp` |

**OAuth** is the easiest way to connect. Clients that support MCP OAuth send you to Dregs to approve the connection in
your browser. If you belong to more than one team, you choose the team on the approval page, and you can review or
revoke connected agents under **Settings → AI Agents** in the Dregs dashboard.

**MCP tokens** are for clients that can't do OAuth, and for scripted agents. Create one in the Dregs dashboard under
**Settings → AI Agents**, copy it right away (it's only shown once), and send it as `Authorization: Bearer YOUR_TOKEN`.
A token acts as you within the team where you created it, so treat it like a password.

## Connecting Claude Code to Dregs

The quickest way is the plugin from this repository, which adds the Dregs MCP server plus skills for integrating Dregs
into your application and for working with the data afterwards:

```
/plugin marketplace add dregs-sdk/dregs-mcp
/plugin install dregs@dregs
```

Then run `/mcp`, pick **dregs**, and approve the OAuth connection in your browser. Ask Claude something like
_"Summarize my Dregs account"_ to confirm it can reach your team.

Without the plugin, add the server directly and Claude Code will walk you through OAuth:

```bash
claude mcp add --transport http dregs https://dregs.com/mcp
```

Or with an MCP token, for headless use:

```bash
claude mcp add --transport http dregs https://dregs.com/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

The equivalent project-level `.mcp.json`:

```json
{
  "mcpServers": {
    "dregs": {
      "type": "http",
      "url": "https://dregs.com/mcp"
    }
  }
}
```

### Plugin skills

The plugin's skills are workflows, not tool schemas; each reads the relevant manual chapter through
`get_documentation` before it acts, proposes changes for you to review, and confirms before anything is written.

| Skill | What it does |
| --- | --- |
| `connect` | Connects Claude Code to the Dregs MCP server over OAuth or a token and verifies it with `get_account_summary`. |
| `setup` | Drives a complete integration of Dregs into your application, checking what exists and asking which parts to do. |
| `setup-tracking` | Adds the `dregs.js` snippet, `dregs.identify()` at signup and login, and `dregs.track()` for key actions, then verifies events arrive. |
| `setup-events` | Sends server-side events with the official Dregs SDK for your language and maps your event names and identity fields to Dregs's canonical types. |
| `setup-webhooks` | Builds a webhook receiver with signature verification and a first response to scores and escalations, then verifies deliveries. |
| `health-check` | Read-only diagnosis of an integration: ingestion, identification, mappings, analyzer observations, limits. |
| `investigate-identity` | Walks one identity from scores to observations, history, links, devices, and events, and reaches a verdict. |
| `investigate-cluster` | Maps a ring of related accounts across shared devices, IPs, and sessions, with the evidence for each member. |
| `tune-rules` | Authors or adjusts badge and escalation rules with a preview of their impact and confirmation before writing. |
| `weekly-review` | A read-only account health review: volume, score distribution, rule activity, open escalations, usage. |

The integration skills edit your code, so they need a client with repository access such as Claude Code or Codex. The
others work anywhere the server is connected.

## Connecting Claude Desktop and claude.ai to Dregs

Claude Desktop and claude.ai connect to remote MCP servers as custom connectors, which use OAuth.

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Enter a name and the URL `https://dregs.com/mcp`.
3. Choose **Connect** and approve the Dregs OAuth connection when your browser opens.
4. In a conversation, open the tools menu and turn on the Dregs connector.

## Connecting ChatGPT and Codex to Dregs

ChatGPT on the web connects through plugins created in **Developer mode** (**Settings → Security and login**). Open
**Plugins**, add a connection with the URL `https://dregs.com/mcp`, then add it to a new conversation from the tools
menu and approve the OAuth connection.

Codex in the ChatGPT desktop app, the Codex CLI, and the Codex IDE extension share one configuration:

```bash
codex mcp add dregs --url https://dregs.com/mcp
codex mcp login dregs
```

Or with an MCP token in `~/.codex/config.toml`:

```toml
[mcp_servers.dregs]
url = "https://dregs.com/mcp"
bearer_token_env_var = "DREGS_MCP_TOKEN"
```

## Connecting Cursor to Dregs

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=dregs&config=eyJ1cmwiOiJodHRwczovL2RyZWdzLmNvbS9tY3AifQ==)

Or add it to `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json` for all projects) and approve the OAuth
connection when Cursor first connects:

```json
{
  "mcpServers": {
    "dregs": {
      "url": "https://dregs.com/mcp"
    }
  }
}
```

To use an MCP token instead, add `"headers": { "Authorization": "Bearer ${env:DREGS_MCP_TOKEN}" }` and set
`DREGS_MCP_TOKEN` in the environment before starting Cursor.

## Connecting VS Code to Dregs

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Dregs_MCP-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=dregs&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fdregs.com%2Fmcp%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_Dregs_MCP-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=dregs&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fdregs.com%2Fmcp%22%7D)

Or run **MCP: Add Server** from the Command Palette, or add it to `.vscode/mcp.json`:

```json
{
  "servers": {
    "dregs": {
      "type": "http",
      "url": "https://dregs.com/mcp"
    }
  }
}
```

To use an MCP token, declare a `promptString` input with `"password": true` and reference it in an `Authorization`
header. The [manual](https://dregs.com/manual/ai-agents/) has the complete example.

## Connecting other MCP clients to Dregs

Any client that supports remote servers over Streamable HTTP can connect the same way. For clients that only support
local (stdio) servers, the [mcp-remote](https://www.npmjs.com/package/mcp-remote) bridge usually works. Leave out the
`--header` arguments to have it run the OAuth flow instead of using a token:

```json
{
  "mcpServers": {
    "dregs": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://dregs.com/mcp", "--header", "Authorization: Bearer YOUR_TOKEN"]
    }
  }
}
```

## What an agent can do with Dregs

The server exposes the investigative and rule-tuning parts of the Dregs dashboard. Clients fetch the live tool list
when they connect, so this table is a map of the surface rather than the source of truth.

| Group | Tools |
| --- | --- |
| Account and dashboard | `get_account_summary`, `dashboard_stats`, `dashboard_score_distribution`, `dashboard_rule_activity`, `get_documentation` |
| Identities and events | `list_identities`, `get_identity`, `get_identity_analysis`, `get_identity_history`, `get_identity_links`, `search_events`, `analyze_identity`, `set_identity_disregarded` |
| Devices | `list_devices`, `get_device` |
| Escalations | `list_escalations`, `get_escalation`, `escalation_summary`, `update_escalation_status` |
| Escalation rules | `list_escalation_rules`, `get_escalation_rule`, `preview_escalation_rule`, `create_escalation_rule`, `update_escalation_rule`, `delete_escalation_rule` |
| Badge rules | `list_badge_rules`, `get_badge_rule`, `create_badge_rule`, `update_badge_rule`, `delete_badge_rule` |
| Notification channels | `list_channels`, `get_channel`, `list_channel_deliveries`, `test_channel` |
| Datasets and mappings | `list_datasets`, `list_dataset_entries`, `add_dataset_entry`, `remove_dataset_entry`, `list_mappings`, `set_mapping`, `delete_mapping` |

Reads are available to every team member. Writes follow your dashboard role: creating, changing, or deleting rules,
disregarding an identity, re-scoring, testing a channel, and editing datasets or mappings need the admin role. Some
things are left to humans on purpose. An agent cannot create or edit notification channels or their secrets, manage
your team, credentials, or billing, delete identities or events, or change how Dregs scores. It investigates, proposes,
and hands off.

Every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`), so clients
that honor them can prompt before writes and deletes.

## Server SDKs

Getting data into Dregs is a separate job from connecting an agent to it, and there is an official SDK for each of five
server languages. They cover sending events, reading an identity with its scores or its full analysis, requesting a
re-score, and verifying webhook signatures, with typed errors per status and retries with backoff built in. Reach for
one before writing an HTTP client by hand; the plugin's `setup-events` skill does.

| Language | Package | Install | Minimum | Source |
| --- | --- | --- | --- | --- |
| Python | `dregs` on PyPI | `pip install dregs` or `uv add dregs` | Python 3.10 | [dregs-sdk-python](https://github.com/dregs-sdk/dregs-sdk-python) |
| TypeScript | `@dregs/sdk` on npm | `npm install @dregs/sdk` | Node 20 | [dregs-sdk-typescript](https://github.com/dregs-sdk/dregs-sdk-typescript) |
| Java | `com.dregs:dregs-sdk` on Maven Central | add the Gradle or Maven coordinate | Java 17 | [dregs-sdk-java](https://github.com/dregs-sdk/dregs-sdk-java) |
| Ruby | `dregs` on RubyGems | `bundle add dregs` | Ruby 3.1 | [dregs-sdk-ruby](https://github.com/dregs-sdk/dregs-sdk-ruby) |
| PHP | `dregs/dregs-sdk` on Packagist | `composer require dregs/dregs-sdk` | PHP 8.2 | [dregs-sdk-php](https://github.com/dregs-sdk/dregs-sdk-php) |

The SDKs are server-side only and authenticate with a credential's secret key (`sk_…`) as a bearer token. Browser
tracking remains the `dregs.js` script with the public key (`pk_…`). On npm the bare `dregs` package is that browser
script, not the SDK; the server SDK is `@dregs/sdk`.

## Safety practices for AI fraud review

- **The data is written by the people being scored.** Identity attributes, display names, and event payloads are
  user input, and a bad actor can put text in them aimed at an agent. The server tells agents to treat everything it
  returns as data, never as instructions. Keep that in mind when you read an agent's conclusions too.
- **The agent acts in your account.** Rule changes and disregarded identities affect the whole team's scoring and
  escalations. Ask for a preview (`preview_escalation_rule` reports how many identities a rule would open against
  today) and review proposed changes before approving them.
- **Scope your access.** Use one token per agent or machine, name them clearly, and revoke any you no longer use.
  Review OAuth connections under **Settings → AI Agents**.

## Support

- Setup help and questions: [support@dregs.com](mailto:support@dregs.com)
- Security issues: see [SECURITY.md](SECURITY.md)
- Problems with the contents of this repository (the README, plugin, or directory metadata): open an issue here

## What's in this repository

| Path | Purpose |
| --- | --- |
| `server.json` | Dregs's entry in the [official MCP Registry](https://registry.modelcontextprotocol.io/) |
| `glama.json` | Maintainer metadata for the [Glama](https://glama.ai/mcp/servers) listing |
| `chatgpt-app-submission.json` | Tool annotations, justifications, and test cases for the ChatGPT plugin directory submission |
| `.claude-plugin/`, `.mcp.json`, `skills/` | The Claude Code plugin and its marketplace manifest |
| `.github/workflows/` | Validation on every push, and registry publishing on release |

## Releasing a new version of the surface

The tool surface is a public API once published. When it changes: bump `version` in `server.json`,
`.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` together, update the tool table above and the
skills if tools were added or renamed, tag the release `vX.Y.Z`, and publish a GitHub release. The
`publish-registry` workflow pushes `server.json` to the MCP Registry from the release.

## License

The contents of this repository are released under the [MIT License](LICENSE). The Dregs service itself is governed
by [Dregs's terms of service](https://dregs.com/legal/terms/).
