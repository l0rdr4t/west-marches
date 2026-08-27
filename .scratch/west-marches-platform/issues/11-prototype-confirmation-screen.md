# Prototype: the Game Master's confirmation screen

Type: prototype
Status: resolved
Blocked by: -

## Question

Confirmation is the single moment a quest becomes real (glossary; ADR 0003). The Game Master
has to make one judgement while holding a lot in view, and if this screen is bad they will
go back to doing it in chat.

Build a rough, throwaway prototype of what the Game Master sees:

- Which of their slots each interested character can make, and therefore what party each
  slot would produce
- **Added by `06`: this screen has two axes, not one.** Under ADR 0006 a slot is offered
  unattached and several quests may target the same date, so a Game Master confirming is
  choosing *which quest* as well as *which slot*. `10`'s column footer ("4 seated · 1 seat
  short") prices one axis only. What the other axis looks like — three quests competing for
  one Friday — has never been drawn.
- The standing order — who has played least recently — so the consequences of picking a
  slot are visible before picking it
- The moment of confirmation, and what it visibly spends: the slot, and the other quests
  that were counting on it
- ~~Overriding the automatic cut~~ — **struck by `15`.** There is no override: ADR 0005 makes
  the rota binding, and `Q13`, cited above, turned out to be uncitable. The Game Master's only
  judgement is **which slot**, which makes this screen a slot-picker rather than a party-editor.
- **Slot choice is therefore party engineering** (`15`). A Game Master who wants a different
  party has to pick a different date. That is oblique and a little perverse, and people will
  use it that way — design knowing it.
- ~~Out-of-range characters, flagged for their attention~~ — **struck by `15`.** The platform
  no longer compares a character's level to a quest's level range at all. Both are text.
- What they do when no slot produces a decent party

## Findings

Prototype: [`prototypes/11-confirmation-screen/confirmation-screen.html`](../prototypes/11-confirmation-screen/confirmation-screen.html).
One self-contained file, no build. Open it and use the floating bar: `←`/`→` (or the arrow keys)
cycle three variants, `clock` moves the day between 2 Sep and 10 Sep, `width` swaps in a 390px
phone frame. Confirming is live and mutates the board — reload to reset.

- **A · The Grid** — your dates down the side, the quests targeting them across the top; each cell
  is the party that pairing would produce. Refuses to pick an axis.
- **B · By Date** — one card per date you offered, the competing quests side by side inside it.
- **C · By Quest** — one card per quest, the dates it could run on inside it, ordered by which
  dies soonest.

Fake data: four dates of yours plus one of Nia Osei's, four quests, and Friday wanted by three of
them — including one whose only date in the world it is.

### B is the screen. The date is the object, not the pairing and not the quest

`10` recommended starting from its variant A, and that recommendation is **superseded rather
than wrong**: it was written before ADR 0006 made slots a pool, and it was answering a
one-axis question. Once several quests may target one date, A's grid is dates × quests, and
two things break.

**The grid is mostly empty.** Four dates against four quests is twenty cells, of which nine are
real pairings. The rest are dots. A cross-tab whose job is comparison spends half its space
saying "these two have nothing to do with each other", and it gets worse as the campaign grows,
because quests target two or three dates each however many dates exist.

**The grid cannot hold the rota.** The cell has room for the price — `5 seated · 1 waiting` —
and nothing else, so the evidence has to go in a drawer under the table. The decision and the
reason for it are then never on screen together, which is the one thing ADR 0005 says this
interface owes the people it cuts. B has the rota inside the contender, a click away and in
place.

**C leads with the wrong pressure.** Ordering quests by which dies soonest is the abandonment
clock driving the page, and it answers "what am I about to lose", not "what am I running on
Friday". A Game Master arrives at this screen because they have an evening, not because a
quest is expiring. But **C's clock is worth lifting into B** — B currently flags only "this is
its only live date", which is the sharp case and not the general one.

### The two-axis question needed no second axis

ADR 0006's consequence — a Game Master chooses which quest as well as which slot — reads like
it demands a matrix. It does not. **Put the contenders side by side inside the date's card**
and the second axis becomes a row of two or three cards, each priced in the party it would
produce. Three quests fighting over one Friday fits on a phone, because it is three cards and
not a column.

The reason it works is that the two axes are not symmetrical. The date is scarce and the Game
Master owns it; the quest is what the campaign is offering for it. Drawing them as equals, which
is what a grid does, misrepresents the decision.

### The scribe is derived, not chosen — and the picker was the tell

All three variants shipped with a scribe picker in the confirmation dialog, because `07` says
confirmation names a scribe and never says who names it. **It is struck.** It was the one
control on the screen that let a Game Master exercise judgement over a named person, which is
exactly the thing ADR 0005 removed from the party and did not intend to re-admit through the
report. A Game Master who cannot choose who plays should not be able to choose who writes.

