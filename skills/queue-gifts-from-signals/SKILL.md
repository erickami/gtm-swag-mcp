---
name: queue-gifts-from-signals
description: |
  Turn sales signals into physical gifts queued for approval: a champion job change, a new VP at a
  target account, a funding round or launch, event or webinar attendees, a stalled deal, a renewal, or
  a target-account list. Matches each person to one gift moment, picks a gift for it within budget,
  writes a short note that names it, dedupes on a stable row ID, shows a dry run, and after one
  explicit approval adds people to a GTM Swag gifting playbook, where an owner or admin approves or
  skips each gift before anything is charged or shipped. Recipients confirm their own US address
  through a private link; no home address is looked up. Use whenever someone asks: send a gift when a
  champion changes jobs, gift new VPs at target accounts, congratulate funded accounts, follow up
  event attendees with a gift, run a gifting play off Clay signals, or queue meeting gifts for
  approval. Do NOT use it to research hobbies for gift ideas, write cold emails, find emails, phones
  or addresses, or approve, pay for or ship gifts.
---

# Queue gifts from signals (the moment picks the gift; the recipient gives the address; a person approves)

**The insight: for a cold prospect you do not have a shipping address, and you should not go looking for one, so the gift travels as a private link the person uses to confirm their own address, and that link expires 14 days after the gift is created.** That one fact sets the order of everything here. A signal is only fresh for a short time. The gift has to be approved close to the moment the outreach carrying its link goes out, or the link expires unused. A list approved weeks ahead of its outreach is a list of expired links. So this skill queues people as their signals arrive, a small batch at a time, rather than one large list once a quarter.

The second half of the insight: **the moment is the personalization.** A gift that names why now ("congratulations on the new role", "congratulations on the raise", "good to meet you at the summit") needs nothing else about the person. Researching someone's private interests is a different job with different risks; if it has already been done by another skill, its output can feed the note here, and nothing in this skill infers interests, health, family, religion, identity or drinking habits.

Do not start a step before the steps above it have their answers. If a declared input is missing, ask for it; never assume a default and continue.

## Declared inputs

**Nothing here ships with a value.** Each one is the installer's, not the author's: ask for it, never substitute a plausible default, and if an answer does not exist say which step becomes unavailable rather than guessing. Where a default IS defensible it is named below, and using it means saying so in the output.

| Input | What the installer supplies | If it is missing |
|---|---|---|
| **The people and their signal** | a signal digest from another skill (for example a champion job-change digest or a buying-signal digest), a table export, or a CSV; per person a name, and at least one of a personal LinkedIn `/in/` profile URL or a stable row ID, plus the signal and its date | no default: there is nothing to gift. A row with no name, no profile URL and no row ID is reported as `not_added`, never guessed at |
| **Which signals count** | which signal types become gifts, from the moment table in `references/signal-to-moment.md` | ask as a choice. Never widen it silently: a signal type nobody chose is not a reason to spend |
| **Freshness window** | how old a signal can be and still earn a gift | the author uses 30 days; offer it, and if they take it say it is borrowed. Never leave it unset, which silently means any age |
| **Budget per gift** | the most one gift may cost, delivered, in US dollars | no default: ask. It decides which gifts can be offered at all |
| **Sender name and company** | the name and company printed with the note | no default: a substituted sender is not recoverable once printed |
| **Suppression list** | who must never get a gift: open opportunities, current customers, competitors, people gifted recently, anyone on a do-not-contact list | ask. If they have none, say that the run had no suppression list in the output |
| **Where people already have a US address** | a column with a full US street address, or none | the default is none: every recipient confirms their own address through the private link. Never look one up |
| **The gifting workspace** | a GTM Swag account connected to this agent | no reduced write mode: without it, Step 7 delivers the plan only (moment, gift kind, note) and nothing is queued |

**If an answer sheet is present beside this skill, load it and ask only for what it does not cover.** A partial sheet is normal; a value it is missing gets asked for on its own rather than restarting the interview. **Say which values came from the sheet** before using them; a sheet applied silently is a wrong field nobody catches. **If there is no sheet, say nothing about sheets.** At delivery, offer to save the answers back (identifiers only, never a token or a password), private and never published: *"want me to save your answers to a file, so the next run doesn't have to ask these again?"*

## What this skill touches

