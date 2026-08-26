# Campfire's identity, roles and onboarding

Research for issue `02-campfire-identity-roles-and-onboarding`. Every claim below is
cited to this repo at `main` (e63402d). Paths are relative to the repo root.

Vocabulary note: Campfire's `Account` is our Campaign, its `Room` is a quest's room,
its `Session` is an auth session (`docs/adr/0001-hard-fork-of-campfire.md:18-20`). Below
I use Campfire's class names when talking about code and `CONTEXT.md`'s words when
talking about the design.

---

## 1. What roles exist today?

### The role enum

One integer column, one enum, three values:

- `app/models/user/role.rb:5` — `enum :role, %i[ member administrator bot ]`
- `db/schema.rb:154` — `t.integer "role", default: 0, null: false` on `users`

No `_prefix`/`_suffix`, so Rails generates the predicates `member?`, `administrator?`,
`bot?` and the class scopes `User.member`, `User.administrator`, `User.bot`. The scope
`User.administrator` is used in anger at `app/views/accounts/_help_contact.html.erb:1`,
where the *first* administrator is treated as the deployment's owner/support contact —
the only place anything resembling a hierarchy above "administrator" survives.

The values are positional integers, so **appending** a fourth value is safe and
**inserting** one is not. There is precedent for the enum having changed shape: the
original enum had an `owner` role at index 2, removed by
`db/migrate/20240115124901_remove_owner_role.rb:3-4` (`update users set role = 1 where
role = 2`). Campfire has already collapsed three human roles into two once.

### The single authorization predicate

`app/models/user/role.rb:8-10`:

```ruby
def can_administer?(record = nil)
  administrator? || self == record&.creator || record&.new_record?
end
```

That is the entire authorization model. There are no per-capability checks, no policy
objects, no Pundit/CanCan. Authority is: *are you an administrator, or did you create
this thing* (plus a permissive clause for unsaved records, asserted deliberately in
`test/models/user/role_test.rb:16-21`).

### Complete inventory of authorization sites

Every place `role` or `can_administer?` is consulted. This is the surface a third role
would have to touch.

**The shared before_action** — `app/controllers/concerns/authorization.rb:3-5`,
`ensure_can_administer` → `head :forbidden unless Current.user.can_administer?` (no
argument, so strictly `administrator?`). Mixed into every controller via
`app/controllers/application_controller.rb:2`. Used by:

| Site | Effect |
| --- | --- |
| `app/controllers/accounts_controller.rb:2` (`update`) | rename the campaign, upload logo, flip settings |
| `app/controllers/accounts/join_codes_controller.rb:2` | regenerate the join code |
| `app/controllers/accounts/logos_controller.rb:5` (`destroy`) | delete the logo |
| `app/controllers/accounts/custom_styles_controller.rb:2` | edit custom CSS |
| `app/controllers/accounts/users_controller.rb:2` (`update`, `destroy`) | change someone's role; deactivate them |
| `app/controllers/accounts/bots_controller.rb:2` | full CRUD on bots |
| `app/controllers/accounts/bots/keys_controller.rb:2` | reset a bot key |
| `app/controllers/users/bans_controller.rb:2` | ban and unban |

**Overridden, record-scoped variants** (admin *or* creator):

- `app/controllers/rooms_controller.rb:36-38` — `can_administer?(@room)`, gating
  `destroy` (`rooms_controller.rb:3`) and, via subclasses, `update`
  (`app/controllers/rooms/opens_controller.rb:3`,
  `app/controllers/rooms/closeds_controller.rb:3`).
- `app/controllers/messages_controller.rb:57-59` — `can_administer?(@message)`, gating
  `edit`/`update`/`destroy` (`messages_controller.rb:6`). Inherited by
  `app/controllers/messages/by_bots_controller.rb:8`, which is how a bot gets authority
  over its own messages — through the `creator` clause, not through its role.
- `app/controllers/rooms/directs_controller.rb:34-36` — overridden to `true`; everyone
  in a direct room can administer it.

**Role checked directly, not via `can_administer?`:**

