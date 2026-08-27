# Slots are a pool, and a quest can go back on the board

ADR 0003 settled the happy path and ADR 0005 settled who gets a seat. This one settles the
edges: where slots come from, and every way a quest can end or fail to.

**A slot is an unattached offer of a Game Master's time.** It is posted with no quest on it.
A quest is then *pointed at* a slot — by its proposer, or by a Game Master — which makes it
one of that quest's **candidate slots**, and several quests may target the same date.
Confirmation picks one of them and spends the slot for all the others.

This was implicit in ADR 0003, whose consequences already say confirming "invalidates that
slot for every other quest targeting it", but it was never stated and the two other readings
were live. **A slot offered *to* a quest** was rejected because it leaves no board: dates
would only exist once a Game Master had already committed to a particular quest, and ticket
`03`'s contradiction 8 says the critical path is the opposite — campaigns die of *under-supply
of initiative*, and what saved every surviving account was the Game Master publishing dates
unprompted. **No targeting at all**, with every open slot a candidate for every open quest,
was rejected on prototype `10`'s evidence: three Game Masters with four dates each puts twelve
candidate dates on every quest, and `10` found four dates against eight characters was already
the hard problem.

The pool is also what practice does — Lutes' "slate of dates, with **claiming** rather than
voting"; Colvillian's `#player-quest-request`, where players request and Dungeon Masters pick
up — and it subsumes the rejected first reading, since a Game Master who wants to offer a date
for one particular quest simply points it there in the same act.

**A slot carries a start and an end.** Not a duration: `10` found every layout wanted to write
"17:00–21:00 · about 4 hours" and that "the version with only a start reads as an ambush",
and `07`'s report clock already runs "14 days after the slot ends".

**A quest has six states.**

- **proposed** — on the board, collecting candidate slots and interest.
- **confirmed** — bound to one slot.
- **played** — its slot has passed. Automatic; `07` made this not a Game Master action.
- **closed** — the report is published, or 14 days have passed. `07`'s trigger, and what
  freezes the room under `08`.
- **withdrawn** — ended deliberately from proposed.
- **abandoned** — ended by the clock from proposed.

There is no **declined** state, though `07` refers to one in passing. Pointing a quest at a
slot is a *claim*, not an application, so there is nothing for a Game Master to reject;
declining is indistinguishable from not confirming, and the clock below carries such a quest
to abandoned on its own. A state whose only distinction is that a human said no out loud costs
a transition, a permission and a notification, and buys nothing.

There is no **stale** state either. The clock *is* the staleness rule, and `08`'s sidebar
already orders by soonest candidate slot with slotless quests below, so a quest with no live
date is visibly sinking without needing a state to say so.

**Cancellation is not a state — it is confirmed → proposed.** The usual reason a Game Master
cancels is a diary clash, not a judgement on the quest, so the quest goes back on the board
with its **interest intact** and its other candidate slots still live. Killing it would throw
away a room full of planning and a list of people who already said they wanted to go.

The slot, however, **stays spent**. Cancelling *is* the Game Master saying that evening will
not be run, so returning it to the pool would advertise time that does not exist. This also
answers what happens to the quests that lost the date when this one was confirmed: they do not
get it back, because it was never the confirmation that took it from them — it was the Game
Master's evening disappearing. A Game Master who cancels the quest but keeps the date offers a
new slot for it, which is one extra act in a rare case.

**Any Game Master may cancel any confirmed quest**, not only the one whose slot it sits in.
The alternative leaves quests that have drifted — a Game Master who stopped showing up without
being deactivated — confirmed onto dead dates with nobody able to touch them. Routing it
through an administrator instead was rejected because `CONTEXT.md` is explicit that "an
administrator's mistakes are operational; a Game Master's are social", and calling off a
session is squarely the second.

**Deactivating a Game Master cancels their confirmed quests.** The alternative is a quest
confirmed into the calendar of somebody who no longer exists, waiting for a session that cannot
happen. Cancelling lands it exactly where a takeover needs it: back at proposed, interest
intact, ready for another Game Master's date. This is deactivation only — ticket `05` already
decided that revoking the flag leaves confirmed quests alone, and that stands.

**A quest dies on a clock when nothing can confirm it any more.** It is abandoned 14 days
after the last moment anything could — its most recent candidate slot's deadline passing
unconfirmed, or, where no slot was ever offered, 14 days from proposal. Offering a new slot
restarts it, so a quest with a live date three months out is never at risk. Ticket `08`
required this specifically: abandonment has to be "a real terminal state that **something
actually sets**", or dead proposals never freeze and slowly fill the sidebar. 14 days is
`07`'s constant, reused rather than reinvented.

**Withdrawing a slot takes every declaration against it.** A Game Master may withdraw a slot
nothing has been confirmed into — `05` already requires it, since revocation withdraws unspent
future slots — and the declarations against it go too, on every quest that was targeting it. An
interest naming no live date is not an interest, by ADR 0003.

**There is no minimum party size.** A maximum only. Nothing in `03` has a floor — the
Alexandrian's table cap is a ceiling, and the founding text is "we'll play with whoever shows
up" — and the format's documented cause of death is under-supply of *sessions*, so a rule
whose only power is to stop one happening is aimed at the wrong failure. `10` found
under-subscription is the common case rather than an edge, and settled the presentation: "1
seat unfilled" as a neutral fact, not a failure. A Game Master who will not run for two people
simply does not confirm.

**A freed seat promotes itself.** The waitlist is frozen and ordered at confirmation precisely
so that "who gets the seat" is not a decision, and a Game Master choosing would be the override
ADR 0005 removed, arriving by the back door. Promotion is automatic at any hour, including one
hour before.

**And the seat is spent unless you give it back before it is used.** ADR 0005 said "withdraw
before the deadline and you keep your place; no-show and you have spent it", which left the gap
between the deadline and play unaddressed. Releasing a seat at any point before the quest
starts costs nothing; failing to release it does. The strict alternative — any drop-out after
the deadline spends your standing — is the tidier sentence and it destroys what ADR 0005 bought:
if announcing costs exactly what ghosting costs, nobody announces. This version also needs no
attendance data, because releasing a seat is an act in the app and ghosting is the absence of
one.

## Consequences

- **Confirmation is a two-axis decision.** With several quests targeting one date, a Game
  Master chooses *which quest* as well as *which slot*. ADR 0005's "a Game Master's one
  judgement is which slot" was about who is in the party and still holds, but the confirmation
  screen is a bigger object than it looked.
- **ADR 0005's definition of a bump is sharpened**, not changed: seated or not **in the quest
  you declared on**. Without that, a player whose quest merely lost the date to another quest
  collects a bump nobody dealt them.
- **A Game Master's act can silently remove someone from a quest.** Withdrawing a slot that was
  a player's only live date on a quest ends their interest in it. They have to be told, which
  is the notification ticket's.
- **Cancellation moves nobody's standing.** Every seat was given back before it was used. This
  is not a special rule; it falls out of the one above.
- **A cancelled or abandoned quest never owes a report.** Only played quests can, per `07`.
- **Room freezing has three triggers, not one**: closed, withdrawn and abandoned. Ticket `08`
  derives freezing from quest state, so this needs no new machinery — but `08` was written
  expecting one.
