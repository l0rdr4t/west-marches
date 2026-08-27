# The Rota cuts every party, and a seat is earned by being refused

ADR 0003 decided that a Game Master offers dated **slots**, that interested players
declare which of them they could make, and that one **confirmation** binds the quest to a
slot and cuts the party. That decision stands and this ADR does not touch it. What it
replaces is the rule 0003 cut *by* — "whoever played least recently" — along with two
things 0003 left implicit: that the cut might be overridden, and that it could happen at
any moment.

**The order has a name: the Rota.** Ticket `03` looked for an established term and found
none. The three real implementations call it three different things — "prioritize players
that have played less frequently" (Colvillian), "the Waiting List" (the Alexandrian), and
a literal `PriorityQueue` (the SeedRoll bot). Shipping this means naming it, and an
unnamed rule cannot be explained to the person it disadvantages.

**A seat is earned by being refused, not by being absent.** This is the correction. The
most-cited treatment of open-table seat allocation is the Alexandrian's, and its mechanism
is not recency:

> To prevent highly active players from monopolizing the slots at every session, I also
> instituted the concept of the Waiting List: If you signed up for a session, but couldn't
> play because of the session cap, then you were given preferential placement for the next
> session.

Shiroiken arrives at the same rule independently on EN World — "assuming the alternate
doesn't get the play, they automatically take the #1 slot of the next game they want to
sign up for." Priority is earned by a *specific disappointment*. Under pure recency, the
player who has never declared interest outranks the player who has been bumped twice,
which rewards lurking and punishes turning up to be turned away.

So the Rota orders by, in order:

1. **Bumps** since you last had a seat, most first.
2. **How long** since you last had a seat; never-seated sorts as infinitely long.
3. **Who declared first.**

A **bump** is recorded when the slot you declared for is confirmed and you were not seated.
(**Sharpened by ADR 0006:** seated or not *in the quest you declared on*. Several quests may
target one date, so a quest that merely loses the date to another quest deals its declarers no
bump — nobody refused them.)
It needs no new state — it is derivable from interests, confirmations and parties we
already keep — and two edges fall out of the definition rather than needing rules of their
own. A quest that dies unconfirmed bumps nobody, because nobody refused you. Declaring for
a quest that is confirmed into a slot you *couldn't make* is not a bump either, because you
were never in the running for that slot.

This also settles the loose end ADR 0003 left in its own consequences: "played least
recently needs a definition that survives a player's first quest and a player who was on
the waitlist. Both sort as never-played." Under the Rota they do not. The waitlisted player
carries a bump; the first-timer does not.

**The Rota is binding, and it applies to every quest.** Colvillian — the only campaign
found that wrote a recency rule down — exempts player-proposed quests: "Player Quests do not
require prioritizing people who have played less frequently." Robbins says the same of party
selection generally: "Picking a team is their job, not yours." We are rejecting both, for a
reason specific to our model rather than to theirs.

This ADR also settles who may propose. `CONTEXT.md` had it that a quest is "proposed by a
player", and `03` found that this is the minority path everywhere it looked: Lutes measured
**70% Game-Master-posted, 30% player-initiated**, and contradiction 8 is blunter still —
campaigns die of *under-supply of initiative*, and what saved every surviving account was
the Game Master putting dates on the board unprompted. So **a Game Master may propose a
quest**, and it is the same object with the proposer recorded, not Colvillian's second
channel with its own rules. A Game Master's proposal simply arrives with its slots already
attached, collapsing proposal and slot-offering into one act.

That is what makes the carve-out untenable. With proposal available to everyone it becomes
trivially gameable: any player who wanted to hand-pick their friends would propose rather
than declare, and the exemption would swallow the rule. Worse, `03`'s distribution is steep
(half a roster never really plays; a third takes most of the seats), and the people who
propose quests are the same people who already take the seats. Colvillian could carry that
carve-out because a human Dungeon Master exercised judgement inside it; automating it keeps
the loophole and discards the judgement.

