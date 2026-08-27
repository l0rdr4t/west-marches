# Who cuts the party — the system, or whoever proposed the quest?

Type: grilling
Status: resolved
Blocked by: -

## Question

ADR 0003 has the system cut the party from the interested characters, ranked by **who played
least recently**, on every quest. Ticket `03` calls the conflict with real practice "the
sharpest in this document", and it arrives from four directions at once.

**The one documented recency rule exempts player-proposed quests.** Colvillian — the only
campaign found that wrote a recency rule down — says "**Player Quests do not require
prioritizing people who have played less frequently**", because the person who organised the
expedition picks who comes. Robbins agrees in general: "Picking a team is their job, not
yours." Our **Confirmation** overrides the proposer on quests the proposer created.

- Does the proposer ever pick the party instead of the system? If yes, on which quests — and
  does that reintroduce the clique problem ADR 0003 rejected first-come to avoid?
- If a Game Master can propose (ticket `14`), does a Game-Master-proposed quest get the
  automatic cut while a player-proposed one doesn't?

**"Played least recently" is not what the traditional fix optimises for.** The attested
mechanism is **first-come plus waitlist carry-over** (the Alexandrian's Waiting List,
reinvented independently on EN World): priority is earned by *trying and being bumped*, not by
absence. It needs no database and no explanation.

- Under pure recency, a player who has never declared interest outranks one who has been
  bumped twice. Is that the intended behaviour?
- Should the ranking be by *unrewarded interest* — declared and not seated — before raw
  recency? Or a blend?
- ADR 0003's consequence "played least recently needs a definition that survives a player's
  first quest and a player who was on the waitlist. Both sort as never-played" is a symptom of
  this. Carry-over answers it directly.

**The cut needs a deadline, and we have not specified one.** Every recency-or-rotation system
in the wild has an explicit signup deadline — Colvillian has a `Signup Deadline` field;
EricVulgaris forbids confirming until a block's retrospective. Without one, seats fill
first-come and the recency rule has nothing to rank.

- When does interest close? Is Confirmation only possible after it does?

**Visibility of interest is undecided, and the two sources disagree with each other.**
Colvillian hides the interest list until the deadline "**so that players are not discouraged
from signing up**". ADR 0003's own consequences require the opposite: "players must be able to
see their own standing before confirmation ('3rd of 8, party of 5'), or the cut reads as
arbitrary."

- Can both be true? Show me my own standing but not the roster? Show counts but not names?
- If a quest displays "6 interested, 4 seats", does the seventh player stop declaring — and
  does the recency cut then have nothing to rank?

**Two smaller ones that belong here:**

- **Waitlist ordering.** Sesh, rpg-schedule and westmarches.games all promote strict FIFO. If
  ours is ordered by recency, that is a second novel rule to teach, and it makes "someone
  dropped out, who gets the seat?" non-obvious in the moment.
- **Which character you bring.** Uneven attendance produces level spread, and the countermeasure
  real groups use is character stables — bring the character closest in level. **Interest**
  already names a character; the cut does not currently see it. Should it?

**And a naming problem.** `03` found **no established term** for "cut the party by who played
least recently" — three implementations call it three different things ("prioritize players
that have played less frequently", "the Waiting List", a literal `PriorityQueue`). If we ship
it we are naming it, and `CONTEXT.md` has no word for it yet.

Read `research/03-practice.md` §2 in full, plus contradictions 2, 3, 4, 5, 7 and 12. This
ticket challenges ADR 0003 the way `13` challenged ADR 0002, and may end in an ADR 0005.

**Coupled to `14`.** Answerable in either order; probably want taking in one sitting.

## Answer

**ADR 0005** — *The Rota cuts every party, and a seat is earned by being refused*. It amends
ADR 0003, which is retitled: 0003's three acts stand, its ranking does not.

### The system cuts, on every quest, and nobody overrides it

