---
name: gifting
description: Use when the user wants personalized physical B2B gifts that help get meetings: gift ideas, an always-on GTM Swag gifting playbook, a one-time campaign draft, recipients added to a campaign, a private gift link, a CSV of gift links, or the status of gifts already sent. Covers prospect outreach, account-based marketing, launches, new roles, events, quiet deals and customer thank-yous. Not for general prospect research, email-only outreach or CRM enrichment.
---

# GTM Swag gifting

GTM Swag helps go-to-market marketers and sellers in the US get meetings by sending personalized, thoughtful physical gifts. Good moments for a gift are a new role, an event or webinar follow-up, a funding round, a product launch, a closed deal or a renewal. A gift is a reason to start a conversation, not a guarantee: never promise a reply or a meeting.

With the GTM Swag connector you can search the catalog, set up playbooks that gift each person a trigger brings in, prepare one-time campaign drafts, and retrieve gift links and status. A teammate reviews every campaign on gtmswag.com, and an owner or admin approves and pays; the connector cannot pay, approve spending or send gifts. For a playbook, an owner or admin can instead turn on auto-approve on gtmswag.com: gifts within a per-gift limit and monthly budget they set are then approved and paid from the workspace balance without a review. The connector can read that rule but never turn it on or change it.

If the GTM Swag tools are not available, ask the user to connect GTM Swag from this plugin's Connectors tab, then continue.

## Start with what the user asked for

- **Gift ideas:** search the catalog and offer a short, relevant shortlist.
- **Gifts whenever something happens** (a new role, a signup, a closed deal): set up or reuse a playbook. This is the default for ongoing gifting.
- **A one-time gift for a fixed list:** reuse the context already given, create one batch draft, add the people, and share the review link and anything still missing.
- **More people for an existing campaign:** read that campaign first and add only the people who are missing.
- **A link or a list:** use the existing campaign and return its current link or CSV. Do not create another campaign.
- **Status:** read the existing gifts and report their actual state.

Use the recipient, reason, budget, timing and assets the user already gave. When a missing detail changes the next step, ask one bundled question. Never invent prospects, people or lists.

At the start of a session, `get_workflow_guide` returns the workspace's mode, current limits, review URL format and retry rules. Prefer what it returns over anything here.

Tell the user what the mode means when it matters. With mode "test", which every workspace has until GTM Swag enables delivery, everything works end to end but nothing ships and no real money moves, so gift links shouldn't go to real prospects. To enable delivery, a person on the team contacts GTM Swag at https://gtmswag.com/support; the connector can't. With mode "live", gifts ship, only verified gifts can be ordered, and each gift needs a US address the sender supplies: a person without one can be added but waits in the queue with "Needs a shipping address" and can't be approved (`LIVE_ADDRESSLESS_UNAVAILABLE`). When delivery is turned on, playbooks stay on, but the balance starts again from zero: funds added before never pay for gifts that ship.

## Find a gift

`search_gifts` ranks gifts for the business goal (`useCase`) within a per-person `budgetCents`, and adds campaign estimates for `recipientCount`. `get_gift` returns the chosen gift's full requirements: personalization and lettering limits, photo, artwork and age requirements, pricing flags and lead time. `browse_gifts` pages through the whole catalog when the user wants to see everything. Prices are US cents. Choose only gifts that are not retired (check this first: a retired gift can still show `available` true) and have `available` not set to `false`, within the user's per-person budget. `search_gifts` already leaves out retired and unavailable gifts, but not unverified ones.

- Offer three distinct gifts. For each, give the fit, the cost (gift plus listed shipping; the approved total includes applicable sales tax), the personal touch and one practical tradeoff. Skip gifts priced at zero.
- `shippingQuoteRequired` means shipping is extra and not yet confirmed; never present that subtotal as a delivered total, and expect auto-approve to leave those gifts for a person. A gift with `verified` false can be planned and tried, but its price is an estimate and it can't be ordered for delivery until GTM Swag verifies it.
- Lead times are production estimates, not delivery promises. Never invent availability, costs, delivery dates or supplier details.
- Base recommendations only on facts the user supplied, such as a company milestone or a stated interest. Never guess hobbies, health, religion, family, identity or drinking habits from names, titles or photos.
- Alcohol needs the sender to confirm the recipient is 21 or older. Never assume it.

