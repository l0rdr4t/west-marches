# Prototype: the Game Master's confirmation screen

Type: prototype
Status: open
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

## Prototype

Built, awaiting reaction — no findings yet.
[`prototypes/11-confirmation-screen/confirmation-screen.html`](../prototypes/11-confirmation-screen/confirmation-screen.html).
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

Read ticket `06` for the lifecycle, ticket `10` for the player-side vocabulary this has to
agree with, and ticket `15` for what confirmation can and cannot do. `10` recommends starting
from its variant A — the matrix with rows in rota order and the column footer that prices each
slot in parties ("4 seated · 1 seat short") — which is now the whole of the decision aid.
