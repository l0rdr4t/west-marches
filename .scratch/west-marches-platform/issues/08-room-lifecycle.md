# What happens to a quest's room when the quest is over?

Type: grilling
Status: resolved
Blocked by: 01

## Question

Every quest gets an Open room at proposal (Q15, Q17). Quests are single-use and short-lived
(ADR 0002). A campaign running weekly produces a room a week at minimum, and more from
proposals that never get scheduled.

- What happens to the room when its quest is played, abandoned, or declined? Deleted,
  archived, left alone?
- If rooms persist, how does the sidebar stay usable at a hundred quests? Should quest
  rooms appear in it at all, or only be reachable from the quest?
- Should a played quest's room stay writable, so people can keep arguing about what
  happened, or close to new messages once the report lands?
- Do the campaign-wide rooms Campfire ships with survive as general chat alongside the
  quest rooms?
- Is the room genuinely open to the whole campaign, including people not on the quest and
  not interested in it? (Q15 assumed yes, on the grounds that open planning is the point.)
- Does anyone need to mute or leave a quest room without withdrawing their interest?

Read the findings from ticket `01` first — what Campfire can already do here decides how
much of this is a decision and how much is a build.

## Answer

**A quest's room outlives its quest but stops being a live surface.** Nothing is ever deleted
automatically. When a quest reaches a terminal state the room **freezes**: it accepts no further
writes, drops out of every sidebar, and becomes an archive reachable from the quest. Freezing is
*derived from the quest's state*, not stored on the room, so there is no flag to keep in sync and
no unfreeze mechanism to build — reopening the quest reopens the room.

The premise from Q15/Q17 survives intact: every quest gets a `Rooms::Open` at proposal, open to
the whole campaign, for its whole life. Rebuilding chat as a quest-page thread was considered and
rejected — the chat is the only thing this fork actually inherits, and ticket `01` showed the
model is cheap while it is *navigation* that breaks. So: fix navigation, don't rebuild chat.

### The room never narrows and never dies

- **Open to everyone, always.** Never narrowed to the party at confirmation. Narrowing means
  revocation, and each destroyed membership fires a websocket kick (`app/models/membership.rb:7`) —
  ejecting, at the exact moment of confirmation, the **waitlist**, who are precisely the people
  who need to keep watching for a freed seat. A **report** readable by the whole campaign on top
  of a room readable by six is incoherent.
- **No automatic deletion, ever.** ADR 0002 makes the report the durable record of the *world*,
  but the room holds the *planning* — why this quest, who argued for which slot — which the report
  deliberately doesn't carry, and deletion is synchronous and irreversible
  (`app/controllers/rooms_controller.rb:14-19`).
- **Declined and abandoned quests behave identically to played ones.** A proposal that drew no
  interest is itself campaign information. One rule, one trigger, no special cases.
- **A mid-campaign joiner gets the whole history.** `grant_membership_to_open_rooms`
  (`app/models/user.rb:22,53-54`) already hands a new player every Open room that exists; keep it.
  Being able to search what the campaign already learned is the onboarding a West Marches campaign
  wants. Ticket `01` priced the rows as negligible.

### Freezing

Derived from quest state. A room with no quest — **The Tavern** — is never frozen.

Frozen forbids **new messages, new boosts, and edits or deletes of existing messages**. An archive
that can be quietly rewritten isn't an archive. Three separate guards, and two of them don't go
through `RoomScoped`, which is how they get missed:

- messages: `MessagesController#create`
- boosts: resolved via `Current.user.reachable_messages` (`app/controllers/messages/boosts_controller.rb:27`)
- edits/deletes: gated only by `can_administer?(@message)` (`app/controllers/messages_controller.rb:6`)

**Freezing clears `unread_at` across the room's memberships**, hung off the quest's transition. Without
this the room leaves the sidebar with unreads still set and becomes a phantom: the push badge is
`user.memberships.unread.count` (`app/models/push/subscription.rb:5`) and the app-icon badge counts
unread rooms (`app/javascript/controllers/badge_dot_controller.js:24-26`), so players would carry a
permanent number pointing at a room they can no longer open to clear.

The composer is replaced by one line saying the quest is over, linking to its **report** — or to the
quest itself where there is none, which is the declined and abandoned case. It gives the room a way
back out to the durable record.

### Navigation