The proposer never picks the party. Colvillian's carve-out — "Player Quests do not require
prioritizing people who have played less frequently" — and Robbins' "picking a team is their
job, not yours" are both rejected, for a reason that is ours rather than theirs: ticket `14`
resolved in the same sitting that **a Game Master may propose**, so proposal is open to
everyone, and an exemption keyed to who proposed is trivially gameable. Any player wanting to
hand-pick their friends would propose rather than declare, and the exemption would swallow
the rule. `03`'s distribution makes it worse — a third of the roster takes most of the seats,
and those are the same people who propose. Colvillian could carry the carve-out because a
human Dungeon Master exercised judgement inside it; automating it keeps the loophole and
discards the judgement.

There is also **no Game Master override at confirmation**, which is stricter than ADR 0003
was. Note that `Q13` — cited by ticket `11` as having settled that a Game Master "can reorder
or remove" — **cannot be verified**: no `Q`-series document exists in this repo or in its
history, the numbers survive only as citations in tickets written after them, and
`CONTEXT.md`'s Confirmation entry never mentioned an override. It was treated as open and is
now closed the other way.

What survives is that a Game Master picks **which slot**, and that **attendance** still lets
them record who actually went. The system cuts, the table does what tables do, and attendance
says what happened.

### The Rota: bumps, then recency

Ranked by bumps since your last seat, then how long since that seat (never seated sorts as
longest), then who declared first.

This is contradiction 3, conceded. Pure recency optimises for absence; the attested mechanism
— the Alexandrian's Waiting List, reinvented independently by Shiroiken — optimises for
**being refused**. Under pure recency a player who has never declared outranks one bumped
twice, which rewards lurking and punishes turning up to be turned away.

A **bump** needs no new state: it is derivable from interests, confirmations and parties. Two
edges fall out of the definition rather than needing rules — an unconfirmed quest bumps
nobody, and a slot you could not have made cannot bump you, because you were never in the
running for it.

It also closes ADR 0003's own loose end. "Both sort as never-played" is a symptom of the
wrong ranking: under the Rota the waitlisted player carries a bump and the first-timer does
not.

We kept recency as the second key rather than going to pure first-come-plus-carry-over,
because the platform *knows* when you last played. Colvillian had to ask players to
self-report it ("include the date of the last game you played"). That is the one real
advantage we have over every campaign in the research, and throwing it away for legibility
would be a poor trade.

### Everything is per candidate slot

Standing, bumps, deadlines and the cut line. This is `10`'s finding carried to its
conclusion: a quest offering the 4th and the 23rd is two different questions.

**Each slot closes 72 hours before it starts**, set by the Game Master who offered it — the
slot's owner, not the quest's proposer, since on a player-proposed quest the dates are not
the proposer's to close. **Confirmation into a slot is hard-gated on that slot's deadline.**
A soft gate is no gate: the first Game Master to confirm early restores first-come for
everybody. This is contradiction 5, and `10`'s invented `interestCloses`, both answered.

### It shows its working

Full disclosure while interest is open: the roster, the reason on every row, and the
provisional cut line per slot — `10`'s variant B with A's recency line lifted in.