- `app/controllers/rooms_controller.rb:41` — `ensure_permission_to_create_rooms`:
  `Current.account.settings.restrict_room_creation_to_administrators? && !Current.user.administrator?`.
  Applied at `app/controllers/rooms/opens_controller.rb:6` and
  `app/controllers/rooms/closeds_controller.rb:6`. Notably *not* applied to
  `Rooms::DirectsController` — direct rooms are always creatable.
- `app/controllers/accounts_controller.rb:7` — `users.partition(&:administrator?)`.
- `app/controllers/accounts_controller.rb:26` — `can_administer?` decides whether the
  roster includes banned users.
- `app/controllers/accounts/users_controller.rb:24` — the role whitelist (see §5).
- `app/controllers/users/avatars_controller.rb:14` — `@user.bot?` picks the default
  bot avatar.

**Views:**

- `app/helpers/application_helper.rb:40` — `admin_body_class` adds `"admin"` to `<body>`
  for anyone who `can_administer?`; CSS then shows/hides admin affordances.
- `app/views/accounts/edit.html.erb:8` — `administrator?` gates the bots and
  custom-styles nav links.
- `app/views/accounts/edit.html.erb:24` — `can_administer?` gates the whole
  campaign-settings form; the `else` at line 91 shows a read-only logo + name.
- `app/views/accounts/edit.html.erb:103-109` — renders `@administrators` then a divider
  then `@members`.
- `app/views/accounts/users/_user.html.erb:12,13,16,18` — the role toggle.
- `app/views/accounts/_invite.html.erb:28` — `can_administer?` gates regenerating the
  join link (but *not* seeing or sharing it — see §2).
- `app/views/users/show.html.erb:36,49,56` — `can_administer?` reveals someone's email
  address, the session-transfer link, and the ban button.
- `app/views/users/sidebars/show.html.erb:39` — `administrator? || !restrict_room_creation…`
  gates the "New room" button.
- `app/views/rooms/opens/_user.html.erb:12`, `app/views/rooms/closeds/_user.html.erb:12`,
  `app/views/rooms/layouts/_edit.html.erb:13`, `app/views/rooms/layouts/_form.html.erb:3,25`,
  `app/views/rooms/opens/_form.html.erb:15`, `app/views/rooms/closeds/_form.html.erb:3` —
  `can_administer?(room)`.
- `app/views/accounts/_help_contact.html.erb:1` — `User.administrator.first`.

**Serialised out over the API:** `app/views/users/_user.json.jbuilder:2` emits
`:id, :name, :role`. A new role value becomes part of the bot API's payload immediately
(asserted at `test/controllers/messages/by_bots_controller_test.rb:86`).

### So: what can an administrator do that a member cannot?

Rename the campaign and change its logo and custom CSS; toggle
`restrict_room_creation_to_administrators`; regenerate the join code; promote/demote
anyone else; deactivate anyone else; create, edit and delete bots and reset their keys;
ban and unban; see everyone's email address; see anyone's session-transfer (auto-login)
link; see banned users in the roster; and destroy or rename any room, not just their
own. A member can do all of the ordinary things — create rooms (unless restricted),
post, edit and delete *their own* messages, create direct rooms, invite by sharing the
join link.

### The `Ban` model

`app/models/ban.rb` is **not a role**. It is an IP blocklist:

- `app/models/ban.rb:2` — `belongs_to :user` (the user the IP was harvested from)
- `db/schema.rb:63-70` — `bans` is `(user_id, ip_address)`
- `app/models/ban.rb:6-8` — `Ban.banned?(ip_address)` is a bare `exists?` on ip
- `app/models/ban.rb:11-19` — rejects loopback/private/link-local addresses
- `app/controllers/concerns/block_banned_requests.rb:9-15` — rejects a banned IP with
  `429`, but **only for non-GET/HEAD requests**; banned IPs can still read.

The user-facing state lives in a *second* enum: `app/models/user.rb:18` —
`enum :status, %i[ active deactivated banned ], default: :active`,
`db/schema.rb:155`. So Campfire already has two orthogonal enums on `User`: `role`
(what you may do) and `status` (whether you exist). `status` replaced a boolean
`active` column in `db/migrate/20251126115722_change_active_to_status_on_users.rb`.

