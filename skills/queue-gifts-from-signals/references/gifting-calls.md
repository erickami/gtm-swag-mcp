# Gifting calls

The GTM Swag calls this skill makes, with their REST equivalents for agents that use the HTTP API instead of MCP. Prices are US cents. Every success is `{"data": ...}`; every error is `{"error", "code", "nextAction", "url", "retryable", "retryAfterMs"}`. Check `isError` on MCP results even when the transport succeeded.

## Connecting

Sign-in is the recommended connection: there is no key to store, and access can be ended at any time from the workspace.

- **Claude Code:** `claude mcp add --transport http gtm-swag https://gtmswag.com/mcp`, then run `/mcp` and sign in.
- **Any other MCP client (Codex, Cursor and others):** add a Streamable HTTP server named `gtm-swag` at `https://gtmswag.com/mcp` in the client's MCP settings and sign in when prompted. The JSON form most clients accept:

  ```json
  { "mcpServers": { "gtm-swag": { "type": "http", "url": "https://gtmswag.com/mcp" } } }
  ```

- **A workspace key instead of sign-in:** the installer creates one in the workspace under **Integrations, For developers**, keeps it in an environment variable named `GTM_SWAG_KEY` set from their password manager or shell, and points the client's `Authorization` header at that variable (in Claude Code's `.mcp.json`: `"headers": { "Authorization": "Bearer ${GTM_SWAG_KEY}" }`). Use `https://gtmswag.com/mcp?profile=gifting` so a key sees the same gifting tools a sign-in connection does. **Never ask anyone to paste a key into the conversation, and never print one.**
- **No account yet:** `https://gtmswag.com/login?utm_source=clay-skills&utm_medium=skill&utm_campaign=queue-gifts-from-signals`. Creating one is free; every new workspace starts in test mode.

No connection can approve, pay, add funds, change auto-approve, send gifts or change team access. A person does those on gtmswag.com.

## The calls, by step

| Step | MCP tool | REST | What to read in the result |
|---|---|---|---|
| 0 | `get_workflow_guide` | `GET /api/v1/workflow` | `mode`: `test` or `live` |
| 3 | `list_campaigns` with `mode: "always_on"` | `GET /api/v1/campaigns?mode=always_on` | each playbook's `id`, `productId`, `message`, `status` and `autoApprove` |
| 3 | `get_campaign` with `id` | `GET /api/v1/campaigns/CAMPAIGN_ID` | `autoApprove: {enabled, perGiftMaxCents}` and `monthlyBudgetCents` |
| 3 | `search_gifts` with `useCase`, `budgetCents`, `recipientCount`, `excludeAlcohol` | `GET /api/v1/catalog` (public; filter yourself) | per gift `id`, `estimatedTotalCents`, `shippingQuoteRequired`, `verified` |
| 3 | `get_gift` with `productId` | `GET /api/v1/catalog` | personalization, photo, artwork and age requirements, lead time |
| 3 | `get_balance` | `GET /api/v1/balance` | `cents`, `mode`, `lowBalance`, `topUpUrl` |
| 5 | `create_campaign` | `POST /api/v1/campaigns` | the new playbook's `id`; share `https://gtmswag.com/campaigns/CAMPAIGN_ID?review=1` |
| 5 | `add_to_playbook` with `id` and `recipient` | `POST /api/v1/playbooks/CAMPAIGN_ID/people` with `{"recipient": {...}}` | `queued`, `alreadyQueued`, `recipientId`, `approvalUrl`, `approval: {state, reason, sendId}` |
| 7 | `list_queue` with `campaignId` | `GET /api/v1/queue?campaignId=CAMPAIGN_ID` | each waiting person's `approvalUrl`, `blockers` and `autoApprove.reason` |

### `search_gifts`

`useCase` is one of `cold_outreach`, `company_milestone`, `new_role`, `event_follow_up`, `deal_reengagement`, `customer_thanks`, `team_recognition` or `general`. `budgetCents` is the per-person ceiling. Results leave out retired and unavailable gifts. A gift with `shippingQuoteRequired: true` has unpriced shipping: never present its subtotal as a delivered total, and expect it to wait for a person even with auto-approve on. With `mode: "live"`, only a gift with `verified: true` can be ordered.

### `create_campaign` for a new playbook

```json
{
  "mode": "always_on",
  "status": "draft",
  "name": "New role gifts",
  "productId": "PRODUCT_ID_FROM_SEARCH",
  "senderName": "Jordan Lee",
  "senderCompany": "Example Co",
  "message": "Congrats on the new role, {{firstName}}. A small something for your first month.",
  "idempotencyKey": "new-role-gifts-2026-10"
}
```