Contradiction 4 pits Colvillian ("not list players that have signed up until the close date,
so that players are not discouraged from signing up") against ADR 0003. Both arguments
resolve the same way here. A binding automatic cut has no appeal, so being checkable by the
person it disadvantages is the only accountability it has — `10`'s "the reason printed on
every row is what stops the ranking being the platform's opinion". And Colvillian's fear does
not survive the Rota: their bumped player got *nothing* for declaring, so discouragement was
pure loss. Here being passed over is exactly what buys the next seat, and the page can say
so.

### The seat spends your standing, not attendance

This overturns `CONTEXT.md`'s claim that attendance "is what 'played least recently' counts".
Under an attendance-based rule a player who takes seats and never appears never resets, never
accrues a bump, and sorts high forever; nothing in the system notices, and the attested fix is
a human removing them (Shiroiken). Making the **seat** the currency closes the leak with no
new state and buys the incentive the tool actually wants: **withdraw before the deadline and
you keep your place; no-show and you have spent it.** It costs a player who genuinely falls
ill their turn. Accepted.

### The waitlist is the Rota continuing, frozen

Not FIFO — contradiction 7 notes Sesh, rpg-schedule and westmarches.games are all
first-in-line, but two orderings on one screen is two rules to teach, and `10`'s variant B
already numbers the waitlist continuously. Frozen at confirmation is the load-bearing half:
"first seat to free up is yours" is a lie if the list re-sorts underneath the person it was
promised to.

### The Rota cannot see your character

Contradiction 12 declined. Level is not in the order and an interest names one character, not
a stable. Then the level question was reopened and went further: **the platform models level
as text and never compares it.** The research is one-sided on this — Colvillian *prints*
`Recommended Level: 13-24` on the quest post for humans to read, and westmarches.games, the
only tool that models level in software, makes it a hard **requirement**. Advisory-but-computed
is attested nowhere; we invented it. And a human-entered number about a sheet that lives at
the table (out of scope since ADR 0001) goes stale, so the one thing it did, it would
eventually do wrongly.

This deletes the advisory flag, and with it the map's unowned question of whether a level
mismatch is public — there is no computed mismatch left to publish.

### Naming

**Rota**, added to `CONTEXT.md` along with **Bump**. `03` found no established term — three
implementations, three names — so shipping it meant naming it. Rejected: *Precedence* (exact
but reads as a legal register), *the Roll* (collides with dice). "Rota" says rotation without
implying a fixed order of turns, and the glossary's `_Avoid_` list already ruled out rank,
priority, score and queue position.

### Rejected, with reasons worth keeping

- **A carve-out for player-proposed quests.** The one documented recency rule has it, and it
  cannot survive a model where anyone may propose. See above.
- **A Game Master override with friction.** Preferred during the grilling and voted down. The
  argument for it was Robbins' counter-lever and the out-of-range seat; the argument against
  is that a visible-but-available override becomes habit, and that an automatic cut nobody can
  bend is the only version whose fairness is checkable.
- **Interest naming a stable of characters**, with the system seating whichever is closest to
  the level range. This was contradiction 12's real countermeasure and it fits "standing
  follows you, not them" — but it was overtaken by dropping level computation entirely.
- **A lookahead cap** ("you can't sign up beyond the next session you're in"). Dissolved: the
  Rota resets you to zero bumps on being seated, which forces the same rotation without a
  second rule.

### Handed on

- **`06` — a cancelled quest must not spend anybody's standing.** Under Q9 the seat is the
  currency, so a Game Master calling off a session would silently cost five people their turn.
  This is now load-bearing for the fairness rule, not just a lifecycle nicety.
- **`06` — stale interest on distant slots.** Per-slot deadlines mean someone can tick a date
  seven weeks out and be seated on it having forgotten. Withdrawal before the deadline is free
  (Q9); what happens *after* it is `06`'s.
- **`11` — two of its bullets are moot.** "Overriding the automatic cut" no longer exists, and
  "out-of-range characters, flagged for their attention" no longer exists either. What is left
  is a slot-picker: the Game Master's only judgement is which date, and `10`'s column footer
  ("4 seated · 1 seat short") is the decision aid.
- **`11` — slot choice is now party engineering.** With no override, a Game Master who wants a
  different party picks a different date. That is oblique and slightly perverse, and the screen
  should be designed knowing people will use it that way.
- **`05` — dissolved, not deferred.** "Does a departed player's past attendance still weigh on
  other players' standing?" No: the Rota's sort key is entirely individual — your bumps, your
  last seat — so nobody's history moves anybody else's position.
- **The character-lifecycle ticket shrinks.** Level no longer drives anything, so "who sets a
  character's level, and when" stops being load-bearing. Death and retirement remain.

### Vocabulary

Added **Rota** and **Bump**. Rewrote **Attendance** (it no longer feeds the ranking),
**Waitlist** (the Rota continuing, frozen), **Confirmation** (deadline-gated, no override),
**Slot** (carries its own deadline), **Level range** and **Character** (level is a note, never
compared), **Quest** and **Game Master** (a Game Master may propose), and **Standing** (read
off the Rota).