### Bots

Bots are `User` rows with `role: :bot` and no email or password:

- `app/models/user/bot.rb:15` — `User.create!(**attributes, bot_token:, role: :bot)`
- `app/controllers/accounts/bots_controller.rb:37` — bot params are only
  `:name, :avatar, :webhook_url`
- `app/models/user/bot.rb:5-6` — `active_bots` / `without_bots` scopes
- `app/models/user/bot.rb:38-40` — `bot_key` is `"#{id}-#{bot_token}"`
- `app/models/user/bot.rb:20-23` — `authenticate_bot` splits the key and matches
  against `active_bots`

Bots authenticate by URL segment, not cookie:
`app/controllers/concerns/authentication.rb:43-48` sets `Current.user` from
`params[:bot_key]`; `config/routes.rb:66` puts `:bot_key` in the path. CSRF is skipped
for them (`authentication.rb:10`). Crucially, `authentication.rb:7` installs a global
`before_action :deny_bots`, and `authentication.rb:94-96` `head :forbidden` for anything
authenticated by bot key. Bots are then re-admitted one controller at a time via
`allow_bot_access` — only two exist:
`app/controllers/messages/by_bots_controller.rb:4` and
`app/controllers/messages/boosts/by_bots_controller.rb:4`.

**This is the seam.** Campfire's third role is not implemented by adding branches to
the member/administrator UI. It is implemented by (a) a deny-by-default guard that
excludes the role from everything, (b) an explicit allowlist of the handful of places
it belongs, and (c) `without_bots` scoping to keep it out of the human roster
(`app/controllers/accounts_controller.rb:6`,
`app/controllers/accounts/users_controller.rb:5`). Every human-facing surface still
believes there are exactly two roles; the third one is simply routed around.

---

## 2. How does someone join?

### First run — the bootstrap

`app/models/first_run.rb:5-15`, in one call:

1. `Account.create!(name: ACCOUNT_NAME)` — `first_run.rb:6`. **The account name is
   hardcoded to `"Campfire"`** (`first_run.rb:2`); `FirstRunsController` never permits
   an account param (`app/controllers/first_runs_controller.rb:25`). It is renamed
   afterwards through `AccountsController#update`.
2. Creates `Rooms::Open` named `"All Talk"` (`first_run.rb:3,7`).
3. `User.new(user_params.merge(role: :administrator))` — `first_run.rb:9`. **The first
   user is the only user ever created as an administrator by the system.** Every other
   administrator is promoted by hand.
4. Grants that user membership of the first room (`first_run.rb:12`).

The controller is unauthenticated (`first_runs_controller.rb:2`), guarded by
`prevent_repeats` → `redirect_to root_url if Account.any?` (`first_runs_controller.rb:20-22`),
and the resulting race is closed at the database level: `accounts.singleton_guard` has a
unique index (`db/schema.rb:20-22`,
`db/migrate/20251212154340_add_singleton_constraint_to_accounts.rb`), and the second
concurrent request is caught by `rescue ActiveRecord::RecordNotUnique` at
`first_runs_controller.rb:15-16`. This is tested at
`test/controllers/first_runs_controller_test.rb:34-56`. This unique index *is* the
single-tenancy the ADR relies on; `Current.account` is literally `Account.first`
(`app/models/current.rb:14-16`).

`SessionsController` redirects to first run when there are no users at all
(`app/controllers/sessions_controller.rb:5,26-28`).

### The join code — the only invitation mechanism

- `app/models/account/joinable.rb:5` — `before_create { self.join_code = generate_join_code }`
- `app/models/account/joinable.rb:13-15` — `SecureRandom.alphanumeric(12)` chunked into
  `xxxx-xxxx-xxxx`
