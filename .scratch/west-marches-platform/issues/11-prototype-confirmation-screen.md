# Prototype: the Game Master's confirmation screen

Type: prototype
Status: open
Blocked by: 06, 10

## Question

Confirmation is the single moment a quest becomes real (glossary; ADR 0003). The Game Master
has to make one judgement while holding a lot in view, and if this screen is bad they will
go back to doing it in chat.

Build a rough, throwaway prototype of what the Game Master sees:

- Which of their slots each interested character can make, and therefore what party each
  slot would produce
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

Read ticket `06` for the lifecycle, ticket `10` for the player-side vocabulary this has to
agree with, and ticket `15` for what confirmation can and cannot do. `10` recommends starting
from its variant A — the matrix with rows in rota order and the column footer that prices each
slot in parties ("4 seated · 1 seat short") — which is now the whole of the decision aid.
