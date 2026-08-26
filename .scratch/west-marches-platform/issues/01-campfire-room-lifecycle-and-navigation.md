# Campfire's room model, lifecycle and navigation at scale

Type: research
Status: resolved
Blocked by: -

## Question

Every quest gets a room (ADR 0002 makes quests numerous and short-lived), so we need to
know what Campfire actually gives us before deciding what happens to a quest's room when
the quest is over.

Answer, from this codebase:

- How is a room created, and what does creating one cost? (`Room`, `Rooms::Open`,
  `Rooms::Closed`, `Room.create_for`, `memberships.grant_to`)
- How does membership work — `involvement` levels, defaults, what "everyone is in an Open
  room" means in practice, and whether an Open room actually grants memberships to all
  users or resolves them dynamically.
- How does the sidebar list rooms? What orders it, what filters it, is there any paging?
  What does it look like with 200 rooms?
- Is there any existing notion of archiving, hiding, closing or deactivating a room? Or is
  deletion the only exit?
- What is deleted along with a room (messages, attachments, searches, push subscriptions)?
- How do unreads and notification badges work per room, and would 200 quest rooms produce
  200 badges?

Deliverable: findings written to `.scratch/west-marches-platform/research/01-rooms.md`,
with file and line references.

## Answer

Findings: [`research/01-rooms.md`](../research/01-rooms.md), with file:line references throughout.

**Campfire has no room lifecycle at all.** A room exists from creation until someone deletes
it. The `rooms` table is five columns (`db/schema.rb:119-125`) — no status, no archive, no
soft-delete, no read-only mode. There is no ordering by activity, no paging, no filter box, and
no way to search for a room by name. `Rooms::Closed` means *restricted membership*, not *finished*.

**Creating a room per quest is cheap.** `Room.create_for` (`app/models/room.rb:33-40`) is one
INSERT plus a bulk membership insert. `Rooms::Open` materialises a real membership row per user
(`app/models/rooms/open.rb:6-8`) — nothing resolves openness dynamically — so 200 quests across
40 players is 8,000 rows, which is nothing. Default involvement is `mentions`
(`app/models/room.rb:59-61`), so a new quest room pushes only to mentionees. No spam.

**What breaks is the sidebar, and it breaks hard.** One flat alphabetical list, no limit, no
paging, no grouping, no filter (`app/controllers/users/sidebars_controller.rb:5-10`), rendered
with an *uncached* partial per room (`app/views/users/sidebars/show.html.erb:34-36`) while the
directs beside it are cached. It re-fetches on every cable reconnect and after 60s hidden. Read
and unread events do O(rooms) DOM scans.

**Unread badges count rooms.** The PWA app-icon badge is a count of unread *rooms*
(`app/javascript/controllers/badge_dot_controller.js:24-26`); the push badge is
`user.memberships.unread.count` (`app/models/push/subscription.rb:5`). Involvement does not gate
unread marking except `invisible` (`app/models/room.rb:74-76`), and there is no "mark all read".
200 live quest rooms means 200 bolded rows and `200` on the app icon.

**There is nothing to reach for.** The only hide is per-player `invisible`, four clicks per room,
no bulk and no admin version. The tempting "revoke everyone" trick is a trapdoor, not an archive:
authorization is purely the join table with no admin backdoor (`app/controllers/rooms_controller.rb:33`),
so a memberless room is permanently unreachable by everyone. The `room_settings` route
(`config/routes.rb:75`) points at a controller that does not exist, so there is no per-room admin
surface to hang a lifecycle control on.

**Deletion works but is a cliff.** It cleans up completely — messages, boosts, rich text,
attachments, FTS rows — but it is synchronous and irreversible
(`app/controllers/rooms_controller.rb:14-19`). And `can_administer?` grants the room's *creator*
delete rights (`app/models/user/role.rb:8-10`), so a quest's proposer would own the button that
erases the whole discussion.

Two couplings for `08`:
- Deleting quest rooms is only safe once reports exist — ADR 0002 already makes the report the
  sole durable record of the world.
- Scoping a room to the party rather than the campaign means `Rooms::Closed` plus revocation,
  which fires a websocket kick per membership destroyed (`app/models/membership.rb:7`).