- `app/models/account/joinable.rb:8-10` — `reset_join_code`
- `db/schema.rb:17` — `join_code` is `null: false` on `accounts`
- `config/routes.rb:32-33` — `get/post "join/:join_code"` → `UsersController#new`/`#create`
- `app/controllers/users_controller.rb:27-29` — `verify_join_code`:
  `head :not_found if Current.account.join_code != params[:join_code]`
- `app/controllers/users_controller.rb:2` — `require_unauthenticated_access only: %i[new create]`
- `app/controllers/users_controller.rb:12-14` — creates the user and immediately signs
  them in; `rescue ActiveRecord::RecordNotUnique` bounces an existing email to sign-in
  (`users_controller.rb:15-16`)
- `app/controllers/users_controller.rb:31-33` — self-service params: `:name, :avatar,
  :email_address, :password`. **`role` is not permitted here**, so everyone who joins by
  link is a `member` (enum default 0, `db/schema.rb:154`).

There are **no invitation records, no per-person invite tokens, no email sending, and no
approval step.** Campfire does not do invitations at all. There is one shared,
account-wide secret URL. Anyone holding it can create an account. Anyone signed in can
read and share it — `app/views/accounts/_invite.html.erb:2-26` shows the URL, a copy
button, a QR button and a web-share button to *every* signed-in user; only regeneration
is admin-gated (`_invite.html.erb:28`). The join form itself is
`app/views/users/new.html.erb`.

Consequence: joining is a *self-serve, unreviewed* act. There is nowhere in this
codebase to hang "an admin approves this person" or "this person is a Game Master from
the start".

### QR codes

`app/controllers/qr_code_controller.rb` is a generic, unauthenticated
(`qr_code_controller.rb:2`) SVG renderer: it base64-decodes `params[:id]` and draws a QR
of whatever URL that is (`qr_code_controller.rb:5-9`), cached for a year. It is not a
join flow of its own; it is used to render the join link
(`app/views/accounts/_invite.html.erb:13`) and the session-transfer link
(`app/views/users/profiles/_transfer.html.erb:24`) via
`app/helpers/qr_code_helper.rb:2-6`.

### New-user side effect

`app/models/user.rb:22,53-55` — `after_create_commit :grant_membership_to_open_rooms`
inserts a `Membership` for every `Rooms::Open`. Conversely
`app/models/rooms/open.rb:6-8` grants every active user membership when a room becomes
open. Open rooms are therefore campaign-wide by construction, which matches
`CONTEXT.md`'s "Open to the whole campaign, not just the party".

---

## 3. Authentication, and the one-identity assumption

### Mechanism

- `app/models/session.rb:4` — `has_secure_token` (`sessions.token`, unique index at
  `db/schema.rb:143`)
- `app/models/session.rb:6` — `belongs_to :user`; `app/models/user.rb:15` —
  `has_many :sessions, dependent: :destroy`. **A user may hold many sessions**; each row
  records `user_agent`, `ip_address`, `last_active_at` (`db/schema.rb:135-145`).
  Multi-device is expected, and `Session#resume` refreshes at most hourly
  (`session.rb:2,14-18`).
- `app/controllers/concerns/authentication.rb:6` — `before_action :require_authentication`
  on every controller
- `authentication.rb:33-35` — `restore_authentication || bot_authentication || request_authentication`
- `app/controllers/concerns/authentication/session_lookup.rb:2-6` — session found by the
  signed `:session_token` cookie
- `authentication.rb:86-88` — cookie is `signed.permanent`, `httponly`, `same_site: :lax`
- `app/controllers/sessions_controller.rb:11` —
  `User.active.authenticate_by(email_address:, password:)`; note the `active` scope, so
  deactivated and banned users cannot sign in
- `app/controllers/sessions_controller.rb:3` — rate limited to 10 per 3 minutes
- `app/models/user.rb:20` — `has_secure_password validations: false`; there is **no
  password length/presence validation** and no confirmation
  (`test/models/user_test.rb:4-7` asserts a 300-char password is fine).
  `db/migrate/20240131105830_alter_users_set_password_digest_not_null.rb` backfilled
  `''` and made the column non-null, so a passwordless user has an unmatchable digest.
