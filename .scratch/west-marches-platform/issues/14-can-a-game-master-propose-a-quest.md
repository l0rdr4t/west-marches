# Can a Game Master propose a quest?

Type: grilling
Status: resolved
Blocked by: -

## Question

`CONTEXT.md` defines a **quest** as "proposed by a player", and ADR 0003 builds the whole
three-act flow on it: a player proposes, Game Masters offer **slots**, players declare
**interest**, a Game Master **confirms**. Ticket `03` found that this is the minority path in
every campaign it could observe.

- Lutes measured the split at **70% Game-Master-posted, 30% player-initiated**.
- Colvillian runs `#dm-quest-posting` and `#player-quest-request` as **two separate channels
  with different rules** — they are different objects, not one object with a different author.
- Lutes' "town opportunities" is a job board the Game Master stocks; Ghelfi and
  Hipsters & Dragons describe Game-Master-posted missions only.
- The founding text is the other way: Robbins' players email the list with destination, date
  and recruitment call together. But Robbins had 10-14 players and no job board.

The questions:

- Can a Game Master propose a quest at all? If not, what stops them from proposing one through
  a player, which is what a workaround looks like on day one?
- Is a Game-Master-proposed quest the *same object* with a different proposer, or a different
  object with different rules? Colvillian's answer is the second, and ticket `15` depends on
  which.
- A player-proposed quest needs a Game Master to pick it up and offer slots for it. A
  Game-Master-proposed quest arrives with its slots already attached — does that collapse
  proposal and slot-offering into one act, and is that a different lifecycle?
- Does a Game-Master-proposed quest still need **interest** declared against candidate slots,
  or does it carry one date and take signups? All three major bots and westmarches.games use
  single-datetime-plus-RSVP (`03` §5b, §5c).
- If both paths exist, does the campaign's front page show one list or two?
- Does anything change about who may edit or withdraw a quest that a Game Master proposed?
  (Overlaps the roster ticket `05`.)

Read `research/03-practice.md` §5a and contradiction 1 first.

**Coupled to `15`.** Colvillian's carve-out — that player-proposed quests are exempt from the
recency cut — keys off exactly the distinction this ticket draws. The two are answerable in
either order, but probably want taking in one sitting.

## Answer

**Yes, and it is the same object.** Recorded in **ADR 0005** alongside the Rota, because the
two decisions are load-bearing for each other: it is precisely because proposal is open to
everyone that ticket `15` could not keep Colvillian's exemption for player-proposed quests.

### One quest, with the proposer recorded

Not Colvillian's two channels with two rule sets. A Game Master's proposal is a quest whose
proposer happens to be a Game Master, and the only difference is that **they can point it at
one of their own dates in the same act** — proposal and targeting collapse together, because
there is nobody to wait for. That is a shorter path through the same lifecycle, not a second
lifecycle. (This paragraph originally said the proposal "arrives with its slots already
attached", which ticket `06` corrected: under ADR 0006 a slot is never attached to a quest, it
is offered unattached and quests are pointed at it.)

`03` made the case and it is not close. Lutes measured **70% Game-Master-posted, 30%
player-initiated**. Contradiction 8 is blunter: campaigns die of *under-supply of initiative*,
and what saved every surviving account was the Game Master putting dates on the board
unprompted. A model where only players may propose would have Game Masters working around it
on day one — by asking a player to post for them, which is exactly what a workaround looks
like.

The counter-argument for two objects is Colvillian's, and it is real: they treat
`#dm-quest-posting` and `#player-quest-request` as different things with different rules. But
the *only* rule that actually differed was the recency exemption, and `15` rejected that. With
the exemption gone, two objects would buy a distinction that is one nullable column.

### Interest still works the same way

A Game-Master-proposed quest still collects **interest** against its candidate slots and is
still cut by the Rota. `03` §5b notes that all three major bots and westmarches.games use
single-datetime-plus-RSVP, and that shape is available here without a second model: **a
single-date quest is simply a quest with one candidate slot.** Its deadline is 72 hours
before it, standing is computed against it, and the Rota cuts at the deadline. Nothing
special is needed.

### What this changes

`CONTEXT.md`'s **Quest** no longer reads "proposed by a player", and **Game Master** now says
a Game Master may propose. ADR 0003's three acts are untouched: a Game Master proposing is
performing acts one and two together.

### Handed on

- **The front page: one list or two?** One model implies one list, but the layout question is
  the map's *campaign front page* item, which is already waiting on prototype `11`. Routed
  there rather than answered here.
- **Who may edit or withdraw a Game Master's quest?** The map's *permissions in detail* item
  already owns this, unblocked by `05`. Nothing about Game-Master proposal makes it a
  different question from the player-proposed case.
- **`11`** — a Game Master proposing a quest offers its slot in the same act, and under `05` a
  character is never a candidate for a slot its own player offered. So a Game Master who
  proposes a quest cannot play in it. That is right — they are running it — but it is the
  fourth cell state `10` asked for, arriving by a second route.
