# How do West Marches groups coordinate today, and what breaks?

Type: research
Status: resolved
Blocked by: -

## Question

We are building a coordination tool for a format that already has established practice.
Find out what that practice is so the spec is arguing with reality rather than with my
assumptions.

- What tools do real West Marches campaigns use for proposing expeditions and scheduling
  them? (Discord plus bots, spreadsheets, Doodle/When2meet, Foundry, Obsidian, forum
  threads.)
- What specifically breaks or frustrates people at roster sizes above one table? Look for
  first-hand accounts from Game Masters running these campaigns.
- How do those groups handle the fairness problem — the same handful of players taking
  every seat? Is "least recently played" a practice anyone uses, and what do they call it?
- What conventions exist around session reports, and what makes people actually write them?
- Are there existing purpose-built tools, and where do they stop?

Use high-trust primary sources: Ben Robbins' original West Marches posts, Game Master
write-ups, and the tools' own docs. Note where you are reporting one group's practice
rather than a general convention.

Deliverable: findings written to
`.scratch/west-marches-platform/research/03-practice.md`.

## Answer

Findings: [`research/03-practice.md`](../research/03-practice.md), ~870 lines, every claim
carrying a URL and a confidence label — **[FOUNDING]** (Robbins' own posts),
**[CONVERGENT]** (several independent Game Masters agree), **[SINGLE]** (one group; an
existence proof, not a norm).

**The founding text does not contain our design.** Robbins' West Marches has *no
party-cutting rule at all*. Scheduling is one email carrying destination, date and
recruitment call together — no separate propose, offer-slot or declare-interest step — and
"it's a chaotic process". His only fairness levers are party-composition rules that point in
*opposite* directions ("require parties to mix after two adventures" vs "require parties to
stay together for two adventures"), chosen by which failure you actually have. He does
validate two things we assumed: quests are played once and unfinished business must be
rescheduled as a new expedition, and the Game Master must never write the summaries.

**The real-world fairness mechanism is first-come plus waitlist carry-over, not recency.**
The Alexandrian's Waiting List gives priority to whoever *tried and got bumped*. It is earned
by a specific disappointment rather than by absence, needs no database, and — unlike recency
— does not reward lurking. Recency priority does exist: the Colvillian West Marches wrote it
down in 2017 ("prioritize players that have played less frequently"; DMs "accept players
based on regency"), and one Discord bot implements a literal least-recently-played
`PriorityQueue`. But Colvillian's version comes with a **signup deadline**, a **hidden
interest list** ("so that players are not discouraged from signing up"), **self-reported**
recency, and an **exemption for player-proposed quests**, where the proposer picks their own
party. We have none of those four.

**Scheduling in practice inverts our direction.** Six independent Game Masters post a slate
of dates that players claim, rather than players proposing and GMs offering. Lutes measured
**70% GM-posted, 30% player-initiated**; Colvillian runs two separate channels and treats
them as different objects with different rules. Candidate-slot polling across dates is rarer
than we assumed — all three major bots and westmarches.games use a single datetime plus RSVP.

**Scheduling breaks *below* about nine engaged players, not above one table.** The ticket
asked what breaks at large roster sizes; the evidence says the failure is under-supply of
initiative. "Player scheduling was FAIL... No one wanted to be the guy who said fuck it,
we're playing Sat." "It died by scheduling." What fixed it in every surviving account was the
GM publishing dates unprompted — which makes "a Game Master keeps slots on the board" the
critical path, not a nicety.

**Reports survive when they are paid for, and the synthesis dies anyway.** Every group that
sustained reports attached a small immediate in-game reward (an Inspiration, a downtime
action, "the Chronicle"), and the convention that works is a **named Scribe assigned before
play**, not open authorship afterwards. What dies in all four accounts is the cross-session
synthesis: "wikis tend to rot and not be read by players" (Necropraxis), "the participation
rate was low" (Alexandrian), "I'm not sure players would want to maintain it" (Mors
Immortalis). Ticket `07` acted on the first half of this; ticket `13` exists because of the
second.

**Nobody has built this.** westmarches.games is the serious incumbent — Discord sync,
adventures, waitlists, character hub, world wiki, 590 communities — and it stops at one date
per adventure with no selection criterion beyond level requirements. The generic bots are
single-datetime, first-come, FIFO waitlist, and RPG Schedule *deliberately deletes* games
48-72h after they run. The gap is stated out loud on EN World with nobody answering it. Two
useful negatives: **Room-per-quest is validated by someone who tried the alternative first**
(Watkinson moved from a channel per location to a channel per adventure), and **there is no
established term for "cut the party by who played least recently"** — three implementations
call it three different things. If we ship it, we are naming it.

**And these campaigns die.** Surveying the 2008 cohort four years on: "all of the ones I am
familiar with are gone." Cause of death is Game Master burnout or scheduling failure, not the
absence of a tool. Every long-lived campaign in the research has some pacing device for the
GM — adventuring blocks with a retrospective, two-GM rotation, enforced downtime. We have no
unit above the Slot.

### What this reaches into

`research/03-practice.md` closes with twelve numbered contradictions, ordered by how exposed
each assumption is. Acted on already: `07` (contradictions 9 and 10) and `13` (contradiction
9). The rest are recorded on the map under *Not yet specified*; the sharpest is contradiction
2, which challenges ADR 0003 the way `13` challenges ADR 0002.

**Confidence caveat carried up from the research:** Reddit blocked both `WebFetch` and
scripted access from that environment, so r/westmarches and r/DMAcademy went unread — a whole
tier of first-hand accounts. Two Patreon posts and ars ludi itself are Cloudflare-gated and
were read via search extracts and the Wayback Machine. And no first-hand account of a
recency rule *backfiring* was found, which is an absence of evidence rather than evidence of
absence.