- `app/channels/application_cable/connection.rb:5,12-18` — ActionCable reuses the same
  cookie lookup and is `identified_by :current_user`.

### Session transfer (a second, link-based credential)

`app/models/user/transferable.rb:12-14` mints `signed_id(purpose: :transfer,
expires_in: 4.hours)`; `app/controllers/sessions/transfers_controller.rb:2,8-9` accepts
it unauthenticated and starts a session for `User.active.find_by_transfer_id`. Admins can
see and share *other people's* transfer links
(`app/views/users/show.html.erb:49-53`, `app/views/users/profiles/_transfer.html.erb:12-16`).
This is a deliberate impersonation-adjacent recovery path: an administrator can put
anyone back into their own account without a password reset. There is no password reset
flow otherwise.

### Does anything assume one identity per person?

Yes, but weakly, and only at the account level:

- `db/schema.rb:158` — unique index on `users.email_address`. The column is **nullable**,
  which is what lets bots exist without one (SQLite permits multiple NULLs in a unique
  index), and multiple bots do (`app/models/user/bot.rb:15` never sets an email).
- `app/models/user.rb:44,57-59` — deactivation *mangles* the email
  (`foo@bar` → `foo-deactivated-<uuid>@bar`), explicitly freeing the address so the same
  human can create a fresh `User` later. Deactivation is designed to be non-final.
- There is no `Account#users` association at all (`app/models/account.rb` has none);
  "in the campaign" simply means "a `User` row exists". `AccountsController#account_users`
  is a global `User.where(...)` (`accounts_controller.rb:25-31`).

So: one email per user, many sessions per user, no notion of linked or secondary
identities, and no concept anywhere of one person owning several sub-entities. **The
`User` → `Character` (one-to-many) relationship `CONTEXT.md` needs has no analogue in
Campfire.** The closest structural precedent is `User has_many :memberships` — a
per-room join row, not a per-campaign one.

---

## 4. Deactivation and banning: what is preserved, what is severed

### `User#deactivate` — `app/models/user.rb:35-46`

In one transaction:

| Action | Line |
| --- | --- |
| disconnect live ActionCable connections | `user.rb:37`, `user.rb:61-63` |
| `memberships.without_direct_rooms.delete_all` | `user.rb:39` |
| `push_subscriptions.delete_all` | `user.rb:40` |
| `searches.delete_all` | `user.rb:41` |
| `sessions.delete_all` | `user.rb:42` |
| `status: :deactivated` + mangled email | `user.rb:44` |

**Preserved:** every `Message` they wrote, and every `Boost`. Nothing in `deactivate`
touches `messages`. **Their direct-room memberships also survive** —
`without_direct_rooms` (`app/models/membership.rb:12`) deliberately spares them, so DM
history stays reachable for the other party. Their `role` is preserved too: a
deactivated administrator still answers `administrator?`, and
`app/views/accounts/_help_contact.html.erb:1` (`User.administrator.first`) does not
filter by status.

**Severed:** access (no sessions, cannot sign in — `sessions_controller.rb:11` scopes to
`User.active`), all shared/open/closed room memberships, push, search history, and the
email address as an identifier.

The `User` row is never destroyed by the app. `Accounts::UsersController#destroy`
(`app/controllers/accounts/users_controller.rb:13-16`) calls `deactivate`, despite the
UI copy saying "permanently remove … can't be undone"
(`app/views/accounts/users/_user.html.erb:25`). `app/models/user.rb:8` does declare
`has_many :messages, dependent: :destroy`, so a real `User#destroy` *would* take the
messages with it — but nothing in the application ever calls it.

Deactivation is **not reversible through the UI**: there is no reactivate action
anywhere (`Accounts::UsersController` has only `index`/`update`/`destroy`, and both
`update` and `destroy` scope to `User.active` at line 20).

In the roster and profile, a deactivated user shows as "is no longer on this account"
(`app/views/users/show.html.erb:61-66`), and is excluded from
`app/controllers/accounts/users_controller.rb:5`,
`app/controllers/rooms/opens_controller.rb:16,27`,
`app/controllers/rooms/closeds_controller.rb:16,28`,
`app/controllers/users/sidebars_controller.rb:19` and
`app/controllers/autocompletable/users_controller.rb:8`.

