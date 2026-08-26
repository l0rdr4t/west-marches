# Who cuts the party — the system, or whoever proposed the quest?

Type: grilling
Status: open
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