- **Reads**: the people and signals you supply; the gifting catalog; your gifting workspace's playbooks, their auto-approve rule, the approval queue and the balance.
- **Writes**: people added to gifting playbooks you approved at Step 4, each with name, company, title, LinkedIn `/in/` URL when supplied, your row ID and the note; and, only when no playbook fits a moment, a new playbook saved as a draft that a teammate must turn on. That is a write to your gifting workspace, not a read.
- **Never**: approves, pays for, ships or sends a gift; adds funds; turns on or changes auto-approve; emails, messages or enrolls anyone; writes to a CRM or a sequence; looks up a home address, email or phone number; deletes or edits a playbook it did not create.
- **Halts**: Step 4 `sample-review`, Step 4 `write-approval`, Step 4 `spend-approval`, Step 5 `other`.
- **Vendor-specific**: GTM Swag (gtmswag.com) is where the gift is chosen, queued, approved and fulfilled. An account is free to create and works end to end in test mode, where nothing ships and no real money moves. Without an account the skill still runs Steps 1 to 3 and delivers the reviewed gift plan, and queues nothing.

## Step 0: Check the platform, and say what this does

Say this first, in two sentences: *this reads the people and signals you point me to, matches each person to a gift moment, and, after one approval from you, adds them to a gifting playbook where an owner or admin on your team approves or skips each gift. This skill never pays for, ships or sends anything, and it never looks up anyone's address.*

1. **The gifting tools.** Call `get_workflow_guide` and read `mode`.
   - If the tools are not available, give the one fix and stop. In Claude Code: `claude mcp add --transport http gtm-swag https://gtmswag.com/mcp`, then run `/mcp` and sign in to GTM Swag. Other agents, and the key-based alternative, are in `references/gifting-calls.md`. With no account, the installer creates one at `https://gtmswag.com/login?utm_source=clay-skills&utm_medium=skill&utm_campaign=queue-gifts-from-signals`. Do not install, upgrade or fetch anything to repair it. If they would rather not connect, continue in plan-only mode and say so.
   - If `mode` is `test`, say so plainly: everything works end to end, but nothing ships and no real money moves until delivery is enabled for the workspace, so try it with your own rows, not real prospects.
2. **The signal source, only if it is Clay.** Run `clay whoami; echo "exit_code=$?"` and name the workspace. If it fails, give the one fix (`clay login`) and stop. A CSV or a digest already in the conversation needs no check.

Where the work runs: matching, deduplication and note drafting happen in the agent and cost nothing. Reading the catalog and playbooks is free. The only spend is a gift a person approves, or one an auto-approve rule approves inside limits an owner or admin set.

## Step 1: Collect the definition (interview; do not guess)

Front-load what changes scope or cost, in this order: which signals count, the freshness window, the budget per gift. Ask the sender name and company, the suppression list and the address column at the step that needs them.

Read the people file and **show the mapping you found** before asking anything about it: which column is the name, the company, the title, the profile URL, the row ID, the signal type and the signal date, and which you could not match. Ask only about the gaps. Never ask for a recital of column names.

## Step 2: Match each person to one moment (the decision this skill makes)

Each row gets exactly one verdict from this set, resolved in this order, first match wins. Seven values, no eighth.

1. `not_added`, reason `suppressed`: the person or their company is on the suppression list.
2. `not_added`, reason `cannot_identify`: no name, or neither a personal `/in/` profile URL nor a stable row ID. Sales Navigator, company and `/pub/` links do not count; drop them from the row rather than send them.
3. `not_added`, reason `outside_us`: the person is known to be outside the US. Unknown country is not a reason; the recipient confirms the address.
4. `not_added`, reason `stale_signal`: the signal date is older than the window, or missing.
5. `not_added`, reason `signal_not_chosen`: the signal type is not one the installer chose.
6. `duplicate`: the same person (same profile URL or same row ID) appeared earlier in this run. The earlier row keeps its verdict.
7. Otherwise, the moment from the table in `references/signal-to-moment.md`: `new_role`, `company_milestone`, `event_follow_up`, `cold_outreach`, `deal_reengagement` or `customer_thanks`.

A row that matches two signals takes the one listed first in that table, and the output says so.

## Step 3: Free checks, the gift, and the note

Everything here is a read and costs nothing.