Bots are deactivated by the same method
(`app/controllers/accounts/bots_controller.rb:27`).

### `User#ban` — `app/models/user/bannable.rb:4-10`, `31-41`

1. Harvests every distinct IP from the user's sessions into `Ban` rows
   (`bannable.rb:31-35`).
2. Disconnects cable, `sessions.delete_all`, enqueues `RemoveBannedContentJob`
   (`bannable.rb:37-41`, `app/jobs/remove_banned_content_job.rb`).
3. `banned!`.

`remove_banned_content` (`bannable.rb:23-28`) **destroys every message they ever wrote**
and broadcasts the removal. This is the one path in the codebase that deletes a user's
content.

**Preserved by a ban:** memberships (all of them — ban does *not* call the membership
cleanup deactivate does) and the real email address (not mangled). **Severed:** sessions,
all messages, and network access from their recorded IPs for any non-GET request
(`app/controllers/concerns/block_banned_requests.rb:9-15`).

`unban` (`bannable.rb:12-17`) deletes the `Ban` rows and sets `active!`. The messages do
not come back. Banned users remain visible to admins in the roster
(`app/controllers/accounts_controller.rb:27`) with a `banned` CSS class
(`app/views/accounts/users/_user.html.erb:1`, `app/views/users/show.html.erb:19`), which
is how they can be unbanned (`app/views/users/_ban_button.html.erb:8-14`).

Ban is gated by `ensure_can_administer` at `app/controllers/users/bans_controller.rb:2`,
and the button hidden from self at `app/views/users/show.html.erb:56`.

Summary: **deactivate = the person leaves, their words stay. ban = the person is
expelled, their words are erased.** For a campaign, deactivate is the right default for
someone who stops playing; the report/quest record survives, which matters given
`CONTEXT.md` calls Report "the campaign's only durable record of the world".

---

## 5. Does anything assume exactly two roles?

Yes. `bot` is technically a third value, but every human-facing surface is written for
two, and bots are held out of those surfaces rather than accommodated by them.

1. **`app/controllers/accounts/users_controller.rb:23-25`** — the hard whitelist:
   ```ruby
   { role: params.require(:user)[:role].presence_in(%w[ member administrator ]) || "member" }
   ```
   Any value that is not exactly `"administrator"` silently becomes `"member"`. A new
   role posted here is *downgraded, not rejected*. This is the single line that must
   change for any third assignable role.
2. **`app/views/accounts/users/_user.html.erb:18`** — the control is a
   `form.check_box :role, {...}, "administrator", "member"` — a boolean toggle whose two
   states *are* the two roles. There is no room for a third state.
3. **`app/views/accounts/users/_user.html.erb:16`** — the screen-reader label is
   `user.administrator? ? "Administrator" : "Member"` — a binary ternary.
4. **`app/controllers/accounts_controller.rb:7`** —
   `users.partition(&:administrator?)` into `@administrators, @members`, rendered as
   exactly two groups with one divider at
   `app/views/accounts/edit.html.erb:103-109`.
5. **`app/controllers/accounts_controller.rb:6` and
   `app/controllers/accounts/users_controller.rb:5`** — `.without_bots`
   (`app/models/user/bot.rb:6`). The roster is defined as "the users who are not the
   third role".
6. **`app/models/user/role.rb:8-10`** — `can_administer?` is one boolean. There is no
   vocabulary for "may do X but not Y". A third role that is *not* a strict superset or
   subset of member has nothing to hook into.
7. **`app/views/accounts/_help_contact.html.erb:1`** — `User.administrator.first` stands
   in for a fourth, vestigial role (owner) that the enum no longer has
   (`db/migrate/20240115124901_remove_owner_role.rb`).
8. **`test/models/user/role_test.rb:5,9-21`, `test/controllers/accounts/users_controller_test.rb:9-29`,
   `test/controllers/accounts_controller_test.rb:13-56`, `test/fixtures/users.yml:7,13`,
   `config/environments/performance.rb:16`** — all written in terms of two human roles.