Leave out `recipients`. Resending the same body with the same `idempotencyKey` returns the same draft. A teammate turns it on at the review link; until then every add fails with `PLAYBOOK_OFF`.

### `add_to_playbook` recipient

```json
{
  "name": "Dana Whitfield",
  "company": "Northwind",
  "title": "VP Sales",
  "profileUrl": "https://www.linkedin.com/in/dana-whitfield-example",
  "sourceSystem": "clay",
  "externalId": "ROW_ID_FROM_THE_SOURCE",
  "addressMode": "confirm"
}
```

- Needs a `name`, a personal `profileUrl`, or `sourceSystem` plus `externalId`. Always send `sourceSystem` and `externalId`: a repeat then returns `alreadyQueued: true` and never approves or charges twice.
- `profileUrl` must be `https://www.linkedin.com/in/SLUG` (a two-letter country subdomain is accepted). Sales Navigator, company and `/pub/` links return `400 VALIDATION_ERROR`; leave the field out instead.
- `message` (up to 500 characters) overrides the playbook's note for this person. A note with `{{firstName}}`, `{{name}}` or `{{company}}` needs that field on the person, or the add fails with `400 RECIPIENT_MESSAGE_REQUIRED` and nothing is saved.
- With a full US address: `"addressMode": "known"` plus `address`, `city`, `state` (`TX` or `Texas`) and `zip`. Country is US only.
- Unknown fields are refused with `400 VALIDATION_ERROR`. Send only these.

## Approval states

| `approval.state` | Meaning | This skill's verdict |
|---|---|---|
| `waiting` | Waits for an owner or admin; `reason` says why | `queued` |
| `scheduled` | Auto-approve is finishing it in the background, usually within minutes | `queued`, reason "being approved by the team's rule" |
| `auto_approved` | The playbook's rule approved it and charged the balance; `sendId` names the gift | `auto_approved` |
| `approved` | A person had already approved this person's gift | `already_queued` |
| `skipped` | Someone on the team skipped this person | `skipped_by_team` |

A result with `alreadyQueued: true` is `already_queued` whatever its state.

## Waiting reasons, in plain words

| `reason` | Say |
|---|---|
| `auto_approve_off` | waits for a person to approve |
| `over_gift_limit` | costs more than the per-gift limit the team set |
| `monthly_cap` | this month's playbook budget is used up |
| `low_balance` | the workspace balance can't cover it; an owner or admin adds funds at `topUpUrl` |
| `repeat_recipient` | this person already received a gift from another campaign |
| `photo_review` | a recipient photo needs review |
| `age_restricted` | age-restricted gifts always wait for a person |
| `shipping_quote` | delivery isn't priced yet, so it always waits for a person |
| `email_unverified` | the person who turned on auto-approve hasn't verified their email |
| `blocked` | something else; read the queue item's `blockers` |

Adding funds or raising a limit does not re-check people already waiting. With auto-approve on, adding the same person again (same `sourceSystem` and `externalId`) runs the rule again and never creates a second gift.

## Errors

| Code | What to do |
|---|---|
| `409 PLAYBOOK_OFF` | The playbook is a draft or paused. A teammate turns it on at the returned `url`; then resend |
| `409 PLAYBOOK_BUSY` | Other people were being added at that moment; nothing was saved. Wait `retryAfterMs` (under a second) and resend the same call |
| `429 RATE_LIMIT` | 240 requests a minute per workspace. Wait `retryAfterMs` and resend |
| `400 RECIPIENT_MESSAGE_REQUIRED` | Give this person their own `message`, then resend |
| `400 VALIDATION_ERROR` | Read the field named in the error; drop a bad `profileUrl` rather than send it |
| `401 AUTH_REQUIRED` | The connection expired or the key is wrong; the installer reconnects. Never switch accounts |
| `403 HUMAN_APPROVAL_REQUIRED` | A signed-in person must do this on gtmswag.com; hand them the `url` |

After a timeout, `add_to_playbook` can be resent as is, and `create_campaign` with the same `idempotencyKey` and body. Re-read state before retrying anything else.

## Gift links

With `addressMode: "confirm"`, a private gift link exists once the gift is approved. It belongs to one recipient and expires 14 days after the gift is created, opened or not. The installer sends it in their own outreach; the gifting workspace never emails the recipient. This skill does not fetch, share or send links.