1. **Existing playbooks.** `list_campaigns` with `mode: "always_on"`. For each moment in use, find a playbook that fits (same moment, gift within budget, note that names the moment). Reuse it. Read its rule with `get_campaign`: whether auto-approve is on and its per-gift limit. This decides whether Step 4 is also a spend approval.
2. **The gift, only for moments with no playbook.** `search_gifts` with that moment as `useCase`, `budgetCents` set to the budget, `recipientCount` set to the people in that moment, and `excludeAlcohol: true` unless the installer explicitly confirms every recipient is 21 or older. Offer three gifts per moment, each with its delivered price, the personal touch and one tradeoff. `get_gift` on the chosen one: if it needs a portrait photo, artwork or lettering the installer has not supplied, say so and offer the next option. Never invent a price, lead time or availability.
3. **The note.** One note per moment, two sentences, under 500 characters, that names the moment and makes no ask. A thank-you never asks for a meeting, and no gift is conditional on a reply. Use `{{firstName}}` or `{{company}}` only if every row in that moment has the field; otherwise write the row's own note in its `message`. Patterns are in `references/signal-to-moment.md`; never invent a conversation, a compliment or a shared connection.
4. **Balance.** `get_balance`. If auto-approve is on and the balance will not cover the people in that playbook, say so now, with `topUpUrl` for an owner or admin.

## Step 4: Dry run, then ONE gate

Adding a person to a playbook is a write to the gifting workspace, and with auto-approve on it approves and charges a gift. It cannot be taken back by this skill. So nothing is added before this gate, and the gate carries everything in one message:

- the verdict table from Step 2 for every row, including every `not_added` and `duplicate` with its reason;
- per moment: the playbook (existing, or a new draft to be created), the gift, its delivered price and the note;
- **the write, in words**: *"this adds N people to playbook P in your gifting workspace. That is a write, not a read."*;
- **the spend, in words**: if auto-approve is off, *"nothing is charged now: each gift waits for an owner or admin to approve or skip it."* If it is on, *"auto-approve is on with a per-gift limit of L: gifts within it are approved and charged to the workspace balance as they are added, up to N × price = T."*;
- the one ask: *"Add these N people?"*

Then stop and wait for an explicit yes. A partial yes ("only the new VPs") narrows the run; re-show the narrowed counts before writing. Stop a second time only if something appears that nobody could have anticipated, such as a row count far from the file's.

## Step 5: Do the work