The role is also **single-valued**. A `User` has exactly one `role`. There is no way to
be an administrator *and* something else, which collides directly with `CONTEXT.md`:
"Game Masters are also players when they aren't running."

---

## 6. Where a third role slots in with least disruption

### What Campfire actually offers as precedent

- `role` — a single-valued integer enum on `users` for *what you may do*
  (`app/models/user/role.rb:5`).
- `status` — a *second*, orthogonal single-valued enum on the same table for *whether
  you exist* (`app/models/user.rb:18`). Campfire's own answer to "we need another axis
  on User" was a second column, not a model.
- `Membership` — a join row, but it is **room-scoped, not campaign-scoped**
  (`app/models/membership.rb:4-5`, `db/schema.rb:82-95`).
- **There is no campaign membership model.** `Account` has no `users` association
  (`app/models/account.rb` in full), and account-level queries are global `User` scopes
  (`app/controllers/accounts_controller.rb:25-31`). A "per-campaign membership" would be
  new invention with no seam to attach to, and the singleton `Account`
  (`db/schema.rb:20-22`, `app/models/current.rb:14-16`) means it would carry exactly one
  distinct value forever. **Ruled out** — it also contradicts
  `docs/adr/0001-hard-fork-of-campfire.md:12-16`.

### The three options, assessed

**(a) A fourth value on the `role` enum** — `app/models/user/role.rb:5` becomes
`%i[ member administrator bot game_master ]`. Appending is index-safe (existing rows
unaffected; `db/migrate/20240115124901_remove_owner_role.rb` shows what happens when
indices *do* move). No migration needed; the column already exists.
Cost: the whitelist at `accounts/users_controller.rb:24`, the checkbox at
`accounts/users/_user.html.erb:18` and the partition at `accounts_controller.rb:7` all
have to be rewritten from binary to n-ary.
**Fatal flaw:** a role is single-valued. `game_master` would mean a Game Master is *not*
a `member` and *not* an `administrator`, so every `member`-shaped behaviour would have
to be re-granted to them explicitly, and a Game Master who is also an admin becomes
unrepresentable. This contradicts `CONTEXT.md`'s "Game Masters are also players when
they aren't running."

**(b) A new boolean column on `users` (e.g. `game_master`)** — mirrors what Campfire did
for `status`: a second, orthogonal axis on the same table. `role` keeps meaning
"administrative authority over the deployment"; the new flag means "may offer slots and
confirm quests". They compose freely: a player who is also a GM, a GM who is also an
admin, an admin who never runs. New authorization sites would be new predicates
(`Current.user.game_master?`) alongside the existing `can_administer?` rather than
inside it, so **not one existing authorization site in §1 needs to change** — the
inventory above stays valid, and quest/slot controllers add a new `before_action`
modelled on `app/controllers/concerns/authorization.rb:3-5`.
The admin UI change is additive: a second toggle next to the crown at
`app/views/accounts/users/_user.html.erb:14-20`, and a second permitted key alongside
`accounts/users_controller.rb:24`. Note `role` is serialised at
`app/views/users/_user.json.jbuilder:2`, so a new column is *not* silently exposed —
that's a decision, not an accident.

**(c) A separate model** (e.g. `GameMastery belongs_to :user`) — buys nothing at this
scale. A campaign has "several" Game Masters (`CONTEXT.md:19-20`), single-tenant, with
no per-relationship attributes identified. It duplicates the `Membership` shape without
`Membership`'s reason to exist (a room to scope to). Reconsider only if a GM assignment
needs its own attributes (granted-by, granted-at, per-quest rather than per-campaign).

**Recommendation from the code: (b).** It follows the precedent Campfire itself set with
`status`, it is compositional where `role` is exclusive, and it leaves the 30-odd
authorization sites inventoried in §1 untouched.

---

## What this constrains

**Opened:**