For the same reason there is **no Game Master override at confirmation**. A Game Master's
one judgement is *which slot*. What a human can still do is record the truth afterwards:
**attendance** already lets a Game Master add a waitlist character who came along anyway.
The system cuts, the table does what tables do, and attendance says what happened.

**The Rota needs a deadline, and the deadline belongs to the slot.** Every recency or
rotation system found in the wild has an explicit close: Colvillian has a `Signup Deadline`
field and lists players only once signups close; EricVulgaris forbids confirming until a
block's retrospective. Without one, seats fill first-come and the Rota has nothing to rank.
Each slot therefore closes **72 hours before it starts**, set by the Game Master who offered
it, and a slot cannot be confirmed until its own deadline has passed. The gate is hard: a
soft one is no gate at all, because the first Game Master to confirm early has restored
first-come for everybody.

Deadlines are **per slot** rather than per quest for the same reason standing is: a quest
offering the 4th and the 23rd is two different questions, and one deadline pinned to the
earlier date would close the later one twenty days before anybody had to decide.

**The Rota shows its working.** The interested roster is public while interest is open,
each row carries the reason it is where it is, and the cut line is drawn per slot. Colvillian
hides its list, explicitly "so that players are not discouraged from signing up", and we are
going the other way on two grounds. A binding automatic cut has no appeal and no appeal
process, so being checkable by the person it disadvantages is the only accountability it has.
And Colvillian's fear does not survive the Rota: their bumped player got nothing for
declaring, so discouragement was pure loss, whereas here being passed over is exactly what
buys the next seat. That is a sentence they could not write and we can.

**The Rota cannot see your character.** Level is not in the order, and an interest names one
character rather than a stable. Contradiction 12 argued the cut should see it — uneven
attendance produces level spread and modern players read spread as unfairness — and we are
declining, because level is not something this platform authoritatively knows. It is a
human-entered number about a sheet that lives at the table, which ADR 0001 puts out of scope,
and a stale number driving an automatic decision is worse than no number at all.

## Consequences

- **The rule is `CONTEXT.md`'s Rota, and a bump is its unit.** Both are new vocabulary and
  both have to be used consistently, in the interface as well as in the spec.
- **A seat spends your standing whether or not you walk in.** This is what stops a player
  taking seats and never appearing: under an attendance-based rule they would never reset,
  never accrue a bump, and sort high forever. It buys the incentive a coordination tool
  actually wants — *withdraw before the deadline and you keep your place; no-show and you
  have spent it* — and it costs a player who genuinely falls ill their turn. Accepted.
  **Restated by ADR 0006:** the free window runs to the start of play, not to the deadline —
  **the seat is spent unless you give it back before it is used.** The original sentence left
  the gap between the deadline and the session unaddressed, and closing it strictly would have
  made announcing cost the same as ghosting.
  **Attendance no longer feeds the Rota.** It keeps its other job, deciding who may edit
  the report.
- **The waitlist is the Rota continuing past the cut line, frozen at confirmation.** Not
  FIFO, which is what Sesh, rpg-schedule and westmarches.games all do, and not recomputed
  later. Frozen is the load-bearing half: "first seat to free up is yours" is a lie if the
  list re-sorts underneath the person it was promised to.
- **A Game Master who wants a different party has to offer or pick a different date.** Slot
  choice is now the only lever on who plays, which makes it quietly into party engineering.
  The confirmation screen should be designed knowing that.
- **Robbins' counter-lever is inexpressible.** "You can require parties to stay together for
  two adventures" has no home in a model where every party is cut fresh by a binding rule.
  A campaign that wants it will do it by hand, at the table.
- **An out-of-range character can take a seat and nothing in the app can stop it.** Level is
  never compared, the cut is blind to it, and there is no override. This is chosen, not
  overlooked.
- **A cancelled quest must not spend anybody's standing**, or a Game Master calling off a
  session silently costs five people their turn. The quest lifecycle owes this.
- **Nothing here distinguishes a Game-Master-proposed quest from a player-proposed one.**
  One model, one Rota, one lifecycle. A single-date quest is simply a quest with one
  candidate slot.