1. **New playbooks, if any.** `create_campaign` with `mode: "always_on"`, `status: "draft"`, a `name` that says the moment, the `productId`, `senderName`, `senderCompany`, the note as `message`, and a stable `idempotencyKey` (for example the moment plus the date). Give the installer the review link `https://gtmswag.com/campaigns/CAMPAIGN_ID?review=1`: a teammate checks the gift and note and selects **Turn on playbook**. **Halt here** until they say it is on; adds to a draft fail with `PLAYBOOK_OFF`.
2. **Add people.** `add_to_playbook` once per approved person, with `name`, `company`, `title`, `profileUrl` (only a personal `/in/` URL), `sourceSystem` (`clay` for Clay rows, otherwise the file's name) and `externalId` (the stable row ID), `addressMode: "confirm"`, and `message` only when the row has its own note. With a full US address column, send `addressMode: "known"` with `address`, `city`, `state` and `zip`. Send nothing else: no email unless the installer supplied it for this purpose, and no extra fields.
3. **Errors.** `PLAYBOOK_BUSY` or `RATE_LIMIT`: wait `retryAfterMs` and resend the same call; it never creates a second gift. `RECIPIENT_MESSAGE_REQUIRED`: the note uses a field this row lacks; write the row its own note and resend. `VALIDATION_ERROR` on `profileUrl`: drop it and resend if the row still has a row ID. Anything else: record it as `blocked` with the error code and continue.

## Step 6: One verdict per person

Read `approval.state` from each result and map it, no other values:

- `queued`: waiting for an owner or admin; give the `reason` in plain words (`auto_approve_off` is "waits for a person to approve").
- `auto_approved`: the playbook's rule approved it and charged the balance.
- `already_queued`: the result had `alreadyQueued: true`; nothing new was created or charged.
- `skipped_by_team`: someone on the team skipped this person earlier.
- `blocked`: an error the skill could not resolve, with its code.
- plus every `not_added` and `duplicate` from Step 2.

## Step 7: Deliver

The verdict table, the counts, and the approval link (`approvalUrl` from the results, or `list_queue` filtered to the playbook). Then say what happens next, once: after a gift is approved, its private link appears in the gifting workspace and expires 14 days after the gift was created, so the outreach that carries each link should go out that week. This skill does not write or send that outreach.

In plan-only mode, deliver the same verdict table with the moment, a gift kind from `references/signal-to-moment.md` and the note, and say that nothing was queued.

## Representative output

### Gift queue for this run

| Person | Company · Title | Signal | Moment | Gift · delivered price | Note | Result |
|---|---|---|---|---|---|---|
| Dana Whitfield | Northwind · VP Sales | started new role, Sep 29 | new role | personalized desk piñata · $54 | "Congrats on the new role, Dana. A small something for your first month." | waits for a person to approve |
| Sam Ortiz | Contoso · Head of Growth | Series B announced, Oct 2 | company milestone | engraved tumbler · $48 | "Congratulations on the raise, Sam. Here's to what the team builds next." | approved by the team's auto-approve rule, $48 charged to the balance |
| Priya Nair | Fabrikam · RevOps Lead | attended the summit, Oct 8 | event follow-up | cookie box · $62 | "Good to meet you at the summit, Priya." | already in this playbook from an earlier run; nothing new created |

### People not added

| Person | Reason |
|---|---|
| Lee Park, Tailspin | on your suppression list (open opportunity) |
| row 14 | no name and no LinkedIn profile or row ID, so nobody could be identified |
| Chris Moe, Litware | job-change signal is 41 days old, outside your 30-day window (window borrowed from the author) |

### Coverage line

38 rows in · 31 matched to a moment · 4 not added · 3 duplicates · 31 added: 22 waiting for approval, 6 approved by auto-approve ($288 charged), 3 already queued · 0 blocked. Approve or skip at the playbook's queue link. Each gift link expires 14 days after its gift is created.

## What this skill does not claim

- Whether a gift sent this way leads to a reply or a meeting has not been measured, and the gifting workspace does not track replies or meetings.
- The moment table is the author's editorial judgment about which gift fits which signal, not a tested ranking.
- The 30-day freshness window is the author's default, not a tested figure.
- A signal is only as accurate as its source; this skill does not re-verify a job change, a funding round or an attendee list.
- It does not check whether the recipient's employer limits the gifts they may accept.
- Delivery is US-only, and lead times are production estimates, not delivery dates.
- In a workspace in test mode nothing ships and no real money moves.
- It has not been run end to end by an installer outside the author's team.

## What good looks like

- Every row in the file appears exactly once in the output, either with a result from the gifting workspace or with a `not_added` or `duplicate` reason a person can read without opening the skill.
- No person was added before the Step 4 yes, and the gate named both the write and, when auto-approve is on, the maximum charge.
- Running the same file twice creates nothing new: the second run reports `already_queued` for everyone the first run added.
- Each note names the moment and asks for nothing.
- A thin or failed run looks different: most rows `not_added` for `cannot_identify` usually means the profile URL column was mapped wrong in Step 1, and a run where every row is `blocked` with `PLAYBOOK_OFF` means the new playbook was never turned on.
- The common mistake: approving a large list long before the outreach that carries the links, so the links expire. Queue small batches as signals arrive.

## Rules

- MUST get one explicit yes at Step 4 before any `create_campaign` or `add_to_playbook`, and MUST name the write and any possible charge in that gate.
- MUST send `sourceSystem` and a stable `externalId` with every person, so a repeat can never create or charge a second gift.
- MUST use `addressMode: "confirm"` unless the installer supplied a full US address; NEVER look up, infer or enrich a home address, email or phone number.
- MUST exclude alcohol unless the installer confirms every recipient is 21 or older.
- NEVER approve, pay, add funds, turn on or change auto-approve, send gift links, or write or send outreach.
- NEVER infer hobbies, health, religion, family, identity or drinking habits from names, titles or photos.
- NEVER edit or delete a playbook this skill did not create.

## Worked example

Ask: "Gift the new VPs from my champion job-change digest, budget $60, from Jordan Lee at Example Co." Step 0: gifting tools connected, `mode` is `test`, so the skill says nothing will ship; Clay signed in to the "Example Co" workspace. Step 1: 38 rows; mapping shown (name, company, title, profile URL, row ID, signal, date), installer confirms; window 30 days (borrowed, said so); suppression list is their open-opportunities export. Step 2: 31 matched to `new_role`, 4 not added (2 suppressed, 1 cannot identify, 1 stale), 3 duplicates. Step 3: no playbook for new roles, so three gifts offered under $60; installer picks the piñata at $54; note drafted with `{{firstName}}`, which every matched row has. Step 4: one message with the table, "this adds 31 people to a new draft playbook 'New role gifts'. That is a write", and "nothing is charged now: auto-approve is off"; installer says yes. Step 5: draft created, review link shared, a teammate turns it on, 31 people added. Step 6: 31 `queued`, reason "waits for a person to approve". Step 7: table, counts and the queue link, with the 14-day link note.
