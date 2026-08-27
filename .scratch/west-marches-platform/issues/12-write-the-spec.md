# Write the spec

Type: task
Status: resolved
Blocked by: 03, 04, 05, 06, 07, 08, 09, 10, 11, 14, 15

## Question

The destination. With every decision above resolved, write
`.scratch/west-marches-platform/spec.md`: complete enough that implementation sessions can
build from it without making further design decisions.

It needs to carry:

- The domain model and its lifecycle, in the vocabulary of `CONTEXT.md`
- The data model: entities, their fields, and the states they move through
- Screen by screen, what each one shows and who can reach it
- The notification matrix from ticket `09`
- What the fork changes in Campfire, and what it leaves alone
- Slices, ordered, so implementation has a first thing to build and a defensible stopping
  point short of the whole
- The non-goals from the map's Out of scope section, so nobody relitigates them

Fold the prototypes' outcomes in and then delete them. Update `CONTEXT.md` with any term
the spec needed and the glossary lacked.

## Answer

[`spec.md`](../spec.md) — the destination. Ten sections: the domain and its lifecycle, the data
model, the rules, screen by screen, the notification matrix, what the fork changes and what it
leaves alone, the slices, the non-goals, the decisions this spec makes that no ticket made, and a
provenance table back to every ADR and ticket.

**It settles four of the map's *Not yet specified* items**, because the spec could not be written
without them:

- **The campaign's front page** is **The Board** (§4.1), and it is load-bearing rather than
  pleasant: ADR 0007's constraint means every silence in the matrix has to be visible somewhere,
  and most of them land there. One list for Game-Master-proposed and player-proposed quests, since
  `14` made them one model.
- **Permissions in detail** (§3.10) — the full table, including the two the map named: who may edit
  or withdraw a quest that is not theirs, and who may remove someone's interest. The answer to the
  second is **nobody but the player**.
- **Player-to-player direct rooms stay** (§4.10). No ticket ever owned this and ADR 0007 depended
  on the answer. The Herald needs the model regardless, removal is work that buys nothing, and
  private lobbying cannot buy a seat from a binding rota — which is exactly why it is cheap here
  and would not be in a product where a human picks the party.
- **Character lifecycle** (§8) resolves to no lifecycle: name and level, edited freely, never
  deleted, no death or retirement state. A retired character is one nobody declares with.

**And it ratifies the two inventions the map was carrying unowned**: `11`'s **scribe derivation**
and its **contender ordering**. `07` does not disagree with the first, so it is not disowned — but
it is **amended twice**: the scribe is derived rather than named, and when attendance drops the
scribe the rule re-derives instead of asking a Game Master to name a replacement. §9 lists all
sixteen decisions the spec makes that no ticket made, so they can be argued with rather than
discovered.

**The prototypes are folded in and deleted** — `10`'s into §4.2, `11`'s into §4.5 — and both
tickets now say so where they linked to them.

**`CONTEXT.md` gained four entries and lost none**: **The Board**, **Candidate slot** and **Seat**
(all three used a dozen times in the spec with no definition behind them), plus **contenders** on
Confirmation. **Scribe** is amended to match the derivation above.

### What the spec deliberately leaves open

Named in §8 as limits rather than left silent, because each is a gap an implementation session will
meet and helpfully close: no world model, permanent holes in the trail, no reward for reports,
no level enforcement, and no relevance ranking in search. The map's remaining leftovers — a pacing
device for Game Masters, multi-Game-Master continuity, and listing pages at a year in — are
recorded there too, and none of them blocks a slice.
