# Map: West Marches campaign platform

Label: `wayfinder:map`

## Destination

A written spec, backed by a domain model, for turning this Campfire fork into a
West Marches campaign platform: complete enough that implementation sessions can
build from it without making further design decisions.

Reached when every ticket below is resolved and the spec exists at
`.scratch/west-marches-platform/spec.md`.

## Notes

**Domain.** A single West Marches TTRPG campaign. A large roster of players, no fixed
party. Players propose **quests**; Game Masters offer dated **slots**; interested
players declare which slots they can make; a Game Master **confirms** a quest into one
slot, which cuts the **party** by the **rota**. Afterwards someone writes the **report**.
Play happens elsewhere.

**Read first, every session.**
- `CONTEXT.md` — the glossary. It is opinionated about vocabulary; Campfire's inherited
  terms are deliberately redefined there. Use its words in tickets and in the spec.
- `docs/adr/0001-hard-fork-of-campfire.md` — why this is a Campfire fork, and single-tenant
- `docs/adr/0002-a-quest-is-a-single-expedition.md` — why quest and session are one model
- `docs/adr/0003-scheduling-and-party-selection.md` — how a quest gets scheduled and a party gets cut
- `docs/adr/0005-the-rota-cuts-every-party.md` — the rota: what the cut ranks by, and why nobody overrides it

**Skills.** `/grilling` and `/domain-modeling` for the decision tickets, `/prototype` for
the prototype tickets, `/research` for the research tickets. Update `CONTEXT.md` inline
whenever a ticket sharpens or adds a term.

**This map plans, it does not build.** No feature code. The one exception is throwaway
prototypes, which exist to be reacted to and then deleted.

## Decisions so far

<!-- one line per resolved ticket: gist + link -->

