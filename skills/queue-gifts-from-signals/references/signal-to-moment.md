# Signal to moment

The editorial mapping this skill uses in Step 2 and Step 3. It is the author's judgment about fit, not a measured ranking. The installer chooses which rows apply.

## The moments, in precedence order

When a row carries two signals, the one listed first here wins, and the output says which signal was used.

| Order | Signal (what the source calls it) | Moment, sent as `useCase` | Why this order |
|---|---|---|---|
| 1 | Attended, spoke at or registered for an event or webinar you ran or went to | `event_follow_up` | A shared moment beats a public one: you were in the same room |
| 2 | Champion or past buyer changed jobs; new hire into a buying role at a target account | `new_role` | The new seat is the reason, and it decays fastest |
| 3 | Funding round, product launch, acquisition, company anniversary | `company_milestone` | Public and dated, so the note can name it exactly |
| 4 | Stalled or closed-lost deal being reopened | `deal_reengagement` | Only with an owner who will follow up |
| 5 | Renewal, closed-won, customer milestone | `customer_thanks` | A thank-you, never an ask |
| 6 | On a target-account list with no event | `cold_outreach` | The weakest why-now; keep the note about them, not you |

Signals not in this table (website visits, intent scores, technographic matches) are not a gift moment on their own. Report them as `signal_not_chosen` unless the installer maps one explicitly, and say which moment they mapped it to.

## Freshness

The author's default window is 30 days from the signal date for every moment. It is borrowed, not tested. An installer may set one window per moment; record which they chose. A row with no signal date is `stale_signal`, never assumed fresh.

## Notes

Two sentences, under 500 characters, naming the moment and asking for nothing. Use `{{firstName}}` and `{{company}}` only when every row in the moment has the field.

| Moment | Pattern | Never |
|---|---|---|
| `event_follow_up` | "Good to meet you at {event}, {{firstName}}. A small thank-you for the conversation." | claim a conversation that did not happen |
| `new_role` | "Congrats on the new role, {{firstName}}. A small something for your first month." | mention the previous employer's internal matters |
| `company_milestone` | "Congratulations on {milestone}, {{firstName}}. Here's to what the team builds next." | guess a round size or investor not in the source |
| `deal_reengagement` | "Thinking of the {{company}} team, {{firstName}}. Hope the quarter is treating you well." | reference pricing, the lost deal or a competitor |
| `customer_thanks` | "Thank you for {reason}, {{firstName}}. We're glad to be working with {{company}}." | ask for a referral, a review or a meeting |
| `cold_outreach` | "{{firstName}}, a small something for the {{company}} team." | flattery, invented common ground, a meeting ask |

Replace each `{...}` with a fact from the source row. If the fact is not in the row, use the pattern without it rather than invent one.

## Plan-only gift kinds

Used only when no gifting workspace is connected, so the plan still names a sensible kind of gift. These are kinds, not products, and carry no price.

| Moment | Gift kind |
|---|---|
| `event_follow_up` | a food gift that arrives within the week |
| `new_role` | a desk item personalized with their name or new company |
| `company_milestone` | something the team can share |
| `deal_reengagement` | a modest, practical desk item |
| `customer_thanks` | a quality item they would keep, branded lightly or not at all |
| `cold_outreach` | an unusual, personalized desk item that does not need explaining |

## Exclusions that apply to every moment

- Alcohol only when the installer confirms every recipient is 21 or older (`excludeAlcohol: true` otherwise).
- Portrait gifts only when the installer supplies a photo they have the rights to.
- No inference about hobbies, health, religion, family, identity or drinking habits from names, titles or photos.
