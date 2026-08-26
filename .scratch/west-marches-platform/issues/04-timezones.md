# One campaign clock, or does every player see their own?

Type: grilling
Status: resolved
Blocked by: -

## Question

Slots are dated windows (ADR 0003) and this is the decision that has to be right before
any of them are stored, because getting it wrong is a data migration rather than a
refactor.

- Does the campaign have a single declared timezone that everything is displayed in, or
  does each player see slots converted to their own?
- If per-player: where does that timezone come from — a profile setting, the browser, or
  both with one overriding the other?
- What is stored on a slot: an absolute instant, or a wall-clock time plus a zone? These
  differ when a Game Master schedules across a DST boundary and expects "7pm" to stay 7pm.
- Does a player ever need to state availability in a way that isn't tied to a specific
  slot (a recurring "weeknights are bad for me")? If not, say so explicitly — it keeps
  the model small.
- What does the interface show when a player's local rendering of a slot lands on a
  different calendar day than the Game Master's?

## Answer

**The whole ticket resolves to "change nothing".** Campfire's existing pattern is already
correct for this campaign, and no new mechanism, column or setting is needed.

- **The campaign is in one timezone.** Cross-timezone coordination is not a real case here.
- **Times render per-player, in the browser.** Reuse `app/javascript/controllers/local_time_controller.js`
  and the `local_datetime_tag` helper exactly as messages do (`app/views/messages/_message.html.erb:6`).
  No campaign clock, no second convention.
- **A slot stores an absolute UTC instant. Nothing else.** No authoring zone, no wall-clock plus
  zone. Discrete dated slots (Q12) have no recurrence to survive, so the instant is unambiguous, and
  with a single-zone campaign the authoring zone is always the same zone and therefore dead weight.
- **No standing availability.** Availability is always declared against specific offered slots.
  "I'm away all fortnight" is a message in the room, not a state in the schema.
- **No campaign timezone setting**, and **no zone label anywhere in the interface**. A single-zone
  campaign never needs one, and a label shown 99% redundantly gets ignored on the 1% that matters.

### Stated assumption, not decided here

Push payloads are built server-side as plain strings (`lib/web_push/notification.rb:2`), where no
browser zone exists. With no campaign setting and no per-user column, Rails would format in UTC.
This resolution assumes `config.time_zone` is set to the campaign's zone in `config/application.rb`
— one line, no UI, not a feature. **Whether notification copy should contain times at all is
handed to `09`.**

### Revisit trigger

If a player relocates or joins from another timezone, three of these reverse together: per-player
rendering silently starts shifting times, the absent zone label becomes a hazard, and the server's
single zone becomes wrong for someone. Treat "a player is in another zone" as the signal to reopen
this as a fresh ticket.