- **The sidebar carries live quest rooms only**, campaign-wide — a `where` on the sidebar query
  (`app/controllers/users/sidebars_controller.rb:5`), *not* per-player `invisible`, which is a
  preference and shouldn't be silently consumed by freezing. The live set is bounded by campaign
  tempo (a handful at once), not by history, which is what makes the sidebar work.
- **No filtering by stake.** Hiding proposals from uninterested players hides exactly the quests
  that need eyes on them to attract interest.
- **Ordered by soonest candidate slot**, quests with no slots yet below by proposal date. Replaces
  `scope :ordered, -> { order("LOWER(name)") }` (`app/models/room.rb:29`), which sorts the campaign's
  agenda by the alphabet. Grouped under a heading, below the campaign rooms.
- **A frozen room is reached by a link from its quest page**, labelled with its message count. Not
  embedded below the report — that buries the thing meant to be read. No separate archive index:
  the list of past quests is that index, and search is the other route in.

### Involvement

Default is `mentions` (`app/models/room.rb:65-67`, `db/schema.rb:86`), which means a player who
declared **interest** hears nothing about the one quest they're trying to get on.

- **Declaring interest raises that membership to `everything`; withdrawing drops it back.** Same for
  the **party** and the Game Master running it.
- **Muting without withdrawing already works** — set the room to `nothing` via the existing
  `Rooms::InvolvementsController`. Interest and involvement are separate records; nothing to build.
- Note `nothing` still bolds the sidebar row — only `invisible` escapes `unread_memberships`
  (`app/models/room.rb:79-81`). That's correct here: it's a quest you asked to go on.
- Push copy and the wider matrix stay ticket `09`'s.

### Search

Resolves the map's open **Search and history** item. `SearchesController` runs FTS over
`Current.user.reachable_messages` capped at `last(100)` (`app/controllers/searches_controller.rb:23`,
`app/models/user.rb:7`). With rooms immortal that is eventually every message ever sent, so a year in,
searching a monster's name returns planning chatter from forty dead quests.

**Reports get their own FTS index and their own result group, above messages.** Search then answers
"what do we know about the Tomb" from the reports and "who said what while we planned it" from the
messages, without one drowning the other. No live/finished toggle is needed — the grouping subsumes it.

### Room creation and ownership

- **Restrict Open room creation to administrators** — flip `restrict_room_creation_to_administrators`
  to default `true` (`app/models/account.rb:9`; enforced at `app/controllers/rooms_controller.rb:41`,
  button at `app/views/users/sidebars/show.html.erb:39`). Everything above rests on the sidebar holding
  fixed furniture plus a bounded live set; a player-created Open room lands permanently in forty
  sidebars with no quest behind it and therefore nothing that can ever freeze it. Directs stay
  unrestricted.
- **Exactly one campaign room: rename "All Talk" (`app/models/first_run.rb:3`) to "The Tavern".** Where
  the campaign talks when no quest is at stake. Resist adding a notice board — the sidebar's live quest
  list already is one. A Game-Masters-only room may be warranted but is ticket `05`'s to raise.
- **The quest owns its room: `has_one :room, dependent: :destroy`.** A quest room has no independent
  existence and has **no delete route at all**; an administrator deletes the *quest*. This is what
  disarms ticket `01`'s landmine — `can_administer?` grants room deletion to the room's creator
  (`app/models/user/role.rb:8-10`), so as things stand a quest's proposer owns an irreversible button
  that erases the campaign's whole discussion of it. Campaign rooms keep ordinary administrator delete.
- **Delete the dead `resource :settings` route** (`config/routes.rb:75`, no `Rooms::SettingsController`).
  Ticket `01` flagged it as the natural home for lifecycle controls; freeze being derived and quest
  rooms having no delete means the surface it was looking for turned out not to be needed.

### Handed on

- **The freeze trigger** — which event ends a quest for good (report published, abandoned, declined) is
  ticket `07`'s. This ticket specifies the shape and defers the trigger.
- **Abandonment must be a real terminal state that something actually sets** — ticket `06`'s. Without it
  dead proposals never freeze and slowly fill the sidebar, which is the one way the navigation decision
  above fails.
- **A deactivated player's messages stay** in frozen rooms; `deactivate` deletes non-direct memberships
  but not messages (`app/models/user.rb:39`). Deactivation removes access, not history. Ticket `05` owns
  whether that's the right call.
