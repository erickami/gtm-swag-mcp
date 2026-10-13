# GTM Swag: personalized gifts that get meetings, for AI agents

GTM Swag helps sales, marketing and customer success teams in the US send personal physical gifts to prospects and customers: for a new role, a funding round, an event follow-up, a quiet deal or a thank-you. This repository holds the GTM Swag MCP server listing, the Claude plugin and two agent skills.

**Agents prepare; people approve.** An agent can find a gift, prepare a campaign or an always-on playbook, add people as signals arrive, and read gift links and delivery status. Only a signed-in person on gtmswag.com approves, pays, adds funds or turns on auto-approve. No platform fee: you pay per gift.

## The MCP server

| | |
|---|---|
| URL | `https://gtmswag.com/mcp` (Streamable HTTP) |
| Auth | OAuth 2.1 sign-in with dynamic client registration and PKCE. Or a workspace key as `Authorization: Bearer ...`, kept in an environment variable, never in a chat |
| Registry | `com.gtmswag/gtm-swag` on the official MCP Registry. See `server.json` |
| Docs | https://gtmswag.com/guides/connect and https://gtmswag.com/agent-guide.md |
| OpenAPI | https://gtmswag.com/openapi.json |

A sign-in connection gets 31 gifting tools: search and compare gifts, create campaign and playbook drafts, add recipients from a CRM or a signal, read the approval queue and balance, fetch private gift links and CSV exports (a separate permission you grant), and track delivery. Tools that pay, approve, send, add funds or change team access are never offered to an agent.

### Connect

- **Claude Code:** `claude mcp add --transport http gtm-swag https://gtmswag.com/mcp`, then run `/mcp` and sign in.
- **Claude apps:** install this plugin, or add GTM Swag from the Connectors directory.
- **Any other MCP client** (Codex, Cursor, Gemini CLI, VS Code and others): add a Streamable HTTP server at `https://gtmswag.com/mcp` and sign in when prompted:

  ```json
  { "mcpServers": { "gtm-swag": { "type": "http", "url": "https://gtmswag.com/mcp" } } }
  ```

You need a GTM Swag account: create one free at [gtmswag.com/login](https://gtmswag.com/login?utm_source=github&utm_medium=readme&utm_campaign=gtm-swag-mcp). Every new workspace starts in test mode, where everything works end to end but nothing ships and no real money moves, until GTM Swag enables delivery for it. Delivery is to US addresses.

## The Claude plugin

Install it in Claude Code:

```
/plugin marketplace add erickami/gtm-swag-mcp
/plugin install gtm-swag@gtm-swag
```

It connects the server above and adds two skills.

## The skills

| Skill | What it does |
|---|---|
| `gifting` | Finds gifts for a moment and a budget, sets up always-on playbooks and one-time campaign drafts, retrieves private gift links and CSVs, and reports delivery status |
| `queue-gifts-from-signals` | Turns sales signals (a champion's job change, a new VP at a target account, a funding round, event attendees, a stalled deal, a renewal, or a target-account list) into gifts queued for your team to approve. Matches each person to one moment, picks a gift within budget, writes a short note, dedupes on a stable row ID, shows a dry run, and adds people only after one explicit approval |

Install the skills on their own in any agent that reads Agent Skills:

```
npx skills add erickami/gtm-swag-mcp
```

## Example prompts

- "Find three gifts under $75 for a prospect whose company just launched. Show what I'd need to provide."
- "Here's my champion job-change list from Clay. Queue a new-role gift for everyone who moved in the last 30 days, budget $60."
- "Create a draft campaign for these five event contacts with a $60 per-person budget and a short thank-you note."
- "Which gifts in my latest campaign are still waiting for an address?"
- "Export the gift links for my 'Q4 launch' campaign as a CSV."

## What it never does

It never pays, approves spending, adds funds, ships or sends a gift, emails a recipient, turns on or changes auto-approve, or changes team access. It never looks up a home address: when you don't have one, the recipient confirms their own US address through a private gift link that expires 14 days after the gift is created. GTM Swag doesn't track replies or meetings.

## Privacy and support

- Privacy policy: https://gtmswag.com/privacy
- Terms of service: https://gtmswag.com/terms
- Support: https://gtmswag.com/support
- Disconnect an agent from your workspace at any time: https://gtmswag.com/oauth/disconnect

GTM Swag is powered by Better Swag.

## License

MIT. See [LICENSE](LICENSE).
