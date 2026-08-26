# Campfire's room model, lifecycle and navigation at scale

Research for issue `01-campfire-room-lifecycle-and-navigation`. Every claim below is
cited to this repo at `main` (e63402d). Paths are relative to the repo root.

Vocabulary note: Campfire's `Room` is what `CONTEXT.md` calls a **Room** (a quest's
chat room); its `Account` is our **Campaign**; its `User` is a **Player**. Campfire's
`Rooms::Closed` means "restricted membership", not "finished" — I use "closed-type"
below to keep that distinct from a quest being over.

Headline: **Campfire has no room lifecycle at all.** A room is created and it exists
until someone deletes it. There is no archived state, no closed state, no ended state,
no hidden state, no ordering by activity, no paging, no filtering, and no way to search
for a room by name. Everything the downstream ticket wants is a build.

---

## 1. How is a room created, and what does creating one cost?

### The three types

Single-table inheritance on `rooms.type` (`db/schema.rb:119-125`). The whole table is
five columns: `created_at`, `creator_id`, `name`, `type`, `updated_at`. No counter
caches, no `last_message_at`, no `archived_at`, no status of any kind.

- `app/models/rooms/open.rb:2` — `Rooms::Open`, "open to all users on the account"
- `app/models/rooms/closed.rb:2` — `Rooms::Closed`, "only a subset ... explicitly
  granted membership"
- `app/models/rooms/direct.rb:3` — `Rooms::Direct`, DM singleton

Scopes for each at `app/models/room.rb:24-27`. The only ordering scope on `Room` is
`app/models/room.rb:29` — `scope :ordered, -> { order("LOWER(name)") }`. **Alphabetical
by name is the only ordering Campfire knows for rooms.** There is no recency ordering
for shared rooms anywhere in the app (directs get one, see §3).

### `Room.create_for`

`app/models/room.rb:33-40`:

```ruby
def create_for(attributes, users:)
  transaction do
    create!(attributes).tap do |room|
      room.memberships.grant_to users
    end
  end
end
```

`grant_to` is defined on the association at `app/models/room.rb:3-6`: a single
`Membership.insert_all` of `{ room_id:, user_id:, involvement: room.default_involvement }`
rows. `insert_all` (not `insert_all!`) skips conflicts, so re-granting an existing
member is a no-op rather than an error against the unique index on
`[room_id, user_id]` (`db/schema.rb:92`). This is load-bearing and proven by
`test/models/rooms/open_test.rb:30-34`, which converts the `watercooler` fixture — which
already has memberships (`test/fixtures/memberships.yml:26-37`) — into an Open room and
expects the grant-to-everyone callback to succeed.

`creator` defaults to `Current.user` (`app/models/room.rb:20`) and is `null: false`
(`db/schema.rb:121`). This matters for permissions — see §4.

### An Open room grants to everyone on `after_save_commit`

`app/models/rooms/open.rb:3-8`:

```ruby
after_save_commit :grant_access_to_all_users

def grant_access_to_all_users
  memberships.grant_to(User.active) if type_previously_changed?(to: "Rooms::Open")
end
```

Note the guard: it fires **only when the type changes to `Rooms::Open`** — i.e. on
create, or on conversion from closed-type. It does not fire on a rename or any other
update. Users who join the campaign later are caught by the other half of the pair,
`app/models/user.rb:22` / `:53-55`:

```ruby
after_create_commit :grant_membership_to_open_rooms

def grant_membership_to_open_rooms
  Membership.insert_all(Rooms::Open.pluck(:id).collect { |room_id| { room_id:, user_id: id } })
end
```

That insert omits `involvement`, relying on the column default `"mentions"`
(`db/schema.rb:86`).

### Cost of creating one Open room

Three writes, all synchronous, all in one request:

1. `INSERT INTO rooms` — one row.
2. One multi-row `INSERT INTO memberships`, one row per `User.active`. (Bots have
   `status: active` too — `app/models/user.rb:18` — so bots get rows.)
3. One Turbo Stream broadcast, `app/controllers/rooms/opens_controller.rb:49-51`:
   `broadcast_prepend_to :rooms, target: :shared_rooms, partial: "users/sidebars/rooms/shared"`
   — a single global stream, the partial rendered once. Cheap and constant.

