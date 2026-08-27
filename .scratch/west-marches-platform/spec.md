# The West Marches platform: build specification

This is the destination of the `wayfinder:map` effort: what to build, in enough detail that an
implementation session never has to make a design decision to proceed. Every ruling here is
either carried from a resolved ticket or an ADR, or is made in this document and listed in
[§9](#9-decisions-this-spec-makes). Nothing is left to taste.

## 0. How to read this, and what wins

Three documents, three jobs:

- **`CONTEXT.md`** is the vocabulary. Use its words — in code, in copy, in the interface. It is
  opinionated, and Campfire's inherited terms are deliberately redefined there.
- **`docs/adr/`** is the reasoning. When this spec says a thing, the ADR says *why*, and the why
  is what stops the decision being quietly undone. The ADRs are a log: they amend each other in
  place with pointers and are never rewritten.
- **This spec** is the build. Where it and an ADR disagree, this spec is later and wins, and
  [§9](#9-decisions-this-spec-makes) says so explicitly with the amendment named. There is no
  silent divergence anywhere in this document.

Two rules bind every section below, and both come from ADRs rather than taste:

1. **No notification may be the only place a fact lives** (ADR 0007). Every event is also legible
   as state on a screen someone can walk up to. If push never arrives, the campaign runs slower
   and still runs.
2. **State is derived from time; a recurring job only tells** (ADR 0007). `played?` is a
   comparison against a timestamp, not a column a worker was supposed to set. A dead worker loses
   messages and never corrupts data.

---

## 1. The campaign, in one page

A single West Marches campaign, one deployment (ADR 0001). A large **roster**, no fixed party.
Play happens elsewhere; this platform covers the time between sessions.

**Three acts** (ADR 0003, as amended by ADR 0005 and ADR 0006):

1. **Quests are proposed** — by a player, or by a **Game Master**, whose proposal simply arrives
   with its slots already attached. One model, one lifecycle, the proposer recorded (ADR 0005).
   Every quest gets a chat **room** at proposal, open to the whole campaign.
2. **Slots are offered and pointed at** — a Game Master posts a dated window **unattached**, and
   quests are *pointed at* it, by their proposer or by any Game Master, making it one of that
   quest's **candidate slots**. Several quests may target one date. Interested players declare,
   per quest, which of its candidate slots they could make, naming one **character**. An interest
   that names no slot is not an interest.
3. **A Game Master confirms** the quest into one of its candidate slots, once that slot's
   deadline has passed. Confirmation spends the slot for every other quest targeting it, cuts the
   **party** by the **rota**, freezes the **waitlist** behind it, and shows who the **report**
   will fall to. The Game Master chooses the slot and nothing else.

Afterwards the quest becomes **played** when its slot passes, the scribe owes a **report**, and
the quest is **closed** by the report or by 14 days, whichever comes first — at which point its
room **freezes** and the campaign moves on. What the campaign remembers is the trail of reports
plus the frozen rooms where nobody wrote one, and search is the way back to it (ADR 0004).

**Six states, and every way a quest ends** (ADR 0006):

```
      ┌────── cancel ───────┐
      ▼                     │
   proposed ──confirm──▶ confirmed ──slot passes──▶ played ──report, or 14 days──▶ closed
      │  │
      │  └── withdraw ──▶ withdrawn (terminal)
      └── 14 days with nothing left that could confirm it ──▶ abandoned (terminal)
```

`closed`, `withdrawn` and `abandoned` are terminal, and all three freeze the quest's room.
Cancellation is not a state: a cancelled quest goes back to **proposed** with its interest intact,
and the slot stays spent. There is no `declined` state and no `stale` state.

---

## 2. The data model

SQLite, one deployment, one campaign. Nothing below carries a tenant column: belonging to the
campaign is what having a row means (ADR 0001).

### 2.1 Inherited tables, amended

**`users`** — two columns added, one behaviour repaired.

| Column | Type | Notes |
|---|---|---|
| `game_master` | boolean, `default: false, null: false` | A standing property of a person, granted and revoked by administrators only. **Never** a fourth value of the single-valued `role` enum — that would stop a Game Master being a member (ticket `02`). |
| `deactivated_at` | datetime, nullable | `users.status` records *that* a transition happened and never *when*. Set on deactivation, cleared on reactivation. |

`User#deactivate` **stops mangling the email address** (`app/models/user.rb:44,57-59`): email is the
only identity key, and rewriting it makes a returning player structurally a stranger with no route
back to their characters (ticket `05`). Everything else about `deactivate` stands, and three
domain consequences hang off it, gathered here because they are easy to miss:

- Their **unconfirmed proposals are withdrawn** — a quest is a proposal of intent and nobody is
  left to answer questions in its room (ticket `05`).
- If they are a Game Master, their **unspent future slots are withdrawn** and their **confirmed
  quests are cancelled**, landing each back at proposed where another Game Master can point it at
  a new date (ADR 0006). Merely *revoking* the flag does only the first of those.
- **Reactivation is a real path**, not a status flip ([§4.11](#411-administration)).

Their characters, messages, reports and past placements are untouched. Deactivation removes access,
never history.

Every "who are the Game Masters" query scopes `active` — a deactivated Game Master is simply not
one, and `role`-style unscoped lookups are the inherited trap ticket `02` flagged.

**`rooms`** — one column added.

| Column | Type | Notes |
|---|---|---|
| `quest_id` | integer, nullable, unique index | Null for **The Tavern** and for every **Herald** room. Set for a quest's room, which the quest owns. |

No `status`, no `archived_at`, no `frozen` flag. Freezing is derived from the quest's state
(ADR 0006, ticket `08`), so there is nothing to keep in sync and no unfreeze to build.

### 2.2 New tables

**`characters`**

| Column | Type | Notes |
|---|---|---|
| `user_id` | integer, not null, indexed | Its player. |
| `name` | string, not null | |
| `level` | integer, nullable | A note for people to read. The platform records it and **never compares it** (ADR 0005). |

A player creates characters freely and may edit either field at any time. Characters are never
deleted: past parties, placements and attendance reference them. There is no death or retirement
state — see [§8](#8-non-goals-and-stated-limits).

**`quests`**

| Column | Type | Notes |
|---|---|---|
| `proposer_id` | integer, not null, indexed | A player or a Game Master; nothing distinguishes the two afterwards (ADR 0005). |
| `title` | string, not null | |
| `description` | ActionText rich text | Same filter chain as `Message` (`app/helpers/content_filters.rb`), so attachments and rich text come free. |
| `level_range` | string, nullable | Free text — "levels 3–5". Never parsed, never compared. |
| `party_size` | integer, not null, default 5 | The **maximum** number of seats. There is no minimum (ADR 0006). |
| `follows_from_quest_id` | integer, nullable, indexed | A pointer for people to read. **Nothing traverses it** (ADR 0004). |
| `confirmed_slot_id` | integer, nullable, indexed | Set at confirmation, cleared by cancellation. |
| `confirmed_at` | datetime, nullable | |
| `withdrawn_at` | datetime, nullable | |
| `created_at` | datetime | The moment of proposal; anchors the abandonment clock when no slot was ever offered. |

`has_one :room, dependent: :destroy`. `has_one :report, dependent: :destroy`. A quest's room has no
independent existence and **no delete route at all** (ticket `08`).

**`slots`** — an unattached offer of one Game Master's time (ADR 0006).

| Column | Type | Notes |
|---|---|---|
| `game_master_id` | integer, not null, indexed | Its owner. "Who is running this one" is read off here and never stored on the quest (ticket `05`). |
| `starts_at` | datetime, not null, indexed | A bare UTC instant. No authoring zone, no wall-clock-plus-zone (ticket `04`). |
| `ends_at` | datetime, not null | A slot carries a start **and** an end. "The version with only a start reads as an ambush" (ticket `10`). |
| `spent_at` | datetime, nullable | Set at confirmation. **Never cleared** — cancelling a confirmed quest leaves the slot spent (ADR 0006). |
| `withdrawn_at` | datetime, nullable | A Game Master may withdraw a slot nothing has been confirmed into. |

`closes_at` is **derived**, not stored: `starts_at - 72.hours`. One campaign, no settings page
(ticket `04`).

**`candidacies`** — quest ↔ slot. Unique on `(quest_id, slot_id)`.

| Column | Type | Notes |
|---|---|---|
| `quest_id`, `slot_id` | integer, not null | |
| `added_by_id` | integer, not null | The proposer or a Game Master. Recorded because removing one is a permission question ([§3.10](#310-permissions)). |

**`interests`** — one per player per quest. Unique on `(quest_id, user_id)`.

| Column | Type | Notes |
|---|---|---|
| `quest_id`, `user_id` | integer, not null | |
| `character_id` | integer, not null | One named character per interest. Not a stable (ADR 0005). |
| `created_at` | datetime | The rota's third sort key: who declared first. |

**`declarations`** — interest ↔ candidate slot: the dates this interest could make. Unique on
`(interest_id, slot_id)`. Destroying the last declaration on an interest **destroys the interest**
(ADR 0003, as amended by ticket `10`).

**`placements`** — the frozen rota, written at confirmation. Unique on `(quest_id, character_id)`
and on `(quest_id, position)`.

| Column | Type | Notes |
|---|---|---|
| `quest_id`, `character_id`, `user_id` | integer, not null | |
| `position` | integer, not null | 1-based rota position at the moment of confirmation. **Frozen**: the waitlist never re-sorts, because "first seat to free up is yours" is a lie if it does (ADR 0005). |
| `released_at` | datetime, nullable | The seat or waitlist place given back, before the slot starts. |

Every eligible interest that named the confirmed slot gets a row — party *and* waitlist, one
ordered list, with the cut line at `party_size`. Cancellation destroys them all.

**`reports`** — one per quest. Unique on `quest_id`.

| Column | Type | Notes |
|---|---|---|
| `quest_id` | integer, not null | |
| `body` | ActionText rich text | One body. Not accumulated attributed contributions — that is a chat log, and the quest already has one next to it (ticket `07`). |
| `author_id` | integer, not null | The byline: whoever actually wrote it, which is usually but not always the scribe. |
| `published_at` | datetime, not null | Starts the settling clock. |
| `last_edited_by_id` | integer, nullable | |
| `updated_at` | datetime | "Last edited" line. There is no version history and no `paper_trail`. |

**`attendances`** — who actually went. Unique on `(quest_id, character_id)`.

Rows exist **only once a Game Master has corrected attendance**. Until then, attendance *is* the
party at the slot's start, derived. On the first correction the whole list is materialised and is
authoritative from then on. This keeps the correction an act rather than something a job writes
([§0](#0-how-to-read-this-and-what-wins), rule 2).

**`announcements`** — the recurring job's ledger. Unique on `key`.

| Column | Type | Notes |
|---|---|---|
| `key` | string, not null | e.g. `slot:412:ripe`, `quest:88:played`, `quest:88:chase-11`, `quest:88:abandoned`. |
| `created_at` | datetime | |

The job's entire responsibility is to notice what has newly become true and say so, recording that
it did. The ledger guards duplicates and nothing else: no state anywhere depends on a row existing.

### 2.3 Derived, and never stored

This list is load-bearing. Every entry is a column somebody will want to add and must not.

| Fact | Derivation |
|---|---|
| A quest's **state** | See [§3.7](#37-the-clocks). Six states, all comparisons against timestamps. |
| A room being **frozen** | Its quest is `closed`, `withdrawn` or `abandoned`. Three triggers, no flag (ADR 0006, ticket `08`). |
| A slot's **deadline** | `starts_at - 72.hours`. |
| A slot being **available** | `spent_at` and `withdrawn_at` both null. |
| The **Game Master of a quest** | The owner of the slot it was confirmed into (ticket `05`). |
| The **rota**, and a player's **standing** | [§3.1](#31-the-rota), [§3.2](#32-standing). Computed per candidate slot, on read. |
| **Bumps** and **last seat** | [§3.3](#33-what-a-placement-becomes). From placements, slots and `released_at`. |
| The **party** and the **waitlist** | The first `party_size` unreleased placements by position; the rest. |
| A **promotion** | Falls out of the above the moment a placement is released. Nothing is written. |
| The **scribe** | [§3.8](#38-the-scribe). |
| **Attendance**, before any correction | The party at the slot's start. |
| A report having **settled** | `published_at + 14.days`. |
| A quest's **abandonment date** | [§3.7](#37-the-clocks). |

### 2.4 Constants

Hard-coded, no settings page (the precedent is ticket `04`, and every later ticket has held it):

- **72 hours** — a slot's deadline before its start.
- **14 days** — the report obligation, the report's settling window, and the abandonment clock.
  One constant, three clocks, explainable in a sentence: *two weeks to write it, two weeks to fix
  it.*
- **Day 11** — the single scribe reminder.
- `config.time_zone` is set to the campaign's zone in `config/application.rb`. One line, no UI.

---

## 3. The rules

### 3.1 The rota

The order a party is cut in (ADR 0005). Computed **per candidate slot**, on read, and applied once
at confirmation. It ranks **players**, not characters, and it cannot see a character's level.

Candidates for slot `S` on quest `Q`: every interest on `Q` that has a declaration for `S`, minus
any interest whose player owns `S`. (A character is never a candidate for a slot its own player
offered — derived, never stored, and the screen says why in prose, not as a grid state: *"Vetch can
make this date but is not in the running for it — that player offered it."* Tickets `05`, `11`.)

Sort, in order:

1. **Bumps since your last seat**, most first.
2. **How long since that seat**, longest first; **never seated sorts as infinitely long**.
3. **Who declared first** — `interests.created_at`, ascending.

The bot user never appears. Deactivated players never appear (their interests remain but they
cannot be seated; scope candidates to `User.active`).

**Running a quest is not a seat.** The rota's keys are seats and bumps, and running is neither, so
a Game Master's own standing is untouched by how much they run — which is ticket `05`'s instinct on
record ("running is not playing") and falls out of the definition rather than needing a rule. The
consequence, stated so nobody is surprised by it: a Game Master who mostly runs will sort high on
the rare occasions they declare.

### 3.2 Standing

Where a player sits in the rota for **one candidate slot** — "4th of 6, seated", "6th of 8, first
on the waitlist". There is one per candidate slot, never one per quest: ADR 0003's "3rd of 8, party
of 5" **cannot be written**, and ticket `10` proved it on a plausible roster where the same
unchanged player is simultaneously 3rd of 4 and seated on one date and 6th of 8 on another.

Before confirmation, standing is a **forecast, not a verdict**, and every surface that shows it
says so. Five rules for the copy, all of them from ticket `10` and all of them load-bearing:

1. **Hedge every number.** *"The party this date would produce"*, *"you'd be on it"*, *"the cut, if
   that date were confirmed today"*. A bare "6th of 8" is a door closing; "6th of 8 **so far**" is a
   queue you are standing in.
2. **Print the reason on every row, not just yours.** Losing is survivable; losing to an
   unexplained ordering is not. A binding automatic cut has no appeal, so being checkable by the
   person it disadvantages is the only accountability it has (ADR 0005).
3. **Never round the reason.** Exact days under four weeks, weeks beyond. Two identical reasons in
   different positions reads as a bug or as favour.
4. **A losing position comes with a future.** *"1st on the waitlist — first seat to free up is
   yours."* Without that, a position is information with nothing attached to it.
5. **Say the thing the player will suspect**: *"which character you bring doesn't change your place
   in the order — that follows you, not them."*

The reason line names **seats**, not play: *"2 bumps · last seat 11 days ago"*, *"no seat yet"*. The
prototypes wrote "played 11 days ago" and "never played", which is wrong under ADR 0005 — a seat
spends your standing whether or not you walk in, and the copy has to mean what the rule means.

### 3.3 What a placement becomes

Evaluated at the slot's **start**, and stable forever after, because releases are impossible once a
quest has started.

| At the start of play, the placement was… | Outcome | Effect on the rota |
|---|---|---|
| Seated (position within `party_size` among unreleased placements) | **a seat** | Resets: `last_seat_at` becomes the slot's `ends_at`, and bumps before it stop counting. Whether or not the player turned up. |
| Released before the start | **nothing** | Standing untouched. Give the seat back and you keep your place. |
| Not seated and not released | **a bump** | +1 bump. Somebody refused you, and that is what buys the next seat. |

This is ADR 0005 as restated by ADR 0006: **the seat is spent unless you give it back before it is
used.** The strict alternative — any drop-out after the deadline spends it — destroys what the rule
bought, because if announcing costs exactly what ghosting costs, nobody announces.

Three edges fall out rather than needing rules:

- **A quest that dies unconfirmed bumps nobody.** No placements were ever written.
- **A quest that merely lost the date to another quest bumps nobody.** Its declarers were never
  placed (ADR 0006's sharpening of the bump).
- **A cancelled quest moves nobody's standing.** Its placements are destroyed and every seat was
  given back before it was used.

Computed for player `P` at time `T`:

```
resolved   = P.placements joined to quests joined to slots
             where slots.starts_at <= T and quests.confirmed_slot_id is not null
seats      = resolved.select(&:seated_at_start?)     # unreleased, and inside the cut line
last_seat  = seats.max_by { slot.starts_at }        # nil when never seated
last_seat_at = last_seat&.slot&.ends_at
bumps      = resolved.count { |p| !p.seated_at_start? && p.released_at.nil? &&
                                  (last_seat.nil? || p.slot.starts_at > last_seat.slot.starts_at) }
```

### 3.4 Seats, releases and promotion

- The **party** is the first `party_size` placements by `position` whose `released_at` is null. The
  **waitlist** is every unreleased placement after them, in the same frozen order.
- A player may **release** their placement — seat or waitlist place — at any time before the slot
  starts. Releasing costs nothing.
- **Promotion is automatic, at any hour, including one hour before.** It is not an act and nothing
  is written: the next unreleased placement simply *is* in the party once the one ahead of it is
  released. A Game Master choosing would be ADR 0005's override arriving by the back door.
- The release action is responsible for the Herald message to whoever newly became seated
  ([§5](#5-notifications)). The promotion itself needs no job, and a dead worker therefore costs a
  message and never a seat.
- After the slot starts, releasing is refused. The party at start is what the rota resolves against.

### 3.5 Confirmation

The single moment a quest becomes real. One transaction.

**Preconditions**, all hard:

- The actor is an active Game Master **and owns the slot**. A slot is an offer of that person's
  time; other Game Masters' dates appear on the screen greyed and cannot be confirmed into
  (ticket `11`).
- `now >= slot.closes_at`. The gate is hard, not advisory: the first Game Master to confirm early
  has restored first-come for everybody (ADR 0005).
- `now < slot.starts_at`.
- `slot.spent_at` and `slot.withdrawn_at` are null.
- The quest is in state `proposed` and has a candidacy for this slot.
- At least one eligible interest names this slot. A slot with zero interest cannot be confirmed
  because there is no party to cut — arithmetic, not a rule, and **not** a minimum party size.

**Effects**, in order:

1. `quest.confirmed_slot_id = slot`, `confirmed_at = now`.
2. `slot.spent_at = now` — spent for every other quest targeting it, and never returned.
3. Placements written for **every** eligible interest naming the slot, `position` 1..n in rota
   order. The cut line is `party_size`; there is no separate waitlist record.
4. The slot owner's membership on the quest's room is raised to `everything`.
5. One system message into the quest's room: confirmed, which date, the party, the waitlist, the
   scribe. It mentions nobody ([§5](#5-notifications)).
6. Herald messages, one per affected player, composed per act ([§5](#5-notifications)).

**What confirmation does not do**: it does not narrow the room to the party (ticket `08`), it does
not choose the party, and it does not name the scribe. The Game Master's one judgement is which
slot — which makes slot choice quietly into party engineering, and the confirmation screen is
designed knowing it (ADR 0005).

### 3.6 Cancellation

**Any** active Game Master may cancel any confirmed quest, not only the one whose slot it sits in —
otherwise a Game Master who drifts away leaves quests confirmed onto dead dates that nobody can
touch (ADR 0006). Only **before the slot starts**: after that the evening either happened or it did
not, and attendance is the record of which.

- `confirmed_slot_id` and `confirmed_at` cleared: the quest is **proposed** again, with its interest
  and its other candidate slots intact.
- All placements destroyed. Nobody's standing moves.
- **The slot stays spent.** Cancelling is the Game Master saying that evening will not be run, so
  returning it to the pool would advertise time that does not exist. The quests that lost the date
  do not get it back — it was never the confirmation that took it from them.
- No report is ever owed: only played quests can owe one.
- **Deactivating a Game Master cancels their confirmed quests**, landing each back at proposed where
  another Game Master can point it at a new date. Merely *revoking* the flag leaves confirmed quests
  alone (ticket `05`) but withdraws their unspent future slots.

### 3.7 The clocks

All four are comparisons. Nothing sets a state.

```ruby
def state
  return :withdrawn if withdrawn_at?
  if confirmed_slot_id?
    return :confirmed unless confirmed_slot.ends_at.past?
    return :played    unless closed?
    :closed
  else
    abandoned? ? :abandoned : :proposed
  end
end

def closed?                       # played quests only
  report&.published_at.present? || confirmed_slot.ends_at + 14.days < Time.current
end

def abandoned?                    # proposed quests only
  last_confirmable_at + 14.days < Time.current
end

def last_confirmable_at
  candidate_slots.maximum(:starts_at)&.-(72.hours) || created_at
end
```

Notes that are decisions, not implementation detail:

- **`played` is automatic.** A Game Master who forgets to mark a quest played would silently stop
  the campaign's memory (ticket `07`).
- **The abandonment anchor is the latest deadline among *all* candidate slots**, spent and withdrawn
  ones included. Deliberately generous: the alternative reads the pool's churn as the quest's fault.
  Pointing the quest at a new slot moves the anchor, so **offering a date restarts the clock**,
  and a quest with a live date three months out is never at risk.
- **A quest with no slot ever offered dies 14 days after proposal.** This is the transition ticket
  `08` demanded — abandonment has to be something that actually happens, or dead proposals never
  freeze and slowly fill the sidebar.
- **A quest may be confirmed between a slot's deadline and its start.** The deadline opens
  confirmation; it does not close it.
- **Withdrawal** is the explicit dead end from proposed: the proposer's own act, and the automatic
  one when a player is deactivated (their unconfirmed proposals are withdrawn).

### 3.8 The scribe

**Derived, never chosen, and never stored.** Ticket `11` struck the scribe picker from the
confirmation screen: it was the one control that let a Game Master exercise judgement over a named
person, which ADR 0005 removed from the party and did not mean to re-admit through the report. A
Game Master who cannot choose who plays should not be able to choose who writes.

The rule, ratified here (it was invented by ticket `11` and sits in no ADR):

> The report is owed by whoever in the quest's **attendance** has gone longest without publishing
> one — **never-scribed first** — ties broken by **rota position**.

Precisely, evaluated over the attendance (which is the party at the slot's start until a Game
Master corrects it), counting only reports published **before this quest's slot started**:

1. Players who have never published a report, first.
2. Otherwise, oldest `published_at` first.
3. Ties by `placements.position` ascending.

Deliberately **not the rota itself**. If the two were one rule, finally earning a seat after three
bumps would arrive with a chore attached, and the rota would be handing out compensation and
homework in the same act.

Because it is derived from the attendance, the debt follows what actually happened: a scribe who
releases their seat before play, or whom a Game Master removes from attendance, is not the scribe.
The name shown in the confirmation dialog is therefore a **forecast**, like everything else on that
screen, and the screen says so. This corrects ticket `07`'s "the system asks the Game Master to name
a new one" — nobody is asked, and nobody names it.

The debt is derived as a **character** because that is what a party is made of, but it belongs to
the **player** behind it and outlives that character.

### 3.9 Contender ordering

On the confirmation screen, the quests competing for one date are ordered **best party first** —
by the size of the party that pairing would produce, descending. Ratified here; it was implemented
and accepted by ticket `11` and sits in no ADR.

Its cost is named rather than avoided: it sinks the quest whose only live date this is, because
that quest is often the one with the least interest. The counterweight is not the sort order but
the flag underneath it — **"this is its only live date"** — and both are required. A contender card
also carries how long that quest has before its abandonment clock runs out, which ticket `11`
lifted from its variant C.

### 3.10 Permissions

Two standing flags, orthogonal, neither implying the other: `administrator` (`can_administer?`) and
`game_master`. An administrator's mistakes are operational; a Game Master's are social.

| Act | Who |
|---|---|
| Propose a quest | any active player, including a Game Master |
| Edit a quest's title, description, level range, party size, follows-from | its proposer, any Game Master, an administrator — while the quest is `proposed` or `confirmed` |
| Withdraw a quest | its proposer, or an administrator. Automatic on the proposer's deactivation |
| Delete a quest | administrators only, and the confirmation names the room and the report it will take with it |
| Offer a slot | any active Game Master |
| Withdraw a slot | its owner, or an administrator, and only while `spent_at` is null |
| Point a quest at a slot | the quest's proposer, or any Game Master |
| Un-point a quest from a slot | whoever added the candidacy, or an administrator — and only while the quest is `proposed`. This can end someone's interest, so it carries the same notification as a withdrawn slot |
| Declare, edit or withdraw interest | the player themselves, and **nobody else**. Not a Game Master, not an administrator. The platform does not police who wants to go |
| Release a seat or waitlist place | the player themselves, before the slot starts |
| Confirm a quest into a slot | the slot's owner |
| Cancel a confirmed quest | any active Game Master |
| Correct attendance | any active Game Master, after the slot ends |
| Write or edit a report | anyone in the quest's attendance, until it settles. **Never** a Game Master, in either capacity, unless they were in the attendance as a player |
| Reopen a settled report | administrators only, for a genuine correction |
| Grant or revoke `game_master` | administrators only |
| See the join code | administrators only |
| Deactivate or reactivate a player | administrators only |
| Create an Open room | administrators only |

Nothing about who proposed a quest makes any of these a different question (ticket `14`). That is
the whole of the map's "Permissions in detail" item.

---

## 4. Screens

Ten surfaces. Every one of them is reachable by an active player unless said otherwise; there are
no per-quest visibility rules anywhere in this product.

### 4.1 The Board — the campaign's front page

`GET /` (replacing `welcome#show` as root). The map left this unspecified and ADR 0007 made it
load-bearing: **every silence in the notification matrix is a fact that has to be visible
somewhere, and most of them land here.** It is the answer to "what should I be doing?"

Sections, in this order:

1. **Yours, now.** Only the rows that exist:
   - *Report owed* — the quest, and "6 days left". This is the debt the platform records and a Game
     Master pays at the table (ticket `07`).
   - *Seats you hold* — confirmed quests you are in the party for, soonest first, with the date, the
     party, and a **give the seat back** control.
   - *Waitlist places you hold* — "2nd on the waitlist — first seat to free up is yours."
   - *Your live interests* — proposed quests you declared on, each with its next candidate date and
     your standing on it. This is where pre-confirmation drift becomes visible without a message
     (ADR 0007's deliberate silence).
2. **On the board.** Every `proposed` quest, ordered by soonest live candidate slot, then by
   proposal date. Each row: title, proposer, level range, seats, how many have declared, its next
   date or **"no date yet"**, and a quiet marker when its abandonment clock is inside 3 days. No
   filtering by stake — hiding proposals from uninterested players hides exactly the quests that
   need eyes on them (ticket `08`).
3. **Dates on offer.** Unspent, unwithdrawn slots, soonest first, each naming its Game Master, how
   many quests are pointed at it, and when interest closes. A player reads this to know when to
   propose something; a Game Master reads it to know what the campaign is already committed to.
4. **Coming up.** Confirmed quests across the whole campaign, whether or not you are on them.
5. **Played, awaiting a report.** Quest, scribe, days remaining. Public, because the debt is public;
   the *chase* is private ([§5](#5-notifications)).
6. **The trail.** The most recent published reports, with a link to all of them.

A Game Master additionally sees **You run** at the top: their own dates, with the ripe ones
("interest closed — 3 quests want this date") first. It links to
[§4.5](#45-the-confirmation-screen); it is not that screen.

One quest-proposal button and, for Game Masters, one offer-a-date button.

### 4.2 The quest page

`GET /quests/:id`. The screen the platform is judged on, and the one ticket `10` prototyped three
ways. **Variant B — a stack of per-slot cards — is the player's page.** No grid anywhere: the
cross-tab answers the Game Master's question, not the player's, and a list degrades into scrolling
rather than into illegibility.

**Header.** Title, proposer, level range, party size, and — if set — "follows from *[quest]*" as a
plain link. State, in the campaign's own words rather than a status pill: *on the board*, *confirmed
for Friday 4 September*, *played, report owed*, *closed*, *withdrawn*, *abandoned*.

**Description**, then **the room link**, directly above the declaration control. Arguing for a date
is what you do before you tick one, not after (ticket `10`).

**While `proposed` — one card per candidate slot**, soonest first. Each card carries:

- `Fri 4 Sept · 17:00–21:00 · about 4 hours`, rendered in the browser by the existing
  `local_time_controller.js`. No zone label anywhere (ticket `04`).
- **Interest closes in 8 days**, or *closed — this date can be confirmed*. This is the most
  surprising rule on the screen and the one most likely to be worked around if it is not explained
  where it bites (ticket `11`).
- **The party this date would produce**, in rota order, numbered, the cut line drawn, with the
  reason printed on every row ("2 bumps · last seat 11 days ago"). The waitlist continues the
  numbering below the line.
- **"1 seat unfilled"** as a neutral fact where the party is short. Under-subscription is the common
  case, not an edge (tickets `10`, `11`).
- **Your standing on this date**, hedged, per [§3.2](#32-standing) — or "you can't make this one",
  or the exclusion sentence where you offered the date yourself.

**The declaration control.** Pick a character, tick the dates you can make. Declaring interest and
naming at least one date are **one act**: the button is disabled until a date is ticked, and
unticking your last date withdraws you from the quest. Editing is the same control.

**After confirmation** the card for the confirmed slot leads and the other cards fall away: they
are no longer candidates for this quest, though any that were not spent are still live dates for
somebody else's. The confirmed card carries the party, the waitlist in frozen order, the scribe, and — for the players in it —
the release control. Below: the report, or **"no report"** once the quest is closed without one. The
gaps are permanent and visible (ADR 0004).

**Terminal quests** keep the whole page, plus the link into the frozen room labelled with its
message count.

### 4.3 Proposing a quest

`GET /quests/new`. Title, description, level range, party size, optional "follows from". Optional
date-pointing in the same form for anyone who can already see the pool, which is how a Game
Master's proposal "arrives with its slots already attached" (ADR 0005) — the same object, the same
form, no second channel.

On create: the quest, its `Rooms::Open` room (named for the quest, `quest_id` set, memberships
granted to all active players by the inherited `Rooms::Open` callback), and one message in **The
Tavern**.

### 4.4 Offering a date

`GET /slots/new`, Game Masters only. Start, end. The deadline is shown as a derived consequence —
"interest closes Tuesday 4 September, 17:00" — never as a field. Optionally point it at one or more
open quests in the same act.

`GET /slots` is the pool: unspent dates, whose they are, which quests want them.

### 4.5 The confirmation screen

`GET /slots/mine` (reached from **You run**), Game Masters only, and it deliberately shows **only
dates the viewer offered**, plus other Game Masters' dates greyed for context — a quest that can run
on somebody else's Saturday is a reason not to spend your Friday on it.

**Variant B — one card per date, the contenders side by side inside it.** ADR 0006's two-axis
decision needs no second axis: the date is scarce and the Game Master owns it, the quest is what the
campaign is offering for it, and a grid draws them as equals when they are not. It also does what a
grid cannot — hold the rota next to the price.

Each **date card**: the window, the countdown to its deadline or *interest closed*, and the
contenders ordered best party first ([§3.9](#39-contender-ordering)).

Each **contender**, inside the card: the quest, its proposer, `5 seated · 1 waiting` or
`3 of 5 · 2 seats unfilled` as a neutral price, the flag **"this is its only live date"** where it
is, its abandonment countdown, and — in place, not in a drawer — the rota for this pairing with the
reason on every row. The exclusion sentence sits under the rota as prose.

**Confirm** opens a dialog that enumerates exactly what the act spends, because this is the loudest
single moment in the system: the seats, the waitlist frozen in this order, the scribe it derives,
the bumps dealt to everyone who declared and could not be seated, and the other quests that lose
this date. Nothing on this dialog is editable. There is no override and no scribe picker.

When no date produces a decent party the screen says so plainly and offers nothing else: there is no
minimum, so a Game Master may run it for two, wait, or offer another date. There is no third thing.

### 4.6 The report

`GET /quests/:id/report`, `edit` for anyone in the attendance until it settles.

Pre-filled with a heading template — *who was there · where they went · what they found · rumours
and leads · what they took*. Headings, not fields: a form would fight every quest that does not fit,
and the rumours-and-leads section is the part that feeds the next quest and the part a chat log
never produces (ticket `07`). Attendance renders itself from the amended list.

On publish: the report is readable campaign-wide, one message goes into the quest's room (the last
thing in it before it freezes, pointing at the durable record), and one goes into **The Tavern**.
Edits do not re-post. A scribe byline and a "last edited by" line; no version history. It settles 14
days after publication, and only an administrator can reopen it — when a later quest shows the party
read something wrong, the correction belongs in the *new* report, and the old one should keep saying
what they thought at the time.

A Game Master may neither write nor correct one, and the system enforces it.

### 4.7 Attendance

On the quest page once the slot has ended, for Game Masters: tick who actually went. Defaults to the
party at the start; may add a waitlist character who came along anyway and drop someone who fell
ill. It decides who may edit the report and it re-derives the scribe. It does **not** feed the rota
(ADR 0005) — a seat spends your standing whether or not you walk in — so it is a record of what
happened and nothing hangs on it but the report.

### 4.8 Characters and profiles

`GET /users/:id` gains: characters (name, level, editable by their player), **reports written**, and
**report owed** where one is. The reports-written count is the whole of the reward the platform can
offer; the map is explicit that the mechanism which actually works — a reward at the table — is out
of scope, so the platform records the debt and a Game Master pays it (ticket `07`).

No standing is shown on a profile. Standing is per candidate slot and means nothing away from one.

### 4.9 Search

The way back to the trail, and load-bearing rather than convenient (ADR 0004).

- **Reports get their own FTS index and their own result group, above messages** (ticket `08`).
  Search then answers "what do we know about the Tomb" from the reports and "who said what while we
  planned it" from the messages, without one drowning the other. No live/finished toggle is needed —
  the grouping subsumes it.
- Messages keep their existing FTS index, and `last(100)`
  (`app/controllers/searches_controller.rb:23`) is replaced by ordered, paged results using
  `geared_pagination`, which is already in the `Gemfile`.
- **Relevance ranking is not solved here**, and the limit is named rather than hidden: results are
  newest-first within each group. See [§8](#8-non-goals-and-stated-limits).

### 4.10 Rooms

Campfire's chat, kept and re-pointed. Three kinds of room exist and only three:

- **A quest's room.** `Rooms::Open`, created with the quest, open to the whole campaign for its
  whole life, **never narrowed to the party** — narrowing means revoking memberships, which fires a
  websocket kick per destroyed membership and would eject, at the moment of confirmation, exactly
  the waitlist who need to keep watching for a freed seat.
- **The Tavern.** The one campaign room, where the campaign talks when no quest is at stake. The
  platform speaks here about three things only: a new quest, a new slot, a new report.
- **The Herald.** One `Rooms::Direct` per player, between them and the bot user, which the player
  cannot write into and which never freezes.

**The sidebar** carries the campaign rooms, then live quest rooms only — `proposed`, `confirmed`,
`played` — ordered by **soonest candidate slot**, with slotless quests below by proposal date. This
replaces `scope :ordered, -> { order("LOWER(name)") }` (`app/models/room.rb:29`), which sorts the
campaign's agenda by the alphabet. The live set is bounded by campaign tempo, not by history, which
is the whole reason the sidebar keeps working at a year in. It is a `where` on the sidebar query,
**not** per-player `invisible`, which is a preference and must not be silently consumed by freezing.

**A frozen room** — its quest `closed`, `withdrawn` or `abandoned` — refuses new messages, new
boosts, and edits or deletes of existing ones. An archive that can be quietly rewritten is not an
archive. Three guards, and two of them do not go through `RoomScoped`, which is how they get
missed:

| Path | Guard |
|---|---|
| `MessagesController#create` | the room's quest is live |
| `Messages::BoostsController` (`create`, `destroy`) | same, resolved via `Current.user.reachable_messages` |
| `MessagesController#edit/update/destroy` | same, in addition to `can_administer?(@message)` |

**A frozen room's unreads stop counting.** Ticket `08` asked for `unread_at` to be *cleared* across
the room's memberships, hung off the quest's transition — which ADR 0007 later made impossible for
two of the three triggers, because closing and abandonment happen because time passed and have no
writer. So the badge is **derived** instead: every unread count excludes memberships whose room's
quest is terminal (`user.memberships.unread` gains that scope, and it is the count behind
`Push::Subscription`). This meets what `08` actually wanted — the app-icon badge counts unread
*rooms*, so without it players would carry a permanent number pointing at a room they can no longer
open to clear — and it needs nothing to run.

The composer is replaced by one line saying the quest is over, linking to its report — or to the
quest itself where there is none.

**Involvement follows interest.** Quest rooms keep the inherited `mentions` default. Declaring
interest raises **your own** membership to `everything`; confirmation raises the slot owner's. **The
platform only ever raises an involvement, never lowers it** — a player who set the bell by hand and
later withdrew interest has said something the platform must not quietly reverse. Muting without
withdrawing already works, through the existing `Rooms::InvolvementsController`.

**Player-to-player direct rooms survive, unchanged.** No ticket ever owned this and ADR 0007
depends on the answer, so it is settled here: Campfire's DMs stay. The Herald is a `Rooms::Direct`,
so the model has to survive regardless; removing the feature is work that buys nothing; and private
lobbying cannot buy a seat from a binding rota, which is precisely why this is cheap to allow here
and would not be in a product where a human picks the party.

### 4.11 Administration

The inherited account screens, with four changes: the **join code is visible to administrators
only** (roster growth should be a deliberate act, not a link any player can forward); the **ban
button is gone** from the interface and its routes removed; **deactivate** and **reactivate**
are the only exits and entrances; and the roster page carries the `game_master` toggle.

Reactivation is a real path, not a status flip: restore `active`, clear `deactivated_at`, and
**re-grant memberships to every Open room** — `Rooms::Open#grant_to(User.active)` means flipping the
status alone never restores access, and under ticket `08` every quest room is an Open room.

---

## 5. Notifications

No notification system is built and no mail is sent. The fork has **one channel** — a message in a
room — and **no email** anywhere in it. A `Notification` model would need an inbox, read state, a
delivery path and a mute story, every one of which a room already has (ADR 0007).

**The Tavern for the campaign's news, the quest's room for that quest's life, The Herald for your
own.**

### 5.1 The matrix

| Event | Who hears | Surface | Pushes |
|---|---|---|---|
| A quest is proposed | the campaign | The Tavern | no |
| A slot is offered | the campaign | The Tavern | no |
| Interest is declared | — | silence | — |
| A quest is pointed at your slot | — | silence | — |
| A slot's deadline passes | the slot's Game Master | The Herald | yes |
| A quest is confirmed | the campaign | the quest's room | interested only |
| → you are in the party | you | The Herald | yes |
| → you are on the waitlist, at *n* | you | The Herald | yes |
| → your last live date on another quest went | you | The Herald | yes |
| → you still have live dates there | — | silence | — |
| A slot is withdrawn, and it was your last live date | you | The Herald | yes |
| A seat is given back; the party changes | — | silence | — |
| You are promoted off the waitlist | you | The Herald | yes |
| A confirmed quest is cancelled | the campaign | the quest's room | interested only |
| A confirmed quest is cancelled | the party **and** the waitlist | The Herald | yes |
| A quest becomes played | — | silence | — |
| The scribe is asked, and reminded at day 11 | the scribe | The Herald | yes |
| A report is published | the campaign | The Tavern + the quest's room | no |
| A quest is withdrawn or abandoned | the proposer and everyone who declared interest | The Herald | yes |
| Attendance is corrected | — | silence | — |

Everything marked *silence* is visible as state on a screen; [§4.1](#41-the-board--the-campaigns-front-page)
and [§4.2](#42-the-quest-page) between them carry all of it. That is the constraint, not a
courtesy.

Un-pointing a quest from a slot ([§3.10](#310-permissions)) takes the "slot is withdrawn" row: the
event is *your last live date on this quest went*, whatever caused it.

### 5.2 How messages are composed

- **System messages in a quest's room mention nobody.** Under this split they are about the quest,
  never about a person, so `@`-mentions stay purely the human affordance they were inherited as.
- **The Herald sends one message per act, per player**, composed from every consequence that act
  had for them — not one per event. A single confirmation can hit one player four ways at once, and
  four pushes in a second read as a broken application:
  > *"Tuesday 7pm went to The Sunken Mill — you're seated, with Vess. Your interest in Blackfen
  > ended when that date went."*
  Copy is therefore a function of `(act, player)` rather than a template per event type.
- **Notifications name times**, rendered server-side in `config.time_zone`. A message saying a quest
  was confirmed and not saying when is useless, and `local_time_controller.js` cannot reach a push
  payload.
- **The platform never overrides a player's own preference.** Loudness is set by defaults, never by
  overrides — decisively, because a denied browser permission is outside our reach anyway, so
  "unmutable" would be a promise we could not keep.

### 5.3 The bot, and the Herald

A `User` with `role: :bot`, named **The Herald**. The webhook and HTTP posting paths are not needed
because the platform writes server-side. It must be kept out of `without_bots` listings, out of the
sidebar's direct-room placeholder list (`User.active` → `User.active.without_bots` in
`Users::SidebarsController`), and it must never hold a character, declare interest, or appear on a
rota.

A player's Herald room is the `Rooms::Direct` between them and the bot, created on demand
(`Rooms::Direct.find_or_create_for`). It inherits `default_involvement` of `everything`, so it
pushes, and unread state, history and a mute control all come free. It refuses player writes — the
same write-refusal as a frozen room, without the sidebar half: the Herald keeps its place.

### 5.4 The clock

A scheduler enters the `Gemfile` — `resque-scheduler`, matching the Resque queue already in
`config/environments/production.rb`. One recurring job, every ten minutes, whose whole
responsibility is to notice what has newly become true and say so:

| It notices | It sends | Ledger key |
|---|---|---|
| A slot's deadline has passed, unspent and unwithdrawn | The Herald, to the slot's Game Master, naming how many quests want the date | `slot:<id>:ripe` |
| A confirmed quest's slot has ended | The Herald, to the scribe, with the template and the deadline | `quest:<id>:played` |
| Day 11 of a report clock, still unwritten | The Herald, to the scribe, naming the three days left | `quest:<id>:chase-11` |
| A proposed quest has passed its abandonment date | The Herald, to the proposer and everyone who declared | `quest:<id>:abandoned` |

Then silence: the obligation lapses at 14 days, the debt does not spread, and the scribe is never
chased again. Nothing public — a public chase is shaming rather than the reward that actually works.

The **ripe slot** is the event nobody had listed. ADR 0005 gates confirmation on a deadline and ADR
0006 abandons quests fourteen days later; between them they created a failure with no owner, and
this row is what gives every confirmation in the campaign a trigger instead of relying on somebody
remembering to look.

Everything else is sent by the act that caused it — confirmation, cancellation, slot withdrawal,
un-pointing, release-and-promotion, publication, quest withdrawal — inside the same transaction's
`after_commit`.

---

## 6. What the fork changes, and what it leaves alone

Hard fork, no upstream tracking, single-tenant (ADR 0001). The genuinely expensive parts — real-time
rooms, mentions, attachments, search, Web Push, a working Docker deployment — already exist and are
exactly what the arguing around an expedition needs.

### 6.1 Changed

| File / area | Change | Source |
|---|---|---|
| `db/schema.rb` | Ten new tables ([§2.2](#22-new-tables)); `users.game_master`, `users.deactivated_at`, `rooms.quest_id`; a `report_search_index` FTS table | this spec |
| `app/models/user.rb` | `deactivate` stops mangling `email_address`, records `deactivated_at`; `reactivate` restores status and re-grants Open room memberships | ticket `05` |
| `app/models/user/role.rb` | untouched. `game_master` is a column, never a `role` value | ticket `02` |
| `app/models/user/bannable.rb` | **unreachable**: `Users::BansController` and its routes are removed, so nothing calls `ban`. The model and `BlockBannedRequests` stay in place, unused — ban destroys every message a person ever wrote, which under ADR 0004 is the campaign's memory | ticket `05` |
| `app/models/room.rb` | `belongs_to :quest, optional: true`; `scope :ordered` replaced by the sidebar ordering in [§4.10](#410-rooms); write guards consult the quest's state | ticket `08` |
| `app/models/rooms/open.rb` | unchanged. Quest rooms rely on `grant_access_to_all_users` exactly as inherited | ticket `01` |
| `app/models/first_run.rb` | `FIRST_ROOM_NAME = "The Tavern"`; account name is the campaign's | ticket `08` |
| `app/models/account.rb` | `restrict_room_creation_to_administrators` defaults to **true** | ticket `08` |
| `app/models/account/joinable.rb` | unchanged; the join code's *display* is restricted to administrators in the view | ticket `05` |
| `app/controllers/messages_controller.rb` | `create`, `edit`, `update`, `destroy` refuse in a frozen room | ticket `08` |
| `app/controllers/messages/boosts_controller.rb` | `create`, `destroy` refuse in a frozen room | ticket `08` |
| `app/controllers/rooms_controller.rb` | quest rooms have no delete route; `can_administer?`'s creator clause must never reach one | tickets `01`, `08` |
| `app/controllers/users/sidebars_controller.rb` | live quest rooms only, ordered by soonest slot; placeholder users exclude bots | tickets `08`, `09` |
| `app/controllers/searches_controller.rb` | reports group above messages; `last(100)` replaced by paged results | tickets `08`, `13` |
| `config/routes.rb` | `resources :quests`, `:slots`, `:characters`, nested `:interests`, `:placements`, `:report`, `:attendance`; root becomes The Board; the dead `resource :settings` route (`config/routes.rb:75`) is deleted; `resource :ban` removed | tickets `08`, `05` |
| `config/application.rb` | `config.time_zone` set to the campaign's zone | ticket `04` |
| `Gemfile` | `resque-scheduler` | ADR 0007 |
| Copy throughout | Campfire's vocabulary replaced by `CONTEXT.md`'s | ADR 0001 |

### 6.2 Left alone

Messages, boosts, mentions, attachments, unfurling, the composer, sounds, avatars, the PWA and its
service worker, Web Push and its subscription management, the search index for messages, sessions
and authentication, transfers, bots as an inherited concept, `Rooms::Closed`, `Rooms::Direct`, the
QR join flow, custom styles, and the Docker deployment. Nothing here is rebuilt, and rebuilding chat
as a quest-page thread was considered and rejected: the chat is the one thing this fork actually
inherits (ticket `08`).

Three inherited bugs are repaired on the way past, because decisions in this spec rest on them
(ticket `05`): `DELETE /users/:id/ban` un-deactivating a user, `User.administrator.first` being
unscoped in `app/views/accounts/_help_contact.html.erb`, and the account UI's "permanently remove"
copy describing an action that in fact deactivates.

---

## 7. Slices

Ordered. Each one is shippable, each one leaves the campaign better off than it found it, and the
things it does not yet do are done in chat, as they are today.

**1 — The roster and the furniture.** `game_master` and `deactivated_at`; deactivation and
reactivation repaired; ban removed; join code restricted; characters; The Tavern; Open room creation
restricted to administrators. *After this the campaign is Campfire with a roster that means
something.*

**2 — Quests and their rooms.** The quest model and its six-state derivation (with only proposal and
withdrawal reachable), proposal, the quest page's header and description, quest-owns-room, the
sidebar re-ordered, freezing derived and guarded. *Quests exist and are argued over; scheduling is
still a conversation.*

**3 — The pool.** Slots, candidacies, interests, declarations. The per-slot cards on the quest page
showing who can make what, with deadlines. No cut yet. *This is already better than a spreadsheet,
and it is the earliest honest stopping point.*

**4 — The rota and confirmation.** Standing, bumps, the confirmation screen, placements, party and
waitlist, releases and automatic promotion, cancellation. **This is the defensible stopping point.**
The coordination problem the platform exists to solve is solved: dates are offered, interest is
declared, parties are cut by a rule that shows its working, and nobody has to be fastest to refresh.
Everything after this is memory and legibility.

**5 — The clocks.** `played`, `closed` and `abandoned` derived; rooms freeze on all three terminal
states; frozen rooms drop out of the unread counts. No job yet — these are comparisons. *Dead
quests stop accumulating.*

**6 — The report.** The report model and template, the derived scribe, attendance corrections, the
reports FTS index and its search group, "no report" for good. *The campaign starts remembering.*

**7 — The Board.** Every silence made visible in one place. Deliberately before notifications: the
constraint in [§0](#0-how-to-read-this-and-what-wins) says the screen has to exist for the messages
to be safe.

**8 — The three surfaces.** The bot user, The Herald, Tavern announcements, system messages in quest
rooms, involvement following interest, `resque-scheduler` and the recurring job with its ledger.
*The campaign stops depending on people remembering to look.*

**9 — Search.** Paging and scoping for messages, on top of slice 6's report group.

---

## 8. Non-goals and stated limits

Ruled beyond this destination by the map. These do not graduate: revisiting one means redrawing the
destination as a fresh effort.

- **The discovered world map.** Hexes, locations, fog-of-war. A genuinely different product with
  spatial data and image handling, ruled out on those merits.
- **The synthesis** — leads with state, a lore index, an accumulated picture of what is known. The
  one artefact this format has never sustained. Retrieval is our answer instead (ADR 0004). The most
  valuable thing in ticket `03`'s research that we are deliberately not building, and the strongest
  candidate for a future effort with its own destination.
- **Anything at the table.** Dice, sheets, stats, inventory, initiative, rules content, VTT
  features.
- **Calendar sync.** Real and tempting, but an integration, not a domain decision. It can follow.
- **Payments.** **A public campaign website.** **Multi-tenancy** (ADR 0001). **Tracking upstream.**

And five limits this spec states outright, because each is a gap an implementation session will meet
and helpfully close:

1. **The campaign has no world model.** No locations, no accumulated state, no wiki. A report is one
   party's account of what they believe they saw, and nothing reconciles two that disagree. The
   audience for this sentence is not players — it is whoever meets the gap and builds a wiki to fill
   it. Written down it is a decision; unwritten it is an oversight someone will correct (ADR 0004,
   ticket `13`).
2. **The gaps are permanent and visible.** An unreported quest reads "no report" for good. A record
   showing its own holes is preferred to one that hides them.
3. **The reward that makes reports work is out of scope.** The platform records the debt; a Game
   Master pays it at the table. This is the honest limit of the design, not an oversight.
4. **An out-of-range character can take a seat and nothing in the app can stop it.** Level is never
   compared, the cut is blind to it, and there is no override.
5. **Relevance ranking in search is not solved** ([§4.9](#49-search)), and search is load-bearing.
   Newest-first within each group is the floor, not the answer.

Four things remain genuinely unspecified, and none of them blocks a slice. They are the map's
leftovers, recorded here so nobody mistakes them for decisions:

- **Character death and retirement.** No state is held: a retired character is one nobody declares
  with. Whether the platform should say more is a fresh question.
- **A pacing device for Game Masters.** Adventuring blocks, retrospectives, enforced downtime. Every
  long-lived campaign has one; ours has none, and burnout is the documented cause of death
  (contradiction 11).
- **Multi-Game-Master continuity.** Several Game Masters share one world with no notion of who has
  claimed which part of it. A Game-Masters-only room is cheap to scope now that the flag exists, and
  belongs to that effort rather than to this one.
- **Life at a year in.** Hundreds of played quests and their reports may need paging or retention on
  their own listing pages. The sidebar no longer degrades; the listing pages are untested at scale.

---

## 9. Decisions this spec makes

Everything else here is carried from a ticket or an ADR. These are not, and they are listed so they
can be argued with.

**Ratifications** — invented in a prototype, accepted, previously in no ADR:

1. **The scribe derivation** ([§3.8](#38-the-scribe)) — longest without writing one, never-scribed
   first, ties by rota position. From ticket `11`. It **amends ticket `07`** twice: the scribe is
   derived rather than named, and when attendance removes the scribe the rule re-derives instead of
   asking a Game Master to name a replacement.
2. **Contender ordering** ([§3.9](#39-contender-ordering)) — best party first, with the "only live
   date" flag doing the work the sort order does not. From ticket `11`.
3. **The 72-hour deadline as a derived value**, not a per-slot field. ADR 0005 says the Game Master
   sets it; they set it by choosing the start.

**New rulings**, made here because the spec cannot be written without them:

4. **The scribe is derived at read time over the attendance, not stored at confirmation.** Ticket
   `11` and `CONTEXT.md` say "derived at confirmation"; storing it would need re-derivation hooks on
   release, promotion and attendance correction — three places to drift. The name in the
   confirmation dialog is a forecast, and the screen says so. `CONTEXT.md`'s **Scribe** is amended.
5. **Player-to-player direct rooms stay.** ADR 0007 depended on an answer nobody owned. The model
   must survive for the Herald regardless, removal is work that buys nothing, and private lobbying
   cannot buy a seat from a binding rota.
6. **The party-size cap lives on the quest**, not the slot, and any Game Master may edit it. It is
   what the player's page shows and what the proposer sets; a per-slot cap would need a second
   number nobody has asked for.
7. **Running a quest is not a seat** ([§3.1](#31-the-rota)). Derived from the rota's own keys, with
   the consequence named.
8. **The abandonment anchor is the latest deadline among all candidate slots**, spent and withdrawn
   included ([§3.7](#37-the-clocks)). Deliberately generous.
9. **Attendance is derived until a Game Master corrects it**, then materialised. Keeps the
   correction an act and keeps the clock's job to telling.
10. **An `announcements` ledger** ([§2.2](#22-new-tables)) is how the recurring job records what it
    has said. Nothing depends on a row existing, so a dead worker still loses only messages.
11. **The Board is the front page**, and it is named. Added to `CONTEXT.md`.
12. **Permissions in detail** ([§3.10](#310-permissions)) — the map's open item, settled in full.
13. **The rota's copy names seats, not play.** "Never played" is wrong under ADR 0005; the
    prototypes' wording is corrected wherever it appears.
14. **Characters have no lifecycle**, and are never deleted.
15. **Cancellation is possible only before the slot starts** ([§3.6](#36-cancellation)). ADR 0006
    says any Game Master may cancel a confirmed quest and does not bound it; cancelling mid-play
    would destroy placements that had already been used, and after the slot ends the quest is
    played rather than confirmed anyway. Attendance is the record of an evening that happened.
16. **A frozen room's unreads are excluded from the badge rather than cleared**
    ([§4.10](#410-rooms)). Ticket `08` asked for a write hung off the transition; two of the three
    triggers are clocks with no writer, so the count is derived instead. Same outcome, nothing to
    run.

---

## 10. Provenance

| Source | Where it lands |
|---|---|
| ADR 0001 — hard fork, single-tenant | [§6](#6-what-the-fork-changes-and-what-it-leaves-alone), [§8](#8-non-goals-and-stated-limits) |
| ADR 0002 — a quest is a single expedition | [§1](#1-the-campaign-in-one-page), [§2.2](#22-new-tables) |
| ADR 0003 — three acts, amended | [§1](#1-the-campaign-in-one-page), [§3.5](#35-confirmation) |
| ADR 0004 — a trail, not a world | [§4.6](#46-the-report), [§4.9](#49-search), [§8](#8-non-goals-and-stated-limits) |
| ADR 0005 — the rota | [§3.1](#31-the-rota)–[§3.5](#35-confirmation) |
| ADR 0006 — the slot pool and the quest's life | [§2.2](#22-new-tables), [§3.6](#36-cancellation), [§3.7](#37-the-clocks) |
| ADR 0007 — nothing is only a notification | [§0](#0-how-to-read-this-and-what-wins), [§5](#5-notifications) |
| `01` rooms, `02` identity, `03` practice (research) | [§6](#6-what-the-fork-changes-and-what-it-leaves-alone), [§4.10](#410-rooms), and the reasoning throughout the ADRs |
| `04` timezones | [§2.2](#22-new-tables), [§2.4](#24-constants), [§5.2](#52-how-messages-are-composed) |
| `05` roles and roster | [§3.10](#310-permissions), [§4.11](#411-administration) |
| `06` quest lifecycle edges | ADR 0006 → [§3.4](#34-seats-releases-and-promotion)–[§3.7](#37-the-clocks) |
| `07` report lifecycle | [§4.6](#46-the-report), [§4.7](#47-attendance), amended by [§9](#9-decisions-this-spec-makes) |
| `08` room lifecycle | [§4.10](#410-rooms), [§4.9](#49-search) |
| `09` notification matrix | ADR 0007 → [§5](#5-notifications) |
| `10` quest page prototype | [§3.2](#32-standing), [§4.2](#42-the-quest-page) |
| `11` confirmation screen prototype | [§3.8](#38-the-scribe), [§3.9](#39-contender-ordering), [§4.5](#45-the-confirmation-screen) |
| `13` does the report carry the world | ADR 0004 → [§8](#8-non-goals-and-stated-limits) |
| `14` can a Game Master propose | [§1](#1-the-campaign-in-one-page), [§3.10](#310-permissions) |
| `15` who cuts the party | ADR 0005 → [§3.1](#31-the-rota) |

Both prototypes were folded into [§4.2](#42-the-quest-page) and [§4.5](#45-the-confirmation-screen)
and then deleted, as ticket `12` required. They existed to be reacted to, and they were.
