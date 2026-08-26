# Does the report still carry the world?

Type: grilling
Status: resolved
Blocked by: -

## Question

ADR 0002 makes a quest a single expedition that does not persist, and justifies that by
saying the **report** carries the world instead. `CONTEXT.md` states it more strongly still:
the report is "the campaign's only durable record of the world".

Ticket `07` weakened that claim from two directions, and ticket `03`'s research argues it was
the riskiest sentence in the file to begin with.

- A **Game Master** may not write or edit a report, and may never correct a party's
  interpretation (`07`, citing Robbins and Domain of Many Things). So a report is the players'
  account of what they think they saw, not the campaign's authoritative world record. Does
  ADR 0002's justification survive that distinction, or was it relying on the stronger reading?
- Reports have **documented gaps**. `07` decided the obligation lapses at 14 days and a quest
  may be terminal and permanently unreported. What carries the world across a hole in the
  record?
- The research found that per-session reports survive when they are paid for, but that **the
  cross-session synthesis is what dies** — "wikis tend to rot and not be read by players"
  (Necropraxis), "the participation rate was low" (Alexandrian), "I'm not sure players would
  want to maintain it" (Mors Immortalis). Our reports carry more weight than any wiki that has
  survived one of these campaigns. Is asking a pile of per-quest reports to *be* the campaign's
  memory the same mistake with a different shape?
- If it isn't sufficient, what is the smallest thing that would be — and does it survive the
  out-of-scope ruling on the discovered world map?
- Does ADR 0002 need amending, superseding, or is the wording in `CONTEXT.md` the only thing
  that was ever overclaiming?

Read `research/03-practice.md` §4 and contradiction 9 first. Both `07` and `08` lean on the
ADR's current wording, so a change here reaches back into resolved tickets.

## Answer

**No — and ADR 0002's decision survives anyway.** The outcome is
[`docs/adr/0004-the-campaign-remembers-a-trail-not-a-world.md`](../../../docs/adr/0004-the-campaign-remembers-a-trail-not-a-world.md),
which amends ADR 0002 rather than superseding it.

**The decision was never the thing under threat; the justification was.** ADR 0002 argues
for the ephemeral quest on grounds that never mention reports — a 1:1 quest/session pair is
a sync bug, and a many-sessions quest raises questions (whose party? which level range now?)
the ephemeral shape does not ask. "The report is the campaign's only durable record of the
world" sits under **Consequences**; it was doing load-bearing work it was never argued for.
Robbins actively confirms the decision itself: parties who stayed out *"were required to get
another game scheduled with the same people and couldn't join other expeditions until they
got back."* So: amend the consequences, keep the decision.

**Two things the word "record" was hiding.** Ticket `03` separates them and they have
opposite fates:

- **The trail** — what happened, quest by quest. Reports do this, and real campaigns sustain
  them, but only when paid for with an immediate in-game reward, which `07` established is
  out of scope for us.
- **The synthesis** — "what do we know about the Black Door", cross-cutting and curated.
  This dies in *every* account. Necropraxis, the Alexandrian and Mors Immortalis all report
  the same wiki rot independently.

**We owe the trail and a way back to it. We do not owe synthesis.** Retrieval is the answer,
chosen precisely because it needs no maintenance: a search index rots at zero, a curated one
rots at the rate nobody updates it. `08` already gives reports their own FTS index and their
own result group. Building the synthesis would mean building the one artefact this format
has never sustained.

**The holes are smaller than they looked.** `07` accepted permanent gaps, but `08` keeps
every **room** forever — open to the whole campaign, searchable, including by mid-campaign
joiners. An unreported quest still has its planning, its argument and usually its after-action
chatter. That is a genuinely worse record — unstructured, and `03` is clear that chat logs
never produce the leads that feed the next quest — but it is a record, and it costs nothing
because it already exists. ADR 0004 names it as the fallback.

**A quest may now name the quest it follows from.** ADR 0002 flagged this itself as
"reversible if it chafes", and once the trail is the only structure the campaign has, a trail
with no threads through it is a pile. One optional pointer, to *any* quest including a
declined or abandoned one — "we proposed going back to the Tomb, nobody bit, and three months
later someone did" is the thread worth having. Stored one-way, displayed both ways.

The guard is the whole of the decision: **nothing in the system traverses the chain.** No
inherited party, no inherited level range, no arcs, no "part 3 of 5". It is a pointer for
people to read. The moment anything reasons about it we have rebuilt the durable quest ADR
0002 rejected. It is described inside `CONTEXT.md`'s **Quest** entry rather than given a
glossary term of its own, deliberately — `03` warns that naming a mechanism is how it becomes
real, and this one must not.

### What was declined

- **Leads with state** — taken up, resolved, dead. This is the highest-value thing in `03`'s
  research (*"the part that feeds the next quest, and the part a plain chat log never
  produces"*), and it is a lead somebody must maintain, which is the exact rot pattern. A
  campaign-wide "open leads" view looks like a cheap middle but collapses into the same thing:
  you cannot assemble a view from free-text headings without structuring them. Recorded on the
  map as out of scope and as the strongest candidate for a future effort with its own
  destination.
- **Superseding ADR 0002.** Its decision is untouched.
- **Editing ADR 0002's consequences in place.** ADRs are a log; rewriting one hides that the
  thinking moved. It carries "Amended by ADR 0004" pointers instead.

### Consequences elsewhere

- **The map's out-of-scope entry for the discovered world map** loses its "the report carries
  the world instead" clause and now stands on its own merits, which were always the real
  reason. The campaign has **no world model**, recorded as a deliberate limitation.
- **Search quality became load-bearing.** Nothing in `08` changes — the index, the grouping
  and the immortal rooms all stand — but `last(100)` with no ranking (`app/controllers/searches_controller.rb:23`)
  was sized for search-as-convenience. Ranking, paging and scoping now need an owner; recorded
  on the map. `geared_pagination` is already in the `Gemfile`.
- **Ticket `12` must state the limitation explicitly.** The audience for "there is no world
  model" is not players — it is the implementation sessions that will meet the gap and
  helpfully build a wiki to close it. Written down it is a decision; unwritten it is an
  oversight someone will correct.
