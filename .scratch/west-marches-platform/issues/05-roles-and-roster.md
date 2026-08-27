# How does someone become a player, a Game Master, or an admin?

Type: grilling
Status: resolved
Blocked by: 02

## Question

The campaign has several Game Masters, players with characters, and someone who runs the
deployment. Pin down how people enter and change state.

- How does a new person get on the roster? Open join code, invitation by an existing
  member, or admin approval?
- Who grants the Game Master role, and can it be revoked? Is Game Master a property of a
  person, or something they hold for a given quest?
- Is a Game Master also a player, with their own characters, who can go on other people's
  quests? (The glossary assumes yes.)
- What happens to a person who leaves: their characters, their proposed quests, their
  reports, and their contribution to other players' "played least recently" standing?
- Does a player create their own characters freely, or does a Game Master have to admit
  them to the campaign?
- Is there anything the admin does that a Game Master cannot? Should those be the same role?

Read the findings from ticket `02` first.

## Answer

**Nobody is admitted to anything.** Entry, character creation and the ordinary departure all
resolve to *change nothing, or nearly nothing* — this campaign's roster is a social object and
the app should stop pretending to guard it. The two decisions with teeth are that **Game Master
and administrator are separate standing flags with no implicit escalation**, and that **ban can
never be the mechanism for leaving**, because under ADR 0004 it destroys the campaign's memory.

### Entry: the shared join code, shown to administrators only

Keep Campfire's one account-wide join code (`app/models/account/joinable.rb:5`). No invitations,
no per-person tokens, no approval queue, and therefore no pending roster state to model, notify
or render. Ticket `04`'s precedent holds: don't build what practice hasn't asked for.

**One change.** Restrict *display* of the join code to administrators — today every signed-in
user can see and forward it, and only regeneration is admin-gated. Roster size is not cosmetic
here: under ADR 0003 every person on the roster dilutes everyone else's standing, so growth
should be a deliberate act rather than a link anyone can forward. The real gate stays social and
out of band, which is where a West Marches group already keeps it.

### Game Master is a standing flag; the Game Master *of* a quest is derived

Ticket `02` settled the storage — a separate boolean, never a fourth value of the single-valued
`role` enum (`app/models/user/role.rb:5`), because `game_master` as a role would stop a Game
Master being a member. This ticket settles the model.

- **Person-level, not per-quest.** A person either may offer slots or may not; that is what the
  roster page and every "who can confirm this" check needs to ask cheaply.
- **The Game Master of a quest is not stored.** A quest is confirmed into a **slot**, and a slot
  belongs to whoever offered it, so "who is running this one" is already a fact the schema holds.
  Nothing extra to write, and no way for the two to disagree.
- **Granted and revoked by administrators only.** Letting any Game Master anoint another is the
  more sociable model and has no revocation authority: nobody can say who un-anoints, or what
  happens when two disagree.
- **Revocation withdraws unspent future slots**, because a slot is an offer of that particular
  person's time and the offer no longer stands. **Quests already confirmed into their slots are
  untouched** — someone still has to run them, and that is a conversation, not a state
  transition. The "confirmed quest with nobody to run it" case goes to ticket `06` alongside
  cancellation.
- **Departure requires no revocation.** Ticket `02`'s trap already forces every "who are the Game
  Masters" query to scope `active`; a deactivated Game Master is simply not one.

### Administrator and Game Master are orthogonal, and neither implies the other

`can_administer?` must not confer Game Master, and the Game Master flag must confer nothing
administrative. Ticket `02` already puts the flag outside `role`, so separation costs nothing to
build. Three reasons to spend it:

- **They fail differently.** A Game Master's bad call is social and recoverable. An admin's is
  operational and frequently not.
- **Ban is a loaded weapon.** `remove_banned_content` destroys every message a person ever wrote
  (`app/models/user/bannable.rb:24-29`). Collapsing the roles hands that button to every new
  Game Master by default.
- **The cardinalities differ.** A campaign wants several Game Masters and one or two admins. In
  practice whoever stood the deployment up carries both flags on one row; that is a coincidence,
  not an identity.

### A Game Master is a full player, and the exclusion is derived

Yes to the glossary's assumption, without qualification: characters, proposed quests (subject to
ticket `14`), interest, seats, and the **Scribe** debt like anyone else.

The one rule: **a character is never a candidate for a slot offered by its own player.** It
cannot be enforced at interest time — a Game Master may declare interest in a quest *and* offer
a slot for it, and the contradiction only becomes real if that quest is confirmed into that
slot. So interest is never restricted; the character is simply absent from the pool for that one
slot, and the confirmation screen (ticket `11`) says why. Like the derivation above it stores
nothing and cannot fall out of sync.

Whether *running* a quest counts toward a Game Master's own "played least recently" standing is
**ticket `15`'s**, which may replace recency ranking outright. The instinct on record: running is
not playing.

### Characters are created freely

A player creates their own characters, with a name and a level, and nobody admits them. Levels
are earned at the table and known to the group socially; a player who inflates one is caught by
the Game Master who runs them, not by a form. Q16 already establishes the level check is
**advisory, never refused** — the system does not enforce on that field, so it has no business
policing it either. Admission would put a Game Master in the loop for the most routine act a
player performs.

Deliberately unanswered: **who edits a level, and when**, plus death and retirement. `map.md`
already flags character lifecycle as its own future ticket and it stays there.

### Leaving: silent by default, amended deactivation for the hard exit

Both inherited exits are wrong out of the box.