Closed-type rooms broadcast per member instead, but deliberately render the HTML once
(`app/controllers/rooms/closeds_controller.rb:75-80`, comment: "Optimization to avoid
rendering the same partial for every user").

At campaign scale this is nothing: 200 rooms × 40 players = 8,000 membership rows in a
five-column table. A new player joining inserts 200 rows in one statement. **Room
creation cost is not a constraint.**

### There is no non-HTTP creation path

Rooms are only created by `Rooms::OpensController#create`
(`app/controllers/rooms/opens_controller.rb:19-24`), `Rooms::ClosedsController#create`
(`app/controllers/rooms/closeds_controller.rb:19-24`),
`Rooms::Direct.find_or_create_for` (`app/models/rooms/direct.rb:6-8`) and
`FirstRun.create!` (`app/models/first_run.rb:5-15`). A quest model creating its own room
would call `Rooms::Open.create_for` directly, which is a supported public API —
`FirstRun` already does the equivalent at `app/models/first_run.rb:7-12`.

Both HTTP create paths are gated by
`app/controllers/rooms_controller.rb:40-44`, which reads the account setting
`restrict_room_creation_to_administrators` (`app/models/account.rb:9`). The sidebar's
"New Chat Room" button is hidden by the same condition
(`app/views/users/sidebars/show.html.erb:39`). So there is an existing, one-line way to
stop players creating rooms by hand while a quest still creates one automatically.

---

## 2. How does membership work?

### Materialised rows, never resolved dynamically

**An Open room really does write one `memberships` row per user.** Nothing anywhere
resolves "is this room open, therefore everyone is in it" at read time. Every
authorization path in the app goes through the join table:

- `app/controllers/rooms_controller.rb:32-34` — `room_scope` is `Current.user.rooms`
- `app/controllers/concerns/room_scoped.rb:10` — `Current.user.memberships.find_by!(room_id:)`
- `app/channels/room_channel.rb:12` — `current_user.rooms.find_by(id: params[:room_id])`
- `app/channels/room_messages_channel.rb:30` — `user.rooms.find_by(id: room.id)`
- `app/models/user.rb:7` — `has_many :reachable_messages, through: :rooms` (search scope)

So `Rooms::Open` is a *policy about who gets rows*, not a *runtime rule*. Consequence:
if a room's memberships were ever deleted without deleting the room, the room becomes
unreachable to everyone including its creator — see §4.

### The `involvement` enum

`app/models/membership.rb:9`:

```ruby
enum :involvement, %w[ invisible nothing mentions everything ].index_by(&:itself), prefix: :involved_in
```

Human labels at `app/helpers/rooms/involvements_helper.rb:19-24`:

| value | meaning (Campfire's own words) |
|---|---|
| `invisible` | "Notifications are off and room invisible in sidebar" |
| `nothing` | "Notifications are off" |
| `mentions` | "Notifying about @ mentions" |
| `everything` | "Notifying about all messages" |

Defaults: `"mentions"` for every room type (`app/models/room.rb:59-61`, column default
`db/schema.rb:86`), overridden to `"everything"` for directs
(`app/models/rooms/direct.rb:19-21`). Confirmed by
`test/models/room_test.rb:33-36` and `test/models/rooms/direct_test.rb:17-20`.

Two scopes carry all the weight (`app/models/membership.rb:14-15`):

```ruby
scope :visible, -> { where.not(involvement: :invisible) }
scope :unread,  -> { where.not(unread_at: nil) }
```

**Critical detail for the quest-room design: `involvement` does not affect unread
marking, except for `invisible`.** Unread is computed over `memberships.visible`
(`app/models/room.rb:74-76`), so a player who sets a room to `nothing` still gets it
bolded in the sidebar and still counts toward the app badge. Only `invisible` — which
also removes the room from the sidebar entirely — quiets it. The bell control is a
four-state cycle (`app/helpers/rooms/involvements_helper.rb:26-35`), one room at a time,
one click per state. There is no bulk operation and no default-involvement-per-room
setting.

### Granting and revoking

`app/models/room.rb:2-17` — `grant_to` (insert_all), `revoke_from` (`destroy_by user:`),
`revise(granted:, revoked:)` in a transaction. `Rooms::ClosedsController#update` uses it
(`app/controllers/rooms/closeds_controller.rb:33`).

`revoke_from` uses `destroy_by`, so callbacks run. `app/models/membership.rb:7`:

```ruby
after_destroy_commit { user.reset_remote_connections }
```

which does `ActionCable.server.remote_connections.where(current_user: self).disconnect
reconnect: true` (`app/models/user.rb:48-50`, `:61-63`). **Revoking N memberships kicks
every affected player's websocket N times.** A "close down this quest room by revoking
everyone" operation over 40 players is 40 destroys and 40 forced reconnects. Deleting
the room instead uses `dependent: :delete_all` (`app/models/room.rb:2`) and skips all of
that.

Deactivating a player deletes all their non-direct memberships
(`app/models/user.rb:39`), so a deactivated player silently loses every quest room.

---

## 3. How does the sidebar list rooms?

### What builds it

`app/controllers/users/sidebars_controller.rb:4-10`:

```ruby
def show
  all_memberships     = Current.user.memberships.visible.with_ordered_room
  @direct_memberships = extract_direct_memberships(all_memberships)
  @other_memberships  = all_memberships.without(@direct_memberships)

  @direct_placeholder_users = find_direct_placeholder_users
end
```

`with_ordered_room` is `includes(:room).joins(:room).order("LOWER(rooms.name)")`
(`app/models/membership.rb:11`) — so rooms are eager-loaded, no N+1 on `membership.room`.

- **Order**: `LOWER(rooms.name)` in SQL, then re-sorted client-side by name
  (`app/javascript/controllers/sorted_list_controller.js:23-33`). Directs sort by
  recency instead, via a `sorted_list_number` of `room.updated_at`
  (`app/views/users/sidebars/rooms/_direct.html.erb:5`). **Shared rooms have no
  recency sort, server-side or client-side.**
- **Filter**: `visible` only — i.e. involvement ≠ `invisible`. Open and closed-type
  rooms are not distinguished; both render into `#shared_rooms`
  (`app/views/users/sidebars/show.html.erb:33-37`).
- **Paging**: none. No `limit`, no offset, no infinite scroll, no "show more". Every
  visible non-direct membership is rendered.

This is a deliberate contrast with how Campfire treats *user* lists, where it does cap
and filter: `DIRECT_PLACEHOLDERS = 20` (`app/controllers/users/sidebars_controller.rb:2`)
and `user_filter_search_tag if users.count > 20`
(`app/views/rooms/opens/_form.html.erb:28`). Campfire knows how to add a filter box; it
just never needed one for rooms.

### What 200 rooms actually does

**Rendering.** `app/views/users/sidebars/show.html.erb:34-36`:

```erb
<% @other_memberships.each do |membership| %>
  <%= render "users/sidebars/rooms/shared", room: membership.room, unread: membership.unread? %>
<% end %>
```

Note: this is a bare `each` with an uncached partial. The directs immediately above it
*are* a cached collection render (`app/views/users/sidebars/show.html.erb:23`,
`cached: true`, and `_direct.html.erb:1` wraps in `cache membership`). So 200 quest rooms
means 200 uncached partial renders per sidebar request. The partial itself is trivial
(`app/views/users/sidebars/rooms/_shared.html.erb:1-5`) — one `link_to` and a `span` —
so this is ERB overhead, not query overhead.

**Query count.** Two loads of the membership set: `extract_direct_memberships` calls
`.select` on the relation (`app/controllers/users/sidebars_controller.rb:14`), loading
it; then `.without(...)` resolves to `ActiveRecord::Relation#excluding`, issuing a
second `SELECT ... WHERE id NOT IN (...)` with the same `includes(:room)`. Small and
constant-shaped, but it's the full set twice.

**How often it's rebuilt.** This is the part that bites. The sidebar is a lazily-loaded
Turbo Frame (`app/views/rooms/show.html.erb:10`,
`app/helpers/users/sidebar_helper.rb:2-9`) that reloads:

- on cable reconnect — `app/javascript/controllers/rooms_list_controller.js:39-44`,
  `this.element.reload()` on every `#channelConnected` after a disconnect;
- after the tab has been hidden for 60s —
  `app/javascript/controllers/refresh_room_controller.js:6`, `:37-47` dispatching
  `refresh-room:visible`, wired to `turbo-frame#reload` at
  `app/helpers/users/sidebar_helper.rb:7`.

So the whole 200-room list is re-queried, re-rendered and re-sent on every laptop-lid
open and every flaky-wifi reconnect, not once per page load.

**DOM cost.** Every room link carries three Stimulus targets
(`app/helpers/rooms_helper.rb:3-5`: `rooms_list_target`, `badge_dot_target`,
`sorted_list_target`). Consequences at 200 items:

- `rooms_list_controller#findRoomTarget` is a linear scan over 200 targets on **every**
  unread and read event (`app/javascript/controllers/rooms_list_controller.js:62-64`).
- `sorted_list_controller` re-sorts and `appendChild`s **all** 200 nodes on every
  `itemTargetConnected` (`:7-9`, `:32`) — i.e. once per newly broadcast room, throttled.
- `badge_dot_controller` filters all 200 targets on every read/unread event (`:24-26`).

None of this is fatal, but it is O(rooms) work on every message-adjacent event.

**Visual.** `.rooms` is a single flat vertical flex column
(`app/assets/stylesheets/sidebar.css:196-201`) inside a `100dvh` scroller
(`app/assets/stylesheets/sidebar.css:7-11`, `.overflow-y` at
`app/assets/stylesheets/utilities.css:75`). The "new room" button is sticky at the
bottom (`app/assets/stylesheets/sidebar.css:203-207`). Sidebar width is `26vw`
(`app/assets/stylesheets/layout.css:20`), collapsing to an overlay below `100ch`
(`app/assets/stylesheets/sidebar.css:27-73`).

So 200 rooms is **one alphabetical scroll list of 200 pill buttons, no headings, no
grouping, no collapse, no filter box, no counts.** Room names get an
`overflow-ellipsis` (`_shared.html.erb:4`) and nothing else.

**There is no way to find a room by name.** Search is messages-only (§7). The only room
navigation aids that exist are: the sidebar list, `Room.original` as a fallback home
(`app/models/room.rb:42-44`, `app/controllers/concerns/tracked_room_visit.rb:13-18`),
and a `last_room` cookie (`app/controllers/concerns/tracked_room_visit.rb:9`).

Note also `app/controllers/rooms_controller.rb:6-8`:

```ruby
def index
  redirect_to room_url(Current.user.rooms.last)
end
```

`.last` on the `has_many :through` orders by `rooms.id`, so `/rooms` always lands you in
the **most recently created** room. With a room per proposal, that is whatever quest was
proposed last. `WelcomeController#show` (`app/controllers/welcome_controller.rb:3-5`) is
better behaved: last room visited, falling back to `Room.original` — the oldest room,
which `FirstRun` names "All Talk" (`app/models/first_run.rb:4`, `:7`). A campaign-wide
home room is therefore a concept Campfire already has.

The full membership list appears in a second place with the same no-paging problem: the
profile page renders every membership, direct and shared, each with its own involvement
control (`app/controllers/users/profiles_controller.rb:5-7`,
`app/views/users/profiles/show.html.erb:98`,
`app/views/users/profiles/_membership.html.erb:1-13`). That view uses
`Current.user.memberships` *without* `.visible`, so it is the only place an `invisible`
room can be found again and un-hidden.

---

## 4. Archiving, hiding, closing, deactivating

**Campfire does not do any of this.** A grep for `archiv|closed_at|hidden_at|deactivat`
across `app`, `db`, `lib`, `config` returns only `User#deactivate` and unrelated
JavaScript. There is no room status column (`db/schema.rb:119-125`), no soft-delete, no
`discarded_at`, no read-only mode, no lock.

What does exist, and what each is actually good for:

**a. Per-user `invisible` involvement.** `app/models/membership.rb:9`, `:14`. Removes the
room from that user's sidebar and stops it being marked unread
(`app/models/room.rb:74-76` filters on `.visible`). Set through the bell, one room at a
time, four clicks from the default (`app/helpers/rooms/involvements_helper.rb:26-35`).
Broadcasts a targeted sidebar remove for that user only
(`app/controllers/rooms/involvements_controller.rb:16-25`). This is **per player, not
per campaign** — hiding a finished quest room for everyone would mean 40 players each
doing it themselves. There is no admin-side "make this invisible to all".

**b. Open ↔ closed-type conversion.** `app/controllers/rooms/opens_controller.rb:39-41`
and `app/controllers/rooms/closeds_controller.rb:41-43` (`@room.becomes!`). Directs are
blocked from converting by a validation this fork added
(`app/models/room.rb:22`, `:64-72`) and by `room_scope` narrowing
(`app/controllers/rooms/opens_controller.rb:45-47`).

Converting a finished quest room to closed-type with an empty `user_ids` list would
revoke everyone (`app/controllers/rooms/closeds_controller.rb:33`, `:51-61`) and make the
room vanish from every sidebar. **Do not do this**: (i) it fires N membership destroys,
each kicking that player's websocket (§2); (ii) because authorization is purely the
join table (§2), the room becomes unreachable to *everyone including its creator and
administrators* — `room_scope` is `Current.user.rooms`
(`app/controllers/rooms_controller.rb:33`), so there is no admin backdoor. The room and
its messages are orphaned in the database with no UI that can reach them. It is a
one-way trapdoor, not an archive.

**c. Deletion.** The only supported exit. See §5.

Also worth knowing: `config/routes.rb:75` declares `resource :settings, only: :show`
inside the rooms scope, but no `Rooms::SettingsController` exists
(`app/controllers/rooms/` contains only closeds, directs, involvements, opens,
refreshes). That route is dead. There is no per-room settings surface to hang a
lifecycle control on; the nearest thing is the edit page
(`app/views/rooms/layouts/_edit.html.erb`), which is a name field, a membership picker,
and a delete button.

Who may delete or rename: `app/controllers/rooms_controller.rb:3`, `:36-38` →
`Current.user.can_administer?(@room)` → `app/models/user/role.rb:8-10`:
`administrator? || self == record&.creator`. **The player who proposed a quest would be
the room's creator and could therefore delete the quest's room and all its discussion.**

---

## 5. What is deleted along with a room?

`app/controllers/rooms_controller.rb:14-19`:

```ruby
def destroy
  @room.destroy
  broadcast_remove_room
  redirect_to root_url
end
```

Synchronous, in the request. No background job.

**Deleted:**

- **Memberships** — `dependent: :delete_all` (`app/models/room.rb:2`). One `DELETE`, no
  callbacks, so no websocket-kick storm.
- **Messages** — `dependent: :destroy` (`app/models/room.rb:20`). Every message is
  instantiated and destroyed individually. Each one cascades:
  - **Boosts** — `has_many :boosts, dependent: :destroy` (`app/models/message.rb:7`).
  - **Rich text body** — `has_rich_text :body` (`app/models/message.rb:9`), which
    dependent-destroys the `action_text_rich_texts` row (`db/schema.rb:25-33`).
  - **Attachment** — `has_one_attached :attachment`
    (`app/models/message/attachment.rb:8-10`), default `dependent: :purge_later`, so each
    attachment enqueues an Active Storage purge job for its blob and variants
    (`db/schema.rb:35-61`).
  - **Search index row** — `after_destroy_commit :remove_from_index`
    (`app/models/message/searchable.rb:7`, `:21-23`), one
    `delete from message_search_index where rowid = ?` per message.

  So deleting a room with 500 messages is ~500 instantiations, ~2,000+ statements, and
  N purge jobs, all inside one HTTP request against SQLite. For a quest room with a
  week of chatter this is fine; it is worth knowing it is not a constant-time operation.

**Not deleted:**

- **Push subscriptions** — user-scoped, not room-scoped (`db/schema.rb:107-117`,
  `app/models/user.rb:10`). Nothing about a room touches them.
- **Searches** — user-scoped recent-query strings (`app/models/search.rb`,
  `app/models/user.rb:13`), capped at 10 per user (`app/models/search.rb:16`).
- **The FTS table itself** (`db/schema.rb:182`) — rows go one at a time, the virtual
  table is untouched.
- **The `last_room` cookie** — can point at a deleted room; `last_room_visited` handles
  it by falling back to `Room.original`
  (`app/controllers/concerns/tracked_room_visit.rb:13-18`).

**Broadcast:** `app/controllers/rooms_controller.rb:60-62` —
`broadcast_remove_to :rooms, target: [ @room, :list ]`. That is the *global* rooms
stream, which every sidebar subscribes to
(`app/views/users/sidebars/show.html.erb:2`). For a closed-type room this leaks the
removal to non-members, but harmlessly — the target element isn't in their DOM.

The delete button carries a `turbo_confirm` and no undo
(`app/helpers/rooms_helper.rb:25-31`): *"Are you sure you want to delete this room and
all messages in it? This can't be undone."*

---

## 6. Unreads and notification badges — would 200 quest rooms produce 200 badges?

**Yes.**

### How unread is set

`app/models/room.rb:44-47`, `:74-76`:

```ruby
def receive(message)
  unread_memberships(message)
  push_later(message)
end

def unread_memberships(message)
  memberships.visible.disconnected.where.not(user: message.creator).update_all(unread_at: message.created_at, ...)
end
```

Triggered by `after_create_commit -> { room.receive(self) }` (`app/models/message.rb:12`).
`disconnected` means `connected_at` older than 60s (`app/models/membership/connectable.rb:5-9`).

`unread_at` is a **per-membership timestamp** (`db/schema.rb:88`), i.e. a boolean-ish
"has unread" flag, not a count. There is no unread *message count* anywhere in Campfire.

Then `app/models/message/broadcasts.rb:14-18` fans out a cable message per member:

```ruby
room.memberships.pluck(:user_id).each do |user_id|
  ActionCable.server.broadcast UnreadRoomsChannel.stream_name_for(user_id), { roomId: room.id }
end
```

Note this plucks *all* memberships, including `invisible` ones — harmless, since the
client looks for a DOM node that isn't there
(`app/javascript/controllers/rooms_list_controller.js:50-60`).

### How unread is cleared

Visiting a room subscribes to `PresenceChannel`, which calls `membership.present`
(`app/channels/presence_channel.rb:5-9`) → `Membership.connect` sets
`unread_at: nil` (`app/models/membership/connectable.rb:16-18`) and broadcasts
`user_N_reads` so other tabs clear the highlight
(`app/channels/presence_channel.rb:24-26`,
`app/javascript/controllers/read_rooms_controller.js:19-21`).

**Clearing unread requires actually opening the room.** There is no "mark all read", no
"mark this room read", and no bulk action. 200 quest rooms with activity means 200
individual room visits to clear the sidebar.

### The badges

1. **Sidebar bolding** — the `.unread` class on each room link
   (`app/views/users/sidebars/rooms/_shared.html.erb:3`,
   `app/assets/stylesheets/sidebar.css:221-230`). One per room. 200 rooms → up to 200
   bolded entries.
2. **App icon badge (PWA)** — `app/javascript/controllers/badge_dot_controller.js:12-26`:

   ```js
   const unreadCount = this.#unreadCount   // count of DOM targets carrying .unread
   navigator.setAppBadge(unreadCount)
   ```

   `badge_dot_target` is on every room link, direct and shared
   (`app/helpers/rooms_helper.rb:4`). So the number on the app icon is **a count of
   unread rooms**. 200 active quest rooms literally shows `200`.
3. **Push notification badge** — `app/models/push/subscription.rb:5`:
   `badge: user.memberships.unread.count`. Same count, computed in SQL, **per
   subscription, per push** (`lib/web_push/pool.rb:27` builds the notification inside the
   per-subscription loop). One `COUNT(*)` query per subscription per message.
4. **Mobile sidebar-toggle dot** — `app/assets/stylesheets/sidebar.css:37-51`, a single
   dot if *any* room is unread. This one is the only badge that doesn't degrade.

### The interaction that actually hurts

Every quest room is Open, so every player is a member of every quest room (§2). Combined
with "involvement doesn't gate unread except `invisible`" (§2), the result is: **any
message in any of the 200 quest rooms marks all ~40 players unread in that room.** A
player who has no interest in 195 of the quests still sees them all bold and still sees
`200` on the app icon. The only per-player escape hatch is setting each room `invisible`,
four clicks each, 200 times.

Web *push* is better behaved, because the default involvement is `mentions` — see §8.

---

## 7. Search indexing

One global FTS5 virtual table, `message_search_index` (`db/schema.rb:182`), keyed by
`rowid = messages.id` (`app/models/message/searchable.rb:14`). **Not partitioned by
room**, and rooms themselves are not indexed at all.

Maintained by three per-message commit callbacks
(`app/models/message/searchable.rb:5-7`, `:13-23`) — one statement each on message
create, update and destroy.

Scoping to what a user may see happens at query time, not index time
(`app/controllers/searches_controller.rb:21-27`):

```ruby
@messages = Current.user.reachable_messages.search(query).last(100)
```

`reachable_messages` is `has_many through: :rooms` (`app/models/user.rb:7`), so the query
joins `memberships → rooms → messages → message_search_index`
(`app/models/message/searchable.rb:9`). With 200 rooms the membership side of that join
is 200 rows for the searching player — trivial for SQLite. `.last(100)` on an ordered
relation compiles to a reversed `LIMIT 100`, so it does not load everything into Ruby.

**Room count is not a search scaling problem.** But note two design-relevant gaps:

- **You cannot search for a room.** No room-name index, no room filter on the search
  page, no room facet in results. `app/views/searches/index.html.erb` reuses the `.rooms`
  sidebar slot to show *recent queries*, not rooms (`:31-47`).
- Search results do show which room each hit came from, as a small line of text
  (`app/assets/stylesheets/sidebar.css:269-288`, `.message__room`) — so message search is
  the de facto cross-room finder, and it will keep working across 200 rooms.

---

## 8. Push subscription fan-out

`app/models/room/message_pusher.rb`. Per message (enqueued as `Room::PushMessageJob`,
`app/models/room.rb:78-80`, `app/jobs/room/push_message_job.rb`, on Resque in production
per `config/environments/production.rb:95`):

```ruby
def relevant_subscriptions
  Push::Subscription
    .joins(user: :memberships)
    .merge(Membership.visible.disconnected.where(room: room).where.not(user: message.creator))
end
```

then two queries off it (`app/models/room/message_pusher.rb:48-54`):

- `.merge(Membership.involved_in_everything)` → everyone who opted into everything;
- `.merge(Membership.involved_in_mentions).where(user_id: mentionees.ids)` → mentioned
  people only.

**Because the default involvement is `mentions` (§2), an Open quest room does not push to
the whole campaign by default.** Only people who explicitly turned a room up to
"everything", plus people actually @-mentioned. This is the one place where 200 open
rooms is already well-behaved.

Delivery is a 50-thread pool with a 10,000-deep queue and a 150-connection persistent
HTTP pool (`lib/web_push/pool.rb:6-8`), enqueued one subscription at a time via
`find_each` (`:12-16`). Expired subscriptions self-destruct
(`config/initializers/web_push.rb:6-13`). Fan-out is proportional to *subscriptions of
interested users in one room*, never to room count.

---

## 9. Turbo Stream / cable volume

Per sidebar, exactly two Turbo Stream subscriptions regardless of room count
(`app/views/users/sidebars/show.html.erb:2-3`): the global `:rooms` stream and the
per-user `[Current.user, :rooms]` stream. Room pages add one stream for the room being
viewed (`app/views/rooms/show.html.erb:20`), authorized per-subscription by this fork's
`RoomMessagesChannel` (`app/channels/room_messages_channel.rb:1-13` explains why).

Unread notification uses its own per-user stream, `user_N_unreads`
(`app/channels/unread_rooms_channel.rb:5-11`) — the comment there records that a single
global stream was rejected precisely because it leaks room activity.

Per message: 1 Turbo append to the room stream + N cable broadcasts (N = members) + 1
push job (`app/models/message/broadcasts.rb:3-4`, `:15-17`). **Volume scales with players
per room and with message rate, not with room count.** 200 rooms only matter to the
extent that they carry 200 rooms' worth of messages.

The genuinely room-count-dependent cable cost is the sidebar reload (§3): a full
200-room HTML render pushed down the wire on every reconnect and every 60s-hidden
return.

---

## 10. Other things that would bite at 100-200 rooms

- **No recency ordering for shared rooms.** Directs get `sorted_list_number` from
  `room.updated_at` (`app/views/users/sidebars/rooms/_direct.html.erb:5`); shared rooms
  get `sorted_list_name` and sort alphabetically
  (`app/views/users/sidebars/rooms/_shared.html.erb:2`,
  `app/javascript/controllers/sorted_list_controller.js:26-29`). The data to sort by
  recency exists — `Message belongs_to :room, touch: true`
  (`app/models/message.rb:4`) keeps `rooms.updated_at` fresh — it is simply not used for
  shared rooms. Switching shared rooms to recency ordering is a two-line change to the
  partial plus the controller's ordering scope.
- **No room-name uniqueness and no name length limit** (`db/schema.rb:122`, no
  validation in `app/models/room.rb`). Two quests called "Return to the Barrow" are
  indistinguishable in the sidebar, which only ellipsises
  (`app/views/users/sidebars/rooms/_shared.html.erb:4`). With a room per proposal this
  will happen.
- **`/rooms` lands you in the newest room** (`app/controllers/rooms_controller.rb:7`).
- **Uncached shared-room partial** (`app/views/users/sidebars/show.html.erb:34-36`) while
  the direct partial next to it *is* cached — a cheap win if the list stays long.
- **`Rooms::Direct.find_for` is O(all direct rooms) in Ruby**
  (`app/models/rooms/direct.rb:10-16`, with an upstream `FIXME` admitting it). Not
  triggered by quest rooms, but it is the one existing room-count landmine and it shows
  Campfire's own expectation of scale.
- **No fixture, seed or admin tooling for bulk room operations.** No rake task, no
  console helper, nothing under `lib/tasks` touching rooms.

---

## What this constrains

**Open (free choices, already supported by the code):**

- **Creating a room per quest is cheap and safe.** `Rooms::Open.create_for` is a clean
  public API (`app/models/room.rb:33-40`), the membership fan-out is one bulk insert, and
  the create broadcast is a single global Turbo Stream. Nothing about room *creation*
  argues against a room per proposal.
- **Stopping players from creating rooms by hand is a one-setting change**
  (`app/models/account.rb:9`, `app/controllers/rooms_controller.rb:40-44`) — quest rooms
  can be the only rooms that exist without touching Campfire's create paths.
- **A campaign-wide home room already exists as a concept**: `Room.original`, the oldest
  room, is the fallback for a player with no `last_room` cookie
  (`app/models/room.rb:42-44`, `app/controllers/concerns/tracked_room_visit.rb:13-18`).
- **Web push will not spam the campaign.** Default involvement `mentions`
  (`app/models/room.rb:59-61`) means a new quest room pushes only to mentionees.
- **Ordering the sidebar by recency instead of name is a small, local change** — the
  `updated_at` touch is already in place (`app/models/message.rb:4`).
- **Deleting a quest's room is a genuinely complete cleanup** (§5) and the messages,
  boosts, attachments and search rows all go with it. If the design says "a quest's room
  dies with the quest", Campfire supports that today, in one button.

**Closed (must be built, or must be designed around):**

- **There is no archive, and nothing to extend.** No status column, no soft-delete, no
  read-only mode, no per-room admin surface (the `room_settings` route is dead —
  `config/routes.rb:75`). "Retire the room but keep the history" is a schema change plus
  a scope change plus a sidebar change plus a new control. Budget it as a build, not a
  configuration.
- **"Hide it from the sidebar" cannot be done campaign-wide.** `invisible` is per
  membership, set by each player, four clicks per room
  (`app/models/membership.rb:9`, `app/helpers/rooms/involvements_helper.rb:26-35`). There
  is no admin-side hide and no bulk involvement change.
- **Emptying a room's memberships is not an archive — it is an unrecoverable orphan.**
  Authorization is purely the join table with no admin backdoor
  (`app/controllers/rooms_controller.rb:33`), so a room nobody is a member of is
  unreachable by everyone forever. Rule this approach out explicitly.
- **The sidebar cannot hold 200 rooms.** One flat alphabetical scroll list, no paging, no
  grouping, no filter box, no headings, and no way to search for a room by name (§3, §7).
  If quest rooms go in the sidebar, the sidebar needs new structure — that is a build,
  and it is the single largest one this research turns up.
- **Badges will read the room count.** The app icon badge is literally a count of unread
  *rooms* (`app/javascript/controllers/badge_dot_controller.js:24-26`,
  `app/models/push/subscription.rb:5`), and involvement below `invisible` does not
  suppress unread marking (`app/models/room.rb:74-76`). A campaign with 200 live quest
  rooms shows `200` and there is no "mark all read". Either quest rooms stay out of the
  unread system, or unread needs to become opt-in per quest (e.g. members of the party
  and the interested, not the whole campaign).
- **"Everyone is in every room" is a materialised fact, not a rule.** Every Open quest
  room writes a row per player and every new player writes a row per Open room
  (`app/models/rooms/open.rb:6-8`, `app/models/user.rb:53-55`). Scoping a quest room's
  audience to the party would mean `Rooms::Closed` plus membership revision on
  confirmation — and revocation kicks websockets, one destroy at a time
  (`app/models/membership.rb:7`).
- **The quest's proposer would own the room's delete button.** `can_administer?` grants
  the room's creator full rights (`app/models/user/role.rb:8-10`), so whoever creates the
  room can delete every message in it. If quests create rooms with `creator: Current.user`
  the proposer gets that power; setting the creator to a system/GM user is the lever.
- **Whatever the exit is, it must not be "delete on completion" if the room is the
  record.** ADR 0002 already says the **report** is the campaign's only durable record;
  §5 confirms Campfire will happily and irreversibly take the room's whole discussion
  with it. The two decisions are coupled: deleting quest rooms is only safe once reports
  exist.