The rule invented in its place: **the report is owed by whoever in the party has gone longest
without writing one, never-scribed first, ties broken by rota position.** Same shape as the
rota — derivable from reports already kept, binding, and checkable — and it spreads a debt that
`07`'s research says fails when it is diffuse and fails again when it always lands on the same
person.

Deliberately **not** the rota itself. In the prototype's Friday party the scribe is Kestrel
Ashdown, 4th of 5, while the rota's first seat went to a player with three bumps. If the two
were one rule, finally getting a seat after being refused three times would come with a chore
attached, and the rota would be handing out compensation and homework in the same act.

This is an invention on the scale of `10`'s `interestCloses` and it needs ratifying. It is
`12`'s to write down, and `07`'s to disown if it disagrees.

### Sorting the contenders by party size is accepted, and here is what it costs

The card orders competing quests by the size of the party they would produce. That is the Game
Master's number, and `10` warned specifically against leading with the Game Master's number —
so the cost is named rather than avoided: **on the prototype's Friday it sinks "Bring the horses
back" to the bottom**, at 3 of 5, on the one date in the world it has. The thing that stops that
being a trap is not the ordering but the flag underneath it — *this is its only live date* —
which is doing the work the sort order isn't. Keep both.

### Smaller things the prototype settled or exposed

- **A Game Master's board is mostly not actionable, and that is the deadline working.** At the
  prototype's opening clock, one of four dates can be confirmed: one is open, one closes in two
  hours, two are more than a week out. The screen cannot get away with a disabled button — it
  has to say *interest closes in 8 days* per date, or the hard gate reads as a bug. This is the
  most surprising rule on the screen and the one most likely to be worked around if it is not
  explained where it bites.
- **Under-subscription is the common case here too.** Three of the nine real pairings produce
  less than a full party, and one produces two characters against five seats. `10` found the
  same on the player's side. "3 of 5 · 2 seats unfilled" as a neutral price, never as a failure.
- **What they do when no slot produces a decent party is nothing, and the screen should say so
  plainly.** `What is under the mill` can seat two. There is no minimum (ADR 0006), so the Game
  Master may run it, and the only other moves are to wait or to offer another date. The screen
  offers no third thing because there is no third thing.
- **The fourth cell state `10` asked for is not a cell.** A character is never a candidate for a
  slot its own player offered, and in B that is a sentence under the rota — *Vetch can make this
  date but is not in the running for it — that player offered it* — not a grid state. It reads
  better as prose because it is the only exclusion that is not about availability, and a fourth
  dot would have made it look like one.
- **Your own character on someone else's date.** The Game Master's character is excluded from
  their own Friday and seated on Nia Osei's Saturday, in the same view. Worth keeping in any
  future fake data: it is the clearest possible statement that Game Master is not a role you
  hold instead of playing.
- **Other Game Masters' dates belong on this screen, greyed.** A quest that can run on somebody
  else's Saturday is a reason not to spend your Friday on it. B shows them as a card you cannot
  confirm into, labelled with whose it is.
- **The confirmation dialog is the notification matrix in miniature.** Enumerating what
  confirmation spends — seats, a frozen waitlist, bumps to everyone who could not make the
  date, and the losing quests — produced a list of four or five distinct things that happen to
  named people at one instant. That list is `09`'s input, and writing it here was easier than
  writing it there will be.

### Handed on

- **`09` — confirmation is the loudest single moment in the system.** One act seats five people,
  waitlists one, bumps two, and takes a date away from two other quests full of people who were
  counting on it. The dialog names all of them; the notification matrix has to decide which of
  them get told and how.
- **`09` — the losing quests' declarers.** ADR 0005 is explicit that losing the date is not being
  refused and deals no bump. Whether it deals a *notification* is `09`'s, and the prototype's
  Friday makes the case: three people declared on the horses quest, and if that quest quietly
  loses its only date they find out by looking.
- **`12` — the contender ordering is a rule and needs writing down.** "Best party first" is
  implemented here and accepted, but it is not in any ADR.
- **`12` — the scribe rule above**, which is invented and unratified.
- **The campaign's front page** (map, *Not yet specified*) is now unblocked: `11` was the last
  thing it waited on. Note that this screen is *not* it — it is the Game-Master surface, reached
  from "You run" in the sidebar, and it deliberately shows only dates the viewer offered.

### Vocabulary

`CONTEXT.md`'s **Scribe** and **Confirmation** both said the scribe is *named* at confirmation,
which was written when a human did the naming. Both amended: the scribe is **derived**, by the
rule above, and no one chooses it.

Read ticket `06` for the lifecycle, ticket `10` for the player-side vocabulary this has to
agree with, and ticket `15` for what confirmation can and cannot do. `10` recommends starting
from its variant A — the matrix with rows in rota order and the column footer that prices each
slot in parties ("4 seated · 1 seat short") — which is now the whole of the decision aid.