- **Ban destroys the campaign's memory.** `messages.each(&:destroy)` across every room,
  asynchronously, and `unban` does not restore it (`app/models/user/bannable.rb:24-29,12-17`).
  Under ADR 0004 the trail *is* the product. **Ban must never be the departure mechanism**, and
  the button should come out of the UI.
- **Deactivate is nearly right and then destroys the identity.** It keeps messages, name and
  avatar and drops the person from active lists (`app/models/user.rb:35-45`) — but it rewrites
  `email_address` to `name-deactivated-<uuid>@domain` and stores the original nowhere
  (`app/models/user.rb:44,57-59`). Email is the only identity key, and signup does no matching
  against prior records (`app/controllers/users_controller.rb:12-17`), so a returning player is
  structurally a stranger with no route back to their characters.

**The ordinary departure is silent and needs no state at all.** People go quiet for six months
and come back; a player who declares no interest is already absent from every party, and no
button, state or notification is required to express that. Do not build a "leave" control.

**For the genuine hard exit, keep admin-initiated deactivation, amended twice:** stop mangling
the email address, and add a `deactivated_at` timestamp — `users.status` is a bare three-value
integer that records *that* a transition happened and never *when*, and there are no soft-delete
or lifecycle timestamps anywhere in `db/schema.rb`.

This also answers ticket `08`'s handoff: a deactivated player's messages stay in frozen rooms,
and that is the right call. Deactivation removes access, never history.

### What survives a departure

- **Characters survive untouched** and stay visible on every past party and attendance record.
  They never reappear in a candidate pool for the simple reason that their player declares no
  interest. Nothing needs deleting.
- **Unconfirmed quests they proposed are withdrawn** on explicit deactivation: a quest is a
  proposal of intent and nobody is left to answer questions in its room. Withdrawal is terminal,
  which freezes the room per ticket `08` and keeps the discussion in the trail. A silently
  departed player's quests go stale instead — ticket `06`'s transition, not this ticket's.
  (**`06` answered it:** there is no stale state. They are **abandoned** by the clock, 14 days
  after the last moment anything could still confirm them.)
- **An owed report lapses and is never reassigned.** Ticket `07` already has the obligation die
  at 14 days with the quest reading "no report" for good. Reassigning a Scribe would be the first
  crack in `07`'s rule that a Game Master may never write or correct one, and it invents a
  pressure mechanism the format doesn't need.
- **Published reports survive verbatim, always.** ADR 0004 makes this non-negotiable, and it is
  the whole reason ban is disqualified above.

### Coming back

**Explicit admin reactivation**, not auto-reactivation on login. Since the ordinary six-month
disappearance now carries no state, the only people ever explicitly deactivated are the hard
exits — exactly the case where the exit should stick until someone decides otherwise.

Reactivation is a real path that has to be built, not a status flip:
`SessionsController` authenticates against `User.active` (`app/controllers/sessions_controller.rb:11`),
so a deactivated player cannot log in at all, and `Rooms::Open#grant_to(User.active)`
(`app/models/rooms/open.rb:7`) means flipping the status alone never restores their access to Open
rooms — which under ticket `08` is *every quest room*. So: restore `active`, re-grant Open room
memberships, touch neither characters nor trail.

Preserving the email closes the duplicate-identity trap for free: a returning player who tries to
sign up fresh now collides with the unique index on `email_address` and is bounced to the login
page. That is already the behaviour (`app/controllers/users_controller.rb:12-17`); it is now the
correct one.

### Inherited bugs this ticket's decisions depend on

- **`DELETE /users/:id/ban` un-deactivates.** `Users::BansController#destroy` loads via unscoped
  `User.find` and calls `unban`, which does `active!` (`app/controllers/users/bans_controller.rb:11,17`;
  `app/models/user/bannable.rb:12-17`) — producing an active user with a mangled email, no
  memberships and no way to log in. The UI hides the button; the route is reachable.
- **`Rooms::Open#grant_to(User.active)`** (`app/models/rooms/open.rb:7`) silently excludes
  deactivated users, so reactivation never restores open-room access.
- **`User.administrator.first`** is unscoped (`app/views/accounts/_help_contact.html.erb:1`) and
  will mail a deactivated admin's mangled address.
- **`User has_many :messages, dependent: :destroy`** (`app/models/user.rb:8`) while the account UI's
  confirm text reads "permanently remove this person from the account? This can't be undone"
  (`app/views/accounts/users/_user.html.erb:25`) and in fact calls `deactivate`. A one-word "fix"
  of that mismatch would wipe the trail. Rename the action, or the copy.

### Handed on

- **Does a departed player's past attendance still weigh on other players' standing?** Ticket
  `15`'s, deliberately. It may replace recency ranking outright, and answering it here would be
  guessing at its conclusion. **Answered by `15`: no, and it dissolved rather than being
  decided.** The rota's sort key is entirely individual — your bumps, your last seat — so
  nobody's history moves anybody else's position, and a departure changes nothing for anyone
  who stays.
- **A confirmed quest whose Game Master was revoked or departed** — ticket `06`, with cancellation.
- **Character level, death and retirement** — the character-lifecycle ticket `map.md` already flags.
- **A Game-Masters-only room**, raised by ticket `08` as this ticket's to raise. Raised, and
  deliberately not settled: the orthogonal flag makes it cheap to *scope*, but it is a
  `Rooms::Closed` whose membership has to track a flag that changes, and its real justification is
  **multi-Game-Master continuity** — a `map.md` item with no owner. It belongs to that effort, not
  to the roster.