- [Campfire's room model, lifecycle and navigation at scale](issues/01-campfire-room-lifecycle-and-navigation.md) —
  Campfire has **no room lifecycle**: no archive, no status, no read-only, no filter, no paging.
  Creating a room per quest is cheap; the sidebar and the unread badges are what break, and the
  app-icon badge literally counts unread rooms. Deletion is complete but irreversible, and the
  quest's proposer would own the delete button.
- [One campaign clock, or every player their own?](issues/04-timezones.md) — resolves to **change
  nothing**. One-zone campaign; per-player browser rendering via the existing `local_time_controller.js`;
  a slot stores a bare UTC instant; no standing availability; no campaign setting and no zone label.
  Server-side push copy is assumed to use `config.time_zone`; whether notifications name times at
  all is handed to `09`.
- [Campfire's identity, roles and onboarding](issues/02-campfire-identity-roles-and-onboarding.md) —
  authorization is one single-valued `role` enum and one `can_administer?` method across ~30 sites.
  Game Master must be a **separate boolean**, not a fourth role, or a Game Master stops being a
  player. No invitations exist: one shared join code, no approval. Per-campaign membership is ruled out.
- [What happens to a quest's room when the quest is over?](issues/08-room-lifecycle.md) — the room
  **outlives the quest but stops being a live surface**. Never deleted; it **freezes** at the quest's
  terminal state — no messages, boosts or edits — drops out of every sidebar, and is reached by a link
  from its quest. Freezing is *derived from quest state*, so there is no flag and no unfreeze to build.
  Stays `Rooms::Open` to the whole campaign for its whole life, never narrowed at confirmation. The
  sidebar carries live quest rooms only, ordered by soonest slot. Quest owns room (`dependent: :destroy`,
  no delete route). Open room creation restricted to administrators; one campaign room, **The Tavern**.
- [Report lifecycle: prompting, unreported quests, and editing](issues/07-report-lifecycle.md) —
  **overturns Q23's open authorship.** Confirmation names one **Scribe** from the party, who owes the
  report; the research says diffuse responsibility is the documented failure mode. A Game Master may
  **never** write or correct one. A quest becomes played when its slot passes, and terminal — freezing
  its room — on the report or 14 days, whichever is first; the obligation then lapses and the quest
  reads "no report" for good. One ActionText body from a heading template, settling 14 days after
  publication. Readable campaign-wide, on the quest page, with one message into the room on publish.
  The reward that makes reports work in practice is out of scope, so the platform records the debt
  and a Game Master pays it at the table.
- [How do West Marches groups coordinate today, and what breaks?](issues/03-how-west-marches-groups-coordinate-today.md) —
  the founding text contains **no party-cutting rule**, and the real-world fairness mechanism is
  **first-come plus waitlist carry-over**, not recency: priority earned by being bumped, not by
  absence. Recency does exist in the wild, but always with a signup deadline, a hidden interest
  list, and an **exemption for player-proposed quests**. In practice the Game Master posts dates
  and players claim them (70% GM-posted), which inverts our direction. Scheduling breaks *below*
  ~9 engaged players, not above one table. Nobody has built this: the incumbent stops at one date
  per adventure with no selection criterion. Twelve numbered contradictions in the research.
- [Does the report still carry the world?](issues/13-does-the-report-still-carry-the-world.md) —
  no. **ADR 0004** amends ADR 0002: a report is one party's account of what they believe they
  saw, with permanent holes, and **the campaign has no world model**. What it remembers is a
  **trail** (reports, plus the frozen room where nobody wrote one) and **a way back to it**
  (search). Retrieval is our answer to synthesis, chosen because it needs no maintenance. A
  quest may now name the quest it *follows from*, but nothing traverses the chain. ADR 0002's
  own decision — the ephemeral quest — is untouched.

- [How does someone become a player, a Game Master, or an admin?](issues/05-roles-and-roster.md) —
  **nobody is admitted to anything.** Entry keeps the one shared join code, but only administrators
  may see it; characters are created freely; the ordinary departure is **silent and carries no state
  at all**. Game Master is a person-level flag granted and revoked by administrators, and the Game
  Master *of* a quest is derived — the owner of the slot it was confirmed into. Game Master and
  administrator are **orthogonal, with no implicit escalation**, and a Game Master is a full player,
  save that a character is never a candidate for a slot its own player offered. For the hard exit,
  deactivation is amended to stop destroying the email address and to record `deactivated_at`;
  **ban may never be the departure mechanism** — it destroys every message the person ever wrote,
  which under ADR 0004 is the campaign's memory. Coming back is an explicit admin reactivation.

- [Prototype: the quest page, as an interested player sees it](issues/10-prototype-quest-page.md) —
  three variants built and reacted to. **B, a stack of per-slot cards, is the player's page; the
  matrix is `11`'s.** Eight characters by four slots does fit a phone, but a cross-tab answers the
  Game Master's question, not the player's. **ADR 0003's "3rd of 8, party of 5" cannot be written**:
  standing is **per candidate slot** — one unchanged player is simultaneously 3rd of 4 and seated on
  one date and 6th of 8 on another. **ADR 0003 is amended in place**: that consequence now asks
  for *per candidate slot, whether they would be seated and where they stand*, and `15` has to
  produce one answer per slot, not one per quest. The ranking rule itself is left to `15`.
  What makes a cut bearable rather than discouraging: a forecast not a verdict, the recency reason
  printed on **every** row and never rounded, and a waitlist position that is worth something.
  An interest that names no slot is not an interest. Added **Standing** to `CONTEXT.md`, and
  invented an `interestCloses` deadline the page cannot be written without — `15`'s to own.
- [Can a Game Master propose a quest?](issues/14-can-a-game-master-propose-a-quest.md) — **yes,
  and it is the same object**, with the proposer recorded and no second lifecycle. A Game
  Master's proposal simply arrives with its slots attached, collapsing acts one and two.
  `03` is not close on this: 70% of quests are Game-Master-posted in the one measured campaign,
  and contradiction 8 says campaigns die of *under-supply of initiative*. Colvillian's two
  channels are rejected because the only rule that actually differed between them was the
  recency exemption, which `15` struck. A single-date quest is just a quest with one candidate
  slot, so the RSVP shape every existing tool uses needs no second model.
- [Who cuts the party — the system, or whoever proposed the quest?](issues/15-who-cuts-the-party.md) —
  the system, on every quest, **binding**. **ADR 0005** amends ADR 0003 and retitles it: the
  three acts stand, the ranking does not. The cut is the **rota** — bumps since your last seat,
  then how long since that seat, then who declared first — because the attested mechanism
  rewards *being refused*, not being absent, and pure recency let a player who never declared
  outrank one bumped twice. No Game Master override (`Q13`, which `11` cited for one, turns out
  not to exist anywhere in this repo); a Game Master picks **which slot** and nothing else.
  Everything is per candidate slot: standing, bumps, and a **deadline 72h before each slot**
  that hard-gates confirmation into it. The interest list is **fully disclosed** with the reason
  on every row, because a binding cut has no appeal and checkability is its only accountability
  — and Colvillian's chilling-effect objection dies with the rota, where being passed over is
  what buys the next seat. **A seat spends your standing, not attendance**: withdraw before the
  deadline and keep your place, no-show and you have spent it. Waitlist is the rota continuing,
  **frozen**. The cut is blind to your character, and level went further — the platform records
  it and **never compares it**, because advisory-but-computed is attested nowhere and a stale
  number driving an automatic decision is worse than none.

## Not yet specified

- **Character lifecycle.** What happens on death or retirement? **Unblocked, and much
  smaller than it was.** `05` settled that players create characters freely; `15` settled that
  level is a note the platform never reasons about, so "who sets a character's level, and when"
  no longer gates anything; and the standing half is gone entirely, because the rota ranks
  players, not characters. What is left is death and retirement, and whether either is a state
  the platform holds at all.
- **The campaign's front page.** What a player sees on arrival: open quests, my standing,
  upcoming slots, unreported quests? `01` and `10` are in — `10` drew its page inside a fake
  sidebar carrying `08`'s rules and nothing fought it — so this now waits on `11` alone.
  `14` adds one question to it: whether Game-Master-proposed and player-proposed quests share
  one list. One model implies one list; the layout is this item's.
- **Permissions in detail.** Who may edit or withdraw a quest that isn't theirs, or remove
  someone's interest. **Unblocked** by `05`, which settled the two flags and their scope but
  not these. `14` routed one more here: whether a Game-Master-proposed quest is any different,
  and its answer was that nothing about who proposed it makes it a different question.
  (Correcting a report is already answered: `07` says a Game Master may never do it.)
- **Life at a year in.** `08` settled that rooms are kept forever and frozen ones leave the
  sidebar, so navigation no longer degrades. What remains: whether hundreds of played quests
  and their reports need paging or retention on their own listing pages.
- **Is a played quest that "didn't happen" a thing?** `07` depends on a Game Master being able
  to say a confirmed quest was cancelled on the day, or the system demands a report from a party
  that never met. Ticket `06`'s, and `15` raised the stakes: under ADR 0005 the *seat* spends
  your standing, so a cancelled quest that isn't handled costs five people their turn on the
  rota.

From `03`'s research. Its two sharpest contradictions became tickets `14` and `15`, both now
resolved — `15` in **ADR 0005**. What remains here has no ticket:

- **No unit above the Slot, and burnout is the documented cause of death.** Adventuring blocks with
  a retrospective, two-GM rotation, enforced downtime — every long-lived campaign has a pacing
  device for the Game Master. Ours has none. (Contradiction 11.)
- **Search quality is now load-bearing.** ADR 0004 makes retrieval the whole of our answer to
  "what do we know about X", but `SearchesController` still does `last(100)` with no ranking
  and no scoping (`app/controllers/searches_controller.rb:23`). Ranking, paging and scoping
  need an owner. `geared_pagination` is already in the `Gemfile`. Nothing in `08` changes — the
  bar moved, not the decisions.
- **Multi-Game-Master continuity.** Several Game Masters share one world with no notion of who has
  claimed which part of it; real campaigns use tile-claim spreadsheets and out-of-band briefings.

## Out of scope

Ruled beyond this destination. These do not graduate; revisiting one means redrawing the
destination as a fresh effort.

- **The discovered world map.** Hexes, locations, fog-of-war rendering. A genuinely
  different product with spatial data and image handling. Ruled out on those merits alone —
  ADR 0004 struck the old justification that "the report carries the world instead", and the
  campaign now has no world model at all.
- **The synthesis: leads with state, a lore index, an accumulated picture of what is known.**
  The one artefact this format has never sustained (`03`); retrieval is our answer instead.
  See ADR 0004. This is the most valuable thing in `03`'s research that we are deliberately
  not building, and the strongest candidate for a future effort with its own destination.
- **Anything at the table.** Dice, character sheets, stats, inventory, initiative, rules
  content, VTT features. This platform covers the time between sessions.
- **Calendar sync.** iCal and Google Calendar export of confirmed quests. A real and
  tempting integration, but an integration, not a domain decision — it can follow the spec.
- **Payments.** Patreon, subscriptions, paid seats.
- **A public campaign website.** Anything readable without an account.
- **Multi-tenancy.** One deployment stays one campaign. See ADR 0001.
- **Tracking upstream Campfire.** Hard fork. See ADR 0001.
