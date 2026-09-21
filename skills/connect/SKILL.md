---
name: connect
description: Connect Claude Code to Dregs's MCP server and verify the connection. Use when Dregs tools are missing, a Dregs tool returns 401 or unauthorized, or the user asks to connect or reconnect Claude to Dregs. For integrating Dregs into the user's own application, use the setup skill instead.
---

# Setting up the Dregs MCP connection

The Dregs plugin registers one MCP server, `dregs`, at `https://dregs.com/mcp`. It authenticates with OAuth, so the
user has to approve the connection once in their browser. You cannot complete that approval for them, but you can get
them there and confirm the result.

## Check whether Dregs is connected

Call `get_account_summary`. Three outcomes:

- **It returns an account.** The connection works. Tell the user which team it acts in and the roles it holds there,
  because everything the agent does is scoped to that team, and writes need the admin role.
- **The tool is missing.** The plugin's MCP server hasn't started or isn't authenticated. Follow the steps below.
- **It fails with 401 or "unauthorized".** The OAuth connection was revoked, the access token was revoked from the
  dashboard, or the user is using a token that was revoked or mistyped. Follow the steps below.

## Connect with OAuth

Ask the user to:

1. Run `/mcp` in Claude Code.
2. Select **dregs** from the list and choose **Authenticate**.
3. Sign in to the Dregs dashboard if asked, then approve the connection. If they belong to more than one Dregs team,
   the team they pick on the approval page is the one the agent will work in.
4. Come back and confirm.

Then call `get_account_summary` again to verify.

## Connect with an MCP token instead

Only for headless use or clients where OAuth isn't possible. Tell the user to create a token in the Dregs dashboard
under **Settings → AI Agents**, copy it immediately (it's shown once), and register the server outside the plugin:

```bash
claude mcp add --transport http dregs https://dregs.com/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

Never ask the user to paste a token into the conversation, and never write a token into a file that will be
committed.

## Troubleshooting

- **Stale or missing tools after a change:** ask the user to run `/mcp` and reconnect, or restart Claude Code.
- **Wrong team:** the user has to disconnect and reconnect, choosing the right team on the approval page.
- **A write is refused with a role message:** the connected user is not an admin on that team. Reads still work;
  hand the proposed change to an admin instead of retrying.
- **Still stuck:** point them at https://dregs.com/manual/ai-agents/ or support@dregs.com.
