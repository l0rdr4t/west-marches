# Quest lifecycle: cancellation, drop-outs, waitlist promotion, and spent slots

Type: grilling
Status: resolved
Blocked by: -

## Question

The happy path is settled (ADR 0003). The edges are not, and they are where the design
either holds or falls over.

- What are a quest's exact states, and which transitions are legal? Proposed, confirmed,
  played, abandoned, declined, stale — is that the set?
- What is the party size on a quest: a maximum only, or a minimum too? Can a Game Master
  confirm a quest that hasn't attracted enough interest?
- A confirmed party member drops out. Does the waitlist auto-promote the next character,
  or does someone decide? What if it's an hour before the quest starts?
- A Game Master cancels a confirmed quest. Is the slot returned to the pool? Does the
  quest go back to proposed with its interest intact, or die? Do the players who lost that
  slot for their own quests get it back?
- A Game Master withdraws a slot that nothing has been confirmed into. What happens to the
  availability players declared against it?
- A quest is confirmed and the day passes with nobody marking anything. Does it become
  played automatically, or sit waiting?
- Does being in a party that never actually met count toward "played least recently"?
- What does "stale" mean concretely — how long, and does staleness hide a quest or just
  sort it down? (Q21 settled: no auto-deletion.)
- Surfaced by `04`: does a slot carry a duration or end time, or only a start? Players need to know
  whether they are committing to three hours or six before they declare interest.
- Surfaced by `15`, and now load-bearing for fairness rather than tidiness: **a cancelled
  quest must not spend anybody's standing.** ADR 0005 makes the *seat* the currency, not
  attendance, so a Game Master calling off a session would otherwise silently cost five
  people their turn on the rota. (This also answers the older bullet above — "does being in a
  party that never actually met count toward played least recently" — in the affirmative
  unless this ticket says otherwise, which is precisely the danger.)
- Surfaced by `15`: **stale interest on distant slots.** Deadlines are per slot, so someone
  can tick a date seven weeks out and be seated on it having long since forgotten. Withdrawing
  before that slot's deadline is free and costs no standing; what happens after it is this
  ticket's.
- Surfaced by `15`: **confirming an under-subscribed slot** — `10` found this is the common
  case, not an edge, with two of four candidate slots producing a party of four against five
  seats. Related to the party-size bullet above, and now with a deadline in play: the Rota
  cannot rank a set it has not finished collecting.

## Answer

**ADR 0006** — *Slots are a pool, and a quest can go back on the board*. It carries both
halves: where slots come from, and every way a quest ends or fails to.

### The question the ticket didn't ask: are slots offered to a quest, or offered generally?

Raised mid-grilling and it turned out to be the root of half this ticket. **A slot is an
unattached offer of a Game Master's time.** Quests are *pointed at* it — by their proposer or
by a Game Master — several may target the same date, and confirmation spends it for all of
them.

This was already implicit in ADR 0003, whose consequences say confirming "invalidates that
slot for every other quest targeting it", and in this ticket's own phrase "returned to the
**pool**". It had never been stated, and both other readings were live. **Slots offered to a
quest** leaves no board at all: dates would exist only after a Game Master had already
committed to a particular quest, and contradiction 8 says the critical path is the opposite —
campaigns die of under-supply of *initiative*, and what saved every surviving account was the
Game Master publishing dates unprompted. **No targeting**, every open slot a candidate for
every open quest, dies on `10`'s evidence: three Game Masters with four dates each is twelve
candidate dates per quest, and `10` found four against eight characters was already the hard
problem.