- Adding a Game Master axis is a one-column migration plus one predicate. It does not
  require touching any existing authorization site, because Campfire concentrates all
  role authority in one method (`app/models/user/role.rb:8-10`) and one `before_action`
  (`app/controllers/concerns/authorization.rb:3-5`), and quest authority is a new
  question those don't answer.
- Deactivation is already the right shape for "a player leaves": it preserves their
  messages and their DM history and severs access
  (`app/models/user.rb:35-46`). Quest/character/report records attached to a `User` will
  survive it by default, since `deactivate` deletes only memberships, sessions, pushes
  and searches by name. Any new association is preserved unless we opt it in.
- The bot role is a working template for "a role that is excluded from the human roster":
  `without_bots` (`app/models/user/bot.rb:6`) + deny-by-default + explicit allowlist
  (`app/controllers/concerns/authentication.rb:7,18-20,94-96`). If Game Masters ever need
  to be *hidden from* a player-facing list, that pattern already exists.
- Multi-device is free: `User has_many :sessions` (`app/models/user.rb:15`), and the
  transfer link (`app/models/user/transferable.rb:12-14`) is an existing, admin-shareable
  account-recovery path — no password-reset flow needs building.

**Closed, or expensive:**

- **A fourth `role` enum value is the wrong shape.** `role` is single-valued
  (`app/models/user/role.rb:5`), so `game_master` would exclude being a `member`, which
  `CONTEXT.md:19-26` explicitly forbids. Use a separate column.
- **A per-campaign membership model has nothing to attach to.** `Account` has no `users`
  association at all, and the account is a singleton by unique index
  (`db/schema.rb:20-22`). Building one means inventing a layer the ADR
  (`docs/adr/0001-hard-fork-of-campfire.md:12-16`) says we are not building.
- **There is no invitation system to extend.** One shared join code
  (`app/models/account/joinable.rb:5,13-15`,
  `app/controllers/users_controller.rb:27-29`), visible to every signed-in user
  (`app/views/accounts/_invite.html.erb:2-26`), no per-person tokens, no email, no
  approval. "Someone becomes a Game Master at join time" cannot be expressed today —
  joining always yields `member` (`app/controllers/users_controller.rb:31-33`). Either
  Game Master is always a post-hoc promotion by an admin, or the join flow gains
  something Campfire does not have.
- **There is no `Character`-shaped precedent.** No one-person-many-personas relationship
  exists anywhere; the only `User has_many` of a domain thing is `memberships`, which is
  room-scoped. `Character` is entirely new construction.
- **The role UI is a binary toggle, not a picker.** `accounts/users/_user.html.erb:18`
  (checkbox with two role values), `accounts_controller.rb:7` (two-way partition),
  `accounts/edit.html.erb:103-109` (two groups) and
  `accounts/users_controller.rb:24` (two-value whitelist, silently downgrading anything
  else) all have to be rewritten if a third *assignable* value ever lands in `role`.
  Option (b) avoids all four.
- **Deactivation is one-way in the UI.** No reactivate action exists
  (`app/controllers/accounts/users_controller.rb:20` scopes to `User.active`), and the
  email is deliberately freed for re-registration (`app/models/user.rb:57-59`). A player
  who comes back returns as a *new* `User` and loses any characters attached to the old
  row — unless we build reactivation or re-parent characters.
- **A deactivated administrator keeps `role: administrator`**
  (`app/models/user.rb:44` never clears it), and `User.administrator.first`
  (`app/views/accounts/_help_contact.html.erb:1`) does not filter by status. Whatever
  "the Game Masters of this campaign" query we write must scope by `status: :active`
  explicitly; nothing does it for us.
- **Banning erases content.** `app/models/user/bannable.rb:23-28` destroys every message
  the user wrote. If Reports are `Message`-backed or `User`-owned, a ban would take the
  campaign's durable record of the world with it. Ban must not be the mechanism for
  "this player left".
- **`role` is public API.** `app/views/users/_user.json.jbuilder:2` emits it to bots.
  Any change to the enum changes that payload
  (`test/controllers/messages/by_bots_controller_test.rb:86`).