A printed gift note is separate from outreach. When asked, draft a short note (usually two sentences) using a real supplied detail. Do not invent conversations, praise or promises. A gift is never conditional on a meeting, review or referral, and a thank-you note does not ask for a meeting.

## Set up a playbook

A playbook is an always-on campaign: one gift, one note template and an optional monthly budget. Each time a trigger fires, the user's workflow adds one person, who waits in an approval queue until someone on the user's team approves the gift, unless the playbook's auto-approve covers it.

1. `list_campaigns` with `mode: "always_on"` lists existing playbooks. Reuse one that fits the goal instead of creating another.
2. Otherwise choose a gift (above) and call `create_campaign` with `mode: "always_on"`, `status: "draft"`, a `name`, the `productId`, `senderName`, `senderCompany`, the printed note in `message`, an optional `monthlyBudgetCents` and one stable `idempotencyKey`. Leave recipients empty; people arrive one per trigger.
3. Share the review link (`https://gtmswag.com/campaigns/{id}?review=1`). A signed-in teammate checks the gift and note there and selects **Turn on playbook**. An owner or admin may also turn on auto-approve there with a per-gift limit and a monthly budget. The connector cannot do either, and nothing is paid at this step. After the playbook is on, its settings change on gtmswag.com: an `update_campaign` from the connector returns it to draft, and adds then refuse with `PLAYBOOK_OFF` until a teammate turns it on again.
4. Once the playbook is on, call `add_to_playbook` once per trigger with one person and their `sourceSystem` and `externalId`. The playbook stays on and a repeat returns `alreadyQueued: true` without approving or charging twice. Read `approval.state` in each result: "auto_approved" means auto-approve approved the gift and paid it from the balance (tell the user); "waiting" comes with an `approval.reason`; "scheduled" finishes in the background within minutes. Without auto-approve, every person waits and nothing is charged or sent. A draft or paused playbook refuses with `PLAYBOOK_OFF`; share the review link again. `PLAYBOOK_BUSY` means other people were being added at that moment and nothing was saved: wait the returned `retryAfterMs` (under a second) and send the same call. If the note uses a name or company placeholder, the person needs that field.
5. `list_queue` (filter by `campaignId`) lists the people waiting, with any blockers, the auto-approve reason and each person's `approvalUrl`. Share those links: a signed-in owner or admin approves or skips each gift on gtmswag.com, and approving charges the workspace balance. Explain the reason in plain words: the gift costs more than the per-gift limit, the monthly budget is used up, the balance is too low, the person was gifted before, a photo needs review, the gift is age-restricted, its shipping isn't priced yet (these always wait for a person), the person who turned on auto-approve still needs to verify their email, or something else is blocking it, such as a missing shipping address once delivery is on.
6. `get_balance` shows whether the balance covers everyone waiting and the next gift a playbook can approve. When `lowBalance` is true, or a gift waits because the balance is too low, share `topUpUrl` so a person can add funds. Adding funds, raising a limit or turning on auto-approve doesn't re-check people already waiting: an owner or admin approves them in the queue, or you add the same person again with the same `sourceSystem` and `externalId`, which lets auto-approve try again and never creates a second gift.

To backfill several people into an active playbook at once, `add_recipients` works without `expectedPolicyVersion` and keeps the playbook on. Follow each refusal's `nextAction` and `url`.

## Prepare a one-time batch draft

1. Check for an existing campaign with `list_campaigns` and reuse it if the user means that one.
2. Otherwise call `create_campaign` with `mode: "batch"`, `status: "draft"`, a `name`, the chosen `productId`, `budgetCents`, `perPersonCents`, `maxSends`, `senderName`, `senderCompany`, the shared `message` (printed note, up to 500 characters) and one stable `idempotencyKey` for this campaign. Set `perPersonCents` to the user's per-person budget, `maxSends` to the number of people, and `budgetCents` to the two multiplied; `perPersonCents` cannot exceed `budgetCents`, and adding people later raises these limits automatically. Retrying with the same key and content returns the same draft.
3. When the gift has `personalizationText`, `customization.displayText` is required before payment. Ask the user for the text; do not invent it.
4. Add people with `add_recipients`, using the campaign's current `expectedPolicyVersion` from `get_campaign`. Each person needs at least one of a name, a personal LinkedIn `profileUrl` (https://www.linkedin.com/in/…; Sales Navigator and company links are refused), or a `sourceSystem` and `externalId` from the user's CRM; include whatever the user supplied. Email and company are optional. For a pasted list without CRM IDs, use the list's name (or "manual") as `sourceSystem` and something that stays the same per person, such as the LinkedIn profile slug or row number, as `externalId`, and reuse them on every retry. A per-person `message` overrides the shared note. If some people have no company, keep {{company}} out of the shared note or give them their own note; a placeholder without a value blocks approval.
5. Call `preview_campaign` and `get_campaign_repeats`, then report the total, any blockers, `nextAction` and the review link (`https://gtmswag.com/campaigns/{id}?review=1`).