This is **contradiction 6**, which had no ticket — "our one-directional model covers only one
of the two real flows". The pool covers both: a Game Master posts a slate and quests claim it
(Lutes, and Colvillian's `#player-quest-request`), or a Game Master points a date at a quest
themselves in one act.

### The state set

**proposed → confirmed → played → closed**, plus **withdrawn** and **abandoned** as dead ends
from proposed. Cancellation is not a state; it is confirmed → proposed.

**No `declined`**, though `07` refers to one in passing: pointing a quest at a slot is a claim,
not an application, so there is nothing to reject, and declining is indistinguishable from not
confirming. **No `stale`**: the abandonment clock *is* the staleness rule, and `08`'s sidebar
already sinks a quest with no live date without needing a state to say so.

Two of this ticket's own bullets were already answered elsewhere and are struck rather than
decided: a confirmed quest becomes **played automatically when its slot passes** (`07` — "a GM
who forgets to mark it played silently stops the campaign's memory"), and whether a party that
never met counts toward standing is `15`'s, answered below.

### Cancellation keeps the quest and kills the date

Confirmed → proposed, **interest intact**, other candidate slots still live. The usual reason
is a diary clash rather than a judgement on the quest, and killing it would throw away a room
full of planning and a list of people who already said they wanted to go.

The **slot stays spent**. Cancelling is the Game Master saying that evening will not be run, so
returning it to the pool would advertise time that does not exist. That also answers this
ticket's question about the quests that lost the date at confirmation: **they do not get it
back**, because it was never the confirmation that took it from them.

**No report is owed** — only played quests can owe one (`07`), which is the "it didn't happen"
path `07` was waiting for. **Nobody's standing moves**, which `15` demanded and which now falls
out rather than needing a rule: every seat was given back before it was used.

### Drop-outs, and the sentence ADR 0005 was missing

**Promotion is automatic**, from the frozen waitlist, at any hour including one hour before.
The waitlist is ordered at confirmation precisely so this is not a decision, and a Game Master
choosing would be ADR 0005's override arriving by the back door.

**The seat is spent unless you give it back before it is used.** ADR 0005 said "withdraw before
the deadline and you keep your place; no-show and you have spent it", which left the gap between
the deadline and play unaddressed. Releasing a seat costs nothing right up to the start; failing
to release it costs your standing.

The strict alternative — any drop-out after the deadline spends it — is the tidier sentence and
it destroys what ADR 0005 bought. The whole point of making the seat the currency was to buy
*tell us you can't come*, and if announcing costs exactly what ghosting costs, nobody announces.
This version also keeps `15`'s independence from attendance data: releasing a seat is an act in
the app, ghosting is the absence of one, so the platform tells them apart with nobody correcting
anything afterwards.

It also makes late promotion safe with no special case — someone promoted with sixty minutes'
notice who cannot make it releases the seat and keeps their place.

### The clock, and the two ways a slot dies

**A quest is abandoned 14 days after the last moment anything could still confirm it** — its
most recent candidate slot's deadline passing unconfirmed, or 14 days from proposal where no
slot was ever offered. Offering a new slot restarts it. Explicit **withdrawal** exists too
(`05` already uses it for a deactivated player's unconfirmed quests), but the clock does the
real work, because `08`'s requirement was specifically that abandonment be "a real terminal
state that **something actually sets**". 14 days is `07`'s constant, reused rather than
reinvented, per `04`'s precedent that this campaign has no settings page.

**Withdrawing an unconfirmed slot takes every declaration against it**, on every quest
targeting it. `05` requires slots to be withdrawable; keeping the declarations would keep rows
that can never be ranked or seated, which ADR 0003 already says are not interests.

### Sizes, ends and who may cancel

- **A slot carries a start and an end.** `10`: every layout wanted "17:00–21:00 · about 4
  hours", and "the version with only a start reads as an ambush". `07`'s report clock already
  ran from "the slot ends", so this was load-bearing before it was decided.
- **No minimum party size.** A ceiling only. Nothing in `03` has a floor — the Alexandrian's
  table cap is a maximum, and the founding text is "we'll play with whoever shows up" — and the
  format's cause of death is under-supply of *sessions*, so a rule whose only power is to stop
  one happening aims at the wrong failure. Not even an advisory minimum: `10` already settled
  that the page says "1 seat unfilled" as a neutral fact rather than a warning. A slot with zero
  interest still cannot be confirmed, because there is no party to cut — arithmetic, not a rule.
- **Any Game Master may cancel any confirmed quest**, not only its own. Otherwise a Game Master
  who has drifted away without being deactivated leaves quests confirmed onto dead dates that
  nobody can touch. Routing it through an administrator was rejected: `CONTEXT.md` says "an
  administrator's mistakes are operational; a Game Master's are social", and cancelling a
  session is the second.
- **Deactivating a Game Master cancels their confirmed quests**, landing them back at proposed
  where another Game Master can point them at a new date. `05`'s rule that mere *revocation*
  leaves confirmed quests alone stands untouched.

### Rejected, with reasons worth keeping

- **A minimum party size**, blocking or advisory. Aimed at the wrong failure mode.
- **Returning a cancelled quest's slot to the pool.** It advertises an evening the Game Master
  has just said they cannot run.
- **Killing a cancelled quest.** Throws away the room, the interest and the planning to model a
  diary clash.
- **A Game Master deciding who gets a freed seat.** ADR 0005's override, by the back door.
- **`declined` and `stale` as states.** Both cost a transition, a permission and a notification
  and add nothing the clock and the sidebar do not already do.

### Handed on

- **`09` is unblocked, and inherits three events it must carry.** A Game Master withdrawing a
  slot can **end someone's interest in a quest without them acting** — that is a silent removal
  by another person's hand and the sharpest notification in the set. Cancellation has to reach a
  party that has already put the date in a diary. And automatic promotion can hand someone a
  seat an hour before play, which is only useful if it reaches them.
- **`11` is a two-axis screen.** With several quests targeting one date, a Game Master chooses
  *which quest* as well as which slot. ADR 0005's "the only judgement is which slot" was about
  the party and still holds, but the confirmation screen is a bigger object than `11` currently
  describes, and `10`'s column footer prices only one axis of it.
- **`08` — freezing has three triggers, not one.** Closed, withdrawn and abandoned. `08` derives
  freezing from quest state so no machinery changes, but it was written expecting a single
  trigger.
- **ADR 0005's bump definition was sharpened** rather than changed: seated or not *in the quest
  you declared on*. Without it, a quest that merely loses its date to another quest deals its
  declarers a bump nobody dealt them.
- **ADR 0003 gained a second "those proposers have to be told" case** — slot withdrawal.

### Vocabulary

Rewrote **Slot** (unattached, targeted by quests, start and end), **Quest** (the six states),
**Confirmation** (chooses the quest as well as the date), **Party** (a seat may be given back
free before play), and **Waitlist** (promotion is automatic).
