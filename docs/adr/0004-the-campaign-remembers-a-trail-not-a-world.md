# The campaign remembers a trail, not a world

ADR 0002 concluded that "the **report** is the campaign's only durable record of the
world." That sentence overclaimed, in two ways this ADR corrects. It is amended here,
not superseded: the decision 0002 actually made — that a quest is a single expedition
and does not persist — stands untouched, and this ADR only replaces its consequences.

**A report is not a record of the world.** Ticket `07` established that a Game Master
may neither write a report nor correct one, following the founding text ("don't write
game summaries... if you do it, you'll just train them not to") and the stronger
convention that a Game Master "must never correct players' interpretations in these
reports or update the world to match them." What a report contains, therefore, is what
one party believes it saw. Nothing reconciles two reports that disagree, and nothing
is authoritative.

**Reports have holes.** Ticket `07` also decided that the obligation to write one lapses
after fourteen days, and that a quest may be terminal and permanently unreported.

Ticket `03`'s research sharpens both points. Per-quest reports do survive in real
campaigns, but only when they are paid for with an immediate in-game reward — which is
out of scope here — and what dies in every account is not the individual report but the
**cross-session synthesis**: the wiki, the lore index, the answer to "what do we know
about the Black Door." "Wikis tend to rot and not be read by players." "The participation
rate was low." "I'm not sure players would want to maintain it." Our reports were being
asked to carry more weight than any wiki that has survived one of these campaigns.

So the campaign remembers **a trail and a way back to it**, and nothing more:

- **The trail** is the reports, one per quest, in the players' own words — plus, where
  nobody wrote one, the quest's frozen **room**, which ticket `08` keeps forever, open
  to the whole campaign and searchable by everyone including players who joined later.
  A room is a poor record: unstructured, and it never produces the rumours and leads
  that feed the next quest. But it is not nothing, and it costs nothing, because it
  already exists.
- **The way back** is search. Ticket `08` gives reports their own index and their own
  result group above messages. Retrieval is our answer to synthesis, chosen precisely
  because it needs no maintenance — a search index rots at zero, and a curated one rots
  at the rate nobody updates it.

We considered building the synthesis — leads with state, a world wiki, an accumulated
picture of what is known. We rejected it as the one artefact this format has never
sustained. That is a choice, not an omission, and it is the most valuable thing in
ticket `03`'s research that we are declining to build.

## Consequences

- **The campaign has no world model.** No hexes, no locations, no accumulated state, and
  now no report standing in for one. The discovered world map stays out of scope, but on
  its own merits — a different product with spatial data and image handling — not because
  the report carries the world.
- **Search quality is load-bearing**, not a convenience. It is the whole of our answer to
  "what do we know about X".
- **A quest may name the quest it follows from.** One optional pointer, replacing ADR
  0002's "'same place, next week' has no first-class expression." Nothing in the system
  reasons about the chain: no inherited party, no inherited level range, no arcs, no
  "part 3 of 5". It is a pointer for people to read. The moment anything traverses it,
  we have rebuilt the durable quest that ADR 0002 rejected.
- **The gaps are permanent and visible.** An unreported quest reads "no report" for good.
  The record showing its own holes is preferred to a record that hides them.
- Implementation sessions will meet this gap and want to close it. That is the failure
  mode this ADR exists to prevent.