Addresses: when the user supplies a full US address, use `addressMode: "known"`. Otherwise use `addressMode: "confirm"`: after a person pays, the recipient confirms their own address through a private gift link. Never invent an address.

With mode "live", a recipient can confirm their own address only in a batch or single campaign whose `preview_campaign` returns a `prepaidOffer` with `nextAction: "review"`: GTM Swag has qualified the gift for a fixed, tax-inclusive price per recipient, which a person pays on gtmswag.com before links are prepared; recipients then confirm an address in one of the offer's states. Playbooks are paid from the balance and never have one. When it returns `nextAction: "provide_sender_address"`, or approval returns `LIVE_ADDRESSLESS_UNAVAILABLE`, explain that a US shipping address is needed and share the campaign's edit page, or stop with the saved draft. Never invent a payment or gift link.

Photos: portrait gifts need an uploaded photo whose rights the sender confirmed (`upload_artwork` with `purpose: "portrait"`), or a LinkedIn profile for staff to source. The sender approves the exact image on the website. Never assume photo permission.

Adding people returns the campaign to draft for another review. On `POLICY_CHANGED`, read the campaign again and add only the people still missing. Match people by recipient ID or exact CRM IDs, never by name alone. Send batches one at a time, up to 500 people per campaign; ask before splitting a larger list.

Nothing has been charged or sent at this point. Hand the review link to the user: a teammate reviews the people, notes and final price, and an owner or admin pays on gtmswag.com.

## Gift links and CSVs

After review and payment, use the same campaign:

- **One person:** `get_gift_link_records` with the `campaignId` and either the person's `recipientIds` (from `get_campaign`) or a `search` for their name, email or CRM ID returns their private link and its expiry, or a reason it isn't ready. Explain the reason; never create another gift just to get a link.
- **Up to 25 people:** show the links as a table with names and expiry.
- **More than 25:** `export_gift_links` with `scope` `"all"`, `"selection"` or `"filtered"`, then `download_gift_link_export`, which returns the CSV as text; give it to the user as a file. The export's download URL needs the same credentials, so it isn't a link the user can open. Report counts, exclusions and the earliest expiry.
- To keep the user's original CSV columns, call `save_recipient_import` with the selected columns and match rows by import and recipient IDs. For large files use `upload_gift_source_chunk` with the same `uploadId` on retries. `remove_recipient_source_columns` deletes retained columns without touching recipients or gifts.

Only gifts added with `addressMode: "confirm"` have links, and only once the gift is approved or paid. Gift links are private, single-recipient links that expire 14 days after the gift is created, whether or not anyone opened them, so suggest approving close to the outreach (a LinkedIn connection request can take longer than 14 days). An unused gift whose link expired can be regifted on gtmswag.com. Keep links out of shared summaries. Reading or exporting never buys, sends, renews or changes a gift, and does not show that anyone opened it. The user sends their own outreach; do not write outreach copy unless asked.

## Status

`get_tracking` (filter by `campaignId`) reports existing gifts: waiting for an address, in production, shipped and delivered. Use `list_tracking_events` with its returned cursor for later changes, and `get_send` for one gift's timeline. Keep postal and email addresses out of summaries. Delivery is not evidence of a reply, a meeting or revenue. GTM Swag doesn't track replies or meetings; never infer them or ask the user to report them.

## Permissions and limits

Reading needs the `gifts:read` permission; drafts, recipients, imports and uploads need `gifts:write`; private links and CSVs need `gifts:links:read`, which the user grants separately. If access is denied, ask the user to reconnect; never switch accounts or credentials.

The connector cannot pay, turn on playbooks, turn on or change auto-approve, approve campaigns, playbook gifts or repeat gifts, add funds, launch or send gifts, email recipients, issue refunds or change team access. Do not work around that with other tools or browser automation. Users can disconnect at https://gtmswag.com/oauth/disconnect; saved drafts stay in their workspace.
